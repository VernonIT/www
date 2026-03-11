---
title: "Hello World"
date: 2026-03-10
draft: false
tags: ["meta", "engineering"]
categories: ["blog"]
description: "First post on VernonIT — what this site is and what's coming."
---

## Welcome to VernonIT

This is the first post on [vernonit.com](https://vernonit.com) — a site about full-lifecycle product engineering: web apps, backend infrastructure, and the craft of shipping software.

### What to expect

- Deep dives into systems architecture and engineering decisions
- Lessons from building and scaling production systems
- Practical guides on tooling, workflows, and automation

The site is built with [Hugo](https://gohugo.io) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed automatically to GitHub Pages via GitHub Actions on every push to `main`.

### Publishing workflow

Every new post follows a simple path:

```sh
hugo new posts/post-title.md
# write, set draft: false
git add . && git commit -m "post: title" && git push
```

~2 minutes later it's live at `https://vernonit.com/posts/post-title/`.

More soon.
