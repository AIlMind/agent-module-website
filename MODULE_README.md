# Agent Modules

This directory contains all AI agent configuration for this repository.

## Structure

```
agents/
  websites/          # Website agent rules (content, code, deploys)
    main.md          # Core rules — read by all agents
    deploy-cloudflare.md  # Cloudflare Pages deployment workflow
  admin/             # Elevated-privilege rules (admin branch only)
    main.md          # Extra permissions for infra, data, auth changes
```

## How It Works

The root `AGENTS.md`, `CLAUDE.md`, and `.github/copilot-instructions.md` are
**stubs** that point here. They must not be edited directly.

Each subdirectory is a **module** scoped to a domain:

| Module | Scope | Branch |
|--------|-------|--------|
| `websites/` | Static site content, code, styling, tests, deploys | `main` (all branches) |
| `admin/` | Database, auth, payment, infrastructure changes | `admin` branch only |

## Adding a Module

Create a new directory (e.g. `agents/forms/`) with a `main.md` file, then
reference it from the root stubs.

## Submodule Roadmap

This directory is designed to become a **git submodule** so agent rules can be
shared across multiple repositories. When that happens:

- Each module directory may come from a separate repo
- The root stubs will include pre-flight instructions to init the submodule
- CI and devcontainer will run `git submodule update --init --recursive`

For now, the files are committed directly in this repo.
