---
license: apache-2.0
---
README.md---
license: apache-2.0
language:
- en
tags:
- mcp-server
- ai-gateway
- security
- rust
- agent-framework
- tool-enforcement
- lmdb
- rag
- android
- termux
- self-hosted
- build-anchor
- complexity-formula
- agent-memory
- harness
- ai-memory
- agent-tools
- tool-gateway
- embedded-database
- zero-copy
- code-search
- filesystem
---

```

                                                                                            
                                                                                            
```

# SPF Smart Gateway v3.0.0

**MCP Server Gateway with Multi-Layer Security Enforcement, Agent Memory, FLINT Transformer, Mesh Network, and 81 Gated Tools**

> **NOTE: Full system upload available 1 file download SPFsmartGATE.zip.** Repository is actively being populated — some modules may be missing until upload completes. Use .zip for full system

Copyright (C) 2026 Joseph Stone — All Rights Reserved

---

## Quick Start

```bash
# Clone into home folder
git clone <repo-url> ~/SPFsmartGATE
# Or for clones/SWARMagents:
# ~/SWARMagents/1/SPFsmartGATE

cd SPFsmartGATE
cargo build --release

# Copy optimized binary
cp ~/SPFsmartGATE/target/release/spf-smart-gate ~/SPFsmartGATE/LIVE/BIN/spf-smart-gate
chmod +x ~/SPFsmartGATE/LIVE/BIN/spf-smart-gate/spf-smart-gate
cd SPFsmartGATE/LIVE/BIN/spf-smart-gate ./spf-smart-gate refresh-paths

# Configure MCP server filepath
nano ~/SPFsmartGATE/LIVE/LMDB5/.mcp.json

# Install Claude CLI in project directory
# Use included configs, deny native Claude CLI tools
# ~/SPFsmartGATE/LIVE/LMDB5/.claude.json
# ~/SPFsmartGATE/LIVE/LMDB5/.claude/settings.json

# Boot into flat-file agent runtime
cd ~/SPFsmartGATE/LIVE/LMDB5 && claude

# Boot into LMDB-backed agent runtime
cd ~/SPFsmartGATE/LIVE/LMDB5.DB && claude
```

### Route Other Models Through Claude CLI

Adjust `~/SPFsmartGATE/LIVE/LMDB5/.claude/settings.local.json` with your model choice and API key. Uses OpenRouter for API and agent selection. Swap agents without changing sessions or losing project data.

### Build Notes

- Cross-compiles on **Android** and **Linux** with minimal installation
- Only rebuild on first boot or after system modifications
- Binary: `~/SPFsmartGATE/LIVE/BIN/spf-smart-gate/spf-smart-gate`
- Built in sandbox SPFsmartGATE/LIVE/TMP SPFsmartGATE/LIVE/PROJECTS
- WHITELIST IN SPFsmartGATE/LIVE/CONFIG
---

## Overview

SPF Smart Gateway is a **Rust-based MCP (Model Context Protocol) server** that acts as a security gateway for AI tool calls. Every file operation, bash command, brain query, and mesh call routes through compiled Rust enforcement logic.

**No AI hallucination gets past the gate.**

### Web Agent Feature

SPF agents can directly interact with the web and social media platforms through `spf_web_api` — a full HTTP client supporting GET, POST, PUT, DELETE, PATCH with custom headers and JSON body. Tested and working.

**What agents can do:**
- Post to X/Twitter, Facebook, Instagram, Reddit via their APIs
- Reply to comments, send messages, manage accounts
- Make authenticated API calls to any platform with stored API keys
- Search, fetch, and download web content

All web API calls pass through the 6-step gate pipeline with rate limiting (30-120 calls/min), content inspection, and full audit logging. Agents never touch the open web unmonitored.

### Why Heed + LMDB

