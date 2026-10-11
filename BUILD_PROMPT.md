# AI Orchestrator - Complete Build Prompt for Google AI Studio / Gemini Code Generation

**Copy and paste the entire content below into Google AI Studio, Gemini, or your preferred code generation tool to build the complete project.**

---

## **MASTER BUILD PROMPT**

Build a complete, production-ready, runnable Python project for a lightweight local AI orchestrator designed for a 16GB RAM Windows machine running local Ollama models.

### **Project Location**
- **Installation Path:** `C:\Users\Chris\Smackit\ai-orchestrator`
- **All files and directories must respect this path.**
- **Setup commands assume this directory structure.**

### **Core Requirements**

#### **1. Technology Stack**
- Python 3.11+
- **LangGraph** (preferred) or LangChain ReAct agent loop for autonomous reasoning
- Local Ollama OpenAI-compatible API: `http://localhost:11434/v1`
- **Default Model:** `dolphin-llama3:8b`
- **Fallback Model:** `qwen2.5-coder:3b`
- **Search:** Live web search via `duckduckgo_search` (zero API key required)
- **Memory:** SQLite for structured data + **ChromaDB for semantic vector memory**
- **Backend:** FastAPI with simple modern HTML/JS frontend
- **Alternative UI:** Rich CLI with advanced terminal rendering
- **No Docker. No cloud dependency. Local-first design.**

---

### **2. Architecture & Core Features**

#### **A. Dual-Tier Memory System (Key Upgrade #1)**
1. **SQLite (Structural Memory)**
   - Conversation history
   - Session metadata & summaries
   - User profile & preferences
   - Tool execution logs
   - Session timestamps & metadata

