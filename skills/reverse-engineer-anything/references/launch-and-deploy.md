# Backend, launch and deploy

Read this after [rebuild-workflow.md](rebuild-workflow.md) Stages 1–7, when the
rebuild needs a real backend, a landing page, pricing, a store listing, or a
production deploy. Adapted from the MIT-licensed Replica skill pack
(github.com/Jakeschincariol/replica-skill). Tool and template paths are relative
to this skill's folder. For the complete instructions of each stage, load the
full skills `replica-backend`, `replica-launch` and `replica-deploy`.

## Rules that apply to every stage here

- **The user creates accounts and keys.** Never sign up for services, buy a
  domain, type a password or card, or paste a live key. The user creates the
  Stripe, database, email and OAuth projects and puts keys in `.env.local`.
  Write `.env.example` with every variable name and no values.
- **Official, public APIs only, with the user's own keys.** Never call the
  original's private endpoints (including any REA surfaced from bundles or
  captures), never reuse its OAuth client, never proxy through it.
- **Test mode first.** Stripe test keys until the deploy step.
- **Nothing goes live without the user's explicit go.**
- **Not until it is rebranded.** `tools/sweep.py` must exit 0.
- **No fake proof.** No invented testimonials, user counts, ratings, press
  logos or "trusted by" rows. Real beta users, with permission, are fine. The
  original's reviews are research, never testimonials.

## Backend

Write `replica/backend.md` with this checklist and keep it current.

**Auth**: email sign-up with verification, password reset, magic link if the
original has it, OAuth with the user's own developer apps. Http-only secure
cookie sessions, sign out everywhere. Roles and teams if the recon map has
them, behind one function answering "can this user do this to this record".
Account deletion that actually deletes (Apple requires it for apps with sign-up).

**Database**: checked-in migrations run by a script; access rules on every
table, tested by confirming a second user gets nothing back; a seed script with
realistic fake data (no real people); host backups confirmed enabled.

**Payments**: Stripe Checkout for plans and the Customer Portal for changes and
cancelling; never build card forms. Webhooks verify the signature, store the
event id, and are idempotent. Handle `checkout.session.completed`,
`customer.subscription.updated`, `customer.subscription.deleted` and
`invoice.payment_failed`. Subscription state lives in the database, updated by
webhooks; never trust the client. Cancelling is one click.

**Email and jobs**: transactional email from the user's own domain, templates
written fresh. Time-based jobs (reminders, digests, sync, cleanup) with retries
and a dead-letter log; store UTC, display in the user's zone.

**Integrations**: for each one, the official API, the fewest OAuth scopes, the
provider's review process and rate limits. Sensitive Google scopes need
Google's OAuth verification before public launch, which takes weeks; start early.

**Security checklist**
- [ ] secrets only in env vars, `.env*` git-ignored, nothing in client bundles
- [ ] server-side input validation on every route
- [ ] authorisation on every read and write, tested with a second user
- [ ] rate limits on auth, sign-up, and anything that sends email or SMS
- [ ] webhook signatures verified
- [ ] uploads size- and type-limited, served from a separate domain or bucket
- [ ] no user data in URLs or logs
- [ ] dependencies audited (`npm audit`)
- [ ] privacy policy lists every processor

## Landing page

Write `replica/launch/landing.md`, then build it:

1. **Hero**: the positioning angle as the headline (what, for whom), one line,
   one button, a screenshot of the real product.
2. **The problem**: the top two "hate" themes, paraphrased, never quoted.
3. **How it works**: three steps from the core flow.
4. **Features**: lead with the fixes from `replica/fixes.md`; parity is the
   price of entry.
5. **Pricing** table, 6. **FAQ** with real objections (importers, migration),
   7. **Final call to action**.

Write in the brand voice and run `tools/contrast.py` on the page tokens.

## Pricing

Write `replica/launch/pricing.md`:

- The original's public pricing and 2–3 alternatives in one table, each with
  its URL and the date read (prices change).
- What reviewers said about price and billing, with counts from `feedback.md`.
- The model: free plan or trial, per seat or flat, monthly and annual (annual
  usually two months free). Three tiers at most, named by who they are for.
- Fix billing complaints in the product: one-click cancel, clear renewal
  emails, no surprise per-seat jumps.
- The Stripe products and prices for the user to create.

## Store listing

Start from `templates/listing.example.json`, fill
`replica/launch/listing.json`, then:

```bash
python3 -I tools/listing.py replica/launch/listing.json
```

It checks App Store limits (name 30, subtitle 30, promotional text 170,
keywords 100, description 4000) and Google Play's (title 30, short description
80, full 4000), counts characters as a person does, flags ranking and price
claims in short fields, emoji, wasted keyword characters, and any name from
`avoid` anywhere in the listing. Exit 1 on an error.

Apple's App Review Guideline 4.1 (Copycats) rejects apps that simply copy a
popular app or make minor changes to its name or UI. Make the fixes and the new
brand visible in the screenshots and the first lines of the description.

Also prepare screenshots at the sizes App Store Connect and Play Console
require at upload time, privacy labels and the data safety form, privacy and
support URLs, the age rating, and review notes with a demo account.

## Launch plan

`replica/launch/launch-plan.md`: a waitlist or beta list, analytics and error
tracking live, the communities where the original's unhappy users talk (from
the review research), a Product Hunt or Hacker News post that leads with the
fix, and the first ten users to talk to by hand.

## Deploy

Fill `templates/preflight.md` into `replica/deploy.md`.

**Preflight** (any failure stops the deploy; say which and why):

```bash
npx playwright test
python3 -I tools/parity.py replica/features.csv             # all must-haves done
python3 -I tools/sweep.py . --config replica/brand.json     # exit 0
python3 -I tools/listing.py replica/launch/listing.json     # if shipping to stores
npm run build
```

Plus: no open S1/S2 bugs, privacy policy and terms live, a cookie banner if
non-essential cookies are used in the EU or UK, account deletion works, and the
favicon, titles and OG image are the user's own.

**Production services**: a separate production database with backups and
migrations run by the deploy; host env vars matching `.env.example`, live keys
only there; Stripe live mode with products and prices recreated, the production
webhook endpoint and secret, and one real purchase refunded; the production
domain added to every OAuth provider's redirect URIs and origins; the email
sending domain verified.

**Host and domain**: default Vercel for Next.js (Netlify, Cloudflare Pages, Fly
or Render otherwise), `main` deploys and pull requests get previews. After the
user buys the domain:

| record | name | value |
| --- | --- | --- |
| A | @ | the host's apex IP, from its domain settings |
| CNAME | www | the host's target (Vercel: `cname.vercel-dns.com`) |
| TXT | @ or subdomain | the host's verification value, if asked |

Email DNS from the provider: SPF (TXT), DKIM (CNAME or TXT), and DMARC starting
at `v=DMARC1; p=none; rua=mailto:you@yourdomain`, tightened to `quarantine`
once reports are clean. Choose one canonical host and redirect the other;
confirm HTTPS.

**Watch it**: error tracking, uptime checks on the home page and the core
flow's API, logs, privacy-friendly analytics, and an alert to the user. Then
run the core flow on the live site, and ask the user to try it on their phone.

**Mobile**: Expo `eas build` then `eas submit` to TestFlight and Play internal
testing, or Xcode archive and a Gradle bundle. The user owns the developer
accounts (Apple $99 a year, Google $25 once). Beta first, then review with the
listing above.
