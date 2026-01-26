# AI CI/CD Demo

A minimal FastAPI learning template for CI/CD with Python.

## Prerequisites

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) (Python package manager)

Install uv if you don't have it:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Setup

Clone the repository and install dependencies:

```bash
git clone <repo-url>
cd ai-cicd-demo
make install
```

Or without make:

```bash
uv sync --all-extras
```

## Commands

| Task | Make Command | Direct Command |
|------|--------------|----------------|
| Install deps | `make install` | `uv sync --all-extras` |
| Run server | `make run` | `uv run uvicorn ai_cicd_demo.main:app --reload` |
| Run tests | `make test` | `uv run pytest -v` |
| Run coverage | `make coverage` | `uv run pytest --cov=src --cov-report=term-missing --cov-report=xml --cov-report=html` |
| Lint code | `make lint` | `uv run ruff check . && uv run mypy src` |
| Format code | `make format` | `uv run ruff format . && uv run ruff check --fix .` |
| All checks | `make check` | Runs lint + test |
| LLM evals | `make llm-evals` | `uv run python tools/run_llm_evals.py` |

## Project Structure

```
ai-cicd-demo/
├── pyproject.toml          # Project config, dependencies, tool settings
├── uv.lock                  # Locked dependencies
├── Makefile                 # Developer commands
├── README.md                # This file
├── src/
│   └── ai_cicd_demo/
│       ├── __init__.py      # Package init
│       ├── main.py          # FastAPI app and endpoints
│       ├── models.py        # Pydantic models
│       └── ai/              # AI module
│           ├── __init__.py
│           ├── openai_client.py  # OpenAI wrapper
│           └── intent.py         # Intent classifier
├── tests/
│   ├── __init__.py
│   └── test_main.py         # API tests using TestClient
├── evals/
│   └── golden_intent.json   # Golden set for LLM evals
├── tools/
│   ├── shared.py            # Shared utilities for AI tools
│   ├── ai_pr_summary.py     # AI PR summary generator
│   ├── ai_test_draft.py     # AI draft test generator
│   ├── ai_release_notes.py  # AI release notes generator
│   └── run_llm_evals.py     # LLM eval runner
└── prompts/
    ├── pr_summary.md        # PR summary prompt template
    ├── test_generation.md   # Test generation prompt template
    └── release_notes.md     # Release notes prompt template
```

## API Endpoints

