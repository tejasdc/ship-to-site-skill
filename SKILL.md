---
name: ship-to-site
description: Publish any project, essay, note, or stream thought to Tejas's personal website tejas.nyc. Use when Tejas says "ship this to my site/personal website", "publish this", "add this to my garden/portfolio", "publish this thought/snippet to my stream", or similar, from ANY project directory. Handles first-publish judgment for slugged artifacts, then runs the deterministic publisher script.
---

# ship-to-site — publish to tejas.nyc

The site is registry-driven: one manifest (`scripts/artifacts-registry.json` in the site
repo) is the single source of truth; indexes (/projects, /essays, /notes, /stream),
flat artifact pages, composed project pages, chrome, analytics beacon, screenshots,
deploy, and verification are ALL handled by one deterministic script. Your job is only
the judgment the script cannot make.

## The command (does everything mechanical)

    node /Users/tejasdc/workspace/chann-app/scripts/ship-to-site.mjs [flags]

Run from the project's directory (or pass --source). Useful queries:
`--help` (usage), `--list` (registry), `--check-slug <slug>` (availability),
`--dry-run` (build, preview deploy, and verify without committing or promoting).
`--check-slug` reports `available: true` with a note when the slug is already
reserved for THIS source dir — publishing then updates the entry in place.
The site build warns about empty content collections ("projects"/"writing"/"threads"
do not exist or are empty) — benign noise, not a failure.

## Stream publish (no descriptor, no questions)

If Tejas says to publish a thought/snippet to the stream, ask nothing and run:

    node /Users/tejasdc/workspace/chann-app/scripts/ship-to-site.mjs --stream "exact text"

For longer snippets already in a file, use `--stream-file <path>`. Stream entries do not
take slugs and do not compete with artifact URLs. The live address is `/stream/#<id>`.

## Re-publish (descriptor exists)

If `.publish.json` exists in the project directory: run the command with NO flags.
Report the live URL. Ask nothing. Done.

## Updating or RENAMING an existing project — never create a duplicate

### Taking a page offline

