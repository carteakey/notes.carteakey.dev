---
title: Digital Garden Publishing Pipeline
publish: true
garden: seedling
kind: how-to
slug: digital-garden-publishing-pipeline
tags:
  - obsidian
  - quartz
  - publishing
  - digital-garden
description: How I export public notes from my Obsidian vault into Quartz without creating a second source of truth.
---

I wanted a digital garden without turning my notes site into a second writing system.

The obvious trap is ending up with the same note in two places:

- one copy in Obsidian where I actually think
- one copy in the website repo where I "prepare" things for publishing

That creates drift fast. Edits land in the wrong place, links diverge, attachments go missing, and suddenly the website becomes a weird fork of the vault.

So the rule is simple:

- The vault is the source of truth.
- The website repo is generated output.
- A note is public only when I say it is.

## Repos

These are the paths the current setup assumes:

```text
Vault
/Users/kchauhan/.superset/worktrees/vault-76/flannel-feet

Quartz site
/Users/kchauhan/repos/carteakey.dev/notes.carteakey.dev
```

The export script lives in:

```text
1. system 📊/scripts ⚡/export-garden.py
```

## Publishing Contract

Public notes are opt-in through frontmatter:

```yaml
---
title: My Note
publish: true
garden: seedling
kind: concept
slug: my-note
description: Short summary for the page and previews.
tags:
  - topic
  - topic-2
---
```

The fields I care about right now:

- `publish: true` means the note is eligible for export.
- `garden` tracks note maturity.
- `slug` gives the note a stable public URL instead of inheriting the vault path.
- `kind` or `bucket` is optional and decides whether the note lands in a shallow public bucket.

Garden states:

- `seedling`: useful, public, still forming
- `evergreen`: stable, refined, worth referencing
- `private`: keep it in the vault only

Current public buckets:

- `notes`: concepts, references, how-tos, snippets
- `projects`: project notes
- `writing`: essays and blog-like longform notes

The exporter maps `kind` to one of those buckets. If `kind` is missing and no `bucket` is set, the note publishes at the site root.

## Architecture

The publishing flow is intentionally one-way:

1. I write in Obsidian.
2. I mark notes with `publish: true`.
3. I run the export script.
4. The script exports public notes to the site root by default, or into shallow public buckets like `notes/`, `projects/`, and `writing/` when requested.
5. The script copies only referenced attachments into `notes.carteakey.dev/content/_attachments/garden`.
6. Quartz builds the site from that exported content.

This means I never author notes directly in the Quartz content folder.

## What The Script Does

The exporter currently does a few opinionated things:

- scans the vault for Markdown notes
- excludes only non-note system areas like `.obsidian`, `.agent`, `.trash`, and `.vscode`
- keeps only notes with `publish: true`, regardless of where they live in the vault
- generates public output paths from `kind`/`bucket` plus `slug`
- defaults to `<slug>.md` at the site root when no public bucket is specified
- preserves note content and frontmatter
- rewrites embedded attachment links to point at the exported attachment folder
- copies only the attachments actually referenced by exported notes
- fails if a published note links to a non-published note
- fails if export paths collide
- removes previously exported files through a manifest so stale public files do not linger

That last rule matters. A public note should not silently depend on a private note.

## Public Shape, Private Shape

The vault structure exists for storage. The public structure exists for navigation.

Those should not be the same thing.

Inside the vault, a note may live somewhere deeply nested because that is convenient for me while working:

```text
2. knowledge 🧠/data-engineering ⚙️/airflow/Apache Airflow.md
```

Publicly, that should become something much simpler:

```text
/apache-airflow
```

That keeps the site clean and reader-friendly:

- tags do the topical grouping
- `garden` shows maturity
- buckets provide only the lightest amount of structure

I do not want readers to see my internal storage scheme.

## Attachment Strategy

Quartz will happily publish non-Markdown files if they exist in the content tree, so attachments need the same level of discipline as notes.

The exporter copies only referenced assets and places them under:

```text
content/_attachments/garden
```

This avoids dumping the entire vault attachment history into the public site.

## Avoiding Duplication

There are three kinds of duplication I want to avoid.

### 1. The same note in the vault and the site repo

This is the easiest rule:

- edit in the vault
- export to the site
- never hand-edit the generated note in Quartz

### 2. The same idea as both a blog post and a garden note

The split I want is:

- blog posts are polished arguments
- garden notes are living notes, references, and unfinished thinking

If both exist, one should be the canonical version and the other should link to it rather than restating everything.

### 3. Duplicate notes inside the vault

The best defense is:

- use `slug` for public notes
- merge overlapping notes aggressively
- keep one canonical note per concept
- turn near-duplicates into links, aliases, or sections instead of separate pages

## Recommended Frontmatter

This is the current public-note contract I want to stick to:

```yaml
---
title: Apache Airflow
publish: true
garden: evergreen
slug: apache-airflow
tags:
  - data-engineering
  - orchestration
description: Distributed workflow orchestration for data pipelines.
---
```

That publishes at:

```text
/apache-airflow
```

If I need to override the bucket manually, I can use:

```yaml
bucket: projects
```

Or use `kind: concept`, `kind: project`, or `kind: essay` for the common shallow buckets.

## Operational Workflow

### Dry run

```bash
cd /Users/kchauhan/.superset/worktrees/vault-76/flannel-feet
python3 "1. system 📊/scripts ⚡/export-garden.py" --dry-run
```

### Export

```bash
cd /Users/kchauhan/.superset/worktrees/vault-76/flannel-feet
python3 "1. system 📊/scripts ⚡/export-garden.py"
```

### Preview Quartz

```bash
cd /Users/kchauhan/repos/carteakey.dev/notes.carteakey.dev
npm run docs
```

## Current Failure Mode

At the moment, some notes already marked `publish: true` still link to non-public notes. The exporter deliberately stops on that instead of quietly publishing broken public pages.

That is not a bug in the exporter. It is a useful pressure mechanism:

- either publish the dependency
- or remove the public link
- or rewrite the note so it stands on its own

## Philosophy

This system is trying to preserve a specific feeling:

- low friction for writing
- high friction for leaking private material
- no duplicate editing surfaces
- just enough structure that "writing in public" does not turn into "dumping the vault online"

The garden should feel open, but not accidental.

## Next Improvements

Things I still want:

- a lightweight privacy scan for obvious secrets and local hostnames
- a small report of which published notes are blocked by private links
- better attachment deduplication
- a review step before commit and deploy
- a cleaner way to surface `garden` state in the Quartz UI
