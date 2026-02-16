# VoxTether Backend

Python FastAPI backend for VoxTether - speech-to-text transcription using faster-whisper.

## Features

- **Speech-to-Text**: Local transcription using faster-whisper (no cloud required)
- **GPU Acceleration**: Native CUDA 12 support with automatic fallback to CPU
- **Model Management**: Download, load, and manage Whisper models via API or CLI
- **REST API**: FastAPI-based HTTP API for transcription and model management
- **Privacy-First**: All processing is local, no telemetry, no network calls
- **Configurable**: Environment variable and `.env` file configuration

## Requirements

- Python 3.13+
- NVIDIA GPU with CUDA 12 support (optional, for GPU acceleration)

## Quick Start

### 1. Install Dependencies

```bash
cd src/backend
python -m venv venv
source venv/bin/activate  # Linux/macOS
# .\venv\Scripts\Activate.ps1  # Windows PowerShell
pip install -r requirements.txt
```

### 2. Start the Server

```bash
cd src/backend
python -m uvicorn main:app --host 127.0.0.1 --port 5678
```

The server will start on `http://127.0.0.1:5678`.

### 3. Verify

```bash
curl http://127.0.0.1:5678/api/health
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/health` | Health check and status |
| `GET` | `/api/devices` | GPU/CPU device information |
| `POST` | `/api/transcribe` | Transcribe audio file |
| `GET` | `/api/models` | List available models |
| `POST` | `/api/models/{name}/download` | Download a model (SSE) |
| `POST` | `/api/models/{name}/load` | Load a model |
| `POST` | `/api/models/{name}/unload` | Unload current model |
| `DELETE` | `/api/models/{name}` | Delete a model |
| `POST` | `/api/settings` | Update transcription settings |

## CLI

The backend includes a CLI tool for managing models and configuration:

```bash
cd src/backend
python cli.py list          # List available models
python cli.py download small  # Download a model
python cli.py serve         # Start the server
python cli.py config        # Show configuration
python cli.py info          # Show system/GPU information
```

## Project Structure

```
VoxTether-backend/
├── src/
│   └── backend/                 # Python Backend (FastAPI)
│       ├── api/                 # REST API endpoints
│       │   ├── health.py        # Health check endpoint
│       │   ├── models.py        # Model management endpoints
│       │   └── transcribe.py    # Transcription endpoint
│       ├── services/            # Business logic
│       │   ├── model_manager.py # Model download/management
│       │   └── transcriber.py   # faster-whisper integration
│       ├── utils/               # Utility modules
│       │   └── logging.py       # Structured logging
│       ├── tests/               # Unit tests
│       ├── main.py              # FastAPI entry point
│       ├── cli.py               # CLI for model management
│       ├── config.py            # Configuration settings
│       └── requirements.txt     # Python dependencies
│
├── tests/                       # Integration test scripts
├── docs/                        # Documentation
├── requirements-dev.txt         # Development dependencies
└── README.md
```

## Configuration

All settings can be configured via environment variables with the `VOXTETHER_` prefix:

| Variable | Default | Description |
|----------|---------|-------------|
| `VOXTETHER_HOST` | `127.0.0.1` | Server bind address |
| `VOXTETHER_PORT` | `5678` | Server port |
| `VOXTETHER_DEBUG` | `False` | Enable debug mode |
| `VOXTETHER_MODELS_PATH` | `~/.voxtether/models` | Models directory |
| `VOXTETHER_DEFAULT_MODEL` | `large-v3-turbo` | Default model to load |
| `VOXTETHER_PRELOAD_MODEL` | `True` | Preload model on startup |
| `VOXTETHER_DEVICE` | `auto` | Device (auto, cuda, cpu) |
| `VOXTETHER_COMPUTE_TYPE` | `auto` | Compute type (auto, float16, int8, float32) |

## GPU Acceleration

For NVIDIA GPU acceleration:

```bash
pip install nvidia-cublas-cu12 nvidia-cudnn-cu12
```

Or install CUDA 12 Toolkit from NVIDIA.

## Development

### Running Tests

```bash
# Install dev dependencies
pip install -r requirements-dev.txt

# Run unit tests
cd src/backend
python -m pytest tests/ -v

# Linting
ruff check src/backend/
```

### Building for Release

```bash
cd src/backend
pip install pyinstaller
pyinstaller --onefile --name vox-backend main.py
```

## Documentation

- [Backend API](docs/BACKEND-API.md) - Full API reference
- [Backend Setup](docs/BACKEND-SETUP.md) - Installation and configuration guide
- [Backend Features](docs/BACKEND-FEATURES.md) - Feature overview
- [Architecture](docs/ARCHITECTURE.md) - System architecture

## License

MIT License - see [LICENSE](LICENSE) for details.

## Credits

- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) - Fast Whisper transcription
- [CTranslate2](https://github.com/OpenNMT/CTranslate2) - Efficient inference engine
- [FastAPI](https://fastapi.tiangolo.com/) - Modern Python web framework