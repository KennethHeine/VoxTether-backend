# VoxTether Architecture

This document describes the client-server architecture of VoxTether, a voice dictation application.

## Overview

VoxTether uses a **client-server architecture** with complete separation of frontend and backend:

- **Client (Frontend)**: Electron 40.x - Desktop application with UI and system tray
- **Server (Backend)**: Python FastAPI - Speech-to-text transcription service

The backend runs as a standalone Python server. It does NOT require PyInstaller or any executable bundling - just Python with the required packages.

| Component | Technology | Deployment |
|-----------|------------|------------|
| **Client** | Electron 40.x | Windows desktop application |
| **Server** | Python / FastAPI | Python script on any machine |
| **Transcription** | faster-whisper (CTranslate2) | Runs on server |
| **GPU Support** | CUDA 12 (native) | Server-side only |

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          CLIENT (User's Windows PC)                          │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────┐                               │
│   │         VoxTether.exe (Electron)         │                               │
│   ├─────────────────────────────────────────┤                               │
│   │  • System Tray Icon                      │                               │
│   │  • Global Hotkey Detection               │                               │
│   │  • Settings UI (HTML/CSS/JS)             │                               │
│   │  • Text Injection (Clipboard)            │                               │
│   │  • Audio Recording (Web Audio API)       │                               │
│   └─────────────────┬───────────────────────┘                               │
│                     │                                                        │
└─────────────────────┼────────────────────────────────────────────────────────┘
                      │
                      │ HTTP REST API
                      │ (localhost:5678 or network)
                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SERVER (Same machine or remote)                      │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────┐                               │
│   │      Python Backend (FastAPI + Uvicorn)  │                               │
│   ├─────────────────────────────────────────┤                               │
│   │  • REST API for transcription            │                               │
│   │  • faster-whisper integration            │                               │
│   │  • Model management (HuggingFace)        │                               │
│   │  • CUDA/CPU device management            │                               │
│   └─────────────────┬───────────────────────┘                               │
│                     │                                                        │
│                     ▼                                                        │
│   ┌─────────────────────────────────────────┐                               │
│   │           GPU (CUDA) / CPU               │                               │
│   │     (faster-whisper processing)          │                               │
│   └─────────────────────────────────────────┘                               │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Deployment Options

### Option 1: Same Machine (localhost)
Both client and server run on the same Windows PC. The server binds to `127.0.0.1:5678`.

### Option 2: Network Deployment
The server runs on a dedicated machine (with GPU), and multiple clients connect over the network. The server binds to `0.0.0.0:5678`.

---

## Component Responsibilities

### Client (Electron)

| Component | Purpose |
|-----------|---------|
| `main.js` | Electron main process, window management |
| `preload.js` | Secure IPC bridge to renderer |
| `renderer/` | UI (HTML/CSS/JS) |
| System Tray | Tray icon and context menu |
| Backend Client | HTTP client for backend API |
| Settings Service | Load/save user preferences |

### Server (Python / FastAPI)

| Component | Location | Purpose |
|-----------|----------|---------|
| `main.py` | `src/backend/` | FastAPI application entry point |
| `TranscriberService` | `src/backend/services/transcriber.py` | faster-whisper integration |
| `ModelManager` | `src/backend/services/model_manager.py` | Model download and management |
| `health.py` | `src/backend/api/health.py` | Health check endpoints |
| `transcribe.py` | `src/backend/api/transcribe.py` | Transcription endpoints |
| `models.py` | `src/backend/api/models.py` | Model management endpoints |

---

## REST API

The frontend communicates with the backend via HTTP REST API on `localhost:5678`.

### Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/api/health` | Health check and status |
| `GET` | `/api/devices` | Get GPU/CPU device info |
| `POST` | `/api/transcribe` | Transcribe audio file |
| `GET` | `/api/models` | List available models |
| `POST` | `/api/models/{name}/download` | Download a model (SSE) |
| `POST` | `/api/models/{name}/load` | Load a model |
| `DELETE` | `/api/models/{name}` | Delete a model |
| `POST` | `/api/settings` | Update transcription settings |

### Example: Transcription Request

```http
POST /api/transcribe
Content-Type: multipart/form-data

file: <audio.wav>
language: auto
translate: false
```

**Response:**
```json
{
  "text": "Hello, this is a test.",
  "language": "en",
  "duration": 0.82,
  "success": true
}
```

---

## Core Workflow

```
User holds hotkey              User releases hotkey
       │                              │
       ▼                              ▼
┌─────────────┐               ┌──────────────┐
│ Start Audio │               │ Stop Audio   │
│  Recording  │──────────────▶│  Recording   │
│(Web Audio)  │               │ (Web Audio)  │
└─────────────┘               └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ Save to WAV  │
                              │    File      │
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │  HTTP POST   │
                              │  to Backend  │
                              │ /api/transcribe
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ Backend runs │
                              │faster-whisper│
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ Return JSON  │
                              │  with text   │
                              └──────┬───────┘
                                     │
                                     ▼
                              ┌──────────────┐
                              │ Inject Text  │
                              │ (Clipboard + │
                              │  Ctrl+V)     │
                              └──────────────┘
```

---

## Project Structure

```
VoxTether-backend/
├── src/
│   └── backend/                     # Python Backend
│       ├── api/
│       │   ├── __init__.py
│       │   ├── health.py
│       │   ├── transcribe.py
│       │   └── models.py
│       ├── services/
│       │   ├── __init__.py
│       │   ├── transcriber.py
│       │   └── model_manager.py
│       ├── utils/
│       │   └── logging.py
│       ├── tests/
│       │   ├── conftest.py
│       │   ├── test_api_health.py
│       │   ├── test_api_models.py
│       │   ├── test_api_transcribe.py
│       │   ├── test_model_manager.py
│       │   └── test_transcriber.py
│       ├── main.py                  # FastAPI entry point
│       ├── cli.py                   # CLI for model management
│       ├── config.py                # Configuration
│       ├── constants.py             # Constants and model definitions
│       ├── dependencies.py          # FastAPI dependency injection
│       ├── exceptions.py            # Custom exception classes
│       ├── protocols.py             # Protocol definitions
│       ├── schemas.py               # Pydantic schemas
│       └── requirements.txt
│
├── tests/                           # Integration test scripts
├── docs/
│   ├── ARCHITECTURE.md              # This document
│   ├── BACKEND-API.md               # API documentation
│   ├── BACKEND-FEATURES.md          # Features overview
│   ├── BACKEND-SETUP.md             # Setup guide
│   ├── CHANGELOG.md                 # Version history
│   └── api-schemas/                 # API schema references
│
├── .github/workflows/
│   └── ci-backend.yml               # Backend CI pipeline
│
├── requirements-dev.txt             # Development dependencies
└── README.md
```

---

## Backend Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| Python | 3.13+ | Runtime |
| FastAPI | 0.109+ | Web framework |
| faster-whisper | 1.0+ | Transcription |
| uvicorn | 0.27+ | ASGI server |
| huggingface-hub | 0.20+ | Model downloads |

---

## Process Management

The frontend manages the backend process lifecycle:

1. **Startup**: Frontend starts → Launches backend → Waits for health check
2. **Runtime**: Frontend sends HTTP requests to backend
3. **Shutdown**: Frontend terminates → Kills backend process

---

## Data Storage

| Data | Location |
|------|----------|
| Settings | `%APPDATA%\VoxTether\settings.json` |
| Models | `%APPDATA%\VoxTether\models\` |
| Logs | `%APPDATA%\VoxTether\logs\` |

---

## Build and Release

### Local Development

```bash
# Start the backend
cd src/backend
pip install -r requirements.txt
python -m uvicorn main:app --port 5678
```

### CI/CD Pipeline

```
┌──────────────┐
│ Test Backend │
│   (Python)   │
└──────┬───────┘
       │
       ▼
┌──────────────────┐
│  Build Complete  │
│ (verify + upload)│
└──────────────────┘
```

---

## Security

- Backend binds to `127.0.0.1` only (localhost)
- No authentication required (local-only communication)
- Temporary audio files deleted after transcription
- No telemetry or network calls (except HuggingFace downloads)

---

## Performance Targets

| Metric | Target |
|--------|--------|
| Backend startup | < 5 seconds |
| Model loading | < 30 seconds |
| Transcription (8s audio, GPU) | < 1 second |
| Transcription (8s audio, CPU) | < 8 seconds |
| Idle memory (backend) | < 100 MB |
| Active memory (with model) | < 1.5 GB |

---

## See Also

- [Backend API Documentation](BACKEND-API.md) - API reference
- [Backend Setup Guide](BACKEND-SETUP.md) - Setup instructions
- [README](../README.md) - Project overview
