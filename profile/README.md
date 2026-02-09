# OminiX-AI

**A full-stack, pure-Rust AI platform for Apple Silicon — from GPU kernels to desktop UI.**

Part of the [Moxin Organization](https://github.com/moxin-org) open-source AI ecosystem.

[![Rust Version](https://img.shields.io/badge/Rust-1.82.0+-blue)](https://releases.rs/docs/1.82.0)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue)](LICENSE)

---

## What is OminiX?

OminiX is an open-source ecosystem for running multimodal AI **entirely on-device** on Apple Silicon Macs. It covers the full vertical: low-level ML inference via Apple's MLX framework, an OpenAI-compatible API server, and a native cross-platform desktop application — all written in Rust with zero Python dependencies at runtime.

The goal is to give developers and end-users a single, cohesive platform where they can chat with LLMs, generate images, clone voices, and transcribe audio — all running locally on their M1/M2/M3/M4 hardware, with the option to connect to cloud providers as well.

The three OminiX repositories form a complete stack:

```
┌──────────────────────────────────────────────────────────────┐
│                      OminiX-Studio                           │
│           Native desktop app (Rust + Makepad)                │
│       Chat · Image Gen · Voice · Transcription               │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                       OminiX-API                             │
│         OpenAI-compatible HTTP/WebSocket server              │
│     /v1/chat  ·  /v1/audio  ·  /v1/images  ·  ws/tts         │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                       OminiX-MLX                             │
│      Pure-Rust inference crates on Apple MLX (Metal GPU)     │
│   LLMs · ASR · TTS · Image Gen · VLMs                        │
└──────────────────────────────────────────────────────────────┘
```

---

## Repositories

### [`OminiX-MLX`](https://github.com/OminiX-ai/OminiX-MLX) — ML Inference Engine

The foundation layer. A comprehensive Rust crate ecosystem for running ML models on Apple Silicon via Apple's [MLX](https://github.com/ml-explore/mlx) framework.

**What it provides:**

- **`mlx-rs`** — Safe Rust bindings to Apple's MLX C++ library (array ops, neural network layers, transforms, JIT)
- **`mlx-rs-core`** — Shared inference infrastructure: KV cache management, RoPE embeddings, scaled dot-product attention, audio processing, Metal kernels, speculative decoding
- **14+ model crates** covering four modalities:

| Modality | Crates | Models |
|----------|--------|--------|
| **LLM** | `qwen3-mlx`, `glm4-mlx`, `glm4-moe-mlx`, `mixtral-mlx`, `mistral-mlx` | Qwen 2/3 (0.5B–235B), GLM-4, GLM-4-MoE, Mixtral 8x7B/8x22B, Mistral 7B |
| **ASR** | `funasr-mlx`, `funasr-nano-mlx`, `funasr-qwen4b-mlx` | Paraformer-large (18× real-time), SenseVoice + Qwen |
| **TTS** | `gpt-sovits-mlx` | GPT-SoVITS few-shot voice cloning (4× real-time) |
| **Image** | `flux-klein-mlx`, `zimage-mlx`, `qwen-image-mlx` | FLUX.2-klein, Z-Image-Turbo (~3s/image) |

All models run with Metal GPU acceleration, unified memory (zero-copy CPU↔GPU), and lazy evaluation with automatic kernel fusion.

---

### [`OminiX-API`](https://github.com/OminiX-ai/OminiX-API) — API Server

An OpenAI-compatible API server that wraps OminiX-MLX into a drop-in replacement for the OpenAI API. Built with [Salvo](https://salvo.rs/) (Rust HTTP framework).

**Key features:**

- **Drop-in OpenAI compatibility** — works with the official `openai` Python/JS client libraries out of the box
- **Four model slots** (LLM, ASR, TTS, Image) — one model per category, dynamically loadable/unloadable at runtime without restart
- **Actor-model architecture** — all MLX models run on a dedicated inference thread; HTTP/WebSocket handlers communicate via bounded async channels
- **Endpoints:**
  - `POST /v1/chat/completions` — LLM chat (Qwen3, Mistral, GLM)
  - `POST /v1/audio/transcriptions` — speech-to-text (Paraformer)
  - `POST /v1/audio/speech` — text-to-speech with voice cloning (GPT-SoVITS)
  - `POST /v1/images/generations` — text-to-image and image-to-image (FLUX, Z-Image)
  - `WebSocket /ws/v1/tts` — streaming TTS with per-message voice switching (MiniMax T2A protocol)
  - `POST /v1/models/load|unload` — dynamic model management
- **Voice registry** — named voices with aliases, pre-computed semantic codes for high-quality TTS, few-shot and zero-shot modes
- **Voice cloning training** — train new voices from reference audio via a 5-stage pipeline

---

### [`OminiX-Studio`](https://github.com/OminiX-ai/OminiX-Studio) — Desktop Application

A native cross-platform desktop AI application built in Rust using the [Makepad](https://github.com/makepad/makepad) UI toolkit, evolving from the [Moly](https://github.com/moly-ai/moly-ai) AI super app client.

**What it provides:**

- Native GUI for chatting with local and cloud LLMs
- Multi-platform support (macOS, Windows, Linux) via Makepad's cross-platform rendering (Metal, DirectX, OpenGL, WebGL)
- Integration with local model backends (OminiX-API) and cloud-hosted OpenAI-compatible providers
- Model discovery, download, and management
- Modular architecture: `moly-shell` (app shell), `moly-widgets` (reusable UI components), `moly-data` (data layer)

---

## Performance

Benchmarks on Apple M3 Max (128GB):

| Task | Model | Throughput | Memory |
|------|-------|------------|--------|
| LLM | Qwen3-4B | 45 tok/s | 8 GB |
| LLM | GLM4-9B-4bit | 35 tok/s | 6 GB |
| LLM | Mixtral-8x7B-4bit | 25 tok/s | 26 GB |
| ASR | Paraformer | 18× real-time | 500 MB |
| TTS | GPT-SoVITS | 3–4× real-time | 2 GB |
| Image | Z-Image-Turbo | ~3s/image | 12 GB |
| Image | FLUX.2-klein | ~5s/image | 13 GB |

---

## Quick Start

### Prerequisites

- macOS 14.0+ (Sonoma)
- Apple Silicon (M1/M2/M3/M4)
- Rust 1.82+
- Xcode Command Line Tools

### Run an LLM locally via the API

```bash
# Clone the repos (API depends on MLX as a sibling directory)
git clone https://github.com/OminiX-ai/OminiX-MLX.git
git clone https://github.com/OminiX-ai/OminiX-API.git

# Start the server (model downloads automatically)
cd OminiX-API
LLM_MODEL=mlx-community/Qwen3-4B-bf16 cargo run --release
```

Then use it with any OpenAI-compatible client:

```python
import openai

client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="not-needed")
response = client.chat.completions.create(
    model="qwen3",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### Use MLX crates directly in Rust

```rust
use qwen3_mlx::{load_model, Generate, ConcatKeyValueCache};

let mut model = load_model("./models/Qwen3-4B")?;
let mut cache = Vec::new();
let generator = Generate::<ConcatKeyValueCache>::new(
    &mut model, &mut cache, 0.7, &prompt_tokens
);

for token in generator.take(100) {
    print!("{}", tokenizer.decode(&[token?.item::<u32>()], true)?);
}
```

---

## Why Pure Rust?

The entire stack — from FFI bindings to Metal GPU kernels to the HTTP server to the desktop UI — is written in Rust. This means:

- **No Python runtime** at inference time. No venvs, no pip, no GIL.
- **Single binary deployment.** `cargo build --release` produces a self-contained executable.
- **Memory safety** across the full stack, including GPU memory management.
- **Consistent performance.** No interpreter overhead, no garbage collection pauses.
- **Cross-compilation** to macOS, Windows, Linux, iOS, Android, and WebAssembly via Makepad.

---

## Architecture Highlights

**MLX Layer (`OminiX-MLX`):** Builds on Apple's MLX framework through `mlx-sys` (bindgen FFI) → `mlx-rs` (safe Rust API) → `mlx-rs-core` (shared inference primitives) → individual model crates. Each model crate is self-contained with its own tokenizer, weights loading, and generation logic.

**API Layer (`OminiX-API`):** Uses an actor model where a single inference thread owns all loaded models (MLX types aren't `Send`/`Sync`). HTTP handlers serialize requests through a bounded `mpsc` channel. This gives thread-safe concurrent access without locks on the GPU.

**UI Layer (`OminiX-Studio`):** Built with Makepad, a Rust UI framework that compiles to native Metal/DX11/OpenGL and WebGL/WASM. The app connects to local or remote OpenAI-compatible backends.

---

## Supported Models

| Category | Models | Formats |
|----------|--------|---------|
| **LLM** | Qwen 2/3, Qwen3-MoE, GLM-4, GLM-4-MoE, Mixtral, Mistral | bf16, 4-bit, 8-bit via HuggingFace |
| **ASR** | Paraformer-large, SenseVoice + Qwen | Safetensors |
| **TTS** | GPT-SoVITS (zero-shot, few-shot, pre-computed codes) | Custom weights |
| **Image** | FLUX.2-klein, Z-Image-Turbo | MLX format / Safetensors |
| **VLM** | Moxin-VLM | Experimental |

Models are loaded from HuggingFace Hub (auto-download) or local directories.

---

## License

All repositories are dual-licensed under **MIT** and **Apache 2.0**.

## Acknowledgments

- [Moxin Organization](https://github.com/moxin-org) for the broader AI ecosystem, open language models, and the Moly app
- [Apple MLX Team](https://github.com/ml-explore/mlx) for the MLX framework
- [Makepad](https://github.com/makepad/makepad) for the cross-platform Rust UI toolkit
- [Project Robius](https://github.com/project-robius) for multi-platform Rust app framework tooling
