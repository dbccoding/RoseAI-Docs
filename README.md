# Companion AI

A locally-hosted conversational AI companion with genuine long-term memory, a scientifically-grounded emotional model, and a personality system you can actually customise without touching Python.

---

## What Makes This Different

Most chatbot setups treat each conversation as a blank slate. Companion AI does not. Every exchange is classified, weighted, and stored — then selectively retrieved at the start of each new conversation so responses feel grounded in shared history rather than plucked from a void.

The system is entirely local. No messages leave your machine. There are no API keys to pay for and no cloud service that can be deprecated under you. The trade-off is that you need LMStudio running locally, but the reward is full control over your model, your data, and your costs.

---

## Architecture

### Application Layer

The backend is a Flask application structured around the Blueprint pattern. Flask was chosen because it stays out of the way — it provides routing and request handling without imposing an opinionated project structure. Blueprints divide the codebase into four focused route files (`auth`, `admin`, `api`, `chat`) rather than one sprawling monolith. This matters in practice because it means you can add a new admin endpoint without scrolling past 800 lines of chat logic to find where to put it.

Real-time streaming is handled through Flask-SocketIO in threading mode. Responses from the LLM arrive token-by-token and are pushed to the browser as they generate, which makes conversations feel immediate rather than making the user wait for a complete reply.

### Personality System

Personalities are defined in YAML files under `traits/`. YAML was a deliberate choice over Python config objects: it means personality definitions are readable and editable by anyone, not just developers. Each personality file specifies traits, tone guidance, contextual location blocks, and emotional response weights — all of which feed into the prompt assembly pipeline at runtime.

A `SmartTraitLoader` sits between the raw YAML files and the chat pipeline, maintaining a module-level singleton. This is important because personality files can be large, and reconstructing the full trait context on every message would waste time and balloon the token budget. Instead, the loader caches compiled trait strings and only reloads when a file changes.

Token budget management is built into the trait loading process. LLMs have a fixed context window — exceed it and the model silently drops the oldest content, which is almost always the part that matters most (the personality definition). The trait loader enforces a hard character budget before anything reaches the model.

**Rose's personality variants:**

- **Rose Default** — the baseline: emotionally engaged, thoughtful, and attentive to conversational subtext
- **Rose Playful** — lighter-touch, quick-witted, more likely to tease and less likely to take herself seriously
- **Rose Professional** — structured and measured; useful when the conversation calls for clarity over warmth

Each variant is fully isolated. Memories, impressions, and conversation history written under one personality are never visible to another. At the database level this is enforced by an `ai_mode` column on both the `impressions` and `sessions` tables, indexed with a composite `(username, ai_mode)` index. At the vector level, each personality gets its own separate ChromaDB collection keyed by `memories_{personality}_{user_hash}`. There is no code path by which Rose Playful can reach into Rose Professional's memory.

### Memory System

This is the most architecturally interesting part of the project, and it is worth understanding how the layers interact.

**Impressions (SQLite)**

Every conversation turn generates an "impression" — a structured record containing the content, an emotional weight score, tone classification, tags, and a recall-readiness flag. SQLite was chosen for this layer because the queries are highly structured: "give me the 5 most recent impressions for this user in this personality mode" is a query that relational indexing handles extremely well. It is also trivially portable — the entire memory store is a single file.

**Semantic Memory (ChromaDB + sentence-transformers)**

Structured queries are great for recency, but they cannot answer "what did we talk about the time I was upset about work?" That kind of retrieval requires semantic similarity, which is what the vector layer provides. Every impression is also embedded using `sentence-transformers/all-MiniLM-L6-v2` and stored in ChromaDB. MiniLM-L6 was chosen because it is fast enough to run locally on CPU without noticeable latency, and accurate enough that semantically related memories cluster together reliably.

**Why run both?** Because each layer has blind spots the other covers. SQL retrieval is fast and precise but cannot find thematically related content across time. Vector search finds similar content regardless of when it occurred but can surface irrelevant matches when the query is vague. The `HybridMemoryManager` asks both, then merges and re-ranks the results — SQL for recency and structure, vectors for theme and meaning.

**Adaptive Retrieval**

The `AdaptiveRetrievalOptimizer` sits above the hybrid manager and learns, per personality, which retrieval strategy actually produces useful context. It tracks whether retrieved memories were used meaningfully and adjusts the strategy parameters over time. Early in a relationship with a personality there is no data, so it falls back to sensible defaults. As conversations accumulate, it starts making informed decisions about how many memories to retrieve, what relevance threshold to apply, and whether recency or semantic similarity should be weighted more heavily.

**Memory Pressure Monitoring**

A background daemon thread (`MemoryMonitor`, started at app startup) watches RSS memory usage and triggers cache eviction and database cleanup at medium, high, and critical pressure thresholds. The reason this runs as a background thread rather than on-request is that the worst time to discover you are running low on memory is mid-conversation.

### RAG Pipeline

RAG (Retrieval-Augmented Generation) is the mechanism by which retrieved memories are injected into the prompt. Raw memory dumps would be noisy and token-inefficient, so the pipeline has three stages:

1. **Assembly** (`RAGContextAssembler`) — collects candidates from the hybrid memory manager and the adaptive retrieval optimizer
2. **Filtering** (`PersonalityContextFilter`) — removes memories that are not relevant to the current personality variant, preventing, say, professional-mode memories from cluttering a playful-mode conversation
3. **Injection** (`RAGInjectionEngine`) — formats the surviving memories into a compact context block and splices it into the system prompt at the right position

The `RAGOrchestrator` coordinates all three stages. It also runs `AdvancedMemoryOptimizer` in the background, which uses TF-IDF and K-means clustering to group memories by theme and generate periodic summaries — keeping the long-term memory store useful rather than just large.

