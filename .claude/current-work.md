# VernonIT/www — Current Work

## Site status (as of 2026-03-12)

Live at https://vernonit.com. Hugo Extended + Hextra v0.12.1 + GitHub Pages.

### Pages
- `/` — Hero with dot-grid background, service feature cards, recent posts section
- `/about` — Placeholder copy (issue #34 open — David to write)
- `/blog` — Listing page, newest first, cover images, 2 posts live
- `/contact` — Formspree form (ID: `myknjgwe`)

### Blog posts
- `split-horizon-dns-azure-ase.md` — YAML front matter (`---`)
- `securing-azure-app-with-msal.md` — TOML front matter (`+++`)

---

## Open issues (lowest first)

| # | Title | Owner |
|---|---|---|
| #34 | Write real About page copy | David |
| #44 | Style blog listing page — card layout | Claude |
| #45 | Tighten homepage section spacing and typography | Claude |
| #46 | Add social/contact links to footer | Claude |
| #47 | Add Open Graph meta tags for social sharing | Claude |

---

## Key file map

| File | Purpose |
|---|---|
| `hugo.toml` | Site config, theme module, navbar menu, blog sort order, brand params |
| `assets/css/custom.css` | Brand color vars, hero background, cover images, contact form, footer, signature |
| `content/_index.md` | Homepage — uses `layout: hextra-home` |
| `content/about.md` | About page |
| `content/contact.md` | Contact page with raw HTML Formspree form |
| `content/blog/_index.md` | Blog section index |
| `layouts/blog/single.html` | Overrides Hextra — adds cover image + signature sign-off |
| `layouts/blog/list.html` | Overrides Hextra — adds cover image thumbnails to listing |
| `layouts/_shortcodes/recent-posts.html` | Custom shortcode used on homepage |
| `layouts/_partials/custom/footer.html` | Custom footer: © year + GitHub/LinkedIn links |
| `static/vernonit-icon.png` | Navbar logo (32×32, #006699 triangle) |
| `static/vernonit-signature.svg` | Signature used as post sign-off |
| `static/CNAME` | Custom domain — must not be deleted |

---

## Established patterns

### Brand color
`#006699` = `hsl(200, 100%, 30%)`. Set via CSS vars in `custom.css`:
```css
:root { --primary-hue: 200; --primary-saturation: 100%; --primary-lightness: 30%; }
.dark { --primary-lightness: 55%; }
```

### Cover images (front matter)
YAML posts:
```yaml
cover:
  image: "https://images.unsplash.com/photo-XXXX?w=1200&q=80"
  alt: "Description"
```
TOML posts:
```toml
[cover]
  image = "https://images.unsplash.com/photo-XXXX?w=1200&q=80"
  alt = "Description"
```

### Adding a blog post
```sh
# Copy/create file in content/blog/
# YAML (---) or TOML (+++) front matter both work
# draft: false to publish
git add content/blog/my-post.md
git commit -m "post: title"
git push
```

### Hextra icon names (useful ones)
`cloud`, `code`, `cog`, `server`, `terminal`, `users`, `globe`, `lightning-bolt`, `puzzle`, `chart-bar`, `briefcase`, `academic-cap`

### Overriding Hextra templates
Copy from `~/Library/Caches/hugo_cache/modules/filecache/modules/pkg/mod/github.com/imfing/hextra@v0.12.1/layouts/`
into `layouts/` at the same relative path. Already overridden:
- `layouts/blog/single.html`
- `layouts/blog/list.html`

### Custom partial slots (Hextra)
Only `layouts/_partials/custom/footer.html` and `navbar-title.html` are available as clean override slots.
For anything else, override the full layout file.

---

## Things to watch
- `hero.txt` is sitting in the repo root (not committed, not gitignored) — safe to delete once hero is final
- Unsplash cover image URLs may need replacing with owned assets long-term
- LinkedIn URL in footer (`/in/davidvernon`) — verify this is correct slug
