# From evidence to your own build

Read this when the user wants to **build their own version** of a feature or app
after (or instead of) analyzing it. REA explains *how* a target works; this guide
turns that into a scoped, tested, measurably comparable, legally clean rebuild.

Adapted from the MIT-licensed Replica skill pack by Jake Schincariol
(github.com/Jakeschincariol/replica-skill). The tools in `../tools/` are
standard-library Python 3.8+, touch no network, and need no installs.
`LICENSE-replica` in that folder carries their license.

This guide is a condensed map. The plugin also ships Replica's eleven full
skills, one per stage (`replica-recon`, `replica-entrepreneur`,
`replica-architect`, `replica-design`, `replica-build`, `replica-backend`,
`replica-test`, `replica-diff`, `replica-brand`, `replica-launch`,
`replica-deploy`). Use this guide to connect REA's Evidence to the rebuild,
and load the matching skill for the complete instructions of each stage.

Planning files live in a `replica/` folder in the user's project, so each stage
can read the previous one. Paths like `tools/parity.py` are relative to this
skill's folder; when installed as the rea-replica plugin that is
`${CLAUDE_PLUGIN_ROOT}/skills/reverse-engineer-anything/`. Run each tool from
the user's project with that absolute path, not from inside an analysed target.

## The clean-room rule (read first)

REA can surface decompiled pseudocode, bundled JavaScript, strings, resources
and network endpoints. That is **evidence of behavior**, not material to ship.

- Describe behavior from the Evidence in your own words, then write every line
  of the rebuild fresh. Never paste or lightly rename decompiled functions,
  extracted JS modules, SVGs, images, fonts, sounds or UI copy.
- Never call the target's private endpoints, reuse its OAuth client, or proxy
  through its servers. Use official public APIs with the user's own keys.
- Keep reference material (screenshots, extracted trees, Evidence bundles) in
  `replica/screens/` or the analysis workspace. It never goes into the build.
- If the user's account terms forbid building a competitor, say so and rely
  on public sources and artifacts the user may lawfully study.
- What can be rebuilt: features, flows and UX patterns. What cannot: licensed
  content, the user network, data the app owns, partner deals, regulated
  licences. Record those as `skip` rows with a reason.
- This is not legal advice. For anything sold, the user should check terms and
  talk to a lawyer.

## Stage 1: recon map

