# ai-writing-workflow
```mermaid
graph TB
    Start([Game Start]) --> Init[Initialize Game]
    Init --> LoadMemory[Load Long-term Memory]
    LoadMemory --> AssignRoles[Assign Roles]
    AssignRoles --> GameLoop{Game Main Loop}

    GameLoop --> NightPhase[Night Phase]
    GameLoop --> DayPhase[Day Phase]
    GameLoop --> CheckWin{Check Victory Conditions}

    CheckWin -->|Werewolf Win| WolfWin[Werewolf Team Wins]
    CheckWin -->|Good Win| GoodWin[Good Team Wins]
    CheckWin -->|Continue| GameLoop

    WolfWin --> Analysis[Game Analysis]
    GoodWin --> Analysis
    Analysis --> SaveMemory[Save Experience to Memory]
    SaveMemory --> End([Game End])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style WolfWin fill:#FF6B6B
    style GoodWin fill:#4ECDC4
    style GameLoop fill:#FFE66D
```

## Multi-Agent Interaction Architecture

```mermaid
graph LR
    subgraph "DeepSeek API"
        API[DeepSeek Chat API]
    end

    subgraph "Game Control Layer"
        GameMaster[Game Master<br/>WerewolfGame]
        GameState[Game State Management<br/>GameState]
        Memory[Long-term Memory<br/>MemoryManager]
    end

    subgraph "AI Agent Layer"
        Wolf1[Werewolf Agent #1]
        Wolf2[Werewolf Agent #2]
        Villager1[Villager Agent #1]
        Villager2[Villager Agent #2]
        Villager3[Villager Agent #3]
        Seer[Seer Agent]
        Witch[Witch Agent]
        Hunter[Hunter Agent]
    end

    GameMaster --> GameState
    GameMaster --> Memory

    GameMaster -.Query.-> Wolf1
    GameMaster -.Query.-> Wolf2
    GameMaster -.Query.-> Villager1
    GameMaster -.Query.-> Villager2
    GameMaster -.Query.-> Villager3
    GameMaster -.Query.-> Seer
    GameMaster -.Query.-> Witch
    GameMaster -.Query.-> Hunter

    Wolf1 --> API
    Wolf2 --> API
    Villager1 --> API
    Villager2 --> API
    Villager3 --> API
    Seer --> API
    Witch --> API
    Hunter --> API

    API -.Return Decision.-> Wolf1
    API -.Return Decision.-> Wolf2
    API -.Return Decision.-> Villager1
    API -.Return Decision.-> Villager2
    API -.Return Decision.-> Villager3
    API -.Return Decision.-> Seer
    API -.Return Decision.-> Witch
    API -.Return Decision.-> Hunter

    style GameMaster fill:#FFE66D
    style API fill:#FF6B6B
```

## Night Phase Flow

```mermaid
sequenceDiagram
    participant GM as Game Master
    participant Wolf1 as Werewolf 1
    participant Wolf2 as Werewolf 2
    participant Seer as Seer
    participant Witch as Witch
    participant API as DeepSeek API
    participant GS as Game State

    GM->>GM: Enter Night Phase

    rect rgb(240, 240, 240)
        Note over GM,Wolf2: 1. Werewolves Discussion
        GM->>Wolf1: Choose kill target
        Wolf1->>API: Send context + history
        API-->>Wolf1: Return decision
        Wolf1-->>GM: Suggest killing player X

        GM->>Wolf2: Choose kill target
        Wolf2->>API: Send context + history
        API-->>Wolf2: Return decision
        Wolf2-->>GM: Suggest killing player Y

        GM->>GM: Count werewolf votes
        GM->>GS: Record kill target
    end

    rect rgb(230, 230, 250)
        Note over GM,Seer: 2. Seer Check Phase
        GM->>Seer: Choose check target
        Seer->>API: Send current info
        API-->>Seer: Return decision
        Seer-->>GM: Check player X
        GM->>GS: Query player X identity
        GS-->>GM: Return identity info
        GM-->>Seer: Tell check result
    end

    rect rgb(255, 240, 245)
        Note over GM,Witch: 3. Witch Action Phase
        GM->>Witch: Notify killed player
        Witch->>API: Send current info
        API-->>Witch: Return decision
        Witch-->>GM: Decide to use potion
        GM->>GS: Update night result
    end

    GM->>GM: End Night Phase
```

## Day Phase Flow

