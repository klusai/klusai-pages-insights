---
layout: post
title: "Our pre-push gate was a second CI, and the slower one"
date: 2026-09-28
categories: kos ci
excerpt: >-
  Every push from our shared build box ran the full pull-request suite locally
  before it could reach GitHub, and the queue grew faster than it drained. We
  kept the checks that catch cheap mistakes, let CI prove the rest, and found
  the two guards that only existed locally.
---

The message was short: nothing seems to be moving to GitHub for hours.

Several coding agents share one Mac Studio. Every `git push` from it runs a
pre-push hook, and until this week that hook ran `make check-pr`: a local copy
of every required pull-request job, with a fresh virtualenv and fresh test
databases. That box can run two of those gates at once. When we looked, five
pushes were in flight: two gates running, three waiting for a slot.

## It was not stuck, it was full

The first suspect was a deadlock. It wasn't one. No lock was stale and every
process was making progress, just slowly. The gate writes a timing ledger, so
the next step was to read it instead of guessing.

Between 15:29 and 16:41 UTC that day, ten gates ran. Full runs took 15 to 19
minutes each, and the core test phase alone took 15 to 17 minutes, at a load
average peaking between 25 and 29. Six of the ten were red, and every red run
sends its push back to the end of the queue.

The "for hours" part was partly a false lead. The ledger has no gate at all
between 11:18 and 15:29 UTC. Nothing was pushed in that window, so the gate had
nothing to block. Once people did push, the queue grew faster than it drained.

## Making the local gate smaller had already failed

This was not the first attempt at speeding the gate up. We had built a selector
that ran only the core tests a diff could affect, and audited it against every
committed negative control, the mutations we keep to prove a test can fail. A
version that respected every way this codebase loads code at runtime could scope
only 7 merges in 100. A looser version scoped more, but missed about 5% of the
mutations it should have caught. We parked it.

The next idea was to run the core suite on CI runners for the pushed commit
before the push completed. That works, but the pull request then runs the same
jobs again: CI twice per push.

## So what was the local gate for?

Pull-request CI ran the same phases on its own runners. Across 33 green PR runs
the week before, the median took 8 minutes, and branch protection already
required those jobs. The local gate was a second CI, running on the most
contended machine we own.

Before deleting anything, we mapped every local phase to the CI job that
enforces it. Almost all of them lined up. Two did not:

- **Negative controls.** CI ran the PR's changed negative controls, but that job
  was not a required check. The local gate was the only thing that enforced it.
- **Browser-gate stability.** When a change touches a must-merge browser test or
  its runtime, the local gate ran that suite five times in clean processes. CI
  ran it once.

## The change

The hook now runs `make check-push`: lint, the static contracts, test-name
integrity and the translation catalogs. It needs no database and no gate slot,
and it measured 4 minutes 31 seconds at a load of about 15. The hook still
refuses to push anything but the exact, clean, committed head on top of current
main. Everything else is proved by pull-request CI.

The negative-control job now feeds the required test check, so a PR whose
control does not go red cannot merge. The five-run browser proof stayed local,
only for the changes that need it.

The fast lane caught something on its first run. Our own edit to the CI workflow had
broken the patch context of two existing negative controls, and the lint that
checks controls still apply refused the push.

## The second bug was a string match

The first push through the new hook still took 18 minutes 57 seconds. The five
browser runs had fired, although the change touched no browser test.

The selector decided on the literal string `e2e_gate`. Any diff line containing
it qualified: rule prose, a Makefile comment, and test docstrings explaining
that a nightly test is *deliberately not* in the gate. Later that night, a
walkthrough script that only reads the browser endpoint sent another branch
through the same fifteen minutes.

Now a module counts only if it actually tags a test into the gate, and a file
counts only if it is on the list of runtime files the gate executes. Prose,
comments and negative-control fixtures never count. Each rule has a test and a
committed mutation that turns that test red. The push that shipped this fix took
2 minutes 21 seconds.

## What we would tell someone at the same wall

A local gate that duplicates required CI buys you very little. It catches the
same failures later, on a busier machine, and it makes every other push wait.
Keep locally what is cheap and fails often, and let CI prove the rest.

Before you take the local gate away, list what each phase enforces and find
where CI enforces it. The gaps you find are guards that only ever existed on one
laptop, or in our case one shared box.

We are reviewing a week of numbers next: push-to-green time, CI queueing and
first-attempt red rate.

If your delivery pipeline has grown a second CI of its own, we can help you find
out which parts of it still earn their place:
[book a scoping session](https://klusai.com/contact/?intent=scoping-session).
