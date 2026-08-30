
# Claude Instructions for docx-editor

## Quick Reference

```bash
# Development
uv sync --dev              # Install dependencies
uv run ruff check .        # Lint
uv run ruff format .       # Format
uv run ty check            # Type check (CI gates on this)
uv run mkdocs serve        # Preview docs

# Tests — ALWAYS serial, niced, and memory-capped:
systemd-run --user --scope -p MemoryMax=8G -- nice -n 10 uv run pytest -q

# While iterating, cap the same way but target files:
systemd-run --user --scope -p MemoryMax=8G -- nice -n 10 uv run pytest -q tests/test_foo.py

# No systemd (macOS, containers)? Cap with ulimit in a subshell instead:
( ulimit -v 8388608; nice -n 10 uv run pytest -q )
```

**The memory cap is not optional: a leak must fail the run, not the desktop.**
An unbounded loop in `accept_all` once grew a single pytest process to 42 GB
RSS and got the machine OOM-killed; under the cap the same defect fails in
under a second. `--user` is required — plain `systemd-run --scope` needs root.
On platforms without a systemd user session, use the `ulimit -v` form above;
what matters is that *some* cap is in place, not which mechanism.

**Never pass `-n` / `-n auto`, and never add it to `addopts`.** `pytest-xdist`
is installed as a dependency but must not be used for local runs: parallel
workers multiply both load and peak memory on a shared machine. CI has its own
separate settings. Skip `--cov` locally too — it is slow and adds nothing while
iterating.

## Pull Requests

**Title:** `type: what changed in the code (ROADMAP.md #N)` — imperative, under ~72
chars. Describe the change, not where it came from. No round numbers, batch
names, phase names, or "as requested".

- ✅ `docs: document the changeset tier and error fields (ROADMAP.md #55)`
- ❌ `docs: SKILL.md round-2 sync + [create] extra` — "round-2" is our process, not the reader's problem

**Body:** four short sections, ~200 words total. Longer only if the change is
genuinely intricate.

```markdown
## What
Behavior before → after, in 1-3 sentences.

## Why
The user-visible problem this solves.

## How
The mechanism, only where it is not obvious from the diff. Name the files and
functions a reviewer should look at first.

## Testing
What proves it works: test names or counts, and gate results. One line each.
```

**Rules:**

- **Describe the code, not the process.** No dogfooding rounds, batch numbers,
  agent workflow, review iterations, planning artifacts, or self-criticism.
- **A PR describes its final state.** A defect introduced and fixed inside the
  same branch is not part of that state — do not narrate it. If it produced a
  regression test, name the test under Testing; that is the part that survives.
  - ❌ `The 42 GB OOM this PR caused...`
  - ✅ `Adds a regression test that fails in 0.17s if loop termination regresses.`
- **Prefer specifics over adjectives.** "1426 passed, peak RSS 133 MiB" beats
  "comprehensive testing".
- No emoji, no marketing language, no attribution to any tool or assistant.

## Code Quality Principles (CRITICAL)

**MANDATORY: These principles are CRITICAL for all code changes.**

### Core Principles

1. **KISS (Keep It Simple, Stupid)** - Choose the simplest solution that works.
2. **DRY (Don't Repeat Yourself)** - Search for and reuse existing code and patterns.
3. **YAGNI (You Aren't Gonna Need It)** - Only build what you need now.
4. **Leverage Existing Libraries** - Use defusedxml for XML parsing. Don't reinvent wheels.
5. **No Magic** - Explicit configuration only. No guessing, parsing, or fallbacks.
6. **Clear Separation of Concerns** - One purpose per file/class.
7. **Named Objects Over Tuples** - For 3+ return values, use dataclasses with named fields.
8. **Small, Focused Interfaces** - Minimal abstract base classes. Easy to extend.

### Before Writing Code

- [ ] Searched codebase for existing solutions
- [ ] Verified no library can solve this
- [ ] Kept it simple (KISS)

### Common Mistakes to Avoid

- ❌ **DON'T** create new files unless necessary → ✅ Edit existing files
- ❌ **DON'T** use `git add -A` → ✅ Use `git add <specific-file>`
- ❌ **DON'T** reinvent existing functionality → ✅ Reuse patterns
- ❌ **DON'T** skip tests → ✅ Write tests for changes
