# Multi-Stack CI/CD Template

A portable, language-agnostic CI/CD template with Cursor AI integration. Supports Python, Node.js, Go, and Java with automatic stack detection.

## Features

- **Multi-stack support**: Python (pip/uv), Node.js, Go, Java
- **Automatic stack detection**: CI detects your stack from marker files
- **Pre-activation friendly**: Works before stack is decided (planning/architecture phase)
- **Reusable workflows**: Parameterized GitHub Actions for consistent CI
- **Cursor AI integration**: Skills, agents, and rules for AI-assisted development
- **AI CI tools**: PR summaries, test drafts, release notes (see examples/)

## Quick Start

### Use as GitHub Template

1. Click "Use this template" on GitHub
2. Clone your new repository
3. Start planning your architecture (no stack needed yet!)
4. When ready, run the `bootstrap-stack` Cursor skill to activate your stack
5. Push to trigger CI

### Manual Setup

```bash
git clone <repo-url>
cd <project-name>

# Option 1: Use bootstrap skill in Cursor
# Run: /bootstrap-stack

# Option 2: Create stack manually
echo "stack: python-uv" > tools/stack.yml
# Then create pyproject.toml, package.json, go.mod, or pom.xml
```

## Project Lifecycle

```
┌─────────────────┐     ┌──────────────┐     ┌────────────────┐
│ Pre-Activation  │ ──▶ │  Activation  │ ──▶ │   Active Dev   │
│ (planning/arch) │     │  (bootstrap) │     │  (full CI)     │
└─────────────────┘     └──────────────┘     └────────────────┘

Pre-Activation:          Activation:           Active Development:
- CI shows notice        - Run bootstrap       - Full lint/test/build
- PR title validated     - Creates markers     - Coverage reports
- No build checks        - Locks stack         - Security scans
```

## Stack Detection

CI automatically detects your stack using two methods:

### 1. Explicit Lock (Preferred)

Create `tools/stack.yml`:

```yaml
stack: python-uv  # or: python, node, go, java
version: "1.0"
```

### 2. Marker File Detection

If no lock file exists, CI detects stack from marker files:

| Stack | Marker Files |
|-------|--------------|
| `python-uv` | `uv.lock` |
| `python` | `pyproject.toml`, `setup.py`, `requirements.txt` |
| `node` | `package.json` |
| `go` | `go.mod` |
| `java` | `pom.xml`, `build.gradle` |

Detection order matters: `uv.lock` > `pyproject.toml` > others.

## Supported Stacks

### Python (uv) - Recommended

```yaml
# tools/stack.yml
stack: python-uv
```

Commands:
- Install: `uv sync --all-extras`
- Lint: `uv run ruff check . && uv run mypy src`
- Test: `uv run pytest -v`
- Coverage: `uv run pytest --cov=src --cov-report=xml`
- Build: `uv build`
- Scan: `uv run pip-audit`

### Python (pip)

```yaml
stack: python
```

Commands:
- Install: `pip install -e '.[dev]'`
- Lint: `ruff check . && mypy src`
- Test: `pytest -v`

### Node.js

```yaml
stack: node
```

Commands:
- Install: `npm ci`
- Lint: `npm run lint`
- Test: `npm test`
- Typecheck: `npm run typecheck`
- Build: `npm run build`
- Scan: `npm audit --audit-level=moderate`

### Go

```yaml
stack: go
```

Commands:
- Install: `go mod download`
- Lint: `golangci-lint run`
- Test: `go test -v ./...`
- Coverage: `go test -coverprofile=coverage.out ./...`
- Build: `go build ./...`
- Scan: `govulncheck ./...`

### Java (Maven)

```yaml
stack: java
```

Commands:
- Install: `mvn dependency:resolve`
- Lint: `mvn checkstyle:check`
- Test: `mvn test`
- Coverage: `mvn test jacoco:report`
- Build: `mvn package -DskipTests`
- Scan: `mvn org.owasp:dependency-check-maven:check`

## Customizing Commands

Edit `tools/tooling-matrix.yml` to customize CI commands:

```yaml
stacks:
  python-uv:
    commands:
      lint: "uv run ruff check . && uv run mypy src"
      test: "uv run pytest -v --timeout=60"  # Add timeout
```

## Project Structure

```
project/
├── .cursor/
│   ├── rules/           # Cursor AI rules
│   ├── agents/          # Cursor AI subagents
│   └── skills/          # Cursor AI skills
├── .github/
│   └── workflows/
│       ├── ci.yml              # Main CI (stack detection)
│       └── reusable-ci.yml     # Parameterized workflow
├── docs/                # Documentation
├── evals/               # Evaluation test data
├── examples/
│   └── python-fastapi/  # Reference FastAPI example
├── prompts/             # AI prompt templates
├── src/                 # Source code (created by bootstrap)
├── tests/               # Tests (created by bootstrap)
├── tools/
│   ├── ci/
│   │   ├── detect_stack.py     # Stack detection script
│   │   └── select_commands.py  # Command selection script
│   ├── tooling-matrix.yml      # Stack → commands mapping
│   └── stack.yml               # Stack lock (optional)
├── README.md
└── SECURITY.md
```

## Cursor AI Integration

This template includes Cursor AI defaults for AI-assisted development.

### Rules

- `001-global-best-practices.mdc` - Safety, detection, quality gates
- `002-ci-cd-template.mdc` - CI/CD configuration guidelines
- `003-security-quality.mdc` - Security and quality gates

### Skills

| Skill | Description |
|-------|-------------|
| `architecture` | Design system architecture |
| `bootstrap-stack` | Activate technology stack |
| `code-review` | Review code changes |
| `implementation` | Implement features |
| `planning` | Plan project requirements |
| `qa` | Quality assurance |
| `refactor-simplify` | Refactor and simplify code |
| `release-management` | Manage releases |
| `security` | Security review |

### Agents

Corresponding subagents for each skill that can run autonomously.

## CI Workflow

### Pre-Activation (No Stack)

When no stack is detected, CI:
1. Validates PR title (Conventional Commits)
2. Shows informative notice about missing stack
3. Provides instructions to activate

### Active Development

When stack is detected, CI:
1. Validates PR title
2. Detects stack from `tools/stack.yml` or markers
3. Selects commands from `tools/tooling-matrix.yml`
4. Runs: install → lint → typecheck → test → coverage → build → scan
5. Uploads coverage artifacts

## PR Title Convention

PRs must follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope)?: description

Types: feat, fix, docs, chore, refactor, test, perf, ci, build
```

Examples:
- `feat: add user authentication`
- `fix(api): handle null response`
- `docs: update README`

## Examples

See `examples/python-fastapi/` for a complete FastAPI example with:
- Full CI/CD setup
- AI PR summary generator
- AI test draft generator
- AI release notes generator
- LLM evaluation tests

## AI CI Features (Optional)

The FastAPI example includes optional AI-powered CI tools:

| Feature | Workflow | Required Secret |
|---------|----------|-----------------|
| PR Summary | `ai_pr_summary.yml` | `OPENAI_API_KEY` |
| Test Drafts | `ai_test_draft.yml` | `OPENAI_API_KEY` |
| Release Notes | `release_notes.yml` | `OPENAI_API_KEY` |
| LLM Evals | `llm_evals.yml` | `OPENAI_API_KEY` |

These workflows skip gracefully if the secret is not configured.

## Security

See [SECURITY.md](SECURITY.md) for security policy and practices.

## License

MIT
