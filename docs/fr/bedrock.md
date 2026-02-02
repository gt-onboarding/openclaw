---
title: Bedrock
summary: "Utiliser les modèles Amazon Bedrock (Converse API) avec OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles Amazon Bedrock avec OpenClaw
  - Vous avez besoin de configurer les identifiants AWS et la région pour les appels de modèles
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw peut utiliser les modèles **Amazon Bedrock** au moyen du fournisseur de streaming **Bedrock Converse** de pi‑ai. L’authentification Bedrock s’appuie sur la **chaîne d’identifiants par défaut du SDK AWS**, et non sur une clé d’API.

<div id="what-piai-supports">
  ## Ce que pi‑ai prend en charge
</div>

- Fournisseur : `amazon-bedrock`
- API : `bedrock-converse-stream`
- Authentification : identifiants AWS (variables d'environnement, configuration partagée ou rôle d'instance)
- Région : `AWS_REGION` ou `AWS_DEFAULT_REGION` (par défaut : `us-east-1`)

<div id="automatic-model-discovery">
  ## Découverte automatique des modèles
</div>

Si des informations d’identification AWS sont détectées, OpenClaw peut découvrir automatiquement les
modèles Bedrock qui prennent en charge le **streaming** et la **sortie textuelle**. La découverte utilise
`bedrock:ListFoundationModels` et est mise en cache (par défaut : 1 heure).

Les options de configuration se trouvent dans `models.bedrockDiscovery` :

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

Notes :

* `enabled` a pour valeur par défaut `true` lorsque des identifiants AWS sont présents.
* `region` utilise par défaut `AWS_REGION` ou `AWS_DEFAULT_REGION`, puis `us-east-1`.
* `providerFilter` correspond aux noms de fournisseurs Bedrock (par exemple `anthropic`).
* `refreshInterval` est exprimé en secondes ; définissez-le sur `0` pour désactiver la mise en cache.
* `defaultContextWindow` (par défaut : `32000`) et `defaultMaxTokens` (par défaut : `4096`)
  sont utilisés pour les modèles découverts (remplacez-les si vous connaissez les limites de votre modèle).


<div id="setup-manual">
  ## Configuration (manuelle)
</div>

1. Assurez-vous que les identifiants AWS sont disponibles sur l’**hôte du Gateway** :

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Optionnel (clé api/jeton bearer Bedrock) :
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. Ajoutez un fournisseur Bedrock et un modèle à votre configuration (aucune `apiKey` n’est nécessaire) :

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
  ## Rôles d’instance EC2
</div>

Lorsque vous exécutez OpenClaw sur une instance EC2 avec un rôle IAM associé, le SDK
AWS utilise automatiquement le service de métadonnées d’instance (IMDS) pour l’authentification.
Cependant, la détection des identifiants par OpenClaw ne vérifie actuellement que les variables
d’environnement, et non les identifiants IMDS.

**Solution de contournement :** définissez `AWS_PROFILE=default` pour indiquer que des identifiants
AWS sont disponibles. L’authentification effective continue d’utiliser le rôle d’instance via IMDS.

```bash
# Ajouter à ~/.bashrc ou à votre profil shell
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Autorisations IAM requises** pour le rôle d’instance EC2 :

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels` (pour la découverte automatique)

Ou associez la stratégie gérée `AmazonBedrockFullAccess`.

**Configuration rapide :**

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

# 4. Définir les variables d'environnement de contournement
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```


<div id="notes">
  ## Remarques
</div>

- Bedrock nécessite que **l’accès au modèle** soit activé dans votre compte/région AWS.
- La découverte automatique requiert la permission `bedrock:ListFoundationModels`.
- Si vous utilisez des profils, définissez `AWS_PROFILE` sur l’hôte du Gateway.
- OpenClaw détermine la source des identifiants dans cet ordre : `AWS_BEARER_TOKEN_BEDROCK`,
  puis `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, puis `AWS_PROFILE`, puis la
  chaîne par défaut du SDK AWS.
- La prise en charge du raisonnement dépend du modèle ; consultez la fiche du modèle Bedrock pour
  connaître les capacités actuelles.
- Si vous préférez un flux de gestion de clés managé, vous pouvez aussi placer un proxy compatible OpenAI
  devant Bedrock et le configurer comme fournisseur OpenAI à la place.