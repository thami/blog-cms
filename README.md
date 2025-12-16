# Blog CMS

Directus CMS configuration for the blog platform - schema migrations, extensions, and K8s deployment.

## Overview

This repo manages the Directus CMS backend for [blog.thamimagi.dev](https://blog.thamimagi.dev). Changes here trigger schema migrations and extension deployments via GitHub Actions.

**This is a configuration-only repository** - unlike typical Directus projects with full Node.js setups, this repo intentionally avoids heavy dependencies. See [Why No package.json?](#why-no-packagejson) below.

## Architecture

See [Directus CMS Development Workflow](https://github.com/thami/kubernetes-spec-kit/blob/main/.spec/architecture/directus-cms-workflow.md) for the full architecture documentation.

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│   Directus CMS      │     │    GitHub Actions    │     │   K3s Cluster       │
│  (K3s Cluster)      │────>│  (Repository Dispatch)│────>│  (Blog Deployment)  │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

## Structure

```
blog-cms/
├── snapshots/              # Directus schema snapshots (YAML)
│   └── schema.yaml
├── src-extensions/         # Extension source code (when needed)
│   ├── hooks/              # Lifecycle hooks source
│   └── operations/         # Flow operations source
├── extensions/             # Built extensions (dist only)
│   ├── hooks/
│   └── operations/
├── k8s/                    # K8s manifests for Directus
├── scripts/                # Build scripts
└── .github/workflows/      # CI/CD pipelines
```

## Why No package.json?

Traditional Directus projects include a full Node.js setup:

```
typical-directus-project/
├── package.json            # Directus as dependency
├── package-lock.json       # 800MB+ dependencies
├── node_modules/           # Heavy!
├── .env                    # Local config
└── extensions/
```

**This repo takes a different approach:**

| Aspect | Typical Directus | This Repo |
|--------|-----------------|-----------|
| Repository size | ~800MB+ with node_modules | ~10MB |
| Local Directus | Yes, runs locally | No, uses deployed K3s instance |
| Dependencies | Full Directus + plugins | None at root level |
| Extension dev | Integrated with local instance | Per-extension package.json |
| CI/CD | Needs full Directus setup | Deploys artifacts directly |

**Benefits:**
- Fast clone and CI/CD (no `npm install` at root)
- Schema and config version-controlled without bloat
- Extensions developed independently when needed
- Works directly with production-deployed Directus

**Trade-offs:**
- No local Directus preview (test against deployed instance)
- Extension development requires more manual steps

See [ADR-017: Directus Extension Development](https://github.com/thami/kubernetes-spec-kit/blob/main/.spec/decisions/017-directus-extension-development.md) for full architectural decision.

## Development Workflow

### 1. Schema Changes

1. Modify schema in Directus admin UI at `cms.thamimagi.dev` (via Pangolin)
2. Export schema snapshot:
   ```bash
   # From Directus container or with directus CLI
   npx directus schema snapshot --yes ./snapshots/schema.yaml
   ```
3. Commit and push to this repo
4. GitHub Actions applies schema to production

### 2. Extension Development

Extensions are developed in `src-extensions/` with their own dependencies:

```bash
# Create a new hook extension
mkdir -p src-extensions/hooks/my-hook
cd src-extensions/hooks/my-hook
npx create-directus-extension@latest

# Develop and build
npm install
npm run build

# Copy to extensions/ for deployment
mkdir -p ../../../extensions/hooks/my-hook
cp -r dist package.json ../../../extensions/hooks/my-hook/
```

Each extension has its own `package.json` - no root-level Node.js dependencies needed.

### 3. Content Triggers

Content changes in Directus trigger the blog rebuild via:
1. Directus Flow with [GitHub Operation](https://github.com/directus-labs/extensions/tree/main/packages/github-operation)
2. Repository dispatch to `thami/blog`
3. Astro site rebuild and deploy

## Access Architecture

| Tier | Endpoint | Access |
|------|----------|--------|
| **Public API** | `api.thamimagi.dev` | Read-only, Cloudflare CDN |
| **Admin UI** | `cms.thamimagi.dev` | Authenticated via Pangolin |
| **Internal** | `directus.directus.svc.cluster.local:8055` | Cluster only |

See [ADR-016: Hybrid Access Architecture](https://github.com/thami/kubernetes-spec-kit/blob/main/.spec/decisions/016-hybrid-access-architecture.md) for details.

## MCP Integration

Use [Directus MCP Server](https://directus.io/tv/directus-mcp-server) with Claude Code for AI-assisted:
- Schema design via natural language
- Content management
- Automated translations
- Media analysis

## Related Repos

- [thami/blog](https://github.com/thami/blog) - Astro frontend
- [thami/kubernetes-spec-kit](https://github.com/thami/kubernetes-spec-kit) - Infrastructure & K8s manifests

## Directus Access

- **Public API**: https://api.thamimagi.dev (read-only)
- **Admin UI**: https://cms.thamimagi.dev (via Pangolin)
- **Direct**: http://192.168.1.108:8055 (LAN only)
- **Credentials**: Stored in Vault at `secret/directus`
