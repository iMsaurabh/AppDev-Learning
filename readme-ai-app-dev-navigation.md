# AI Agent — Component Relationships & Cheatsheet
## 30-Day Web Development Learning Path

---

## The Big Picture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        YOUR AI AGENT SYSTEM                         │
│                                                                     │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐  │
│  │   Frontend  │────►│   Backend   │────►│     AI Layer        │  │
│  │  (Browser)  │◄────│  (Express)  │◄────│  (Groq + Tools)     │  │
│  └─────────────┘     └──────┬──────┘     └─────────────────────┘  │
│                             │                                       │
│                    ┌────────┴────────┐                             │
│                    │                 │                             │
│             ┌──────┴──────┐  ┌──────┴──────┐                     │
│             │  Vector DB  │  │  Sessions   │                     │
│             │ (Embeddings)│  │   (Memory)  │                     │
│             └─────────────┘  └─────────────┘                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Every Component Explained

### 1. Frontend (Browser)
```
What:     The UI users interact with
Where:    Runs in user's browser
Built:    HTML/CSS/JS or React
Talks to: Backend via HTTP requests
Status:   Not built yet (Days 28-30)
```

### 2. Backend (Express — server.js)
```
What:     HTTP server that handles requests
Where:    Runs on your machine or Render
Built:    Node.js + Express
Talks to: Frontend, Agent, Session Store
Commands:
  Start:  node server.js
  Test:   curl http://localhost:3000/health
```

### 3. Agent (agent.js)
```
What:     Brain of the system
          Decides which tools to call
          Manages conversation loop
Where:    Called by server.js
Built:    Node.js + Groq SDK
Talks to: Groq API, all tools
```

### 4. LLM API (Groq)
```
What:     The actual AI model
          Understands language
          Decides tool calls
          Forms final answers
Where:    Groq's servers (cloud)
Auth:     GROQ_API_KEY in .env
Cost:     Free tier available
```

### 5. Tools (tools/*.js)
```
What:     Real world capabilities
          Each is a JS function
          Agent calls these
Where:    Your code, runs locally
          or on Render

Your tools:
  calculator.js  → eval() math
  weather.js     → OpenWeatherMap API
  notes.js       → fs read/write
  rag.js         → vector DB search
```

### 6. External APIs
```
What:     Third party services your tools call
Examples:
  OpenWeatherMap → real weather data
  Groq           → LLM responses
Auth:     API keys stored in .env
```

### 7. Vector DB (vector-db/)
```
What:     Stores document embeddings
          Enables semantic search
          Powers RAG system
Where:    Local folder (vectra library)
Built by: node ingest.js
Used by:  tools/rag.js
Format:   Binary files (don't edit manually)
```

### 8. Embeddings (@xenova/transformers)
```
What:     Converts text to numbers
          Numbers capture meaning
          Similar text = similar numbers
Where:    Runs locally (no API needed)
Used in:  ingest.js and tools/rag.js
Model:    Xenova/all-MiniLM-L6-v2 (~25MB)
          Downloads automatically on first run
```

### 9. Sessions (sessions.json)
```
What:     Stores conversation history
          One entry per sessionId
          Persists across server restarts
Where:    Local file (sessionStore.js)
Format:   JSON
Gitignore: Yes — contains user data
```

### 10. Docker
```
What:     Packages app into container
          Runs identically everywhere
          No "works on my machine"
Files:    Dockerfile, .dockerignore
Commands: docker build, docker run
Used for: Local testing, deployment
```

### 11. Render
```
What:     Cloud hosting platform
          Runs your Docker container
          Gives you a public URL
          Auto-deploys from GitHub
Free tier: Spins down after 15min inactivity
Dashboard: render.com
```

### 12. GitHub
```
What:     Code version control
          Source of truth for your code
          Triggers Render auto-deploys
Files:    Everything except .gitignore entries
```

---

## How Components Relate

### Development Flow
```
You write code
      ↓
Test locally (node server.js)
      ↓
git push to GitHub
      ↓
Render auto-deploys
      ↓
Live on internet
```

### Request Flow (every user message)
```
User types message
      ↓
Frontend sends HTTP POST /chat
      ↓
server.js receives request
  ├── validates input (express-validator)
  ├── checks rate limit (express-rate-limit)
  ├── loads session history (sessions.json)
  └── calls runAgent(messages)
            ↓
        agent.js
          ├── sends messages + tools to Groq API
          └── Groq responds with tool_call
                    ↓
                tool router
                  ├── calculator? → eval()
                  ├── weather?    → OpenWeatherMap API
                  ├── notes?      → fs read/write
                  └── rag?        → vector-db search
                            ↓
                        result sent back to Groq
                            ↓
                        Groq forms final answer
                            ↓
                    server.js saves to session
                            ↓
                    response sent to frontend
```

### RAG Ingestion Flow (run once per document update)
```
document.txt
      ↓
ingest.js reads file
      ↓
chunkText() splits into pieces
      ↓
getEmbedding() converts each chunk to numbers
  (uses @xenova/transformers locally)
      ↓
vector-db/ stores embeddings
      ↓
Ready for queries
```

### RAG Query Flow (every relevant user question)
```
User question
      ↓
tools/rag.js called by agent
      ↓
getEmbedding() converts question to numbers
      ↓
vector-db/ finds similar chunks
      ↓
relevant text returned to agent
      ↓
agent sends to Groq with question
      ↓
Groq answers from your document
```

---

## File Relationships

