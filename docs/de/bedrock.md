---
title: Bedrock
summary: "Amazon-Bedrock-Modelle (Converse API) mit OpenClaw verwenden"
read_when:
  - Du möchtest Amazon-Bedrock-Modelle mit OpenClaw verwenden
  - Du musst AWS-Anmeldedaten und Region für Modellaufrufe konfigurieren
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw kann **Amazon Bedrock**‑Modelle über pi‑ais **Bedrock Converse**‑
Streaming‑Anbieter verwenden. Die Bedrock‑Authentifizierung nutzt die **Standard‑Credential‑Chain („default credential chain“) des AWS SDK** –
nicht einen API‑Schlüssel.

<div id="what-piai-supports">
  ## Was pi‑ai unterstützt
</div>

- Anbieter: `amazon-bedrock`
- API: `bedrock-converse-stream`
- Auth: AWS-Zugangsdaten (Umgebungsvariablen, gemeinsame Konfigurationsdatei oder Instanzrolle)
- Region: `AWS_REGION` oder `AWS_DEFAULT_REGION` (Standard: `us-east-1`)

<div id="automatic-model-discovery">
  ## Automatische Modellerkennung
</div>

Wenn AWS-Zugangsdaten erkannt werden, kann OpenClaw Bedrock-Modelle, die
**Streaming** und **Textausgabe** unterstützen, automatisch erkennen.
Die Erkennung verwendet `bedrock:ListFoundationModels` und wird
zwischengespeichert (Standard: 1 Stunde).

Konfigurationsoptionen befinden sich unter `models.bedrockDiscovery`:

```json5
{
  models: {
    bedrockDiscovery: {
      enabled: true,
      region: "us-east-1",
      providerFilter: ["anthropic", "amazon"],
      refreshInterval: 3600,
      defaultContextWindow: 32000,
      defaultMaxTokens: 4096
    }
  }
}
```

Hinweise:

* `enabled` ist standardmäßig auf `true` gesetzt, wenn AWS-Zugangsdaten vorhanden sind.
* `region` verwendet standardmäßig `AWS_REGION` oder `AWS_DEFAULT_REGION` und fällt andernfalls auf `us-east-1` zurück.
* `providerFilter` entspricht Bedrock-Anbieternamen (zum Beispiel `anthropic`).
* `refreshInterval` ist in Sekunden angegeben; auf `0` setzen, um Caching zu deaktivieren.
* `defaultContextWindow` (Standard: `32000`) und `defaultMaxTokens` (Standard: `4096`)
  werden für automatisch entdeckte Modelle verwendet (setze sie explizit, wenn du die Limits deines Modells kennst).


<div id="setup-manual">
  ## Einrichtung (manuell)
</div>

1. Stelle sicher, dass AWS-Anmeldeinformationen auf dem **Gateway-Host** vorhanden sind:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Optional (Bedrock-API-Schlüssel/Bearer-Token):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. Fügen Sie Ihrer Konfiguration einen Bedrock-Anbieter und ein Modell hinzu (kein `apiKey` erforderlich):

```json5
{
  models: {
    providers: {
      "amazon-bedrock": {
        baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
        api: "bedrock-converse-stream",
        auth: "aws-sdk",
        models: [
          {
            id: "anthropic.claude-opus-4-5-20251101-v1:0",
            name: "Claude Opus 4.5 (Bedrock)",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  },
  agents: {
    defaults: {
      model: { primary: "amazon-bedrock/anthropic.claude-opus-4-5-20251101-v1:0" }
    }
  }
}
```


<div id="ec2-instance-roles">
  ## EC2-Instanzrollen
</div>

Wenn du OpenClaw auf einer EC2-Instanz mit einer zugewiesenen IAM-Rolle ausführst, verwendet das AWS SDK
automatisch den Instance Metadata Service (IMDS) für die Authentifizierung.
Die Anmeldeinformations-Erkennung von OpenClaw prüft derzeit jedoch nur auf Umgebungsvariablen,
nicht auf IMDS-Anmeldeinformationen.

**Workaround:** Setze `AWS_PROFILE=default`, um zu signalisieren, dass AWS-Anmeldeinformationen
verfügbar sind. Die eigentliche Authentifizierung erfolgt weiterhin über die Instanzrolle via IMDS.

```bash
# Zu ~/.bashrc oder Ihrem Shell-Profil hinzufügen
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Erforderliche IAM-Berechtigungen** für die EC2-Instanzrolle:

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels` (für automatische Erkennung)

Oder verwende die verwaltete Richtlinie `AmazonBedrockFullAccess`.

**Schnellstart:**

```bash
# 1. Create IAM role and instance profile
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. Attach to your EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. Auf der EC2-Instanz Discovery aktivieren
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. Set the workaround env vars
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```


<div id="notes">
  ## Hinweise
</div>

- Bedrock erfordert in deinem AWS-Account/deiner Region aktivierten **Modellzugriff**.
- Automatische Erkennung benötigt die Berechtigung `bedrock:ListFoundationModels`.
- Wenn du Profile verwendest, setze `AWS_PROFILE` auf dem Gateway-Host.
- OpenClaw ermittelt die Anmeldedatenquelle in dieser Reihenfolge: `AWS_BEARER_TOKEN_BEDROCK`,
  dann `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, dann `AWS_PROFILE`, dann die
  Standard-Credential-Chain des AWS SDK.
- Unterstützung für Reasoning-Funktionen hängt vom Modell ab; prüfe die Bedrock-Modellkarte für
  den aktuellen Funktionsumfang.
- Wenn du einen verwalteten Schlüssel-Workflow bevorzugst, kannst du auch einen OpenAI‑kompatiblen
  Proxy vor Bedrock schalten und ihn stattdessen als OpenAI-Anbieter konfigurieren.