```mermaid
sequenceDiagram
    participant GM as Game Master
    participant P1 as Player 1
    participant P2 as Player 2
    participant PN as Player N
    participant API as DeepSeek API
    participant GS as Game State

    GM->>GM: Enter Day Phase

    rect rgb(255, 250, 205)
        Note over GM,GS: 1. Announce Deaths
        GM->>GS: Query night deaths
        GS-->>GM: Return death list
        GM->>GM: Announce dead players
        GM->>GM: Dead players' last words
    end

    rect rgb(240, 255, 240)
        Note over GM,PN: 2. Player Discussion
        loop All Alive Players
            GM->>P1: Your turn to speak
            P1->>API: Send complete game info<br/>+ history memory
            API-->>P1: Generate speech
            P1-->>GM: Express opinion
            GM->>GS: Record speech
        end
    end

    rect rgb(255, 240, 240)
        Note over GM,PN: 3. Voting Phase
        loop All Alive Players
            GM->>P1: Please vote
            P1->>API: Analyze all speeches
            API-->>P1: Return vote decision
            P1-->>GM: Vote for player X
            GM->>GS: Record vote
        end

        GM->>GM: Count vote results
        GM->>GS: Execute most voted player
        GM->>GM: Trigger Hunter ability (if Hunter)
    end

    GM->>GM: End Day Phase
```

## Single AI Agent Decision Flow

```mermaid
graph TB
    Start([Receive Game State]) --> LoadHistory[Load History Memory]
    LoadHistory --> BuildContext[Build Context]

    BuildContext --> ContextType{Context Type}

    ContextType -->|Werewolf| WolfContext[Werewolf Context<br/>- Teammate info<br/>- Kill history<br/>- Disguise strategy]
    ContextType -->|Seer| SeerContext[Seer Context<br/>- Check history<br/>- Identity reveal<br/>- Check strategy]
    ContextType -->|Witch| WitchContext[Witch Context<br/>- Potion usage<br/>- Killed player<br/>- Save/poison decision]
    ContextType -->|Hunter| HunterContext[Hunter Context<br/>- Identity exposure<br/>- Shoot target choice]
    ContextType -->|Villager| VillagerContext[Villager Context<br/>- Speech analysis<br/>- Logical reasoning]

    WolfContext --> CallAPI[Call DeepSeek API]
    SeerContext --> CallAPI
    WitchContext --> CallAPI
    HunterContext --> CallAPI
    VillagerContext --> CallAPI

    CallAPI --> ParseResponse[Parse AI Response]
    ParseResponse --> ExtractDecision[Extract Decision]

    ExtractDecision --> ValidateDecision{Validate Decision}
    ValidateDecision -->|Valid| ReturnDecision[Return Decision]
    ValidateDecision -->|Invalid| Retry[Retry Request]
    Retry --> CallAPI

    ReturnDecision --> UpdateMemory[Update Memory]
    UpdateMemory --> End([Decision Complete])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CallAPI fill:#FF6B6B
```

## Long-term Memory System Flow

```mermaid
graph TB
    Start([Game End]) --> Analyze[Game Analysis]

    Analyze --> ExtractKey[Extract Key Moments]
    ExtractKey --> EvaluatePlayer[Evaluate Player Performance]
    EvaluatePlayer --> GenerateInsight[Generate Lessons Learned]

    GenerateInsight --> RoleInsight{Classify by Role}

    RoleInsight --> WolfInsight[Werewolf Experience<br/>- Disguise techniques<br/>- Kill strategy<br/>- Response methods]
    RoleInsight --> SeerInsight[Seer Experience<br/>- Check priority<br/>- Reveal timing<br/>- Info relay]
    RoleInsight --> WitchInsight[Witch Experience<br/>- Potion timing<br/>- Save judgment<br/>- Poison target]
    RoleInsight --> OtherInsight[Other Role Experience]

    WolfInsight --> SaveMemory[Save to Memory File]
    SeerInsight --> SaveMemory
    WitchInsight --> SaveMemory
    OtherInsight --> SaveMemory

    SaveMemory --> LimitHistory{Exceed History Limit?}
    LimitHistory -->|Yes| CleanOld[Clean Oldest Records]
    LimitHistory -->|No| Done
    CleanOld --> Done[Complete Save]

    Done --> NextGame([Next Game Starts])
    NextGame --> LoadMemory[Load History Experience]
    LoadMemory --> InjectPrompt[Inject into Role Prompt]
    InjectPrompt --> ImprovedAI[Improved AI Performance]

    style Start fill:#FFE66D
    style ImprovedAI fill:#4ECDC4
```

