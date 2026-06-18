---
title: "Git Best Practices for Trunk-Based, Continuous-CD Teams"
date: 2026-06-18
draft: false
tags: ["git", "devops", "ci-cd", "trunk-based-development", "engineering"]
categories: ["engineering"]
description: "Ten rules for keeping trunk green and deployable in a world where every merge ships."
showToc: true
cover:
  image: "https://images.unsplash.com/photo-1556075798-4825dfaaf498?w=1200&q=80"
  alt: "Code on a screen with git diff output"
---

## The Golden Rule

Trunk is always deployable. What lands on trunk ships — so treat every merge like it's going to production, because it is.

Everything below follows from that one rule.

---

## 1. Sync before you start, and often

Run `git pull --rebase` on trunk first thing, and again throughout the day. The longer you drift from trunk, the worse the merge.

```sh
git pull --rebase origin main
```

Set a habit: pull before you pick up a ticket, pull after lunch, pull before you open a PR.

## 2. Keep branches short-lived

Hours to a day — not weeks. Branch, do one small thing, merge, delete. Long branches accumulate conflicts, go stale, and take down whoever reviews them.

If a branch is living past two days, it's probably too big. Split it.

## 3. Commit small and often

Tiny, logical commits are easier to review, easier to revert, and easier to reason about than one giant blob. Each commit should be a single coherent change — not a save point.

```sh
# Good: one concern per commit
git commit -m "Add retry logic to payment processor"
git commit -m "Add unit tests for payment retry"

# Bad: one blob of everything
git commit -m "WIP payment stuff"
```

## 4. Never break trunk

Run the build and tests **locally** before you push. If CI would fail, don't push it. A red trunk blocks every deploy for the whole team.

This isn't optional. Broken trunk is a team emergency, not someone else's problem to fix.

## 5. Hide unfinished work behind feature flags

Long branches exist because people don't want to merge incomplete features. Feature flags solve that. Merge early, keep the code dark until it's ready.

This lets you:
- Ship the scaffolding safely
- Test in production with a limited audience
- Roll back instantly without reverting commits

## 6. Write commit messages a stranger could understand

Present tense. What + why. The diff shows *what changed* — the message should explain *why it changed*.

```
# Good
Fix null check in payment retry — processor returns null on timeout

# Bad
fixes
wip
minor changes
```

Future you, and your teammates, will thank you.

## 7. Keep PRs small

Small PRs get reviewed in minutes. Big PRs rot for days. Aim for something a reviewer can understand in a single sitting.

If you find yourself writing a two-paragraph PR description to explain all the moving parts, the PR is probably too large.

## 8. Never force-push shared branches

Force-push only your own un-shared branch — and even then, carefully. Force-pushing a branch others have checked out rewrites history they've already built on, which creates a painful mess.

Trunk is strictly off-limits. Full stop.

## 9. Roll forward or revert fast

When something breaks in trunk, you have two options: fix it fast, or revert it fast. Don't leave trunk broken while you investigate.

```sh
# Revert the offending commit cleanly
git revert <sha>
git push
```

`git revert` is your friend. It creates a new commit that undoes the change — no history rewriting, no drama, safe on shared branches.

## 10. When in doubt, ask before you push

A 30-second question is cheaper than a broken deploy. If you're not sure whether a change is safe to push, ask. No one has ever been criticized for checking first.

---

## TL;DR

Trunk is sacred. Keep it green. Keep it shippable. Short branches, small commits, fast reverts. Everything else is just detail.
