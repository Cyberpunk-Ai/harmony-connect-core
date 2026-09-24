# Round 2 — Teams, admin, persistence, storage, payments, launch readiness

Already done last round: port, database set up on your Supabase, admin email rule, live Spaces audio, real developer API keys and webhooks, poll and location fields, About/Terms/Privacy/Contact/Guidelines pages, new not-found page, follower counts removed from Explore.

## 1. Team workspaces (complete)
- Owner invites by username or email and picks a role (admin, editor, viewer).
- The invited person gets a notification with **Accept** and **Decline** buttons; the owner is notified of the answer.
- Members list, change role, remove member, leave team, seat limits by plan.
- Roles are enforced in the database, not just hidden in the UI.

## 2. Everything saves and syncs between users
- Audit and fix likes, reposts, comments, bookmarks, follows, messages, message reactions, story views, Space chat and tips so each one is saved and shows up live for the other person.
- **Follows resetting on refresh:** the follow buttons read their saved state, so the likely cause is that they load before sign-in finishes and see a guest. I'll confirm this first, then make them reload once sign-in is ready. Same fix for the right-side panel and profile pages.
- Notifications for likes, comments, follows, mentions, messages, tips and team invites, delivered live with an unread badge.

## 3. Storage that works with Cloudflare R2
- Your Supabase project has **no media storage bucket yet**, so uploads currently fail. I'll create it.
- Uploads go through one storage layer with two backends: Supabase Storage (default) or Cloudflare R2 (S3-compatible), picked with an environment setting. Signed uploads, size and type checks, and one media address format so switching providers doesn't break old links.

## 4. Admin, production-ready
- Real data on every tab: overview stats, users (suspend, verify, change role), content removal, reports queue, payouts approval, audit log, system settings.
- Every admin action checked on the server and written to the audit log.

## 5. AI
- AI draft, summaries and suggestions work with any OpenAI-compatible provider from settings, with clear loading states, retries, and friendly errors when no key is set.

## 6. Mobile story creator
- On small screens the live preview becomes a toggle ("Edit" / "Preview") instead of covering the editing controls.

## 7. Payments
- Amounts under $1 allowed wherever the provider's minimum permits (Paystack about KES 50 / NGN 50; Stripe about $0.50). Below that, the button explains the minimum.
- Changing between paid plans is prorated: you're only charged the difference, and the unused time on your old plan is credited, so nobody pays for two plans.
- Stripe checkout and webhooks built alongside the existing Paystack flow.

## 8. Launch checks
- Security scan and fixes, full sign-up-to-payment browser test with two test accounts (to prove likes, follows and messages sync between people), mobile pass, speed pass.

## What I'll need from you
- **Paystack secret key**, **Stripe secret key and webhook secret**. I'll ask for these through a secure form.
- **Cloudflare R2** (only if you want R2 now): account ID, access key, secret key, bucket name, public URL.
- **Google sign-in:** turn on the Google provider in your Supabase dashboard (Authentication, Providers).

## Technical details
- New migration: `workspace_invites` (token, status, expiry), `notifications.action` jsonb + `link`, triggers for invite/accept/decline notifications; `workspace_role()` security-definer helper used in RLS for workspace tables.
- Follow fix: gate hydration on `supabase.auth.onAuthStateChange` / profile-ready event and invalidate the follow-state cache on SIGNED_IN.
- Realtime: add persisted tables to the `supabase_realtime` publication; one subscription per table in a shared hook.
- Storage: `src/lib/storage.server.ts` with `SupabaseStorage` and `R2Storage` (AWS SigV4 via `aws4fetch`, Worker-compatible), chosen by `STORAGE_PROVIDER`; presigned PUT from a server function; the media proxy route reads from the active provider.
- Payments: provider minimums table in config; proration computed server-side from the remaining period of the current plan.
- Migrations continue to live in `database/migrations` and run with `bun run db:migrate`.