```
day24/
│
├── server.js          ← HTTP entry point
│     imports ──────────────────────────────┐
│                                           │
├── agent.js           ← AI brain           │
│     imports ──────────────────────┐       │
│                                   │       │
├── tools/                          │       │
│   ├── calculator.js ◄─────────────┤       │
│   ├── weather.js    ◄─────────────┤       │
│   ├── notes.js      ◄─────────────┤       │
│   └── rag.js        ◄─────────────┘       │
│         uses                              │
│         vector-db/                        │
│                                           │
├── services/                               │
│   └── sessionStore.js ◄──────────────────┘
│
├── middleware/
│   ├── validate.js    ← input validation
│   └── rateLimiter.js ← rate limiting
│
├── utils/
│   └── logger.js      ← logging
│
├── config.js          ← constants
├── ingest.js          ← RAG ingestion (run manually)
├── document.txt       ← knowledge base
│
├── vector-db/         ← generated by ingest.js
├── sessions.json      ← generated at runtime
├── logs/              ← generated at runtime
│
├── Dockerfile         ← Docker build instructions
├── .dockerignore      ← what Docker ignores
├── .env               ← secrets (never commit)
├── .env.example       ← template (commit this)
└── .gitignore         ← what Git ignores
```

---

## What Goes Where

```
┌─────────────────┬──────────────────────────────────────┐
│ File/Folder     │ Purpose                              │
├─────────────────┼──────────────────────────────────────┤
│ .env            │ secrets — API keys, passwords        │
│ .env.example    │ template showing required keys       │
│ config.js       │ non-secret constants                 │
│ server.js       │ HTTP layer only — wiring             │
│ agent.js        │ AI logic — tools, loop               │
│ tools/*.js      │ real world capabilities              │
│ middleware/*.js │ request processing pipeline          │
│ services/*.js   │ business logic                       │
│ utils/*.js      │ generic helpers                      │
│ ingest.js       │ one-time RAG setup                   │
│ document.txt    │ your knowledge base                  │
│ vector-db/      │ generated — don't edit               │
│ sessions.json   │ generated — don't edit               │
│ logs/           │ generated — don't edit               │
│ Dockerfile      │ container build recipe               │
│ .dockerignore   │ what stays out of container          │
│ .gitignore      │ what stays out of GitHub             │
└─────────────────┴──────────────────────────────────────┘
```

---

## Dev vs Production

```
DEVELOPMENT                        PRODUCTION (Render)
───────────                        ──────────────────
node server.js          →          Docker container
localhost:3000          →          https://your-app.onrender.com
.env file               →          Render environment variables
sessions.json (local)   →          sessions.json (in container)
manual ingest.js        →          npm run build (auto on deploy)
console.log debugging   →          logs/ files + Render log viewer
```

---

## Master Cheatsheet

### Node.js / npm
```bash
npm init -y                     # create new project
npm install package-name        # install package
npm install                     # install all from package.json
node server.js                  # run server
node ingest.js                  # run ingestion
npm start                       # run start script
npm run build                   # run build script
```

### Git / GitHub
```bash
git init                        # initialize repo
git add .                       # stage all changes
git commit -m "message"         # commit changes
git push                        # push to GitHub
git pull                        # pull latest
git status                      # check what changed
git log --oneline               # view commit history
```

### Docker
```bash
docker build -t app-name .      # build image
docker run -p 3000:3000 \       # run container
  --env-file .env app-name
docker images                   # list images
docker ps                       # list running containers
docker ps -a                    # list all containers
docker stop <id>                # stop container
docker logs <id>                # view logs
docker exec -it <id> sh         # open shell in container
docker rmi app-name             # remove image
docker tag app username/app     # tag for Docker Hub
docker push username/app        # push to Docker Hub
```

### curl (API Testing)
```bash
# Health check
curl http://localhost:3000/health

# Chat
curl -X POST http://localhost:3000/chat \
  -H "Content-Type: application/json" \
  -d '{"sessionId": "user1", "message": "your question"}'

# List sessions
curl http://localhost:3000/sessions

# Delete session
curl -X DELETE http://localhost:3000/chat/user1
```

### Express API Endpoints
```
GET  /health              → check if server is running
POST /chat                → send message to agent
GET  /sessions            → list all active sessions
DELETE /chat/:sessionId   → clear a session
```

### Environment Variables
```bash
# Check if variable is set
echo $GROQ_API_KEY

# Set temporarily (current terminal only)
export GROQ_API_KEY=your_key

# Load from .env (handled by dotenv in code)
import "dotenv/config";
```

---

## When Something Goes Wrong

```
Problem                          First thing to check
───────                          ────────────────────
Server won't start          →    port already in use? rm sessions.json?
Tool not being called       →    tool description specific enough?
Wrong tool being called     →    tool descriptions too vague?
Groq API error              →    GROQ_API_KEY set? quota exceeded?
Weather tool failing        →    OPENWEATHER_API_KEY set? city spelling?
RAG not finding answers     →    did you run ingest.js? score too high?
Docker build failing        →    check Dockerfile syntax, layer order
Render deploy failing       →    check build logs, env vars set?
Sessions not persisting     →    sessions.json gitignored (correct)?
Old data after doc update   →    bump DOCUMENT_VERSION, re-ingest
```

---

## The Three Phases of Your Project

```
Phase 1 — BUILD (Days 21-24)
  Write code → Test locally → Fix bugs → Repeat

Phase 2 — DEPLOY (Days 25-27)
  Dockerize → Push to GitHub → Deploy to Render → Share URL

Phase 3 — SCALE (Days 28-30)
  Add frontend → Add auth → Add database → Monitor
```