Before `--unpublish`, read the site's takedown rules and existing `public/_redirects`.
A native Astro page needs its route source removed as well as its listing. Cloudflare
Pages can keep serving removed files despite successful URL, prefix, and whole-zone
purges: use the site's established retirement redirects for the former route and its
dedicated public assets. Verify the custom-domain redirects and listing absence in a
browser; a missing build file or a 404 on the deployment URL is not sufficient.
Evidence: the [2026-09-09 10K takedown](https://github.com/tejasdc/chann.app/commit/23bb572)
reproduced retained assets after all three purge methods; retirement redirects bypassed them.

The registry key is the **slug**. To change a published project's title, description,
external URL, or specimen, edit those fields in the EXISTING `.publish.json` and re-run
the publisher **with the same slug**. This updates the entry in place.

**NEVER change the slug to rename a project.** A new slug mints a brand-new registry
entry and leaves the old one behind — the project then appears TWICE on /projects.
(This happened 2026-08-02 renaming Electric Love → Pass the Aux: changing the slug from
`electric-love` to `pass-the-aux` created a duplicate.) The user-facing title is the
`title` field, not the slug; the slug can stay as-is even when the product is renamed.

If a slug genuinely must change, you must remove the old one too:
`node scripts/ship-to-site.mjs --unpublish <old-slug>`, then verify /projects lists the
project exactly once. Gotcha: `--unpublish` on an EXTERNAL entry used to fail its own
verification (it checked for a 404 on a URL that legitimately 200s); fixed 2026-08-02.
If a duplicate ever persists in `scripts/artifacts-registry.json` after an unpublish,
remove the stale entry from that JSON directly, commit, and `npm run deploy`.

## Pre-staged registry entry (no descriptor, but the slug is already in the registry)

A third state: no `.publish.json` in the project dir, but the registry already holds an
entry for this source dir (usually `state: disabled`, staged ahead of time). Publishing
from the same source dir updates the entry in place and flips it live — the pre-staged
judgment values (slug, title, level, files, specimen) win unless overridden by flags.
BUT pre-staged entries can be stale against Tejas's later instructions: before the
first real publish, surface the consequential values — especially the level ("this will
stamp the site header/footer onto the page" for framed) — and confirm rather than
silently inheriting.

## First publish (no descriptor) — the judgment pass

NEVER ask about mechanics, placement, or where types go — that is codified. Exactly two
judgment items, then run:

1. **Slug options.** Propose 2-3 short flat slugs (check each with --check-slug).
   Present to Tejas, he picks. Flat namespace: no /projects/ or /essays/ prefixes ever;
   `https://tejas.nyc/<slug>/` is canonical for projects, essays, and notes.
2. **Specimen brainstorm (projects only).** Open/screenshot the actual page. Find the
   element that carries the piece's soul (a canvas, an animation, a hero region) and
   propose: "I'd export THIS element as the live card; alternatives: full-page iframe /
   this screenshot moment / a short loop." Discuss with Tejas briefly; record his pick.
   Kinds: live (element or preview.html or full page, lazy-loaded), loop, image
   (auto-screenshot; --specimen-selector for element-level), text (notes).

   **If the page animates, the specimen is live (or loop). Never let an animated page
   fall through to the static auto-screenshot** — the script's `image` default produces
   a blurry upscaled frame on the /projects card, and Tejas rejected exactly that
   (shape-of-stories, 2026-08-01). The proven live pattern: add a small `preview.html`
   to the PROJECT dir (include it in --files) that iframes the published page scaled to
   fit — fixed 1280px inner width, `transform: scale(innerWidth/1280)`,
   `pointer-events: none`, `scrolling="no"` — and set
   `--specimen live --specimen-source "/<slug>/preview.html"`. The card then shows the
   real page with its animation running, sharp at any size. `image` is acceptable only
   for genuinely static pages.

   Card title/desc are reading surface: no counts, statistics, or method language
   (write "an atlas of the shapes of stories", not "114 stories").

Then run the script with: --slug --title --desc --type project|essay|note --level
embedded|framed|sovereign [--attribution flow|fixed] [--file essay.md] [--files a,b,c]
[--specimen ... --specimen-selector ...].
Title/desc: write in Tejas's register, no marketing copy, no invented facts.

## Levels (choose, don't ask)

- **embedded** — essays/notes; the site renders them (markdown via --file).
- **framed** — simple static pages WITHOUT their own visual identity; site
  header/footer stamped into the same document at build. This is first-party code on
  tejas.nyc, not iframe isolation.
- **sovereign** — any page that ships its own complete visual identity and nav (a
  committed color world, its own header) — most finished projects are this. Only a
  one-line "made by tejas.nyc" attribution is added. Verdict from the first real
  publish (shape-of-stories, 2026-08-01): Tejas rejected the framed chrome on sight
  on a fully-designed page — framed is NOT the default for projects; steer
  fully-designed standalone pages to sovereign. Subdomain tenants: see the runbook
  (docs/runbooks/publish-subdomain.md) — rare, deliberate. When a path-mounted
  project previously had a subdomain, 301 the subdomain to the path (zone dynamic
  redirect rule; stories.tejas.nyc is the precedent).

### External projects (hosted elsewhere)

For a project that runs on someone else's infrastructure (Render, Fly, a server
app that can't be static): publish with `--external https://...` instead of
`--files`/`--level`. No page is composed on tejas.nyc — the /projects card links
straight out (meta shows ↗) and the specimen image is auto-captured from the live
external URL. Precedent: electric-love (Render, X-Frame-Options: DENY so no live
iframe either — image specimen is the only option). Verify the external URL is up
before publishing; the publisher checks it returns 200.

### Sovereign attribution placement (choose, don't ask)

The "made by tejas.nyc" line on sovereign pages has two variants, recorded per
artifact as `attribution` in the descriptor/registry (`--attribution flow|fixed`):

- **flow** (default) — a normal footer at the end of the document. Correct for any
  page that scrolls; it sits at the true bottom, visible only when you get there.
- **fixed** — a small viewport-anchored bottom-right badge. REQUIRED for fullscreen
  apps whose root is `position: fixed`/`inset: 0` (games, canvas apps): those pages
  have no document flow, so a flow footer surfaces at the TOP-LEFT corner instead
  of the bottom (heartsville, 2026-08-01). The reverse mistake is just as bad: a
  fixed badge on a scrolling page floats persistently over content while reading
  (stories, same day).

Test to apply: does the page scroll? → flow. Is it a fullscreen fixed-layout app?
→ fixed. Check the actual page (screenshot or DOM), not the vibe.

## Laws

- Content markdown is copied into the site repo at publish; the site's git history is
  the publication history. Project runtime files stay pointer-only: source of truth
  stays in the PROJECT directory and the site pulls those files at build. Never copy
  project files into the site repo by hand.
- Publish only Tejas-owned project HTML. Composed projects execute as first-party
  tejas.nyc pages and can access same-origin browser state.
- No invented copy anywhere; real dates only (the script stamps them).
- The script builds, deploys a Cloudflare Pages preview, verifies 200s + beacon + the
  new artifact, then commits and promotes to production. If PREVIEW verification fails,
  it leaves no registry/content/descriptor trace. Failures AFTER promote are different:
  the commit, production deploy, and descriptor already exist — the fix is usually to
  re-run with no flags, not to unwind. (Production verification retries with backoff
  for ~60s to absorb CDN propagation lag before declaring failure.) If it exits
  nonzero, report the error; never treat a failed verification as a success.
- On `--dry-run`, screenshot the preview URL and look at it (or show Tejas) before
  promoting — this caught a rejected chrome once already.
