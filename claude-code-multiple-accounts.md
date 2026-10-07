# Running Multiple Claude Code Accounts at the Same Time

Claude Code has no built-in "profiles" feature, but you can run separate **work** and **personal** accounts side by side using the `CLAUDE_CONFIG_DIR` environment variable. Each account gets its own login, so token usage and rate limits stay separate.

## How it works

By default, Claude Code keeps its config, session history, plugins and sign-in in `~/.claude`. When you set `CLAUDE_CONFIG_DIR`, it uses that folder instead. Point each account at its own folder and each one becomes a fully separate login.

On macOS, the OAuth token is stored in the Keychain under a key derived from the config directory path, so the two logins don't overwrite each other.

## Setup (macOS / Linux)

### 1. Add aliases

Put these in `~/.zshrc` or `~/.bashrc`:

```bash
alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work command claude'
alias claude-personal='CLAUDE_CONFIG_DIR=~/.claude-personal command claude'
```

Reload your shell:

```bash
source ~/.zshrc   # or ~/.bashrc
```

### 2. Log in once per account

```bash
claude-work       # then run /login and sign in with your work account
claude-personal   # then run /login and sign in with your personal account
```

### 3. Use them side by side

Open two terminals, run `claude-work` in one and `claude-personal` in the other. Both run at the same time, each on its own account.

## Setup (Windows PowerShell)

Add to your PowerShell profile (`notepad $PROFILE`):

```powershell
function claude-work     { $env:CLAUDE_CONFIG_DIR="$env:USERPROFILE\.claude-work"; claude @args }
function claude-personal { $env:CLAUDE_CONFIG_DIR="$env:USERPROFILE\.claude-personal"; claude @args }
```

## Things to know

| Topic | Details |
| --- | --- |
| **Confirm the account** | Run `/status` in a session to see which account it's signed into. |
| **Nothing is shared** | Each profile has its own `CLAUDE.md`, skills, commands, MCP servers and history. |
| **Sharing config anyway** | Symlink folders you want in both, e.g. `ln -s ~/.claude/commands ~/.claude-work/commands` |
| **GitHub identity** | `gh` supports `GH_CONFIG_DIR` the same way: `GH_CONFIG_DIR=~/.claude-work/gh gh auth login` |
| **Existing login** | Your current `~/.claude` setup is untouched. You can keep using plain `claude` as one of the accounts. |
| **More accounts** | Create another folder, add another alias, log in once. |

## Optional: block the plain `claude` command

To avoid accidentally using the wrong account:

```bash
alias claude="echo 'Use claude-work or claude-personal'"
```

The aliases above use `command claude`, so they still work.

## Sources

- [Claude Code settings docs](https://code.claude.com/docs/en/settings)
- [Codeminer42: Running Two Claude Code Accounts](https://blog.codeminer42.com/ai-tip-running-two-claude-code-accounts/)
- [claude-multiprofile (GitHub)](https://github.com/jmdarre-v/claude-multiprofile)
- [Maestro: Multiple Accounts](https://docs.runmaestro.ai/multi-provider)
