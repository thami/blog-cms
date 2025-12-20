# Blog CMS Architecture Overview

## System Context

```
┌─────────────────────────────────────────────────────────────────┐
│                    Content Management Flow                       │
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Editor  │───>│ Directus │───>│  GitHub  │───>│   Blog   │  │
│  │  (User)  │    │   CMS    │    │ Actions  │    │ (Astro)  │  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│                        │                                        │
│                        ▼                                        │
│                  ┌──────────┐                                   │
│                  │ Schema   │                                   │
│                  │ Snapshot │                                   │
│                  └──────────┘                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Repository Architecture

```
blog-cms/
├── snapshots/           # Schema version control
│   └── schema.yaml      # Current schema state
├── extensions/          # Production extensions (built)
│   ├── hooks/           # Lifecycle hooks
│   └── operations/      # Flow operations
├── src-extensions/      # Extension source code
│   ├── hooks/           # Hook source (dev)
│   └── operations/      # Operation source (dev)
└── .github/workflows/   # CI/CD
    ├── schema-apply.yaml
    └── extension-deploy.yaml
```

## Extension Types

| Type | Purpose | Location |
|------|---------|----------|
| **Hooks** | Lifecycle events (create, update, delete) | `extensions/hooks/` |
| **Operations** | Flow actions (GitHub dispatch, etc.) | `extensions/operations/` |
| **Interfaces** | Custom UI components | `extensions/interfaces/` |
| **Modules** | Custom admin pages | `extensions/modules/` |

## Content Trigger Flow

```
1. Editor publishes content in Directus
   └─> Directus Flow triggered on item.update

2. Flow executes GitHub Operation
   └─> POST /repos/thami/blog/dispatches
       Body: { event_type: "content-update" }

3. GitHub Actions workflow triggered
   └─> Rebuilds Astro static site

4. ArgoCD detects change
   └─> Deploys updated blog to K3s
```

## Access Architecture

### Three-Tier Access

```
┌─────────────────────────────────────────────────────────────────┐
│                        PUBLIC TIER                              │
│  api.thamimagi.dev (Cloudflare CDN)                            │
│  └─> Read-only API, cached responses                           │
├─────────────────────────────────────────────────────────────────┤
│                        ADMIN TIER                               │
│  cms.thamimagi.dev (Pangolin Auth)                             │
│  └─> Full admin UI, authenticated users                        │
├─────────────────────────────────────────────────────────────────┤
│                       INTERNAL TIER                             │
│  directus.directus.svc.cluster.local:8055                      │
│  └─> Cluster-only, service-to-service                          │
└─────────────────────────────────────────────────────────────────┘
```

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| CMS | Directus | Self-hosted, flexible API, Vue-based |
| Schema Format | YAML | Human-readable, git-friendly |
| Extensions | Built dist | Minimal repo size, fast CI |
| Triggers | Flows + Operations | Native Directus, no external deps |
