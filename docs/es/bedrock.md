---
title: Bedrock
summary: "Usa modelos de Amazon Bedrock (Converse API) con OpenClaw"
read_when:
  - Quieres usar modelos de Amazon Bedrock con OpenClaw
  - Necesitas configurar credenciales y región de AWS para invocar modelos
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw puede usar modelos de **Amazon Bedrock** a través del proveedor de streaming **Bedrock Converse** de pi‑ai. La autenticación de Bedrock usa la **cadena de credenciales predeterminada del SDK de AWS**,
no una clave de API.

<div id="what-piai-supports">
  ## Qué admite pi‑ai
</div>

- Proveedor: `amazon-bedrock`
- API: `bedrock-converse-stream`
- Autenticación: credenciales de AWS (variables de entorno, configuración compartida o rol de instancia)
- Región: `AWS_REGION` o `AWS_DEFAULT_REGION` (valor predeterminado: `us-east-1`)

<div id="automatic-model-discovery">
  ## Descubrimiento automático de modelos
</div>

Si se detectan credenciales de AWS, OpenClaw puede descubrir automáticamente
modelos de Bedrock que admitan **streaming** y **salida de texto**. Este
descubrimiento utiliza `bedrock:ListFoundationModels` y se almacena en caché
(de forma predeterminada, durante 1 hora).

Las opciones de configuración se encuentran en `models.bedrockDiscovery`:

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

Notas:

* `enabled` es `true` de forma predeterminada cuando hay credenciales de AWS configuradas.
* `region` toma por defecto `AWS_REGION` o `AWS_DEFAULT_REGION`, y en su defecto `us-east-1`.
* `providerFilter` coincide con nombres de proveedores de Bedrock (por ejemplo, `anthropic`).
* `refreshInterval` se mide en segundos; configúralo en `0` para deshabilitar la caché.
* `defaultContextWindow` (predeterminado: `32000`) y `defaultMaxTokens` (predeterminado: `4096`)
  se usan para modelos detectados (sobrescríbelos si conoces los límites de tu modelo).


<div id="setup-manual">
  ## Configuración (manual)
</div>

1. Asegúrate de que las credenciales de AWS estén disponibles en el **host del Gateway**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Opcional (clave de API/token de portador de Bedrock):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. Añade un proveedor Bedrock y un modelo a tu configuración (no se requiere `apiKey`):

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
  ## Roles de instancias EC2
</div>

Cuando ejecutes OpenClaw en una instancia de EC2 con un rol de IAM asociado, el SDK
de AWS utilizará automáticamente el servicio de metadatos de la instancia (IMDS) para la autenticación.
Sin embargo, la detección de credenciales de OpenClaw actualmente solo comprueba las variables
de entorno y no las credenciales obtenidas mediante IMDS.

**Solución temporal:** Establece `AWS_PROFILE=default` para indicar que las credenciales de AWS
están disponibles. La autenticación real sigue usando el rol de la instancia a través de IMDS.

```bash
# Agregar a ~/.bashrc o al perfil de tu shell
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Permisos de IAM necesarios** para el rol de la instancia de EC2:

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels` (para descubrimiento automático)

O bien adjunta la política administrada `AmazonBedrockFullAccess`.

**Configuración rápida:**

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

# 4. Configurar las variables de entorno de la solución alternativa
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Verify models are discovered
openclaw models list
```


<div id="notes">
  ## Notas
</div>

- Bedrock requiere que el **acceso al modelo** esté habilitado en tu cuenta/región de AWS.
- El descubrimiento automático requiere el permiso `bedrock:ListFoundationModels`.
- Si usas perfiles, establece `AWS_PROFILE` en el host donde se ejecuta el Gateway.
- OpenClaw determina la fuente de las credenciales en este orden: `AWS_BEARER_TOKEN_BEDROCK`,
  luego `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, luego `AWS_PROFILE` y luego
  la cadena predeterminada del SDK de AWS.
- La compatibilidad con capacidades de razonamiento depende del modelo; consulta la model card del modelo de Bedrock para ver
  las capacidades actuales.
- Si prefieres un flujo con claves gestionadas, también puedes colocar un proxy compatible con OpenAI
  delante de Bedrock y configurarlo como un proveedor de OpenAI en su lugar.