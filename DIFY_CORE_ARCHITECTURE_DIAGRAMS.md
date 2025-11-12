# Dify Core 架构图

## 1. 整体架构图

```mermaid
graph TB
    subgraph "Frontend 前端"
        WebUI[Web UI<br/>Next.js + React]
    end
    
    subgraph "API Layer 接口层"
        Controllers[Controllers<br/>路由和控制器]
        Services[Services<br/>业务服务层]
    end
    
    subgraph "Core Layer 核心层"
        App[Application Framework<br/>应用框架]
        Workflow[Workflow Engine<br/>工作流引擎]
        Agent[Agent System<br/>智能体系统]
        RAG[RAG System<br/>检索增强生成]
        Tools[Tools System<br/>工具系统]
        ModelRuntime[Model Runtime<br/>模型运行时]
        Plugin[Plugin System<br/>插件系统]
    end
    
    subgraph "Infrastructure 基础设施"
        DB[(PostgreSQL<br/>主数据库)]
        Redis[(Redis<br/>缓存&队列)]
        VectorDB[(Vector DB<br/>向量数据库)]
        Celery[Celery<br/>异步任务]
    end
    
    subgraph "External Services 外部服务"
        LLM[LLM Providers<br/>OpenAI/Anthropic/etc]
        Embedding[Embedding Models<br/>嵌入模型]
        Monitoring[Monitoring<br/>LangFuse/LangSmith/etc]
    end
    
    WebUI --> Controllers
    Controllers --> Services
    Services --> App
    Services --> Workflow
    Services --> Agent
    
    App --> ModelRuntime
    Workflow --> ModelRuntime
    Workflow --> Tools
    Workflow --> RAG
    Agent --> ModelRuntime
    Agent --> Tools
    
    RAG --> VectorDB
    RAG --> ModelRuntime
    Tools --> Plugin
    
    ModelRuntime --> LLM
    ModelRuntime --> Embedding
    ModelRuntime --> Redis
    
    App --> DB
    Workflow --> DB
    Services --> Celery
    
    ModelRuntime -.监控.-> Monitoring
    Workflow -.监控.-> Monitoring
```

## 2. Model Runtime 架构

```mermaid
graph TB
    subgraph "Factory Layer 工厂层"
        Factory[ModelProviderFactory<br/>模型提供商工厂]
    end
    
    subgraph "Provider Layer 提供商层"
        Provider1[OpenAI Provider]
        Provider2[Anthropic Provider]
        Provider3[Hugging Face Provider]
        ProviderN[... 更多提供商]
    end
    
    subgraph "Model Layer 模型层"
        LLM[LLM<br/>大语言模型]
        Embedding[Text Embedding<br/>文本嵌入]
        Rerank[Rerank Model<br/>重排序]
        STT[Speech2Text<br/>语音转文字]
        TTS[Text2Speech<br/>文字转语音]
        Moderation[Moderation<br/>内容审核]
    end
    
    subgraph "Config & Auth 配置和认证"
        Config[Provider Configuration<br/>提供商配置]
        Credentials[Credentials Cache<br/>凭证缓存]
        LoadBalance[Load Balancing<br/>负载均衡]
    end
    
    Factory --> Provider1
    Factory --> Provider2
    Factory --> Provider3
    Factory --> ProviderN
    
    Provider1 --> LLM
    Provider1 --> Embedding
    Provider1 --> Moderation
    
    Provider2 --> LLM
    Provider3 --> LLM
    Provider3 --> Embedding
    
    Config --> Factory
    Credentials --> Provider1
    Credentials --> Provider2
    LoadBalance --> Provider1
```

## 3. Workflow Engine 架构

