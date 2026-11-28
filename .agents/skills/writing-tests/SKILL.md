---
name: writing-tests
description: Load before adding or editing any test or check script (scripts/check-*.mjs): the four-question gate, owner boundaries, junk patterns, retention bar; what plays the test role here today.
---

# Writing tests

First decide whether the test should exist (gate), then write it at its owner boundary. Skill `verify`
says which check proves which change.

## What is a test here today

This repo has **no unit-test suite and no test runner** (`package.json` has none; do not add one as
a side effect of a change: new tooling is the owner's decision). The tests are:

- `npm run check:types` (`astro check`) - types and diagnostics;
- `npm run build:all` - every page of the main site and both brand sites must build;
- `npm run check:links` (`scripts/check-links.mjs`) - internal links and `#anchors` in the built `dist*` dirs;
- `npm run check:ads` (`scripts/check-ads.mjs`) - ad texts against the site, needs `private/ads/`, not in CI;
- the CI guards for agent files, comments and the docs budget (vendored in `.github/scripts/`, never edited here).

A new check goes next to these: a `scripts/check-*.mjs` (plain Node, no new dependency unless the owner
agrees) wired as an `npm run check:*` script, and into `pr-checks.yml` only with the owner's yes, since
the PR must then say it touches the gate. Whether a test runner (for example for a pure function in
`src/data/`) is worth adding is a question for the owner, not a decision for the change in hand.

## Gate (four questions; a missing answer means do not add the test yet)

1. **Observable contract:** what observable behaviour, invariant or independent contract does it protect?
2. **Credible regression:** what realistic bug makes it fail?
3. **Owner boundary:** why does existing coverage not already catch it? Each contract has one primary
   test at the strongest boundary (table below); another layer needs its own distinct risk. Prefer
   extending an existing check or a shared table over a near-duplicate.
4. **No test-only seams:** does it require a production export, flag, wrapper or helper that no
   production caller needs? If yes, test at the real boundary instead.

Check the test against every junk pattern below. A test that breaks under a behaviour-preserving
refactor asserts implementation, not behaviour: rewrite it at the owning boundary.

A bug's regression test must demonstrably fail on the pre-fix code for the intended reason and pass
after the fix. If a test uncovers a real bug, fix the bug; never weaken an assertion to match broken
output. Record the four answers in the PR description (section "Bramka testów"). Synthetic data only.

## Owner boundaries

Default owners for the contracts of this project:

| Contract | Owner boundary |
|---|---|
| Every page of the main site and the brand sites builds | `npm run build:all` (the production build runs the same) |
| Internal links and anchors resolve | `scripts/check-links.mjs` on the built `dist*` dirs |
| Types of components, data and `sites/<brand>/brand.ts` | `astro check` |
| Ad texts, prices, address and hours agree with the site | `scripts/check-ads.mjs` (workstation, with `private/`) |
| Rendered markup of a page (headings, links, a brand's own photos only) | a check script that reads the built HTML in `dist*`, never page source |
| Contact form and panel handlers (PHP) | no automated test in this repo; the PR states what the owner has to try on the hosting |
| Agent files, comments, docs budget | the vendored guards in `.github/scripts/` (not edited here) |

## Junk patterns

Reject a new test matching one of these unless the retention bar names the independent contract it guards:

- an assertion-free probe (only runs code, or only checks a status or exit code when a content check exists at the same boundary);
- self-comparisons and identity copiers (`x === x`, a value copied straight back);
- restating declarations (a route list, a data list, an enum kept in step by hand);
- exact greps of source files or large HTML snapshots instead of the rendered contract;
- private-helper or call-shape tests duplicated by a test at the boundary;
- the same contract at two layers with no distinct risk;
- a test whose only purpose is to keep a test-only export, wrapper or dead code;
- expected values computed by the same logic under test;
- mocks that implement the asserted behaviour;
- fixtures that pre-set the state the code under test should produce;
- negative controls that pass for an unrelated reason (a missing file instead of a broken link);
- names promising more than the input exercises.

## Retention bar

Keep a test that independently enforces a public contract (links and anchors of the built site, a
page that must build), a configuration or build contract, or a project rule that a build cannot
see (for example a brand page showing only its own brand). Also keep regressions with a credible
failure mode (including order- or time-dependence fixes), and source inspection only when it is the
cheapest independent guard that fails on a real contract break and survives renames. A flaky test is
never skipped, retried until green or given a `sleep`: find the cause, or delete it and say why in the PR.
