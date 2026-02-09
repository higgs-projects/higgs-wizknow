# Higgs WizKnow - AI 智能问答系统

## 项目概述

Higgs WizKnow 是基于 [AgentScope Java](https://java.agentscope.io/) 构建的智能问答系统，采用检索增强生成（RAG）技术，为企业提供精准的AI知识管理解决方案。

### 核心特性
- **RAG 检索增强生成**：基于本地/云端知识库提供准确回答
- **ReAct 智能体**：采用推理-行动（Reasoning + Acting）模式
- **多格式文档支持**：文本、PDF、Word等
- **两种RAG模式**：Generic（自动）和 Agentic（智能）
- **向量存储**：内存、Qdrant、Elasticsearch、PostgreSQL

### 技术栈

#### 核心框架
- **AgentScope Java 1.0.8** - AI智能体框架
- **Reactor Core** - 响应式编程

#### AI/ML
- **DashScope** - 阿里云通义千问大模型
  - 聊天模型：qwen3-max, qwen3-plus
  - 嵌入模型：text-embedding-v3
- **可选模型**：OpenAI, Google Gemini, Anthropic Claude

#### 向量存储
- **InMemoryStore** - 内存存储（开发/测试）
- **Qdrant** - 生产级向量数据库
- **Elasticsearch** - 企业级搜索
- **PgVector** - PostgreSQL扩展

#### 构建工具
- **Maven 3.6+** - 项目管理和构建
- **JDK 17+** - 运行环境

## 项目结构

```
higgs-wizknow/
├── src/
│   └── main/
│       └── java/
│           └── dev/
│               └── higgs/
│                   └── wizknow/
│                       ├── RAGQAAgentExample.java      # 基础RAG智能体示例
│                       ├── AdvancedRAGExample.java      # 高级功能示例
│                       └── DocumentLoader.java            # 文档加载工具类
├── documents/                                    # 知识文档目录（git忽略）
│   ├── 技术架构.txt
│   └── 产品说明.txt
├── pom.xml                                       # Maven配置
├── AGENTS.md                                     # 本文件
├── RAG_README.md                                 # RAG使用文档
├── QUICKSTART.md                                 # 快速入门指南
├── run.sh                                        # Linux/Mac启动脚本
└── run.bat                                       # Windows启动脚本
```

## 核心组件

### 1. 智能体（Agent）

#### ReActAgent
项目使用 `ReActAgent` 作为核心智能体，配置如下：

```java
ReActAgent.builder()
    .name("智能助手")
    .sysPrompt("系统提示词")
    .model(chatModel)
    .knowledge(knowledge)
    .ragMode(RAGMode.GENERIC)  // 或 RAGMode.AGENTIC
    .retrieveConfig(RetrieveConfig.builder()
        .limit(5)
        .scoreThreshold(0.3)
        .build())
    .maxIters(10)
    .build()
```

#### RAG模式

**Generic模式（推荐）**
- 自动为每个用户查询检索知识
- 适合大多数问答场景
- 配置简单，易于使用

**Agentic模式（高级）**
- 智能体拥有 `retrieve_knowledge` 工具
- 智能体自主决定何时检索
- 需要更强的模型推理能力

### 2. 知识库（Knowledge）

#### SimpleKnowledge
本地知识库实现，支持文档管理、向量化、检索。

```java
SimpleKnowledge.builder()
    .embeddingModel(embeddingModel)
    .embeddingStore(store)
    .build()
```

#### BailianKnowledge
阿里云百炼知识库，支持重排序、查询改写、多轮对话。

#### DifyKnowledge / RAGFlowKnowledge
第三方知识库集成。

### 3. 文档读取器（Reader）

#### TextReader
文本文件读取器，支持按段落、句子、字符、Token分割。

```java
TextReader reader = new TextReader(
    512,                      // chunk size
    SplitStrategy.PARAGRAPH,  // 分割策略
    50                        // overlap
);
```

#### PDFReader
PDF文档读取器。

#### WordReader
Word文档读取器，支持表格、图片提取。

### 4. 向量存储（EmbeddingStore）

#### InMemoryStore
内存存储，适合开发测试和小数据集（<10K文档）。

#### QdrantStore
生产级向量数据库，适合大规模数据。

```java
QdrantStore.builder()
    .location("localhost:6334")
    .collectionName("my_collection")
    .dimensions(1024)
    .build()
```

### 5. 文档加载工具（DocumentLoader）

提供便捷的文档加载方法：

```java
// 从字符串加载
List<Document> docs = DocumentLoader.loadFromString(content);

// 从文件加载
List<Document> docs = DocumentLoader.loadFromFile("doc.txt");

// 批量加载目录
List<Document> docs = DocumentLoader.loadFromDirectory("docs", true);

// 打印统计
DocumentLoader.printDocumentStats(docs);

// 测试检索
DocumentLoader.testRetrieval(knowledge, "查询", 3);
```

## 开发指南

### 环境配置

#### 必需环境
- JDK 17+
- Maven 3.6+
- DashScope API Key

#### 环境变量设置
```bash
export DASHSCOPE_API_KEY=your_api_key_here
```

### 依赖管理

项目使用 Maven 管理依赖，主要依赖在 `pom.xml` 中：

```xml
<dependencies>
    <!-- AgentScope All-in-One -->
    <dependency>
        <groupId>io.agentscope</groupId>
        <artifactId>agentscope</artifactId>
        <version>1.0.8</version>
    </dependency>
    
    <!-- Simple RAG Extension -->
    <dependency>
        <groupId>io.agentscope</groupId>
        <artifactId>agentscope-extensions-rag-simple</artifactId>
        <version>1.0.8</version>
    </dependency>
</dependencies>
```

#### 添加可选依赖

**使用 Qdrant:**
```xml
<dependency>
    <groupId>io.qdrant</groupId>
    <artifactId>client</artifactId>
</dependency>
```

**使用 Elasticsearch:**
```xml
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
</dependency>
```

**使用 PostgreSQL:**
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <dependency>
        <groupId>com.pgvector</groupId>
        <artifactId>pgvector</artifactId>
    </dependency>
</dependency>
```

**处理 PDF:**
```xml
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
</dependency>
```

**处理 Word:**
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
</dependency>
```

### 开发流程

#### 1. 创建新的智能体类

```java
package dev.higgs.wizknow;

import io.agentscope.core.ReActAgent;
import io.agentscope.core.model.DashScopeChatModel;

public class MyCustomAgent {
    
    public ReActAgent createAgent() {
        return ReActAgent.builder()
            .name("我的智能体")
            .sysPrompt("你的角色和任务...")
            .model(createChatModel())
            .knowledge(createKnowledge())
            .ragMode(RAGMode.GENERIC)
            .build();
    }
    
    private DashScopeChatModel createChatModel() {
        return DashScopeChatModel.builder()
            .apiKey(System.getenv("DASHSCOPE_API_KEY"))
            .modelName("qwen3-max")
            .build();
    }
    
    private SimpleKnowledge createKnowledge() {
        // 实现知识库创建逻辑
        return null;
    }
}
```

#### 2. 添加知识文档

使用 `DocumentLoader` 工具类：

```java
// 从字符串
String content = "知识内容...";
List<Document> docs = DocumentLoader.loadFromString(content);

// 从文件
List<Document> docs = DocumentLoader.loadFromFile("path/to/doc.txt");

// 批量加载
List<Document> docs = DocumentLoader.loadFromDirectory("docs", true);

// 添加到知识库
knowledge.addDocuments(docs).block();
```

#### 3. 实现问答逻辑

```java
public void askQuestion(ReActAgent agent, String question) {
    Msg message = Msg.builder()
        .textContent(question)
        .build();
    
    Mono<Msg> responseMono = agent.call(message);
    Msg response = responseMono.block();
    
    System.out.println("回答: " + response.getTextContent());
}
```

#### 4. 添加自定义工具

```java
class MyTools {
    
    @Tool(name = "search_database", description = "搜索数据库")
    public String searchDatabase(
        @ToolParam(name = "query", description = "查询条件") 
        String query
    ) {
        // 实现数据库搜索逻辑
        return "搜索结果";
    }
}

// 注册工具
Toolkit toolkit = new Toolkit();
toolkit.registerTool(new MyTools());

ReActAgent agent = ReActAgent.builder()
    .model(model)
    .knowledge(knowledge)
    .toolkit(toolkit)  // 添加工具包
    .build();
```

### 配置最佳实践

#### 检索配置

```java
RetrieveConfig.builder()
    .limit(5)              // 检索文档数：3-5为宜
    .scoreThreshold(0.3)   // 相似度阈值：0.3-0.5
    .build()
```

#### 文档分块配置

```java
// 通用推荐
new TextReader(512, SplitStrategy.PARAGRAPH, 50);

// 精细内容
new TextReader(256, SplitStrategy.SENTENCE, 20);

// 长文档
new TextReader(1024, SplitStrategy.CHARACTER, 100);
```

#### 模型选择

| 场景 | 推荐模型 | 说明 |
|------|---------|------|
| 生产环境 | qwen3-max | 最强性能，高准确率 |
| 开发测试 | qwen3-plus | 平衡性能和速度 |
| 快速验证 | qwen3-turbo | 响应快，成本低 |

## 代码规范

### 命名规范

- **类名**：大驼峰（PascalCase），如 `RAGQAAgentExample`
- **方法名**：小驼峰（camelCase），如 `createAgent`
- **常量**：全大写下划线分隔，如 `DEFAULT_CHUNK_SIZE`
- **变量名**：小驼峰（camelCase）

### 代码风格

- 使用4空格缩进
- 大括号另起一行
- 每行最大长度120字符
- 使用try-with-resources管理资源

### 注释规范

- 类注释：说明类的用途和功能
- 方法注释：说明方法的作用、参数、返回值
- 复杂逻辑：添加行内注释

```java
/**
 * 智能问答智能体 - 带RAG功能
 * 使用AgentScope构建的ReAct智能体
 */
public class RAGQAAgentExample {
    
    /**
     * 创建带有RAG功能的智能问答智能体
     * @return 配置好的ReActAgent实例
     * @throws Exception 配置失败时抛出
     */
    public ReActAgent createQAAgentWithRAG() throws Exception {
        // 实现逻辑
    }
}
```

## 常见任务

### 任务1: 添加新的文档格式支持

创建自定义Reader：

```java
public class CustomReader implements Reader {
    
    @Override
    public Mono<List<Document>> read(ReaderInput input) {
        return Mono.fromCallable(() -> {
            // 1. 读取文件
            String content = readFile(input);
            
            // 2. 分块
            List<String> chunks = chunkContent(content);
            
            // 3. 创建Document对象
            List<Document> documents = new ArrayList<>();
            for (String chunk : chunks) {
                Document doc = Document.builder()
                    .content(chunk)
                    .metadata(Map.of("source", input.getSource()))
                    .build();
                documents.add(doc);
            }
            
            return documents;
        });
    }
    
    @Override
    public List<String> getSupportedFormats() {
        return List.of("custom");
    }
}
```

### 任务2: 切换向量存储

从内存存储切换到Qdrant：

```java
// 修改前
InMemoryStore store = InMemoryStore.builder()
    .dimensions(1024)
    .build();

// 修改后
QdrantStore store = QdrantStore.builder()
    .location("localhost:6334")
    .collectionName("higgs_knowledge")
    .dimensions(1024)
    .build();
```

### 任务3: 实现多轮对话

Generic模式自动支持多轮对话。Agentic模式需配置：

```java
RetrieveConfig config = RetrieveConfig.builder()
    .limit(5)
    .scoreThreshold(0.3)
    .conversationHistory(history)  // 传入对话历史
    .build();
```

### 任务4: 添加长期记忆

```java
ReActAgent.builder()
    .name("助手")
    .model(model)
    .knowledge(knowledge)
    // 启用长期记忆
    .longTermMemoryMode(LongTermMemoryMode.AGENT_CONTROL)
    .build();
```

### 任务5: 部署到生产环境

#### 使用Qdrant
```bash
# 启动Qdrant
docker run -p 6334:6334 qdrant/qdrant
```

#### 配置应用
```java
QdrantStore store = QdrantStore.builder()
    .location(System.getenv("QDRANT_URL"))
    .apiKey(System.getenv("QDRANT_API_KEY"))
    .collectionName("production_knowledge")
    .dimensions(1024)
    .build();
```

## 测试指南

### 单元测试

```java
@Test
public void testDocumentLoading() {
    String content = "测试内容";
    List<Document> docs = DocumentLoader.loadFromString(content);
    
    assertNotNull(docs);
    assertFalse(docs.isEmpty());
    assertEquals("测试内容", docs.get(0).getContent());
}
```

### 集成测试

```java
@Test
public void testRAGRetrieval() throws Exception {
    // 1. 创建知识库
    SimpleKnowledge knowledge = createKnowledge();
    
    // 2. 添加文档
    List<Document> docs = DocumentLoader.loadFromString("测试知识");
    knowledge.addDocuments(docs).block();
    
    // 3. 测试检索
    List<Document> results = knowledge.retrieve("测试", 
        RetrieveConfig.builder().limit(1).build()
    ).block();
    
    assertNotNull(results);
    assertFalse(results.isEmpty());
}
```

## 性能优化

### 1. 向量存储优化
- 使用Qdrant或Elasticsearch替代内存存储
- 配置适当的索引参数
- 定期清理过期数据

### 2. 检索优化
- 调整 `scoreThreshold` 过滤低质量结果
- 减少 `limit` 值
- 使用更合适的分块策略

### 3. 模型优化
- 根据场景选择合适的模型
- 配置合理的超时时间
- 实现缓存机制

### 4. 文档优化
- 控制文档块大小（256-1024字符）
- 合理设置overlap（10-20%）
- 清理无用文档

## 故障排查

### 常见问题

**问题1: API Key错误**
```bash
# 检查环境变量
echo $DASHSCOPE_API_KEY

# 重新设置
export DASHSCOPE_API_KEY=your_key
```

**问题2: 检索结果不准确**
- 提高 `scoreThreshold` 值
- 调整文档分块策略
- 增加相关知识文档

**问题3: 响应速度慢**
- 使用较小的模型（qwen3-plus）
- 减少检索文档数量
- 使用更快的向量存储

**问题4: 内存溢出**
- 使用Qdrant等持久化存储
- 减小批量加载的文档数量
- 调整JVM堆内存大小

### 日志配置

在 `pom.xml` 中添加SLF4J实现：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.14</version>
</dependency>
```

配置日志级别（`logback.xml`）：

```xml
<configuration>
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
    
    <!-- 调试模式 -->
    <!-- <root level="DEBUG"> -->
    
    <logger name="io.agentscope" level="DEBUG" />
</configuration>
```

## 扩展资源

### 官方文档
- [AgentScope Java 文档](https://java.agentscope.io/)
- [AgentScope GitHub](https://github.com/agentscope-ai/agentscope-java)
- [DashScope 文档](https://help.aliyun.com/zh/dashscope/)
- [Qdrant 文档](https://qdrant.tech/documentation/)

### 项目文档
- [RAG使用文档](./RAG_README.md)
- [快速入门指南](./QUICKSTART.md)

### 示例代码
- `RAGQAAgentExample.java` - 基础RAG智能体
- `AdvancedRAGExample.java` - 高级功能演示
- `DocumentLoader.java` - 文档加载工具

## 贡献指南

### 代码提交规范

```
类型(范围): 简短描述

详细说明（可选）

类型:
- feat: 新功能
- fix: 修复bug
- docs: 文档更新
- style: 代码格式调整
- refactor: 重构
- test: 测试相关
- chore: 构建/工具相关
```

示例：
```
feat(agent): 添加自定义工具支持

实现了工具注册和调用机制，允许用户扩展智能体能力。
```

### Pull Request流程

1. Fork项目到个人仓库
2. 创建特性分支：`git checkout -b feature/my-feature`
3. 提交更改：`git commit -m 'feat: add my feature'`
4. 推送分支：`git push origin feature/my-feature`
5. 创建Pull Request

## 许可证

MIT License

## 联系方式

- 项目地址：https://github.com/your-org/higgs-wizknow
- 问题反馈：GitHub Issues
