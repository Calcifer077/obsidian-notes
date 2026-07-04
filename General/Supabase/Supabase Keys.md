---
title: Supabase keys
created: 2026-07-04
---
## The keys involved

Supabase is mid-migration from legacy JWT keys to a new key format. Both work simultaneously right now, but legacy keys are being deprecated by end of 2026.

|Purpose|Legacy (JWT-based)|New format|
|---|---|---|
|Client-side / browser-safe|`anon` key (starts `eyJ...`)|**publishable** key (`sb_publishable_xxx`)|
|Server-side / full access|`service_role` key (starts `eyJ...`)|**secret** key (`sb_secret_xxx`)|

There's also the **JWT secret** (legacy) or **JWT signing keys** (new, asymmetric EC keys) — not an API key you use in client code at all, just the thing Supabase Auth uses to sign/verify user session tokens. You'd only touch this if you're minting custom JWTs yourself.

### publishable / anon key

- only grants whatever your Row Level Security policies allow
- the publishable key is "safe to expose online: web page, mobile or desktop app, GitHub actions, CLIs, source code"
- Used for unauthenticated requests, and combined with a user's session JWT for authenticated requests.

### secret / service_role key

- bypasses every RLS policy via the Postgres BYPASSRLS attribute and must never appear in a browser, mobile, or desktop bundle
- meant to be used only in secure, developer-controlled components of your application, such as: Servers that implement prior authorization themselves, such as Edge Functions, microservices, traditional or specialized web servers. Periodic jobs, queue processors, topic subscribers. Admin and back-office tools, with prior authorization checks only. Data processing pipelines, such as for analytics, reports, backups, or database synchronization.
- The new secret key actually enforces this: you cannot use a secret key in the browser (matches on the User-Agent header) and it will always reply with HTTP 401 Unauthorized — a real guardrail the old `service_role` JWT never had.

## "Exposed on the browser" — what it actually means

It means the key is shipped in code that runs on the user's device — your webpage's JS bundle, a mobile app binary, a desktop app. Anyone can open devtools, inspect network requests, or decompile the app and read it out. So "safe to expose" keys are ones where leaking them doesn't matter because they carry no inherent privilege on their own — privilege comes entirely from RLS policies on your tables. The secret/service_role key is _never_ safe here because it bypasses RLS entirely — whoever has it can read/write/delete anything.

## Which keys pass RLS vs bypass it

- **publishable/anon** (and any authenticated user JWT built on top of it) → **RLS is enforced**. Postgres roles `anon`/`authenticated` don't have `BYPASSRLS`.
- **secret/service_role** → **RLS is completely bypassed**, full table access regardless of policies.

## Common cases

1. **Frontend (React/Vue/mobile app) talking directly to Supabase** → publishable/anon key + RLS policies doing the access control. This is the intended pattern (Supabase's whole pitch is "skip a custom backend, let Postgres RLS be your authorization layer").
2. **Your own backend server talking to Supabase** → depends:
    - If your backend should act _on behalf of_ a logged-in user and respect that user's permissions → use publishable key + forward/set the user's JWT so RLS still applies to that user's row-level policies.
    - If your backend needs admin-level access (bypass RLS, run privileged operations, cron jobs, webhooks, admin tools) → use the **secret** key, kept only in server environment variables, never bundled into anything shipped to a client.
3. **Edge Functions** → can use either, but if privileged, use secret key server-side only (and it works there since Edge Functions are a trusted server context, not a browser — the browser User-Agent block doesn't apply).

So in short if the your app is acting on behalf of the user (the user is logged in), you would use anon key or publishable keys. But if your app is acting as a standalone backend server (it doesn't care about the user, you have your own security framework), you would secret key.