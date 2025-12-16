# Blog CMS

Directus CMS configuration for the blog platform - schema migrations, extensions, and K8s deployment.

## Overview

This repo manages the Directus CMS backend for [blog.thamimagi.dev](https://blog.thamimagi.dev). Changes here trigger schema migrations and extension deployments via GitHub Actions.

## Architecture

See [Directus CMS Development Workflow](https://github.com/thami/kubernetes-spec-kit/blob/main/.spec/architecture/directus-cms-workflow.md) for the full architecture documentation.

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│   Directus CMS      │     │    GitHub Actions    │     │   K3s Cluster       │
│  (192.168.1.204)    │────►│  (Repository Dispatch)│────►│  (Blog Deployment)  │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

## Structure

```
blog-cms/
├── snapshots/              # Directus schema snapshots (YAML)
├── extensions/
│   ├── interfaces/         # Custom UI interfaces
│   ├── hooks/              # Lifecycle hooks
│   └── operations/         # Flow operations
├── k8s/                    # K8s manifests for Directus
└── .github/workflows/      # CI/CD pipelines
```

## Development Workflow

### 1. Schema Changes

1. Modify schema in Directus admin UI at http://192.168.1.204:8055
2. Export schema snapshot:
   ```bash
   npx directus schema snapshot --yes ./snapshots/schema.yaml
   ```
3. Commit and push to this repo
4. GitHub Actions applies schema to production

### 2. Extension Development

1. Create extension in `extensions/` folder
2. Test locally by mounting into Directus container
3. Commit and push
4. K8s deployment updated with new extension

### 3. Content Triggers

Content changes in Directus trigger the blog rebuild via:
1. Directus Flow with [GitHub Operation](https://github.com/directus-labs/extensions/tree/main/packages/github-operation)
2. Repository dispatch to `thami/blog`
3. Astro site rebuild and deploy

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

- URL: http://192.168.1.204:8055
- Credentials: Stored in Vault at `secret/directus`
