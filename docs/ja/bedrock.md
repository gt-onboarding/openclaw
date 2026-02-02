---
title: Bedrock
summary: "OpenClaw で Amazon Bedrock (Converse API) モデルを使う"
read_when:
  - OpenClaw で Amazon Bedrock モデルを使いたい
  - モデル呼び出し用の AWS クレデンシャルやリージョン設定が必要
---

<div id="amazon-bedrock">
  # Amazon Bedrock
</div>

OpenClaw は、pi‑ai の **Bedrock Converse**
ストリーミングプロバイダーを通じて **Amazon Bedrock** モデルを利用できます。Bedrock での認証は API キー方式ではなく、**AWS SDK のデフォルト認証チェーン** を使用します。

<div id="what-piai-supports">
  ## pi‑ai がサポートする項目
</div>

- プロバイダー: `amazon-bedrock`
- API: `bedrock-converse-stream`
- 認証: AWS 認証情報（環境変数、共有設定ファイル、またはインスタンスロール）
- リージョン: `AWS_REGION` または `AWS_DEFAULT_REGION`（デフォルト: `us-east-1`）

<div id="automatic-model-discovery">
  ## モデルの自動検出
</div>

AWS の認証情報が検出されると、OpenClaw は **ストリーミング** と
**テキスト出力** をサポートする Bedrock モデルを自動検出します。
検出処理では `bedrock:ListFoundationModels` を使用し、その結果は
（デフォルトでは 1 時間）キャッシュされます。

設定オプションは `models.bedrockDiscovery` 配下にあります:

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

注意:

* AWS の認証情報が存在する場合、`enabled` のデフォルト値は `true` です。
* `region` のデフォルト値は、`AWS_REGION` または `AWS_DEFAULT_REGION` があればそれを使用し、それ以外の場合は `us-east-1` です。
* `providerFilter` は Bedrock のプロバイダー名にマッチします (例: `anthropic`)。
* `refreshInterval` は秒数です。キャッシュを無効にするには `0` を設定します。
* `defaultContextWindow` (デフォルト: `32000`) と `defaultMaxTokens` (デフォルト: `4096`)
  は検出されたモデルに対して使用されます (利用しているモデルの制約が分かっている場合は上書きしてください)。


<div id="setup-manual">
  ## セットアップ（手動）
</div>

1. **Gateway ホスト**上で AWS の認証情報が利用できる状態になっていることを確認します。

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Optional:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# オプション (Bedrock API キー/ベアラートークン):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2. 設定ファイルに Bedrock プロバイダーとモデルを追加します（`apiKey` は不要です）：

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
  ## EC2 インスタンスロール
</div>

IAM ロールがアタッチされた EC2 インスタンス上で OpenClaw を実行している場合、AWS SDK は
認証のために自動的にインスタンスメタデータサービス (IMDS) を使用します。
ただし、OpenClaw のクレデンシャル検出は現在、環境変数のみを確認しており、
IMDS を通じたクレデンシャルは検出しません。

**回避策:** `AWS_PROFILE=default` を設定して、AWS クレデンシャルが利用可能であることを
示します。実際の認証処理自体は、引き続き IMDS 経由のインスタンスロールを使用します。

```bash
# ~/.bashrc またはシェルプロファイルに追加してください
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

EC2 インスタンスロールに必要な **IAM 権限**:

* `bedrock:InvokeModel`
* `bedrock:InvokeModelWithResponseStream`
* `bedrock:ListFoundationModels`（自動検出用）

または、マネージドポリシー `AmazonBedrockFullAccess` をアタッチします。

**クイックセットアップ:**

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

# 3. EC2インスタンス上でディスカバリーを有効化
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
  ## 注意事項
</div>

- Bedrock を利用するには、使用している AWS アカウントとリージョンで **モデルアクセス** が有効化されている必要があります。
- 自動検出には `bedrock:ListFoundationModels` 権限が必要です。
- プロファイルを使用する場合は、Gateway を稼働させているホスト上で `AWS_PROFILE` を設定してください。
- OpenClaw は認証情報の取得元を次の優先順位で扱います：`AWS_BEARER_TOKEN_BEDROCK`、
  次に `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`、次に `AWS_PROFILE`、最後に
  デフォルトの AWS SDK クレデンシャルチェーンです。
- Reasoning の対応状況はモデルに依存します。最新の機能については Bedrock のモデルカードを確認してください。
- マネージドキー方式を優先する場合は、Bedrock の前段に OpenAI 互換のプロキシを配置し、
  それを OpenAI プロバイダーとして構成することもできます。