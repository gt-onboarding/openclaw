---
title: メモリ
summary: "OpenClaw のメモリの仕組み（ワークスペース内ファイル + 自動メモリフラッシュ）"
read_when:
  - メモリファイルのレイアウトとワークフローを知りたいとき
  - 自動プリコンパクションによるメモリフラッシュを調整したいとき
---

<div id="memory">
  # メモリ
</div>

OpenClaw のメモリは**エージェントのワークスペース内にあるプレーンな Markdown ファイル**です。ファイルが
唯一の信頼できる情報源であり、モデルはディスクに書き込まれた内容だけを「記憶」します。

メモリ検索ツールは、有効なメモリプラグイン（デフォルト:
`memory-core`）によって提供されます。メモリプラグインは `plugins.slots.memory = "none"` で無効化できます。

<div id="memory-files-markdown">
  ## メモリファイル（Markdown）
</div>

デフォルトのワークスペース構成では、2 つのメモリ層を使用します:

* `memory/YYYY-MM-DD.md`
  * 日次ログ（追記のみ）。
  * セッション開始時に、当日分 + 前日分を読み込む。
* `MEMORY.md` (任意)
  * 厳選された長期メモリ。
  * **メインのプライベートセッションでのみ読み込む**（グループコンテキストでは決して読み込まない）。

これらのファイルはワークスペース（`agents.defaults.workspace`、デフォルトは
`~/.openclaw/workspace`）配下に配置されます。全体的なレイアウトについては [Agent workspace](/ja/concepts/agent-workspace) を参照してください。

<div id="when-to-write-memory">
  ## メモリを書き込むタイミング
</div>

* 判断、好み、長期的な事実は `MEMORY.md` に書き込みます。
* 日々のメモや進行中のコンテキストは `memory/YYYY-MM-DD.md` に書き込みます。
* 誰かが「これを覚えて」と言ったら、それを書き留めてください（RAM に保持したままにしないでください）。
* この仕組みはまだ発展途上です。モデルにメモリへ保存するようリマインドすると効果的です。モデルは何をすべきか理解しています。
* 何かをしっかり残したい場合は、**ボットにそれをメモリに書き込むよう依頼**してください。

<div id="automatic-memory-flush-pre-compaction-ping">
  ## 自動メモリフラッシュ（コンパクション前 ping）
</div>

セッションが**自動コンパクションに近づく**と、OpenClaw はモデルに対して、
コンテキストが圧縮される**前に**永続メモリへ書き出すよう促す
**サイレントなエージェントターン**を発生させます。デフォルトのプロンプトでは、
モデルは返信してもよい（*may reply*）と明示されていますが、通常は
ユーザーにこのターンを見せないために `NO_REPLY` が適切な応答です。

