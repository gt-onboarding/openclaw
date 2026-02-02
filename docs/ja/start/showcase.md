---
title: "ショーケース"
description: "コミュニティ発の実運用 OpenClaw プロジェクト"
summary: "コミュニティ主導で構築された OpenClaw プロジェクトおよび各種連携事例"
---

<div id="showcase">
  # ショーケース
</div>

コミュニティの実際のプロジェクト事例です。OpenClaw でどんなものが作られているか確認してみましょう。

<Info>
**掲載を希望する場合は、** あなたのプロジェクトを [Discord の #showcase チャンネル](https://discord.gg/clawd) で共有するか、[X で @openclaw をメンション](https://x.com/openclaw)してください。
</Info>

<div id="openclaw-in-action">
  ## 🎥 OpenClaw のデモ
</div>

VelvetShark によるフルセットアップ解説動画（28分）。

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw: Siri が本来あるべきだった自己ホスト型 AI（フルセットアップ）"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube で視聴](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="OpenClaw ショーケース動画"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube で視聴](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="OpenClaw コミュニティショーケース"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube で視聴](https://www.youtube.com/watch?v=5kkIJNUGFho)

<div id="fresh-from-discord">
  ## 🆕 Discord発の最新情報
</div>

<CardGroup cols={2}>
  <Card title="PR レビュー → Telegram フィードバック" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
    **@bangnokia** • `review` `github` `telegram`

    OpenCode で変更を完了 → PR を作成 → OpenClaw が差分をレビューし、Telegram で「軽微な提案」と、まず適用すべき重大な修正を含む明確なマージ可否の判断を返してくれます。

    <img src="/assets/showcase/pr-review-telegram.jpg" alt="Telegram で配信される OpenClaw の PR レビューフィードバック" />
  </Card>

  <Card title="数分で使えるワインセラー管理スキル" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
    **@prades&#95;maxime** • `skills` `local` `csv`

    「Robby」(@openclaw) にローカルのワインセラー用スキルを依頼。サンプルの CSV エクスポートと保存場所を確認したうえで、素早くスキルを構築・テストする（例ではボトル 962 本分）。

    <img src="/assets/showcase/wine-cellar-skill.jpg" alt="CSV からローカルのワインセラースキルを構築する OpenClaw" />
  </Card>

  <Card title="Tesco買い物オートパイロット" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
    **@marchattonhere** • `automation` `browser` `shopping`

    1週間分のミールプラン → 定番品 → 配送スロットを予約 → 注文を確定。API は使わず、ブラウザ制御だけで実現。

    <img src="/assets/showcase/tesco-shop.jpg" alt="チャット経由での Tesco ショッピング自動化" />
  </Card>

  <Card title="SNAG スクリーンショットをMarkdownに変換" icon="scissors" href="https://github.com/am-will/snag">
    **@am-will** • `devtools` `screenshots` `markdown`

    画面の一部をホットキーで選択 → Gemini vision → 即座にMarkdownがクリップボードにコピーされます。

    <img src="/assets/showcase/snag.png" alt="SNAG スクリーンショットをMarkdownに変換するツール" />
  </Card>

  <Card title="エージェント UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
    **@kitze** • `ui` `skills` `sync`

    エージェント群、Claude、Codex、OpenClaw 間のスキルやコマンドを一元管理するデスクトップアプリケーション。

    <img src="/assets/showcase/agents-ui.jpg" alt="エージェント用 UI アプリ" />
  </Card>

  <Card title="Telegram 音声メッセージ (papla.media)" icon="microphone" href="https://papla.media/docs">
    **コミュニティ** • `voice` `tts` `telegram`

    papla.media の TTS をラップし、生成された音声を Telegram のボイスメッセージとして送信します（煩わしい自動再生はありません）。

    <img src="/assets/showcase/papla-tts.jpg" alt="TTS から出力された Telegram のボイスメッセージ" />
  </Card>

  <Card title="Codex Monitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
    **@odrobnik** • `devtools` `codex` `brew`

    Homebrew でインストールできる、ローカルの OpenAI Codex セッションを一覧表示・確認・監視するためのヘルパー (CLI + VS Code)。

    <img src="/assets/showcase/codexmonitor.png" alt="ClawHub 上の CodexMonitor" />
  </Card>

  <Card title="Bambu製3Dプリンター制御" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
    **@tobiasbischoff** • `hardware` `3d-printing` `skill`

    BambuLab プリンタの制御やトラブルシューティングを行います。ステータス、ジョブ、カメラ、AMS、キャリブレーションなどに対応。

    <img src="/assets/showcase/bambu-cli.png" alt="ClawHub 上の Bambu CLI スキル" />
  </Card>

  <Card title="ウィーン交通局（Wiener Linien）" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
    **@hjanuschka** • `travel` `transport` `skill`

    ウィーンの公共交通機関のリアルタイム出発情報、運行状況の乱れ、エレベーターの稼働状況、およびルート案内。

    <img src="/assets/showcase/wienerlinien.png" alt="ClawHub 上の Wiener Linien スキル" />
  </Card>

  <Card title="ParentPay の学校給食" icon="utensils" href="#">
    **@George5562** • `automation` `browser` `parenting`

    ParentPay を使ったイギリスの学校給食予約を自動化します。テーブルセルを確実にクリックするために、マウス座標を利用します。
  </Card>

  <Card title="R2 アップロード（自分のファイルを送信する）" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
    **@julianengel** • `files` `r2` `presigned-urls`

    Cloudflare R2/S3 にアップロードし、安全な事前署名付きダウンロードリンクを生成します。リモート環境の OpenClaw インスタンスに最適です。
  </Card>

  <Card title="Telegram 経由のiOSアプリ" icon="mobile" href="#">
    **@coard** • `ios` `xcode` `testflight`

    マップ機能と音声録音機能を備えたフル機能の iOS アプリを作成し、Telegram のチャットだけで TestFlight へデプロイしました。

    <img src="/assets/showcase/ios-testflight.jpg" alt="iOS app on TestFlight" />
  </Card>

  <Card title="Oura Ring ヘルスアシスタント" icon="heart-pulse" href="#">
    **@AS** • `health` `oura` `calendar`

    Oura Ring のデータをカレンダーや予定、ジムのスケジュールと連携するパーソナルAIヘルスアシスタント。

    <img src="/assets/showcase/oura-health.png" alt="Oura ring health assistant" />
  </Card>

  <Card title="Kev のドリームチーム（14以上のエージェント）" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
    **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

    単一の Gateway 上で 14 以上のエージェントを動かし、Opus 4.5 オーケストレーターが Codex ワーカーへ処理を委譲する構成。Dream Team ロスター、モデル選択、サンドボックス化、Webhook、ハートビート、委譲フローを網羅した[包括的な技術解説](https://github.com/adam91holt/orchestrated-ai-articles)。エージェントのサンドボックス用の [Clawdspace](https://github.com/adam91holt/clawdspace)。[ブログ記事](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/)。
  </Card>

  <Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
    **@NessZerra** • `devtools` `linear` `cli` `issues`

    エージェント指向ワークフロー（Claude Code、OpenClaw）と統合された Linear 向け CLI。ターミナルから Issue、プロジェクト、ワークフローを管理できます。初の外部からの PR がマージされました！
  </Card>

  <Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
    **@jules** • `messaging` `beeper` `cli` `automation`

    Beeper Desktop 経由でメッセージを閲覧・送信・アーカイブします。Beeper のローカル MCP api を使用することで、エージェントがすべてのチャット（iMessage、WhatsApp など）を一元的に管理できます。
  </Card>
</CardGroup>

<div id="automation-workflows">
  ## 🤖 自動化とワークフロー
</div>

<CardGroup cols={2}>

<Card title="Winix 空気清浄機の制御" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

  Claude Code が空気清浄機の制御方法を特定・検証し、その後は OpenClaw が引き継いで部屋の空気の質を管理します。

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="OpenClaw 経由で Winix 空気清浄機を制御している様子" />
</Card>

<Card title="きれいな空のカメラショット" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

  屋上カメラをトリガーに、「空がきれいなときに OpenClaw に空の写真を撮らせる」ように設定 — OpenClaw が skill を設計し、そのまま撮影まで行います。

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="OpenClaw が撮影した屋上カメラによる空のスナップショット" />
</Card>

<Card title="ビジュアルな朝のブリーフィングシーン" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

  毎朝スケジュールされたプロンプトから、天気・タスク・日付・お気に入りの投稿/引用をまとめた 1 枚の「シーン」画像を、OpenClaw のペルソナ経由で生成します。
</Card>

<Card title="パデルコート予約" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Playtomic の空き状況チェッカー + 予約 CLI。空いているコートを二度と見逃しません。
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cli のスクリーンショット" />
</Card>

<Card title="会計データの取り込み" icon="file-invoice-dollar">
  **Community** • `automation` `email` `pdf`
  
  メールから PDF を収集し、税理士向けの資料を準備します。毎月の会計処理を自動操縦で回せます。
</Card>

<Card title="Couch Potato Dev モード" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

  Netflix を見ながら、Telegram 経由だけで個人サイト全体を再構築 — Notion → Astro、18 本の投稿を移行し、DNS を Cloudflare に切り替え。ノート PC を一度も開いていません。
</Card>

<Card title="求人検索エージェント" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

  求人情報を検索し、職務経歴書 (CV) のキーワードと照合して、関連する募集をリンク付きで返します。JSearch API を使って 30 分で構築されました。
</Card>

<Card title="Jira Skill ビルダー" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

  OpenClaw を Jira に接続し、ClawHub にまだ存在していない skill を、その場で自動生成させました。
</Card>

<Card title="Telegram 経由の Todoist Skill" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

  Todoist のタスクを自動化し、OpenClaw に Telegram チャット内から直接 skill を生成させました。
</Card>

<Card title="TradingView 分析" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

  ブラウザ自動化で TradingView にログインし、チャートをスクリーンショットして、要求に応じてテクニカル分析を実行します。API は不要 — ブラウザ制御だけで完結します。
</Card>

<Card title="Slack 自動サポート" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

  会社の Slack チャンネルを監視して適切に返信し、通知を Telegram に転送します。頼まれてもいないのに、デプロイ済みアプリの本番バグを自律的に修正しました。
</Card>

</CardGroup>

<div id="knowledge-memory">
  ## 🧠 ナレッジとメモリ
</div>

<CardGroup cols={2}>

<Card title="xuezh Chinese Learning" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  OpenClaw 経由で発音フィードバックと学習フローを提供する中国語学習エンジン。
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh の発音フィードバック" />
</Card>

<Card title="WhatsApp Memory Vault" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  WhatsApp のエクスポート全体を取り込み、1,000 件超のボイスメモを書き起こし、git ログと突き合わせ、相互リンクされた Markdown レポートとして出力する仕組み。
</Card>

<Card title="Karakeep Semantic Search" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Qdrant と OpenAI/Ollama 埋め込みを用いて、Karakeep のブックマークにベクトル検索機能を追加します。
</Card>

<Card title="Inside-Out-2 Memory" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  セッションファイルをメモリ → 信念 → 進化するセルフモデルへと変換する、独立したメモリマネージャ。
</Card>

</CardGroup>

<div id="voice-phone">
  ## 🎙️ 音声 & 電話
</div>

<CardGroup cols={2}>

<Card title="Clawdia Phone Bridge" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Vapi 音声アシスタント ↔ OpenClaw HTTP ブリッジ。あなたのエージェントとほぼリアルタイムで通話できます。
</Card>

<Card title="OpenRouter 文字起こし" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

  OpenRouter（Gemini など）経由の多言語音声文字起こし。ClawHub で利用可能です。
</Card>

</CardGroup>

<div id="infrastructure-deployment">
  ## 🏗️ インフラストラクチャとデプロイ
</div>

<CardGroup cols={2}>

<Card title="Home Assistant アドオン" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  SSH トンネル対応と永続状態を備えた OpenClaw Gateway を Home Assistant OS 上で動作させます。
</Card>

<Card title="Home Assistant スキル" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  自然言語で Home Assistant デバイスを制御および自動化します。
</Card>

<Card title="Nix パッケージング" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  再現可能なデプロイメントのための、必要なものが一通り揃った Nix 化 OpenClaw 設定です。
</Card>

<Card title="CalDAV カレンダー" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  khal/vdirsyncer を用いたカレンタースキル。セルフホスト型のカレンダー連携です。
</Card>

</CardGroup>

<div id="home-hardware">
  ## 🏠 ホームとハードウェア
</div>

<CardGroup cols={2}>

<Card title="GoHome 自動化" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  OpenClaw をインターフェイスとして利用する、Nix ネイティブのホームオートメーションと、美しい Grafana ダッシュボード。
  
  <img src="/assets/showcase/gohome-grafana.png" alt="GoHome の Grafana ダッシュボード" />
</Card>

<Card title="Roborock 掃除機" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  自然な会話で Roborock ロボット掃除機を操作できます。
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Roborock のステータス" />
</Card>

</CardGroup>

<div id="community-projects">
  ## 🌟 コミュニティプロジェクト
</div>

<CardGroup cols={2}>

<Card title="StarSwap Marketplace" icon="star" href="https://star-swap.com/">
  **コミュニティ** • `marketplace` `astronomy` `webapp`
  
  本格的な天文機器向けマーケットプレイス。OpenClaw エコシステム上に構築されています。
</Card>

</CardGroup>

---

<div id="submit-your-project">
  ## プロジェクトを投稿する
</div>

共有したいものがありますか？ぜひここで紹介させてください。

<Steps>
  <Step title="共有する">
    [Discord の #showcase チャンネル](https://discord.gg/clawd) に投稿するか、[@openclaw にポスト](https://x.com/openclaw)してください
  </Step>
  <Step title="詳細を記載する">
    何をするプロジェクトなのか、リポジトリ／デモへのリンク、あればスクリーンショットを添えて教えてください
  </Step>
  <Step title="掲載される">
    注目度の高いプロジェクトはこのページで紹介します
  </Step>
</Steps>