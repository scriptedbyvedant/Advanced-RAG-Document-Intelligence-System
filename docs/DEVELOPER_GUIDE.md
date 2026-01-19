# Developer Guide

## 1. Development setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn api.main:app --reload --port 8080
```

## 2. Project structure

- `api/`: FastAPI app and routing
- `src/`: core pipelines (ingestion, analysis, chat, comparison)
- `utils/`: shared utilities and model loader
- `model/`: Pydantic schemas
- `prompt/`: prompt templates
- `static/` and `templates/`: UI assets

## 3. Adding a new endpoint

1) Implement logic in a module under `src/`.
2) Add a route in `api/main.py`.
3) Update `docs/API.md` with examples.

## 4. Changing providers

- Update `utils/model_loader.py` to add or modify LLM/embeddings.
- Ensure provider credentials are loaded via environment variables.

## 5. Prompt updates

- Add new prompts in `prompt/prompt_library.py`.
- Validate output schema in `model/models.py`.
- Update parsing logic if schema changes.

## 6. Testing

- Unit tests: add under `tests/`.
- API tests: run curl-based smoke tests or add pytest integration tests.

## 7. Logging and debugging

- Logs are emitted through `logger/GLOBAL_LOGGER`.
- Check errors in the console or CloudWatch if deployed.

## 8. Style and conventions

- Keep modules focused on a single responsibility.
- Prefer explicit error handling and clear log messages.
- Use ASCII-only text in docs and code comments.