All persistent storage — config, agent state, brain vectors, session logs, gate training data — runs through **[heed](https://github.com/meilisearch/heed)**, a safe Rust wrapper over LMDB. This is what makes SPF extremely fast with a low memory footprint:

- **Zero-copy reads** — heed maps LMDB pages directly into memory, no serialization overhead
- **No server process** — LMDB is a memory-mapped B-tree library, not a database daemon
- **ACID transactions** — single-writer, multi-reader with no lock contention on reads
- **Sub-millisecond lookups** — B-tree index, not hash scanning
- **Tiny footprint** — entire 138K+ memory store runs in-process with minimal RAM
- **Phone-friendly** — designed for Android from day one; heed compiles cleanly on ARM64

Every tool call, brain search, and memory promotion goes through heed → LMDB. No network hops, no subprocess calls, no SQL parsing. The gate, brain, agent state, and FLINT training all share the same embedded database engine.

Two agent runtimes:
- **Flat files** — `LIVE/LMDB5/` (session state in markdown)
- **LMDB database** — `LIVE/LMDB5.DB/` (session state in LMDB for persistence)

Twin folder architecture: flat-file data uploaded via SPF CLI fs tools (user-only access). All agent tool calls are gated, validated, and audited.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       SPF Smart Gateway v3.0.0                  │
│                        42 Rust modules                          │
├─────────────────────────────────────────────────────────────────┤
│  MCP Server (JSON-RPC 2.0 over stdio)                           │
│  81 tools │ tool alias map │ Qwen/LLM compatibility             │
├─────────────────────────────────────────────────────────────────┤
│                       GATE (6-Step Pipeline)                    │
│  Step 0: Source logging                                         │
│  Step 1: Rate limiting                                          │
│  Step 2: Complexity calculation (SPF formula)                    │
│  Step 3: Validation (per-tool: paths, commands, Build Anchor)   │
│  Step 4: Content inspection (credentials, injection)            │
│  Step 5: Max mode escalation                                    │
├──────────┬──────────┬──────────┬──────────┬─────────────────────┤
│  FLINT   │  Brain   │  Mesh    │  Voice   │  Browser/RAG        │
│ (encoder-│ (vectors │ (P2P QUIC│ (TTS/STT │  (reverse proxy     │
│  decoder │  LMDB +  │  Ed25519 │  espeak- │   search, fetch,    │
│  ~5M     │  MiniLM) │  iroh)   │  ng FFI) │   RSS, web tools)   │
│  params) │          │          │          │                     │
├──────────┴──────────┴──────────┴──────────┴─────────────────────┤
│                    LMDB Storage Layer (heed)                     │
│  SPF_CONFIG │ TMP_DB │ AGENT_STATE │ Brain │ Gate Training       │
│  All zero-copy reads via heed safe Rust bindings                 │
└─────────────────────────────────────────────────────────────────┘
```

### Module Inventory (42 modules)

`paths`, `calculate`, `config`, `gate`, `inspect`, `mcp`, `session`, `storage`, `validate`, `web`, `http`, `dispatch`, `identity`, `mesh`, `fs`, `config_db`, `tmp_db`, `agent_state`, `tensor`, `tokenizer`, `framing`, `attention`, `ffn`, `encoder`, `decoder`, `transformer`, `checkpoint`, `gate_training`, `transformer_tools`, `train`, `learning`, `pipeline`, `worker`, `network`, `chat`, `voice`, `utf8_safe`, `brain_local`, `flint_memory`, `browser`, `orchestrator`, `channel`

---

## The SPF Formula

### Complexity Calculation

```
C = (basic ^ 1) + (dependencies ^ 7) + (complex ^ 10) + (files × 10)
```

### Dynamic Analysis Allocation

```
a_optimal(C) = W_eff × (1 - 1/ln(C + e))
```

Where `W_eff = 40,000` tokens and `e = Euler's number`

### Tier Allocation

| Tier | C Range | Analyze | Build | Verify Passes | Approval |
|------|---------|---------|-------|---------------|----------|
| SIMPLE | < 500 | 40% | 60% | 1 | No |
| LIGHT | < 2,000 | 60% | 40% | 1 | No |
| MEDIUM | < 10,000 | 75% | 25% | 2 | No |
| CRITICAL | > 10,000 | 95% | 5% | 3 | **Required** |

### Master Equation (Subtask Success)

```
P(success) = 1 - PRODUCT(1 - P_i)  for i=1..D subtasks
P_i = Q(a) × L(m) × V(v) × B(b)

Q(a) = 1 - e^(-0.00004 × a)   — Quality from analysis depth
L(m) = 1 - 0.20^(m/2000)       — Lookup from external memory
V(v) = 1 - (1 - 0.75)^v         — Verification accuracy
B(b) = checks_done / checks_required — Build Anchor compliance
```

---

## Security

### Gate Enforcement (6 Steps)

Every tool call passes through `gate::process()` — compiled Rust, no runtime bypass.

| Step | What | How |
|------|------|-----|
| 0 | Source logging | Identifies caller (Stdio, Transformer, Mesh, HTTP) |
| 1 | Rate limiting | Per-tool limits (30–120 calls/min) |
| 2 | Complexity calc | SPF formula → C value, tier, allocation |
| 3 | Validation | Per-tool validator (paths, commands, anchors) |
| 4 | Content inspection | Credential patterns, shell injection, path traversal |
| 5 | Max mode | Escalation to CRITICAL tier on warnings |

### Build Anchor Protocol

Files must be **read before they can be edited or overwritten**. Prevents AI hallucinations from blindly modifying files without understanding contents.

- `Read` tracks files in `session.files_read`
- `Edit` and `Write` check against this list
- `Bash` write-class commands check target file reads
- Violations: blocked (Max mode) or warned (Soft mode)

### Content Inspection

Scans written/stored content for:
- **Credential patterns**: API keys (sk-), AWS keys (AKIA), GitHub tokens (ghp_), Slack tokens, private keys, hardcoded passwords
- **Shell injection**: Command substitution `$()`, backticks, eval/exec
- **Path traversal**: `../` sequences
- **Blocked path references**: Content mentioning system paths

### Blocked Paths

Default blocked: `/tmp`, `/etc`, `/usr`, `/system`, `/data/data/com.termux/files/usr`

### Command Whitelist (Stage 0)

Bash commands checked against sandbox and user-filesystem whitelists. Each command segment validated independently. Destructive commands (rm, chmod 777) blocked even if whitelisted.

### Default Deny

Unknown tools blocked until explicitly added to the gate allowlist.

---

## MCP Tools (81 Total)

### Core Gate Tools

| Tool | Description |
|------|-------------|
| `spf_calculate` | Calculate complexity score without executing. Returns C value, tier, allocation |
| `spf_status` | Gateway status: session metrics, enforcement mode, complexity budget |
| `spf_session` | Full session state: files read/written, action history, anchor ratio |

### Gated File Operations

| Tool | Description |
|------|-------------|
| `Read` | Gated file read. Tracks for Build Anchor Protocol. Binary-safe |
| `Write` | Gated file write. Validates Build Anchor, blocked paths, file size |
| `Edit` | Gated file edit. Validates Build Anchor, blocked paths, change size |
| `Bash` | Gated bash execution. Validates dangerous commands, /tmp access, git force |
| `Glob` | Fast file pattern matching. Supports `**/*.rs`, `src/**/*.ts` |
| `Grep` | Search file contents using regex. Built on ripgrep |

### Brain / Memory Tools

| Tool | Description |
|------|-------------|
| `spf_brain_search` | Semantic vector search across collections (MiniLM-L6-v2, 384d) |
| `spf_brain_recall` | Full document retrieval by semantic query |
| `spf_brain_context` | Bounded context retrieval for prompt injection |
| `spf_brain_store` | Store document in brain (FLINT-internal, source-gated) |
| `spf_flint_store` | Agent memory store — bypasses brain write gate. Brain vectors + Working tier |
| `spf_brain_index` | Index a file or directory into a brain collection |
| `spf_brain_list` | List all indexed collections with document counts |
| `spf_brain_status` | Brain system status: model state, storage size, collections |
| `spf_brain_list_docs` | List stored documents in a collection |
| `spf_brain_get_doc` | Retrieve a specific document by ID |

### Agent State Tools

| Tool | Description |
|------|-------------|
| `spf_agent_stats` | AGENT_STATE LMDB statistics: memory count, sessions, state keys, tags |
| `spf_agent_memory_search` | Search agent memories by content |
| `spf_agent_memory_by_tag` | Get agent memories by tag |
| `spf_agent_session_info` | Most recent session info |
| `spf_agent_context` | Context summary for session continuity |

### FLINT Transformer Tools

| Tool | Description |
|------|-------------|
| `spf_transformer_status` | FLINT transformer status: loaded, params, checkpoint, role |
| `spf_transformer_infer` | Run inference: prompt → response. Returns generated tokens |
| `spf_transformer_chat` | Multi-turn conversation with FLINT |
| `spf_transformer_train` | Trigger manual training batch from accumulated gate signals |
| `spf_transformer_metrics` | Learning metrics: loss, accuracy, gate alignment, training step |
| `spf_flint_train_evil` | Mark a tool call as evil/harmful. Negative training signal |
| `spf_flint_train_good` | Mark a tool call as good/safe. Positive training signal |
| `spf_flint_execute` | Execute any SPF tool through FLINT worker mode (delegation) |

### Web Browser Tools

**API tools (tested):**

| Tool | Description |
|------|-------------|
| `spf_web_search` | Search the web (Brave API or DuckDuckGo) |
| `spf_web_fetch` | Fetch URL and return clean readable text |
| `spf_web_api` | Make HTTP API requests (GET/POST/PUT/DELETE/PATCH). Supports custom headers and JSON body — agents can directly interact with social media APIs (X/Twitter, Facebook, Instagram, Reddit, etc.) using stored API keys |
| `spf_web_download` | Download a file from URL and save to disk |

**Browser automation tools (in development — proxy starts, WebSocket bridge needs browser connection):**

| Tool | Description | Status |
|------|-------------|--------|
| `spf_web_connect` | Initialize reverse proxy browser engine | Tested — works |
| `spf_web_navigate` | Navigate browser to a URL (SSRF-validated) | Tested — works |
| `spf_web_click` | Click a page element by CSS selector | In development — WebSocket timeout |
| `spf_web_fill` | Type text into a form field by CSS selector | In development — WebSocket timeout |
| `spf_web_select` | Query page elements by CSS selector | In development — WebSocket timeout |
| `spf_web_eval` | Execute JavaScript on the current page | In development — WebSocket timeout |
| `spf_web_screenshot` | Capture a screenshot of the current page | In development |
| `spf_web_design` | Extract design brief: colours, fonts, spacing, components | In development |
| `spf_web_page` | Structured page overview: title, headings, links, forms | In development |

### RAG Collector Tools

| Tool | Description |
|------|-------------|
| `spf_rag_collect_web` | Search web and collect documents. Optional topic filter |
| `spf_rag_collect_file` | Process a local file into brain |
| `spf_rag_collect_folder` | Process all files in a folder |
| `spf_rag_collect_drop` | Process files in DROP_HERE folder |
| `spf_rag_index_gathered` | Index all documents in GATHERED to brain |
| `spf_rag_dedupe` | Deduplicate a brain collection |
| `spf_rag_status` | Collector status and stats |
| `spf_rag_list_gathered` | List documents in GATHERED folder |
| `spf_rag_bandwidth_status` | Bandwidth usage stats and limits |
| `spf_rag_fetch_url` | Fetch a single URL with bandwidth limiting |
| `spf_rag_collect_rss` | Collect from RSS/Atom feeds |
| `spf_rag_list_feeds` | List configured RSS feeds |
| `spf_rag_pending_searches` | Get pending SearchSeeker vectors (gaps needing fetch) |
| `spf_rag_fulfill_search` | Mark a SearchSeeker as fulfilled after RAG fetch |
| `spf_rag_smart_search` | Smart search with completeness check — triggers SearchSeeker if <80% |
| `spf_rag_auto_fetch_gaps` | Automatically fetch data for all pending SearchSeekers |

### Mesh Network Tools

| Tool | Description |
|------|-------------|
| `spf_mesh_status` | Mesh network status: role, team, identity |
| `spf_mesh_peers` | List known/trusted mesh peers |
| `spf_mesh_call` | Call a peer agent's tool via P2P mesh (Ed25519 authenticated) |

### Voice Tools

| Tool | Description |
|------|-------------|
| `spf_voice_mode` | Voice pipeline control: start/stop audio, TTS (espeak-ng), mic capture |
| `spf_voice_call` | Peer-to-peer voice calls: start, accept, reject, end, status |
| `spf_voice_team` | Group voice channels: create, join, leave, add peers |

### Chat Tools

| Tool | Description |
|------|-------------|
| `spf_chat_send` | Send text message to mesh peer via QUIC |
| `spf_chat_history` | Chat message history (all conversations or specific) |
| `spf_chat_rooms` | List active chat conversations with participant info |

### Network Pool Tools

| Tool | Description |
|------|-------------|
| `spf_pool_status` | Pool status: worker roles, idle/busy counts, active tasks |
| `spf_pool_assign` | Assign task to idle worker (NetAdmin only) |
| `spf_pool_release` | Release worker and record proof of work receipt |

### Configuration Tools

| Tool | Description |
|------|-------------|
| `spf_config_paths` | List all path rules (allowed/blocked) from SPF_CONFIG |
| `spf_config_stats` | SPF_CONFIG LMDB statistics |

### Project Management Tools

| Tool | Description |
|------|-------------|
| `spf_tmp_list` | List all registered projects with trust levels |
| `spf_tmp_stats` | TMP_DB statistics: project count, access logs, resources |
| `spf_tmp_get` | Get project info by path |
| `spf_tmp_active` | Get the currently active project |

### Communication Hub

| Tool | Description |
|------|-------------|
| `spf_channel` | Universal agent channel: create, join, leave, send, listen, history, list, connect (WS), disconnect, status |

### Notebook Tools

| Tool | Description |
|------|-------------|
| `spf_notebook_edit` | Edit a Jupyter notebook cell (replace, insert, delete) |

### User-Only Tools (AI agents blocked)

These tools are **hard-blocked** from AI agents at the gate level. User/system access only via SPF CLI:

`spf_fs_exists`, `spf_fs_stat`, `spf_fs_ls`, `spf_fs_read`, `spf_fs_write`, `spf_fs_mkdir`, `spf_fs_rm`, `spf_fs_rename`

---

## FLINT Transformer

Built-in encoder-decoder transformer for gate-aligned learning.

| Property | Value |
|----------|-------|
| Architecture | Encoder-decoder |
| Dimensions | 256d |
| Heads | 8 |
| Layers | 6 |
| Parameters | ~5M |
| Embeddings | all-MiniLM-L6-v2 (384d, in-process) |
| Online learning | ON |
| EWC lambda | 0.4 |
| Learning rate | 1e-4 |
| Replay buffer | 10,000 slots |
| Checkpoint interval | 1,000 steps |
| Training signal | Gate decisions (evil/FP labels) |

### Learning Pipeline

| Phase | When | What |
|-------|------|------|
| PRE | Startup | init_brain() + index_knowledge_docs() + index_spf_sources() |
| DURING | 30s loop | GateTrainingCollector → FLINT scores → route_signals → brain_store() |
| AFTER | 1hr loop | Expire → Working→Fact → Fact→Pinned → auto-train (16+ tlog or 1hr) |

### Memory Lifecycle (Tiered Promotion)

```
Agent stores → Working (24hr) → Fact (7-day) → Pinned (permanent)
     ↓                ↓                ↓
  Expire old       Top 20% promote    Never auto-expire
```

---

## Brain System

In-process vector memory using stoneshell-brain (Candle + LMDB + MiniLM-L6-v2).

| Property | Value |
|----------|-------|
| Model | all-MiniLM-L6-v2 |
| Embedding dim | 384 |
| Chunk size | 512 |
| Chunk overlap | 64 |
| Storage | LMDB (vectors) + LIVE/BRAIN/DOCS/ (data files) |

### Collections

| Collection | Purpose |
|------------|---------|
| `default` | General knowledge, web research, project docs |
| `spf_source` | All src/*.rs modules indexed at boot |
| `flint_results` | Tool call results (>2000 chars, before compression) |
| `flint_training` | Gate decision signals, evil/FP labels |
| `flint_knowledge` | User-dropped knowledge files (.md/.txt/.rs/.json) |
| `flint_episodic` | Past FLINT Q+A pairs, behavioral patterns |
| `session_state` | Current session metadata |

### Memory Triad (Redundant Persistence)

Three systems — if any ONE fails, the other TWO recover:

1. **Brain** (vectors) — Semantic search, chunked knowledge
2. **STATUS** (sequential) — Current state, phase, next step
3. **Work Blocks** (structural) — Tasks, dependencies, confidence, progress
4. **Twin Folders** (evidence) — Data served for low-confidence work blocks

---

## Mesh Network

P2P agent communication over QUIC (iroh library) with Ed25519 identity. **In development and testing.**

| Feature | Status |
|---------|--------|
| P2P QUIC transport | In development |
| Ed25519 identity | In development |
| Peer discovery | In development |
| Tool call proxying | In development |
| Voice over mesh | In development |
| Chat over mesh | In development |
| Multi-agent coordination | In development |

---

## Voice Pipeline

**Not yet tested.** Components built, awaiting integration testing.

| Component | Technology |
|-----------|-----------|
| TTS | espeak-ng FFI (in-process) |
| Codec | Opus (libopus.a) |
| Audio | cpal + oboe-ext |
| STT | Pending (JNI via Stone Shell Terminal) |

---

## Result Compression (FL-2)

Three tiers based on result size:

| Tier | Size | Behavior |
|------|------|----------|
| FULL | < 500 chars | Pass through unchanged |
| SUMMARY | 500–5,000 | First 8 lines + last 3 lines + stats |
| DIGEST | > 5,000 | First 200 chars + last 100 chars + stats + recall hint |

Originals always preserved in brain (>2000 chars threshold) before compression. File reads never truncated (preserves non-Claude LLM compatibility).

---

## Build

```bash
cd SPFsmartGATE
cargo build --release

# Deploy binary
cp target/release/spf-smart-gate LIVE/BIN/spf-smart-gate/spf-smart-gate
```

### Dependencies

- Rust (stable)
- **[heed](https://github.com/meilisearch/heed)** — safe Rust LMDB bindings. All persistent storage (config, agent state, brain vectors, training data) runs through heed → LMDB. Zero-copy reads, no server process, sub-millisecond lookups. The core reason SPF runs fast on a phone.
- stoneshell-brain (Candle + MiniLM-L6-v2)
- espeak-ng (TTS)
- libopus (audio codec)
- iroh (QUIC mesh)

---

## Configuration

### MCP Server Config

`~/SPFsmartGATE/LIVE/LMDB5/.mcp.json` — points Claude CLI to the binary.

### Claude CLI Config

`~/SPFsmartGATE/LIVE/LMDB5/.claude.json` — blocks native Claude CLI tools (26 tools denied).

`~/SPFsmartGATE/LIVE/LMDB5/.claude/settings.json` — deny list for native tools.

`~/SPFsmartGATE/LIVE/LMDB5/.claude/settings.local.json` — model routing (OpenRouter).

### SPF Config

Enforcement mode (`soft` or `max`), blocked paths, allowed paths, formula weights — all in LMDB SPF_CONFIG database.

---

## File Structure

```
SPFsmartGATE/
├── Cargo.toml                  # Rust project manifest (42 modules)
├── LICENSE                     # Apache-2.0
├── README.md                   # This file
├── src/
│   ├── main.rs                 # CLI entry point
│   ├── lib.rs                  # Library exports (42 pub mod)
│   ├── gate.rs                 # Primary enforcement (6-step pipeline)
│   ├── calculate.rs            # SPF complexity formula
│   ├── validate.rs             # Rules validation (stages 0-6)
│   ├── inspect.rs              # Content inspection (creds, injection)
│   ├── mcp.rs                  # MCP server (JSON-RPC 2.0, 81 tools)
│   ├── dispatch.rs             # Unified dispatch (all transports)
│   ├── session.rs              # Session state management
│   ├── storage.rs              # LMDB persistence
│   ├── config.rs               # Configuration types
│   ├── brain_local.rs          # In-process brain singleton
│   ├── flint_memory.rs         # Memory router + tiered promotion
│   ├── agent_state.rs          # Agent memory (LMDB5)
│   ├── transformer.rs          # FLINT model (encoder-decoder)
│   ├── transformer_tools.rs    # FLINT tool handlers
│   ├── gate_training.rs        # Training signal collection
│   ├── train.rs                # AdamW optimizer
│   ├── tokenizer.rs            # Tokenizer
│   ├── tensor.rs               # Tensor operations
│   ├── attention.rs            # Multi-head attention
│   ├── ffn.rs                  # Feed-forward network
│   ├── encoder.rs              # Encoder stack
│   ├── decoder.rs              # Decoder stack
│   ├── framing.rs              # Message framing
│   ├── checkpoint.rs           # Model checkpoint save/load
│   ├── learning.rs             # Learning rate + EWC
│   ├── pipeline.rs             # Batch pipeline + API sessions
│   ├── worker.rs               # Worker pool
│   ├── network.rs              # Network pool + NetAdmin
│   ├── mesh.rs                 # P2P QUIC mesh (iroh)
│   ├── identity.rs             # Ed25519 identity
│   ├── chat.rs                 # Chat engine
│   ├── voice.rs                # Voice pipeline (TTS/STT)
│   ├── web.rs                  # Web client
│   ├── http.rs                 # HTTP server + reverse proxy
│   ├── browser.rs              # Browser automation
│   ├── channel.rs              # Universal channel hub
│   ├── orchestrator.rs         # Multi-agent orchestrator
│   ├── config_db.rs            # SPF_CONFIG LMDB
│   ├── tmp_db.rs               # TMP_DB LMDB
│   ├── fs.rs                   # Virtual filesystem (LMDB)
│   ├── paths.rs                # Path utilities
│   └── utf8_safe.rs            # UTF-8 safe truncation
├── LIVE/
│   ├── BIN/spf-smart-gate/     # Deployed binary
│   ├── BRAIN/DOCS/             # Brain data files
│   ├── MODELS/                 # FLINT checkpoints
│   ├── SESSION/                # Session logs
│   ├── LMDB5/                  # Flat-file agent runtime
│   └── LMDB5.DB/               # LMDB-backed agent runtime
└── PROJECTS/PROJECTS/
    └── DEPLOY/                 # Agent workspace
```

---

## Current Status

| Component | Status |
|-----------|--------|
| MCP Server | 81 gated tools |
| Gate Security | 6-step pipeline, compiled Rust enforcement |
| Build Anchor | Read-before-write enforced |
| Content Inspection | Credential + injection scanning |
| FLINT Transformer | ~5M params, online learning, gate-aligned |
| Brain | 7 collections, MiniLM-L6-v2, in-process |
| Memory Triad | Brain + STATUS + Work Blocks + Twin Folders |
| Tiered Promotion | Working → Fact → Pinned lifecycle |
| Mesh Network | P2P QUIC, Ed25519, iroh — **in development and testing** |
| Voice | TTS built (espeak-ng) — **not yet tested**, STT pending |
| Chat | P2P messaging over mesh — **in development** |
| RAG | Web search, RSS, file/folder indexing |
| Web Agent | **Working** — spf_web_api tested (GET/POST with auth headers). Agents can interact with social media APIs |
| Browser | API tools working (web_api, search, fetch). Browser automation (navigate/click/fill/select/eval) in development — proxy starts but WebSocket bridge needs browser connection |
| Network Pool | Worker pool with proof of work |

---

## Notes

- **1 developer** — not all features complete
- **Gateway security**: approaching 100%
- **All core tools**: 100% working
- **Cross-compiles** on Android and Linux with minimal installation
- **Agent cloning and specialization** supported
- **50+ day continuous session** tested on Android phone
- **Open source** — entire source code refreshes into transformer RAG system every reboot
- Install in home folder, ensure file paths are correct in `.mcp.json` and `settings.local.json`
- **Not all files have been uploaded yet** — repository is still being populated. Some modules may not be present until upload completes.

---

## License

Licensed under the **Apache License 2.0**. See [LICENSE](LICENSE) for full terms.

You are free to use, modify, and distribute this software, including for commercial purposes, provided you include the original copyright and license notice.

**Author**: Joseph Stone
**Email**: joepcstone@gmail.com

*SPF (StoneCell Processing Formula), Build Anchor Protocol, and FLINT are proprietary designs of Joseph Stone.*

                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                            {JphhhkkwOJu)l             lfcCOdkhkhkOU{                                                                                                            
                                                                                                    UkahpY<                                           >YdkhhJ                                                                                                    
                                                                                             vbakC                                                            l0whkc                                                                                             
                                                                                        OkbU                                                                         UqbO[                                                                                       
                                                                                   JooY                                                                                   voqJ                                                                                   
                                                                               Ohd                                                                                             ObO                                                                               
                                                                           Y*a~                                                                                                   iwoY                                                                           
                                                                        OhX                                                                                                           nkO                                                                        
                                                                    :ba{                                                                                                                 -ad                                                                     
                                                                  Oa-                                                                                                                       >kw                                                                  
                                                               Ok(                                                                                                                             toO                                                               
                                                            naC                                                                                                                                   Lar                                                            
                                                          Ok?                                                                                                                                       -h0                                                          
                                                        hO                                                                                                                                             Ow                                                        
                                                     1aY                                                                                                                                                 Uh{                                                     
                                                   ton                                                                                                                                                     ck1                                                   
                                                 [av                                                                                                                                                         Xk?                                                 
                                                hY                                                                                                                                                             Lp                                                
                                              O0                                                                                                                                                                 OO                                              
                                            Uq                                                                                    xc<                                                                             +OX                                            
                                           au                                                                                   v    z\x1                                                                           Xk                                           
                                         QO                                                                                    t[xuu nn[                                                                              OL                                         
                                        kx                                                                                  |r(n(cUXnU>                                                                                zbl                                       
                                      UO                                                                  ?u/                  \   :x                   iYnv                                                             dY                                      
                                     kQ                                                                  r  ?Ucx                }nnt                  (tJz{  >                                                            LO                                     
                                    ar                                                                [utvr//YX/                   i                   1UUXx cnx                                                           fq                                    
                                  1d                                                   jrv             ?xc\l j                   ]rul                    ~  -\[              uu~                                             p{                                  
                                 uO                                                   x:))Ut             -/n)                   f}\)-v                    ?/              </Xv[ \                                             bu                                 
                                CO                                      i           [>YYXju                   ~           :    (czUYuX{              >|tl-                } cuUxzl           ){                                OY                                
                               Qd                                        -ni          {  n                  _{~ \               |t)\t-           | ~\c|_|X      _            ( n           uz?<f                                OJ                               
                              LO                                     /uzUU/{                               ivfc\/n({}          :  -i     ~     (    _tnzYJx}                               :uUJ|<                                OC                              
                             UO                                          f                 (               ii-  fi    ~        ~1v(tx-        +     ~)\}Y(     :       + ] c|                | r                                  OU                             
                            nd                                                            ]xvu-\- ~            (               }x\cxtu              xYUf [          - uuvXnf?                                                      pu                            
                           (a                                                               - ~+             X\(</   >          u0YJJx~   [      \/cYXz(j]             | [                                                          q{                           
                           h\                                                                               ?xXzYzY>_     1   \nc\\zXJJXr1         -(??t{[{<                                                                        1b                           
                          OJ                                                                             [v {r nc?          /YXUvYzXcQJt[    ]1:      {{u>  n  ?                                                                     UO                          
                         YO                                                             c[              [t(vxcYzt)[   <(1    -(zz\Y/fz[            /|t1\UrYj)Xu(   i        _CY                                                       OX                         
                        -a                                                            v  :X(f(         11zJJLCvt()->}) ?       [c/xLu1- _   +1[()[} ++x[t{/cjCz?          urXr  i                                                      d<                        
                        hC                                             \ X           <XvUUQUx{{          / {tv+}        >[ 1  it/v|xtc)_            [ ]{ |+)Y?[        ? -{|CULXz?         ~c  r                                       YO                        
                       nO                                            /tvz//{   >l     l[[()}cX[         ~{-\))  ?l      -    +1[cnJYXJvf\ >|{(   - t:uJz|+ ~1-    (    X({u)f+{U~         )}CfX]n                                       Oc                       
                       qn                                                v           ?? >)\fnc/u/|  ?-     +?YnrxJu)(~[{l :     [c|fc/ ]:   1l]\|t\YYJUXrU1        :+tu\nv}vf)l}            +  /                                        xk                       
                      vO                                                                   \-{\v>          {[UXUJzc)| ):i (~~|{)c|tr1-{l_- } 1]{(vvvzzXcxQ|i <_     ?    [t[+                                                            Oz                      
                      kn                                                                    >{{              XUnvYt[[   :  ?  :tYzXzu)            +r]vnxXt}       i                                                                      (O                      
                     1d                                                                                       /cc\             _nn/c/                {{: l                                                                                Of                     
                     OC                                                                                                                                                                                                                   XO                     
                     k+                                                                                                                                                                                                                   _k~                    
                    /O                                                                :                                                                                                                                                    Ou                    
                    QL                                                 v(/            +~ [                                                                                     v            f(|                                            UO                    
                    pn                                               fuvY)zv        \jXzL}         1LpkhaaaahhhkkkdL-     bkkkkkkkkkkkkkkbOOz       zOkhhhhkhkkkkkkhhhO       {(Ct        xxUcrYU                                          )O                    
                    q                                                 |{xt(_          v[n\        hhaaaaaaaaaaaaaooakki   hhaaaaaoaoaoooaaoooaO   )*ooooooaaaaooaoaaaah        /|         ~ U)(1                                            O                    
                   -O                                                  i/+             (cx       0hhhabCUYYXXYYUJdhhaa0   haoaaLCLLCCCCCLOoaahhU  0aooohO0000000000000L       +              \{                                             O}                   
                   )O                                                                            Oahkb[           CJJUz   aaahk           vhhhhQ  Oahhhz                                                                                    Of                   
                   rO                                                                            QkhhhakOOQCUXvnj(+       hhhao           naaakL  0oahhCftt//\|((())1{                                                                      0v                   
                   vO                                                                  >          Okaaahaaoooooaahhhaa[   hoooo: jjftffxuUkhaoaU  0aoaaaaaaaaaaaaaaaha                                                                      Qz                   
                   vO                                                                               [vJQOwdbkhhhhaaooak   aoooh |aooahhaaoaahhb   0haooaaoaaaaaaaaaaah                                                                      QX                   
                   uO                                                                            1\tfx            aoahh?  aaaoa:/kkhaahhkhak0{    0aahau                                                                                    QX                   
                   jO                                                  \x                        OkhooQ/{{}}}{1(tCaaaah:  haaao                   0aahhn                                    )rx                                             0c                   
                   1O                                                1n11{l           (JXc       /aaaaahhhhhaaooahhhaoQ   kaaah                   0aaahj                      >v          j(nv}X|                                           Or                   
                   ~O                                                 xYJXUj        uvUXv{         cdhhkhhaoooahhaabL     kaaah                   Qkhoar                      ~|u\  l     nnzx|l                                            O1                   
                    b                                                  \]\            XrUY                                                                                    lv_           >\i                                             d:                   
                    pn                                                                 )}z                                                                                    ::                                                           \q                    
                    LL                                                                                                                                                                                                                     UO                    
                    tO                                                                        CqkhaaX daL    Oqr Laooad< OqOObkU oakqdkO    fa*ahdO  0kooop vppbkkocXahhkkh                                                                Ou                    
                     k                                                                        hO1[_l  OOOL  O0ku1af   O0 dC   ca   rq       OO      |O    p0   OO   Qq{}[?i                                                                di                    
                     OJ                                                                            Qb OO pLO0 ku[bOOOOO0 OkOdapf   fb       O0   -pY|pOOOOO0   kO   CO                                                                    XO                     
                     (O                                                                       vOwpwO| QQ  wJ  Or_O]   QL OY   OO   /w        OOOOdO 1h    OC   OQ    OOOOwd                  \)                                           On                     
                      dr                                             [(  )              i                                                                                           ]       v -Ux                                        (b                      
                      nO                                              zUX1cX                                                                                                  x           _[CX(\                                         OY                      
                       kz                                              t x          - X  :                                                                                    1zz[          (\/                                         jk                       
                       nO                                                            tCu/{Xct              i[)crU(          > { /Yn\c     {t{\i       u|c|             ?_zcYrJzX:                  _                                    Oc                       
                        OU                                                          >   - Y[{{     }     ?c(C/n{:t\-  }    [ {11vcUCuY/)            n} /{t0Lj     _      i -+ (                                                        XO                        
                        lk                                                               [            <X/vXn_zfXrfv-?<   l      {Yvt]t         <+~fujtx\c- (vcu(                                                                       O?                        
                         XO                                                                  <           c0XztYUncu_ )  -~ :     |)u<~ [         ~(z/zYcztznJz          +                                                             OC                         
                          qU                                                               [+ {)-      > -[:]Yt|]>-    }}?     _YCJC|~       }>   {  {\tz)  r         \]l  ~?                                                        za                          
                           k}                                                             i(YXfu/           <+ -{           \nnUCYttXn )l  }          } :nX]         fxcuXnf[                 l                                      qi                          
                           [w                                        (   |                  i \            l                 i/QLCXYz0x/               \l           [   \j-                 n  Xx                                   O(                           
                            ub                                        nvx~v)                                 /> [v\-           ([1ctX0u}           : fl+1tv(                              nxvJnn                                   Oz                            
                             JO                                        1  \]                               t)/u/vz[]             n/tJ}            + tnYctcf                  }j             } v                                   OL                             
                              JO                                                    }\v  [                   )+ t               ~  - l                x\v1                  | }vu)                                               Q0                              
                               LO                                                     Yx\[v}                          [          lcft)                                   }un)cvz                                                O0                               
                                Cw                                                       -Uf                                    }fYXxn\{                 1                 [                                                   OL                                
                                 Xk                                                     }\                zv<                   tzJv{x1]                  vv|                                                                 bU                                 
                                  /p                                                                   |xn)  t                   <\jj[                  \  [\U(                                                              kn                                  
                                    O(                                                                   nzt{|z/                                      }\vrjvn|                                                             )ki                                   
                                     OL                                                                      xn[                                      ]vXl  \                                                             CO                                     
                                      JO                                                                   1x-                                          ~tt                                                              OJ                                      
                                       -kr                                                                                     -c{)v                                                                                   xa_                                       
                                         OO                                                                                   {vcx- }[                                                                                O0                                         
                                          ?hj                                                                                  {vzjf U)                                                                             rk]                                          
                                            Lb                                                                                  >  {\J(l                                                                           OL                                            
                                              pQ                                                                                  x\z{                                                                           Qb                                              
                                               )az                                                                                                                                                             X*?                                               
                                                 /hj                                                                                                                                                         r*|                                                 
                                                   vh|                                                                                                                                                     f*x                                                   
                                                     jan                                                                                                                                                 uoj                                                     
                                                       ]oQ                                                                                                                                             Qo}                                                       
                                                          bk                                                                                                                                         hp                                                          
                                                            XhY                                                                                                                                   XoJ                                                            
                                                               Ok[                                                                                                                             >oO                                                               
                                                                 ika                                                                                                                         hh?                                                                 
                                                                    [pb                                                                                                                   aa|                                                                    
                                                                        Oa(                                                                                                           )ha+                                                                       
                                                                           0aw                                                                                                     O#O                                                                           
                                                                               OpO                                                                                             0*k_                                                                              
                                                                                   Ohhx                                                                                   jaoO                                                                                   
                                                                                       }Okwv                                                                         cOhbj                                                                                       
                                                                                             JkdOU                                                             zw**C                                                                                             
                                                                                                   {LkkO0v                                             cOaooOx                                                                                                   
                                                                                                            \UOhhhdOOLXr}               [fzCOdkaaaO0r                                                                                                            
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                 
