----
Please update to https://github.com/dmtrKovalenko/fff.nvim
We've migrated the pi-fff extension to be a first class citizen within the fff mono repo.
---

# pi-fff

A [pi](https://github.com/badlogic/pi-mono) extension that replaces the built-in `find` and `grep` tools with [FFF](https://github.com/dmtrKovalenko/fff.nvim) — a Rust-native, SIMD-accelerated file finder with built-in memory.

## What it does

| Built-in tool | pi-fff replacement | Improvement |
|---|---|---|
| `find` (spawns `fd`) | `find` (FFF `fileSearch`) | Fuzzy matching, frecency ranking, git-aware, pre-indexed |
| `grep` (spawns `rg`) | `grep` (FFF `grep`) | SIMD-accelerated, frecency-ordered, mmap-cached, no subprocess |
| *(none)* | `multi_grep` (FFF `multiGrep`) | OR-logic multi-pattern search via Aho-Corasick |
| `@` file autocomplete (fd-backed) | `@` file autocomplete (FFF-backed, default) | Fuzzy ranking from FFF index/frecency |

### Key advantages over built-in tools

- **No subprocess spawning** — FFF is a Rust native library called through the Node binding. No `fd`/`rg` process per call.
- **Pre-indexed** — files are indexed in the background at session start. Searches are instant.
- **Frecency ranking** — files you access often rank higher. Learns across sessions.
- **Query history** — remembers which files were selected for which queries. Combo boost.
- **Git-aware** — modified/staged/untracked files are boosted in results.
- **Smart case** — case-insensitive when query is all lowercase, case-sensitive otherwise.
- **Fuzzy file search** — `find` uses fuzzy matching, not glob-only. Typo-tolerant.
- **Cursor pagination** — grep results include a cursor for fetching the next page.

## Install

Requirements:
- pi

### Install as a pi package

Global install:

```bash
pi install git:github.com/SamuelLHuber/pi-fff
```

Project-local install:

```bash
pi install -l git:github.com/SamuelLHuber/pi-fff
```

This is the recommended installation method. pi will clone the repo, install dependencies, and load the extension from the `pi` manifest in `package.json`.

### Pin to a release

If you want a fixed version instead of tracking the default branch:

```bash
pi install git:github.com/SamuelLHuber/pi-fff@v0.2.3
```

### Local development / manual install

```bash
git clone https://github.com/SamuelLHuber/pi-fff.git
cd pi-fff
bun install
```

Then add to your pi `settings.json`:

```json
{
  "extensions": ["/path/to/pi-fff/src/index.ts"]
}
```

Or test directly:

```bash
pi -e /path/to/pi-fff/src/index.ts
```

This extension overrides pi's built-in `find` and `grep` tools by registering tools with the same names.

## Tools

### `grep` (overrides built-in)

Search file contents. Smart case, plain text by default, regex optional.

Parameters:
- `pattern` — search text or regex
- `path` — directory/file constraint (e.g. `src/`, `*.ts`)
- `ignoreCase` — force case-insensitive
- `literal` — treat as literal string (default: true)
- `context` — context lines around matches
- `limit` — max matches (default: 100)
- `cursor` — pagination cursor from previous result

### `find` (overrides built-in)

Fuzzy file name search. Frecency-ranked.

Parameters:
- `pattern` — fuzzy query (e.g. `main.ts`, `src/ config`)
- `path` — directory constraint
- `limit` — max results (default: 200)

### `multi_grep` (new)

OR-logic multi-pattern content search. SIMD-accelerated Aho-Corasick.

Parameters:
- `patterns` — array of literal patterns (OR logic)
- `constraints` — file constraints (e.g. `*.{ts,tsx} !test/`)
- `context` — context lines
- `limit` — max matches (default: 100)
- `cursor` — pagination cursor

## Commands

- `/fff-health` — show FFF status (indexed files, git info, frecency/history DB status)
- `/fff-rescan` — trigger a file rescan
- `/fff-mode both|tools-only` — switch mode and persist it

## Modes

- `both` (default): tool overrides + `@` autocomplete replacement in UI
- `tools-only`: only tool overrides; keep pi's default fd-backed `@` autocomplete

Mode precedence:
1. `--fff-mode <mode>` CLI flag
2. `PI_FFF_MODE=<mode>` environment variable
3. persisted config (`~/.pi/agent/fff/config.json`)
4. default (`both`)

## Data

FFF stores frecency and query history databases in `~/.pi/agent/fff/`:
- `frecency.mdb` — file access frequency/recency
- `history.mdb` — query-to-file selection history

No project files are uploaded anywhere by this extension. It runs locally and only uses the configured LLM through pi itself.

## Security

- No shell execution
- No network calls in the extension code
- No telemetry
- No credential handling beyond whatever pi and your configured model provider already do
- Search state is stored locally under `~/.pi/agent/fff/`
