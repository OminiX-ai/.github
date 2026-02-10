<p align="center">
  <img width="420" height="132" alt="OminiX" src="https://github.com/user-attachments/assets/bb4ca164-6b13-46d4-9e25-12649fc14fb9" />
</p>

<h3 align="center">A full-stack, pure-Rust AI platform for Apple Silicon.</h3>

<p align="center">
  Part of the <a href="https://github.com/moxin-org">Moxin Organization</a> open-source AI ecosystem.
</p>

<p align="center">
  <a href="https://github.com/OminiX-ai/OminiX-MLX">MLX Inference</a> · <a href="https://github.com/OminiX-ai/OminiX-API">API Server</a> · <a href="https://github.com/OminiX-ai/OminiX-Studio">Desktop App</a>
</p>

---

## Overview

OminiX is an open-source ecosystem for running multimodal AI **entirely on-device** on Apple Silicon Macs — LLMs, image generation, voice cloning, and speech recognition — all in Rust with zero Python dependencies at runtime.

```
┌──────────────────────────────────────────────────────────────┐
│  OminiX-Studio    Native desktop app (Rust + Makepad)        │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  OminiX-API       OpenAI-compatible HTTP/WebSocket server    │
└──────────────────────────┬───────────────────────────────────┘
┌──────────────────────────▼───────────────────────────────────┐
│  OminiX-MLX       Pure-Rust inference on Apple MLX (Metal)   │
└──────────────────────────────────────────────────────────────┘
```

---

## Repositories

| Repo | Description | Details |
|------|-------------|---------|
| [**OminiX-MLX**](https://github.com/OminiX-ai/OminiX-MLX) | Safe Rust bindings to Apple MLX + 14 model crates (LLM, ASR, TTS, Image Gen) | Qwen 2/3, GLM-4, Mixtral, Mistral, Paraformer, GPT-SoVITS, FLUX.2-klein, Z-Image |
| [**OminiX-API**](https://github.com/OminiX-ai/OminiX-API) | OpenAI-compatible API server wrapping OminiX-MLX | Drop-in `/v1/chat`, `/v1/audio`, `/v1/images`, WebSocket TTS, dynamic model loading |
| [**OminiX-Studio**](https://github.com/OminiX-ai/OminiX-Studio) | Native cross-platform desktop app built with Makepad | Chat, image gen, voice — connects to local or cloud backends |

---

## Quick Start

**Requirements:** macOS 14+, Apple Silicon (M1–M4), Rust 1.82+, Xcode CLI Tools.

### For End-Users — OminiX-Studio

```bash
# Clone all three repos (Studio depends on API, which depends on MLX)
git clone https://github.com/OminiX-ai/OminiX-MLX.git
git clone https://github.com/OminiX-ai/OminiX-API.git
git clone https://github.com/OminiX-ai/OminiX-Studio.git

# Start the local API server (model downloads automatically)
cd OminiX-API
LLM_MODEL=mlx-community/Qwen3-4B-bf16 cargo run --release &

# Launch the desktop app
cd ../OminiX-Studio
cargo run --release
```

A native desktop app for chatting with local and cloud LLMs, generating images, and more.

### For Developers — OminiX-API

```bash
git clone https://github.com/OminiX-ai/OminiX-MLX.git
git clone https://github.com/OminiX-ai/OminiX-API.git

cd OminiX-API
LLM_MODEL=mlx-community/Qwen3-4B-bf16 cargo run --release
```

This starts an OpenAI-compatible local server. Use it with any standard client:

```python
import openai

client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="not-needed")
response = client.chat.completions.create(
    model="qwen3",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

---
## Performance

Benchmarks on Apple M3 Max (128GB):

| Task | Model | Throughput | Memory |
|------|-------|------------|--------|
| LLM | Qwen3-4B | 45 tok/s | 8 GB |
| ASR | Paraformer | 18× real-time | 500 MB |
| TTS | GPT-SoVITS | 3–4× real-time | 2 GB |
| Image | Z-Image-Turbo | ~3s/image | 12 GB |

---

## Why Pure Rust?

No Python runtime, no venvs, no GIL. Single binary via `cargo build --release`. Memory safety across the full stack including GPU. Cross-compilation to macOS, Windows, Linux, iOS, Android, and WebAssembly via Makepad.

---

## License

Dual-licensed under **MIT** and **Apache 2.0**.

## Acknowledgments

- [Moxin Organization](https://github.com/moxin-org) — broader AI ecosystem, open language models, and the Moly app
- [Apple MLX Team](https://github.com/ml-explore/mlx) — MLX framework
- [Makepad](https://github.com/makepad/makepad) — cross-platform Rust UI toolkit
- [Project Robius](https://github.com/project-robius) — multi-platform Rust app framework tooling
