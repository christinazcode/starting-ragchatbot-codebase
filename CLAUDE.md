# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Architecture

This is a **Course Materials RAG (Retrieval-Augmented Generation) System** built with FastAPI, ChromaDB, and Anthropic Claude. The system processes course documents into searchable chunks and provides intelligent Q&A capabilities with source citations.

### Key Components & Flow

**Data Processing Pipeline:**
- `document_processor.py`: Parses structured course documents (Title/Link/Instructor → Lessons) into semantically meaningful chunks with overlap
- `vector_store.py`: ChromaDB wrapper managing two collections: `course_catalog` (metadata) and `course_content` (chunks)
- Documents auto-load from `docs/` folder on server startup

**Query Processing Architecture:**
1. `app.py`: FastAPI endpoints (`/api/query`, `/api/courses`) 
2. `rag_system.py`: Main orchestrator coordinating all components
3. `ai_generator.py`: Claude API integration with tool-calling capabilities
4. `search_tools.py`: Tool manager with `CourseSearchTool` for semantic search
5. `session_manager.py`: Conversation history management (max 2 exchanges by default)

**Critical Design Patterns:**
- **Tool-based search**: Claude autonomously decides when to search vs. use general knowledge
- **Semantic course resolution**: Partial course names (e.g. "MCP") resolve to full titles via vector similarity
- **Context-enhanced chunking**: Chunks prefixed with "Course [title] Lesson [number] content:" for better retrieval
- **Source tracking**: Citations flow from search results → tool manager → UI

## Development Commands

**Quick Start:**
```bash
chmod +x run.sh
./run.sh
```

**Manual Setup:**
```bash
# Install dependencies
uv sync

# Create .env with ANTHROPIC_API_KEY
echo "ANTHROPIC_API_KEY=your_key_here" > .env

# Start development server
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Access Points:**
- Web Interface: http://localhost:8000
- API Docs: http://localhost:8000/docs

## Configuration (config.py)

**Model Settings:**
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2" (sentence-transformers)

**Processing Parameters:**
- `CHUNK_SIZE`: 800 characters (sentence-boundary chunking)
- `CHUNK_OVERLAP`: 100 characters
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation exchanges

**Storage:**
- `CHROMA_PATH`: "./chroma_db" (ChromaDB persistent storage)

## Document Format Requirements

Course documents must follow this structure for proper parsing:

```
Course Title: [title]
Course Link: [optional url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [optional url]
[lesson content...]

Lesson 1: Next Topic
[lesson content...]
```

**Supported formats:** PDF, DOCX, TXT files in `docs/` folder

## Key Implementation Notes

**Vector Store Collections:**
- `course_catalog`: Course metadata for semantic name resolution
- `course_content`: Processed chunks with course/lesson context

**AI System Prompt Behavior:**
- One search per query maximum
- No meta-commentary about search decisions
- Direct, educational responses with examples when helpful

**Session Management:**
- Sessions auto-created if not provided
- History limited to prevent context overflow
- Conversation state maintained across API calls

**Tool Execution Flow:**
1. Claude receives query with available tools
2. If course-specific content needed, calls `search_course_content` tool
3. Tool executes vector search with optional course/lesson filters
4. Claude synthesizes results into coherent response
5. Sources tracked and returned to frontend for citation display

## Frontend Integration

**API Contract:**
- Request: `{query: string, session_id?: string}`
- Response: `{answer: string, sources: string[], session_id: string}`

**JavaScript handles:**
- Session persistence across page reloads
- Markdown rendering of AI responses
- Collapsible source citations
- Loading states and error handling