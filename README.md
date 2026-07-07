# 🤖 Local-AI - ChatGPT-like Local Setup

A complete local AI setup running **Ollama** (LLM), **Open WebUI** (Chat Interface), and **SearXNG** (Web Search) using Docker.

## 🎯 Features

- 🧠 **Ollama**: Run large language models locally (Llama2, Mistral, Neural-Chat, etc.)
- 💬 **Open WebUI**: Beautiful, responsive chat interface with RAG (Retrieval-Augmented Generation)
- 🔍 **SearXNG**: Privacy-focused web search engine
- 🔐 **User Authentication**: Secure multi-user support
- 📚 **RAG Support**: Upload PDFs and documents for context-aware answers
- 🌐 **Web Search Integration**: Get latest information directly in chat
- 🎨 **ComfyUI** (Optional): Stable Diffusion for local image generation (see [IMAGE_GENERATION.md](IMAGE_GENERATION.md) for setup)

## 📋 Requirements

- **Docker**: [Download Docker Desktop](https://www.docker.com/products/docker-desktop)
- **Docker Compose**: Comes with Docker Desktop
- **RAM**: Minimum 8GB (16GB+ recommended for better performance)
- **Disk Space**: 10GB+ for models and volumes
- **GPU** (Recommended): 
  - NVIDIA GPU with CUDA support for faster image generation
  - Without GPU: CPU-only mode (slower, but still works)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone git@github.com:dharaniksd/Local-AI.git
cd Local-AI
```

### 2. Start Services
```bash
docker-compose up -d
```

### 3. Wait for Services to Initialize
```bash
# Monitor logs
docker-compose logs -f

# Check service status
docker-compose ps
```

Services should be healthy within 30-40 seconds.

### 4. Access the Interfaces

| Service | URL | Purpose | Status |
|---------|-----|---------|--------|
| **Open WebUI** | http://localhost:3000 | Chat interface with RAG & search | ✅ Active |
| **Ollama API** | http://localhost:11434 | LLM backend (API) | ✅ Active |
| **SearXNG** | http://localhost:8081 | Web search engine | ✅ Active |
| **ComfyUI** (Optional) | http://localhost:8188 | Image generation | ⏸️ Disabled by default |

**To enable ComfyUI (image generation):**
See [IMAGE_GENERATION.md](IMAGE_GENERATION.md) for detailed setup instructions.

## 📥 Pull Your First Model

Once Open WebUI is running, pull a language model:

```bash
# Pull Llama 2 (7B - ~4GB)
docker exec ollama ollama pull llama2

# Or try other models:
# - Mistral (7B - ~4GB, faster, efficient)
docker exec ollama ollama pull mistral

# - Neural-Chat (13B - ~8GB, conversational)
docker exec ollama ollama pull neural-chat

# - Dolphin (70B - ~40GB, most capable)
docker exec ollama ollama pull dolphin
```

Check available models:
```bash
docker exec ollama ollama list
```

## 🎮 How to Use

### Chat with Local Models
1. Go to http://localhost:3000
2. Login with your credentials (or create new account on first visit)
3. Select a model from the dropdown
4. Start chatting!

### Enable Web Search
1. In the chat interface, enable the search toggle
2. Your queries will fetch latest information from the web
3. Responses include both model knowledge and web search results

### Upload Documents (RAG)
1. Click the **📎 Attachment** button
2. Upload PDFs, TXT, or other documents
3. The model will reference these documents for context-aware answers

### Generate Images (ComfyUI)
1. Go to http://localhost:8188
2. First run will download required model files (may take a few minutes on first startup)
3. **Using ComfyUI:**
   - Use "Load Default" workflow or create your own with nodes
   - Connect nodes: CheckpointLoader → KSampler → VAEDecode → SaveImage
   - Enter your prompt in the positive conditioning node
   - Click "Queue Prompt" to generate
   - Generated images saved to Docker volume `sd_outputs`

**Popular Models to Try:**
- **Stable Diffusion 1.5**: Fast, reliable quality
- **Stable Diffusion XL**: Better quality but slower  
- **Proteus v0.2**: Fast and creative

**Models installed automatically on first run.**

For more details, see [IMAGE_GENERATION.md](IMAGE_GENERATION.md)

## 🛠️ Advanced Configuration

### Edit Ollama Settings
```bash
# Modify parallelization and model limits in docker-compose.yml
environment:
  - OLLAMA_NUM_PARALLEL=2      # Parallel request processing
  - OLLAMA_MAX_LOADED_MODELS=2 # Max models in memory simultaneously
```

### Customize Open WebUI
```bash
# Additional environment variables available:
- RAG_TOP_K=10                    # Search top K documents
- RAG_RELEVANCE_THRESHOLD=0.3     # Relevance cutoff (0-1)
- PDF_EXTRACT_IMAGES=true        # Extract images from PDFs
```

### SearXNG Configuration
Modify search engine settings in `docker-compose.yml`:
```bash
environment:
  - BASE_URL=http://localhost:8081/
  - INSTANCE_NAME=Local-AI-Search
  - AUTOCOMPLETE=google
```

### ComfyUI (Stable Diffusion) Configuration
Optimize image generation in `docker-compose.yml`:
```bash
environment:
  - TORCH_DEVICE=cpu             # Force CPU mode
  - COMFYUI_FORCE_CPU=true       # Ensure CPU usage
  - CUDA_VISIBLE_DEVICES=        # Empty = CPU only (set to 0 for GPU)
```
# For CPU-only mode:
  - CUDA_VISIBLE_DEVICES=-1
```

## 📊 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      User Browser                            │
│                  http://localhost:3000                       │
└────────────────────────────┬────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Open WebUI    │
                    │  (Docker Port   │
                    │    3000→8080)   │
                    └────────┬────────┘
                             │
                ┌────────────┼────────────┐
                │            │            │
        ┌───────▼──────┐ ┌──▼──────┐ ┌───▼────────┐
        │    Ollama    │ │ SearXNG │ │ Local Data │
        │  (Port 11434)│ │(Port 81)│ │  Volumes   │
        │   LLM Models │ │  Search │ │  (PDFs,    │
        │  (Llama2,    │ │ Engine  │ │  Configs)  │
        │  Mistral)    │ │         │ │            │
        └──────────────┘ └─────────┘ └────────────┘
                │              │
        ┌───────▼──────────────▼──────┐
        │    Docker Network: local-ai │
        │  (All services connected)   │
        └────────────────────────────┘
```

## 🔄 Service Flow

```
1. User Query
   ↓
2. Open WebUI receives input
   ↓
3. If search enabled → Query SearXNG for latest info
   ├→ Search results returned
   ↓
4. Combine user query + search results + uploaded documents (RAG)
   ↓
5. Send to Ollama API
   ↓
6. Ollama loads selected model & generates response
   ↓
7. Response displayed in Open WebUI
   ↓
8. Conversation saved in database
```

## 📦 Docker Volumes

Persistent data is stored in these volumes:

| Volume | Purpose | Location |
|--------|---------|----------|
| `ollama_data` | Downloaded LLM models | `/root/.ollama` |
| `openwebui_data` | Chats, users, uploads | `/app/backend/data` |

## ⚙️ Common Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f open-webui
docker-compose logs -f ollama
docker-compose logs -f searxng

# Restart a service
docker-compose restart open-webui

# Check service health
docker-compose ps

# Pull a new model
docker exec ollama ollama pull <model-name>

# List available models
docker exec ollama ollama list

# Check Stable Diffusion logs
docker logs stable-diffusion-webui

# Remove all containers (WARNING: keeps volumes)
docker-compose down

# Full cleanup (removes containers & volumes - CAUTION!)
docker-compose down -v

# Monitor resource usage
docker stats
```

## 🐛 Troubleshooting

### Open WebUI won't start
```bash
docker-compose logs open-webui
# Check if Ollama is healthy first
docker-compose logs ollama
```

### Can't connect to Ollama
- Ensure Ollama is running: `docker-compose ps`
- Check Ollama is healthy: `curl http://localhost:11434/api/tags`
- Verify network: `docker network ls | grep local-ai`

### SearXNG warnings (ahmia, torch errors)
- ✅ **Normal** - Optional engines failing to load
- Models will still search using other engines
- Safe to ignore

### Slow responses
- Models need time to load first time
- Check available RAM: `docker stats`
- Consider smaller models (mistral, neural-chat)

## 🎨 Image Generation Setup

The Stable Diffusion WebUI is included and runs in Docker via AUTOMATIC1111. 

**First Time Setup:**
1. Start services: `docker-compose up -d`
2. Wait 2-3 minutes for Stable Diffusion to initialize
3. Visit http://localhost:7860
4. Download a model (will take 10-30 minutes first time)
5. Start generating! 🎨

**Performance Notes:**
- **CPU Mode**: ~2-3 minutes per image (good for testing)
- **GPU Mode**: ~30-60 seconds (requires NVIDIA GPU or Apple Silicon)
- First generation is slowest (model loading)

For detailed setup and advanced options, see [Image Generation Guide](IMAGE_GENERATION.md)

## 📚 Recommended Models

| Model | Size | Speed | Quality | Use Case |
|-------|------|-------|---------|----------|
| **Mistral** | 7B | ⚡⚡⚡ | ⭐⭐⭐⭐ | Fast, efficient, recommended |
| **Llama2** | 7B/13B | ⚡⚡ | ⭐⭐⭐⭐ | Balanced, reliable |
| **Neural-Chat** | 13B | ⚡ | ⭐⭐⭐⭐⭐ | Conversational, best quality |
| **Dolphin** | 70B | 🐌 | ⭐⭐⭐⭐⭐⭐ | Most capable, slow |

## 🔐 Security

- **Local Processing**: All data stays on your machine
- **No API Keys**: Doesn't use external APIs (except optional web search)
- **User Auth**: Supports multiple users with authentication
- **Private**: No data sent to OpenAI or other cloud services

## 📖 Additional Resources

- [Ollama Docs](https://github.com/ollama/ollama)
- [Open WebUI Docs](https://docs.openwebui.com/)
- [SearXNG Docs](https://docs.searxng.org/)
- [📊 Image Generation Architecture](IMAGE_GENERATION.md) - Detailed Stable Diffusion setup & flows

## 🤝 Contributing

Feel free to fork, modify, and improve!

## 📄 License

MIT License - Use freely for personal or commercial projects.

---

**Made with ❤️ for local AI enthusiasts** 🚀