これは `agents.defaults.compaction.memoryFlush` で制御されます。

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "永続的なメモがあればmemory/YYYY-MM-DD.mdに書き込んでください。保存するものがなければNO_REPLYと返信してください。"
        }
      }
    }
  }
}
```

詳細:

* **ソフトしきい値**: セッションのトークン数の推定値が
  `contextWindow - reserveTokensFloor - softThresholdTokens` を超えたときにフラッシュ処理がトリガーされます。
* デフォルトでは **サイレント**: プロンプトに `NO_REPLY` が含まれるため、何も送信されません。
* **2つのプロンプト**: ユーザープロンプトとシステムプロンプトの2つがリマインダーを付加します。
* **コンパクションサイクルごとに1回のフラッシュ**（`sessions.json` で追跡されます）。
* **ワークスペースは書き込み可能である必要があります**: セッションがサンドボックス化されていて
  `workspaceAccess: "ro"` または `"none"` の場合、フラッシュはスキップされます。

コンパクションのライフサイクル全体については、
[セッション管理 + コンパクション](/ja/reference/session-management-compaction)
を参照してください。

<div id="vector-memory-search">
  ## ベクターメモリ検索
</div>

OpenClaw は、`MEMORY.md` および `memory/*.md`（さらに、任意で追加したディレクトリやファイル）に対して小さなベクターインデックスを構築し、語句が異なる場合でもセマンティックなクエリで関連ノートを見つけられるようにします。

デフォルト:

* デフォルトで有効。
* メモリファイルの変更を監視（デバウンス付き）。
* 既定ではリモート埋め込みを使用します。`memorySearch.provider` が設定されていない場合、OpenClaw は自動的に次を選択します:
  1. `memorySearch.local.modelPath` が設定され、かつファイルが存在する場合は `local`
  2. OpenAI キーを取得できる場合は `openai`
  3. Gemini キーを取得できる場合は `gemini`
  4. それ以外の場合は、設定されるまでメモリ検索は無効のまま
* ローカルモードは node-llama-cpp を使用し、`pnpm approve-builds` が必要になる場合があります。
* 利用可能な場合は sqlite-vec を使用して、SQLite 内のベクター検索を高速化します。

リモート埋め込みを使用するには、埋め込みプロバイダー用の API キーが **必須** です。OpenClaw は、認証プロファイル、`models.providers.*.apiKey`、または環境変数からキーを解決します。Codex OAuth はチャット/補完のみを対象としており、メモリ検索用の埋め込み要件を **満たしません**。Gemini の場合は、`GEMINI_API_KEY` または
`models.providers.google.apiKey` を使用してください。OpenAI 互換のカスタムエンドポイントを使用する場合は、
`memorySearch.remote.apiKey`（および任意で `memorySearch.remote.headers`）を設定します。

<div id="additional-memory-paths">
  ### 追加のメモリパス
</div>

デフォルトのワークスペースレイアウト外にある Markdown ファイルをインデックス化したい場合は、
明示的なパスを追加します。

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

メモ:

* パスは絶対パスでも、ワークスペースからの相対パスでも指定できます。
* ディレクトリは `.md` ファイルを対象に再帰的にスキャンされます。
* インデックス化されるのは Markdown ファイルのみです。
* シンボリックリンク（ファイル・ディレクトリとも）は無視されます。

<div id="gemini-embeddings-native">
  ### Gemini embeddings（ネイティブ）
</div>

Gemini の埋め込み API を直接利用するには、プロバイダーを `gemini` に設定します。

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

注意:

* `remote.baseUrl` はオプションです (指定しない場合は Gemini API のベース URL がデフォルトになります)。
* `remote.headers` を使うと、必要に応じて追加のヘッダーを設定できます。
* デフォルトモデル: `gemini-embedding-001`。

**カスタムの OpenAI 互換エンドポイント** (OpenRouter、vLLM、あるいはプロキシ) を使いたい場合は、
OpenAI プロバイダーで `remote` 設定を利用できます:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

API キーを設定したくない場合は、`memorySearch.provider = "local"` を使うか、
`memorySearch.fallback = "none"` を設定してください。

フォールバック:

* `memorySearch.fallback` には `openai`、`gemini`、`local`、`none` を指定できます。
* フォールバック先プロバイダーは、プライマリの埋め込みプロバイダーが失敗した場合にのみ使用されます。

バッチインデックス作成（OpenAI + Gemini）:

* OpenAI および Gemini の埋め込みでは、デフォルトで有効になっています。無効化するには `agents.defaults.memorySearch.remote.batch.enabled = false` を設定します。
* デフォルト動作ではバッチ完了を待機します。必要に応じて `remote.batch.wait`、`remote.batch.pollIntervalMs`、`remote.batch.timeoutMinutes` を調整してください。
* 同時に送信するバッチジョブ数を制御するには `remote.batch.concurrency` を設定します（デフォルト: 2）。
* バッチモードは、`memorySearch.provider = "openai"` または `"gemini"` の場合に適用され、それぞれの API キーを使用します。
* Gemini のバッチジョブは非同期埋め込みバッチエンドポイントを使用し、Gemini Batch API が利用可能である必要があります。

OpenAI のバッチが高速かつ低コストな理由:

* 大規模なバックフィル処理では、OpenAI は通常もっとも高速な選択肢です。多数の埋め込みリクエストを 1 回のバッチジョブにまとめて送信し、OpenAI に非同期で処理させることができるためです。
* OpenAI は Batch API ワークロード向けに割引料金を提供しているため、大規模なインデックス作成は、同じリクエストを同期的に送る場合と比べて、通常より低コストになります。
* 詳細は OpenAI Batch API のドキュメントおよび料金ページを参照してください:
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

設定例:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

ツール:

* `memory_search` — ファイルと行範囲付きのスニペットを返します。
* `memory_get` — パスを指定してメモリファイルの内容を読み込みます。

ローカルモード:

* `agents.defaults.memorySearch.provider = "local"` を設定します。
* `agents.defaults.memorySearch.local.modelPath`（GGUF または `hf:` URI）を指定します。
* オプション: リモートへのフォールバックを避けるには `agents.defaults.memorySearch.fallback = "none"` を設定します。

<div id="how-the-memory-tools-work">
  ### メモリツールの動作
</div>

* `memory_search` は、`MEMORY.md` と `memory/**/*.md` から抽出した Markdown チャンク（目標約 400 トークン、80 トークンのオーバーラップ）をセマンティック検索します。返されるのは、スニペットテキスト（上限約 700 文字）、ファイルパス、行範囲、スコア、プロバイダー/モデル、ローカル埋め込みからリモート埋め込みへのフォールバックが発生したかどうかです。ファイル全体のデータは返されません。
* `memory_get` は、特定のメモリ用 Markdown ファイル（ワークスペース相対）を、必要に応じて開始行と取得する行数 N を指定して読み取ります。`MEMORY.md` / `memory/` の外側のパスは、`memorySearch.extraPaths` に明示的に列挙されている場合にのみ許可されます。
* どちらのツールも、エージェントに対して `memorySearch.enabled` が true と評価された場合にのみ有効になります。

<div id="what-gets-indexed-and-when">
  ### 何がいつインデックス化されるか
</div>

* ファイル種別: Markdown のみ（`MEMORY.md`、`memory/**/*.md`、および `memorySearch.extraPaths` 配下の任意の `.md` ファイル）。
* インデックスの保存先: エージェントごとの SQLite（`~/.openclaw/memory/<agentId>.sqlite`。`agents.defaults.memorySearch.store.path` で設定可能、`{agentId}` トークンをサポート）。
* 更新タイミング（鮮度）: `MEMORY.md`、`memory/`、`memorySearch.extraPaths` に対するウォッチャーがインデックスを「dirty」状態としてマーク（デバウンス 1.5 秒）。同期はセッション開始時・検索時・インターバル実行時にスケジュールされ、非同期で実行される。セッションのトランスクリプトは、バックグラウンド同期をトリガーするための差分しきい値を利用する。
* 再インデックスのトリガー: インデックスは **プロバイダー/モデル + エンドポイントのフィンガープリント + チャンク化パラメータ** を保持する。これらのいずれかが変更された場合、OpenClaw はストア全体を自動的にリセットして再インデックスを行う。

<div id="hybrid-search-bm25-vector">
  ### ハイブリッド検索（BM25 + ベクター）
</div>

有効にすると、OpenClaw は次の 2 つを組み合わせて検索します:

* **ベクター類似度**（意味的な一致。表現が異なっていても一致を検出）
* **BM25 キーワード関連度**（ID、環境変数、コードシンボルのような正確なトークン）

プラットフォームでフルテキスト検索が利用できない場合、OpenClaw はベクター検索のみを使用します。

<div id="why-hybrid">
  #### なぜハイブリッドにするのか？
</div>

ベクター検索は、「意味が同じもの」を見つけるのがとても得意です:

* “Mac Studio gateway host” vs “the machine running the gateway”
* “debounce file updates” vs “avoid indexing on every write”

一方で、厳密でシグナルの強いトークンには弱くなりがちです:

* ID（`a828e60`, `b3b9895a…`）
* コードシンボル（`memorySearch.query.hybrid`）
* エラーメッセージ（“sqlite-vec unavailable”）

BM25（フルテキスト検索）はその逆で、厳密なトークンに強い一方、言い換えには弱い傾向があります。
ハイブリッド検索は、その中間にある実践的な折衷案です。**両方の検索シグナルを使うことで**、
「自然言語」のクエリと「干し草の山から針を探すような」クエリの両方で良い結果を得られます。

<div id="how-we-merge-results-the-current-design">
  #### 結果のマージ方法（現行の設計）
</div>

実装スケッチ：

1. 両方から候補プールを取得する：

* **Vector**: コサイン類似度の上位 `maxResults * candidateMultiplier` 件。
* **BM25**: FTS5 の BM25 ランク（値が小さいほど良い）の上位 `maxResults * candidateMultiplier` 件。

2. BM25 ランクを 0..1 相当のスコアに変換する：

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. チャンク ID で候補を統合し、重み付きスコアを計算する：

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

補足：

* `vectorWeight` + `textWeight` は設定解決時に 1.0 に正規化されるため、重みは割合として扱われる。
* 埋め込みが利用できない場合（またはプロバイダーがゼロベクトルを返した場合）でも BM25 は実行し、キーワードマッチを返す。
* FTS5 を作成できない場合は、ベクターのみの検索を維持する（致命的エラーにはしない）。

これは「情報検索理論的に完全」というわけではないが、シンプルで高速であり、実際のノートに対して再現率・適合率の改善につながることが多い。
今後さらに高度化したい場合、一般的な次のステップは Reciprocal Rank Fusion (RRF) や、ミックス前のスコア正規化
（min/max や z-score）などである。

設定：

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4
        }
      }
    }
  }
}
```

