# CLAUDE.md — project memory for this repo

This file is read automatically by Claude Code at the start of every session
in this repo. It exists so a future session (possibly with very limited
budget) does not have to re-derive any of this from scratch.

## What this repo is

A complete Python training course, in German-flavored informal English
comments, structured as numbered Jupyter notebooks in `topics/`. Beginner to
expert, plus several large optional specialization arcs. `README.md` has the
full, currently-accurate lesson list — always keep it in sync when adding
lessons.

**Branch**: all work happens on `actual_release` (this is the repo's main
line now — `it_wiw` and the old `claude/...` branch were deleted by the
user). Just commit and push directly to `actual_release`, no PRs needed
unless the user asks.

**Status as of now**: lessons 00–48 exist, are all independently validated
(zero execution errors), and are pushed. The course covers: core Python
(00–18), a specialization track — data science/ML/AI/Flask/automation
(19–25), web crawling + full pygame game-dev arc up to an RPG (26–32), game
networking/multiplayer + a deeper RPG + a Pokémon-style clone (33–39), a
SQLAlchemy + multi-database arc (40–46: SQLAlchemy Core/ORM, Alembic
migrations, PostgreSQL, MySQL/MariaDB, Redis, MongoDB-style NoSQL via
mongomock, and a "choosing the right database" capstone), and a
self-implementations/best-practices category (47–48: hand-rolled data
structures contrasted with the built-ins; clean code/SOLID/design
patterns/anti-patterns). Lessons 47-48 were written directly rather than
delegated to parallel agents — no external services/infra needed for that
content, so direct writing was both cheaper and simpler.

## The user's standing preferences (established over many sessions, still valid)

- Communicates in German; content itself is written in English matching the
  repo's existing voice.
- Every lesson notebook must: match the existing informal, heavily-commented
  "we/let's" voice; end with exercises + full worked sample solutions; end
  with a "quick comparison to other languages" section (JS/TS, Java, C#,
  C++, Go, Rust, and others as relevant e.g. ORMs, NoSQL drivers).
- **Never trust a delegated agent's self-report.** Always independently
  re-run the notebook via nbclient yourself, and spot-check actual printed
  cell output content, not just "zero errors" — several real bugs were only
  caught this way (see "Bugs found this way" below).
- Every notebook must execute headlessly end-to-end with zero errors before
  being committed.
- No stray files/services/tables/keys left behind — see "Shared resource
  hygiene" below.
- Commit and push completed, validated work without being asked each time
  (the user's implicit expectation, given how many rounds of this have
  already happened) — but still only push to `actual_release`, never
  force-push, never rewrite history.
- Budget-consciousness matters to this user (explicitly said so). Prefer
  efficient tool use: batch reads, avoid redundant re-validation, delegate
  large independent writing tasks to parallel background agents rather than
  doing them serially yourself.

## How new lessons actually get built (the established workflow)

1. **De-risk technical unknowns yourself first**, via direct Bash
   prototyping, before writing or delegating a lesson plan. This has caught
   real problems ahead of time every single time it's been done (e.g.
   confirming asyncio servers persist across notebook cells, confirming the
   exact Alembic recipe end-to-end, confirming pg_ctlcluster/service
   mariadb/redis-server work without systemd in this container).
