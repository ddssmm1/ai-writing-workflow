# ai-writing-workflow
graph TB
    Start([Start: 用户提交写作任务]) --> Init[初始化任务<br/>加载配置]
    Init --> LoadMemory[加载长期写作经验]
    LoadMemory --> AssignAgents[创建并分配角色 Agent]
    AssignAgents --> MainLoop{写作主循环}

    MainLoop --> PlanPhase[选题与结构规划阶段]
    MainLoop --> DraftPhase[初稿撰写阶段]
    MainLoop --> ReviewPhase[审稿与修改阶段]
    MainLoop --> CheckDone{是否达到完成标准?}

    CheckDone -->|是| Finalize[生成最终稿件]
    CheckDone -->|否| MainLoop

    Finalize --> Analyze[写作过程分析]
    Analyze --> SaveMemory[保存经验到长期记忆]
    SaveMemory --> End([End: 输出论文与报告])

    style Start fill:#C8E6C9
    style End fill:#FFCDD2
    style MainLoop fill:#FFF9C4
    style Finalize fill:#B2EBF2
graph LR
    subgraph "LLM API Layer"
        API[LLM Chat API]
    end

    subgraph "Control Layer"
        Orchestrator[写作协调器<br/>WritingOrchestrator]
        DocState[文稿状态管理<br/>DocumentState]
        Memory[长期记忆管理<br/>WritingMemoryManager]
    end

    subgraph "AI Agent Layer"
        TopicAgent[选题与需求分析 Agent]
        SearchAgent[文献检索 Agent]
        OutlineAgent[提纲规划 Agent]
        DraftAgent[正文撰写 Agent]
        ReviewerAgent[审稿人 Agent]
        RewriterAgent[润色与重写 Agent]
        CitationAgent[引用与格式 Agent]
    end

    Orchestrator --> DocState
    Orchestrator --> Memory

    Orchestrator -.任务指令.-> TopicAgent
    Orchestrator -.任务指令.-> SearchAgent
    Orchestrator -.任务指令.-> OutlineAgent
    Orchestrator -.任务指令.-> DraftAgent
    Orchestrator -.任务指令.-> ReviewerAgent
    Orchestrator -.任务指令.-> RewriterAgent
    Orchestrator -.任务指令.-> CitationAgent

    TopicAgent --> API
    SearchAgent --> API
    OutlineAgent --> API
    DraftAgent --> API
    ReviewerAgent --> API
    RewriterAgent --> API
    CitationAgent --> API

    API -.决策/文本输出.-> TopicAgent
    API -.决策/文本输出.-> SearchAgent
    API -.决策/文本输出.-> OutlineAgent
    API -.决策/文本输出.-> DraftAgent
    API -.反馈.-> ReviewerAgent
    API -.反馈.-> RewriterAgent
    API -.引用建议.-> CitationAgent

    TopicAgent --> Orchestrator
    SearchAgent --> Orchestrator
    OutlineAgent --> Orchestrator
    DraftAgent --> Orchestrator
    ReviewerAgent --> Orchestrator
    RewriterAgent --> Orchestrator
    CitationAgent --> Orchestrator

    style Orchestrator fill:#FFE082
    style API fill:#EF9A9A
sequenceDiagram
    participant User as User
    participant OR as Writing Orchestrator
    participant Topic as Topic Agent
    participant Search as Search Agent
    participant Outline as Outline Agent
    participant API as LLM API
    participant DS as Document State

    User->>OR: 提交写作需求（主题 / 字数 / 风格）
    OR->>DS: 初始化文稿状态

    rect rgb(240, 248, 255)
        Note over OR,Outline: 1. 选题与范围收敛
        OR->>Topic: 分析需求，给出更具体题目
        Topic->>API: 传入需求 + 历史经验
        API-->>Topic: 返回多个备选题目
        Topic-->>OR: 推荐题目与理由
        OR->>DS: 记录选题结果
    end

    rect rgb(255, 250, 240)
        Note over OR,Search: 2. 初步文献方向
        OR->>Search: 生成检索关键词 & 方向
        Search->>API: 生成关键词 / 可能的研究点
        API-->>Search: 返回关键词+小结
        Search-->>OR: 输出建议
        OR->>DS: 附加为“文献方向草稿”
    end

    rect rgb(240, 255, 240)
        Note over OR,Outline: 3. 生成文章结构
        OR->>Outline: 基于选题 + 文献方向生成提纲
        Outline->>API: 调用模型设计章节结构
        API-->>Outline: 输出章节结构（1,1.1,1.2...）
        Outline-->>OR: 返回结构
        OR->>DS: 保存为“文章提纲”
    end
