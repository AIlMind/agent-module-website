# Cloudflare Pages Deployment

## How It Works

Cloudflare Pages is connected to this GitHub repo and builds automatically:

| Push target | Result |
|-------------|--------|
| `main` | **Production** deploy at the live URL |
| Any other branch | **Preview** deploy at a branch-specific URL |

No manual trigger is needed — pushing a branch is enough.

## Preview URL Pattern

Preview URLs follow:

```
https://<branch-slug>.<project>.pages.dev
```

Slashes become hyphens, so `release/update-hero-2025-04-06` becomes:

```
https://release-update-hero-2025-04-06.ailmind.pages.dev
```

## Polling for Build Status

After pushing a release branch, use the credential wrapper + polling script:

```bash
bash scripts/with-credentials.sh bash scripts/cf-poll-deploy.sh <branch-name>
```

Credentials are loaded automatically from the OS keychain. Never pass tokens
as arguments or set them in terminal commands.

The script checks the Cloudflare API every 30 seconds and exits when the build
succeeds or fails. It times out after 20 minutes.

In CI (GitHub Actions), credentials come from repository secrets instead.

### Manual Check (if the script is unavailable)

```bash
curl -s -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/pages/projects/$CLOUDFLARE_PROJECT_NAME/deployments" \
  | jq '[.result[] | select(.deployment_trigger.metadata.branch == "BRANCH")] | .[0] | {status: .latest_stage.status, url: .url}'
```

Replace `BRANCH` with the actual branch name.

## After Human Approval

Once the user approves:

1. Merge the PR to `main`.
2. Cloudflare auto-deploys production from `main`.
3. Poll again for `main` to confirm the production build succeeded.
4. Confirm to the user that the live site is updated with the production URL.
