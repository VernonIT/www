+++
title = "Securing an Azure-Hosted React + C# API with MSAL.js"
date = 2025-03-12
draft = false
tags = ["azure", "msal", "authentication", "react", "csharp", "dotnet"]
categories = ["development", "security"]
description = "A practical guide to configuring MSAL.js on a React frontend and Microsoft.Identity.Web on a C# REST API, both hosted on Azure App Service behind an Application Gateway."
+++

Building an internal app on Azure with a React frontend and a C# REST API is a common pattern — but getting authentication wired up correctly across both layers has a lot of subtle gotchas. This post walks through the full setup based on real questions that came up during a recent project, covering everything from app registrations to popup login issues.

---

## The Architecture

For an internal-only app running multiple services on a shared App Service Plan behind an Application Gateway, the recommended setup looks like this:

```
Internal App Gateway (WAF)
├── app1.internal.com  → App Service Plan
│                          ├── app1-frontend (React + MSAL)
│                          ├── app1-api (C# REST API)
│                          ├── app2-frontend
│                          └── app2-api
```

Since the app is internal only, a Static Web App is not the right fit here — SWA's main benefits are its global CDN and edge caching, which are irrelevant for internal traffic, and getting it truly private behind an App Gateway is painful. With a shared App Service Plan already in play, hosting the React frontend there adds negligible cost and keeps networking simple.

---

## App Registrations: How Many Do You Need?

You need **two app registrations** — one for the frontend, one for the backend API. You do **not** need separate registrations per environment.

| Registration | Represents |
|---|---|
| `myapp-frontend` | React SPA |
| `myapp-api` | C# REST API |

Handle environments by adding all redirect URIs to the same frontend registration:

```
http://localhost:3000
https://myapp-staging.internal.com
https://myapp.internal.com
```

Drive the active URI via an environment variable in your MSAL config:

```js
const msalConfig = {
  auth: {
    clientId: "<same-client-id-across-all-envs>",
    redirectUri: process.env.REACT_APP_REDIRECT_URI,
  }
};
```

The backend registration can be shared across all environments since it has no redirect URIs — it just validates tokens. The only reason to split it is if you need different token policies per environment.

### When separate registrations per environment make sense

- Strict compliance or regulated industry requirements
- Different teams own prod vs. non-prod
- You need different token lifetimes per environment

For a standard internal app, two registrations total is the right call.

---

## Configuring the Frontend App Registration

In **Azure AD → App Registrations → [your frontend app] → Authentication**:

1. Add your app under **Single-page application** platform type (not Web). This is critical — the SPA platform type tells Azure AD to use Authorization Code + PKCE instead of implicit flow.

2. Scroll to **Implicit grant and hybrid flows** and **uncheck both boxes**:
   - ☐ Access tokens
   - ☐ ID tokens

   When the platform is correctly set to SPA, these are not needed. Leaving them checked weakens your security posture by keeping the implicit flow available as a fallback.

3. Hit **Save**.

Your MSAL config should look like this:

```js
const msalConfig = {
  auth: {
    clientId: "<frontend-client-id>",
    authority: "https://login.microsoftonline.com/<tenant-id>",
    redirectUri: "https://myapp.internal.com",
  },
  cache: {
    cacheLocation: "sessionStorage", // avoid localStorage for security
  },
};

const tokenRequest = {
  scopes: ["api://<backend-client-id>/access_as_user"],
};
```

---

## Configuring the Backend API

Register a separate app for the API in Azure AD:

- Expose a scope: `api://<backend-client-id>/access_as_user`
- Under **Manifest**, set `accessTokenAcceptedVersion: 2` for v2 tokens
- Under **Authorized client applications**, add your frontend's client ID to pre-authorize it

In your C# project:

**Program.cs**
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

builder.Services.AddAuthorization();