Combine REA Evidence with public sources (help center, pricing page, changelog,
store listing, walkthrough videos, public API docs, the user's own account).

Write `replica/recon.md`:

- **Scope**: which app, which platform, which slice (default: the core loop the
  users pay for), and who the rebuild is for.
- **Screens** with stable IDs `S01, S02…`: route, purpose, key components,
  states seen (empty, loading, filled, error, permission denied, mobile).
- **Flows** `F01, F02…`: goal, screens passed through, edge cases, and the
  click count on the happy path (the number to beat).
- **Components**: name, variants, states, screens that use them.
- **Inferred data model**: entities, fields and relations, each with its
  evidence and a confidence. Cite REA Evidence IDs where the inference came
  from static or runtime analysis (for example a SQLite schema from
  `inspect_sqlite_database`, or a request shape from a HAR capture). Mark
  guesses as guesses.
- **Size**: S (a weekend), M (a few weeks), L (a quarter), XL (rescope). No
  promises of a perfect clone.

Write `replica/features.csv` from `../templates/features.csv`
(`feature,area,priority,original,clone,notes`). Priority is `must|should|could`;
`clone` starts at `no` and becomes `yes`, `partial` (with a note), `no`, or
`skip` (with a reason).

## Stage 2: architecture

Prefer boring managed tech the user already knows. One database, no
microservices. Turn the data model into SQL: uuid keys, owner column on
user-owned rows, explicit `on delete` rules, indexes on foreign keys and
filtered columns, `timestamptz` in UTC, money as integer cents plus currency,
and access rules (row-level security or one authorisation check per query).
Put hard invariants (no double-booking) in the database, not the UI.

Note the parts that bite: time zones, idempotent webhooks, races, rate limits,
uploads, search, realtime, offline, email deliverability, GDPR deletion.

Build order: a thin vertical slice of the core loop first, then must-haves,
should-haves, could-haves, then the fixes from Stage 6.

## Stage 3: design system as tokens

Record **roles, not colours**: bg, surface, border, text, text-muted, accent,
on-accent, danger, success, warning. Measure type scale, spacing base (usually
4 or 8), radius, shadow and layout widths from screenshots. Start from
`../templates/tokens.json`, list every text/background pair, then:

```bash
python3 -I tools/contrast.py replica/design/tokens.json   # WCAG AA; exit 1 on failure
```

Use open icon sets and open fonts in place of the target's assets. Keep the
target's signature brand colour as a neutral placeholder until Stage 7.

## Stage 4: build and test

Build screen by screen against the recon map with tokens only. For each screen:
every state, 390px and 1440px widths, keyboard-only completion, no console
errors, no borrowed copy, its `features.csv` rows updated, and a screenshot at
the reference viewport saved to `replica/clone-screens/S07.png`.

Testing: one test plan per flow with numbered cases (`F01-H1` happy path,
`F01-E3` edge, `F01-N2` negative). Edge checklist: empty and very long input,
emoji, two tabs, double submit, back and refresh mid-flow, slow or offline
network, expired session, a second user's data (must be invisible), time
zones and DST, keyboard only, screen reader labels. Automate with Playwright
using role and label selectors, failing on console errors and 5xx responses.

Log bugs in `replica/bugs.md` using `../templates/bug-report.md`, only what
was reproduced:

| severity | means |
| --- | --- |
| S1 | data loss, security hole, payments wrong, core flow blocked |
| S2 | a feature broken, no workaround |
| S3 | broken with a workaround, or visibly wrong |
| S4 | cosmetic |

Fix S1/S2 first; write the failing test before each fix. **Test the rebuild,
never the original.** Do not load-test or fuzz the target's servers.

## Stage 5: measure parity

```bash
python3 -I tools/parity.py replica/features.csv
python3 -I tools/imgdiff.py replica/screens/S07.png replica/clone-screens/S07.png --out replica/diffs/S07.png
python3 -I tools/imgdiff.py replica/screens/S07.png replica/clone-screens/S07.png --json > replica/diffs/S07.json
python3 -I tools/parity.py replica/features.csv --visual replica/diffs/*.json --markdown > replica/parity.md
```

`parity.py` weights must 3, should 2, could 1, counts partial as half, ignores
`skip` rows and features the original lacks, and flags missing must-haves as
not shippable. `imgdiff.py` compares edge-map layout, so a rebrand's new colours
do not count against you; `--mode pixel` is for the rebuild's own regressions.
Take both screenshots at the same viewport and in the same state.

Add a behaviour diff that the scores cannot see: clicks to finish the core
flow, error handling, what persists between visits, which emails arrive.
Where REA runtime capture is available, compare the user's own recorded
scenario of the original against the rebuild, as Evidence, not as copied code.

Verdicts: **not shippable** (a must-have missing or an open S1); **shippable**
(all must-haves, feature score 80+, no open S1/S2); **better than the original**
(shippable plus Stage 6 fixes). Report honest numbers.

## Stage 6: what users hate (the reason to exist)

Collect 100+ real public reviews across at least three sources (App Store,
Google Play, G2, Capterra, Trustpilot, Reddit, Hacker News, the product's own
feature-request board, its changelog). Read them as a person does, or use
official feeds within their terms (Apple's customer-reviews RSS, the HN Algolia
API). No scraping against site terms.

Rows go in `replica/reviews.csv` as `source,url,date,rating,text`, text copied
exactly. Then:

```bash
python3 -I tools/reviews.py replica/reviews.csv --out replica/feedback.md
```

It drops rows without a link, ranks themes (edit `tools/themes.json` for the
category), weights low and recent ratings, and marks thin themes (fewer than 3
reviews or one source). Read the unmatched low ratings by hand.

Write three ranked lists (what they hate, what is missing, what is unsolved),
a 5–8 item fix plan sized S/M/L and added to `features.csv` with
`original=no`, and a one-line positioning angle with its evidence count.

**Never fabricate** a review, quote, rating, count or source. State the sample
size. Reviewers' words are research, never testimonials.

## Stage 7: make it yours, then verify nothing was copied

Pick a name not confusingly similar to the original (list trademark, domain,
store and handle checks as **to run**, never "available" unless run), a palette
from a different hue family (re-run `contrast.py`), a logo brief that shares no
shape or colour pair with the original's mark, and fresh copy in a defined voice.

Then sweep the codebase. This is the clean-room check for anything that leaked
from analysis (names, domains, brand colours, identifiers such as
`OriginalAppEmbed`):

```bash
python3 -I tools/sweep.py . --avoid "Original Name,Its Company" --domains original.com --colors "#006bff"
python3 -I tools/sweep.py . --config replica/brand.json
```

Exit 1 means something is left. Fix until clean, and check by eye the favicon,
page titles, email templates, OG image and app icon. Do not launch or deploy
until the sweep is clean.

Also confirm by review that no decompiled routine or extracted module was
carried into the build under a new name. The sweep catches names, not logic.

## Next: backend, launch and deploy

Auth, payments and the security checklist, the landing page, pricing, the
store listing (`tools/listing.py`) and the production deploy with its preflight
gate are in [launch-and-deploy.md](launch-and-deploy.md). Nothing goes live
without the user's explicit go and a clean sweep.

## Templates

In `templates/`: `recon-map.md` (Stage 1), `features.csv` (feature matrix),
`architecture.md` (Stage 2), `tokens.json` (Stage 3), `test-plan.md`,
`bug-report.md` and `e2e.example.spec.ts` (Stage 4), `listing.example.json`
and `preflight.md` (launch and deploy).