<div id="embedding-cache">
  ### 埋め込みキャッシュ
</div>

OpenClaw は **チャンク埋め込み** を SQLite にキャッシュできるため、再インデックス処理や頻繁な更新（特にセッションのトランスクリプト）の際に、変更されていないテキストについては再度埋め込みを行う必要がなくなります。

設定:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

<div id="session-memory-search-experimental">
  ### セッションメモリ検索（実験的）
</div>

オプションで **セッションの会話ログ** をインデックス化し、`memory_search` を通じて参照できるようにできます。
これは実験的フラグによって制御されています。

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

注意事項:

* セッションのインデックス化は **オプトイン**（デフォルトでは無効）です。
* セッション更新はデバウンスされ、デルタしきい値を超えた時点で **非同期にインデックス化** されます（ベストエフォート）。
* `memory_search` がインデックス化待ちでブロックされることはありません。バックグラウンド同期が完了するまで、結果がわずかに古い場合があります。
* 結果には依然としてスニペットのみが含まれます。`memory_get` は引き続きメモリファイルに限定されます。
* セッションのインデックス化はエージェントごとに分離されています（そのエージェントのセッションログのみがインデックス化されます）。
* セッションログはディスク上に保存されます（`~/.openclaw/agents/<agentId>/sessions/*.jsonl`）。ファイルシステムへアクセスできる任意のプロセス／ユーザーがそれらを読み取れるため、ディスクアクセスを信頼境界として扱ってください。より厳密な分離が必要な場合は、エージェントを別々の OS ユーザーやホストで実行してください。

