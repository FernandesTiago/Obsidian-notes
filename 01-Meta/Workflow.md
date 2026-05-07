
How I actually use this vault day-to-day.

## Folder logic

The split is by **intent**, not topic:

- **Studies/** — what I'm *learning* (concepts, tutorials, course notes)
- **Runbooks/** — what I *configured* on my machines (recovery steps if something breaks)
- **Meta/** — about the vault itself
- **Inbox/** — rough captures, organize later

Mental test: "Did I do this on one of my machines?" → Runbook. Otherwise → Study.

## Privacy

Anything sensitive (real IPs, ports, hostnames, tokens) stays in `Runbooks/00-Private/`. That folder is in `.gitignore`. Public notes reference it with `[[PW]]` links — Obsidian resolves the link locally, GitHub never sees the values.

Tokens and passwords go to Bitwarden, not the vault. The vault only references *where* the secret lives.

## Runbook template

Every runbook should answer:

1. **Objective** - what problem this solves
2. **State** - what's running right now, how to verify
3. **Setup** - exact commands I ran, in order
4. **Verify** - how to confirm it works
5. **Troubleshoot** - errors I hit + how I fixed them
6. **Revert** - how to undo if needed
7. **References** - links to related notes

Skip sections that don't apply, but keep the order consistent.

## Daily flow

1. Open vault → check `Meta/TODO.md`
2. Capture rough ideas in `Inbox/` as they come
3. After finishing a config session → write/update the runbook same day (memory fades fast)
4. Commit + push before closing

## Linking

Use `[[note-name]]` liberally. Internal links are free and turn the vault into a graph instead of a folder tree. The graph view (Ctrl+G) gets useful around 30+ notes.

## Sync setup

- **Desktop + notebook:** `obsidian-git` plugin → auto-commit every 30min, auto-push, auto-pull on startup
- **Mobile:** not yet — planned for when self-hosted Nextcloud is running on the Pi

## Things to avoid

- Copying official docs (link them instead)
- Documenting what's obvious (`ls`, `cd`)
- Putting secrets anywhere except `00-Private/`
- Letting Inbox grow past 10 notes — process them weekly