# Blog CMS Plan

## Overview

Directus CMS configuration for the blog platform - schema management, extensions, and automated content workflows.

## Goals

1. Version-controlled schema migrations
2. Automated content-to-blog rebuild pipeline
3. Extensible via custom operations/hooks
4. Lightweight repo (no full Node.js at root)

## Current State

- Schema snapshot workflow established
- GitHub Operation extension for blog rebuilds
- K8s deployment configured
- CI/CD pipeline in place

## Roadmap

### Phase 1: Foundation (Complete)
- [x] Directus deployment on K3s
- [x] Schema snapshot workflow
- [x] GitHub Actions CI/CD
- [x] Access architecture (public/admin/internal)

### Phase 2: Content Automation
- [x] GitHub Operation extension
- [x] Blog rebuild on publish
- [ ] Scheduled content publishing
- [ ] Content validation hooks

### Phase 3: Extensions
- [ ] Custom interfaces for blog posts
- [ ] SEO metadata automation
- [ ] Image optimization hook
- [ ] Analytics integration

### Phase 4: Advanced
- [ ] Multi-language content
- [ ] Content versioning
- [ ] Workflow approvals
- [ ] AI-assisted translations (via MCP)
