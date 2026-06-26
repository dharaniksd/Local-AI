# 🤖 Local-AI - ChatGPT-like Local Setup

A complete local AI setup running **Ollama** (LLM), **Open WebUI** (Chat Interface), and **SearXNG** (Web Search) using Docker.

## 🎯 Features

- 🧠 **Ollama**: Run large language models locally (Llama2, Mistral, Neural-Chat, etc.)
- 💬 **Open WebUI**: Beautiful, responsive chat interface with RAG (Retrieval-Augmented Generation)
- 🔍 **SearXNG**: Privacy-focused web search engine
- 🎨 **Stable Diffusion**: Generate and edit images locally with SDXL & 1.5
- 🔐 **User Authentication**: Secure multi-user support
- 📚 **RAG Support**: Upload PDFs and documents for context-aware answers
- 🌐 **Web Search Integration**: Get latest information directly in chat
- 🖼️ **Image Editing**: Inpaint, upscale, and enhance images

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

| Service | URL | Purpose |
|---------|-----|---------|
| **Open WebUI** | http://localhost:3000 | Chat interface |
| **Ollama API** | http://localhost:11434 | LLM backend (API) |
| **SearXNG** | http://localhost:8081 | Web search engine |
| **Stable Diffusion** | http://localhost:7860 | Image generation |

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

### Download Stable Diffusion Models

Once Stable Diffusion WebUI is running, download models directly in the web interface:

1. Go to http://localhost:7860
2. Click **Model** tab
3. Select from available models:
   - **Stable Diffusion XL** (1.0): Best quality (~7GB)
   - **Stable Diffusion 1.5**: Fast and reliable (~4GB)
   - **Proteus v0.2**: Fast & creative (~4GB)

Or download via command line:
```bash
# Automatic downloads on first use
# Models stored in docker volume: sd_models

# View available space
docker exec stable-diffusion-webui df -h
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

### Generate Images (Stable Diffusion)
1. Go to http://localhost:7860
2. Enter your image description in the prompt box
3. Click **Generate** to create images
4. Adjust settings:
   - **Steps**: 20-50 (more = higher quality but slower)
   - **Guidance Scale**: 7-15 (how strictly to follow prompt)
   - **Seed**: Leave blank for random, or set for reproducibility
5. Download generated images from the output panel

### Image Editing Features
- **Inpaint**: Modify parts of existing images
- **Upscale**: Enlarge images with AI enhancement
- **Face Fix**: Improve facial details with GFPGAN
- **Batch Processing**: Generate multiple variations

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

### Stable Diffusion Configuration
Optimize image generation in `docker-compose.yml`:
```bash
environment:
  - GRADIO_QUEUE_SIZE=32        # Max concurrent requests
  - GRADIO_MAX_SIZE=200         # Max upload size (MB)
  - CUDA_VISIBLE_DEVICES=0      # GPU selection (0=first GPU, -1=CPU only)
  
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
docker-compose logs -f stable-diffusion-webui

# Restart a service
docker-compose restart open-webui
docker-compose restart stable-diffusion-webui

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

### Stable Diffusion won't start
```bash
# Check logs
docker logs stable-diffusion-webui

# Verify GPU (if using NVIDIA)
docker exec stable-diffusion-webui nvidia-smi

# Check disk space (models need 5-10GB)
docker exec stable-diffusion-webui df -h

# For GPU issues, switch to CPU mode:
# Edit docker-compose.yml: CUDA_VISIBLE_DEVICES=-1
```

### Image generation is slow
- First generation takes time to load models
- Use a smaller model (Stable Diffusion 1.5 vs XL)
- Reduce Steps from 50 to 20
- Enable GPU acceleration (NVIDIA CUDA)
- Close other heavy applications

### VRAM/Memory errors
- Reduce batch size in Stable Diffusion settings
- Use smaller model
- Enable CPU offloading in advanced options
- Monitor with: `docker stats`

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
