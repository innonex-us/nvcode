# Changelog

Release notes for the nvcode binaries published in this repository.

## Unreleased

### Fixed

- The inline UI redraws its input region in place. Earlier builds moved the
  cursor back by the height of that region instead of by the row the cursor was
  left on, so every frame after the first overwrote the session header and
  earlier transcript lines.

## 0.2.1

### Fixed

- Unix release archives keep the executable bit on `nvcode`, so the tarballs
  run directly after extraction.

## 0.2.0

### Added

- Inline terminal UI: an inline viewport keeps the conversation in normal
  scrollback, with a fixed editor, header, streaming markdown, highlighted
  diffs, approval prompts, slash completion, input history, and dark, light, or
  no-color themes. `--plain`, a non-TTY, and `-p` / `exec` keep the
  line-oriented printer.
- Live task graph in the inline UI (Ctrl+T), driven by scheduler events for
  planned, started, waiting, progress, done, failed, and merge steps.
- Automatic multi-agent mode: a fast-model assessment escalates complex work to
  a DAG scheduler. Each ready task runs in its own git worktree, dependent tasks
  branch from an integration branch and see upstream summaries, files, and
  blackboard notes. Cycles are rejected, failed tasks retry, replan, then ask.
  `multi_agent`, `worktrees`, `max_parallel_agents`, `--multi`,
  `NVCODE_MULTI_AGENT`, `NVCODE_MAX_PARALLEL`, and `/worktree` control it.
  Verification runs before the merge; conflicts go to a merge agent.
- PHP specialist: `composer.json`, `*.php`, `artisan`, or `symfony.lock`
  enables a PHP pack (framework and runtime detection, modern syntax, PER /
  PSR-12, OWASP Top 10, database rules). `--stack`, `NVCODE_STACK`, and `stack`
  select `auto`, `php`, or `off`. Verification runs `php -l`, phpstan/Larastan,
  Pint or php-cs-fixer, PHPUnit or Pest, and `composer audit`; production
  readiness is checked before completion. `php-security-reviewer` and
  `db-optimizer` run in multi-agent mode. `--db-explain` / `php.db_explain` adds
  a read-only `db_explain` tool, and `/deps` reports outdated and vulnerable
  packages.
- Self-update: interactive sessions check releases in the background and print
  `nvcode vX.Y.Z is available — run /update to install` when a newer version
  exists (cached for six hours; `check_update` and `NVCODE_NO_UPDATE_CHECK` opt
  out). `/update` and `nvcode update` download the archive for this OS and
  architecture, verify `SHA256SUMS` when present, replace the running binary,
  print the release notes, and restart. `nvcode update --check` only reports.
- Release pipeline and installers: six targets (Linux glibc/musl and aarch64,
  macOS Intel and Apple silicon, Windows x64) plus `.pkg`, `.msi`, `.deb`,
  `.rpm`, AppImage, and a Homebrew formula. `install.sh` and `install.ps1` pick
  the archive, verify `SHA256SUMS`, and copy `nvcode` onto `PATH`.
- Model catalog and NVIDIA API parameters: live-probed role defaults (Nemotron
  3 Ultra coder, MiniMax M3.1 Flash reviewer, Nemotron 3.5 Lightning
  researcher, Nemotron 3.5 Bolt fast/complexity), per-role fallback chains,
  `nvcode models` with context, output, tools, reasoning, vision, and
  deprecated marks, and `nvcode models --probe`. Reasoning traces render as one
  collapsed line. 429 responses honour `Retry-After` and rate-limit reset
  headers, and HTTP 202 is reported with `NVCF-REQID` instead of being treated
  as an answer.

### Fixed

- CLI one-shot output keeps the reviewer text and PHP specialist notes.
- Windows secret prompts compile and disable echo through `kernel32`.
- `fallback_models` still works as a custom chain while the default list maps to
  each role's built-in chain.
