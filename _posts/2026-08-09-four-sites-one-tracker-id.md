---
layout: post
title: "Four sites, one tracker id, and three dashboards that said Setup pending"
date: 2026-08-09
categories: kos observability
excerpt: >-
  One Django app serves four public sites. All four shipped the same analytics
  tracker id, so three products filed their traffic under a fourth, and the fix
  was less interesting than the reason we could not see it.
---

kOS serves four public sites from one Django application: the company site, two
vertical products, and our compute property. One codebase, one deployment, four
domains resolved by host at request time.

Analytics was a single setting.

```python
PLAUSIBLE_SCRIPT_ID = os.environ.get("PLAUSIBLE_SCRIPT_ID", "")
```

Every page on every host rendered that one id. So three of the four sites
reported their pageviews into the fourth's dashboard, and their own dashboards
sat at zero, showing the analytics provider's cheerful **Setup pending** badge
for months.

## Why nobody noticed

Because the failure looks exactly like the thing you would expect to see anyway.

A new product subdomain with no traffic is unremarkable. "Setup pending" reads
as *you have not finished wiring this up yet*, which was true at some point and
so never got questioned. Meanwhile the company site's numbers went **up**, and
nobody investigates numbers going up.

The tracker was installed correctly, firing correctly, and reporting correctly.
It was reporting to the wrong place, and there is no error for that.

## The fix, and the part worth arguing about

The id now resolves per request hostname:

```python
PLAUSIBLE_SCRIPT_IDS = {
    "klusai.com": "...",
    "lex.klusai.com": "...",
    "med.klusai.com": "...",
    "razorbridge.eu": "...",
}
```

Hostname, deliberately, and not the internal notion of which product a request
belongs to. Our architecture separates a *vertical* (a brand-neutral capability
like legal or healthcare) from a *deployment profile* (the per-host brand and
storefront). Several hosts can share a vertical. An analytics property is a
domain. Key it by the thing it actually is.

The argument worth having is about the fallback. The obvious design is for an
unrecognised host to fall back to the main site's id, so tracking never silently
stops. That is precisely the bug: it is how three products' traffic ended up
somewhere else, and the failure is invisible because the numbers still look
plausible.

An unmapped host now gets **no tracker at all**. An empty dashboard is a
question somebody eventually asks. Contaminated numbers are a decision somebody
eventually makes.

A test reads the deployment-profile registry against the id map and fails if a
public host has no analytics property, so the next product cannot launch into
the same hole.

## Two things this generalises to

**Config that is global in a multi-tenant app is a claim that the thing is
global.** Every setting the request path touches deserves the question: is this
per deployment, or per instance? Ours was in the wrong bucket for months and
nothing complained, because nothing could.

**A fallback is a decision to be wrong quietly.** Falling back is right when the
degraded state is visible and harmless. When the degraded state is *plausible
data*, failing closed is kinder to the people who will read it later.

There is a postscript. Fixing the split let us look properly, and we found the
company site had recorded nothing at all for five months on a lapsed
subscription. The tracker was firing the whole time. Two different ways to have
no data, both of which look like a quiet quarter.

---

Building something in a regulated European industry and want it in production
rather than in a slide?
[Book a use-case scoping session](https://klusai.com/contact/?intent=scoping-session).