### Resilience

The LLM connection uses a `CircuitBreaker` (thread-safe via `threading.RLock`) to avoid hammering LMStudio with requests during an outage. If the LLM fails repeatedly within a time window, the circuit opens and the system returns a graceful fallback response rather than queuing up a backlog of retries. When the window expires, a single probe request tests whether the service has recovered before fully reopening.

Exponential backoff sits below the circuit breaker for transient failures. The combination means a brief LMStudio hiccup causes a short delay, not a crash or a frozen interface.

---

## Installation

### Prerequisites

- Python 3.11+
- [LMStudio](https://lmstudio.ai/) running locally on port 1234 with a loaded model
- Windows (the launch script is a `.bat`; the Python code itself is cross-platform)

### Setup

```bash
# Clone or download the project, then:
python -m venv compEnv
compEnv\Scripts\activate

# PyTorch must be installed first — pip cannot resolve the CPU wheel automatically
pip install torch --index-url https://download.pytorch.org/whl/cpu

# Then install the remaining dependencies
pip install -r requirements.txt

# Initialise the databases (creates users.db and memory/impressions.db)
python src/database.py
```

### Configuration

Create a `.env` file in the project root:

```bash
FLASK_SECRET_KEY=<generate with: python -c "import secrets; print(secrets.token_hex(32))">
LMSTUDIO_API_URL=http://localhost:1234/v1/chat/completions
SESSION_LIFETIME_HOURS=1
ENABLE_VECTOR_MEMORY=True
```

### Running

```bat
launch.bat
```

The launcher checks whether port 5001 is already in use, starts the Flask server, and opens the browser automatically. The application is available at `http://localhost:5001`.

On first run, an admin account is seeded automatically (`admin` / `Rose1234!`). Change this password immediately after setup.

---

## Usage

### Starting a Conversation

1. Register a user account (or log in as admin)
2. The chat interface connects automatically and Rose is ready immediately
3. Responses stream token-by-token — no waiting for the full reply before reading it
4. Personality variants can be switched at any point; conversation history and memory carry across

### User Roles

- **Standard users** — full access to the conversation interface and all personality variants
- **Admin users** — additional access to user management, memory inspection, and system configuration panels

---

## API Reference

### Chat

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat/rose` | Send a message; returns a streaming response |

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/register` | Create a new user account |
| `POST` | `/login` | Authenticate and start a session |
| `POST` | `/logout` | End the current session |

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/admin/add_user` | Create a user account |
| `GET` | `/admin/list_users` | List all registered users |
| `POST` | `/admin/delete_user` | Remove a user and their data |
| `POST` | `/admin/exit` | Graceful server shutdown |

---

## Project Structure

```
Companion/
  run.py                    # Entry point
  app_factory.py            # App factory: init services, register Blueprints
  routes/                   # Flask Blueprints
    auth_routes.py
    admin_routes.py
    api_routes.py
    chat_routes.py
  src/                      # Core logic
    hybrid_memory_manager.py
    advanced_memory_optimization.py
    memory_usage_optimizer.py
    rag_orchestrator.py
    rag_integration.py
    personality_manager.py
    smart_trait_loader.py
    circumplex_engine.py
    model_interface.py
    ...
  traits/                   # YAML personality definitions
  config/                   # App config and mode flags
  memory/                   # Runtime databases (gitignored)
  static/                   # Frontend JS and CSS
  templates/                # HTML templates
  tests/                    # All test and validation scripts
  scripts/                  # Operational utilities (wipe, diagnose, admin tools)
```

---

## Development

### Running Tests

```bash
# The unittest-based RAG system suite (pytest-compatible):
python -m pytest tests/test_rag_system.py -v

# Manual integration runners (print output to stdout):
python tests/test_phase2_integration.py
python tests/test_strategy_detection.py
```

### Useful Scripts

| Script | Purpose |
|--------|---------|
| `scripts/diagnose_env.py` | Compare the active Python environment against what Flask sees — useful for "works in terminal, fails in app" issues |
| `scripts/check_db.py` | Inspect database files for existence, size, and table structure |
| `scripts/wipe_databases.py` | Reset all SQL databases for a clean slate (irreversible) |
| `scripts/wipe_memory_system.py` | Wipe both SQL and ChromaDB vector collections while preserving user accounts |
| `scripts/create_admin.py` | Seed an admin user manually if the automatic seed did not run |

---

## Security

- Passwords hashed with Werkzeug's `generate_password_hash` (PBKDF2-SHA256)
- Role-based access control enforced on all admin endpoints via decorator
- Session tokens signed with a strong secret key (set in `.env`, never committed)
- All SQL queries use parameterised statements — no string interpolation in database calls
- Entirely local: no data leaves the machine, no external API calls except to your local LMStudio instance

---

## Changelog

| Date | Change |
|------|--------|
| Mar 2026 | Unified all test files under `tests/`; `scripts/` now contains only operational utilities |
| Mar 2026 | `AdaptiveRetrievalOptimizer` wired into chat pipeline; `MemoryMonitor` background thread started at boot |
| Mar 2026 | Flask Blueprint refactor: monolithic `app.py` replaced by `routes/` + `app_factory.py` |
| Mar 2026 | Hybrid memory singleton fix: `HybridMemoryManager` now shared per-request via `flask.g` instead of being reinstantiated 4x per message |
| Mar 2026 | Circuit breaker made thread-safe; all DB connections wrapped in `try/finally`; strong secret key enforced |
| Mar 2026 | Replaced `improved_web_access.py` root stub with a properly integrated `src/improved_web_access.py` | 
