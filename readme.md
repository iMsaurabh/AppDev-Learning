# Node.js Project — Best Practices & Decision Guide
## 30-Day Web Development Learning Path

---

## 1. Directory Structure — Group by Purpose

As projects grow, organize files by **what role they play**, not what they are.

```
project/
├── config.js             ← non-secret constants and app settings
├── server.js             ← wiring only — imports and connects everything
│
├── routes/               ← Express route handlers
│   └── chat.js
│
├── middleware/           ← Express middleware functions
│   ├── validate.js       ← input validation rules
│   └── rateLimiter.js    ← rate limiting setup
│
├── services/             ← business logic specific to your app
│   ├── agent.js          ← agent loop and tool router
│   └── sessionStore.js   ← session read/write
│
├── tools/                ← agent tool functions
│   ├── calculator.js
│   ├── weather.js
│   └── notes.js
│
├── utils/                ← stateless generic helpers
│   └── logger.js
│
└── logs/                 ← generated log files
```

**Visualize your current structure anytime:**
```bash
find . -not -path '*/node_modules/*' -not -path '*/.git/*' | sort
```

---

## 2. Identifying and Selecting Libraries

Follow this evaluation order:

**Step 1 — npmjs.com**
Search your need e.g. "rate limiting express". Check:
- Weekly downloads — higher means more community support
- Last publish date — abandoned libraries are risky
- Dependencies count — fewer is better

**Step 2 — GitHub**
- Stars — popularity indicator
- Open issues — too many unresolved is a red flag
- README quality — good docs = good library

**Step 3 — bundlephobia.com**
Check library size — good habit even for backend.

**Industry standards for common needs:**

| Need | Standard Library |
|------|-----------------|
| Logging | winston + morgan |
| Validation | express-validator or joi |
| Rate limiting | express-rate-limit |
| Auth | passport.js or jsonwebtoken |
| DB ORM | prisma or mongoose |
| Testing | jest or vitest |
| Sessions | express-session + connect-redis |

For most common needs an industry standard already exists. Google "best library for X express node" and the same names will keep appearing.

---

## 3. Correctly Initializing and Using a Library

Every library follows the same pattern. Read in this order:

**Step 1 — README quickstart section always shows:**
- How to install
- How to initialize
- Basic usage example

**Step 2 — Three common initialization patterns:**

```javascript
// Pattern 1 — Direct import and use
import library from "library-name";
library.doSomething();

// Pattern 2 — Create instance first
import Library from "library-name";
const instance = new Library({ options });
instance.doSomething();

// Pattern 3 — Named exports
import { specificThing } from "library-name";
specificThing();
```

**Step 3 — Export correctly:**

```javascript
// Default export — one main thing per file
export default myThing;
import myThing from "./file.js";        // any name works on import

// Named exports — multiple things per file
export { thingOne, thingTwo };
import { thingOne } from "./file.js";   // name must match exactly
```

**Rule of thumb:**
- `export default` → main purpose of the file (e.g. `logger`)
- `export { }` → file provides multiple utilities (e.g. `saveNote, readNotes`)

---

## 4. New File vs Updating Existing File

Follow the **Single Responsibility Principle** — each file should do one thing.

**Create a new file when:**
- The functionality can be described independently in one sentence
- It will be reused across multiple files
- It has its own state or configuration
- Adding it to an existing file makes that file noticeably longer

**Keep in existing file when:**
- It's directly related to that file's purpose
- It's only used once and is very short (under 10 lines)
- Separating it would create unnecessary complexity

**Practical decision guide:**

| Code | Where it belongs |
|------|-----------------|
| Route definitions | `routes/chat.js` |
| Validation rules | `middleware/validate.js` |
| Rate limiters | `middleware/rateLimiter.js` |
| Logger setup | `utils/logger.js` |
| Session read/write | `services/sessionStore.js` |
| Agent loop | `services/agent.js` |
| Tool functions | `tools/*.js` |
| Express app setup | `server.js` |

**`server.js` should only import and wire things together — never contain business logic.**

---

## 5. `.env` vs `config.js`

Two different files for two different purposes:

| Goes in `.env` | Goes in `config.js` |
|----------------|---------------------|
| Secrets and API keys | Non-secret constants |
| Environment specific values | App wide settings |
| Passwords and tokens | Default values |
| Different per deployment | Same across all environments |
| Would be embarrassing on GitHub | Safe to commit |

```bash
# .env — secrets, different per environment
GROQ_API_KEY=sk-xxx
OPENWEATHER_API_KEY=xxx
PORT=3000
```

```javascript
// config.js — constants, same everywhere, safe to commit
export const MAX_ITERATIONS = 10;
export const MAX_HISTORY = 10;
export const MAX_MESSAGE_LENGTH = 1000;
export const MAX_SESSION_ID_LENGTH = 50;
export const SESSIONS_FILE = "./sessions.json";
export const LOGS_DIR = "./logs";
```

Usage:
```javascript
import { MAX_ITERATIONS, MAX_HISTORY } from "./config.js";
```

**Simple test to decide:**
- Would this value differ between your laptop and production? → `.env`
- Is this a number or setting controlling app behavior? → `config.js`
- Would you be embarrassed if this appeared on GitHub? → `.env`

---

## 6. Utility vs Service

A distinction that trips up many developers:

**Utility** — stateless helper that does one small generic task. Knows nothing about your business domain. Could be copy-pasted into any project unchanged.

```javascript
// utils/logger.js
// Knows nothing about sessions, agents, or notes
// Could be dropped into any Node.js project
logger.info("something happened");
logger.error("something broke");
```

**Service** — contains business logic specific to your application. Knows about your domain — your sessions, your agent, your data.

```javascript
// services/sessionStore.js
// Knows about YOUR sessions — specific to this project
export function loadSessions() { ... }
export function saveSessions() { ... }

// services/agent.js
// Knows about YOUR tools and agent loop
export function runAgent(messages) { ... }
```

**Simple test:**
Ask yourself: *"Could I drop this file into a completely different project and use it without changing anything?"*
- **Yes** → it's a utility
- **No, it's specific to this app** → it's a service

---

## Key Principles Summary

| Principle | Rule |
|-----------|------|
| Single Responsibility | Each file does one thing |
| Separation of Concerns | Business logic never in server.js |
| Secret Management | Secrets in .env, constants in config.js |
| Library Selection | Check downloads, maintenance, GitHub stars |
| Export Style | default for single export, named for multiple |
| Utility vs Service | Generic and reusable vs domain specific |

---

## Note on Current Project

Our Days 21-23 project didn't follow perfect modularization — `server.js` contains both middleware and routes. This was intentional to keep focus on learning core concepts. In future projects we'll apply this structure from the start.