- `GET /health` - Health check, returns `{"status": "ok"}`
- `GET /items/{item_id}` - Get item by ID (mock data)
- `POST /ai/classify_intent` - Classify text intent (see [AI Intent Classifier](#ai-intent-classifier))
- `GET /docs` - Swagger UI documentation (when server is running)

## Tooling

- **Dependency management**: [uv](https://docs.astral.sh/uv/)
- **Linting & formatting**: [ruff](https://docs.astral.sh/ruff/)
- **Type checking**: [mypy](https://mypy-lang.org/)
- **Testing**: [pytest](https://pytest.org/) with FastAPI TestClient

## CI

GitHub Actions runs on push to `main` and on pull requests:
- `make lint` — ruff + mypy
- `make coverage` — pytest with coverage (fails if below 80%)

Coverage reports (`coverage.xml` and `htmlcov/`) are uploaded as artifacts.

See [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

## Coverage

CI enforces a minimum **80% code coverage** threshold. Coverage is configured in `pyproject.toml`.

### Running Locally

```bash
make coverage
```

This generates:
- Terminal summary with missing lines
- `coverage.xml` (Cobertura format)
- `htmlcov/` folder (open `htmlcov/index.html` in browser)

### Threshold

The 80% threshold is configured in `pyproject.toml` under `[tool.coverage.report]`:

```toml
[tool.coverage.report]
fail_under = 80
```

To adjust, change the `fail_under` value.

## AI Intent Classifier

Classifies text into one of four intent categories using OpenAI:

- **QUESTION** - User is asking a question or seeking information
- **REQUEST** - User is asking for an action to be performed
- **COMPLAINT** - User is expressing dissatisfaction
- **OTHER** - Doesn't fit the above categories

### Usage

**Endpoint:** `POST /ai/classify_intent`

```bash
curl -X POST http://localhost:8000/ai/classify_intent \
  -H "Content-Type: application/json" \
  -d '{"text": "What time does the store open?"}'
```

**Response:**

```json
{"intent": "QUESTION"}
```

### LLM Evals (Golden Set Testing)

The intent classifier has a golden set of test cases in `evals/golden_intent.json` that validate model behavior.

**Run locally:**

```bash
export OPENAI_API_KEY=sk-...
make llm-evals
```

**Output:**

```
============================================================
LLM Intent Classification Evals
============================================================

Loaded 10 test cases from golden_intent.json
Per-test timeout: 30s
Total timeout: 300s
------------------------------------------------------------
[PASS] q1: QUESTION == QUESTION
[PASS] q2: QUESTION == QUESTION
...
------------------------------------------------------------

Results: 10/10 passed
Time: 12.34s

All LLM evals passed!
```

**CI:** LLM evals run automatically on push to `main` and on pull requests. If `OPENAI_API_KEY` is not configured, the job skips gracefully with a notice.

See [`.github/workflows/llm_evals.yml`](.github/workflows/llm_evals.yml).

### Adding New Golden Test Cases

Edit `evals/golden_intent.json`:

```json
{
  "id": "unique_id",
  "input_text": "Your test input here",
  "expected_intent": "QUESTION",
  "notes": "Optional description"
}
```

### Determinism

To minimize flakiness and ensure consistent results:

- **Temperature:** 0 (fully deterministic sampling)
- **Constrained output:** System prompt forces exactly one of 4 labels
- **Low max_tokens:** 10 tokens prevents verbose responses
- **Model:** gpt-4o-mini (pinned)

### Cost

Using gpt-4o-mini with ~50 tokens per request:
- **Per classification:** ~$0.0001
- **10-case eval run:** ~$0.001
- **Monthly CI (100 runs):** ~$0.10

## AI PR Summary

Automatically posts an AI-generated summary comment on pull requests, including:

- **Summary** — bullet points of key changes
- **Risk assessment** — Low/Medium/High with reasons
- **Suggested checks** — tests or manual verification based on changes
- **Grouped file list** — files organized by area (API, tests, config, etc.)

**Required Secret:** Add `OPENAI_API_KEY` to your repository secrets (Settings → Secrets → Actions).

If the secret is not configured, the workflow skips gracefully with a notice.

Security features:
- Potential secrets are redacted before sending to OpenAI
- Large diffs are truncated to prevent excessive API costs
- Minimal permissions (contents read, pull-requests write)

See [`.github/workflows/ai_pr_summary.yml`](.github/workflows/ai_pr_summary.yml).

## AI Draft Test Generator

Automatically generates draft pytest test suggestions for changed Python source files in pull requests.

**Outputs:**
- **PR Comment** — Summary with sample test suggestions
- **Artifact** — Full `draft_tests.md` with per-file test suggestions (7-day retention)

**Focus:**
- Analyzes Python files under `src/`
- Ignores venv, lockfiles, markdown, existing tests
- For each file: what to test, pytest code blocks, testability notes
- Identifies hard-to-test code and suggests refactor points

**Non-destructive:** Generated tests are suggestions only — they are never committed automatically.

**Required Secret:** `OPENAI_API_KEY` (same as AI PR Summary)

If the secret is not configured, the workflow skips gracefully with a notice.

Security features:
- Potential secrets are redacted before sending to OpenAI
- Large patches are truncated to prevent excessive API costs
- Minimal permissions (contents read, pull-requests write)

See [`.github/workflows/ai_test_draft.yml`](.github/workflows/ai_test_draft.yml).

## AI Release Notes

Automatically generates AI-powered release notes when you push a version tag.

**Trigger:** Push a tag matching `v*` (e.g., `v0.1.0`, `v1.0.0-beta`)

**What it does:**

1. Determines the previous tag automatically
2. Collects commits between tags (with PR titles where available)
3. Generates grouped release notes using OpenAI:
   - Features, Fixes, Documentation, Chore/CI, Other
   - Highlights breaking changes (conventional commits with `!`)
4. Creates a **draft** GitHub release (review before publishing)
5. Uploads release notes as an artifact (7-day retention)

**Creating a release:**

```bash
# Create and push a tag
git tag v0.1.0
git push origin v0.1.0

# Or create an annotated tag
git tag -a v0.1.0 -m "Release v0.1.0"
git push origin v0.1.0
```

After the workflow completes, go to GitHub → Releases to review and publish the draft.

**Required Secret:** `OPENAI_API_KEY` (same as other AI tools)

**Idempotent:** Re-running the workflow on the same tag updates the existing draft release.

Security features:

- Potential secrets are redacted before sending to OpenAI
- Limited to 50 commits per release to control API costs
- Minimal permissions (contents write for releases only)

See [`.github/workflows/release_notes.yml`](.github/workflows/release_notes.yml).

## PR Title Convention

This repository enforces [Conventional Commits](https://www.conventionalcommits.org/) for PR titles.

**Format:** `type(scope)?: description`

**Allowed types:**
- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation
- `chore` — maintenance
- `refactor` — code refactoring
- `test` — tests
- `perf` — performance
- `ci` — CI/CD changes
- `build` — build system

**Examples:**
- `feat: add user profile endpoint`
- `fix(auth): handle missing token`
- `docs: update README`
