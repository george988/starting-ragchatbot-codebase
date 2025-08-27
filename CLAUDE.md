# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Start the application:**
```bash
./run.sh
```

**Manual start (development mode):**
```bash
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Install dependencies:**
```bash
uv sync
```

**Environment setup:**
Create `.env` file in root with:
```
ANTHROPIC_API_KEY=your_api_key_here
```

## Architecture Overview

This is a RAG (Retrieval-Augmented Generation) system with a FastAPI backend and vanilla JS frontend. The system processes course documents into vector embeddings for semantic search and uses Claude API for response generation.

### Core Components

**RAGSystem** (`backend/rag_system.py`) - Main orchestrator that coordinates all components:
- Manages document processing pipeline
- Handles query processing with tool-based search
- Coordinates session management and conversation history

**Vector Storage Architecture:**
- **VectorStore** (`backend/vector_store.py`) - ChromaDB wrapper with separate collections for course metadata and content chunks
- **DocumentProcessor** (`backend/document_processor.py`) - Chunks documents and extracts course/lesson structure
- **Models** (`backend/models.py`) - Pydantic models: `Course`, `Lesson`, `CourseChunk`

**AI Generation:**
- **AIGenerator** (`backend/ai_generator.py`) - Claude API integration with tool calling support
- **Search Tools** (`backend/search_tools.py`) - Implements course search functionality as a tool for Claude
- Uses tool-based approach where Claude decides when to search vs answer from knowledge

**Session Management:**
- **SessionManager** (`backend/session_manager.py`) - In-memory conversation history storage
- Tracks user sessions and maintains context across queries

### Key Design Patterns

1. **Tool-Based Search**: Claude uses search tools rather than direct vector retrieval, allowing it to decide when to search vs answer from existing knowledge

2. **Dual Vector Collections**: 
   - Course metadata collection for high-level course information
   - Content chunks collection for detailed lesson content

3. **Document Processing Pipeline**: Text files → Course/Lesson extraction → Chunking → Vector embedding → ChromaDB storage

4. **Session-Based Context**: Conversation history maintained per session for contextual responses

### Configuration

All settings in `backend/config.py`:
- `CHUNK_SIZE: 800` - Text chunk size for vector storage
- `CHUNK_OVERLAP: 100` - Overlap between chunks
- `MAX_RESULTS: 5` - Vector search results limit
- `MAX_HISTORY: 2` - Conversation messages to remember
- `ANTHROPIC_MODEL: "claude-sonnet-4-20250514"`
- `EMBEDDING_MODEL: "all-MiniLM-L6-v2"`

### Data Flow

1. Documents in `/docs/` are processed on startup
2. User queries trigger RAGSystem.query()
3. Claude decides whether to use search tool or answer directly
4. Search tool queries ChromaDB vector collections
5. Results combined with conversation history for Claude response
6. Response returned with source attribution

### Frontend Integration

- Static files served from `/frontend/` directory
- API endpoints: `/api/query` for questions, `/api/courses` for statistics
- Real-time chat interface with markdown support
- Course analytics sidebar showing loaded courses