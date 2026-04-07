# Cloudflare Pages Deployment

## How It Works

Cloudflare Pages is connected to this GitHub repo and builds automatically:

| Push target | Result |
|-------------|--------|
| `main` | **Production** deploy at the live URL |
| Any other branch | **Preview** deploy at a branch-specific URL |

No API tokens, CI pipelines, or manual triggers needed.

## Preview URL Pattern

Preview URLs follow:

```
https://<branch-slug>.<project>.pages.dev
```

Slashes become hyphens, so `release/update-hero-2025-04-06` becomes:

```
https://release-update-hero-2025-04-06.ailmind.pages.dev
```

## Checking Build Status

After pushing, check if the preview is live by requesting the URL:

```bash
curl -s -o /dev/null -w "%{http_code}" https://<branch-slug>.<project>.pages.dev
```

- **200**: Build succeeded, preview is live.
- **404** or connection error: Still building — retry in 30 seconds.

Typical build time: 1–3 minutes for a static site.

You can also check the [Cloudflare dashboard](https://dash.cloudflare.com) → Pages → project → Deployments.

## After Admin Merges to Main

Once an admin merges the PR on GitHub:

1. Cloudflare auto-deploys production from `main`.
2. The live site updates within 1–3 minutes.
3. Verify by visiting the production URL.
