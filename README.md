# 🤖 AI Agent API

FastAPI AI Agent dengan OpenRouter (LiteLLM), Sessions, Streaming, dan 3 Tools.

## 📋 Features

- **🧠 AI Agent** - LiteLLM integration dengan OpenRouter (Llama 3.1)
- **💬 Chat Sessions** - Conversation history tersimpan di database
- **⚡ Streaming** - Real-time streaming responses
- **🛠️ 3 Tools:**
  - Text Analyzer - analisis text (word count, character count, dll)
  - Calculator - evaluasi math expressions
  - Word Counter - hitung kata dengan regex
- **📊 Scalar UI** - Modern API documentation (bukan Swagger)
- **🗄️ Database** - SQLAlchemy + Alembic migrations
- **🚀 uv** - Modern Python package manager

## 🎯 Quick Start

### 1. Install Dependencies

```bash
make install
# atau: uv sync
```

### 2. Setup Environment

File `.env` udah ada dengan OpenRouter API key. Kalo mau ganti:

```bash
# Edit .env
OPENROUTER_API_KEY=your-key-here
```

### 3. Run Migrations

```bash
make migrate
# atau: uv run alembic upgrade head
```

### 4. Start Server

```bash
make dev
# atau: uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Server jalan di: `http://localhost:8000`

## 🧪 Testing / Cara Pakai

### Option 1: Scalar UI (Recommended)

1. Buka browser: `http://localhost:8000/docs`
2. Kamu bakal liat Scalar UI yang keren
3. Test semua endpoints langsung dari situ

### Option 2: HTML Test Interface

#### A. Tool Testing (`test.html`)

1. Buka `test.html` di browser
2. Test 3 tools:
   - **Text Analyzer**: Masukin text, klik "Analyze Text"
   - **Calculator**: Masukin expression (contoh: `2 + 2 * 3`), klik "Calculate"
   - **Word Counter**: Masukin text dan kata yang mau di-count, klik "Count Words"

#### B. AI Chat (`chat.html`)

1. Buka `chat.html` di browser
2. Session ID otomatis di-create
3. Ketik message, ada 2 pilihan:
   - **Send** - Normal response (tunggu sampe selesai)
   - **Send (Stream)** - Streaming response (real-time)

Contoh chat:
```
You: Calculate 15 * 23 + 100
Agent: *uses calculator tool* The result is 445!

You: Analyze this text: "Hello world"
Agent: *uses text_analyzer* Your text has 11 characters and 2 words.
```

### Option 3: cURL / API Testing

#### Create Session
```bash
curl -X POST http://localhost:8000/api/agent/session/create
# Response: {"session_id": "uuid-here"}
```

#### Chat (Normal)
```bash
curl -X POST http://localhost:8000/api/agent/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Calculate 10 + 20",
    "session_id": "your-session-id"
  }'
```

#### Chat (Streaming)
```bash
curl -X POST http://localhost:8000/api/agent/chat/stream \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Tell me a story",
    "session_id": "your-session-id"
  }'
```

#### Execute Tool Directly
```bash
# Text Analyzer
curl -X POST http://localhost:8000/api/tools/execute \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "analyze_text",
    "parameters": {"text": "Hello world!"}
  }'

# Calculator
curl -X POST http://localhost:8000/api/tools/execute \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "calculate",
    "parameters": {"expression": "2 + 2 * 3"}
  }'

# Word Counter
curl -X POST http://localhost:8000/api/tools/execute \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "count_words",
    "parameters": {"text": "The fox jumps. The fox runs.", "word": "fox"}
  }'
```

#### Get Session History
```bash
curl http://localhost:8000/api/agent/session/{session_id}/history
```

## 📁 Project Structure

```
assigment5/
├── app/
│   ├── api/routes/
│   │   ├── agent.py          # Agent endpoints (chat, streaming, sessions)
│   │   └── tools.py          # Tool endpoints
│   ├── core/
│   │   └── config.py         # Settings & env config
│   ├── db/
│   │   ├── database.py       # SQLAlchemy setup
│   │   └── models.py         # Models (ToolExecution, ChatSession, ChatMessage)
│   ├── schemas/
│   │   ├── agent.py          # Pydantic schemas untuk agent
│   │   └── tool.py           # Pydantic schemas untuk tools
│   ├── services/
│   │   ├── agent_service.py  # Agent logic
│   │   ├── llm_service.py    # LiteLLM integration
│   │   ├── session_service.py # Session management
│   │   └── tool_service.py   # Tool management
│   ├── tools/
│   │   ├── calculator.py     # Calculator tool
│   │   ├── text_analyzer.py  # Text analyzer tool
│   │   └── word_counter.py   # Word counter tool
│   └── main.py               # FastAPI app
├── alembic/                  # Database migrations
├── test.html                 # Tool testing interface
├── chat.html                 # AI chat interface
├── Makefile                  # Make commands
├── .env                      # Environment variables
└── pyproject.toml            # Dependencies (uv)
```

