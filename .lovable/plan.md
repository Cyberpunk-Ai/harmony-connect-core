# Porting Spaces1 and finishing the platform

Goal: bring the `next` branch of harmony-sparkle-29 into this project, finish the
half-built features, polish the look and feel, and make it deployable anywhere —
with your own Supabase backend and every setting driven by environment values.

## Stage 1 — Port the code

- Copy the full app (pages, components, shared logic, database schema and
  migrations) into this project and install its dependencies.
- Point it at your Supabase backend and verify sign-up, sign-in and the feed
  load against it.
- Strip the Lovable-specific pieces so the app runs on any host: replace the
  Lovable auth helper, error reporter and AI gateway calls with provider-neutral
  equivalents selected by environment values, and add an `.env.example`
  documenting every one.

### Applying the database

Your Supabase service key can't create tables. The migrations ship in the repo
and you apply them with one command using your database connection string
(Supabase dashboard, Project Settings, Database, Connection string, URI):

```text
DATABASE_URL="postgresql://..." bun run db:migrate
```

I'll add that script and a README section. If you'd rather I run it, paste the
connection string and I'll apply it for you.

## Stage 2 — Finish the unfinished features

**Live audio in Spaces.** Real microphone audio, not just the UI. Small rooms
(up to 8 speakers) work peer-to-peer with no extra service, reusing the existing
one-to-one call plumbing: join/leave, mute, speaking indicator, raise hand,
host controls, presence in the database. Large rooms get the full integration
wired behind an environment switch so it activates the moment you add a hosted
audio provider's credentials; until then large rooms fall back to the
peer-to-peer path with a clear capacity notice.

**Developer API.** Today keys are cosmetic. Real backend: keys generated once,
stored only as a salted hash, shown once at creation, revocable, with per-key
rate limits and usage counters. Authenticated public API endpoints that accept
those keys. Webhooks actually delivered with a signed payload, retries and a
delivery log you can inspect.

**Team workspaces.** Roles (owner, admin, editor, viewer) enforced in the
database, invitations, member management, role-aware UI.

**Posting: polls and location.** A labelled poll question field, clear messages
when a required field is empty, duplicate/blank option handling, and a location
field you can type into with live suggestions instead of a fixed list.

**Settings.** Complete every panel: account, profile, privacy, notifications,
appearance, security/sessions, blocked accounts, data export, account deletion.

**Admin.** Polish all tabs, and grant full admin to
`n.e.x.t.g.e.n.t.e.c.h.1.67@gmail.com` automatically on first sign-in via a
database rule (roles stay in their own table, never on the profile).

**Messaging.** Verify and polish the previous round of fixes, add real-time
delivery/read state and typing indicators where missing.

## Stage 3 — Missing pages and polish

- New pages: About, Terms, Privacy, Contact, plus a polished not-found page.
- Landing page and footer rewritten so every link goes somewhere real; footer
  navigation completed across the app.
- Explore: remove follower counts from people cards; tighten the layout.
- Feed polish: skeletons, empty states, error recovery, smoother infinite scroll.
- Consistent visual language across every page — one distinctive direction, not
  default template styling.
- Per-page titles, descriptions and social preview data on every route.

## Stage 4 — Payments

Both providers, as you asked: Paystack for African cards and mobile money,
Stripe for everyone else, chosen automatically by the buyer's region with a
manual override. The Stripe side uses Lovable's built-in payments so no Stripe
account setup is needed; Paystack uses your own Paystack secret key — I'll ask
you to save a real Paystack key, since the one referenced in your message is a
Stripe key.

## Stage 5 — Hardening and testing

- Security pass: row-level access rules on every table, privilege checks on
  admin actions, webhook signature verification, input validation, dependency
  scan, and a re-check after the changes land.
- Performance: route-level code splitting, image sizing, query caching, mobile
  layout pass down to small screens.
- Browser test run through every flow — sign-up, feed, posting with poll and
  location, follows, messages, calls, spaces audio, developer keys, workspaces,
  billing, admin — with screenshots.

## Notes

- Your backend keys are saved as project secrets; I never print them.
- Large-room audio needs a hosted audio provider's credentials to go live; the
  code and switch will be ready without them.
- Work is tracked in `roadmap.md` so progress is visible across sessions.

## Technical detail

TanStack Start v1 + React 19 + Tailwind v4, matching this project's stack, so
the port is a direct move. Data access stays split between the browser client
(RLS as the user), authenticated server functions, and the service-role client
for privileged paths only. Spaces audio uses WebRTC mesh with Supabase Realtime
as the signalling channel and a TURN server URL from env; the large-room path is
an adapter interface so a hosted SFU drops in without touching the UI. API keys
use SHA-256 with a per-key salt, prefix-indexed for lookup. Webhook delivery
runs through a queue table with exponential backoff, driven by a scheduled
public endpoint. Every remote resource (Supabase URL/keys, storage bucket, AI
gateway and model, TURN/SFU, payment keys, brand strings, limits, feature
flags) resolves from env with documented defaults.
