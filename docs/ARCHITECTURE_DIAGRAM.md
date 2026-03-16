# FableForge: Architecture Diagrams

Mermaid diagrams for the personal-project architecture. Render in GitHub, VS Code (with Mermaid extension), or any Mermaid-compatible viewer.

---

## 1. High-level system architecture

```mermaid
flowchart TB
    subgraph client [Client]
        Browser[Browser]
        Editor[Chapter Editor - TipTap]
        Dashboard[Dashboard - Books, Spec, Chapters]
    end

    subgraph nextjs [Next.js App]
        API[API Routes]
        App[App Router / Pages]
    end

    subgraph external [External Services]
        Supabase[(Supabase - DB + Auth)]
        Inngest[Inngest - Background Jobs]
        OpenAI[OpenAI API]
    end

    Browser --> App
    Editor --> API
    Dashboard --> API
    App --> API
    API --> Supabase
    API --> Inngest
    API --> OpenAI
    Inngest --> Supabase
    Inngest --> OpenAI
```

---

## 2. Generate chapter: agent pipeline (sync)

User clicks "Generate" → Orchestrator runs Planner then Writer; streams result to editor. Optional: after save, Continuity agent runs async.

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant API
    participant Orchestrator
    participant DB
    participant Planner
    participant Writer
    participant Inngest

    User->>Client: Click Generate
    Client->>API: POST /generate
    API->>Orchestrator: generateChapter(bookId, chapterId)

    Orchestrator->>DB: Load book_spec, style_profile, artifacts
    DB-->>Orchestrator: continuity, character_state, chapter_memory N-1, outline

    Orchestrator->>Planner: outline + spec + chapter_memory
    Planner->>OpenAI: GPT-4o-mini
    OpenAI-->>Planner: chapter_plan JSON
    Planner->>DB: Store chapter_plan artifact
    Planner-->>Orchestrator: chapter_plan

    Orchestrator->>Writer: plan + spec + style + continuity/character
    Writer->>OpenAI: GPT-4o stream
    loop Stream tokens
        OpenAI-->>Writer: chunk
        Writer-->>Orchestrator: chunk
        Orchestrator-->>Client: chunk
        Client-->>User: Append to editor
    end

    Writer-->>Orchestrator: done
    Orchestrator->>DB: Save chapter content + version + generation_run
    API-->>Client: stream end
    opt Continuity update
        Orchestrator->>Inngest: Trigger continuity job for chapter N
    end
```

---

## 3. Agent handoff and data flow (generate path)

What each agent consumes and produces; arrows show handoffs.

```mermaid
flowchart LR
    subgraph load [Orchestrator loads]
        Spec[book_spec]
        Style[style_profile]
        Cont[continuity_summary]
        CharState[character_state]
        ChapMem[chapter_memory N-1]
        Outline[outline]
    end

    subgraph planner [Chapter Planner - GPT-4o-mini]
        PIn[outline + spec + chapter_memory]
        POut[chapter_plan]
    end

    subgraph writer [Chapter Writer - GPT-4o]
        WIn[plan + spec + style + continuity + character_state]
        WOut[Streamed chapter text]
    end

    Outline --> PIn
    Spec --> PIn
    ChapMem --> PIn
    Spec --> WIn
    Style --> WIn
    Cont --> WIn
    CharState --> WIn
    POut --> WIn
    POut --> writer
    WOut --> Client[Client editor]
