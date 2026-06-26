# 🏗️ Local-AI Architecture & Flow Diagrams

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                       │
│                        🖥️  USER BROWSER                             │
│                   (localhost:3000)                                   │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               │ HTTP/WebSocket
                               │
                    ┌──────────▼──────────┐
                    │                     │
                    │   🌐 Open WebUI     │
                    │  (Port: 3000→8080)  │
                    │                     │
                    │ ✓ Chat Interface    │
                    │ ✓ User Auth         │
                    │ ✓ RAG Support       │
                    │ ✓ Search Toggle     │
                    │                     │
                    └─────┬────────┬──────┘
                          │        │
                  ┌───────▼─┐  ┌───▼────────┐
                  │ Query   │  │   Search   │
                  │ Model   │  │   Toggle   │
                  └───┬─────┘  └───┬────────┘
                      │            │
          ┌───────────┴─┬──────────┴────────┐
          │             │                   │
    ┌─────▼──────┐ ┌────▼─────────┐  ┌─────▼──────────┐
    │   📂 RAG   │ │ 🧠 Ollama    │  │ 🔍 SearXNG    │
    │  (Uploads) │ │ (Port: 11434)│  │ (Port: 8081)  │
    │            │ │              │  │               │
    │ • PDFs     │ │ • LLM Models │  │ • Web Search  │
    │ • Docs     │ │ • Generation │  │ • Aggregator  │
    │ • Images   │ │ • Inference  │  │ • Scraping    │
    │            │ │              │  │               │
    └─────┬──────┘ └────┬─────────┘  └─────┬──────────┘
          │             │                  │
          └─────────────┴──────────────────┘
                        │
          ┌─────────────▼──────────────┐
          │                            │
          │  🌉 Docker Network         │
          │  (local-ai bridge)         │
          │                            │
          └─────────────┬──────────────┘
                        │
          ┌─────────────▼──────────────┐
          │                            │
          │   💾 Data Volumes          │
          │                            │
          │ ✓ ollama_data             │
          │   └─ Models (~40GB max)   │
          │                            │
          │ ✓ openwebui_data          │
          │   └─ Chats, users, DBs    │
          │                            │
          └────────────────────────────┘
```

---

## Complete Query Flow

```
USER INPUT
    │
    ▼
┌─────────────────────────────────────┐
│  1. User types in Open WebUI        │
│     "What is the latest news?"      │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│  2. Search Toggle Check             │
│     Is search enabled?              │
└─────────────────────────────────────┘
    │
    ├─────YES─────┬──────NO──────┐
    │             │              │
    ▼             ▼              ▼
[Query      [SearXNG       [Ollama Only]
 Prepared]   Request]         │
    │          │              │
    │     ┌────▼────────┐    │
    │     │ Web Search  │    │
    │     │  Results    │    │
    │     └────┬────────┘    │
    │          │             │
    └──────────┼─────────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 3. Context Building  │
    │                      │
    │ • User query         │
    │ • Search results     │
    │ • RAG documents      │
    │ • Chat history       │
    │ • Model selection    │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 4. Send to Ollama    │
    │                      │
    │ POST /api/generate   │
    │ with full context    │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 5. Ollama Processes  │
    │                      │
    │ • Load model         │
    │ • Tokenize input     │
    │ • Generate response  │
    │ • Stream output      │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 6. Response Stream   │
    │                      │
    │ Tokens sent to UI    │
    │ in real-time         │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 7. UI Displays       │
    │                      │
    │ Animated text        │
    │ response appears     │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ 8. Save Conversation │
    │                      │
    │ • Chat history       │
    │ • User query         │
    │ • Model response     │
    │ • Metadata           │
    └──────────┬───────────┘
               │
               ▼
        ✅ COMPLETE
```

---

## RAG (Retrieval-Augmented Generation) Flow

```
UPLOAD DOCUMENT
    │
    ▼
┌─────────────────────────┐
│ User uploads PDF/DOC    │
│ via Open WebUI          │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Extract Text & Images   │
│ Parse document content  │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Split into Chunks       │
│ 256-1024 tokens each    │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Generate Embeddings     │
│ (semantic vectors)      │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Store in Vector DB      │
│ (openwebui_data volume) │
└──────────┬──────────────┘
           │
           ▼
    💾 INDEXED