## Core Class Relationship Diagram

```mermaid
classDiagram
    class WerewolfConfig {
        +TOTAL_PLAYERS: int
        +ROLE_CONFIG: dict
        +TEMPERATURE: float
        +MAX_SPEECH_WORDS: int
        +validate()
    }

    class DeepSeekClient {
        -client: OpenAI
        -model: str
        +chat(system, user): str
        +chat_with_context(messages): str
    }

    class BasePlayer {
        #player_id: int
        #role: str
        #is_alive: bool
        #client: DeepSeekClient
        +speak(context): str
        +vote(context): int
        #_build_system_prompt(): str
    }

    class Werewolf {
        +discuss_kill_target(): int
    }

    class Seer {
        +decide_check_target(): int
    }

    class Witch {
        +decide_use_antidote(): bool
        +decide_use_poison(): int
    }

    class Hunter {
        +decide_shoot_target(): int
    }

    class Villager {
    }

    class GameState {
        +players: list
        +day: int
        +alive_players: list
        +dead_players: list
        +speech_history: list
        +vote_history: list
        +update_player_status()
        +record_speech()
        +record_vote()
        +export_to_dict(): dict
    }

    class WerewolfGame {
        -config: WerewolfConfig
        -state: GameState
        -players: list
        +initialize_game()
        +night_phase()
        +day_phase()
        +check_win_condition(): str
        +run()
    }

    class GameAnalyzer {
        +analyze_game(state): dict
        +find_key_moments(): list
        +evaluate_players(): dict
        +extract_lessons(): dict
    }

    class MemoryManager {
        +load_memories(): dict
        +save_game_result()
        +update_role_memory()
        +get_role_history(): list
    }

    BasePlayer <|-- Werewolf
    BasePlayer <|-- Seer
    BasePlayer <|-- Witch
    BasePlayer <|-- Hunter
    BasePlayer <|-- Villager

    WerewolfGame --> WerewolfConfig
    WerewolfGame --> GameState
    WerewolfGame --> BasePlayer
    WerewolfGame --> GameAnalyzer
    WerewolfGame --> MemoryManager

    BasePlayer --> DeepSeekClient

    GameAnalyzer --> GameState
    MemoryManager --> GameState
```

## Data Flow Diagram

```mermaid
graph LR
    subgraph "Input Layer"
        ENV[.env Config]
        Memory[History Memory Files]
    end

    subgraph "Processing Layer"
        Config[Game Config]
        GameMaster[Game Master]
        Agents[AI Agent Group]
        API[DeepSeek API]
    end

    subgraph "Output Layer"
        Console[Console Output]
        History[Game History Records]
        Analysis[Game Analysis]
        NewMemory[Update Memory]
    end

    ENV --> Config
    Memory --> Agents

    Config --> GameMaster
    GameMaster --> Agents
    Agents --> API
    API --> Agents
    Agents --> GameMaster

    GameMaster --> Console
    GameMaster --> History
    GameMaster --> Analysis
    Analysis --> NewMemory

    NewMemory -.Next Game.-> Memory

    style ENV fill:#E8F5E9
    style API fill:#FFEBEE
    style Console fill:#E3F2FD
    style NewMemory fill:#FFF3E0
```

## Key Design Patterns

### 1. Observer Pattern
- **GameState** maintains game state
- **WerewolfGame** observes and responds to state changes
- **AI Agents** make decisions based on state

### 2. Strategy Pattern
- **BasePlayer** defines common interface
- Each role class implements different strategy methods
- Runtime calls corresponding strategy based on role type

### 3. Singleton Pattern
- **DeepSeekClient** globally shares one API client
- **MemoryManager** centrally manages memory files

### 4. Template Method Pattern
- **BasePlayer** defines decision-making process framework
- Subclasses override `_build_system_prompt()` to customize behavior

## Performance Optimization Points

1. **API Call Optimization**
   - Cache API client instance
   - Concurrent processing of independent AI decisions (optional)

2. **Memory System Optimization**
   - Limit history record count (MAX_HISTORY_GAMES)
   - Load only relevant role memories

3. **Prompt Optimization**
   - Streamline context information
   - Use structured output format
   - Word limits prevent overly long responses

4. **State Management Optimization**
   - Incremental updates instead of full copy
   - Lazy loading of historical records
