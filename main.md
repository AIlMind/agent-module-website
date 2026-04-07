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

You **must not**:

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
3. If it does exist, explain the change needs admin mode.
4. If there is uncommitted work, warn them it won't carry over — offer to commit or stash first.
5. Offer to switch:
   > "I can switch to admin mode for you. Want me to go ahead?"
6. If they agree, run:
   ```bash
   git stash  # if uncommitted changes
   git checkout admin && git pull --rebase origin main
   ```
7. If both a standard and admin change were requested, explain they are separate
   concerns. Finish or park the standard change first.

Do not attempt restricted operations. Do not work around the restrictions.

---

## Core Rule

Never publish directly to the live website. Always create or resume a **release work branch** first.

---

## Start-of-Chat Triage

At the start of every new request:

1. Check for an existing release work branch (`release/*`).
2. If one exists, explain it in plain language:
   - "I found unpublished website changes from an earlier session. Would you like to continue from those, or discard them and start fresh from the current live website?"
3. Do not force the user to understand branches, PRs, rebases, or merge strategy.
4. If the user wants a fresh start, begin from the current production state.

---

## Required Delivery Flow

Follow this exact sequence:

1. Start from the current production branch (`main`).
2. Create or resume a release work branch: `release/<short-description>-<date>`.
3. Make the requested change.
4. Run local validation **before pushing**:
   ```bash
   npm run check        # Astro type checks
   npm run build        # Full build
   npm run test:e2e     # Playwright browser tests
   ```
5. If any step fails, fix the problem before continuing.
6. Push the release work branch to origin.
7. Open a pull request from the release branch to `main` (if one does not already exist).
8. Wait for GitHub Actions CI to pass on the PR.
9. Poll Cloudflare for the preview deployment (see [`deploy-cloudflare.md`](deploy-cloudflare.md)):
   ```bash
   bash scripts/with-credentials.sh bash scripts/cf-poll-deploy.sh <branch-name>
   ```
   Poll every 30 seconds. Wait until the preview build finishes.
10. Give the human the **preview URL** and a plain-language summary of what changed.
11. Ask for **explicit approval** before merging.
12. Only after explicit human approval, merge the PR to `main`.
13. Confirm the production deploy succeeded (poll Cloudflare again for `main`).
14. Delete the release work branch.

---

## Human Approval Rule

Never merge to production without an explicit human approval in chat.
Accept any clear affirmative ("approve", "looks good", "yes", "go ahead",
"ship it", etc.). If the response is ambiguous, ask again.

---

## Handoff Message

Scale the detail to the risk of the change.

### Cosmetic / text-only changes

Keep it short:

- What changed
- Preview URL
- "Send **approve** (or any yes) to publish."

### Changes that touch layout, logic, or integrations

Include:

- What changed (plain language, no jargon)
- The preview/test URL
- What to check on desktop
- What to check on mobile
- Whether any forms, navigation, auth, or payments were affected
- "Send **approve** (or any yes) to publish."

Example:

> **Changes:** Updated the hero headline and swapped the background image.
>
> **Preview:** https://release-update-hero-2025-04-06.ailmind.pages.dev
>
> **Desktop check:** Homepage hero section — text and image should look correct.
> **Mobile check:** Same section — image should not overflow on small screens.
> **Forms/auth/payments affected:** No.
>
> Send **"approve"** to publish this to the live site.

---

## Testing Rules

- If a change affects user-facing behaviour, update or add the relevant Playwright test.
- Do not weaken, bypass, or fake tests to get a green result.
- If a flow cannot be tested cleanly, skip it explicitly and explain why.
- Call out exact manual testing still required.
- Treat auth, payments, data deletion, admin permissions, and anything customer-visible as **high risk**.

---

## Branch and Release Naming

- Release work branch: `release/<short-description>-<date-or-ticket>`
- Archived branch (if kept): `archived/<same-name>`

---

## Rollback

Roll back through Git history (`git revert`) or Cloudflare's deployment rollback — never by editing production files directly.

---

## Tech Stack

| Layer | Tool | Tier |
|-------|------|------|
| Framework | Astro 5 + MDX + TailwindCSS | — |
| Hosting | Cloudflare Pages | Free |
| Source control | GitHub (private repo) | Free |
| CI | GitHub Actions | Free |
| Testing | Playwright (Chromium) | — |
| Package manager | npm | — |

---

## Credential Security

- **Never** echo, print, log, or display credential values.
- **Never** pass tokens as command-line arguments (they appear in `ps` and shell history).
- Always use `scripts/with-credentials.sh` to load credentials from the OS keychain.
- If credentials are missing, tell the user to contact their setup administrator.
- In CI (GitHub Actions, Codex), credentials come from platform secrets — not from you.

---

## Files You Must Not Modify

| File/Dir | Reason |
|----------|--------|
| `AGENTS.md` | Managed stub — do not edit |
| `CLAUDE.md` | Managed stub — do not edit |
| `.github/copilot-instructions.md` | Managed stub — do not edit |
| `agents/**` | Centrally managed agent configuration |
| `scripts/with-credentials.sh` | Credential loader — do not edit |
| `.github/workflows/protect-main.yml` | Branch protection guard |
