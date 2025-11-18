# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kokoro-FastAPI is a FastAPI wrapper for the Kokoro-82M text-to-speech model that provides OpenAI-compatible endpoints. It supports GPU acceleration (CUDA/MPS), multiple audio formats, voice combination, streaming, and phoneme-based audio generation.

## Development Commands

### Running the Service

**GPU Mode (NVIDIA CUDA):**
```bash
./start-gpu.sh
```

**CPU Mode:**
```bash
./start-cpu.sh
```

**Windows (PowerShell):**
```powershell
.\start-gpu.ps1  # For GPU
.\start-cpu.ps1  # For CPU
```

**Docker Development:**
```bash
cd docker/gpu  # or docker/cpu
docker compose up --build
```

### Testing

**Run all tests:**
```bash
uv run pytest
```

**Run specific test files:**
```bash
uv run pytest api/tests/test_openai_endpoints.py
uv run pytest api/tests/test_tts_service.py
```

**Run with coverage:**
```bash
uv run pytest --cov=api --cov-report=term-missing
```

**Test specific functionality:**
```bash
uv run python examples/assorted_checks/test_openai/test_openai_tts.py
uv run python examples/assorted_checks/test_voices/test_all_voices.py
```

### Dependencies

**Install dependencies for GPU:**
```bash
uv pip install -e ".[gpu]"
```

**Install dependencies for CPU:**
```bash
uv pip install -e ".[cpu]"
```

**Install test dependencies:**
```bash
uv pip install -e ".[test]"
```

### Model Management

**Download models manually:**
```bash
python docker/scripts/download_model.py --output api/src/models/v1_0
```

## Architecture

### Core Components

- **`api/src/main.py`**: FastAPI application entry point with lifespan management
- **`api/src/core/config.py`**: Centralized configuration using Pydantic settings
- **`api/src/inference/`**: Model inference backends and management
  - `model_manager.py`: Singleton model manager with warmup
  - `kokoro_v1.py`: Kokoro V1 model implementation
  - `voice_manager.py`: Voice pack management and combination
- **`api/src/routers/`**: API route handlers
  - `openai_compatible.py`: OpenAI-compatible `/v1/audio/speech` endpoint
  - `development.py`: Development endpoints like `/dev/captioned_speech`
  - `debug.py`: System monitoring endpoints
- **`api/src/services/`**: Business logic services
  - `tts_service.py`: Main TTS generation service
  - `audio.py`: Audio format conversion and processing
  - `streaming_audio_writer.py`: Streaming response handling
  - `text_processing/`: Text normalization and phonemization

### Key Features

**OpenAI Compatibility:** The `/v1/audio/speech` endpoint provides OpenAI-compatible API with standard request/response formats.

**Voice Combination:** Supports weighted voice combinations (e.g., "af_bella(2)+af_sky(1)" for 67%/33% mix).

**Streaming:** Real-time audio streaming with configurable chunk sizes for low latency.

**Multi-format Audio:** Supports mp3, wav, opus, flac, m4a, pcm formats.

**Phoneme Processing:** Text-to-phoneme conversion and phoneme-to-audio generation.

**Timestamped Captions:** Word-level timestamp generation for synchronized captions.

### Configuration

**Environment Variables:**
- `USE_GPU`: Enable/disable GPU acceleration (true/false)
- `API_LOG_LEVEL`: Logging level (DEBUG, INFO, WARNING, ERROR)
- `MODEL_DIR`: Model files directory
- `VOICES_DIR`: Voice packs directory
- `HOST`, `PORT`: Server binding configuration

**Key Settings in `config.py`:**
- `target_min_tokens/max_tokens`: Text chunking parameters
- `sample_rate`: Audio output sample rate (24000 Hz)
- `enable_web_player`: Web UI toggle
- `cors_enabled`: CORS configuration

### Development Patterns

**Async Service Pattern:** Services use async patterns with singleton managers accessed via `get_manager()` functions.

**Streaming Response Pattern:** Audio streaming uses `StreamingAudioWriter` with configurable chunk sizes and gap trimming.

**Voice Management:** Voice combinations are normalized and cached for reuse.

**Error Handling:** Comprehensive error logging with loguru, graceful degradation for missing dependencies.

## File Structure

```
api/src/
├── core/           # Configuration, paths, model configs
├── inference/      # Model backends and voice management
├── routers/        # API endpoint handlers
├── services/       # Business logic and utilities
├── structures/     # Pydantic schemas and data models
└── tests/          # Test suite

docker/
├── gpu/           # GPU Docker configuration
└── cpu/           # CPU Docker configuration

examples/
├── assorted_checks/    # Test and validation scripts
├── phoneme_examples/   # Phoneme usage examples
└── streaming_*.py      # Streaming examples
```

## Important Notes

**Model Initialization:** The model is loaded during application startup with warmup. First requests may be slower until warmup completes.

**Device Detection:** GPU/CUDA is auto-detected but can be overridden via `device_type` setting or `USE_GPU=false`.

**Text Chunking:** Long texts are automatically chunked based on configurable token limits to model's ~30-second output constraint.

**Voice Storage:** Combined voices can be saved locally when `allow_local_voice_saving=true` (disabled by default).

**Temperature Management:** Temporary files for web player are automatically cleaned up based on size/age limits.