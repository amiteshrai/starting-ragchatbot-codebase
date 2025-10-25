# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Starting the Application

```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management

```bash
# Install/sync dependencies
uv sync

# Add new dependency
uv add <package-name>
```

### Environment Setup

- Create `.env` file in root with `ANTHROPIC_API_KEY=your_key_here`
- Application runs on `http://localhost:8000`
- API docs available at `http://localhost:8000/docs`

## Architecture Overview

### Core System Design

This is a **Retrieval-Augmented Generation (RAG) chatbot** built on a modular, tool-enabled architecture:

**Backend (`/backend/`)**: Python FastAPI server with RAG pipeline
**Frontend (`/frontend/`)**: Vanilla HTML/CSS/JavaScript web interface
**Documents (`/docs/`)**: Course materials in structured text format

### RAG Pipeline Flow

1. **Document Processing**: Structured course files → chunked text with metadata
2. **Vector Storage**: ChromaDB with sentence-transformer embeddings
3. **AI Generation**: Claude with tool-calling capability for semantic search
4. **Session Management**: Conversation context and history tracking

### Key Components

**RAGSystem (`rag_system.py`)**: Main orchestrator that coordinates all components

- Manages document ingestion from `/docs/` folder
- Coordinates query processing through AI → Tools → Vector Store
- Handles session management and conversation history

**DocumentProcessor (`document_processor.py`)**: Converts structured course files into searchable chunks

- Parses course metadata (title, instructor, link) from first 3 lines
- Segments content by lesson markers (`Lesson X: Title`)
- Creates overlapping text chunks with configurable size (800 chars, 100 overlap)
- Adds contextual prefixes: `"Course [title] Lesson [num] content: [chunk]"`

**VectorStore (`vector_store.py`)**: ChromaDB-based semantic search

- Stores course metadata and content chunks separately
- Uses `all-MiniLM-L6-v2` sentence transformer for embeddings
- Supports filtered search by course name and lesson number
- Returns `SearchResults` with documents, metadata, and distances

**AIGenerator (`ai_generator.py`)**: Claude API integration with tool support

- System prompt defines smart tool usage rules (search vs. general knowledge)
- Handles tool execution flow: Claude → Tools → Results → Final Response
- Temperature 0, max 800 tokens for consistent responses

**SearchTools (`search_tools.py`)**: Tool interface for Claude

- `CourseSearchTool`: Semantic search with course/lesson filtering
- `ToolManager`: Registers tools and manages execution
- Tracks sources for UI display

**SessionManager (`session_manager.py`)**: Conversation state management

- Creates unique session IDs for conversation continuity
- Maintains rolling history (configurable, default 2 exchanges)
- Formats conversation context for AI prompts

### Document Format

Course files must follow this structure:

```text
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: [lesson title]
Lesson Link: [optional lesson url]
[lesson content...]

Lesson 1: [next lesson title]
[content...]
```

### Configuration (`config.py`)

Key settings in `Config` class:

- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation exchanges
- `ANTHROPIC_MODEL`: claude-sonnet-4-20250514
- `EMBEDDING_MODEL`: all-MiniLM-L6-v2

### Frontend Integration

- Single-page application with real-time chat interface
- API communication via `/api/query` (POST) and `/api/courses` (GET)
- Session continuity through session_id parameter
- Markdown rendering for AI responses with collapsible source attribution

### Data Models (`models.py`)

**Course**: title, course_link, instructor, lessons[]
**Lesson**: lesson_number, title, lesson_link
**CourseChunk**: content, course_title, lesson_number, chunk_index

### Tool Decision Logic

Claude uses the search tool when:

- Questions about specific course content or detailed materials
- Course-specific questions requiring context retrieval

Claude uses general knowledge when:

- General educational questions
- Conceptual explanations not requiring course-specific content

### Adding New Courses

1. Create structured text file in `/docs/` following the required format
2. Restart application or use the API to trigger document reprocessing
3. System automatically prevents duplicate processing of existing courses

## Important Notes

- No test framework is currently implemented
- ChromaDB data persists in `./backend/chroma_db/`
- All course processing happens on application startup
- Session data is in-memory only (lost on restart)
- CORS enabled for all origins (development configuration)
