# Product Building — Practical Q&A Guide
## 30-Day Web Development Learning Path — Post Day 30 Reference

---

## 1. Navigating Multiple Domains Without Mastering All

The goal is not to master every domain — it's to know enough to build and know when to stop.

### The 80/20 Approach Per Domain

```
Database:
  Know:  basic CRUD, indexes, relationships
  Skip:  query optimization, sharding, replication
  Stuck: ask Claude/ChatGPT the specific query

Security:
  Know:  hash passwords, use JWT, validate input, HTTPS
  Skip:  penetration testing, security audits
  Stuck: use established libraries (bcrypt, helmet.js)

UX:
  Know:  clear error messages, loading states, mobile responsive
  Skip:  user research, A/B testing, accessibility audits
  Stuck: copy patterns from apps you use daily

Error Fixing:
  Know:  read error message carefully, check line numbers
  Skip:  memorizing every error type
  Stuck: paste exact error into Claude/Stack Overflow
```

### When to Ask for Help vs Fix Yourself

```
Spent 15 minutes on it           → ask for help
It is blocking everything else   → ask immediately
Minor polish issue                → skip for now, come back later
```

### Libraries That Handle Hard Problems for You

| Domain | Library | What it handles |
|--------|---------|----------------|
| Auth | passport.js, jsonwebtoken | JWT, sessions, OAuth |
| Passwords | bcrypt | Hashing, salting |
| Validation | express-validator, joi | Input sanitization |
| Payments | Stripe SDK | PCI compliance |
| Email | nodemailer, SendGrid | SMTP, templates |
| File upload | multer | Multipart forms |
| DB queries | Prisma, Mongoose | Type safe queries |

The pattern: find the industry standard library, read its quickstart, use it. Don't reinvent solved problems.

---

## 2. Frontend First or Backend First?

### Decision Framework

```
Can you describe the UI clearly?   → start frontend
Core value is data/logic?          → start backend
Not sure yet?                      → start backend
```

### Why Backend First is Usually Better

```
Backend first:
  ✅ Core logic works before UI exists
  ✅ Test with curl — no UI needed
  ✅ Frontend just displays what backend provides
  ✅ Easier to change UI than change API contracts

Frontend first:
  ✅ See results immediately — motivating
  ✅ Good for UI-heavy apps (dashboards, tools)
  ❌ Often leads to mocking data that never matches real API
  ❌ Have to redesign when backend reality differs
```

### The Right Approach — API First

```
1. Define your endpoints on paper
   GET  /users
   POST /chat
   DELETE /session/:id

2. Build backend until curl works
   curl -X POST http://localhost:3000/chat \
     -d '{"message": "hello"}'

3. Build frontend against real API
   No mocking, no guessing — real data from day one
```

This is what professional teams do — called API-first design.

---

## 3. Running LLMs Locally Without API Keys

Run models completely free on your own machine — no internet required.

### Option 1 — Ollama (Recommended)

```bash
# Install from ollama.com
# Pull a model
ollama pull llama3.2        # 2GB — good for most tasks
ollama pull mistral         # 4GB — better quality
ollama pull llama3.2:1b     # 1GB — fastest, less capable

# Starts local API at http://localhost:11434
# OpenAI-compatible format — works with existing code
```

Use in Node.js with OpenAI SDK:
```javascript
import OpenAI from "openai";

const client = new OpenAI({
    baseURL: "http://localhost:11434/v1",
    apiKey: "ollama",  // required field but ignored locally
});

const response = await client.chat.completions.create({
    model: "llama3.2",
    messages: [{ role: "user", content: "Hello" }]
});
```

### Option 2 — LM Studio

GUI application — download models visually, run locally, exposes a local API. Good for testing models without writing code.

### Comparison

| | Local (Ollama) | Cloud (Groq/Anthropic) |
|---|---|---|
| Cost | Free | Pay per token |
| Speed | Depends on hardware | Fast |
| Privacy | 100% private | Data leaves machine |
| Model quality | Good, not best | Best available |
| Internet needed | No | Yes |
| Setup | One time install | Just API key |

