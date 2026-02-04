# frontent_pw3_-tp1-grid

flowchart TB
  %% ============ UI / App Entry ============
  UI[App / Features Layer\n(SwiftUI Views + ViewModels)\n(Not included now)]:::ui

  %% ============ Pipeline ============
  subgraph PIPE[Domain Orchestrator]
    P[VideoRecipePipeline\n(ONE entrypoint)\nimportAndExtractRecipe()]:::core
  end

  %% ============ Domain ============
  subgraph DOMAIN[Domain Layer]
    CFG[PipelineConfig\n(tier, variation, compression, thinking)]:::domain
    VAR[RecipeVariation\nFrontier/Absolute/Budget/Privacy]:::domain
    LIM[RateLimiter\nUsagePolicy + UsageStore]:::domain
    DTO[RecipeCardDTO\n(Strict JSON contract)]:::domain
    STRAT[RecipeExtractionStrategy]:::domain
    FACT[StrategyFactory]:::domain
  end

  %% ============ Providers ============
  subgraph PROVIDERS[Providers Layer]
    VS[VideoStore\n(copy picked video to app storage)]:::prov
    VP[VideoProcessor\n(compress 480p, low fps knob)]:::prov
    AI[RecipeAIProvider]:::prov
    G[GEMINI Provider\n(Gemini 3 Flash multimodal)\n(MVP engine)]:::prov
    PR[Prompts\n(schema + preferences + timelines)]:::prov
  end

  %% ============ Persistence ============
  subgraph DATA[Persistence Layer (SwiftData)]
    REPO[RecipeRepository\n(SwiftData wrapper)]:::data
    MAP[RecipeCardMapper\n(DTO -> SwiftData models)]:::data
    SD[(SwiftData Models)\nUserProfile\nRecipe\nIngredient\nRecipeStep(timelines)\nRecipeAnalysis\nPreferenceReport]:::data
  end

  %% ======= Connections =======
  UI -->|calls| P

  P --> CFG
  P --> LIM
  P --> VS
  P --> VP
  P --> FACT
  FACT --> STRAT
  STRAT --> AI
  AI --> G
  G --> PR
  G -->|returns JSON| DTO

  P --> REPO
  P --> MAP
  MAP --> SD
  REPO --> SD

  %% ============ Notes ============
  CFG --> VAR

  classDef ui fill:#eef,stroke:#556,stroke-width:1px;
  classDef core fill:#ffe9c6,stroke:#b86,stroke-width:1px;
  classDef domain fill:#e8ffe8,stroke:#5a7,stroke-width:1px;
  classDef prov fill:#e8f2ff,stroke:#58a,stroke-width:1px;
  classDef data fill:#fff0f5,stroke:#a58,stroke-width:1px;
