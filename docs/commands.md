# VoxTether Backend Quick Commands

## Start Backend Locally

```bash
cd src/backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 5678 --reload
```

## Run Tests

```bash
cd src/backend
python -m pytest tests/ -v
```

## Linting

```bash
ruff check src/backend/
```