app.UseAuthentication();
app.UseAuthorization();
```

**appsettings.json**
```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "<tenant-id>",
    "ClientId": "<backend-client-id>",
    "Audience": "api://<backend-client-id>"
  }
}
```

Protect your controllers with `[Authorize]` as the default, and opt out selectively with `[AllowAnonymous]` — not the other way around.

---

## App Service Authentication Blade: Leave It Off

This is a critical gotcha. Azure App Service has a built-in "Easy Auth" feature under the Authentication blade. **Do not enable it on either app** if you're handling tokens yourself with MSAL and `Microsoft.Identity.Web`. Easy Auth and code-level auth conflict and cause double-validation issues.

| Setting | Frontend App Service | Backend App Service |
|---|---|---|
| App Service Authentication | **Off** | **Off** |
| CORS | Set to frontend origin | Set to frontend origin |
| HTTPS Only | **On** | **On** |
| TLS Minimum | 1.2 | 1.2 |

Set CORS on the backend App Service to allow only your frontend's specific origin — never use `*`.

---

## Fixing Swagger UI 401s Locally

Getting a 401 from local Swagger UI is expected — Swagger isn't sending a Bearer token by default. Two things to fix:

### 1. Add Bearer token support to Swagger

```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Paste your JWT token here (without 'Bearer ' prefix)"
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});
```

### 2. Get a token from your frontend

Temporarily log the access token to the console from your React app:

```js
const { instance, accounts } = useMsal();

const tokenRequest = {
  scopes: ["api://<backend-client-id>/access_as_user"],
  account: accounts[0]
};

const response = await instance.acquireTokenSilent(tokenRequest);
console.log(response.accessToken);
```

Copy that token, click **Authorize 🔒** in Swagger UI, paste it in (no `Bearer` prefix), and you're good.

Also make sure `http://localhost:<port>` is added as a redirect URI under the SPA platform in your frontend app registration, or token acquisition from localhost will fail.

---

## The `isAuthenticated` / `accounts.length` Gotcha

A very common MSAL issue: your `useEffect` thinks the user is logged in before they've actually authenticated. This happens because `accounts` is populated by `msalInstance.getAllAccounts()`, which reads from the token cache on initialization. If a previous session exists in cache, `accounts.length > 0` is immediately true.

The fix is to check MSAL's `inProgress` flag before acting on account state:

```js
import { useMsal, useIsAuthenticated } from "@azure/msal-react";
import { InteractionStatus } from "@azure/msal-browser";

const { accounts, inProgress } = useMsal();
const isAuthenticated = useIsAuthenticated();

useEffect(() => {
  if (inProgress !== InteractionStatus.None) return;

  if (isAuthenticated && accounts.length > 0) {
    // Safe to treat user as logged in here
  }
}, [isAuthenticated, accounts, inProgress]);
```

Don't try to reset `accounts` or `isAuthenticated` manually — they're derived from MSAL's cache, not values you instantiate. If you need a clean slate during development, use `msalInstance.clearCache()` instead.

---

## Fixing the Login Popup Showing Your App

If the login popup is opening but showing your app inside it instead of the Azure AD login screen, the most likely causes are:

**Redirect URI mismatch** — the URI registered in Azure AD doesn't exactly match what MSAL is sending. Check for trailing slashes, http vs. https, and port numbers.

**Your app loading inside the popup** — for popup flows, MSAL recommends pointing the redirect to a minimal blank page so your full app doesn't load inside the popup:

```html
<!-- public/auth.html -->
<!DOCTYPE html>
<html>
  <head><title>Auth</title></head>
  <body>
    <script src="https://alcdn.msauth.net/browser/2.x.x/js/msal-browser.min.js"></script>
  </body>
</html>
```

```js
auth: {
  redirectUri: "http://localhost:3000/auth.html",
  navigateToLoginRequestUrl: false,
}
```

For an internal app where UX polish isn't the top priority, consider switching to redirect flow instead — it's simpler and has fewer edge cases than popup:

```js
await instance.loginRedirect({
  scopes: ["openid", "profile", "api://<backend-client-id>/access_as_user"]
});

// Handle in your root component:
instance.handleRedirectPromise().then((response) => {
  if (response) console.log("Logged in:", response.account);
});
```

Check the browser console for the MSAL error code — the `error_description` field will pinpoint the exact cause.

---

## Security Checklist

- Use Authorization Code + PKCE — never implicit flow
- Store tokens in `sessionStorage`, not `localStorage`
- Never decode tokens client-side for auth decisions
- Default all controllers to `[Authorize]`, opt out with `[AllowAnonymous]`
- `Microsoft.Identity.Web` validates `aud`, `iss`, `tid`, and `scp` claims automatically — don't disable this
- Use Managed Identity for any service-to-service calls from the API
- Enable Diagnostic Logs on both App Services to catch auth failures early
- Do not enable Easy Auth if you're using `Microsoft.Identity.Web`
