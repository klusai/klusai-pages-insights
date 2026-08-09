# Writing a post here

This site is written by the team and by agents working alongside it. The backlog
lives in the **KlusAI Insights** project in Linear.

## The loop

1. Take an issue from the backlog, set it In Progress.
2. **Read the source material the issue names** — the PR, the incident, the
   measurement. A post that could have been written without reading it is the
   failure mode this site exists to avoid.
3. Write `_posts/YYYY-MM-DD-slug.md`.
4. Open a PR. Merging publishes: GitHub Pages rebuilds in about a minute.
5. Attach the rendered page to the issue and close it.

## Front matter

```yaml
---
layout: post
title: "Sentence case, specific, no colon-subtitle if you can avoid it"
date: 2026-08-09
categories: kos celery     # space separated, lower case, reused
excerpt: >-
  Two sentences that say what the reader will know by the end. This is what
  shows on the home page, the blog index and the feed, so it is not optional.
---
```

The theme is shared: `remote_theme: klusai/klusai-pages-brand`, the same one
research.klusai.com and academy.klusai.com use. Two things it dictates, both
easy to get wrong:

- **`categories`, not `tags`.** The `post` layout renders `page.categories` as
  the eyebrow above the title. Tags render nowhere.
- **The blog index lives at `/blog/`.** Both the home layout and the post layout
  link there by that exact path. A different permalink leaves those links 404.

Never fork a layout into this repo to change presentation. Fix it in the brand
repo and every site picks it up on its next build; that is the whole point of
the shared theme, and three copies of the tagline is the drift it exists to
prevent.

## Rules

**Ground everything.** Every claim traces to code, a PR, an incident or a
measurement. If you find yourself writing "many teams find that", stop: that is
a post about the industry, and this is a site about our work.

**Numbers carry a source.** Same rule the marketing site enforces through
`data/site/stats.yaml`. If you cannot say where a figure came from and when it
was last checked, leave it out.

**Never name a client or partner.** Describe the shape: "a Dutch notarial
software platform", not the partner. This is contractual, not stylistic.

**Nothing security-sensitive.** How a queue starved is a good story. A token
layout or an exploitable timing detail is not.

**Voice.** Short declaratives. No em-dashes.

**Positioning language is machine-checked, and you cannot see the list from
here.** Until September 2027 nothing we publish may sell engineering capacity,
and Copy v2 bans four marketing cliches on top of that. The single source of
truth is `scripts/check_banned_terms.py` in klusai-webos. **Read it before you
write**, and do not restate the terms in this repo: the org-wide sweep scans
these files too, so a document that lists a banned term in order to forbid it
fails the job just as loudly as a post that uses it. (This paragraph is the
second draft for exactly that reason.)

The sweep runs daily and fails *after* your PR has merged, so the guard will not
catch you on the way in.

**End with the CTA.** Close with a link to
`https://klusai.com/contact/?intent=scoping-session`. The `intent` parameter is
what makes reading-to-booking measurable; a bare link to the contact page loses
that.

## Shape that works

Lead with the symptom as it actually appeared, not with the conclusion. Keep the
false leads in — the value is usually in why the obvious explanation was wrong.
Close with what you would tell someone hitting the same wall.

Short is fine. A precise 600 words beats a padded 2,000.