─────────────────────────────────────

QUERY WITH DOCUMENT CONTEXT
    │
    ▼
┌─────────────────────────┐
│ User asks question      │
│ related to document     │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Find Similar Chunks     │
│ RAG_TOP_K=10 chunks     │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Relevance Filtering     │
│ Threshold: 0.3          │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Build Context           │
│ Question + Docs + Chat  │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Send to Ollama          │
│ with document context   │
└──────────┬──────────────┘
           │
           ▼
✅ Response cites document
```

---

## Service Dependencies & Health Checks

```
                    ┌─────────────────┐
                    │  Open WebUI     │
                    │  (Main Service) │
                    └────────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼ (waits for)         ▼ (waits for)
        ┌──────────────────┐   ┌──────────────────┐
        │   Ollama         │   │   SearXNG        │
        │ (HEALTHY)        │   │ (STARTED)        │
        └────────┬─────────┘   └──────────────────┘
                 │
                 ▼ (provides)
        ┌──────────────────┐
        │  API Endpoint    │
        │  :11434/api/tags │
        └──────────────────┘

Health Check Flow:
    │
    ├─→ Open WebUI starts
    │   └─→ Waits for Ollama (HEALTHY)
    │       └─→ curl http://ollama:11434/api/tags
    │           └─→ Returns model list ✓
    │
    └─→ Waits for SearXNG (STARTED)
        └─→ curl http://searxng:8080/
            └─→ Returns 200 OK ✓
                    │
                    ▼
            All Services Ready ✅
```

---

## Container Port Mappings

```
┌─────────────────────────────────────────────────────────────┐
│                   Docker Host (localhost)                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Port 3000 ─────────→ Open WebUI (8080)                    │
│  Port 11434 ────────→ Ollama API (11434)                   │
│  Port 8081 ─────────→ SearXNG (8080)                       │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Docker Container Network (local-ai)           │ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────────────┐ │ │
│  │  │ Ollama                                          │ │ │
│  │  │ 0.0.0.0:11434→11434/tcp                        │ │ │
│  │  │ Accessible as: http://ollama:11434 (internal)  │ │ │
│  │  └─────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────────────┐ │ │
│  │  │ Open WebUI                                      │ │ │
│  │  │ 0.0.0.0:3000→8080/tcp                          │ │ │
│  │  │ Accessible as: http://localhost:3000           │ │ │
│  │  └─────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────────────┐ │ │
│  │  │ SearXNG                                         │ │ │
│  │  │ 0.0.0.0:8081→8080/tcp                          │ │ │
│  │  │ Accessible as: http://searxng:8081 (internal)  │ │ │
│  │  └─────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Model Loading & Memory Flow

```
Startup:
    │
    ├─→ Container starts
    │   └─→ Initialize empty model cache
    │
    └─→ User selects model in Open WebUI
        └─→ Check ollama_data volume
            ├─→ Model exists? LOAD
            │   └─→ 🧠 Loaded into RAM
            │       └─→ Ready for inference
            │
            └─→ Model missing? PULL
                └─→ Download from ollama.ai
                    └─→ ~4GB for 7B model
                        └─→ Save to ollama_data volume
                            └─→ Load into RAM
                                └─→ Ready for inference

Parallel Model Management:
    ┌──────────────────────┐
    │ OLLAMA_MAX_LOADED_=2 │
    │ (config in compose)  │
    └──────────────────────┘
            │
            ├─→ Load Model A (4GB RAM)
            │
            ├─→ Load Model B (4GB RAM)
            │   └─→ Total: 8GB
            │
            └─→ Request Model C?
                └─→ Unload least-recently-used
                    └─→ Load Model C
```

---

## Data Persistence Layer

