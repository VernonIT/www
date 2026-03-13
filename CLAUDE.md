# Claude Code instructions for VernonIT/www

## Git workflow

Always pull and rebase before committing:

```sh
git pull --rebase origin main   # before any commit
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

## Session context

See `.claude/current-work.md` for:
- Current site status and open issues
- Key file map
- Established patterns (brand color, cover images, template overrides)
- Things to watch