```mermaid
graph TB
    Entry[WorkflowEntry<br/>工作流入口]
    
    subgraph "Graph Engine 图引擎"
        GraphEngine[Graph Engine<br/>图执行引擎]
        Graph[Graph Structure<br/>图结构]
        RuntimeState[Runtime State<br/>运行时状态]
        VarPool[Variable Pool<br/>变量池]
    end
    
    subgraph "Workflow Nodes 工作流节点"
        StartNode[Start Node<br/>开始]
        LLMNode[LLM Node<br/>语言模型]
        ToolNode[Tool Node<br/>工具]
        KnowledgeNode[Knowledge Retrieval<br/>知识检索]
        CodeNode[Code Node<br/>代码执行]
        HTTPNode[HTTP Request<br/>HTTP请求]
        IfElseNode[If/Else<br/>条件分支]
        LoopNode[Loop<br/>循环]
        EndNode[End Node<br/>结束]
    end
    
    subgraph "Node Execution 节点执行"
        NodeExecutor[Node Executor<br/>节点执行器]
        EventEmitter[Event Emitter<br/>事件发射器]
    end
    
    Entry --> GraphEngine
    GraphEngine --> Graph
    GraphEngine --> RuntimeState
    RuntimeState --> VarPool
    
    Graph --> StartNode
    StartNode --> LLMNode
    StartNode --> ToolNode
    StartNode --> KnowledgeNode
    LLMNode --> CodeNode
    ToolNode --> HTTPNode
    KnowledgeNode --> IfElseNode
    IfElseNode --> LoopNode
    LoopNode --> EndNode
    
    LLMNode --> NodeExecutor
    ToolNode --> NodeExecutor
    NodeExecutor --> EventEmitter
    EventEmitter --> RuntimeState
```

## 4. RAG System 架构

```mermaid
graph TB
    subgraph "Document Input 文档输入"
        Upload[Document Upload<br/>文档上传]
        WebCrawl[Web Crawling<br/>网页爬取]
        API[API Integration<br/>API集成]
    end
    
    subgraph "Document Processing 文档处理"
        Extractor[Extractor<br/>提取器]
        Cleaner[Cleaner<br/>清洗器]
        Splitter[Splitter<br/>分割器]
    end
    
    subgraph "Embedding & Storage 嵌入与存储"
        EmbedModel[Embedding Model<br/>嵌入模型]
        VectorDB[Vector Database<br/>向量数据库]
        Docstore[Document Store<br/>文档存储]
    end
    
    subgraph "Retrieval 检索"
        VectorSearch[Vector Search<br/>向量搜索]
        FullText[Full-Text Search<br/>全文搜索]
        Hybrid[Hybrid Retrieval<br/>混合检索]
    end
    
    subgraph "Post Processing 后处理"
        Rerank[Rerank<br/>重排序]
        PostProcessor[Post Processor<br/>后处理器]
    end
    
    Upload --> Extractor
    WebCrawl --> Extractor
    API --> Extractor
    
    Extractor --> Cleaner
    Cleaner --> Splitter
    
    Splitter --> EmbedModel
    EmbedModel --> VectorDB
    Splitter --> Docstore
    
    VectorDB --> VectorSearch
    Docstore --> FullText
    
    VectorSearch --> Hybrid
    FullText --> Hybrid
    
    Hybrid --> Rerank
    Rerank --> PostProcessor
    
    PostProcessor --> Result[Retrieved Documents<br/>检索结果]
```

## 5. Application Types 应用类型流程

```mermaid
graph LR
    subgraph "Application Types 应用类型"
        Completion[Completion App<br/>文本补全应用]
        Chat[Chat App<br/>对话应用]
        AgentChat[Agent Chat App<br/>智能体对话应用]
        AdvancedChat[Advanced Chat App<br/>高级对话应用]
        WorkflowApp[Workflow App<br/>工作流应用]
    end
    
    subgraph "Core Components 核心组件"
        BaseRunner[Base App Runner<br/>基础运行器]
        QueueManager[Queue Manager<br/>队列管理器]
        ModelManager[Model Manager<br/>模型管理器]
    end
    
    Completion --> BaseRunner
    Chat --> BaseRunner
    AgentChat --> BaseRunner
    AdvancedChat --> BaseRunner
    WorkflowApp --> WorkflowRunner[Workflow Runner<br/>工作流运行器]
    
    BaseRunner --> QueueManager
    BaseRunner --> ModelManager
    WorkflowRunner --> QueueManager
    
    QueueManager --> Stream[Stream Output<br/>流式输出]
```