sequenceDiagram
    participant OR as Orchestrator
    participant Draft as Draft Agent
    participant API as LLM API
    participant DS as Document State

    OR->>OR: 进入 Draft 阶段
    OR->>DS: 读取当前提纲 & 已写内容

    loop 逐个小节
        OR->>Draft: 写作小节 N 的正文
        Draft->>API: 传入小节标题 + 相关上下文
        API-->>Draft: 返回该小节草稿文本
        Draft-->>OR: 提交小节内容
        OR->>DS: 将小节合并进文稿
    end

    OR->>OR: Draft 阶段结束
sequenceDiagram
    participant OR as Orchestrator
    participant Reviewer as Reviewer Agent
    participant Rewriter as Rewriter Agent
    participant API as LLM API
    participant DS as Document State

    OR->>DS: 获取当前完整草稿
    OR->>Reviewer: 请求审稿（逻辑+结构+学术性）

    Reviewer->>API: 提交全文或分段 + 审稿标准
    API-->>Reviewer: 返回问题列表 & 修改建议
    Reviewer-->>OR: 审稿报告
    OR->>DS: 记录为“审稿记录”

    OR->>Rewriter: 根据审稿意见重写关键段落
    Rewriter->>API: 传入原文 + 审稿意见
    API-->>Rewriter: 返回修改后的文本
    Rewriter-->>OR: 提交改写结果
    OR->>DS: 更新文稿版本
graph TB
    Start([收到任务请求]) --> LoadHistory[加载对应角色历史经验]
    LoadHistory --> GatherState[收集当前文稿状态]

    GatherState --> BuildContext[构建上下文<br/>（任务目标+状态+历史）]

    BuildContext --> CallAPI[调用 LLM API]
    CallAPI --> Parse[解析模型输出]
    Parse --> Validate{结果是否符合格式 / 约束?}

    Validate -->|是| Return[返回结果给协调器]
    Validate -->|否| Retry[调整提示词重试一次]
    Retry --> CallAPI

    Return --> UpdateMemory[追加本次决策摘要到短期记忆]
    UpdateMemory --> End([完成])

    style Start fill:#C8E6C9
    style End fill:#FFCDD2
    style CallAPI fill:#FFAB91
graph TB
    Start([一篇写作任务结束]) --> Analyze[分析写作过程]

    Analyze --> ExtractKey[提取关键决策节点<br/>（选题、提纲、重大修改）]
    ExtractKey --> Evaluate[评估写作质量<br/>和用户反馈]
    Evaluate --> GenInsight[生成经验总结]

    GenInsight --> RoleSplit{按角色分类经验}
    RoleSplit --> TopicMem[选题经验]
    RoleSplit --> SearchMem[检索经验]
    RoleSplit --> OutlineMem[提纲经验]
    RoleSplit --> DraftMem[写作风格经验]
    RoleSplit --> ReviewMem[审稿经验]

    TopicMem --> Save[写入记忆库]
    SearchMem --> Save
    OutlineMem --> Save
    DraftMem --> Save
    ReviewMem --> Save

    Save --> Limit{是否超过最大历史数?}
    Limit -->|是| Clean[删除最旧的记录]
    Limit -->|否| Done[保存完成]

    Done --> Next([下次写作任务])
    Next --> LoadMem[按角色加载相关经验]
    LoadMem --> Inject[注入到对应 Agent 的系统提示词]
    Inject --> Improved[提高写作质量与稳定性]

    style Start fill:#FFF9C4
    style Improved fill:#B2DFDB