2. **ChromaDB (Semantic Memory - Upgrade #1)**
   - Embeddings of past conversation snippets
   - Session summaries for semantic retrieval
   - User context and conversation themes
   - Enables "recall what we discussed about X" functionality
   - Auto-populated from SQLite summaries

#### **B. Context Window Management (Key Upgrade #2)**
- Implement **sliding-window context** to handle long conversations on 8B models
- Auto-summarize conversations older than N turns
- Dynamically inject only **semantically relevant** past context using ChromaDB retriever
- Maintain a compact "session summary" always included in LLM context
- Token budget management: track and limit input tokens per message

#### **C. Tool Execution Sandboxing (Upgrade #3)**
- Limit tool output to max 500 tokens (configurable)
- Implement timeouts (default 10 seconds per tool call)
- Rate limiting: prevent tool spamming (e.g., max 5 searches per minute)
- Structured error handling with fallback outputs
- Log all tool calls with execution time and token usage

#### **D. Structured Logging & Observability (Upgrade #4)**
- Every agent step logged: input, reasoning, tool selection, output
- Debug mode that visualizes decision trees
- Metrics: avg response time, tool call frequency, error rates
- JSON-structured logs for easy parsing
- Simple metrics dashboard endpoint: `GET /metrics`

#### **E. Intelligent Model Fallback Logic (Upgrade #5)**
- Try dolphin-llama3:8b first
- If response time exceeds 15 seconds or model error occurs, auto-switch to qwen2.5-coder:3b
- Track per-model performance (latency, accuracy on certain task types)
- User override capability via config and runtime commands
- Log all fallback events

---

### **3. Project Directory Structure**

```
C:\Users\Chris\Smackit\ai-orchestrator\
├── README.md
├── BUILD_PROMPT.md
├── requirements.txt
├── .env.example
├── .gitignore
├── main.py
├── app/
│   ├── __init__.py
│   ├── main.py (or api.py)
│   ├── routes.py
│   └── frontend.html
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── models.py
│   └── defaults.py
├── src/
│   ├── __init__.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── orchestrator.py (LangGraph graph + ReAct loop)
│   │   ├── tools.py (search, file ops, calculator, summarizer)
│   │   ├── prompts.py (agent system prompts)
│   │   ├── memory.py (memory management and retrieval)
│   │   ├── user_profile.py (user preferences)
│   │   ├── summarizer.py (conversation summarization)
│   │   └── model_selector.py (model fallback logic)
│   ├── db/
│   │   ├── __init__.py
│   │   ├── sqlite_store.py (SQLite database)
│   │   └── chroma_store.py (ChromaDB vector memory)
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── cli.py (Rich CLI interface)
│   │   └── web_app.py (FastAPI web interface)
│   └── utils/
│       ├── __init__.py
│       ├── logger.py (structured logging)
│       ├── healthcheck.py (startup verification)
│       ├── config_loader.py (env/config handling)
│       └── metrics.py (performance tracking)
├── data/
│   ├── user_profiles/ (JSON files per user)
│   ├── sessions/ (session summaries)
│   └── memory.db (SQLite database)
├── memory/
│   ├── chroma_db/ (ChromaDB vector store)
│   └── embeddings/ (embedding cache)
├── logs/
│   ├── app.log
│   ├── agent.log
│   ├── errors.log
│   └── metrics.log
└── templates/
    └── chat.html (simple frontend for FastAPI)
```

---

### **4. Core Functional Requirements**

#### **A. ReAct Agent Orchestration**
- Multi-step autonomous reasoning loop using LangGraph
- Agent states: THINKING → TOOL_SELECTION → TOOL_EXECUTION → RESPONSE → MEMORY_UPDATE
- Max agent steps: 10 per user message (configurable, prevents infinite loops)
- Structured tool input/output with JSON schema validation
- Graceful error recovery and retry logic

#### **B. Tools & External Integrations**

**Tool 1: web_search**
- Input: query string
- Output: JSON with list of results
  ```json
  {
    "results": [
      {"title": "...", "snippet": "...", "link": "...", "source": "..."},
      ...
    ],
    "total_results": N,
    "query_time_ms": 123
  }
  ```
- Rate limit: 5 searches per minute per session
- Timeout: 10 seconds
- Max output: 1500 characters
- Fallback: return empty results on error

**Tool 2: file_read**
- Input: filepath
- Output: file contents (truncated to 5000 chars if large)
- Security: only allow reading from project directory
- Error handling: return clear error message if file not found

**Tool 3: file_write**
- Input: filepath, content
- Output: success/failure with filepath
- Security: only allow writing to data/ directory
- Max file size: 1MB

**Tool 4: calculate**
- Input: mathematical expression
- Output: result with calculation breakdown
- Error handling: catch malformed expressions gracefully

**Tool 5: summarize_text**
- Input: text content
- Output: condensed summary (50-100 words)
- Uses LLM to generate coherent summary

#### **C. Memory Systems**

**User Profile Structure (JSON)**
```json
{
  "user_id": "default",
  "created_at": "2025-01-01T00:00:00Z",
  "preferences": {
    "preferred_model": "dolphin-llama3:8b",
    "response_length": "concise",
    "coding_style": "type-safe, docstrings required",
    "favorite_languages": ["Python", "JavaScript"],
    "environment": {
      "os": "Windows",
      "ram_gb": 16,
      "workspace": "C:/Users/Chris/Smackit"
    }
  },
  "memory": {
    "retention_days": 90,
    "summarize_every_n_turns": 10,
    "auto_recall": true
  },
  "search_preferences": {
    "auto_search": true,
    "max_sources": 3,
    "search_timeout_sec": 10
  },
  "llm_parameters": {
    "temperature": 0.7,
    "top_p": 0.9,
    "max_tokens": 2048
  }
}
```

**Session Summarization Logic**
- After every N turns (default: 10), automatically summarize the conversation
- Summary includes: key topics, decisions, outcomes, user preferences mentioned
- Store summary in SQLite + ChromaDB
- On next turn, inject most relevant historical summaries based on semantic similarity
- Prune old sessions after X days (default: 90) to manage storage

#### **D. Persistent Memory Across Restarts**
- Auto-load user profile on startup
- Restore last active session context
- ChromaDB maintains embeddings for semantic recall
- All preferences and memories survive application restart
- Export/import session data as JSON backup

---

### **5. Implementation Details**

#### **A. Model Configuration Module** (`config/models.py`)
```python
OLLAMA_BASE_URL = "http://localhost:11434/v1"
OLLAMA_API_KEY = "ollama"

MODELS = {
    "primary": {
        "name": "dolphin-llama3:8b",
        "context_window": 8192,
        "timeout_sec": 15,
        "preferred": True
    },
    "fallback": {
        "name": "qwen2.5-coder:3b",
        "context_window": 4096,
        "timeout_sec": 10,
        "preferred": False
    }
}

AGENT_CONFIG = {
    "max_steps": 10,
    "max_reasoning_tokens": 3000,
    "temperature": 0.7,
    "top_p": 0.9
}

MEMORY_CONFIG = {
    "summarize_every_n_turns": 10,
    "retention_days": 90,
    "max_memory_tokens": 2000
}

TOOL_CONFIG = {
    "search_rate_limit": 5,  # per minute
    "search_timeout": 10,
    "max_tool_output": 500,
    "tool_timeout": 10
}
```

#### **B. Database Schema** (`src/db/sqlite_store.py`)

**Tables:**
1. `users` - user profiles
2. `sessions` - session metadata
3. `messages` - individual messages (user + assistant)
4. `summaries` - session summaries
5. `tool_logs` - tool execution history
6. `preferences` - user preferences (denormalized)

#### **C. Health Check on Startup** (`src/utils/healthcheck.py`)
- Verify Ollama server is running at `http://localhost:11434/v1`
- Verify default model (`dolphin-llama3:8b`) is pulled and available
- Verify fallback model (`qwen2.5-coder:3b`) is available
- Verify SQLite database is accessible and schema is initialized
- Verify ChromaDB vector store is ready
- Verify write access to logs/ and data/ directories
- Report status: READY, DEGRADED (fallback), or CRITICAL (cannot start)
- Log all issues; suggest corrective actions to user

#### **D. Model Fallback Logic** (`src/agent/model_selector.py`)
```python
def select_model(performance_history, user_preference):
    """
    1. Use user preference if set
    2. Otherwise, use model with best recent performance
    3. If primary model latency > 15s, switch to fallback
    4. If primary model returns error, switch to fallback
    5. Track all switches in metrics
    """
```

#### **E. API Endpoints** (FastAPI backend)
- `POST /chat` - Send message, get response
- `GET /history` - Get conversation history
- `GET /sessions` - List all past sessions
- `POST /sessions/{session_id}/load` - Load a specific session
- `GET /profile` - Get user profile
- `POST /profile/update` - Update user preferences
- `GET /health` - Health check status
- `GET /metrics` - Performance metrics (avg latency, tool calls, errors)
- `POST /memory/recall` - Semantic memory search
- `POST /memory/export` - Export all data as JSON
- `POST /memory/prune` - Clean up old sessions

#### **F. UI Options**

**Option 1: FastAPI + Simple HTML Frontend** (Recommended)
- Single-page app with chat interface
- Real-time streaming responses
- Session switcher sidebar
- User preferences panel
- Memory/metrics dashboard
- Clean, modern design using Bootstrap/Tailwind

**Option 2: Rich CLI Interface**
- Terminal-based chat with color syntax
- Auto-formatting of tool outputs
- Session history inline
- User preferences menu
- Metrics display
- Keyboard shortcuts for quick actions

#### **G. Logging & Metrics**
- Structured logging: JSON format for parsing
- Log levels: DEBUG, INFO, WARNING, ERROR
- Separate log files:
  - `app.log` - general application events
  - `agent.log` - agent reasoning and decisions
  - `errors.log` - all errors and exceptions
  - `metrics.log` - performance metrics
- Metrics tracked:
  - Response time (P50, P95, P99)
  - Tool call frequency
  - Model used per session
  - Error rate
  - Token usage (input + output)
  - Cache hit rate (memory retrieval)

---

### **6. Generated Files Required**

#### **Configuration & Setup**
- `README.md` - Complete setup and usage guide for Windows
- `requirements.txt` - All Python dependencies with pinned versions
- `.env.example` - Environment variables template
- `.gitignore` - Standard Python .gitignore
- `BUILD_PROMPT.md` - This file

#### **Core Application**
- `main.py` - Entry point for CLI or web app
- `app/main.py` - FastAPI application factory
- `app/routes.py` - All API endpoints
- `app/frontend.html` - Simple HTML/JS frontend

#### **Configuration Modules**
- `config/settings.py` - Load settings from environment
- `config/models.py` - Model and agent configuration
- `config/defaults.py` - Default values for all settings

#### **Agent & Orchestration**
- `src/agent/orchestrator.py` - LangGraph ReAct loop
- `src/agent/tools.py` - Tool definitions and wrappers
- `src/agent/prompts.py` - System prompts and few-shot examples
- `src/agent/memory.py` - Memory retrieval and management
- `src/agent/user_profile.py` - User profile manager
- `src/agent/summarizer.py` - Conversation summarization
- `src/agent/model_selector.py` - Model selection and fallback

#### **Database & Memory**
- `src/db/sqlite_store.py` - SQLite wrapper and schema
- `src/db/chroma_store.py` - ChromaDB vector store wrapper

#### **UI & Interface**
- `src/ui/cli.py` - Rich CLI interface
- `src/ui/web_app.py` - FastAPI web app

#### **Utilities**
- `src/utils/logger.py` - Structured logging setup
- `src/utils/healthcheck.py` - Startup health verification
- `src/utils/config_loader.py` - Environment and config loading
- `src/utils/metrics.py` - Metrics collection and reporting

---

### **7. Code Quality Standards**

- **Style:** PEP 8 compliant
- **Typing:** Use type hints throughout
- **Documentation:** Docstrings for all functions and classes
- **Error Handling:** Comprehensive try/except with specific error messages
- **Testing:** Include example usage and simple integration tests
- **Performance:** Optimize for 16GB RAM Windows machines
- **Security:** Validate all inputs, sandbox file operations, rate limit external calls
- **Modularity:** Easily swappable components (e.g., swap CLI for FastAPI)

---

### **8. Startup & Usage Commands**

For **Windows** (from `C:\Users\Chris\Smackit\ai-orchestrator`):

```bash
# Create virtual environment
python -m venv .venv

# Activate virtual environment
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment (copy and edit .env)
copy .env.example .env
# Edit .env with your preferences

# Initialize database (first run only)
python -c "from src.db.sqlite_store import init_db; init_db()"

# Run as FastAPI web app
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000

# OR run as CLI
python main.py --mode cli

# Check health status
python -c "from src.utils.healthcheck import run_healthcheck; run_healthcheck()"
```

---

### **9. Example Session Flow**

```
User: "Search the web for recent advances in local LLMs in 2025."
Agent: 
  - THINKING: "User wants web search. I'll use web_search tool."
  - TOOL_SELECTION: "web_search"
  - TOOL_EXECUTION: [calls DuckDuckGo, gets 3 results]
  - RESPONSE: "Based on recent searches, here are the top advances..."
  - MEMORY_UPDATE: [stores query and response in SQLite + ChromaDB]

User: "Remember I prefer concise Python examples."
Agent:
  - THINKING: "User is setting a preference. I'll update their profile."
  - TOOL_SELECTION: "update_user_profile"
  - TOOL_EXECUTION: [writes to user_profile.json]
  - RESPONSE: "Got it! I'll provide concise Python examples going forward."
  - MEMORY_UPDATE: [logs preference update]

User: "What did we discuss about local AI last month?"
Agent:
  - THINKING: "User is asking about past conversations. I'll use semantic recall."
  - TOOL_SELECTION: "semantic_memory_search"
  - TOOL_EXECUTION: [ChromaDB query for 'local AI' embeddings, returns 2 relevant summaries]
  - RESPONSE: "Last month we discussed... [relevant context]"
  - MEMORY_UPDATE: [logs recall event]
```

---

### **10. Five Key Upgrades Included**

1. **Dual-Tier Memory (SQLite + ChromaDB)** - Enables semantic memory retrieval for "what did we discuss" queries
2. **Context Window Management** - Auto-summarization + sliding-window to handle long conversations on 8B models
3. **Tool Execution Sandboxing** - Rate limiting, timeouts, output limits prevent resource abuse
4. **Structured Logging & Observability** - Track every agent step, metrics, and decisions
5. **Intelligent Model Fallback** - Auto-switch to faster model on latency/error, track performance per task type

---

### **11. Important Implementation Notes**

- **Lightweight & Local:** No cloud calls except web search. Everything runs locally on the Windows machine.
- **16GB RAM Optimized:** Model context windows managed carefully. Summarization prevents memory overflow.
- **Production-Ready:** Comprehensive error handling, logging, and graceful degradation.
- **Modular:** Easy to swap UI (CLI ↔ FastAPI), add new tools, or extend memory systems.
- **User-Centric:** Preferences, memory, and profiles persist across restarts.
- **Realistic Code:** Not pseudocode. All implementations should be complete and runnable.

---

### **12. Deliverables Checklist**

- ✅ Complete project structure at `C:\Users\Chris\Smackit\ai-orchestrator`
- ✅ All Python source files with full implementation
- ✅ requirements.txt with pinned versions
- ✅ .env.example with all required variables
- ✅ README.md with Windows setup instructions
- ✅ SQLite schema and initialization
- ✅ ChromaDB setup and vector embedding logic
- ✅ LangGraph ReAct agent orchestration
- ✅ All tools (search, file, calculate, summarize)
- ✅ User profile manager with JSON persistence
- ✅ Session summarization with token budgeting
- ✅ Model fallback logic with performance tracking
- ✅ FastAPI backend with simple HTML frontend
- ✅ Rich CLI alternative
- ✅ Health checks on startup
- ✅ Structured logging and metrics
- ✅ Example usage flows and test data
- ✅ Comments and docstrings throughout

---

### **13. Post-Generation Next Steps**

After the project is generated:

1. **Install dependencies:** `pip install -r requirements.txt`
2. **Verify Ollama:** Ensure `http://localhost:11434/v1` is running with models pulled
3. **Initialize database:** Run initialization script
4. **Set up .env:** Copy and edit `.env.example` to `.env`
5. **Run health check:** `python -c "from src.utils.healthcheck import run_healthcheck; run_healthcheck()"`
6. **Start the app:** `uvicorn app.main:app --reload` or `python main.py --mode cli`
7. **Test chat:** Send a test message, verify all systems work
8. **Try web search:** Ask agent to search the web
9. **Check memory:** Verify session data is saved
10. **Monitor metrics:** Check `/metrics` endpoint or logs

---

### **FINAL INSTRUCTION TO CODE GENERATOR**

**Please generate the complete, fully-implemented, production-ready Python project as described above. Every file should be complete code, not stubs or placeholders. The project should be runnable immediately after `pip install -r requirements.txt` with only minor config (setting up .env and ensuring Ollama is running). Include realistic implementations for all major components: agent orchestration, tool wrappers, memory systems, database schema, API endpoints, and UI. Make the code clean, well-commented, type-hinted, and suitable for a real local AI assistant project running on Windows.**

**Project path: `C:\Users\Chris\Smackit\ai-orchestrator`**

**Include all files listed in section 6 (Generated Files Required).**

**Make it a solid, working starting point that someone can clone, set up, and start using immediately.**

---

END OF BUILD PROMPT
