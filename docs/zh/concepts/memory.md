---
title: 记忆
summary: "OpenClaw 记忆机制的工作方式（工作区文件 + 自动记忆刷新）"
read_when:
  - 当你想了解记忆文件的布局和工作流程
  - 当你想调整自动预压缩记忆刷新机制
---

<div id="memory">
  # Memory
</div>

OpenClaw 的记忆以 **智能体工作区中的纯 Markdown 文件** 形式存储。这些文件是
唯一可信的事实来源；模型只会“记得”已经写入磁盘的内容。

记忆搜索工具由当前启用的记忆插件提供（默认：`memory-core`）。
将 `plugins.slots.memory` 设置为 `"none"` 可禁用记忆插件。

<div id="memory-files-markdown">
  ## 记忆文件（Markdown）
</div>

默认的工作区布局采用两层记忆结构：

* `memory/YYYY-MM-DD.md`
  * 每日日志（仅追加写入）。
  * 在会话开始时读取今天和昨天的内容。
* `MEMORY.md`（可选）
  * 精选的长期记忆。
  * **只在主私有会话中加载**（绝不要在群组环境中加载）。

这些文件位于工作区目录下（`agents.defaults.workspace`，默认路径为
`~/.openclaw/workspace`）。完整布局见 [Agent 工作区](/zh/concepts/agent-workspace)。

<div id="when-to-write-memory">
  ## 何时写入记忆
</div>

* 决策、偏好和长期有效的事实写入 `MEMORY.md`。
* 日常笔记和运行时上下文写入 `memory/YYYY-MM-DD.md`。
* 如果有人说“记住这个”，就把它写下来（不要只放在内存里）。
* 这一部分仍在演进中。适当提醒模型去存储记忆会有帮助；它会知道该怎么做。
* 如果你想让某件事真正“记住”，**请让机器人把它写进记忆里**。

<div id="automatic-memory-flush-pre-compaction-ping">
  ## 自动内存刷新（预压缩 ping）
</div>

当一个会话**接近自动压缩**时，OpenClaw 会触发一次**静默的、由智能体自行发起的轮次**，提醒模型在上下文被压缩 **之前** 写入持久化内存。默认提示会明确说明模型*可以回复*，但通常 `NO_REPLY` 才是正确的响应，这样用户就永远不会看到这一轮对话。

