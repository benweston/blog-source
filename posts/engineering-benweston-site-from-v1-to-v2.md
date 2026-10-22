---
title: "Engineering benweston.site: From Zero-Runtime Foundations (v1.0.0) to a Multi-Page Platform (v2.0.0)"
subtitle: "Building a zero-compute edge portfolio with Astro 5, Google Cloud, Trunk-Based CI/CD, and a decoupled Git-driven blog pipeline"
slug: engineering-benweston-site-from-v1-to-v2
tags: google-cloud, cicd, github-actions, web-development, platform-engineering
cover: /assets/banners/engineering-benweston-site-from-v1-to-v2.png
seoTitle: "Engineering benweston.site: From v1.0.0 to v2.0.0"
seoDescription: "A deep dive into the engineering journey behind benweston.site, covering Astro 5 static generation, Google Cloud Edge delivery, Trunk-Based CI/CD, and a decoupled Git-to-Hashnode blog pipeline."
enableToc: true
saveAsDraft: true
ignorePost: false
---

Shipping a personal engineering portfolio sounds deceptively simple on paper. In practice, treating a personal platform with the same architectural rigour as production infrastructure—complete with Trunk-Based CI/CD, isolated staging and production environments, least-privilege IAM, automated documentation synchronisation, and a decoupled publishing pipeline—turns a weekend static site into a genuine platform engineering exercise.