### Hardware Requirements

```
llama3.2:1b   → 8GB RAM minimum
llama3.2      → 16GB RAM recommended
llama3.3:70b  → 64GB RAM (serious hardware)
```

### Switching Between Local and Cloud

Your code barely changes:
```javascript
// Cloud (Groq)
const client = new Groq({ apiKey: process.env.GROQ_API_KEY });

// Local (Ollama) — same interface
const client = new OpenAI({
    baseURL: "http://localhost:11434/v1",
    apiKey: "ollama"
});

// Both use identical API calls
const response = await client.chat.completions.create({
    model: "llama3.2",
    messages: [...]
});
```

---

## 4. Multiple Agents — Structure and Communication

### Directory Structure

```
multi-agent/
├── agents/
│   ├── orchestrator.js     ← master agent, delegates tasks
│   ├── researchAgent.js    ← searches and gathers information
│   ├── writerAgent.js      ← writes content
│   └── reviewerAgent.js    ← reviews and improves output
│
├── tools/
│   ├── search.js           ← shared tools used by any agent
│   ├── scraper.js
│   └── fileWriter.js
│
├── memory/
│   ├── shortTerm.js        ← current task context
│   └── longTerm.js         ← persistent knowledge (Redis/DB)
│
├── coordinator.js          ← wires agents together
└── index.js                ← entry point
```

### How the Orchestrator Works

```
User gives high level task
        ↓
Orchestrator breaks into subtasks
        ↓
Delegates to specialist agents
        ↓
Collects results
        ↓
Combines into final output
        ↓
Returns to user

Example — "Write a blog post about AI trends":
  Orchestrator → Research Agent:  "find latest AI trends"
  Orchestrator → Writer Agent:    "write post using this research"
  Orchestrator → Reviewer Agent:  "check and improve this draft"
  Orchestrator → User:            final polished post
```

### Three Communication Patterns

**Pattern 1 — Direct Function Call (simplest)**
```javascript
// orchestrator.js
import { runResearchAgent } from "./agents/researchAgent.js";
import { runWriterAgent } from "./agents/writerAgent.js";
import { runReviewerAgent } from "./agents/reviewerAgent.js";

export async function runOrchestrator(task) {
    const research = await runResearchAgent(task);
    const draft = await runWriterAgent({ task, research });
    const final = await runReviewerAgent({ draft });
    return final;
}
```

**Pattern 2 — Shared Message Queue**
```javascript
// Agents work independently and in parallel
// Each publishes results, others listen

queue.push({ type: "RESEARCH_DONE", data: results });

queue.on("RESEARCH_DONE", async (data) => {
    const draft = await write(data);
    queue.push({ type: "DRAFT_READY", data: draft });
});
```

**Pattern 3 — Shared Memory/Database**
```javascript
// Agents communicate via shared state in Redis or DB
await redis.set("task:123:research", JSON.stringify(results));

// Another agent reads it when ready
const research = JSON.parse(await redis.get("task:123:research"));
```

### When to Use Which Pattern

| Pattern | Use when |
|---------|---------|
| Direct function call | Simple sequential workflows |
| Message queue | Parallel tasks, agents work independently |
| Shared memory | Long running tasks, agents need shared context |

### When to Split Into Multiple Agents

```
Single agent getting too complex:
  ├── Tool list too long (10+ tools)
  ├── Prompt getting too large
  ├── Agent making poor decisions
  └── Different tasks need different expertise

Split into multiple agents when:
  ├── Clear separation of responsibilities
  ├── Tasks can run in parallel
  ├── Different tools needed per task
  └── One agent's output feeds another
```

---

## 5. When is it Ready to Demo?

### The Simple Rule

**Demo when it does ONE thing well end-to-end.**

### Ready vs Not Ready

```
Not ready to demo:
  ❌ Many features, none working reliably
  ❌ Needs you to explain what it does
  ❌ Crashes on obvious inputs
  ❌ Core value proposition unclear
  ❌ Only works with specific inputs you control

Ready to demo:
  ✅ One clear user journey works perfectly
  ✅ Someone can use it without your help
  ✅ The core problem it solves is obvious
  ✅ Does not crash on normal use
  ✅ A non-technical person understands it in 2 minutes
```

