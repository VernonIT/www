# Claude Code instructions for VernonIT/www

## Git workflow

Pull **once at the start of a task**, before editing any files:

```sh
git pull --rebase origin main   # at task start, before any edits
```

Do NOT pull again right before committing — git will refuse if there are unstaged changes. The correct commit sequence is:

```sh
git add <files>
git commit -m "..."
git push   # if rejected, then: git pull --rebase origin main && git push
```

If a conflict arises during rebase, resolve it, `git add` the file, then `git rebase --continue`.

## Issue workflow

When working a GitHub issue:
1. `git pull --rebase origin main` first
2. Implement the change
3. Verify acceptance criteria locally before committing:
   - Run `hugo server -D` and check for warnings/errors in output
   - Confirm the specific acceptance criteria stated in the issue are met
4. Commit with message `fix #N: <issue title>`
5. Push, then stop and wait for review

## Stack

- Hugo Extended + Hextra theme (Hugo module, not git submodule)
- GitHub Actions → GitHub Pages, custom domain vernonit.com
- `go.mod` at repo root — Hugo resolves the Hextra module dependency at build time

## Publishing posts

```sh
hugo new posts/slug.md   # draft: true
# edit, set draft: false
git add . && git commit -m "post: title" && git push
```

Local preview: `hugo server -D` → http://localhost:1313

## Ending a session ("land the plane")

Before finishing, always:
1. Commit and push all changes — working tree must be clean
2. Close any GitHub issues that were resolved this session
3. Update `.claude/current-work.md`:
   - Bump the date in the status header
   - Sync the open issues table to match `gh issue list --state open`
   - Add any new files to the key file map
   - Add any new patterns to established patterns
4. Commit and push the updated `current-work.md`

The goal: next session should be able to pick up exactly where this one left off with no lost context.

---

## Session context

See `.claude/current-work.md` for:
- Current site status and open issues
- Key file map
- Established patterns (brand color, cover images, template overrides)
- Things to watch