## 🎯 API Endpoints

### Core
- `GET /` - API info
- `GET /health` - Health check
- `GET /docs` - Scalar API documentation

### Tools
- `GET /api/tools/` - List available tools
- `POST /api/tools/execute` - Execute a tool
- `GET /api/tools/history` - Tool execution history

### Agent
- `POST /api/agent/chat` - Chat with AI (normal)
- `POST /api/agent/chat/stream` - Chat with AI (streaming)
- `POST /api/agent/query` - Process query with tools
- `POST /api/agent/session/create` - Create new session
- `GET /api/agent/session/{id}/history` - Get session history
- `GET /api/agent/status` - Agent status

## 🔧 Cara Kerja

### 1. Chat Flow (Normal)
```
User → POST /api/agent/chat
  ↓
Agent Service:
  - Load conversation history dari session
  - Send ke LiteLLM (OpenRouter)
  - LLM decide: perlu tool atau ngga?
  - Kalo perlu tool → execute tool → format response
  - Save message ke database
  ↓
Response ke user
```

### 2. Chat Flow (Streaming)
```
User → POST /api/agent/chat/stream
  ↓
Agent Service:
  - Load conversation history
  - Stream dari LiteLLM
  - Kirim chunks via Server-Sent Events (SSE)
  - Save full response ke database
  ↓
Stream chunks ke user (real-time)
```

### 3. Tool Execution
```
User → POST /api/tools/execute
  ↓
Tool Service:
  - Validate tool name
  - Execute tool function
  - Log ke database (ToolExecution)
  ↓
Return result
```

### 4. Session Management
```
- Session dibuat otomatis atau manual
- Setiap message disimpan (ChatMessage)
- Conversation history di-load untuk context
- Session bisa di-retrieve kapan aja
```

## 🛠️ Makefile Commands

```bash
make install       # Install dependencies
make dev          # Run development server (auto-reload)
make run          # Run production server
make migrate      # Run database migrations
make migrate-create msg="message"  # Create new migration
make migrate-down # Rollback migration
make clean        # Clean database & cache
make help         # Show all commands
```

## 🗄️ Database

SQLite database: `database.db`

Tables:
- `tool_executions` - Log semua tool executions
- `chat_sessions` - Chat sessions
- `chat_messages` - Chat messages (user & assistant)

## ⚙️ Configuration

Edit `.env`:

```env
DATABASE_URL=sqlite:///./database.db
DEBUG=True
API_HOST=0.0.0.0
API_PORT=8000
OPENROUTER_API_KEY=your-key-here
```

## 🤖 How It Works - Detail

### LiteLLM Integration
- Library universal untuk berbagai LLM providers
- Support OpenRouter, OpenAI, Anthropic, dll
- Async & streaming support
- Automatic retry & error handling

### Session Context
- Setiap chat punya session ID
- Conversation history otomatis di-load (last 10 messages)
- LLM punya context dari chat sebelumnya
- Bisa continue conversation kapan aja

### Tool Calling
- LLM decide kapan perlu pake tool
- Format response sebagai JSON
- Agent parse & execute tool
- Result di-format jadi natural language

### Streaming
- Server-Sent Events (SSE)
- Real-time chunks dari LLM
- Better UX untuk long responses
- Full response tetep disimpan ke DB

## 📝 Example Scenarios

### Scenario 1: Simple Chat
```
User: "Hello!"
Agent: "Hi! How can I help you today?"
```

### Scenario 2: Tool Usage
```
User: "Calculate 15 * 23"
Agent: *detects need for calculator*
      *executes calculate tool*
      "The result is 345!"
```

### Scenario 3: Context Awareness
```
User: "Calculate 10 + 20"
Agent: "The result is 30!"

User: "Now multiply that by 2"
Agent: *remembers previous result from context*
      "60!"
```

## 🚀 Production Tips

1. Ganti `DEBUG=False` di `.env`
2. Use proper database (PostgreSQL)
3. Add rate limiting
4. Add authentication
5. Use `make run` instead of `make dev`

---

Built with FastAPI, LiteLLM, SQLAlchemy, and uv 🚀
