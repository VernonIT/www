# VernonIT/www — Current Work

## Site status (as of 2026-03-13)

Live at https://vernonit.com. Hugo Extended + Hextra v0.12.1 + GitHub Pages.

### Pages
- `/` — Hero with dot-grid background, service feature cards, recent posts section (blog-card style)
- `/about` — Placeholder copy (issue #34 open — David to write)
- `/blog` — 2-column card grid layout, cover images, 2 posts live
- `/contact` — Formspree form (ID: `myknjgwe`)

### Blog posts
- `split-horizon-dns-azure-ase.md` — YAML front matter (`---`)
- `securing-azure-app-with-msal.md` — TOML front matter (`+++`)

---

## Open issues (lowest first)

| # | Title | Owner |
|---|---|---|
| #34 | Write real About page copy | David |
| #46 | Add social/contact links to footer (icons + email) | Claude |

---

## Key file map

| File | Purpose |
|---|---|
| `hugo.toml` | Site config, theme module, navbar menu, blog sort order, brand params |
| `assets/css/custom.css` | Brand color vars, hero background, blog cards, contact form, footer, signature |
| `content/_index.md` | Homepage — uses `layout: hextra-home` |
| `content/about.md` | About page |
| `content/contact.md` | Contact page with raw HTML Formspree form |
| `content/blog/_index.md` | Blog section index |
| `layouts/blog/single.html` | Overrides Hextra — adds cover image + signature sign-off |
| `layouts/blog/list.html` | Overrides Hextra — 2-col card grid layout |
| `layouts/_shortcodes/recent-posts.html` | Custom shortcode used on homepage — uses same .blog-card classes as listing |
| `layouts/_partials/custom/footer.html` | Custom footer: © year + GitHub/LinkedIn links |
| `layouts/_partials/opengraph.html` | Overrides Hextra — adds cover.image as og:image fallback |
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

### Blog card markup
Both `layouts/blog/list.html` and `layouts/_shortcodes/recent-posts.html` use the same classes:
```html
<div class="blog-card-grid">
  <article class="blog-card">
    <a class="blog-card-image">…</a>   <!-- overflow:hidden here only, not on .blog-card -->
    <div class="blog-card-body">
      <p class="blog-card-date">…</p>
      <h3 class="blog-card-title"><a>…</a></h3>
      <p class="blog-card-excerpt">…</p>
      <a class="blog-card-read-more">…</a>
    </div>
  </article>
</div>
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
Cover image is also used as `og:image` automatically (via `layouts/_partials/opengraph.html`).

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
- `layouts/_partials/opengraph.html`

### Custom partial slots (Hextra)
Clean override slots: `layouts/_partials/custom/footer.html`, `navbar-title.html`, `head-end.html`.
For anything else, override the full layout file.

---

## Things to watch
- `hero.txt` is sitting in the repo root (untracked, not gitignored) — safe to delete once hero is final
- Unsplash cover image URLs may need replacing with owned assets long-term
- LinkedIn URL confirmed: https://www.linkedin.com/in/vidv
