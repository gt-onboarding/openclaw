---
title: 音声通話
summary: "音声通話プラグイン: Twilio/Telnyx/Plivo 経由の発信・着信通話（プラグインのインストール、設定、CLI）"
read_when:
  - OpenClaw から音声通話を発信したいとき
  - 音声通話プラグインを設定または開発しているとき
---

<div id="voice-call-plugin">
  # Voice Call (plugin)
</div>

プラグイン経由で OpenClaw に音声通話機能を追加します。発信通知と、
着信ポリシー付きのマルチターン会話をサポートします。

対応プロバイダー:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (開発用/ネットワーク接続なし)

ざっくりした流れ:

- プラグインをインストールする
- Gateway を再起動する
- `plugins.entries.voice-call.config` を設定する
- `openclaw voicecall ...` または `voice_call` ツールを使う

<div id="where-it-runs-local-vs-remote">
  ## どこで動作するか（ローカル vs リモート）
</div>

Voice Call プラグインは **Gateway プロセス内** で動作します。

リモートの Gateway を使用している場合は、**Gateway を実行しているマシン** にプラグインをインストール／設定し、その後 Gateway を再起動してプラグインを読み込んでください。

<div id="install">
  ## インストール
</div>

<div id="option-a-install-from-npm-recommended">
  ### オプション A: npm からのインストール（推奨）
</div>

```bash
openclaw plugins install @openclaw/voice-call
```

その後、Gateway を再起動してください。


<div id="option-b-install-from-a-local-folder-dev-no-copying">
  ### オプション B: ローカルフォルダからインストール（開発用途・コピー不要）
</div>

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

その後、Gatewayを再起動してください。


<div id="config">
  ## 設定
</div>

`plugins.entries.voice-call.config` 配下に設定を記述します：

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // または "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "..."
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "..."
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook"
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify" // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream"
          }
        }
      }
    }
  }
}
```

Notes:

* Twilio/Telnyx は**外部から到達可能な** webhook URL が必須です。
* Plivo は**外部から到達可能な** webhook URL が必須です。
* `mock` はローカル開発用のプロバイダーです（ネットワーク呼び出しなし）。
* `skipSignatureVerification` はローカルテスト専用です。
* 無料版の ngrok を使う場合、`publicUrl` を ngrok の URL と完全に一致させてください。署名検証は常に有効です。
* `tunnel.allowNgrokFreeTierLoopbackBypass: true` は、`tunnel.provider="ngrok"` かつ `serve.bind` がループバック（ngrok ローカルエージェント）の場合に**のみ**、不正な署名の Twilio webhook を許可します。ローカル開発専用で使用してください。
* ngrok 無料版の URL は変更されたり、中間画面が挟まれたりすることがあります。`publicUrl` がずれると、Twilio の署名検証は失敗します。本番環境では、安定したドメインか Tailscale funnel の利用を推奨します。


<div id="tts-for-calls">
  ## 通話向けTTS
</div>

Voice Callは、通話中のストリーミング音声にコアの `messages.tts` 設定（OpenAI または ElevenLabs）を使用します。プラグイン側の設定で **同じ構造** の設定を指定して上書きでき、この設定は `messages.tts` とディープマージされます。

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2"
    }
  }
}
```

注意事項:

* **音声通話では Edge TTS は使用されません**（電話音声には PCM が必要であり、Edge の出力は信頼性が低いため）。
* Twilio メディアストリーミングが有効な場合は Core TTS が使用されます。それ以外の場合、通話はプロバイダーのネイティブ音声にフォールバックします。


<div id="more-examples">
  ### 他の例
</div>

コア TTS のみを使用する（オーバーライドなし）:

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" }
    }
  }
}
```

通話時のみ ElevenLabs を使うようオーバーライド（それ以外ではコアのデフォルトを維持）:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2"
            }
          }
        }
      }
    }
  }
}
```

通話で使用する OpenAI モデルだけを上書きする（ディープマージの例）:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin"
            }
          }
        }
      }
    }
  }
}
```


<div id="inbound-calls">
  ## 着信
</div>

着信ポリシーのデフォルトは `disabled` です。着信を有効にするには、次を設定します。

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?"
}
```

自動応答はエージェントシステムを使用します。次の項目で調整できます：

* `responseModel`
* `responseSystemPrompt`
* `responseTimeoutMs`


<div id="cli">
  ## CLI
</div>

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```


<div id="agent-tool">
  ## エージェントツール
</div>

ツール名：`voice_call`

アクション：

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

このリポジトリには、`skills/voice-call/SKILL.md` に対応するスキルドキュメントが含まれています。

<div id="gateway-rpc">
  ## Gateway RPC
</div>

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)