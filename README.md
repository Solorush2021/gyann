<div align="center">

# 🎓 Gyaan

### On-Device AI Classroom — running on a 0.8B model

**Cloud-grade AI teaching. Fully local. Zero egress. Zero cost per use.**

[![Model](https://img.shields.io/badge/LLM-0.8B%20Quantized-success.svg)](#-the-bet)
[![Target](https://img.shields.io/badge/Hardware-Snapdragon%20X%20Elite-red.svg)](#-benchmarks)
[![Offline](https://img.shields.io/badge/Runs-Offline%20%2F%20Air--gapped-black.svg)](#-privacy--cost)

</div>

## 🎯 The Problem

AI-powered education tools today carry three heavy costs:

| Pain | Reality |
|---|---|
| 💸 **Token cost** | Every generation bills a cloud API. Scale = expensive. |
| 🔓 **Privacy** | Student data, prompts, and content flow to third-party servers. Unacceptable for minors / regulated environments. |
| 📶 **Connectivity** | No internet = no lesson. Fails in classrooms, rural areas, offline labs. |

And "on-device AI" usually means **7B–8B models minimum** — too heavy for edge hardware, too hungry on battery.

## 💡 The Bet

> We tested in the harshest condition possible.
> **Not 8B. Not 4B. Not even 1B.**
> A **0.8B model, quantized to ~400 MB.**

The question: *can a sub-1B model, running fully on a Snapdragon X Elite, produce a genuinely useful, animated, narrated classroom?*

**Answer: yes — if you move the intelligence out of the model and into the software around it.**

---

## 🏗️ How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                    SNAPDRAGON X ELITE (laptop)                │
│                                                              │
│   Phone browser ──Wi-Fi──►  Next.js Server (:3000)           │
│   (thin client)                  │                           │
│                                  ▼                           │
│                          GenieX / llama.cpp (:18181)         │
│                                  │                           │
│                                  ▼                           │
│                     Qwen3-0.8B (Q-quant, ~400 MB)            │
│                                  │                           │
│                    ┌─────────────┴─────────────┐             │
│                    ▼                           ▼             │
│              Hexagon NPU                 Oryon CPU            │
│                                                              │
│   ✦ ALL inference is local. Zero cloud. Zero egress. ✦       │
└──────────────────────────────────────────────────────────────┘
```

**Type any topic → get a 5-scene animated lesson:**

1. 🎬 **Intro** — hooks the learner
2. 📋 **Components** — every part of the topic, none skipped
3. 🔬 **Deep dive** — how the parts connect
4. ❓ **Quiz** — checks understanding
5. 📝 **Summary** — key takeaways

Each scene renders with **teacher speech** (TTS), **spotlight** (focus dimming), and **laser pointer** (tracks the word being spoken).

---

## 📷 Screenshots & Demo

### Sample Screens

| Sample 1 | Sample 2 |
| :---: | :---: |
| ![Sample 1 Image](https://github.com/Solorush2021/gyann/raw/main/assets/sample1.png) | ![Sample 2 Image](https://github.com/Solorush2021/gyann/raw/main/assets/sample2.png) |

> 💡 *Note: To display your own screenshots, upload your image files as `sample1.png` and `sample2.png` into an `assets` folder at the root of the repository.*

### 🎥 Video Demo

See a live recording of Gyaan running fully locally on the Snapdragon X Elite:
👉 [Watch the Working Video on Google Drive](https://drive.google.com/your-video-link-here)

---

## 🧠 The Core IP — Making a 0.8B Smart

Small models forget instructions, mangle JSON, and can't do spatial reasoning. We solved this by **offloading intelligence from the model to deterministic code:**

| Technique | What it does | Impact |
|---|---|---|
| **Prompt shrinking** | Moved all pixel-math, SVG paths, height tables OUT of the LLM into TypeScript | 31 KB → ~3 KB prompt (**−90 %**) |
| **Grid-DSL** | Model emits semantic slots (`{role:"bullet", text:"..."}`), code renders pixels | ~12 tokens/element vs ~60 (**−80 %**) |
| **Fixed templates** | Small model stops inventing JSON structure — fills content only | Reliable output |
| **Call merging** | Content + teacher-actions combined into ONE call per scene | 2 calls → 1 (**−50 %**) |
| **JSON repair + retry** | 4-strategy repair parser + 2× retry on malformed output | Recovers ~25 % failure rate |
| **Rolling context** | Rules survive across scenes without blowing VRAM | Fits 4K–32K windows |
| **Role-based IDs** | Predictable element IDs let spotlight/laser target the right object | Animations always land |

> **Net result:** a 0.8B model does work that normally demands a 70B.

---

## ⚡ NPU Memory Mapping & SRAM Tiling (Flash Attention)

To maximize performance on the Snapdragon X Elite, the runtime leverages compiler-driven **SRAM Tiling** (the NPU equivalent of Flash Attention) rather than standard main-memory attention mechanisms.

### 1. Snapdragon X Elite Memory Layout

While main system RAM is shared, high-speed SRAM caches are private to each processing engine to bypass memory bandwidth bottlenecks:

| Hardware Component | Cache/SRAM Type | Capacity | Access / Scope |
|---|---|---|---|
| **System RAM** (LPDDR5x) | Main Memory | 16 GB - 64 GB | Shared (CPU, GPU, NPU) |
| **Oryon CPU** | L2 + L3 Cache | 42 MB | Private to CPU |
| **Adreno GPU** | GMEM Cache | ~2 MB - 4 MB | Private to GPU (Tiled rendering) |
| **Hexagon NPU** | **VTCM** (Vector Tightly Coupled Memory) | **~8 MB - 12 MB** | Private to NPU (Used for SRAM Tiling) |

### 2. How SRAM Tiling Works on the NPU

Unlike GPUs where developers manually invoke PyTorch operators like `flash_attn_func()`, the Snapdragon NPU handles this at the compiler level via the **QNN HTP (Hexagon Tensor Processor) Backend**:

* **Operator Fusion**: During model compilation (e.g., via Qualcomm AI Hub or the QNN converter), the compiler detects the standard Attention layer.
* **VTCM SRAM Tiling**: The compiler replaces standard matrix math operations with optimized HTP operators. These slice tensors into tiles that fit entirely inside the NPU's **8–12 MB VTCM** (on-chip high-speed SRAM), drastically reducing slow roundtrips to the shared LPDDR5x main memory.
* **Burst Mode**: When compiling or running manually, using the performance flag `--ht_performance_mode burst` ensures the runtime aggressively utilizes the VTCM SRAM.

---

## 📊 Benchmarks

### Inference performance on Snapdragon X Elite

| Compute Target | Tokens/sec | Power Draw | Best For |
|:---:|:---:|:---:|---|
| **Hexagon NPU** (w4a16) | **~30 tok/s** ⚑ | **~8–12 W** ⚒ | Long sessions, battery-sensitive, always-on |
| **Oryon CPU** | **~20 tok/s** ⚑ | **~25–35 W** ⚒ | Sustained throughput, plugged-in demo |

> ⚑ *Measured on-device with Qwen3-0.8B Q-quantized via GenieX. Final numbers vary with model variant, quantization, and runtime (`qairt` vs `llama_cpp`). See [GenieX](https://github.com/qualcomm/GenieX) for the NPU stack.*
>
> ⚒ *Power draw is estimated from typical Snapdragon X Elite NPU/CPU envelopes under LLM inference load; not instrumented per-run. Measure with your device's power telemetry for precise figures.*

### Why we chose CPU for live demos
- Higher sustained throughput → faster classroom generation
- NPU reserved for the **low-power, long-session** roadmap (battery + thermal)

### Efficiency vs. the cloud-native pipeline

| Metric | Cloud (original) | Gyaan (on-device) | Savings |
|---|---|---|---|
| Slide prompt size | 31 KB (~8,870 tok) | ~3 KB (~850 tok) | **−90 %** |
| LLM calls per slide | 2 | 1 | **−50 %** |
| Output tokens / element | ~60 | ~12 | **−80 %** |
| Per-deck cost | paid API | **$0** | **−100 %** |
| Data egress | full prompts + PII | **none** | **−100 %** |
| Min model size | 70 B | **0.8 B** | **~89× smaller** |

---

## 🛠️ Tech Stack

| Layer | Choice |
|---|---|
| **App** | Next.js 16 · React 19 · TypeScript · Tailwind v4 · shadcn/ui |
| **Animation** | Motion (Framer successor) · Web Audio API |
| **Local LLM server** | [GenieX](https://github.com/qualcomm/GenieX) (Qualcomm) · llama.cpp fallback |
| **Model** | Qwen3-0.8B-Instruct, Q-quantized (~400 MB) |
| **Context** | 4K (NPU static graph) → 32K (CPU rolling) |
| **Target HW** | Snapdragon X Elite — Hexagon NPU + Adreno GPU + Oryon CPU |

---

## 📱 Phone-as-Thin-Client

The model never runs on the phone. The laptop is the server; any phone on the same Wi-Fi opens the UI.

```
Phone A (hotspot) ◄── provides Wi-Fi
   ├── Snapdragon laptop  → runs Gyaan + GenieX + model
   └── Phone B (viewer)   → opens http://<laptop-ip>:3000
```

- ✅ Zero compute on the phone
- ✅ Works over a phone hotspot (no router needed)
- ✅ Model + student data stay on the laptop

---

## 🔒 Privacy & Cost

| | Gyaan | Cloud AI apps |
|---|---|---|
| Tokens billed | **0** | per-use |
| Data leaves device | **never** | always |
| Works offline | ✅ | ❌ |
| GDPR / data-residency | clean by design | requires DPA |
| Air-gapped deployment | ✅ | ❌ |

> **The smallest attack surface in AI: nothing to exfiltrate.**

---

## 🚀 Setup (Snapdragon X Elite, Windows ARM64)

```powershell
# 1. Install Node.js LTS (winget auto-selects arm64)
winget install OpenJS.NodeJS.LTS

# 2. Unzip the source
Expand-Archive gyaan-dragon.zip C:\Users\you\gyaan
cd C:\Users\you\gyaan

# 3. Install deps (npm, not pnpm — pnpm arm64 is buggy)
npm install

# 4. Install + start GenieX with the Qwen 4B NPU model
#    https://github.com/qualcomm/GenieX/releases
geniex pull ai-hub-models/Qwen3-4B-Instruct-2507
geniex serve --host 0.0.0.0 --port 18181

# 5. Configure the app
copy .env.example .env.local
#   edit .env.local: OPENAI_BASE_URL=http://127.0.0.1:18181/v1
#                    DEFAULT_MODEL=openai:ai-hub-models/Qwen3-4B-Instruct-2507
#                    GYAAN_GRID_DSL=true

# 6. Run
npm run dev -- --hostname 0.0.0.0

# 7. Open firewall for phone access (admin PowerShell)
New-NetFirewallRule -DisplayName "Gyaan 3000" -Direction Inbound `
  -Protocol TCP -LocalPort 3000 -Action Allow -Profile Private
```

Open **`http://localhost:3000`** on the laptop, or **`http://<laptop-lan-ip>:3000`** on your phone.

> ⚠️ If Turbopack errors on ARM64, set `$env:TURBOPACK=0` and re-run.

---

## 🗺️ Roadmap

- [ ] NPU-native 4B via Qualcomm AI Hub (`w4a16` bundle)
- [ ] 3D / simulation / code scenes (when 7B+ is affordable on-device)
- [ ] mTLS + reverse proxy (Caddy) for hardened LAN exposure
- [ ] Qualcomm SPU-backed signing keys for client certs
- [ ] Kokoro neural TTS server-side (reliable phone audio)

---


<div align="center">

**Built for the edge. Runs on the least hardware possible.**

*Privacy by architecture — not by promise.*

</div>
