---
title: Bedrock
summary: "Usa i modelli Amazon Bedrock (Converse API) con OpenClaw"
read_when:
  - Vuoi usare i modelli Amazon Bedrock con OpenClaw
  - Devi configurare le credenziali e la regione AWS per le chiamate ai modelli
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw può utilizzare i modelli **Amazon Bedrock** tramite il **streaming provider Bedrock Converse** di pi‑ai. L'autenticazione per Bedrock utilizza la **catena predefinita delle credenziali dell'AWS SDK**, non una chiave API.

<div id="what-piai-supports">
  ## Cosa supporta pi‑ai
</div>

- Provider: `amazon-bedrock`
- API: `bedrock-converse-stream`
- Autenticazione: credenziali AWS (variabili d'ambiente, configurazione condivisa o ruolo dell'istanza)
- Regione: `AWS_REGION` o `AWS_DEFAULT_REGION` (valore predefinito: `us-east-1`)

<div id="automatic-model-discovery">
  ## Rilevamento automatico dei modelli
</div>

Se vengono rilevate credenziali AWS, OpenClaw può individuare automaticamente i
modelli Bedrock che supportano **streaming** e **output di testo**. Il
rilevamento utilizza `bedrock:ListFoundationModels` e il risultato viene
memorizzato in cache (valore predefinito: 1 ora).

Le opzioni di configurazione si trovano sotto `models.bedrockDiscovery`:

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

Note:

* `enabled` è impostato su `true` per impostazione predefinita quando sono presenti credenziali AWS.
* `region` usa per impostazione predefinita `AWS_REGION` o `AWS_DEFAULT_REGION`, altrimenti `us-east-1`.
* `providerFilter` corrisponde ai nomi dei provider Bedrock (ad esempio `anthropic`).
* `refreshInterval` è in secondi; imposta a `0` per disabilitare la memorizzazione in cache.
* `defaultContextWindow` (predefinito: `32000`) e `defaultMaxTokens` (predefinito: `4096`)
  vengono utilizzati per i modelli rilevati automaticamente (sovrascrivi se conosci i limiti del tuo modello).


<div id="setup-manual">
  ## Configurazione (manuale)
</div>

1. Assicurati che le credenziali AWS siano disponibili sull&#39;**host che esegue il Gateway**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Opzionale (chiave API/bearer token Bedrock):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. Aggiungi un provider Bedrock e un modello alla configurazione (non è richiesta alcuna `apiKey`):

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
  ## Ruoli delle istanze EC2
</div>

Quando esegui OpenClaw su un&#39;istanza EC2 con un ruolo IAM associato, l&#39;AWS SDK
userà automaticamente il servizio di metadati dell&#39;istanza (IMDS) per
l&#39;autenticazione. Tuttavia, il rilevamento delle credenziali di OpenClaw al
momento verifica solo le variabili d&#39;ambiente, non le credenziali IMDS.

**Soluzione alternativa:** imposta `AWS_PROFILE=default` per segnalare che le
credenziali AWS sono disponibili. L&#39;autenticazione effettiva continua a
utilizzare il ruolo dell&#39;istanza tramite IMDS.

```bash
# Aggiungi a ~/.bashrc o al profilo della tua shell
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Autorizzazioni IAM necessarie** per il ruolo dell&#39;istanza EC2:

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels` (per la scoperta automatica)

In alternativa, associa la policy gestita `AmazonBedrockFullAccess`.

**Configurazione rapida:**

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

# 3. On the EC2 instance, enable discovery
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. Imposta le variabili d'ambiente per la soluzione alternativa
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```


<div id="notes">
  ## Note
</div>

- Bedrock richiede che **model access** sia abilitato nel tuo account/region AWS.
- La discovery automatica richiede l'autorizzazione `bedrock:ListFoundationModels`.
- Se usi i profili, imposta `AWS_PROFILE` sull'host del Gateway.
- OpenClaw individua l'origine delle credenziali in questo ordine: `AWS_BEARER_TOKEN_BEDROCK`,
  poi `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, poi `AWS_PROFILE`, quindi la
  catena predefinita dell'SDK AWS.
- Il supporto per il reasoning dipende dal modello; controlla la model card di Bedrock per
  le funzionalità attuali.
- Se preferisci un flusso con chiave gestita, puoi anche posizionare un proxy compatibile con OpenAI
  davanti a Bedrock e configurarlo invece come provider OpenAI.