# 🎨 Stable Diffusion Image Generation - Architecture & Flow

## Image Generation Service Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                     🖥️  USER BROWSER                            │
│                   (localhost:7860)                              │
│                                                                  │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │                 │
                    │ Stable Diffusion│
                    │    WebUI        │
                    │   (Gradio)      │
                    │                 │
                    │ ✓ Text-to-Image │
                    │ ✓ Image Inpaint │
                    │ ✓ Upscaling     │
                    │ ✓ Face Restore  │
                    │                 │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
        ┌─────▼────────┐ ┌──▼──────────┐ ┌▼─────────────┐
        │   Models     │ │  VAE/Encoder│ │   Samplers   │
        │   (SDXL,     │ │  (Compression)│ │   (DPM,     │
        │   1.5)       │ │   │           │    Euler)    │
        │   (~7GB)     │ │   │           │              │
        └─────┬────────┘ └──┬──────────┘ └──────────────┘
              │             │
        ┌─────▼─────────────▼──────────┐
        │                              │
        │  🌉 Docker Network           │
        │     (local-ai)               │
        │                              │
        └─────────────────────────────┘
                    │
        ┌───────────▼────────────┐
        │                        │
        │  💾 Data Volumes       │
        │                        │
        │ • sd_models/           │
        │   Models & Weights     │
        │                        │
        │ • sd_outputs/          │
        │   Generated images     │
        │   & history            │
        │                        │
        └────────────────────────┘
```

---

## Text-to-Image Generation Flow

```
USER PROMPT
    │
    ▼
┌───────────────────────────────────┐
│ User enters prompt in WebUI       │
│ "A beautiful sunset over ocean"   │
└─────────────┬─────────────────────┘
              │
              ▼
    ┌─────────────────────┐
    │ Adjust Settings     │
    │ • Steps: 20-50      │
    │ • Guidance: 7-15    │
    │ • Sampler: DPM++    │
    │ • Seed: Random      │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Load Model          │
    │ (e.g., SDXL)        │
    │ From sd_models vol. │
    │ ~7GB into VRAM      │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Tokenize Prompt     │
    │                     │
    │ "beautiful sunset"  │
    │ → Tokens → IDs      │
    │ → Embeddings        │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ CLIP Encoder        │
    │ (Text Understanding)│
    │ Prompt → Vector     │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Diffusion Sampling  │
    │ (Noise Reduction)   │
    │                     │
    │ Noise → Image       │
    │ (Steps iterations)  │
    │                     │
    │ ~30-60 seconds      │
    │ (depends on steps)  │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ VAE Decode          │
    │ (Latent → Pixel)    │
    │                     │
    │ Compress form       │
    │ → Full resolution   │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Post-Process        │
    │ • Upscale (optional)│
    │ • Face fix (optional)
    │ • Format convert    │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Save Output         │
    │ → sd_outputs vol.   │
    │ PNG + metadata      │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Display in UI       │
    │ • Show image        │
    │ • Save button       │
    │ • Edit button       │
    └────────┬────────────┘
             │
             ▼
        ✅ COMPLETE
```

---

## Image Inpaint (Edit) Flow

```
UPLOAD IMAGE + MASK
    │
    ├─ User selects "Inpaint" mode
    ├─ Uploads reference image
    └─ Paints mask on region to modify
             │
             ▼
    ┌─────────────────────┐
    │ Load Base Image     │
    │ Parse mask region   │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ VAE Encode          │
    │ Image → Latents     │
    │ Compress form       │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Mask Processing     │
    │ • Blur edges        │
    │ • Expand region     │
    │ • Create latent mask│
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Diffusion Inpaint   │
    │ • Preserve outside  │
    │ • Regenerate inside │
    │ • Blend seamlessly  │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ VAE Decode          │
    │ Latents → Image     │
    └────────┬────────────┘
             │
             ▼
    ┌─────────────────────┐
    │ Blend Edges         │
    │ Smooth transitions  │
    └────────┬────────────┘
             │
             ▼
        ✅ EDITED IMAGE
```

---

## Upscaling & Enhancement Pipeline

```
INPUT IMAGE (512x512)
    │
    ├─ Option 1: REAL-ESRGAN (4x upscale)
    │     │
    │     └─→ Denoise + Upscale
    │         → 2048x2048 output
    │
    └─ Option 2: GFPGAN (Face restoration)
          │
          └─→ Detect faces
              → Enhance facial features
              → Blend back into image

UPSCALED OUTPUT (2048x2048)
```

---

## Container Resource Allocation

```
Stable Diffusion Service

Without GPU (CPU only):
    • Speed: 3-5 min per image
    • Memory: 8-10GB RAM
    • CPU: 100% for duration
    
With NVIDIA GPU (CUDA):
    • Speed: 30-60 sec per image (12GB GPU)
    • Memory: 4-6GB RAM + 8-12GB VRAM
    • CPU: 20-30% (coordinate work)
    • Speedup: 4-8x faster

