---
title: Bedrock
summary: "在 OpenClaw 中使用 Amazon Bedrock（Converse API）模型"
read_when:
  - 你想在 OpenClaw 中使用 Amazon Bedrock 模型
  - 你需要为模型调用配置 AWS 凭证和区域信息
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw 可以通过 pi‑ai 的 **Bedrock Converse** 流式提供方来使用 **Amazon Bedrock** 模型。Bedrock 身份验证使用 **AWS SDK 默认凭证链**，而不是使用 API 密钥。

<div id="what-piai-supports">
  ## pi‑ai 的支持范围
</div>

- 提供方：`amazon-bedrock`
- API：`bedrock-converse-stream`
- 身份验证：AWS 凭证（环境变量、共享配置或实例角色）
- 区域：`AWS_REGION` 或 `AWS_DEFAULT_REGION`（默认：`us-east-1`）

<div id="automatic-model-discovery">
  ## 自动模型发现
</div>

如果检测到 AWS 凭证，OpenClaw 可以自动发现支持**流式输出**和**文本输出**的 Bedrock
模型。发现过程会使用 `bedrock:ListFoundationModels`，并会进行缓存（默认：1 小时）。

配置选项位于 `models.bedrockDiscovery` 下：

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

Notes:

* 当检测到 AWS 凭证时，`enabled` 的默认值为 `true`。
* `region` 默认使用 `AWS_REGION` 或 `AWS_DEFAULT_REGION`，否则为 `us-east-1`。
* `providerFilter` 用于匹配 Bedrock 提供方名称（例如 `anthropic`）。
* `refreshInterval` 的单位为秒；设置为 `0` 可禁用缓存。
* `defaultContextWindow`（默认：`32000`）和 `defaultMaxTokens`（默认：`4096`）
  用于自动发现的模型（如果你清楚所用模型的限制，可以在此覆盖这些默认值）。


<div id="setup-manual">
  ## 手动配置
</div>

1. 确保 AWS 凭证已在 **Gateway 主机** 上配置好：

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# 可选 (Bedrock API 密钥/bearer 令牌):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. 在你的配置中添加 Bedrock 提供方及模型（无需 `apiKey`）：

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
  ## EC2 实例角色
</div>

当在附加了 IAM 角色的 EC2 实例上运行 OpenClaw 时，AWS SDK 会自动使用实例元数据服务（IMDS）进行身份验证。
但是，OpenClaw 当前的凭据检测只会检查环境变量，不会检查来自 IMDS 的凭据。

**变通方案：** 将 `AWS_PROFILE=default` 设置为环境变量，用于表明 AWS 凭据是可用的。实际的身份验证仍然是通过 IMDS 使用实例角色完成的。

```bash
# 添加到 ~/.bashrc 或你的 shell 配置文件中
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**EC2 实例角色所需的 IAM 权限**：

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels`（用于自动发现）

或者为其附加托管策略 `AmazonBedrockFullAccess`。

**快速设置：**

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

# 3. 在 EC2 实例上启用模型发现
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
  ## 说明
</div>

- 使用 Bedrock 前，你的 AWS 账户/区域必须启用**模型访问（model access）**。
- 自动发现功能需要 `bedrock:ListFoundationModels` 权限。
- 如果你使用配置文件（profile），请在 Gateway 所在主机上设置 `AWS_PROFILE`。
- OpenClaw 按以下顺序获取凭证来源：首先是 `AWS_BEARER_TOKEN_BEDROCK`，
  然后是 `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`，然后是 `AWS_PROFILE`，最后是
  默认的 AWS SDK 凭证链。
- 推理能力是否可用取决于具体模型；请查看相应的 Bedrock 模型卡了解
  当前能力。
- 如果你更倾向于托管密钥管理流程，也可以在 Bedrock 前面放置一个兼容 OpenAI 的代理，
  并将其配置为一个 OpenAI 提供方。