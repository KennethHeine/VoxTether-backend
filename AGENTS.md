# AGENTS.md

This file provides context and instructions to help AI coding agents work effectively on the VoxTether Backend project.

## Project Overview

VoxTether Backend is a Python FastAPI server for speech-to-text transcription using faster-whisper. It is the backend component of the VoxTether voice dictation application.

## Setup Commands

```bash
# Backend setup
cd src/backend
python -m venv venv
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate    # Windows

pip install -r requirements.txt

# Install dev dependencies (for testing/linting)
pip install -r ../../requirements-dev.txt

# Run backend server
python -m uvicorn main:app --host 127.0.0.1 --port 5678
```

## Architecture

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
│       │   ├── conftest.py      # Pytest fixtures
│       │   ├── test_api_health.py
│       │   ├── test_api_models.py
│       │   ├── test_api_transcribe.py
│       │   ├── test_model_manager.py
│       │   └── test_transcriber.py
│       ├── main.py              # FastAPI entry point
│       ├── cli.py               # CLI for model management
│       ├── config.py            # Configuration settings
│       ├── constants.py         # Constants and model definitions
│       ├── dependencies.py      # FastAPI dependency injection
│       ├── exceptions.py        # Custom exception classes
│       ├── protocols.py         # Protocol definitions
│       ├── schemas.py           # Pydantic schemas
│       └── requirements.txt     # Python dependencies
│
├── tests/                       # Integration test scripts
├── docs/                        # Documentation
├── requirements-dev.txt         # Development dependencies
└── README.md
```

## Key Components

### Backend (FastAPI)
- `main.py` - FastAPI application entry point
- `cli.py` - CLI tool for model management and server control
- `config.py` - Configuration settings (pydantic-settings)
- `api/health.py` - Health check endpoint
- `api/models.py` - Model management endpoints (list, download, delete, load)
- `api/transcribe.py` - Transcription endpoint
- `services/transcriber.py` - faster-whisper integration
- `services/model_manager.py` - Model download and management

## Code Style

- Python 3.13+
- Follow PEP 8 style guidelines
- Use type hints where appropriate
- Use ruff for linting

## Testing

```bash
# Run unit tests
cd src/backend
python -m pytest tests/ -v

# Linting
ruff check src/backend/
```

## CI/CD

CI/CD workflows are defined in `.github/workflows/`:
- `ci-backend.yml` - Backend CI (linting, unit tests, server start test)

Runs on pull requests and pushes to main branch.

## Platform

- Targets Python 3.13+
- Uses faster-whisper for transcription (native CUDA 12 support)
- Windows-focused but works on Linux for development/CI