Model Variants by Memory:
    
    SDXL (Best Quality)
    ├─ GPU: 12GB VRAM minimum
    ├─ RAM: 6GB minimum
    ├─ Speed: 45-60s per image
    └─ Quality: ⭐⭐⭐⭐⭐

    Stable Diffusion 1.5 (Balanced)
    ├─ GPU: 6GB VRAM
    ├─ RAM: 4GB minimum
    ├─ Speed: 20-30s per image
    └─ Quality: ⭐⭐⭐⭐

    TinyDiffusion (Minimal)
    ├─ GPU: 2GB VRAM
    ├─ RAM: 2GB minimum
    ├─ Speed: 10-15s per image
    └─ Quality: ⭐⭐⭐
```

---

## Multi-Model Management

```
Model Switching Flow:

User selects different model
    │
    ▼
┌─────────────────────────┐
│ Unload current model    │
│ Free VRAM               │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Check sd_models volume  │
│ For selected model      │
└────────┬────────────────┘
         │
    ┌────┴─────────────┐
    │                  │
    ▼                  ▼
[Model Found]   [Model Missing]
    │                  │
    │            ┌─────▼─────┐
    │            │ Download  │
    │            │ from hub  │
    │            │ (~30 min) │
    │            └─────┬─────┘
    │                  │
    └────┬─────────────┘
         │
         ▼
┌─────────────────────────┐
│ Load to VRAM            │
│ Ready for inference     │
└─────────────────────────┘
```

---

## Storage & Volume Management

```
sd_models volume:
├─ stable-diffusion-xl-1.0/
│  ├─ model.fp16.safetensors (~7GB)
│  ├─ vae.safetensors (~500MB)
│  └─ metadata.json
│
├─ stable-diffusion-1.5/
│  ├─ model.safetensors (~4GB)
│  └─ vae.safetensors (~500MB)
│
└─ checkpoints/ (symlink to models)

sd_outputs volume:
├─ 2024-01-15/ (date folder)
│  ├─ image_0.png
│  ├─ image_0.txt (metadata)
│  ├─ image_1.png
│  └─ image_1.txt
│
├─ favorites/ (user saved images)
└─ history.json (generation log)

Total space needed:
- Minimal (1 model): 5GB
- Standard (2 models): 12GB
- Full setup (3+ models): 20+GB
```

---

## Performance Tuning

```
For Faster Generation:

1. Reduce Steps
   ├─ Default: 50 steps
   ├─ Fast: 20 steps
   └─ Trade-off: Less detail

2. Use Smaller Model
   ├─ SDXL → SD 1.5
   └─ Trade-off: Lower quality

3. Enable Optimizations
   ├─ Memory efficient attention
   ├─ VAE tiling
   ├─ Use fp16 precision
   └─ Trade-off: Slight quality loss

4. Batch Processing
   └─ Generate multiple images
      in parallel (if VRAM > 10GB)

5. GPU Acceleration
   └─ NVIDIA CUDA 10-100x faster
      AMD ROCm support
```

---

## Quality vs Speed Trade-offs

```
┌──────────────────────────────────────────────────────┐
│       Configuration      │  Speed  │  Quality  │ VRAM │
├──────────────────────────────────────────────────────┤
│ SDXL + 50 Steps          │  ⏱️ 60s │ ⭐⭐⭐⭐⭐ │ 12GB │
│ SDXL + 30 Steps          │  ⏱️ 40s │ ⭐⭐⭐⭐⭐ │ 12GB │
│ SDXL + 20 Steps          │  ⏱️ 30s │ ⭐⭐⭐⭐  │ 12GB │
│ SD 1.5 + 50 Steps        │  ⏱️ 30s │ ⭐⭐⭐⭐  │  6GB │
│ SD 1.5 + 30 Steps        │  ⏱️ 20s │ ⭐⭐⭐⭐  │  6GB │
│ SD 1.5 + 20 Steps        │  ⏱️ 15s │ ⭐⭐⭐   │  6GB │
│ Tiny Model + 20 Steps    │  ⏱️ 10s │ ⭐⭐⭐   │  2GB │
└──────────────────────────────────────────────────────┘
```

---

## Error Recovery & Troubleshooting

```
CUDA Out of Memory
    │
    ├─ Reduce batch size
    ├─ Use smaller model
    ├─ Enable VAE tiling
    └─ Switch to CPU mode

Model Download Failed
    │
    ├─ Check disk space: docker exec sd ... df -h
    ├─ Check internet
    ├─ Retry download
    └─ Manual model download

Generation Hangs
    │
    ├─ Check Docker logs
    ├─ Verify GPU working: nvidia-smi
    ├─ Restart container
    └─ Reduce complexity (lower steps)

Poor Image Quality
    │
    ├─ Increase steps (20→50)
    ├─ Improve prompt description
    ├─ Try different sampler
    └─ Use better model (SD1.5→SDXL)
```

---

## Integration with Open WebUI

```
Potential Future Integration:

1. Image-to-Text in Chat
   • Generate image from text in Open WebUI
   • Embed images in conversation
   • Multi-modal conversation

2. Combined Workflows
   • Chat generates description
   • Auto-generate images
   • Display side-by-side

3. Cross-Service Communication
   • Both services on local-ai network
   • Shared volumes for images
   • Event-driven generation
```

---

This Stable Diffusion setup provides:
- ✅ **Local Processing**: No cloud API calls
- ✅ **Privacy**: All images stay on your machine
- ✅ **Flexibility**: Multiple model support
- ✅ **Extensibility**: Ready for integration with Ollama
- ✅ **Performance**: GPU acceleration when available
