# wix-archive

A static archive of every Wix blog post that was never manually migrated
into [arsene-cms](https://github.com/lionelleboiteux/arsene-cms) as a real
article — so an old `https://www.fantasy-coach.fr/post/{slug}` bookmark
still resolves to something real once the Wix account closes, without
adding a single row to Postgres or touching Supabase Storage.

## How this works

This repo holds **generated output only**. The actual generation script —
`generate.ts` — lives in `arsene-cms/scripts/wix-archive/`, not here: it
needs real imports of that repo's own page shell (`src/site/render.ts`'s
`page()`, for the shared header/CSS/`<fc-nav>`) and the Wix→HTML converter
already built for the real import pipeline
(`scripts/wix-import/convertRicos.ts`), so a single source of truth stays
in one place rather than being duplicated across two repos.

To regenerate:

```bash
# from a sibling checkout of arsene-cms
cd ../arsene-cms
WIX_API_KEY=... WIX_SITE_ID=a45d85ad-4c77-4639-8575-17a33e476277 \
SUPABASE_URL=... SUPABASE_ANON_KEY=... \
  node --experimental-strip-types scripts/wix-archive/generate.ts
```

This fully replaces `../wix-archive/site/` (a sibling checkout of *this*
repo, by default) — so a post that's since been manually promoted to a
real Arsène article correctly loses its stale archive page too, not just
gets orphaned. `ARCHIVE_LIMIT=5` runs a small dry run first — worth doing
before a real full run.

Once `site/` is regenerated, commit and push it here — the
`.github/workflows/pages.yml` in this repo takes care of publishing
whatever's in `site/` to GitHub Pages, the same way
[pronos](https://github.com/lionelleboiteux/pronos)/
[compos](https://github.com/lionelleboiteux/compos)/
[dnp](https://github.com/lionelleboiteux/dnp)/
[groupes](https://github.com/lionelleboiteux/groupes) already publish.

## What's excluded on purpose

- Any post already a real `articles` row in Arsène — those get the real
  editorial treatment (writer attribution, taxonomy, SEO tuning) and are
  served from `cms.fantasy-coach.fr` directly, never from here.
- Writer avatars/bylines — archive posts have no Arsène writer row, so
  each page is attributed generically ("Fantasy Coach — archive Wix")
  rather than resolving a Wix member ID to a name.
- Image optimization — original Wix image bytes are re-hosted as-is, no
  resize/webp conversion. This is a one-time archive of old content, not
  the real image pipeline.

## DNS

`archive.fantasy-coach.fr` → `lionelleboiteux.github.io` (this repo's own
GitHub Pages custom domain, via the `CNAME` file `generate.ts` writes into
`site/` on every run).
