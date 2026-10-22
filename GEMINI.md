# Blog Source (`blog.benweston.site`) - Agent Instructions & Project Context

## Project Overview
This repository (`benweston/blog-source`) is the canonical outbound authoring source for Ben Weston's technical writing hub at [blog.benweston.site](https://blog.benweston.site/), paired with the main engineering portfolio at [benweston.site](https://benweston.site/) (`benweston/app-portfolio`). Articles are authored as Markdown files with YAML frontmatter and published automatically to Hashnode via GitHub Actions.

---

## Core Engineering Pillars & Editorial Focus

Articles and technical series align with the three-tier engineering workflow defined across Ben's portfolio:
1. **Linux & Systems:** Enterprise Linux operations across Red Hat, SUSE, and Debian families (and their derivatives), covering system administration, automation scripting, security hardening, and performance monitoring.
2. **Software & DX:** Workstation/developer tooling (`dx-*`), automated CI/CD pipelines, robust Bash & shell scripting for workflow automation, and Python for systems and software development.
3. **Cloud & Architecture:** Designing resilient, cost-conscious solutions across Google Cloud, Infrastructure as Code with Terraform, containerised services (Docker), and edge delivery.

> [!NOTE]
> **Stack Scope:** TypeScript is utilised strictly as internal tooling in the companion Astro portfolio (`app-portfolio`). It is **not** part of Ben's highlighted personal engineering stack. Do not feature TypeScript as a core personal skill in editorial bios, profile copy, or pillar summaries.

---

## Architecture & Publishing Pipeline

The technical writing hub is intentionally decoupled from `app-portfolio` on its own dedicated subdomain (`blog.benweston.site`), backed by a two-way GitHub pipeline:

- **Outbound Source (`benweston/blog-source`):** Markdown articles with YAML frontmatter are authored in `posts/` and published or updated automatically on Hashnode via `.github/workflows/publish-to-hashnode.yml` (`Hashnode/publish-github-action@v1`).
- **Inbound Backup (`benweston/blog-backup`):** Hashnode's native GitHub integration synchronises published posts back to a dedicated backup repository.
- **Portfolio Integration (`benweston.site/blog`):** Visitors navigating to `benweston.site/blog` (or clicking **Blog ↗** in the portfolio header) are routed via HTTP `301` edge redirects in `app-portfolio` to `https://blog.benweston.site`.
- **Custom Domain & Edge Routing:**
  - `CNAME` record `blog` → `hashnode.network` managed in Squarespace DNS.
  - `TXT` verification record `_vercel` verifies domain ownership on Hashnode's edge router.
- **Future Transition Path (`app-blog`):** Because all articles are authored as standard Markdown with YAML frontmatter in Git, migrating to a self-hosted Astro application (`app-blog` on Firebase Hosting) requires zero content rewriting — only repointing the `blog` DNS record.

---

## Directory Structure

```text
blog-source/
├── .github/
│   ├── ISSUE_TEMPLATE/     # Standardised GitHub issue templates (if applicable)
│   └── workflows/          # GitHub Actions pipelines (publish-to-hashnode.yml)
├── assets/                 # Repository-level assets (banners, diagrams, screenshots)
│   ├── banners/            # Header and promotional banners (main-banner.png)
│   ├── diagrams/           # Architecture diagrams and schematics
│   ├── icons/              # Logos, badges, and iconography
│   ├── misc/               # Miscellaneous asset files
│   └── screenshots/        # Project, CLI, and UI screenshots
├── docs/                   # Project documentation (if applicable)
│   └── wiki/               # Canonical Markdown source for GitHub Wiki
├── posts/                  # Canonical Markdown blog posts published to Hashnode
│   └── example-post.md     # Template/example post with YAML frontmatter
├── .gitignore              # Git ignore rules for POSIX/Linux, Sublime Text, and secrets
├── GEMINI.md               # Persistent agent guidelines and project context
└── README.md               # Repository documentation and frontmatter reference
```

---

## Authoring & Frontmatter Specification

Every post in `posts/` is a Markdown (`.md`) file with YAML frontmatter:

```yaml
---
title: How I built my blog
slug: how-i-built-my-blog
subtitle: A tour of the stack
tags: linux, bash, cloud
cover: /assets/banners/post-cover.png
seriesSlug: enterprise-linux
canonical: https://example.com/original
seoTitle: How I built my blog
seoDescription: A walkthrough of the stack behind my blog.
ogImage: https://example.com/og.png
enableToc: true
disableComments: false
publishedAt: 2026-10-22T09:00:00Z
saveAsDraft: false
ignorePost: false
hideFromCommunity: false
---
```