### The Mom Test

Could your non-technical parent use it and understand what it does in 2 minutes? If yes — demo it.

### What to Skip Before Demo

```
Skip:                          Focus on:
──────                         ─────────
Perfect UI                     Working UI
All edge cases handled         Happy path works perfectly
Scalability                    Core feature works
Performance optimization       Feature exists and runs
100% test coverage             Manual testing passes
Perfect error messages         No crashes on normal use
Every possible feature         One feature done well
```

### The Trap — Infinite Polishing

```
Build feature A → "just need to polish this"
Polish A → "need feature B before showing anyone"
Build B → "A needs to be redone now"
Redone A → still never demos anything
```

**Fix:** Set a demo date before you start building. Work backwards from it. Ship on that date no matter what.

### Minimum Viable Demo Checklist

```
□ Core feature works end-to-end
□ No crashes on happy path
□ Can be used without your guidance
□ One sentence explains what it does
□ Looks reasonable (not broken UI)
□ You can show it in under 3 minutes
```

---

## 6. Where to Sell Your SaaS

### B2C — Selling to Individuals

```
Product Hunt    ← launch here first, huge developer/maker audience
Hacker News     ← "Show HN" post, technical and startup audience
Reddit          ← find subreddit for your specific niche
Twitter/X       ← build in public, share progress regularly
TikTok/YouTube  ← show it working, not talking about it
AppSumo         ← lifetime deals, good for early cash
```

### B2B — Selling to Businesses

```
LinkedIn        ← direct outreach to decision makers
Cold email      ← still works if targeted and personal
Industry forums ← be genuinely helpful, mention product naturally
G2/Capterra     ← software review sites, buyers check here
Your own blog   ← SEO for "[problem] solution" searches
```

### For Developer Tools Specifically

```
GitHub          ← open source version drives awareness
dev.to          ← write tutorials using your tool
npm/PyPI        ← publish as package, developers find it
Discord servers ← join communities where your users hang out
```

### Pricing Models

| Model | How it works | Best for |
|-------|-------------|---------|
| Freemium | Free tier + paid features | High volume consumer apps |
| Free trial | 14 days free then pay | B2B, higher ticket |
| Usage based | Pay per API call/message | Developer tools |
| Flat monthly | Fixed price per month | Predictable SaaS |
| Lifetime deal | One time payment | Early revenue, AppSumo |

### The Most Important Thing About Selling

Talk to 10 potential customers BEFORE building anything.

```
Wrong approach:
  Build → Launch → Crickets → "why isn't anyone buying?"

Right approach:
  Talk to people → Find painful problem people pay to solve →
  Build minimal solution → Charge from day one →
  Improve based on real feedback
```

**Questions to ask potential customers:**
```
"What is the hardest part of [problem]?"
"How are you solving it today?"
"How much does that cost you?"
"Would you pay $X for a better solution?"
```

If they say "yes I'd pay for that" — build it.
If they say "that sounds interesting" — keep looking.

---

## Summary — The Builder's Mindset

```
Domain knowledge:   80/20 rule — good enough beats perfect
Starting point:     API first, then frontend
Local AI:           Ollama for free, private, offline development
Multiple agents:    Orchestrator pattern — one boss, many specialists
Demo readiness:     One thing done well beats many things half done
Selling:            Talk to customers first, build second
```

---

## Quick Reference — Tools and Resources

| Need | Tool/Resource |
|------|--------------|
| Local LLM | ollama.com |
| Free cloud LLM | console.groq.com |
| Launch platform | producthunt.com |
| Lifetime deals | appsumo.com |
| Software reviews | g2.com, capterra.com |
| Domain ideas | answerthepublic.com |
| Competitor research | similarweb.com |
| Landing page | carrd.co (no code) |
| Payments | stripe.com |
| Email | resend.com, sendgrid.com |