这一行为由 `agents.defaults.compaction.memoryFlush` 控制：

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
          prompt: "将任何需要持久保存的笔记写入 memory/YYYY-MM-DD.md;如果没有需要存储的内容,请回复 NO_REPLY。"
        }
      }
    }
  }
}
```

详情：

* **软阈值**：当会话的 token 估计值超过
  `contextWindow - reserveTokensFloor - softThresholdTokens` 时触发一次 flush。
* 默认**静默**：提示词中包含 `NO_REPLY`，因此不会向用户发送任何内容。
* **两个提示词**：一个用户提示词，加上一个系统提示词用于附加提醒内容。
* **每个压缩周期只执行一次 flush**（记录在 `sessions.json` 中）。
* **工作区必须可写**：如果会话在沙箱中运行且
  `workspaceAccess: "ro"` 或 `"none"`，则会跳过 flush。

有关完整的压缩生命周期，请参见
[会话管理与压缩](/zh/reference/session-management-compaction)。

<div id="vector-memory-search">
  ## 向量内存搜索
</div>

OpenClaw 可以基于 `MEMORY.md` 和 `memory/*.md`（以及你选择额外加入的任何目录或文件）构建一个小型向量索引，这样即使措辞不同，语义查询也能找到相关笔记。

默认行为：

* 默认启用。
* 监听内存文件的变更（带防抖处理）。
* 默认使用远程嵌入。如果未设置 `memorySearch.provider`，OpenClaw 会自动选择：
  1. 如果配置了 `memorySearch.local.modelPath` 且文件存在，则使用 `local`。
  2. 如果能解析到 OpenAI 密钥，则使用 `openai`。
  3. 如果能解析到 Gemini 密钥，则使用 `gemini`。
  4. 否则内存搜索保持禁用状态，直到完成配置。
* 本地模式使用 node-llama-cpp，并且可能需要执行 `pnpm approve-builds`。
* 在可用时使用 sqlite-vec 加速 SQLite 内部的向量搜索。

远程嵌入 **需要** 一个嵌入提供方的 API 密钥。OpenClaw 会从认证配置档案、`models.providers.*.apiKey` 或环境变量中解析密钥。Codex OAuth 仅覆盖聊天/补全功能，并**不**满足内存搜索所需的嵌入。对于 Gemini，使用 `GEMINI_API_KEY` 或
`models.providers.google.apiKey`。在使用自定义 OpenAI 兼容端点时，设置 `memorySearch.remote.apiKey`（以及可选的 `memorySearch.remote.headers`）。

<div id="additional-memory-paths">
  ### 附加记忆路径
</div>

如果你想为默认工作区布局之外的 Markdown 文件建立索引，可以添加显式路径：

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Notes:

* 路径可以是绝对路径，也可以是相对于工作区的路径。
* 会递归扫描目录以查找 `.md` 文件。
* 只会索引 Markdown 文件。
* 会忽略符号链接（文件或目录）。

<div id="gemini-embeddings-native">
  ### Gemini 嵌入（原生）
</div>

将提供方设置为 `gemini`，即可直接使用 Gemini embeddings API：

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

备注：

* `remote.baseUrl` 是可选的（默认为 Gemini API 的基础 URL）。
* `remote.headers` 允许你在需要时添加额外的请求头。
* 默认模型：`gemini-embedding-001`。

如果你想使用**自定义的 OpenAI 兼容端点**（如 OpenRouter、vLLM 或代理），
可以将 `remote` 配置与 OpenAI 提供方一起使用：

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

如果你不想设置 API 密钥，可以使用 `memorySearch.provider = "local"`，或者设置
`memorySearch.fallback = "none"`。

回退逻辑：

* `memorySearch.fallback` 可以是 `openai`、`gemini`、`local` 或 `none`。
* 回退提供方仅在主嵌入提供方失败时才会使用。

批量索引（OpenAI + Gemini）：

* 对 OpenAI 和 Gemini 嵌入默认启用。设置 `agents.defaults.memorySearch.remote.batch.enabled = false` 可禁用。
* 默认行为会等待批处理完成；如有需要，可调节 `remote.batch.wait`、`remote.batch.pollIntervalMs` 和 `remote.batch.timeoutMinutes`。
* 设置 `remote.batch.concurrency` 来控制并行提交的批处理任务数量（默认：2）。
* 当 `memorySearch.provider = "openai"` 或 `"gemini"` 时会启用批量模式，并使用对应的 API 密钥。
* Gemini 批处理任务使用异步 embeddings 批量接口，需要 Gemini Batch API 可用。

为什么 OpenAI 批处理又快又便宜：

* 对于大规模回填，OpenAI 通常是我们支持的最快选项，因为我们可以在单个批处理任务中提交大量嵌入请求，让 OpenAI 异步处理。
* OpenAI 为 Batch API 工作负载提供优惠价格，因此大规模索引运行通常比同步发送相同请求更便宜。
* 详情参见 OpenAI Batch API 文档和定价：
  * https://platform.openai.com/docs/api-reference/batch
  * https://platform.openai.com/pricing

配置示例：

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

工具：

* `memory_search` — 返回包含文件路径和行号范围的片段。
* `memory_get` — 按路径读取记忆文件内容。

本地模式：

* 设置 `agents.defaults.memorySearch.provider = "local"`。
* 提供 `agents.defaults.memorySearch.local.modelPath`（GGUF 或 `hf:` URI）。
* 可选：将 `agents.defaults.memorySearch.fallback` 设置为 `"none"`，以避免回退到远程提供方。

<div id="how-the-memory-tools-work">
  ### 记忆工具的工作方式
</div>

* `memory_search` 会对来自 `MEMORY.md` 和 `memory/**/*.md` 的 Markdown 片段进行语义搜索（目标片段大小约 400 个 token，重叠 80 个 token）。它会返回摘要文本（上限约 700 个字符）、文件路径、行号范围、评分、提供方/模型，以及是否从本地回退到远程向量嵌入。不会返回完整文件内容。
* `memory_get` 会读取指定的记忆 Markdown 文件（相对于工作区路径），可选地从某一行开始读取 N 行。只有在 `memorySearch.extraPaths` 中被显式列出的情况下，才允许访问 `MEMORY.md` / `memory/` 之外的路径。
* 仅当 `memorySearch.enabled` 对该智能体解析结果为 true 时，这两个工具才会被启用。

<div id="what-gets-indexed-and-when">
  ### 会被索引的内容（以及何时索引）
</div>

* 文件类型：仅 Markdown（`MEMORY.md`、`memory/**/*.md`，以及 `memorySearch.extraPaths` 下的任意 `.md` 文件）。
* 索引存储位置：按智能体分别使用 SQLite 存储，路径为 `~/.openclaw/memory/<agentId>.sqlite`（可通过 `agents.defaults.memorySearch.store.path` 配置，支持 `{agentId}` 占位符）。
* 新鲜度：对 `MEMORY.md`、`memory/` 和 `memorySearch.extraPaths` 使用文件监视器，标记索引为“脏”状态（防抖 1.5 秒）。在会话启动、执行搜索或按固定间隔时调度同步，并以异步方式运行。会话记录使用增量阈值来触发后台同步。
* 重新索引触发条件：索引会存储嵌入向量的 **provider/model + endpoint 指纹 + 分块参数**。如果上述任一项发生变化，OpenClaw 会自动重置并重新索引整个存储。

<div id="hybrid-search-bm25-vector">
  ### 混合搜索（BM25 + 向量）
</div>

启用后，OpenClaw 会同时使用：

* **向量相似度**（语义匹配，措辞可以不同）
* **BM25 关键词相关性**（对 ID、环境变量、代码符号等 token 的精确匹配）

如果你的平台不支持全文搜索，OpenClaw 会退化为仅使用向量搜索。

<div id="why-hybrid">
  #### 为什么要用混合检索？
</div>

向量检索非常擅长处理“这个意思是一样的”这种情况：

* “Mac Studio gateway host” vs “the machine running the gateway”
* “debounce file updates” vs “avoid indexing on every write”

但它在精确、高信号的 token 上可能会比较弱：

* ID（`a828e60`、`b3b9895a…`）
* 代码符号（`memorySearch.query.hybrid`）
* 错误字符串（“sqlite-vec unavailable”）

BM25（全文检索）则相反：在精确 token 上很强，但对意译的效果较弱。
混合检索是务实的折中方案：**同时利用这两种检索信号**，这样你既能在“自然语言”查询中获得好结果，也能在“大海捞针”式查询中精准命中目标。

<div id="how-we-merge-results-the-current-design">
  #### 我们如何合并结果（当前设计）
</div>

实现示意：

1. 从两种检索方式各自获取候选集合：

* **Vector**：按余弦相似度取前 `maxResults * candidateMultiplier` 条。
* **BM25**：按 FTS5 BM25 排名（数值越低越好）取前 `maxResults * candidateMultiplier` 条。

2. 将 BM25 排名转换为 0..1 左右的分数：

* `textScore = 1 / (1 + max(0, bm25Rank))`

3. 按 chunk id 做并集，并计算加权分数：

* `finalScore = vectorWeight * vectorScore + textWeight * textScore`

说明：

* `vectorWeight` + `textWeight` 会在配置解析中归一化到 1.0，因此权重可以视作百分比。
* 如果嵌入不可用（或提供方返回零向量），我们仍然会运行 BM25，并返回关键词匹配结果。
* 如果无法创建 FTS5，我们就只保留向量搜索（不会出现硬失败）。

这并不是“信息检索理论上的完美方案”，但它简单、快速，而且在真实笔记数据上通常能提升召回率和准确率。
如果之后想做得更复杂，常见的下一步是使用 Reciprocal Rank Fusion (RRF)，或在混合前先做分数归一化
（min/max 或 z-score）。

配置：

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
  ### 嵌入缓存
</div>

OpenClaw 可以在 SQLite 中缓存**文本分块嵌入向量（chunk embeddings）**，这样在重新索引和频繁更新时（尤其是会话对话记录）就不会对未修改的文本重新计算嵌入。

配置：

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
  ### 会话内存搜索（实验性）
</div>

你可以选择为**会话记录**建立索引，并通过 `memory_search` 将其检索出来。
此功能目前受一个实验性开关控制。

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

注意：

* 会话索引是**需主动开启**的（默认关闭）。
* 会话更新会被防抖处理，并在超过变更阈值后**异步建立索引**（尽力而为）。
* `memory_search` 永远不会因索引而阻塞；在后台同步完成前，结果可能会略有滞后。
* 结果仍然只包含片段；`memory_get` 依然仅限于内存文件。
* 会话索引按智能体进行隔离（只为该智能体的会话日志建立索引）。
* 会话日志存储在磁盘上（`~/.openclaw/agents/<agentId>/sessions/*.jsonl`）。任何具有文件系统访问权限的进程/用户都可以 `read` 它们，因此应将磁盘访问视为信任边界。若需更严格的隔离，请在不同的操作系统用户或主机下运行智能体。

变更阈值（默认值如下）：

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // JSONL 行
        }
      }
    }
  }
}
```

<div id="sqlite-vector-acceleration-sqlite-vec">
  ### SQLite 向量加速（sqlite-vec）
</div>

当 sqlite-vec 扩展可用时，OpenClaw 会将嵌入向量（embeddings）存储在一个
SQLite 虚拟表（`vec0`）中，并在数据库中执行向量距离查询。
这样可以在无需将所有嵌入向量加载到 JavaScript 中的情况下保持搜索速度。

配置（可选）：

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

注意事项：

* `enabled` 默认为 true；当禁用时，搜索会回退为在进程内对已存储的 embedding 向量执行余弦相似度计算。
* 如果缺少 sqlite-vec 扩展或加载失败，OpenClaw 会记录错误并继续使用 JS 回退实现（不使用向量表）。
* `extensionPath` 会覆盖内置的 sqlite-vec 路径（对自定义构建或非标准安装位置很有用）。

<div id="local-embedding-auto-download">
  ### 本地嵌入模型自动下载
</div>

* 默认本地嵌入模型：`hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf`（约 0.6 GB）。
* 当 `memorySearch.provider = "local"` 时，`node-llama-cpp` 会解析 `modelPath`；如果该 GGUF 文件不存在，会**自动下载**到缓存目录（或已设置的 `local.modelCacheDir`），然后加载。下载在重试时会自动续传。
* 原生构建要求：运行 `pnpm approve-builds`，选择 `node-llama-cpp`，然后执行 `pnpm rebuild node-llama-cpp`。
* 回退机制：如果本地设置失败且 `memorySearch.fallback = "openai"`，系统会自动切换到远程嵌入模型（默认为 `openai/text-embedding-3-small`，除非被覆盖），并记录原因。

<div id="custom-openai-compatible-endpoint-example">
  ### 自定义 OpenAI 兼容端点示例
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

备注：

* `remote.*` 的优先级高于 `models.providers.openai.*`。
* `remote.headers` 会与 OpenAI 的 headers 合并；发生键冲突时以 remote 为准。省略 `remote.headers` 则使用 OpenAI 的默认设置。
