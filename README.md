# ai-writing-workflow
```mermaid
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
    style NewMem fill:#FFF3E0

```markdown
## Multi-Agent Architecture

```mermaid
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

```markdown
## Planning Phase Sequence

```mermaid
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
