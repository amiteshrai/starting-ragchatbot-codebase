# RAG Chatbot Query Flow Diagram

```mermaid
graph TD
    %% Frontend Layer
    A[👤 User Types Query] --> B[📝 script.js: sendMessage()]
    B --> C[🚫 Disable Input & Show Loading]
    C --> D[📤 POST /api/query<br/>{"query": "...", "session_id": "..."}]

    %% API Layer
    D --> E[🌐 FastAPI: app.py<br/>@app.post("/api/query")]
    E --> F{🔍 Session ID exists?}
    F -->|No| G[🆕 Create New Session]
    F -->|Yes| H[📋 Use Existing Session]
    G --> I[🤖 rag_system.query()]
    H --> I

    %% RAG System Orchestration
    I --> J[📜 Wrap query with prompt<br/>"Answer this question about course materials..."]
    J --> K[💭 Retrieve conversation history<br/>from session_manager]
    K --> L[🔧 Pass to AI Generator with tools]

    %% AI Generation
    L --> M[🧠 ai_generator.py<br/>Claude API Call]
    M --> N{🤔 Claude Decision:<br/>Use search tool?}

    %% Tool Execution Branch
    N -->|Yes| O[🔍 tool_manager.execute_tool()<br/>"search_course_content"]
    O --> P[🎯 CourseSearchTool.execute()]
    P --> Q[🗃️ vector_store.search()<br/>with filters]
    Q --> R[📊 ChromaDB Semantic Search<br/>using embeddings]
    R --> S[📋 Format results with context<br/>"[Course - Lesson X]"]
    S --> T[📤 Return search results to Claude]
    T --> U[✍️ Claude synthesizes final response]

    %% Direct Response Branch
    N -->|No| V[💡 Use general knowledge<br/>Direct response]

    %% Response Assembly
    U --> W[💾 session_manager.add_exchange()<br/>Store query + response]
    V --> W
    W --> X[📊 Extract sources from tool_manager]
    X --> Y[📦 Return QueryResponse<br/>{"answer": "...", "sources": [...], "session_id": "..."}]

    %% Frontend Response Handling
    Y --> Z[📱 script.js receives response]
    Z --> AA[🗑️ Remove loading animation]
    AA --> BB[📝 Render AI response with markdown]
    BB --> CC[📚 Display sources in collapsible section]
    CC --> DD[🔄 Re-enable input for next query]
    DD --> EE[✅ Ready for next interaction]

    %% Data Flow Annotations
    style A fill:#e1f5fe
    style M fill:#fff3e0
    style R fill:#f3e5f5
    style Y fill:#e8f5e8

    %% Component Groupings
    subgraph "🖥️ Frontend Layer"
        A
        B
        C
        D
        Z
        AA
        BB
        CC
        DD
        EE
    end

    subgraph "🌐 API Layer"
        E
        F
        G
        H
    end

    subgraph "🤖 RAG System"
        I
        J
        K
        L
        W
        X
        Y
    end

    subgraph "🧠 AI Generation"
        M
        N
        U
        V
    end

    subgraph "🔍 Search & Retrieval"
        O
        P
        Q
        R
        S
        T
    end
```

## Flow Explanation

### 🖥️ **Frontend Layer**

- **User Interaction**: Query input and UI state management
- **API Communication**: HTTP requests to backend
- **Response Rendering**: Markdown parsing and source display

### 🌐 **API Layer**

- **Request Validation**: Pydantic models for type safety
- **Session Management**: Create or retrieve conversation context
- **Error Handling**: HTTP status codes and exception management

### 🤖 **RAG System**

- **Query Orchestration**: Coordinates all components
- **Context Assembly**: Builds prompts with conversation history
- **Response Coordination**: Manages tool execution and final assembly

### 🧠 **AI Generation**

- **Smart Tool Usage**: Claude decides when to search vs. use knowledge
- **Tool Execution**: Handles search tool calls and result integration
- **Response Synthesis**: Combines search results into natural answers

### 🔍 **Search & Retrieval**

- **Semantic Search**: Vector similarity using sentence transformers
- **Context Filtering**: Course and lesson-specific searches
- **Result Formatting**: Structured output with source attribution

## Key Decision Points

1. **Session Creation**: New vs. existing conversation context
2. **Tool Usage**: Search course content vs. general knowledge response
3. **Search Scope**: Filtered by course/lesson or global search
4. **Response Type**: Tool-enhanced vs. direct AI knowledge

## Data Flow

- **Query**: User input → API → RAG System → AI
- **Search**: AI → Tools → Vector Store → ChromaDB
- **Response**: AI ← Tools ← Search Results
- **Output**: Response + Sources → API → Frontend → User
