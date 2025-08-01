---
id: voltagent
name: VoltAgent
---

# VoltAgent

[VoltAgent](https://github.com/VoltAgent/voltagent) 是一个开源的 TypeScript 框架，用于构建具有模块化工具、LLM 编排和灵活多代理系统的 AI 代理。它内置了类似 n8n 的可观测性控制台，让您能够直观地检查代理行为、追踪操作并轻松调试。

{% Banner type="tip" %}
您可以在以下位置找到完整的示例代码：[VoltAgent with Chroma 示例](https://github.com/VoltAgent/voltagent/tree/main/examples/with-chroma)
{% /Banner %}

## 安装

创建一个集成 Chroma 的新 VoltAgent 项目：

{% TabbedUseCaseCodeBlock language="Terminal" %}

{% Tab label="npm" %}

```terminal
npm create voltagent-app@latest -- --example with-chroma
```

{% /Tab %}

{% Tab label="yarn" %}

```terminal
yarn create voltagent-app --example=with-chroma
```

{% /Tab %}

{% Tab label="pnpm" %}

```terminal
pnpm create voltagent-app --example=with-chroma
```

{% /Tab %}

{% /TabbedUseCaseCodeBlock %}

这将创建一个完整的 VoltAgent + Chroma 设置，包含示例数据和两种不同的代理配置。

安装依赖项：

{% TabbedUseCaseCodeBlock language="Terminal" %}

{% Tab label="npm" %}

```terminal
npm install
```

{% /Tab %}

{% Tab label="yarn" %}

```terminal
yarn install
```

{% /Tab %}

{% Tab label="pnpm" %}

```terminal
pnpm install
```

{% /Tab %}

{% /TabbedUseCaseCodeBlock %}

接下来，您需要启动一个 Chroma 服务器实例。

```terminal
npm run chroma run
```

服务器将在 `http://localhost:8000` 上可用。

**注意**：对于生产部署，您可能更喜欢使用完全托管的托管服务 [Chroma Cloud](https://www.trychroma.com/)。有关云配置，请参见下面的环境设置部分。

## 环境设置

使用以下配置创建 `.env` 文件：

### 选项 1：本地 Chroma 服务器

```env
# 用于嵌入和 LLM 的 OpenAI API 密钥
OPENAI_API_KEY=your-openai-api-key-here

# 本地 Chroma 服务器配置（可选 - 显示默认值）
CHROMA_HOST=localhost
CHROMA_PORT=8000
```

### 选项 2：[Chroma Cloud](https://www.trychroma.com/)

```env
# 用于嵌入和 LLM 的 OpenAI API 密钥
OPENAI_API_KEY=your-openai-api-key-here

# Chroma Cloud 配置
CHROMA_API_KEY=your-chroma-cloud-api-key
CHROMA_TENANT=your-tenant-name
CHROMA_DATABASE=your-database-name
```

代码会根据是否存在 `CHROMA_API_KEY` 自动检测使用哪种配置。

## 运行您的应用程序

启动您的 VoltAgent 应用程序：

```terminal
npm run dev
```

您将看到：

```
🚀 VoltAgent with Chroma is running!
📚 Sample knowledge base initialized with 5 documents
📚 Two different agents are ready:
  1️⃣ Assistant with Retriever - Automatic semantic search on every interaction
  2️⃣ Assistant with Tools - LLM decides when to search autonomously

💡 Chroma server started easily with npm run chroma run (no Docker/Python needed!)

══════════════════════════════════════════════════
  VOLTAGENT SERVER STARTED SUCCESSFULLY
══════════════════════════════════════════════════
  ✓ HTTP Server: http://localhost:3141

  VoltOps Platform:    https://console.voltagent.dev
══════════════════════════════════════════════════
```

{% Banner type="tip" %}
有关更多信息，请参阅官方 [VoltAgent 文档](https://voltagent.dev/docs/)。
{% /Banner %}

## 与您的代理交互

您的代理现在正在运行！要与它们交互：

1. **打开控制台**：点击终端输出中的 [`https://console.voltagent.dev`](https://console.voltagent.dev) 链接（或复制粘贴到浏览器中）。
2. **查找您的代理**：在 VoltOps LLM Observability Platform 页面上，您应该会看到列出的两个代理：
   - "Assistant with Retriever"
   - "Assistant with Tools"
3. **打开代理详情**：点击任一代理的名称。
4. **开始聊天**：在代理详情页面上，点击右下角的聊天图标以打开聊天窗口。
5. **测试 RAG 功能**：尝试以下问题：
   - "What is VoltAgent?"
   - "Tell me about vector databases"
   - "How does TypeScript help with development?"

![VoltAgent with Chroma Demo](https://cdn.voltagent.dev/docs/chroma-rag-example.gif)

您的 AI 代理将提供包含 Chroma 知识库中相关信息的答案，并附带引用，显示在生成响应期间引用了哪些源材料。

## 它是如何工作的

快速了解其内部机制以及如何自定义它。

### 创建 Chroma 检索器

创建 `src/retriever/index.ts`：

```typescript
import {
  BaseRetriever,
  type BaseMessage,
  type RetrieveOptions,
} from "@voltagent/core";
import {
  ChromaClient,
  CloudClient,
  type QueryRowResult,
  type Metadata,
} from "chromadb";
import { OpenAIEmbeddingFunction } from "@chroma-core/openai";

// 初始化 Chroma 客户端 - 支持本地和云
const chromaClient = process.env.CHROMA_API_KEY
  ? new CloudClient() // 使用 CHROMA_API_KEY, CHROMA_TENANT, CHROMA_DATABASE 环境变量
  : new ChromaClient({
      host: process.env.CHROMA_HOST || "localhost",
      port: parseInt(process.env.CHROMA_PORT || "8000"),
    });

// 配置 OpenAI 嵌入
const embeddingFunction = new OpenAIEmbeddingFunction({
  apiKey: process.env.OPENAI_API_KEY,
  modelName: "text-embedding-3-small", // 高效且成本效益高
});

const collectionName = "voltagent-knowledge-base";
```

**关键要素解析**:

- **ChromaClient/CloudClient**: 连接到您的本地 Chroma 服务器或 Chroma Cloud
- **自动检测**: 如果设置了 CHROMA_API_KEY，则使用 CloudClient，否则回退到本地 ChromaClient
- **OpenAIEmbeddingFunction**: 使用 OpenAI 的嵌入模型将文本转换为向量
- **集合（Collection）**: 包含您的文档及其嵌入的命名容器

### 初始化示例数据

添加一些示例文档以开始使用：

```typescript
async function initializeCollection() {
  try {
    const collection = await chromaClient.getOrCreateCollection({
      name: collectionName,
      embeddingFunction: embeddingFunction,
    });

    // 关于你的领域的示例文档
    const sampleDocuments = [
      "VoltAgent 是一个基于 TypeScript 的框架，用于构建具有模块化组件的 AI 代理。",
      "Chroma 是一个原生 AI 的开源向量数据库，能够自动处理嵌入。",
      "向量数据库存储高维向量并支持语义搜索功能。",
      "检索增强生成 (RAG) 结合了信息检索和语言生成。",
      "TypeScript 为 JavaScript 提供了静态类型，使代码更加可靠和易于维护。",
    ];

    const sampleIds = sampleDocuments.map((_, index) => `sample_${index + 1}`);

    // 使用 upsert 避免重复
    await collection.upsert({
      documents: sampleDocuments,
      ids: sampleIds,
      metadatas: sampleDocuments.map((_, index) => ({
        type: "sample",
        index: index + 1,
        topic:
          index < 2 ? "frameworks" : index < 4 ? "databases" : "programming",
      })),
    });

    console.log("📚 样本知识库已初始化");
  } catch (error) {
    console.error("初始化集合时出错:", error);
  }
}

// 模块加载时初始化
initializeCollection();
```

**此代码的功能**：

- 使用 OpenAI 的嵌入功能建立一个集合
- 添加带有元数据的示例文档
- 使用 `upsert` 避免文档重复
- 自动为每个文档生成嵌入

### 实现检索器类

创建主检索器类：

```typescript
async function retrieveDocuments(query: string, nResults = 3) {
  try {
    const collection = await chromaClient.getOrCreateCollection({
      name: collectionName,
      embeddingFunction: embeddingFunction,
    });

    const results = await collection.query({
      queryTexts: [query],
      nResults,
    });

    // 使用新的 .rows() 方法进行更清晰的数据访问
    const rows = results.rows();

    if (!rows || rows.length === 0 || !rows[0]) {
      return [];
    }

    // 格式化结果 - rows[0] 包含实际行数据
    return rows[0].map((row: QueryRowResult<Metadata>, index: number) => ({
      content: row.document || "",
      metadata: row.metadata || {},
      distance: results.distances?.[0]?.[index] || 0, // 距离仍来自原始结果
      id: row.id,
    }));
  } catch (error) {
    console.error("检索文档时出错:", error);
    return [];
  }
}

export class ChromaRetriever extends BaseRetriever {
  async retrieve(
    input: string | BaseMessage[],
    options: RetrieveOptions
  ): Promise<string> {
    // 将输入转换为可搜索字符串
    let searchText = "";

    if (typeof input === "string") {
      searchText = input;
    } else if (Array.isArray(input) && input.length > 0) {
      const lastMessage = input[input.length - 1];

      // 处理不同的内容格式
      if (Array.isArray(lastMessage.content)) {
        const textParts = lastMessage.content
          .filter((part: any) => part.type === "text")
          .map((part: any) => part.text);
        searchText = textParts.join(" ");
      } else {
        searchText = lastMessage.content as string;
      }
    }

    // 执行语义搜索
    const results = await retrieveDocuments(searchText, 3);

    // 添加引用到 userContext 以便跟踪
    if (options.userContext && results.length > 0) {
      const references = results.map((doc, index) => ({
        id: doc.id,
        title: doc.metadata.title || `文档 ${index + 1}`,
        source: "Chroma 知识库",
        distance: doc.distance,
      }));

      options.userContext.set("references", references);
    }

    // 为 LLM 格式化结果
    if (results.length === 0) {
      return "在知识库中未找到相关文档。";
    }

    return results
      .map(
        (doc, index) =>
          `文档 ${index + 1} (ID: ${doc.id}, 距离: ${doc.distance.toFixed(4)}):\n${doc.content}`
      )
      .join("\n\n---\n\n");
  }
}

export const retriever = new ChromaRetriever();
```

**主要功能**：

- **输入处理**：支持字符串和消息数组输入
- **语义搜索**：使用 Chroma 的向量相似性搜索
- **用户上下文**：跟踪引用以实现透明度
- **错误处理**：对搜索失败进行优雅回退

### 创建你的代理

现在在 `src/index.ts` 中使用不同的检索模式创建代理：

```typescript
import { openai } from "@ai-sdk/openai";
import { Agent, VoltAgent } from "@voltagent/core";
import { VercelAIProvider } from "@voltagent/vercel-ai";
import { retriever } from "./retriever/index.js";

// 代理 1：每次交互时自动检索
const agentWithRetriever = new Agent({
  name: "带有检索器的助手",
  description:
    "一个有帮助的助手，能够自动搜索知识库中的相关信息",
  llm: new VercelAIProvider(),
  model: openai("gpt-4o-mini"),
  retriever: retriever,
});

// 代理 2：LLM 决定何时搜索
const agentWithTools = new Agent({
  name: "带有工具的助手",
  description:
    "一个有帮助的助手，可以在需要时搜索知识库",
  llm: new VercelAIProvider(),
  model: openai("gpt-4o-mini"),
  tools: [retriever.tool],
});

new VoltAgent({
  agents: {
    agentWithRetriever,
    agentWithTools,
  },
});
```

## 使用模式

### 自动检索

第一个代理在每次响应之前自动进行搜索：

```
用户："VoltAgent 是什么？"
代理：根据知识库，VoltAgent 是一个基于 TypeScript 的框架，用于构建具有模块化组件的 AI 代理...

来源：
- 文档 1 (ID: sample_1, 距离: 0.1234)：Chroma 知识库
- 文档 2 (ID: sample_2, 距离: 0.2456)：Chroma 知识库
```

### 基于工具的检索

第二个代理仅在判断有必要时进行搜索：

```
用户："介绍一下 TypeScript"
代理：让我搜索有关 TypeScript 的相关信息。
[搜索知识库]
根据搜索结果，TypeScript 为 JavaScript 提供了静态类型，使代码更可靠且易于维护...

来源：
- 文档 5 (ID: sample_5, 距离: 0.0987)：Chroma 知识库
```

### 在代码中访问来源

你可以从响应中访问检索过程中使用的来源：

```typescript
// 生成响应后
const response = await agent.generateText("What is VoltAgent?");
console.log("答案：", response.text);

// 查看使用了哪些来源
const references = response.userContext?.get("references");
if (references) {
  console.log("使用的来源：", references);
  references.forEach((ref) => {
    console.log(`- ${ref.title} (ID: ${ref.id}, 距离: ${ref.distance})`);
  });
}
// 输出：[{ id: "sample_1", title: "文档 1", source: "Chroma 知识库", distance: 0.1234 }]
```

或者在使用 `streamText` 时：

```typescript
const result = await agent.streamText("给我讲讲向量数据库");

for await (const textPart of result.textStream) {
  process.stdout.write(textPart);
}

// 流式传输完成后访问来源
const references = result.userContext?.get("references");
if (references) {
  console.log("\n使用的来源：", references);
}
```

这种集成为你在 VoltAgent 应用中添加语义搜索能力提供了坚实的基础。VoltAgent 的灵活架构与 Chroma 强大的向量搜索相结合，构建了一个强大的 RAG 系统，能够应对现实中的知识检索需求。

{% Banner type="tip" %}
关于如何在 VoltAgent 中使用 Chroma 的更多信息，请参阅 [VoltAgent 文档](https://voltagent.dev/docs/rag/chroma/)。
{% /Banner %}