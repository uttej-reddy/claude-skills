# CLAUDE.md — Global

## Environment
- macOS, zsh, Node v25.2.1
- Use `python3`, `pip3` — never Windows equivalents

## Non-Default Behaviors
- No trailing summaries after completing work
- No emojis

## Skills
Location: `~/.claude/skills/<name>/SKILL.md`

**`${PLACEHOLDER}` variables are NOT auto-substituted.** Each skill must explicitly read profile data from:
`~/.claude/skills/job-data/profile.json`

## MCP
- `playwright` — Chrome headed, for job application automation (`~/.claude/mcp.json`)
- If "Failed to connect": verify command in `mcp.json` matches `claude mcp list` output

## Memory
`~/.claude/projects/-Users-uttejreddy-workspace-claude-projects/memory/MEMORY.md`
Read this at the start of job-search sessions for pipeline state and prior feedback.

## Projects
| Project | Path | Stack |
|---------|------|-------|
| bookmarkx | `~/workspace/claude-projects/bookmarkx` | Next.js 15, TypeScript, Prisma |
| KnowAboutMe | `~/workspace/claude-projects/KnowAboutMe` | FastAPI, Python, ChromaDB |

Each has its own `CLAUDE.md` with commands and architecture.