2. **Use the shared `nb_builder.py` helper** pattern:
   `write(path, cells)` where `cells` is a list of `("md"|"code", text)`
   tuples — matches this repo's existing nbformat/kernelspec metadata. Put a
   fresh copy of this helper in your scratchpad when building a new batch of
   lessons (it doesn't live in the repo itself).
3. **For a batch of independent lessons, delegate to parallel background
   `Agent` calls** (`Agent` tool, one call per lesson, all in the same
   message so they run concurrently). Each prompt must be fully
   self-contained (the agent has zero conversation context) and must
   include: the mandatory-reading list of prerequisite/style-reference
   notebooks, the exact `nb_builder.py` usage snippet, the exact
   nbclient validation snippet (below), and any lesson-specific
   infrastructure details (already-running service connection strings,
   known gotchas/fixes, resource-naming-prefix convention).
4. **Independently re-validate every returned notebook yourself** — never
   just take the agent's word. Re-run nbclient fresh, read actual cell
   outputs, and fix any real bugs directly rather than sending it back to
   the agent (faster, and you already have full context).
5. Update `README.md`'s lesson list, do a full repo-wide re-execution sweep
   of *all* notebooks (not just the new ones — catches regressions), then
   commit and push.

### Standard validation snippet (headless nbclient)

```python
import nbformat
from nbclient import NotebookClient

path = "topics/NN_something.ipynb"
nb = nbformat.read(path, as_version=4)
client = NotebookClient(nb, timeout=300, kernel_name="python3")
client.execute()
nbformat.write(nb, path)  # re-save so outputs are baked in

errors = []
for i, cell in enumerate(nb.cells):
    if cell.cell_type == "code":
        for out in cell.get("outputs", []):
            if out.get("output_type") == "error":
                errors.append((i, out.get("ename"), out.get("evalue")))
print(f"cells: {len(nb.cells)}, errors: {len(errors)}")
```

A full-repo sweep is just this loop over `sorted(glob.glob("topics/*.ipynb"))`
— takes a while (game/network lessons have real `time.sleep()`s), run it in
the background and wait for the notification rather than polling.

## Technical patterns already solved — reuse, don't rederive

- **Headless pygame**: set `SDL_VIDEODRIVER=dummy` and `SDL_AUDIODRIVER=dummy`
  as env vars *before* `import pygame`. To show a frame in a notebook, use a
  `show_surface()` helper that converts the pygame `Surface` → numpy array →
  `matplotlib.pyplot.imshow`. Established in lesson 27, reused through the
  whole game-dev arc.
- **asyncio multiplayer server**: `await asyncio.start_server(...)` works as
  top-level await directly in a Jupyter cell, and the server persists across
  *separate* later cells in the same kernel — no need to keep it in one
  giant cell. Background listener loops via `asyncio.create_task()` feeding
  a `.history` list work fine too. Established in lesson 34, reused through
  the networking/multiplayer/Pokémon-clone arc.
- **SQLAlchemy 2.x**: both the Core expression layer (`Table`/`Column`/
  `insert()`/`select()`) and the modern declarative ORM
  (`Mapped`/`mapped_column`, `relationship()`) are covered starting lesson
  40 — always show Core and ORM side by side with the equivalent raw
  sqlite3 code from lesson 13 for contrast.
- **Alembic migrations, driven entirely from Python** (no shell/CLI, so it
  works inside a notebook cell): create a `tempfile.mkdtemp()` alembic
  project, drive it with `alembic.config.Config` + `alembic.command`
  (`init`, `revision(autogenerate=...)`, `upgrade`, `downgrade`), then
  `shutil.rmtree()` the temp dir at the end. Full working recipe is in
  lesson 41 if you need to copy the exact shape again.
- **Real local database servers run in this container already** (Debian/
  Ubuntu 24.04, no systemd):
  - PostgreSQL 16: already running on `127.0.0.1:5432`, db `pytraining`,
    user `pyuser` / pass `pypass`. Start/manage via `pg_ctlcluster` if it's
    ever down.
  - MariaDB: already running on `127.0.0.1:3306`, same db/user/pass. Start
    via `service mariadb start` if down.
  - Redis: start via `redis-server --daemonize yes` if down.
  - MongoDB: **not actually installable** in this environment — `apt-get
    install mongodb` has no candidate on Ubuntu 24.04's default repos, and
    the tarball download from fastdl.mongodb.org is blocked by the proxy
    (403). Use `mongomock` (pip-installable) instead, and be explicit and
    upfront in the lesson's intro markdown that it's a mock, not a real
    server — mongomock's API is faithful to pymongo though, so everything
    taught transfers directly to a real MongoDB/Atlas connection.
  - `pymysql` install can break with `ModuleNotFoundError: No module named
    '_cffi_backend'` + a Rust/pyo3 panic (conflict between apt's
    `cryptography` and pip's expectations). Fix:
    `pip install --quiet --force-reinstall --no-cache-dir cffi cryptography`
    (a debian-uninstall warning is harmless, ignore it).
- **Shared resource hygiene across parallel agents**: when multiple lessons
  share the same live Postgres/MariaDB/Redis instance, give every lesson N
  its own naming prefix for everything it creates (`l{N}_` for SQL tables,
  `l{N}:` for Redis keys) and have the notebook itself drop/delete
  everything with that prefix at the end, then independently re-verify
  (fresh connection, query `information_schema`/`redis-cli keys`) that
  nothing with that prefix remains — don't just trust the notebook's own
  cleanup cell.

## Real bugs found this way (illustrates why independent re-validation matters)

- Lesson 44 (Redis pub/sub): closing a `pubsub` connection from the main
  thread while a background thread was blocked in `pubsub.listen()` raised
  an uncaught `ConnectionError` in that thread — printed a stray traceback
  into the cell output even though the demo "worked". `client.execute()`
  didn't flag it as a cell error (it's stderr from a background thread, not
  an `output_type == "error"` in the triggering cell), only reading the
  actual printed output caught it. Fix: wrap the listener function body in
  `try/except Exception: pass`.
- Lesson 43 (MySQL): a cell claimed `connection.insert_id()` returns "the
  same value" as `cursor.lastrowid`, but called it *after*
  `connection.commit()` — issuing `COMMIT` is itself a round trip that
  resets pymysql's tracked insert-id back to 0, so the demo's own printed
  output (0) contradicted its own claim. Only caught by reading the actual
  output number, not by the zero-errors check. Fix: read `insert_id()`
  before the commit.
- Lesson 34 (multiplayer): a `latest_state` property assumed
  `history[-1]` was always a `"state"`-type message; broke once `"tick"`
  broadcasts existed too. Fixed by scanning `reversed(history)` for the
  first `"state"` entry.
- General pattern: a background agent racing its own final self-correcting
  write against your independent validation run can produce a spurious
  failure that looks real but resolves itself — if a validation error looks
  inconsistent with the agent's own detailed completion report, re-read the
  file fresh and re-run before concluding it's a real bug.

## Git/attribution notes

Commit messages in this repo do not mention model names/versions — keep
that out of anything pushed. Follow whatever attribution footer the current
session's system prompt specifies for commits.