```
┌──────────────────────────────────────────────────────┐
│            Docker Named Volumes                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  📦 ollama_data                                     │
│  ├─ /root/.ollama/models/                          │
│  │  └─ manifests/                                  │
│  │     └─ llama2, mistral, neural-chat, etc.       │
│  │                                                  │
│  └─ Location: /var/lib/docker/volumes/            │
│                                                    │
│  💾 openwebui_data                                 │
│  ├─ /app/backend/data/                            │
│  │  ├─ users.db (User accounts & auth)           │
│  │  ├─ chats/ (Conversation histories)            │
│  │  ├─ documents/ (Uploaded PDFs)                 │
│  │  ├─ embeddings.db (RAG vector DB)              │
│  │  └─ system/ (Config & metadata)                │
│  │                                                  │
│  └─ Location: /var/lib/docker/volumes/            │
│                                                    │
└──────────────────────────────────────────────────────┘

Backup/Recovery:
    docker run --rm -v ollama_data:/data \
      -v $(pwd)/backup:/backup \
      alpine tar czf /backup/ollama_data.tar.gz -C /data .
```

---

## Typical Resource Usage

```
Service        │ CPU    │ Memory  │ Disk        │ Notes
───────────────┼────────┼─────────┼─────────────┼──────────────
Ollama         │ 50-90% │ 4-8GB   │ 4-40GB      │ Depends on model
Open WebUI     │ 5-15%  │ 500MB   │ 1-2GB       │ Lightweight
SearXNG        │ 10-30% │ 200MB   │ 100MB       │ Lightweight
───────────────┼────────┼─────────┼─────────────┼──────────────
TOTAL          │ ≤100%  │ ≤9GB    │ ≤42GB       │ Max usage


Idle (no queries):
  • Ollama: 0% CPU, 4GB RAM (model cached)
  • Open WebUI: 0% CPU, 100MB RAM
  • SearXNG: 0% CPU, 50MB RAM

During Inference:
  • Ollama: 80-100% CPU, 6-8GB RAM
  • Open WebUI: 5% CPU, 200MB RAM
  • SearXNG: 30% CPU, 150MB RAM (if searching)
```

---

## Environment Variables & Configuration

```
OLLAMA (Service Config)
├─ OLLAMA_NUM_PARALLEL=2
│  └─ Process multiple requests simultaneously
│
└─ OLLAMA_MAX_LOADED_MODELS=2
   └─ Keep max 2 models in memory

OPEN-WEBUI (Service Config)
├─ OLLAMA_BASE_URL=http://ollama:11434
│  └─ Internal network URL to Ollama
│
├─ ENABLE_SEARCH=true
│  └─ Enable web search feature toggle
│
├─ SEARXNG_BASE_URL=http://searxng:8081
│  └─ Internal network URL to SearXNG
│
├─ RAG_TOP_K=10
│  └─ Return top 10 relevant documents
│
├─ RAG_RELEVANCE_THRESHOLD=0.3
│  └─ Minimum relevance score (0-1)
│
└─ PDF_EXTRACT_IMAGES=true
   └─ Process images within PDFs

SEARXNG (Service Config)
├─ BASE_URL=http://localhost:8081/
│  └─ External access URL
│
├─ INSTANCE_NAME=Local-AI-Search
│  └─ Display name for search engine
│
└─ AUTOCOMPLETE=google
   └─ Search suggestion provider
```

---

## Error Handling & Recovery

```
Scenario: Service Crash

Open WebUI Down?
    │
    ├─→ Docker auto-restart (restart: unless-stopped)
    └─→ Reconnect within 10 seconds
        └─→ Data preserved in openwebui_data volume

Ollama Down?
    │
    ├─→ Docker auto-restart
    └─→ Reload model from ollama_data volume
        └─→ ~30-60 seconds to reload
            └─→ Open WebUI queues requests

Model Out of Memory?
    │
    ├─→ Reduce OLLAMA_MAX_LOADED_MODELS
    ├─→ Use smaller model (7B vs 70B)
    └─→ Close other applications

Disk Full?
    │
    └─→ docker system prune -a
        └─→ Remove unused images/containers
            (keeps volumes safe)
```

---

This architecture ensures:
- ✅ **Privacy**: All processing local
- ✅ **Scalability**: Models can be swapped
- ✅ **Reliability**: Auto-restart, health checks
- ✅ **Persistence**: Data survives container restarts
- ✅ **Modularity**: Services can be updated independently
