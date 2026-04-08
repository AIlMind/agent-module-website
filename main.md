# Website Agent Rules

This file is the source of truth for AI coding agents working on this website.
All root agent files (`AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`) redirect here.

---

## Pre-flight

Before starting any work:

1. If `agents/` is empty or missing, run:
   ```bash
   git submodule update --init --recursive 2>/dev/null || true
   ```
   If that does not populate it, the directory is committed directly and should already be present.

2. Run `npm ci` if `node_modules/` is missing.

---

## Privilege Level — Standard Mode

You are in **standard mode**. You may:

- Edit any file under `src/`, `public/`, `tests/`
- Update `package.json` dependencies and scripts
- Modify `astro.config.mjs`, `tailwind.config.cjs`, `tsconfig.json`
- Add or update Playwright tests
- Create release branches and push them
- Open pull requests

You **must not**:

- Merge pull requests to `main` (only an admin merges to production)
- Modify database schemas, data, or connections
- Change authentication, payment, or security configuration
- Alter hosting infrastructure (DNS, domains, Cloudflare dashboard settings)
- Edit files inside `agents/` (centrally managed)
- Edit root agent stubs (`AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`)

### Admin Escalation

If the user requests a restricted operation:

1. Check if the `admin` branch exists locally: `git branch --list admin`
2. If it does not exist, this machine does not have admin access. Tell the user:
   > "This machine isn't set up for admin changes. Contact your administrator."
   Do not attempt to fetch or create the admin branch.
3. If it does exist, explain the change needs admin mode and offer to switch.
4. If there is uncommitted work, offer to commit or stash first.
5. If they agree:
   ```bash
   git stash  # if uncommitted changes
   git checkout admin && git pull --rebase origin main
   ```

Do not attempt restricted operations. Do not work around the restrictions.

---

## Core Rule

Never publish directly to the live website. Always use a **release branch** and pull request.

---

## Start-of-Chat Triage

At the start of every new request:

1. Check for an existing release branch (`release/*`).
2. If one exists, explain in plain language:
   > "I found unpublished changes from an earlier session. Continue from those, or start fresh?"
3. Do not force the user to understand branches, PRs, or merge strategy.
4. If the user wants a fresh start, begin from the current `main` branch.

---

## Editor Delivery Flow

Follow this exact sequence:

1. Start from `main` (`git checkout main && git pull origin main`).
2. Create or resume a release branch: `release/<short-description>-<date>`.
3. Make the requested changes.
4. Before handoff, sync the release branch with the latest `main` so the preview matches what production would receive:
   ```bash
   git fetch origin
   git rebase origin/main
   ```
5. If the rebase has conflicts, resolve them now. If you cannot resolve them safely, stop and explain the branch is no longer aligned with production.
6. Run local validation **after the rebase**:
   ```bash
   npm run check        # Astro type checks
   npm run build        # Full build
   npm run test:e2e     # Playwright browser tests
   ```
7. If any step fails, fix the problem before continuing.
8. Push the release branch to origin. If you rebased an existing remote branch, use `git push --force-with-lease origin <branch-name>`.
9. Open or update the pull request from the release branch to `main`.
10. Construct the Cloudflare preview URL (see [`deploy-cloudflare.md`](deploy-cloudflare.md)):
   ```
   https://<branch-slug>.<project>.pages.dev
   ```
   Replace slashes with hyphens. Example: `release/update-hero` → `release-update-hero`.
11. Wait for the preview to go live — poll with:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" <preview-url>
   ```
   Retry every 30 seconds until it returns 200 (typically 1–3 minutes).
12. Give the human the preview details:
   - Preview URL
   - Release branch name
   - PR number or URL
   - Latest commit SHA
   - Plain-language summary of what changed
13. Tell the user this exact PR is the one an admin must merge to publish.
14. If `main` changes before merge, rebase the release branch again, rerun validation, and wait for the preview to return 200 again before calling it ready.

**You do not merge to `main`.** Only an admin does that via GitHub.

---

## Admin Publish Flow

Use this flow only when an admin explicitly asks you to publish a tested change.

1. Identify the target PR from the strongest evidence available, in this order:
   - PR URL or PR number
   - Preview URL
   - Exact release branch name
   - A human description of the change
2. If given a preview URL, derive the branch from the branch slug and find the open PR for that branch.
   Example: `https://release-update-hero-2025-04-06.<project>.pages.dev` maps to `release/update-hero-2025-04-06`.