## 6. Tools System 架构

```mermaid
graph TB
    ToolManager[Tool Manager<br/>工具管理器]
    
    subgraph "Tool Types 工具类型"
        Builtin[Builtin Tools<br/>内置工具]
        API[API Tools<br/>API工具]
        Plugin[Plugin Tools<br/>插件工具]
        MCP[MCP Tools<br/>MCP工具]
        WorkflowTool[Workflow Tools<br/>工作流工具]
    end
    
    subgraph "Builtin Tools Examples"
        Google[Google Search]
        Wikipedia[Wikipedia]
        Weather[Weather]
        Calculator[Calculator]
    end
    
    subgraph "Tool Runtime"
        ToolRuntime[Tool Runtime<br/>工具运行时]
        ToolEngine[Tool Engine<br/>工具引擎]
    end
    
    ToolManager --> Builtin
    ToolManager --> API
    ToolManager --> Plugin
    ToolManager --> MCP
    ToolManager --> WorkflowTool
    
    Builtin --> Google
    Builtin --> Wikipedia
    Builtin --> Weather
    Builtin --> Calculator
    
    Builtin --> ToolRuntime
    API --> ToolRuntime
    Plugin --> ToolRuntime
    MCP --> ToolRuntime
    WorkflowTool --> ToolRuntime
    
    ToolRuntime --> ToolEngine
```

## 7. Agent System 执行流程

```mermaid
sequenceDiagram
    participant User as User
    participant AgentRunner as Agent Runner
    participant LLM as LLM
    participant Tools as Tool Manager
    participant Memory as Memory
    
    User->>AgentRunner: 发送查询
    AgentRunner->>Memory: 加载对话历史
    Memory-->>AgentRunner: 返回历史记录
    
    loop Agent Reasoning Loop
        AgentRunner->>LLM: 发送提示词 + 工具列表
        LLM-->>AgentRunner: 返回响应（文本 or 工具调用）
        
        alt 需要调用工具
            AgentRunner->>Tools: 调用工具
            Tools-->>AgentRunner: 返回工具结果
            AgentRunner->>LLM: 将结果发送给LLM
        else 直接回答
            AgentRunner-->>User: 返回最终答案
        end
    end
    
    AgentRunner->>Memory: 保存对话
```

## 8. 数据流向图

```mermaid
graph LR
    subgraph "Input Layer 输入层"
        UserInput[User Input<br/>用户输入]
        API[API Request<br/>API请求]
    end
    
    subgraph "Processing Layer 处理层"
        Moderation[Content Moderation<br/>内容审核]
        Transform[Prompt Transform<br/>提示词转换]
        VariableReplace[Variable Replace<br/>变量替换]
    end
    
    subgraph "Execution Layer 执行层"
        AppExec[App Execution<br/>应用执行]
        WorkflowExec[Workflow Execution<br/>工作流执行]
        AgentExec[Agent Execution<br/>智能体执行]
    end
    
    subgraph "Model Layer 模型层"
        ModelCall[Model Invocation<br/>模型调用]
        ToolCall[Tool Invocation<br/>工具调用]
        KnowledgeQuery[Knowledge Query<br/>知识查询]
    end
    
    subgraph "Output Layer 输出层"
        Response[Response<br/>响应]
        Stream[Streaming Output<br/>流式输出]
    end
    
    UserInput --> Moderation
    API --> Moderation
    Moderation --> Transform
    Transform --> VariableReplace
    
    VariableReplace --> AppExec
    VariableReplace --> WorkflowExec
    VariableReplace --> AgentExec
    
    AppExec --> ModelCall
    WorkflowExec --> ModelCall
    WorkflowExec --> ToolCall
    WorkflowExec --> KnowledgeQuery
    AgentExec --> ModelCall
    AgentExec --> ToolCall
    
    ModelCall --> Response
    ModelCall --> Stream
    ToolCall --> Response
    KnowledgeQuery --> Response
```