```

---

## 4. Inspiration pipeline (async)

User adds inspiration (paste, URL, or search) → Inngest job fetches (if needed), then Inspiration agent analyzes → style_profile saved; Writer and Inline editor use it later.

```mermaid
flowchart TB
    User[User] --> UI[Inspiration UI]
    UI -->|paste / URL / search| API[API: create inspiration_source]
    API --> Inngest[Inngest: analyzeInspiration]
    Inngest --> Resolve{Input type?}
    Resolve -->|paste| UseText[Use raw_text]
    Resolve -->|URL| Fetch[Fetch URL, extract text, limit size]
    Resolve -->|search| Search[Run search, get sample]
    Fetch --> UseText
    Search --> UseText
    UseText --> Agent[Inspiration Pipeline Agent]
    Agent --> OpenAI[OpenAI GPT-4o-mini]
    OpenAI --> Agent
    Agent --> Profile[style_profile JSON]
    Profile --> DB[(style_profiles table)]
    DB -.->|read when generating/editing| Writer[Chapter Writer]
    DB -.->|read when editing| InlineEditor[Inline Editor]
```

---

## 5. Inline edit (single call, no pipeline)

User selects text and picks an action → one Writer-style LLM call → replacement text; no other agents.

```mermaid
sequenceDiagram
    participant User
    participant Editor
    participant API
    participant DB
    participant LLM

    User->>Editor: Select text + action (e.g. "more emotional")
    Editor->>API: POST /inline-edit { selection, action }
    API->>DB: Load book tone + style_profile summary
    API->>LLM: GPT-4o: rewrite selection with action
    LLM-->>API: replacement text
    API-->>Editor: replacement text
    Editor-->>User: Replace selection
    API->>DB: Optional: log generation_run
```

---

## 6. Continuity agent (async, after chapter save)

Runs in Inngest after a chapter is finalized. Reads last continuity + character_state + new chapter summary; writes updated artifacts for the next chapter.

```mermaid
flowchart LR
    subgraph trigger [Trigger]
        Save[Chapter N saved]
        Save --> Inngest[Inngest: updateContinuity]
    end

    subgraph continuity [Continuity Agent - GPT-4o-mini]
        In[book_spec + last continuity + last character_state + chapter N summary]
        Out[continuity_summary, character_state, chapter_memory for N]
    end

    Inngest --> Load[Load from DB]
    Load --> In
    In --> LLM[OpenAI]
    LLM --> Out
    Out --> Store[Store artifacts in DB]
    Store -.->|used by| NextGen[Next chapter generation]
```

---

## 7. Data model (core entities and artifacts)

```mermaid
erDiagram
    users ||--o{ books : has
    books ||--o| book_specs : has
    books ||--o{ characters : has
    books ||--o{ inspiration_sources : has
    books ||--o{ style_profiles : has
    books ||--o{ chapters : has
    books ||--o{ artifacts : has
    inspiration_sources ||--o| style_profiles : produces
    chapters ||--o{ chapter_versions : has
    chapters ||--o{ generation_runs : has
    chapters ||--o{ artifacts : "chapter-scoped"
```

---

## 8. End-to-end flow summary

```mermaid
flowchart TB
    subgraph user [User actions]
        A1[Add book + story bible]
        A2[Add inspiration - paste/URL/search]
        A3[Write chapter outline]
        A4[Click Generate]
        A5[Inline edit selection]
    end

    subgraph inspiration [Inspiration pipeline]
        A2 --> I1[Inngest: fetch if URL/search]
        I1 --> I2[Inspiration agent: analyze]
        I2 --> I3[Save style_profile]
    end

    subgraph generate [Generate chapter]
        A4 --> G1[Orchestrator: load spec, style, artifacts]
        G1 --> G2[Planner: outline -> chapter_plan]
        G2 --> G3[Writer: plan + context -> streamed text]
        G3 --> G4[Save chapter + optional Continuity job]
    end

    subgraph edit [Inline edit]
        A5 --> E1[Single LLM call: selection + action]
        E1 --> E2[Replace selection]
    end

    I3 -.-> G3
    I3 -.-> E1
    G4 -.-> G1
```

---

*Diagrams align with the architecture plan: personal project, multi-agent workflow (Orchestrator, Continuity, Planner, Writer, Inline editor, Inspiration pipeline), cost-conscious use of GPT-4o-mini vs GPT-4o.*
