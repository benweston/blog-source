# Publish to Hashnode from GitHub

Write blog posts as markdown files in this repository. Push to main, and they publish to your Hashnode publication through the [Publish to Hashnode GitHub Action](https://github.com/Hashnode/publish-github-action).

Publishing via the API requires a publication on [Hashnode Pro](https://hashnode.com/pro).

## Setup

1. Click **Use this template** to create your own copy of this repository.
2. Generate a Personal Access Token in [Settings → Developer](https://hashnode.com/settings/developer) on Hashnode.
3. In your new repository, add the token as a secret named `HASHNODE_PAT` (Settings → Secrets and variables → Actions).
4. Open `.github/workflows/publish-to-hashnode.yml` and replace `YOUR_BLOG.hashnode.dev` with your publication host.
5. Edit `posts/example-post.md` or add a new `.md` file under `posts/`, commit, and push.

That's it. Each push publishes new files and updates changed ones. The workflow's run summary shows what happened to every file.

## Writing posts

Every post is a markdown file with YAML frontmatter. Only `title` is required. Set a stable `slug`: it is how a file stays linked to its published post across pushes.

A post using every supported field:

```yaml
---
title: How I built my blog
slug: how-i-built-my-blog
subtitle: A tour of the stack
tags: javascript, nextjs, web-dev
cover: ./images/cover.png
seriesSlug: building-in-public
canonical: https://example.com/original
seoTitle: How I built my blog with Next.js
seoDescription: A walkthrough of the stack behind my blog.
ogImage: https://example.com/og.png
enableToc: true
disableComments: false
publishedAt: 2026-07-01T09:00:00Z
saveAsDraft: false
ignorePost: false
hideFromCommunity: false
publishAs: someusername
coAuthors: user1, user2
domain: other-blog.hashnode.dev
---
```

| Field                                   | Notes                                                                               |
| --------------------------------------- | ----------------------------------------------------------------------------------- |
| `title`                                 | Required                                                                            |
| `slug`                                  | Defaults to a slug of the title. Keep it stable. It is the update key               |
| `subtitle`                              | Shown under the title                                                               |
| `tags`                                  | Comma-separated or YAML list of tag slugs, max 15. Unknown tags are created         |
| `cover`                                 | Cover image URL, or a repo path that gets uploaded to the CDN (alias: `coverImage`) |
| `seriesSlug`                            | Slug of an existing series in your publication                                      |
| `canonical`                             | Canonical URL if the article was first published elsewhere (alias: `canonicalUrl`)  |
| `seoTitle`, `seoDescription`, `ogImage` | SEO and Open Graph overrides                                                        |
| `enableToc`                             | Show a table of contents                                                            |
| `disableComments`                       | Disable comments on the post                                                        |
| `publishedAt`                           | ISO 8601 date, backdates the post                                                   |
| `saveAsDraft`                           | Create a draft instead of publishing (new posts only)                               |
| `ignorePost`                            | Skip this file entirely                                                             |
| `hideFromCommunity`                     | Delist the post from feeds (alias: `hideFromHashnodeCommunity`)                     |
| `publishAs`                             | Username of a publication member to publish as (team publications)                  |
| `coAuthors`                             | Usernames of publication members, max 4                                             |
| `domain`                                | Optional per-file publication override                                              |

## Good to know

- **Create vs update.** The action looks up your `slug` in the publication. Found, the post is updated in place; not found, a new post is published. If your slug is already taken by another post, the API assigns a suffixed slug and the run summary warns you.
- **Images.** Relative image paths (body and `cover`) upload to the Hashnode CDN automatically and resolve relative to the markdown file; a leading `/` resolves from the repository root. Supported: jpg, png, gif, webp, avif, up to 8 MB each. SVG is not supported. Absolute URLs pass through untouched.
- **Drafts.** `saveAsDraft: true` creates a draft for new posts. Re-pushing the same file while it is still a draft creates another draft, so publish or remove the flag once done.
- **Deletes.** Deleting a markdown file never deletes the post. Manage published posts from your dashboard.
- Up to 10 changed files are processed per push by default (the `max-files` input raises it).