This article walks through the engineering journey behind [benweston.site](https://benweston.site/) ([`benweston/app-portfolio`](https://github.com/benweston/app-portfolio)): how we established its zero-runtime edge foundations for `v1.0.0`, and how we evolved it into a full multi-page platform integrated with this technical writing hub (`blog.benweston.site`) in `v2.0.0`.

---

## Designing the Foundation: The Path to `v1.0.0`

From the outset, the architectural goal for `benweston.site` was **zero operational drag**:

- **Zero Server Runtime:** No backend daemons, containers, or virtual machines idling or suffering from cold starts.
- **Zero Compute Cost:** Pre-rendered static assets served directly from Google Cloud's global Edge CDN cache.
- **Zero Host Maintenance:** No server-side runtime patching or container vulnerability surface.
- **Sub-100ms Global Latency:** Content delivered from the nearest Google Cloud Edge Point of Presence (PoP) with automatic TLS provisioning.

### Selecting Astro 5 and Atkinson Hyperlegible

To achieve zero client-side JavaScript overhead by default whilst retaining component modularity, we built the site on [Astro 5](https://astro.build/). Rather than shipping heavy client-side hydration bundles, Astro compiles every route down to semantic, accessible HTML and scoped CSS at build time.

Accessibility and legibility were treated as first-class engineering requirements (`WCAG 2.1 AA`). We standardised on **Atkinson Hyperlegible** typography across both light and dark themes, ensuring unambiguous character differentiation and high-contrast readability across devices.

### Trunk-Based CI/CD & Firebase Multisite on Google Cloud

Even for a static site, deploying straight from a local workstation to production is an anti-pattern. Prior to cutting `v0.2.0` and `v1.0.0`, we architected a multi-environment delivery pipeline using **Google Cloud**, **Firebase Multisite Hosting**, and **GitHub Actions**:

| Environment | Target URL | Trigger | Purpose |
| :--- | :--- | :--- | :--- |
| **CI Gate** | N/A | Pull Request to `main` | Executes `npx astro check` diagnostics and verifies clean production builds (`npm run build`) |
| **Staging** | [`stage.benweston.site`](https://stage.benweston.site/) | Push / merge to `main` | Continuous preview environment for pre-release verification |
| **Live (Production)** | [`benweston.site`](https://benweston.site/) | Annotated tag push (`v*`) | Official public release promoted via `deploy-production.yml` |

Authentication between GitHub Actions and Google Cloud is governed by least-privilege service account credentials (`google-github-actions/auth@v2`), ensuring that every commit on `main` is automatically validated on `stage.benweston.site` before an annotated release tag promotes the build to `benweston.site`.

### Operational Documentation as Code

Before declaring `v1.0.0` production-ready, we authored a six-part operational documentation suite in `docs/wiki/` covering User and Operations Guides, Troubleshooting, least-privilege IAM runbooks, CLI validation suites, and architectural FAQs. Coupled with an automated GitHub Action (`sync-wiki.yml`), any documentation update merged to `main` automatically mirrors to the repository's GitHub Wiki, eliminating documentation drift.

---

## Scaling the Platform: The Leap to `v2.0.0`

While `v1.0.0` established a rock-solid single-page landing view and delivery pipeline, `v2.0.0` transformed `benweston.site` into a comprehensive multi-page platform.

### 1. Shared Layout Architecture & Multi-Page Routing

We refactored the site around a unified layout template (`PageLayout.astro`) enforcing `<html lang="en-GB">`, responsive `1280px` container geometry, active route detection (`aria-current="page"`), and consistent page-level call-to-action bars across four core routes:

- **Home (`/`):** Expanded the Core Engineering Focus showcase into a responsive `4x1` grid spanning *Enterprise Linux & Systems*, *Virtualisation & Containers*, *Software & DevEx*, and *Cloud & Architecture*.
- **About (`/about`):** Documented the journey from early hardware builds, Sinclair BASIC, and home-lab Linux rack servers to enterprise platform engineering.
- **Experience (`/experience`):** Built interactive vertical career timelines covering engineering roles across **Springer Nature** and **BioMed Central**, alongside academic progress on the MSc in Computer Science at **Keele University** and Credly-verified **LPI Linux Essentials** certification.
- **Projects (`/projects`):** Organised a nine-repository open-source catalogue aligned with the three core engineering pillars (*Linux & Systems*, *Software & DX*, and *Cloud & Architecture*).

### 2. Decoupling the Technical Writing Hub (`blog.benweston.site`)

A pivotal architectural decision in `v2.0.0` was how to host long-form technical writing. Rather than coupling every blog post edit to an application release of `app-portfolio`, we decoupled the blog onto its own dedicated subdomain—[`blog.benweston.site`](https://blog.benweston.site/)—backed by a two-way GitHub pipeline:

```text
[ benweston/blog-source ]  --(GitHub Actions)-->  [ Hashnode Edge (blog.benweston.site) ]
                                                            |
[ benweston/blog-backup ]  <--(Native GitHub Sync)----------+
```

1. **Outbound Git Authoring ([`benweston/blog-source`](https://github.com/benweston/blog-source)):** Articles are authored in Markdown with YAML frontmatter in Git and published automatically to Hashnode via GitHub Actions (`Hashnode/publish-github-action@v1`).
2. **Inbound Backup ([`benweston/blog-backup`](https://github.com/benweston/blog-backup)):** Hashnode's native GitHub integration synchronises published posts back to a dedicated backup repository.
3. **Seamless Edge Redirection (`/blog`):** In `app-portfolio`, both `firebase.json` (across `live` and `stage` targets) and `astro.config.mjs` configure HTTP `301` redirects from `/blog` and `/blog/**` to `https://blog.benweston.site`, paired with an external **Blog ↗** link in the primary navigation header.
4. **Zero-Lock-In Portability:** Because every article remains stored as standard Markdown with YAML frontmatter in Git, migrating to a self-hosted Astro application (`app-blog` on Firebase Hosting) in the future requires zero content rewriting—only repointing the `blog` DNS record.

### 3. Agentic Engineering Governance (`GEMINI.md`)

Throughout both `v1.0.0` and `v2.0.0`, development velocity and repository hygiene were reinforced by codifying project context, British English (`en-GB`) locale rules, standardised 4-section issue templates, and strict deployment guardrails in `GEMINI.md` for agentic pair programming with **Google Antigravity**. Explicitly gating `git commit`, `git push`, `git tag`, and deployment actions behind human sign-off ensured that automated pipelines on `main` and `v*` tags never fired prematurely.

---

## What Comes Next

With `app-portfolio` `v2.0.0` live on [`benweston.site`](https://benweston.site/) and the Git-to-Hashnode publishing pipeline operational here on [`blog.benweston.site`](https://blog.benweston.site/), the platform foundation is complete. Upcoming articles will dive into enterprise Linux administration, workstation automation (`dx-scripts` / `dx-dotfiles`), and cloud infrastructure patterns on Google Cloud and Terraform.