## 9. 监控和追踪架构

```mermaid
graph TB
    subgraph "Application Layer 应用层"
        App[Dify Application<br/>Dify应用]
    end
    
    subgraph "Tracing Layer 追踪层"
        Callback[Trace Callback<br/>追踪回调]
        Collector[Trace Collector<br/>追踪收集器]
    end
    
    subgraph "Trace Systems 追踪系统"
        LangFuse[LangFuse]
        LangSmith[LangSmith]
        Phoenix[Arize Phoenix]
        Weave[Weave]
        Opik[Opik]
        Aliyun[Aliyun Trace<br/>阿里云追踪]
    end
    
    subgraph "Metrics 指标"
        Latency[Latency<br/>延迟]
        TokenUsage[Token Usage<br/>Token使用量]
        Cost[Cost<br/>成本]
        Errors[Errors<br/>错误]
    end
    
    App --> Callback
    Callback --> Collector
    
    Collector --> LangFuse
    Collector --> LangSmith
    Collector --> Phoenix
    Collector --> Weave
    Collector --> Opik
    Collector --> Aliyun
    
    LangFuse --> Latency
    LangFuse --> TokenUsage
    LangFuse --> Cost
    LangFuse --> Errors
```

## 10. 插件系统架构

```mermaid
graph TB
    subgraph "Plugin Management 插件管理"
        PluginMgr[Plugin Manager<br/>插件管理器]
        PluginRegistry[Plugin Registry<br/>插件注册表]
    end
    
    subgraph "Plugin Types 插件类型"
        ToolPlugin[Tool Plugin<br/>工具插件]
        ModelPlugin[Model Plugin<br/>模型插件]
        ExtensionPlugin[Extension Plugin<br/>扩展插件]
    end
    
    subgraph "Plugin Lifecycle 插件生命周期"
        Install[Install<br/>安装]
        Enable[Enable<br/>启用]
        Configure[Configure<br/>配置]
        Invoke[Invoke<br/>调用]
        Disable[Disable<br/>禁用]
        Uninstall[Uninstall<br/>卸载]
    end
    
    subgraph "Plugin Communication 插件通信"
        Endpoint[Plugin Endpoint<br/>插件端点]
        BackInvoke[Backwards Invocation<br/>反向调用]
        Auth[Plugin Auth<br/>插件认证]
    end
    
    PluginMgr --> PluginRegistry
    PluginRegistry --> ToolPlugin
    PluginRegistry --> ModelPlugin
    PluginRegistry --> ExtensionPlugin
    
    ToolPlugin --> Install
    Install --> Enable
    Enable --> Configure
    Configure --> Invoke
    Invoke --> Disable
    Disable --> Uninstall
    
    ToolPlugin --> Endpoint
    ToolPlugin --> BackInvoke
    ToolPlugin --> Auth
```

## 说明

这些架构图展示了 Dify Core 的主要组件和它们之间的关系：

1. **整体架构图**: 展示了从前端到后端的完整架构层次
2. **Model Runtime 架构**: 展示了模型运行时的三层架构设计
3. **Workflow Engine 架构**: 展示了工作流引擎的核心组件
4. **RAG System 架构**: 展示了从文档处理到检索的完整流程
5. **Application Types**: 展示了不同应用类型及其核心组件
6. **Tools System**: 展示了工具系统的组织结构
7. **Agent System 执行流程**: 展示了智能体的执行序列
8. **数据流向图**: 展示了数据在系统中的流动
9. **监控和追踪架构**: 展示了可观测性系统
10. **插件系统架构**: 展示了插件的生命周期和通信机制

这些图表使用 Mermaid 语法编写，可以在支持 Mermaid 的 Markdown 渲染器中查看（如 GitHub、GitLab、VS Code 等）。
