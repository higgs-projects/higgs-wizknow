# Higgs WizKnow - 智能问答智能体 (带RAG功能)

基于 [AgentScope Java](https://java.agentscope.io/) 构建的智能问答系统，支持检索增强生成（RAG）技术。

## 功能特点

- ✅ **RAG 检索增强生成**：基于本地知识库提供准确回答
- ✅ **ReAct 智能体**：采用推理-行动（Reasoning + Acting）模式
- ✅ **多种文档支持**：支持文本、PDF、Word等多种格式
- ✅ **两种RAG模式**：
  - Generic模式：自动检索，简单易用
  - Agentic模式：智能体自主决定何时检索
- ✅ **向量存储**：支持内存、Qdrant、Elasticsearch等
- ✅ **交互式问答**：提供命令行交互界面

## 环境要求

- JDK 17+
- Maven 3.6+
- DashScope API Key（阿里云通义千问）

## 快速开始

### 1. 安装依赖

项目已配置好 Maven 依赖，直接运行：

```bash
mvn clean install
```

### 2. 配置 API Key

设置环境变量：

```bash
# Linux/Mac
export DASHSCOPE_API_KEY=your_api_key_here

# Windows
set DASHSCOPE_API_KEY=your_api_key_here
```

### 3. 运行智能体

```bash
mvn exec:java -Dexec.mainClass="dev.higgs.wizknow.RAGQAAgentExample"
```

或者编译后运行：

```bash
mvn compile
java -cp target/classes:target/dependency/* dev.higgs.wizknow.RAGQAAgentExample
```

## 使用示例

### 基本问答

```java
// 创建智能体
RAGQAAgentExample app = new RAGQAAgentExample();
ReActAgent agent = app.createQAAgentWithRAG();

// 添加知识文档
app.addKnowledgeToBase(agent);

// 提问
String question = "Higgs WizKnow的主要产品是什么？";
app.askQuestion(agent, question);
```

### 添加自定义文档

修改 `addKnowledgeToBase` 方法，添加你自己的知识文档：

```java
String knowledgeContent = """
    # 你的知识库内容
    
    这里可以添加任何你需要的知识文档，
    支持Markdown格式的结构化内容。
    """;

TextReader reader = new TextReader(512, SplitStrategy.PARAGRAPH, 50);
List<Document> documents = reader.read(ReaderInput.fromString(knowledgeContent)).block();
knowledge.addDocuments(documents).block();
```

### 从文件加载文档

```java
// 加载文本文件
TextReader reader = new TextReader(512, SplitStrategy.PARAGRAPH, 50);
List<Document> documents = reader.read(ReaderInput.fromFile("path/to/document.txt")).block();

// 加载PDF文件
PDFReader pdfReader = new PDFReader(512, SplitStrategy.PARAGRAPH, 50);
List<Document> pdfDocs = pdfReader.read(ReaderInput.fromFile("path/to/document.pdf")).block();

// 加载Word文件
WordReader wordReader = new WordReader(512, SplitStrategy.PARAGRAPH, 50, true, true, TableFormat.MARKDOWN);
List<Document> wordDocs = wordReader.read(ReaderInput.fromFile("path/to/document.docx")).block();
```

## RAG 模式说明

### Generic 模式（推荐初学者）

```java
ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .knowledge(knowledge)
    .ragMode(RAGMode.GENERIC)  // 自动检索模式
    .enableOnlyForUserQueries(true)  // 仅对用户查询检索
    .retrieveConfig(RetrieveConfig.builder()
        .limit(5)              // 检索5个相关文档
        .scoreThreshold(0.3)   // 相似度阈值
        .build())
    .build();
```

**特点**：
- 每次用户查询前自动检索知识库
- 简单直接，适合大多数场景
- 不依赖智能体的推理能力

### Agentic 模式（高级用户）

```java
ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .sysPrompt("使用retrieve_knowledge工具检索信息")
    .model(model)
    .knowledge(knowledge)
    .ragMode(RAGMode.AGENTIC)  // 智能检索模式
    .build();
```

**特点**：
- 智能体拥有 `retrieve_knowledge` 工具
- 智能体自主决定何时检索
- 需要较强的模型推理能力
- 更灵活，避免不必要的检索

## 知识库配置

### 内存存储（开发测试）

```java
InMemoryStore store = InMemoryStore.builder()
    .dimensions(1024)
    .build();
```

### Qdrant 存储（生产环境）

添加依赖：

```xml
<dependency>
    <groupId>io.qdrant</groupId>
    <artifactId>client</artifactId>
</dependency>
```

配置：

```java
QdrantStore store = QdrantStore.builder()
    .location("localhost:6334")
    .collectionName("my_knowledge")
    .dimensions(1024)
    .build();
```

### Elasticsearch 存储

添加依赖：

```xml
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
</dependency>
```

配置：

```java
ElasticsearchStore store = ElasticsearchStore.builder()
    .endpoint("http://localhost:9200")
    .indexName("knowledge_base")
    .dimensions(1024)
    .build();
```

## 文档处理配置

### 文本分块策略

```java
// 按段落分块（推荐）
new TextReader(512, SplitStrategy.PARAGRAPH, 50);

// 按句子分块
new TextReader(256, SplitStrategy.SENTENCE, 20);

// 按字符分块
new TextReader(1024, SplitStrategy.CHARACTER, 100);

// 按Token分块
new TextReader(512, SplitStrategy.TOKEN, 50);
```

### PDF 处理

```java
PDFReader reader = new PDFReader(
    512,                          // chunk size
    SplitStrategy.PARAGRAPH,      // split strategy
    50                            // overlap
);
```

### Word 处理（支持表格）

```java
WordReader reader = new WordReader(
    512,                          // chunk size
    SplitStrategy.PARAGRAPH,      // split strategy
    50,                           // overlap
    true,                         // include images
    true,                         // include tables
    TableFormat.MARKDOWN          // table format
);
```

## 高级功能

### 长期记忆

```java
// 启用自动长期记忆
ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .knowledge(knowledge)
    .longTermMemoryMode(LongTermMemoryMode.AGENT_CONTROL)
    .build();
```

### 计划管理

```java
// 启用计划笔记本
ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .knowledge(knowledge)
    .enablePlan()  // 自动启用
    // 或自定义配置
    .planNotebook(PlanNotebook.builder()
        .maxSubtasks(15)
        .build())
    .build();
```

### 自定义工具

```java
class CustomTools {
    @Tool(name = "search_database", description = "搜索数据库")
    public String searchDatabase(
        @ToolParam(name = "query", description = "查询条件") String query
    ) {
        // 实现数据库搜索逻辑
        return "搜索结果...";
    }
}

// 注册工具
Toolkit toolkit = new Toolkit();
toolkit.registerTool(new CustomTools());

ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .knowledge(knowledge)
    .toolkit(toolkit)
    .build();
```

## 配置参数说明

### RetrieveConfig 参数

| 参数 | 说明 | 默认值 | 推荐值 |
|------|------|--------|--------|
| `limit` | 检索文档数量 | 5 | 3-5 |
| `scoreThreshold` | 相似度阈值 | 0.3 | 0.3-0.5 |
| `conversationHistory` | 对话历史 | null | 多轮对话时设置 |

### Reader 参数

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `chunkSize` | 文档块大小 | 256-1024 |
| `splitStrategy` | 分割策略 | PARAGRAPH |
| `overlap` | 重叠大小 | chunkSize的10-20% |

## 常见问题

### Q: 如何提高回答准确性？

1. 增加 `scoreThreshold` 值（如从0.3提高到0.5）
2. 优化文档分块策略
3. 使用更强大的模型（如 `qwen3-max`）
4. 添加更多相关知识文档

### Q: 如何处理中文文档？

AgentScope 完全支持中文文档。使用中文嵌入模型：

```java
DashScopeTextEmbedding embeddingModel = DashScopeTextEmbedding.builder()
    .apiKey(System.getenv("DASHSCOPE_API_KEY"))
    .modelName("text-embedding-v3")  // 支持中文
    .dimensions(1024)
    .build();
```

### Q: 知识库太大怎么办？

- 使用持久化向量存储（Qdrant、Elasticsearch）
- 调整 `chunkSize` 参数
- 使用 `scoreThreshold` 过滤低质量文档
- 考虑知识库分层策略

### Q: 如何实现多轮对话？

Generic RAG 模式会自动处理多轮对话。对于 Agentic 模式：

```java
RetrieveConfig config = RetrieveConfig.builder()
    .limit(5)
    .scoreThreshold(0.3)
    .conversationHistory(history)  // 传入对话历史
    .build();
```

## 相关资源

- [AgentScope Java 官方文档](https://java.agentscope.io/)
- [AgentScope Java GitHub](https://github.com/agentscope-ai/agentscope-java)
- [DashScope 文档](https://help.aliyun.com/zh/dashscope/)
- [Qdrant 文档](https://qdrant.tech/documentation/)

## License

MIT License
