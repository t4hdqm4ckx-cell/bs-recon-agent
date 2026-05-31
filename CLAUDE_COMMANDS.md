# Claude Code Slash Commands

Complete reference for all slash commands available in Claude Code (CLI, desktop, and IDE extensions).

---

## Built-in commands

### Session management

| Command | Description |
|---|---|
| `/clear` | Clear the current conversation history and start fresh |
| `/compact` | Compress conversation history into a summary to free up context |
| `/exit` | Exit the Claude Code session |
| `/rewind` | Undo the last exchange and restore the previous conversation state |

### Configuration & settings

| Command | Description |
|---|---|
| `/config` | Open the Claude Code settings UI |
| `/settings` | View or edit settings (aliases `/config`) |
| `/model` | Switch the active Claude model (Opus, Sonnet, Haiku) |
| `/fast` | Toggle fast mode — uses Claude Opus with faster output |
| `/effort` | Set the effort level: `low`, `medium`, `high`, or `max` |
| `/permissions` | View and manage tool permissions for the current session |

### Context & usage

| Command | Description |
|---|---|
| `/context` | Show context window usage broken down by category (system, tools, messages) |
| `/memory` | View, edit, or clear Claude's persistent memory files |
| `/memories` | Alias for `/memory` |
| `/skills` | List all available skills (built-in and project-defined) |

### Project setup

| Command | Description |
|---|---|
| `/init` | Initialize a `CLAUDE.md` file for the current project with codebase documentation |
| `/install-github-app` | Install the Claude GitHub app for PR review and CI integrations |

### Auth & account

| Command | Description |
|---|---|
| `/login` | Log in to your Anthropic account |
| `/logout` | Log out of your Anthropic account |
| `/doctor` | Run a health check on the Claude Code installation |
| `/feedback` | Submit feedback or a bug report to Anthropic |

### Integrations

| Command | Description |
|---|---|
| `/mcp` | Manage MCP (Model Context Protocol) servers — list, add, remove, view status |
| `/ide` | Manage IDE integrations (VS Code, JetBrains) |
| `/chrome` | Enable the Claude in Chrome browser integration |
| `/hooks` | View and manage automation hooks configured in `settings.json` |

### Utilities

| Command | Description |
|---|---|
| `/agents` | List available agents and their descriptions |
| `/tasks` | View active background tasks |
| `/stop` | Stop a running background task |
| `/sessions` | View recent sessions |
| `/routines` | Manage scheduled routines (see `/schedule` skill) |

---

## Skills (invoked as slash commands)

Skills are project or user-defined capabilities that extend Claude Code. All skills below are built-in unless noted.

### Code quality

| Command | Description |
|---|---|
| `/code-review` | Review the current diff for bugs and improvements. Options: `low`/`medium`/`high`/`max` effort, `--comment` to post as PR comments, `--fix` to apply fixes, `ultra` for deep multi-agent cloud review |
| `/simplify` | Review changed code for reuse, simplification, and efficiency — applies fixes automatically |
| `/security-review` | Run a security review of the pending changes on the current branch |

### Development workflow

| Command | Description |
|---|---|
| `/run` | Launch and drive the project app to observe a change working in the real app |
| `/verify` | Verify that a code change does what it's supposed to by running the app and observing behavior |
| `/init` | Initialize a new `CLAUDE.md` file with codebase documentation |
| `/review` | Review a pull request |

### AI/API development

| Command | Description |
|---|---|
| `/claude-api` | Build, debug, and optimize Claude API / Anthropic SDK applications; handles prompt caching, model migrations, tool use |

### Automation & scheduling

| Command | Description |
|---|---|
| `/loop` | Run a prompt or skill on a recurring interval. Usage: `/loop 5m /skill-name` or omit the interval for self-paced execution |
| `/schedule` | Create, update, list, or run scheduled remote agents on a cron schedule; supports one-time future runs |

### Configuration

| Command | Description |
|---|---|
| `/update-config` | Configure Claude Code settings via `settings.json` — permissions, hooks, env vars, automated behaviors |
| `/keybindings-help` | Customize keyboard shortcuts and rebind keys in `~/.claude/keybindings.json` |
| `/fewer-permission-prompts` | Scan recent transcripts and add an allowlist to reduce repetitive permission prompts |

### Informational

| Command | Description |
|---|---|
| `/effort` | Set effort level: `low` (quick), `medium`, `high`, or `max` (exhaustive) |
| `/statusline-setup` | Configure the Claude Code status line display |
| `/ultrareview` | Deprecated alias for `/code-review ultra` |

---

## Command modifiers & flags

Some commands accept inline arguments:

| Pattern | Example | Effect |
|---|---|---|
| `/<skill> <effort>` | `/code-review high` | Run skill at specified effort level |
| `/<skill> --fix` | `/code-review --fix` | Apply findings automatically |
| `/<skill> --comment` | `/code-review --comment` | Post findings as inline PR comments |
| `/loop <interval> /<skill>` | `/loop 5m /verify` | Run skill on a recurring interval |
| `/code-review ultra <PR#>` | `/code-review ultra 42` | Deep cloud review of a specific GitHub PR |

---

## Notes

- Skills resolve via `/skill-name` — any `.md` file in a `skills/` directory under `CLAUDE.md` discovery paths is automatically available as a slash command.
- Custom project skills take precedence over built-in skills with the same name.
- `/fast` is available on Opus 4.8/4.7/4.6 and does not downgrade to a smaller model.
- `/code-review ultra` and `/ultrareview` are user-triggered and billed separately — Claude cannot launch them autonomously.
- Commands marked as skills (second table) require the skill system to be enabled; use `--disable-slash-commands` to disable all skills.
