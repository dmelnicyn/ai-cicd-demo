---
name: bootstrap-stack
description: Bootstrap a new project by creating stack markers, tools/stack.yml, and skeleton directories.
---

# Bootstrap Stack Skill

Creates the foundational files for a new project based on selected technology stack.
This skill transitions a project from pre-activation state to active development.

## When to Use

- After architecture decision selects a technology stack
- When initializing a new project from this template
- When converting an existing project to use this CI template

## Prerequisites

- Architecture phase should be complete (stack decision made)
- You should know which stack is appropriate for the project

## Supported Stacks

| Stack | Marker Files | Tools |
|-------|--------------|-------|
| `python` | `pyproject.toml` | pip, pytest, ruff, mypy |
| `python-uv` | `pyproject.toml`, `uv.lock` | uv, pytest, ruff, mypy |
| `node` | `package.json` | npm, jest/vitest, eslint, tsc |
| `go` | `go.mod` | go, golangci-lint, govulncheck |
| `java` | `pom.xml` | maven, junit, checkstyle |

## Instructions

1. **Confirm stack selection with user**
   - Ask which stack to bootstrap: python, python-uv, node, go, java
   - Verify this aligns with architecture decisions

2. **Create `tools/stack.yml`**
   ```yaml
   # Stack lock file - created by bootstrap-stack skill
   # This file locks the CI to a specific stack
   stack: <selected-stack>
   version: "1.0"
   created: <timestamp>
   ```

3. **Create marker files for the selected stack**

   **Python (pip)**:
   ```toml
   # pyproject.toml
   [project]
   name = "<project-name>"
   version = "0.1.0"
   requires-python = ">=3.11"
   dependencies = []

   [project.optional-dependencies]
   dev = ["pytest", "ruff", "mypy"]

   [build-system]
   requires = ["hatchling"]
   build-backend = "hatchling.build"
   ```

   **Python (uv)**: Same as above, then run `uv lock`

   **Node**:
   ```json
   {
     "name": "<project-name>",
     "version": "0.1.0",
     "scripts": {
       "test": "jest",
       "lint": "eslint .",
       "typecheck": "tsc --noEmit",
       "build": "tsc"
     }
   }
   ```

   **Go**:
   ```
   module <module-path>

   go 1.22
   ```

   **Java (Maven)**:
   ```xml
   <project>
     <modelVersion>4.0.0</modelVersion>
     <groupId>com.example</groupId>
     <artifactId>project-name</artifactId>
     <version>0.1.0</version>
   </project>
   ```

4. **Create skeleton directories**
   - `src/` - Source code
   - `tests/` - Test files
   - `docs/` - Documentation

5. **Create basic Makefile** (optional but recommended)
   - Include common commands: install, test, lint, build

6. **Validate activation**
   - Run `python tools/ci/detect_stack.py` to verify detection
   - Should output the selected stack name (not "none")

7. **Update README** with stack-specific instructions

## Verification

After running this skill, verify:
- [ ] `tools/stack.yml` exists with correct stack
- [ ] Marker file exists (pyproject.toml, package.json, etc.)
- [ ] `python tools/ci/detect_stack.py` returns the correct stack
- [ ] `python tools/ci/select_commands.py <stack>` returns valid commands
- [ ] CI workflow would run (not show "no stack detected")

## Example Session

```
User: Bootstrap the project for Python with uv

Agent: I'll bootstrap this project for Python with uv package manager.

1. Creating tools/stack.yml...
2. Creating pyproject.toml...
3. Running uv lock to create uv.lock...
4. Creating src/, tests/, docs/ directories...
5. Verifying stack detection...

Stack detection output: python-uv

Project is now activated for Python (uv). CI will run:
- Install: uv sync --all-extras
- Lint: uv run ruff check . && uv run mypy src
- Test: uv run pytest -v
```

## Related

- See `tools/tooling-matrix.yml` for command configuration
- See `tools/ci/detect_stack.py` for detection logic
- See `.github/workflows/ci.yml` for CI workflow
