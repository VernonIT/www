+++
title = "Vite: Build Once, Deploy Everywhere"
date = 2026-05-11
draft = false
tags = ["vite", "react", "typescript", "docker", "kubernetes", "ci-cd", "frontend"]
categories = ["development"]
description = "Skip per-environment rebuilds. Inject runtime config into your Vite app via a tiny env.js file — works with Docker, Kubernetes, GitHub Actions, and local dev."

[cover]
  image = "https://images.unsplash.com/photo-1518770660439-4636190af475?w=1200&q=80"
  alt = "Circuit board close-up"
+++

The classic Vite problem: environment variables are baked into the bundle at build time. That means a separate build for dev, staging, and prod — or a single build with hardcoded values that you hope are right. Neither is great.

There's a better pattern: build once, inject config at deploy time via a runtime file. One artifact, promoted straight through your environments.

---

## How It Works

1. Build once — no env-specific values baked in.
2. A tiny `env.js` is loaded at runtime, before your app bundle.
3. Each deployment target has its own `env.js`, injected by CI/CD or your container entrypoint.

---

## Step 1: Create the Runtime Config File

### `public/env.js`

Vite copies everything in `public/` to `dist/` unchanged and serves it from root during dev. This file ships with your build — overwrite it per environment at deploy time.

```js
// Local dev defaults — commit these, they're not secrets
window.__ENV__ = {
  API_URL: "http://localhost:3000",
  AUTH_DOMAIN: "dev.auth.example.com",
  FEATURE_FLAGS: "debug,verbose",
};
```

> **Security note:** `window.__ENV__` is visible to anyone who opens DevTools. Never put secrets, API keys, or tokens here — only public config like base URLs and feature flags.

### `index.html` — Load It Before Your App

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <!-- Must come before your app bundle -->
    <script src="/env.js"></script>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## Step 2: Create a Typed Helper

### `src/env.ts`

```ts
declare global {
  interface Window {
    __ENV__: Record<string, string>;
  }
}

/**
 * Read a runtime env var. Falls back to Vite's import.meta.env
 * so local `vite dev` still works with .env files.
 */
export function env(key: string): string {
  return window.__ENV__?.[key] ?? import.meta.env[`VITE_${key}`] ?? "";
}
```

The optional chaining on `window.__ENV__?.[key]` guards against `env.js` failing to load — the app degrades instead of crashing.

The fallback to `import.meta.env` means `.env.local` still works during `vite dev`. Developers can override individual values there without touching `public/env.js`.

### Usage

```ts
import { env } from "./env";

const api = env("API_URL");      // "https://api.prod.example.com" in prod
const auth = env("AUTH_DOMAIN"); // "auth.prod.example.com" in prod
```

---

## Step 3: Inject at Deploy Time

Pick whichever matches your setup.

### Option A: Docker Entrypoint

```dockerfile
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html/

COPY env.sh /docker-entrypoint.d/50-env.sh
RUN chmod +x /docker-entrypoint.d/50-env.sh
```

#### `env.sh`

```sh
#!/bin/sh
cat <<EOF > /usr/share/nginx/html/env.js
window.__ENV__ = {
  API_URL: "${API_URL}",
  AUTH_DOMAIN: "${AUTH_DOMAIN}",
  FEATURE_FLAGS: "${FEATURE_FLAGS}",
};
EOF
```

The `50-` prefix controls execution order — nginx:alpine runs scripts in `/docker-entrypoint.d/` alphabetically on startup. Pass config as normal container env vars:

```bash
docker run \
  -e API_URL=https://api.prod.example.com \
  -e AUTH_DOMAIN=auth.prod.example.com \
  -e FEATURE_FLAGS=analytics \
  my-app
```

### Option B: CI/CD Pipeline

After build, before deploy — just overwrite the file. GitHub Actions example:

```yaml
- name: Inject env config
  run: |
    cat <<EOF > dist/env.js
    window.__ENV__ = {
      API_URL: "${{ vars.API_URL }}",
      AUTH_DOMAIN: "${{ vars.AUTH_DOMAIN }}",
      FEATURE_FLAGS: "${{ vars.FEATURE_FLAGS }}",
    };
    EOF
```

Use `vars.*` (repository/environment variables) not `secrets.*` — these values end up in a public JS file anyway, so they shouldn't be secrets.

### Option C: Kubernetes ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-env
data:
  env.js: |
    window.__ENV__ = {
      API_URL: "https://api.prod.example.com",
      AUTH_DOMAIN: "auth.prod.example.com",
      FEATURE_FLAGS: "analytics",
    };
---
# In your Deployment spec:
volumeMounts:
  - name: env-config
    mountPath: /usr/share/nginx/html/env.js
    subPath: env.js
volumes:
  - name: env-config
    configMap:
      name: frontend-env
```

`subPath` mounts just that one key as a file rather than replacing the whole directory.

---

## Local Dev Still Works

During `vite dev`, Vite serves `public/env.js` at `/env.js` and `window.__ENV__` gets the dev defaults from the file. The `env()` helper's fallback chain covers you either way:

```
.env.local        → vite dev (import.meta.env.VITE_*)
public/env.js     → built app at runtime (window.__ENV__)
src/env.ts        → unified accessor, checks both
```

---

## The Payoff

One build artifact. Promote it through staging → prod by swapping a config file or passing env vars. No rebuilds, no configuration drift, no "works in staging but not prod because someone forgot to set `VITE_X`."
