# agent-module-website

Shared AI agent rules for static website projects (Astro + Cloudflare Pages).

## Usage

Add as a git submodule in your project:

```bash
git submodule add https://github.com/AIlMind/agent-module-website.git agents/websites
```

Then point your root `AGENTS.md` at `agents/websites/main.md`.

## Files

| File | Purpose |
|------|---------|
| `main.md` | Core agent rules: permissions, delivery flow, testing, handoff |
| `deploy-cloudflare.md` | Cloudflare Pages deployment and polling docs |

## Updating

To pull the latest rules into a project using this as a submodule:

```bash
cd agents/websites && git pull origin main && cd ../.. && git add agents/websites && git commit -m "Update website agent module"
```
