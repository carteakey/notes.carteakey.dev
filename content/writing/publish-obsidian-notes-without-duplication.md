---
title: How I Publish My Obsidian Notes Without Duplicating Them
publish: true
garden: seedling
kind: essay
slug: publish-obsidian-notes-without-duplication
tags:
  - obsidian
  - quartz
  - digital-garden
  - publishing
description: A practical publishing model for turning an Obsidian vault into a public digital garden without creating a second source of truth.
---

I have wanted a digital garden for a while, but I did not want to create a second writing system just to get there.

That is the part that always felt wrong to me. The whole point of using Obsidian is that it is already where I think, collect, connect, and revise. If publishing those notes means copying them into another repo, cleaning them up there, fixing assets there, and doing "web-only edits" there, then the website stops being a garden and starts becoming a fork.

The fork is the problem.

Once I accepted that, the architecture got much simpler:

- my vault is the source of truth
- the Quartz site is generated output
- a note is public only if I explicitly mark it as public

That is the core idea.

## The Real Goal

I do not want to publish every note. I want to publish the notes that survive contact with a simple question:

Is this actually useful in public?

That usually rules out a lot of things:

- fleeting inbox material
- raw personal notes
- homelab secrets and internal config
- notes that only make sense because they link to five private notes around them

What remains is the part of the vault that actually benefits from being open:

- working explanations
- reference notes
- project notes with some shelf life
- half-finished ideas that are still coherent enough to be worth sharing

That is what I want the garden to be.

## The Publishing Contract

The note needs to opt in:

```yaml
---
title: My Note
publish: true
garden: seedling
kind: concept
slug: my-note
description: What this note is about.
---
```

`publish: true` is the switch.

`garden` is the maturity level:

- `seedling` means it is public and still forming
- `evergreen` means it is stable enough to stand on its own
- `private` means it stays in the vault

And `slug` is there because vault paths make terrible URLs.

Publicly, I only want a shallow structure:

- `/my-note`
- `/notes/...`
- `/projects/...`
- `/writing/...`

Root-level notes are the default. Buckets are optional when a note benefits from a little more structure.

## The Pipeline I Settled On

The script lives in my vault under `1. system 📊/scripts ⚡/export-garden.py`.

It exports from:

```text
/Users/kchauhan/.superset/worktrees/vault-76/flannel-feet
```

into:

```text
/Users/kchauhan/repos/carteakey.dev/notes.carteakey.dev/content
```

More specifically:

- notes go into shallow buckets like `content/notes`, `content/projects`, and `content/writing`
- referenced assets go into `content/_attachments/garden`

That split matters because I do not want the public site to inherit my entire attachment folder just because one note uses one image.

The script only exports notes marked `publish: true`, and it excludes obviously private top-level areas like inbox, finance, personal, archive, and config.

It also does not preserve vault paths. Public output is driven by `kind` and `slug`, not by wherever the note happens to live internally.

It also fails on an important case: if a public note links to a private note, the export stops.

That sounds strict, but it is exactly the kind of strictness I want. Quietly publishing a page that depends on private context is how weird leaks happen.

## Why This Solves Duplication

There are two kinds of duplication I care about.

The first is literal duplication:

- one note in Obsidian
- one note in the website repo

I solve that by refusing to author notes in the website repo. The repo gets generated output only.

The second is conceptual duplication:

- the same idea as a blog post
- the same idea as a garden note

For that, I want a cleaner split.

The blog is for finished arguments.

The garden is for notes that are alive.

If both exist, one should be canonical and the other should point to it. I do not want to publish the same piece twice just in two different formats.

## The Nice Side Effect

This setup makes public writing easier without making private writing more fragile.

I can keep writing normally in Obsidian.

I can promote notes when they are ready.

I can leave notes as seedlings for a long time without pretending they are polished essays.

And I do not have to maintain a separate copy of my knowledge just to make the website look neat.

That, to me, is the actual promise of a digital garden. Not "publish everything," but "make it easier for good notes to become public without turning publication into a second job."

## The Current Rough Edge

The first dry run of the exporter immediately found something useful: a handful of notes already marked public still link to private notes.

That is good.

It means the pipeline is doing its job early, before anything goes live.

The right move there is not to relax the exporter. The right move is to fix the notes:

- publish the dependency
- remove the dependency
- or rewrite the note so it can stand on its own

That is exactly the kind of friction I want.

Low friction for writing.

High friction for accidental publishing.

That feels like the right trade.
