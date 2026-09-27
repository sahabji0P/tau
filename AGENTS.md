# Tau Agent Instructions

Tau is a Python implementation of Pi's minimalist coding-agent harness architecture. The goal is to develop it incrementally, with each phase clearly documented and tested.

## Project Roadmap

The implementation roadmap is tracked in GitHub issue #1:

- https://github.com/huggingface/tau/issues/1

Use that issue as the primary reference for phase ordering and architectural intent.

## Architecture Principles

Preserve Pi's core separation of concerns:

```text
AgentHarness = reusable agent brain
AgentSession = coding-agent environment
TUI = one possible frontend
```

Tau should be organized around these layers:

```text
tau_ai      provider/model streaming layer
tau_agent   portable agent harness, loop, tools, events, sessions
tau_coding  CLI app, resources, skills, extensions, commands, TUI integration
```

Keep the core agent package independent of CLI, Textual, Rich rendering, session file locations, and application-specific resource loading.

## TUI Direction

Use Textual for the full interactive TUI, but only behind an adapter boundary. The agent harness should emit events; UI layers should consume those events.

Early phases should prioritize:

1. print-mode CLI
2. Rich renderers
3. Textual interactive app

Do not let Textual become a dependency of the reusable agent harness.

## Development Workflow

- Work in small, documented phases.
- Keep changes aligned with the roadmap issue.
- Add or update docs when introducing architectural concepts.
- Add tests for behavior before expanding features.
- Run tests and Python commands through `uv` (for example, `uv run pytest` or `uv run python ...`) so they use the project environment.
- Prefer simple, explicit abstractions over framework-heavy designs.
- Keep commits atomic: one coherent feature, fix, docs update, refactor, or cleanup per commit.

## GitHub Issue and PR Formatting

- When creating or editing GitHub issues and pull requests from the CLI, write multiline Markdown bodies through a temporary file or heredoc and pass them with `--body-file`.
- Do not pass escaped newlines like `\n` inside quoted `--body` strings; GitHub will render them literally instead of as line breaks.
- Use Markdown headings, blank lines, bullets, and backticks for commands/paths so issue and PR descriptions are readable.
- After creating or editing a GitHub issue or PR body, verify the rendered source with `gh issue view ... --json body` or `gh pr view ... --json body` when practical.

## Python Guidelines

- Target the Python version declared in `pyproject.toml`.
- Prefer typed dataclasses or schema models for core messages, events, tools, and sessions.
- Keep async boundaries explicit.
- Use fake providers and fake tools for deterministic agent-loop tests.
- Avoid provider-specific assumptions in core agent code.

## Documentation Expectations

Each substantial phase should leave behind beginner-friendly notes under `dev-notes/` (build journals, design docs, ADRs), explaining:

- what was added
- why it exists
- how it maps to Pi's design
- how to test or use it

When a phase adds or changes user-facing behavior, also update the published docs
under `website/content/` (the "Use Tau" guides and reference).

## Cursor Cloud specific instructions

Tau is a single-process terminal coding agent (CLI/TUI). There is no backend
server, database, or Docker; it talks to external LLM APIs over HTTP. Standard
dev commands live in `README.md` and `CONTRIBUTING.md`; run everything through
`uv` (for example `uv run pytest`, `uv run tau`).

- Tooling: the project uses `uv`. It is preinstalled on the VM and on `PATH`
  via `~/.bashrc`/`~/.profile`. The startup update script runs `uv sync --dev`,
  so dependencies are ready; you do not need to reinstall them.
- Running the app end to end without a provider: the interactive TUI (`uv run
  tau`) and print mode (`uv run tau -p "..."`) require a configured model
  provider (API key or `/login` OAuth), which is not available in this
  environment. For a no-network end-to-end exercise of the agent loop and the
  built-in `read`/`write`/`edit`/`bash` tools, drive a `CodingSession` with
  `tau_ai.FakeProvider` (see `tests/test_coding_session.py` and
  `tests/pi_event_helpers.py` for the scripted-event pattern).
- Known pre-existing test failures (not an environment problem): with the
  locked `rich`/`textual`/`typer` versions, a small set of terminal
  width/wrapping assertions currently fail (`tests/test_cli.py`,
  `tests/test_tui_app.py`, `tests/test_tui_autocomplete.py`). The other ~1461
  tests pass and `ruff`/`ruff format`/`mypy` are clean. Do not treat these
  rendering-width failures as a setup regression.
- Optional docs site (`website/`) needs Hugo extended, which is not installed by
  default; install it only if you specifically need to build/preview docs
  (`cd website && hugo server -D`).

