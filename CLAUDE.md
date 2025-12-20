# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project Overview

Directus CMS configuration repository - schema snapshots, extensions, and K8s deployment for the blog platform.

**Important**: This is a configuration-only repo. No full Node.js setup at root level.

## Related Repositories

| Repository | Path | Purpose |
|------------|------|---------|
| **blog-cms** (this) | `/Users/thami.magi/workspace/blog-cms` | Directus CMS config |
| **blog** | `/Users/thami.magi/workspace/blog` | Astro frontend |
| **design-system** | `/Users/thami.magi/workspace/design-system` | UI components |
| **kubernetes-spec-kit** | `/Users/thami.magi/workspace/kubernetes-spec-kit` | Infrastructure |

## Project Structure

```
blog-cms/
├── snapshots/              # Directus schema snapshots (YAML)
│   └── schema.yaml
├── src-extensions/         # Extension source code
│   ├── hooks/              # Lifecycle hooks source
│   └── operations/         # Flow operations source
├── extensions/             # Built extensions (dist only)
│   ├── hooks/
│   └── operations/
├── k8s/                    # K8s manifests for Directus
└── .github/workflows/      # CI/CD pipelines
```

## Key Commands

### Schema Operations

```bash
# Export schema from running Directus
npx directus schema snapshot --yes ./snapshots/schema.yaml

# Apply schema to Directus
npx directus schema apply ./snapshots/schema.yaml
```

### Extension Development

```bash
# Create new extension
cd src-extensions/hooks
npx create-directus-extension@latest

# Build extension
npm install && npm run build

# Copy to extensions/ for deployment
cp -r dist package.json ../../extensions/hooks/<name>/
```

## Architecture

### Content Flow

```
Directus CMS → Content Change → Flow Triggered → GitHub Dispatch → Blog Rebuilds
```

### Access Tiers

| Tier | Endpoint | Access |
|------|----------|--------|
| Public API | api.thamimagi.dev | Read-only, CDN |
| Admin UI | cms.thamimagi.dev | Auth via Pangolin |
| Internal | directus.directus.svc.cluster.local:8055 | Cluster only |

## Constraints

1. **No root package.json** - Extensions have their own deps
2. **Schema versioned in YAML** - Always commit schema changes
3. **Extensions as dist only** - Commit built code, not source
4. **Test against deployed Directus** - No local instance

## MCP Integration

Use Directus MCP server for:
- Schema design via natural language
- Content management
- Automated translations
- Media analysis

```bash
# Available via Claude Code MCP
mcp__directus__read-collections
mcp__directus__read-items
mcp__directus__create-item
mcp__directus__update-item
```

## Deployment

- **API**: https://api.thamimagi.dev (public, read-only)
- **Admin**: https://cms.thamimagi.dev (via Pangolin)
- **Direct**: http://192.168.1.108:8055 (LAN)
- **Platform**: K3s cluster via ArgoCD

## Credentials

All secrets in HashiCorp Vault at `secret/directus`:
```bash
kubectl exec -n vault vault-0 -- vault kv get secret/directus
```
