---
layout: post
title: Backup Zammad Helpdesk's Knowledge Base To a Static Site
description: Take your current Zammad knowledge base and export to a static site.
date: 2026-02-21 11:33:00 -0700
categories: [DevOps, Support]
tags: [support, zammad, ticketing, n8n, helpdesk, selfhosted, knowledgebase, ssg, docker, export, guide, markdown, api]
pin: false
image:
  path: /assets/img/header/header--zammad--knowledge-base-export-to-markdown-static-site.jpg
  alt: Zammad Ticketing and Helpdesk has a Knowledge Base in PostgreSQL - Export this to a Mkdocs Static Site
---
# Backup Zammad Knowledge Base and Export to Static Site Generator like Docsify or Mkdocs

## Introduction

I like the idea of [Zammad having a KB](https://admin-docs.zammad.org/en/latest/manage/knowledge-base.html).

But I wish I could easily export that information - I dont want to build up my KB and be tied to this one system.

Can I export into some markdown, and use Docusaurus, MkDocs, Docsify, Hugo, Jekyll, Astro, as a KB if we ditch Zammad?


* * *

## Objective

A self-contained Docker tool that exports a Zammad Knowledge Base in its entirety to a directory tree of Markdown files that can then be served as HTML files.

Please use the [Zammad-Knowledge-Base-Export-to-Static-Site](https://github.com/MarcusHoltz/Zammad-Knowledge-Base-Export-to-Static-Site/) repo that goes along with this article.

It gives you a clean, portable copy of everything in markdown.

> If you - instead - need Zammad's knowledge base exported to Excel: 
> > [Sirhexalot](https://n8n.io/creators/sirhexalot) has created an [n8n workflow to Export Zammad objects (users, roles, groups, organizations) to Excel](https://n8n.io/workflows/2596-export-zammad-objects-users-roles-groups-organizations-to-excel) also available on [GitHub](https://github.com/Sirhexalot/n8n-Export-Zammad-Objects-Users-Roles-Groups-and-Organizations-to-Excel)


* * *

## How it works

Configuration is entirely in [docker-compose.yml](https://github.com/MarcusHoltz/Zammad-Knowledge-Base-Export-to-Static-Site/blob/main/docker-compose.yml)

- you set your Zammad URL

- API token

- run docker: `docker compose run --rm zammad-kb-export`

- exports appears in `./output/` 

> The exporter communicates exclusively with the Zammad REST API. It does not require database access.


* * *

## What is exported

**Every article, regardless of state.** Drafts, internally published articles, externally published articles, and archived articles are all included. Nothing is filtered. The `status` field in each file's frontmatter identifies the state so you can handle filtering in your static site layer if needed.

**The complete category tree.** Categories and subcategories are reflected as folders. Each folder contains an `_index.md` landing page carrying the category title and its Zammad ID.

**All metadata per article**, written as YAML frontmatter:

- `title` — the article title
- `slug` — URL-safe version of the title
- `zammad_id` — the numeric ID of the answer in Zammad, useful for cross-referencing
- `status` — one of `draft`, `internal`, `published`, or `archived`
- `category` — slash-delimited path of the parent category hierarchy
- `promoted` — whether the article is marked as promoted in Zammad
- `published_at` — timestamp when the article was made externally visible
- `internal_at` — timestamp when the article was made internally visible
- `archived_at` — timestamp when the article was archived
- `updated_at` — timestamp of the last modification

**Article body**, converted from Zammad's HTML to clean Markdown using `markdownify`. Headings, lists, inline code, tables, and links are all preserved faithfully.

**Embedded images.** Images referenced in article bodies are downloaded authenticated from the Zammad attachments API and saved to an `images/` directory at the root of the output. Each image filename is derived from the slug of the article it belongs to, with a numeric suffix for articles containing multiple images (e.g. `how-to-board-a-vessel-1.png`).


* * *

## Setup

1. Edit `docker-compose.yml` — fill in your URL and token:

- Get a token at: **Your Profile → Token Access → Create**

- The token needs **Knowledge Base** read permission.

2. **Run it:**

   ```bash
   docker compose run --rm zammad-kb-export
   ```


* * *

## Configuration

All options are set in `docker-compose.yml` under `environment`:

| Variable           | Required | Default | Description                                          |
|--------------------|----------|---------|------------------------------------------------------|
| `ZAMMAD_URL`       | Yes      | —       | Base URL of your Zammad instance                     |
| `ZAMMAD_TOKEN`     | Yes      | —       | API token with Knowledge Base read permission        |
| `ZAMMAD_KB_ID`     | No       | `1`     | ID of the Knowledge Base to export                   |
| `RATE_LIMIT`       | No       | `0.1`   | Seconds to pause between API calls                   |
| `FRONTMATTER`      | No       | `true`  | Whether to include YAML frontmatter in output files  |
| `MD_HEADING_STYLE` | No       | `ATX`   | `ATX` for `# Heading`, `UNDERLINED` for `===` style |

To create an API token, go to your Zammad profile, open Token Access, and create a token with the Knowledge Base reader or editor permission.

`ZAMMAD_KB_ID` is almost always `1`. Zammad assigns IDs sequentially and does not reuse them, so if a Knowledge Base was deleted and recreated the ID will have incremented. You can confirm the ID in the Zammad admin panel under Knowledge Base settings.


* * *

### Token permissions

If you are having difficulty generating your token or with your tokens permissions:

Be sure to go to **Admin → Accounts → Token Access**, find your token (or create a new one), and ensure it has **both** of the following permissions:

| Permission              | What it unlocks                                             |
|-------------------------|-------------------------------------------------------------|
| `knowledge_base.reader` | Categories, answers, and embedded images                    |
| `admin.tag`             | Tags on answers — without this, tags will be silently empty |


- `knowledge_base.reader` permission is the minimum needed to export articles. 

- `admin.tag` is required separately because Zammad's tags endpoint is a different API surface from the Knowledge Base. 
  - tokens without `admin.tag` will return an empty tag list with no error

- `ZAMMAD_KB_ID` is almost always `1`. 
  - Zammad assigns IDs sequentially and does not reuse them, so if a Knowledge Base was deleted and recreated the ID will have incremented.


* * *

## Output structure

Here is an example Zammad KB output for a 1680's logicstics company:

```
output/
|
+-- _data/
|   +-- users.yml
|   +-- organizations.yml
|   +-- roles.yml
|   +-- groups.yml
|
+-- images/
|   +-- cannon-loading-procedures-1.png
|   +-- cannon-loading-procedures-2.png
|   +-- navigating-by-the-stars-1.png
|   +-- wanted-poster-template-1.png
|   +-- ship-maintenance-checklist-1.png
|
+-- fleet-operations/
|   +-- _index.md
|   +-- navigation/
|   |   +-- _index.md
|   |   +-- navigating-by-the-stars.md
|   |   +-- reading-ocean-currents.md
|   |   +-- dead-reckoning-in-fog.md
|   |   +-- using-a-sextant.md
|   |
|   +-- gunnery/
|   |   +-- _index.md
|   |   +-- cannon-loading-procedures.md
|   |   +-- powder-magazine-safety.md
|   |   +-- misfires-and-remediation.md
|   |
|   +-- ship-maintenance/
|       +-- _index.md
|       +-- caulking-the-hull.md
|       +-- rigging-inspection-schedule.md
|       +-- ship-maintenance-checklist.md
|
+-- crew-management/
|   +-- _index.md
|   +-- articles-of-agreement.md
|   +-- share-distribution-policy.md
|   +-- disciplinary-procedures.md
|   +-- recruiting-at-port.md
|
+-- plunder-and-prizes/
    +-- _index.md
    +-- acquisition/
    |   +-- _index.md
    |   +-- boarding-tactics.md
    |   +-- negotiating-surrender.md
    |   +-- wanted-poster-template.md
    |
    +-- valuation/
        +-- _index.md
        +-- assessing-cargo-value.md
        +-- known-fence-locations.md
```


* * *

## Using the output with a static site generator

That's right, the point of this was to demonstrate exporting Zammad's KB into a new KB. I have included two static site generators for markdown documentation.

My recommendation is to use `MkDocs`.

> Two ready-made compose files are included. Run either one after exporting your KB to markdown — they both read from the same `./output/` directory. 


* * *

### MkDocs Material — full docs site with dark mode and search

MkDocs is probably the easiest to demonstrate. MkDocs Material will processes the markdown server-side and render a full, polished documentation site. It includes full-text search, dark/light mode toggle, and syntax highlighting on code blocks.

```
docker compose -f docker-compose.mkdocs.yml up
```

Open [http://your_ip_here:8000](http://your_ip_here:8000).

> MkDocs watches the `./output/` directory for changes and live-reloads the browser automatically when the exporter runs.


* * *

#### A note on `_index.md` in MkDocs

The exporter names category landing pages `_index.md` following Hugo's convention. MkDocs uses `index.md` for section index pages and does not treat `_index.md` specially — those files appear as regular pages named "_index" in the MkDocs navigation tree rather than as section titles.

All content is fully accessible; only the visual grouping differs. For a properly structured MkDocs production site, rename `_index.md` → `index.md` throughout the output after exporting.


* * *

### Docsify — instant, no build step

An alternative to MkDocs that I have included is Docsify. It is a single-page app that fetches and renders markdown files client-side. There is no build step and no compilation — nginx serves the files and Docsify handles the rest in the browser.

```
docker compose -f docker-compose.docsify.yml up
```

Open [http://your_ip_here:3000](http://your_ip_here:3000).

> With the new site up, you can use the search bar to find articles, or navigate directly by URL: `http://<your_ip_here>:3000/#/category/article-slug`


* * *

### Why not Hugo, Jekyll, Docusaurus, or Astro?

These are excellent tools for building a production site from the export, but
they require additional setup before they can serve anything:

- **Hugo** needs a theme (pulled from git) and a `hugo.toml` config.

- **Jekyll** needs a `Gemfile`, a theme gem, and a `_config.yml`.

- **Docusaurus** needs a full Node.js project scaffold and a webpack build.

- **Astro** needs a project scaffold, a content collection schema, and a build.

> The SSG integration section below covers how to wire the export into each of these for a production site.


* * *

## Using the output with a static site generator

**Astro** — place the KB output inside `src/content/` and define a content collection. The frontmatter schema maps directly to Astro's `defineCollection` type system. Place the `_data/` files in `src/data/` and import them as JSON or YAML. Filter by `status` in page templates to control public visibility.

**Jekyll** — the KB output drops into `_docs/` or any collection directory without modification. The `_data/` files are consumed automatically by Jekyll's data system. Use `where` filters in Liquid templates to gate content by status.

**Hugo** — the directory structure and `_index.md` convention match Hugo's content organization model exactly. Copy `_data/` files to Hugo's `data/` directory. Use `where .Pages "Params.status" "published"` in templates.


* * *

### Organizational data

**Four** YAML data files are written to `_data/`, one per object type.

These are consumed natively by Jekyll (`_data/`), Hugo (`data/`), Astro (`public/data/` or a content collection).


#### _data/users.yml

This includes all user accounts, excluding the internal Zammad system user. 

Each record includes:

- `id`, `login`, `email`, `firstname`, `lastname`, `active`

- `organization_id` and `organization`

- `role_ids` and `roles`

- `group_access` — a list of `{group, access}` objects

- `last_login`, `created_at`, `updated_at`


#### _data/organizations.yml

- organisations with `id`, `name`, `note`,`domain`, `active`, `member_count`, `created_at`, `updated_at`


#### _data/roles.yml

- roles with `id`, `name`, `note`, `active`, `default_at_signup`, `created_at`, `updated_at`


#### _data/groups.yml

- ticket routing groups with `id`, `name`, `note`, `active`, `email`, `follow_up_possible`, `follow_up_assignment`, `shared_drafts`, `created_at`, `updated_at`


* * *

## Conclusion

Zammad is a capable support platform and its Knowledge Base is genuinely useful for building internal and external documentation.

That content, along with the organizational structure that surrounds it, should not be trapped inside any single system.

I hope my [Zammad-Knowledge-Base-Export-to-Static-Site](https://github.com/MarcusHoltz/Zammad-Knowledge-Base-Export-to-Static-Site) repo was able to help you resolve that concern or motivate you to cross-host it!
