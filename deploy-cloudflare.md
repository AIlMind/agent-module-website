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

The preview URL is **branch-scoped**, not commit-scoped. If you rebase or push
new commits to the same release branch, the same preview URL will rebuild with
the new content. Treat that as a new preview that must be re-tested.

You can also check the [Cloudflare dashboard](https://dash.cloudflare.com) → Pages → project → Deployments.

## Matching a Preview to a PR

When an admin asks to publish a tested preview:

1. Derive the release branch from the preview URL slug.
2. Find the open PR for that exact branch.
3. Confirm the PR summary matches the tested change before merging.

Example:

```text
Preview URL: https://release-update-hero-2025-04-06.ailmind.pages.dev
Branch:      release/update-hero-2025-04-06
```

If more than one PR could match the admin's description, ask which one they mean.

## Rebase and Re-test Rule

If `main` moved after the preview was tested, rebase the release branch onto the
latest `main`, rerun validation, and wait for the preview URL to return 200 again.

Do not merge a stale preview. If the branch changed, the admin must re-test the
updated preview before it goes to production.

## After Admin Merges to Main

Once an admin merges the PR on GitHub:

1. Cloudflare auto-deploys production from `main`.
2. The live site updates within 1–3 minutes.
3. Verify by visiting the production URL.