デルタしきい値（デフォルト値は以下のとおり）:

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // JSONL行
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### SQLite ベクトル検索の高速化 (sqlite-vec)
</div>

sqlite-vec 拡張機能が利用可能な場合、OpenClaw は埋め込みを
SQLite の仮想テーブル (`vec0`) に保存し、データベース内でベクトル距離検索クエリを実行します。これにより、すべての埋め込みを JavaScript に読み込むことなく、高速な検索を維持できます。

設定 (オプション):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Notes:

* `enabled` のデフォルトは true です。無効化すると、検索は保存済み埋め込みに対する
  プロセス内でのコサイン類似度検索にフォールバックします。
* sqlite-vec 拡張機能が存在しない、または読み込みに失敗した場合、OpenClaw は
  エラーをログに記録し、JS 実装へのフォールバック（ベクターテーブルなし）で処理を継続します。
* `extensionPath` は同梱の sqlite-vec のパスを上書きします（カスタムビルドや
  非標準のインストール場所で有用です）。

<div id="local-embedding-auto-download">
  ### ローカル埋め込みの自動ダウンロード
</div>

* デフォルトのローカル埋め込みモデル: `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf`（約 0.6 GB）。
* `memorySearch.provider = "local"` の場合、`node-llama-cpp` が `modelPath` を解決し、GGUF が存在しなければキャッシュ（または設定されていれば `local.modelCacheDir`）に**自動ダウンロード**してから読み込みます。ダウンロードはリトライ時に再開されます。
* ネイティブビルド要件: `pnpm approve-builds` を実行し、`node-llama-cpp` を選択してから、`pnpm rebuild node-llama-cpp` を実行します。
* フォールバック: ローカルセットアップに失敗し、かつ `memorySearch.fallback = "openai"` の場合、リモート埋め込み（上書きされていない限り `openai/text-embedding-3-small`）に自動的に切り替え、その理由を記録します。

<div id="custom-openai-compatible-endpoint-example">
  ### OpenAI 互換のカスタムエンドポイント例
</div>

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

注意:

* `remote.*` は `models.providers.openai.*` より優先されます。
* `remote.headers` は OpenAI のヘッダーとマージされます。キーが競合した場合は remote 側が優先されます。OpenAI のデフォルトを使う場合は `remote.headers` を省略してください。
