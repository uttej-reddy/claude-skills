# claude-skills

My personal Claude Code configuration — global instructions, MCP servers, plugins, and custom skills.

## Structure

```
~/.claude/
├── CLAUDE.md           # Global instructions loaded in every Claude session
├── settings.json       # Enabled plugins
├── mcp.json            # MCP server definitions (Playwright, etc.)
└── skills/             # Custom slash-command skills
    ├── cover-letter/   # Generate tailored cover letters
    ├── job-apply/      # Browser-automated job applications
    ├── job-roles/      # Discover and search job roles
    ├── job-tracker/    # Manage application pipeline (GoogleDrive/One Drive)
    ├── linkedin-referral-search/  # Find referrers at target companies
    └── referral-ask/   # Generate referral request messages
```

## What's NOT tracked

| Path | Reason |
|------|--------|
| `skills/job-data/` | Personal info — name, email, phone, address |
| `skills/job-apply/portals.json` | Portal credentials |
| `skills/leetcode-solver/` | Private |
| `channels/` | Telegram bot token and access list |
| `projects/` | Conversation history |
| `plugins/` | Installed on demand, not portable |

## Setup on a new machine

```bash
# 1. Install Claude Code
npm install -g @anthropic-ai/claude-code

# 2. Clone this repo into ~/.claude
git clone <repo-url> ~/.claude

# 3. Restore secrets (not in repo — keep separately)
#    ~/.claude/skills/job-data/profile.json
#    ~/.claude/skills/job-apply/portals.json
#    ~/.claude/channels/telegram/.env

# 4. Install plugins
claude plugins install telegram
claude plugins install skill-creator
claude plugins install claude-md-management
```