classDiagram
    class WritingConfig {
        +MAX_TOKENS: int
        +TARGET_WORDS: int
        +STYLE: str
        +LANGUAGE: str
        +validate()
    }

    class LLMClient {
        -client: OpenAI
        -model: str
        +chat(system: str, user: str): str
        +chat_with_messages(msgs: list): str
    }

    class BaseAgent {
        #agent_name: str
        #role: str
        #client: LLMClient
        +perform_task(context: dict): dict
        #_build_system_prompt(context): str
        #_build_user_prompt(context): str
        #_parse_output(raw: str): dict
    }

    class TopicAgent {
        +suggest_topics(req): dict
    }

    class SearchAgent {
        +generate_keywords(req): list
        +summarize_papers(info): str
    }

    class OutlineAgent {
        +make_outline(req): dict
    }

    class DraftAgent {
        +write_section(section_info): str
    }

    class ReviewerAgent {
        +review_document(doc): dict
    }

    class RewriterAgent {
        +rewrite_paragraph(paragraph, suggestion): str
    }

    class CitationAgent {
        +format_citations(meta): str
        +check_style(doc): dict
    }

    class DocumentState {
        +title: str
        +requirements: dict
        +outline: dict
        +sections: list
        +history_versions: list
        +reviews: list
        +update_section()
        +export_full_text(): str
    }

    class WritingOrchestrator {
        -config: WritingConfig
        -state: DocumentState
        -agents: dict
        -memory: WritingMemoryManager
        +run()
        +run_planning_phase()
        +run_draft_phase()
        +run_review_phase()
    }

    class WritingAnalyzer {
        +analyze_process(state): dict
        +find_key_decisions(): list
        +evaluate_quality(): dict
        +collect_feedback(): dict
    }

    class WritingMemoryManager {
        +load_role_memories(role: str): list
        +save_task_result()
        +update_role_memory(role, summary)
    }

    BaseAgent <|-- TopicAgent
    BaseAgent <|-- SearchAgent
    BaseAgent <|-- OutlineAgent
    BaseAgent <|-- DraftAgent
    BaseAgent <|-- ReviewerAgent
    BaseAgent <|-- RewriterAgent
    BaseAgent <|-- CitationAgent

    WritingOrchestrator --> WritingConfig
    WritingOrchestrator --> DocumentState
    WritingOrchestrator --> BaseAgent
    WritingOrchestrator --> WritingMemoryManager

    BaseAgent --> LLMClient

    WritingAnalyzer --> DocumentState
    WritingMemoryManager --> DocumentState
graph LR
    subgraph "Input Layer"
        UserReq[用户写作需求<br/>(主题/字数/领域)]
        Env[环境配置 .env]
        PastMem[历史写作记忆]
    end

    subgraph "Processing Layer"
        Config[WritingConfig]
        Orchestrator[WritingOrchestrator]
        Agents[Agent 集合]
        API[LLM API]
    end

    subgraph "Output Layer"
        DraftOut[最终论文文本]
        ProcessLog[写作过程日志]
        Analysis[过程分析报告]
        NewMem[新增经验记忆]
    end

    UserReq --> Orchestrator
    Env --> Config
    Config --> Orchestrator

    PastMem --> Agents
    Orchestrator --> Agents
    Agents --> API
    API --> Agents
    Agents --> Orchestrator

    Orchestrator --> DraftOut
    Orchestrator --> ProcessLog
    Orchestrator --> Analysis
    Analysis --> NewMem
    NewMem -.反馈循环.-> PastMem

    style API fill:#FFEBEE
    style DraftOut fill:#E3F2FD
    style NewMem fill:#FFF3E0
