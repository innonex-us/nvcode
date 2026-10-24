# Security

## Reporting a problem

Please report anything that lets nvcode escape its intended limits, leak an API
key, or run something the permission mode should have blocked through a
[private security advisory](https://github.com/innonex-us/nvcode/security/advisories/new)
rather than a public issue. Include `nvcode --version`, your OS and
architecture, and the exact command.

## What the sandbox guarantees

nvcode is a **path and prompt sandbox, not an OS sandbox**. Know the boundary
before you use `full-auto`.

| Guarantee | How it is enforced | What it is not |
| --- | --- | --- |
| File tools stay inside the project root | Paths are normalized and the existing parent is canonicalized, so `..` and symlinks cannot escape | Not a mount namespace or a chroot; `run_shell` is a separate story |
| `run_shell` runs in the project with a timeout | Working directory pinned to the root, `sh -lc`, timeout clamped to 1–600 s | Not bubblewrap, a container, or a syscall filter. `full-auto` can run whatever `sh` can run, including `cd /` |
| Writes and shell need consent | `suggest` asks for every write and every shell or side-effecting MCP call, `auto-edit` asks for shell, `full-auto` asks for nothing | A model that is being prompt-injected from file or web content can look like a normal request in `full-auto` |
| Non-interactive runs cannot approve | In `-p` / `exec` without `--yes`, every "ask" becomes a deny | `--yes` forces `full-auto` inside the project |
| Sub-agents cannot escalate | Children get their own transcript, cannot spawn further agents, and share one write lock so mutations serialize | They still run with the same permission mode |
| Secrets stay out of shared files | A project `.nvcode/config.toml` cannot set the API key; `config.toml`, `mcp.json`, `history.json`, and sessions are written mode `600` | `NVIDIA_API_KEY` in the environment is visible to every process `run_shell` starts |
| MCP servers are separate processes | stdio, Streamable HTTP, and HTTP+SSE servers are configured by you, and approvals follow the permission mode | A third-party server is trusted code. Read what you add, and note that `readOnlyHint: true` is the server's own claim |
| Plugin hooks are separate commands | Hook payloads come on stdin with `NVCODE_HOOK_EVENT`; a non-zero exit denies a `pre_tool_use` hook; archives with absolute or `..` paths are rejected | Plugin code is not reviewed by anyone. Installing a plugin runs its commands |

## Reading the notes

- `db_explain` refuses production-looking hosts, database names, and `APP_ENV`
  values, and only runs a read-only `EXPLAIN`. Point it at a dev database.
- Worktree isolation keeps parallel tasks from overwriting each other, but a
  merge into your branch is a real merge. nvcode leaves a dirty checkout alone
  unless you confirm a stash.
- The updater checks downloads against `SHA256SUMS` from the release when that
  asset exists, and refuses to install when the checksum does not match.