### Key Publishing Mechanics
- **Create vs Update (`slug`):** `title` is required, and `slug` serves as the immutable update key linking the Markdown file to its published Hashnode post across pushes. Keep `slug` stable once published.
- **Images & CDN Upload:** Relative image paths (e.g. `./images/diagram.png`) resolve relative to the Markdown file, while a leading `/` (e.g. `/assets/diagrams/arch.png`) resolves from the repository root. Supported formats: `jpg`, `png`, `gif`, `webp`, `avif` (up to 8 MB each; **SVG is not supported** by Hashnode CDN upload). Absolute URLs pass through untouched.
- **Drafts (`saveAsDraft`):** Setting `saveAsDraft: true` creates a draft for new posts. Re-pushing the same file while still marked as a draft creates another draft; remove the flag or set `saveAsDraft: false` when ready to publish.
- **Skipping Posts (`ignorePost`):** Set `ignorePost: true` to instruct the GitHub Action to skip processing a file in `posts/`.
- **Deletions:** Deleting a Markdown file in Git never deletes a published post on Hashnode; published deletions must be managed via the Hashnode dashboard.

---

## CI/CD Pipeline (GitHub Actions)

| Workflow | Trigger | Action | Target URL |
| :--- | :--- | :--- | :--- |
| `publish-to-hashnode.yml` | Push to `main` or manual `workflow_dispatch` | Runs `Hashnode/publish-github-action@v1` against `posts/` | `https://blog.benweston.site` |

- **Authentication:** Uses repository secret `HASHNODE_PAT` (Hashnode Personal Access Token).
- **Publication Host:** Configured via `publication-host` in `.github/workflows/publish-to-hashnode.yml` (`blog.benweston.site`).
- **Manual Wiki Sync Fallback (if `docs/wiki/` is used):**
  ```bash
  git clone git@github.com:benweston/blog-source.wiki.git /tmp/blog-source-wiki
  cp docs/wiki/*.md /tmp/blog-source-wiki/
  cd /tmp/blog-source-wiki
  git add .
  git commit -m "docs: sync wiki with main"
  git push origin master
  rm -rf /tmp/blog-source-wiki
  ```

---

## Coding, Editorial & Agent Guidelines

- **Git, Publishing & Ticket Closure Consent (CRITICAL):** Agents **must never** execute `git commit`, `git push`, `git tag`, or close GitHub issues (including using `closes #<n>` in commit messages) without explicit prior instruction and consent from the user for that specific action. Because pushing to `main` automatically triggers the Hashnode publishing workflow (`publish-to-hashnode.yml`), all work must remain in the local working tree unless the user explicitly asks to commit or push, and tickets remain open until signed off on live (`blog.benweston.site`).
- **Language & Locale:** Standardise on British English (`en-GB`) for all articles, copy, documentation, and frontmatter descriptions (e.g. *optimised*, *containerised*, *virtualisation*, *colour*, *standardised*).
- **Google Cloud Naming:** Use "Google Cloud", never "Google Cloud Platform" or "GCP".
- **Accessible & Semantic Markdown:** Use logical heading hierarchies (`##`, `###`, never skipping levels), descriptive link text, fenced code blocks with explicit language identifiers, and meaningful `alt` attributes for all images.
- **Asset Boundaries:** Do **not** overwrite, delete, or relocate binary files in `assets/` (e.g. `main-banner.png`) without explicit user permission.
- **Privacy & Security:** Never include personal email addresses, internal hostnames/IPs, or sensitive credentials/tokens in posts, documentation, wikis, or source files. Always use generic role titles (e.g. *Project Owner*, *Repository Administrator*) and RFC 5737 / RFC 2606 example domains/addresses where needed.
- **Documentation Preservation:** Maintain documentation integrity. Keep existing comments, frontmatter metadata, licenses, and references intact unless instructed otherwise.

---

## Standardised Issue & Ticket Specification

All issues, user stories, and project board cards MUST strictly follow the exact 4-section structure below. No deviations or ad-hoc formats are permitted. Acceptance criteria represent hard requirements that must be verified prior to promoting any work to Done.

```markdown
### Overview

[Concise 1–2 sentence statement of purpose, context, and outcome]

### Requirements & Scope

1. **[Area 1]:** [Specific deliverable]
2. **[Area 2]:** [Specific deliverable]
3. **[Area 3]:**
   - **[Sub-item A]:** [Details]
   - **[Sub-item B]:** [Details]
4. **[Area 4]:** [Specific deliverable]

### Technical Considerations

- [Editorial, structural, frontmatter, or asset constraints]
- [Publishing workflow or Hashnode CDN considerations]
- [Standards compliance (e.g. en-GB locale, accessibility, Google Cloud naming)]

---

### Acceptance Criteria

- [ ] Markdown frontmatter and formatting validated cleanly.
- [ ] [Verifiable check 2]
- [ ] [Verifiable check 3]
- [ ] Published and verified on `blog.benweston.site`.
```