3. Confirm the PR matches what the admin means by comparing the preview URL, branch name, PR title, and change summary.
4. If there is any ambiguity, stop and ask which PR to publish. Do not guess.
5. Fetch the latest remote state and check whether the PR branch is still cleanly mergeable into `main`.
6. If the branch is behind `main` or has merge conflicts, rebase it onto the latest `main`.
7. After any rebase or conflict resolution:
   - rerun validation on the rebased branch
   - push the updated branch
   - wait for the Cloudflare preview on the same branch URL to return 200 again
8. Tell the admin the preview has changed and must be re-tested before merge because `main` moved since the earlier test.
9. Merge only the exact PR that matches the tested preview.
10. After merging, verify production by checking the live URL.

If conflicts cannot be resolved safely, stop and explain that the tested preview is stale and needs a refreshed branch before production merge.

---

## Human Approval Rule

Accept any clear affirmative ("approve", "looks good", "yes", "go ahead",
"ship it", etc.) as approval for your work. If the response is ambiguous, ask again.

In standard mode, the admin decides when to merge to production.
In admin mode, you may merge only after you have identified the exact PR and confirmed it is the tested change.

---

## Handoff Message

Scale the detail to the risk of the change.

### Cosmetic / text-only changes

Keep it short:

- What changed
- Preview URL
- Release branch name
- PR number or URL
- "An admin can merge this exact PR to publish."

### Changes that touch layout, logic, or integrations

Include:

- What changed (plain language)
- Preview URL
- Release branch name
- PR number or URL
- Latest commit SHA
- What to check on desktop and mobile
- Whether forms, navigation, auth, or payments were affected
- "An admin can merge this exact PR to publish."

Example:

> **Changes:** Updated the hero headline and swapped the background image.
>
> **Preview:** https://release-update-hero-2025-04-06.ailmind.pages.dev
> **Branch:** release/update-hero-2025-04-06
> **PR:** #123
> **Commit:** abc1234
>
> **Desktop check:** Homepage hero section — text and image should look correct.
> **Mobile check:** Same section — image should not overflow on small screens.
> **Forms/auth/payments affected:** No.
>
> An admin can merge this exact PR to publish this to the live site.

---

## Testing Rules

- If a change affects user-facing behaviour, update or add the relevant Playwright test.
- Do not weaken, bypass, or fake tests to get a green result.
- If a flow cannot be tested cleanly, skip it explicitly and explain why.
- Call out exact manual testing still required.
- Treat auth, payments, data deletion, admin permissions, and anything customer-visible as **high risk**.

---

## Branch and Release Naming

- Release branch: `release/<short-description>-<date>`

---

## Rollback

Roll back through Git history (`git revert`) — never by editing production files directly.
Cloudflare also supports deployment rollback from its dashboard.

---

## Tech Stack

| Layer | Tool | Tier |
|-------|------|------|
| Framework | Astro 5 + MDX + TailwindCSS | — |
| Hosting | Cloudflare Pages | Free |
| Source control | GitHub | Free |
| Testing | Playwright (Chromium) | — |
| Package manager | npm | — |

Cloudflare Pages builds and deploys automatically on every push.
No CI pipeline or API tokens are needed.

---

## Files You Must Not Modify

| File/Dir | Reason |
|----------|--------|
| `AGENTS.md` | Managed stub — do not edit |
| `CLAUDE.md` | Managed stub — do not edit |
| `.github/copilot-instructions.md` | Managed stub — do not edit |
| `agents/**` | Centrally managed agent configuration |
