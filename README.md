# Vault Workflow

Personal knowledge vault. Split by intent, not topic.
For study notes, technical references, and ongoing learnings.

## Purpose

Centralize knowledge acquired throughout my tech learning journey, focused on long-term retention by writing notes in my own words.

## Structure

- **Meta/** - about the vault itself
- **Studies/** - what I'm learning (CompTIA, CyberSec concepts, Python, Linux fundamentals)
- **Runbooks/** - what I configured on my own machines (Pi, OpenWRT, Linux setups). If something breaks, this is where the recovery steps live.
- **Inbox/** - quick captures before they find a permanent home

## Conventions

- Real IPs, ports, secrets → only in `Runbooks/00-Private/` (gitignored)
- Public notes use placeholders like `<PI_IP>`, `<VPN_PORT>`
- Each runbook has: Objective, Setup, Verify, Troubleshoot, Revert
- Link generously between notes with `[[note-name]]`

## Sync

- Vault is a Git repo, pushed to private GitHub
- Desktop + notebook: `obsidian-git` plugin auto-commits every 30min
- Mobile: not configured yet (planned: self-host on Pi later)

## Daily flow

1. Start session → check `Meta/TODO.md`
2. Capture rough ideas → `00-Inbox/`
3. After completing a config → write/update its runbook same day
4. Commit + push when leaving
