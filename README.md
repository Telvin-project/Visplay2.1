# VisPlay — Next.js + Supabase + Vercel

A clean, independent VisPlay codebase. The storefront, authentication screens, viewer dashboard, creator application, admin overview, Paystack test-mode deposit routes, wallet ledger schema and purchase RPCs are included.

## Important scope note

This is a deployable foundation for testing, not yet a complete production streaming service. The catalogue currently uses fictional demo titles and remote poster imagery; it does not include licensed full-length movies. The preview countdown is a UI demonstration, **not secure playback enforcement**. Protected video playback requires uploaded media plus a streaming/storage provider that can issue expiring playback tokens or signed URLs. Live broadcasts, ad delivery, rewards rules, monthly transfers, bundle purchase UI and full admin moderation are not integrated yet. Do not take real-money payments or launch public paid playback until these are completed and tested.

## Requirements

- Node.js 20+
- A Supabase project
- A Paystack test account
- GitHub and Vercel accounts

## 1. Set up Supabase

1. Create a Supabase project.
2. Open **SQL Editor** and run all of `supabase/schema.sql`.
3. In **Authentication → URL Configuration**, add your local URL `http://localhost:3000` and your Vercel domain to allowed redirect URLs.
4. Copy the project URL and anon key from **Project Settings → API**.
5. Copy the `service_role` key from the same page. Keep it server-only; never put it in a `NEXT_PUBLIC_` variable.
6. In Authentication settings, decide whether email confirmation should be required. For quick local tests, use your own test account and the normal Supabase confirmation settings.

## 2. Configure environment variables

Copy `.env.example` to `.env.local` and fill in:

```env
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY=YOUR_SUPABASE_SERVICE_ROLE_KEY
PAYSTACK_SECRET_KEY=sk_test_your_test_secret
NEXT_PUBLIC_PAYSTACK_PUBLIC_KEY=pk_test_your_test_public_key
NEXT_PUBLIC_SITE_URL=http://localhost:3000
VISPLAY_OWNER_EMAIL=the-email-you-use-to-sign-up
```

The email in `VISPLAY_OWNER_EMAIL` must match the email of your authenticated test account exactly. This does not use or store your password. After the first login, visit `/admin`; the server checks the authenticated email and provisions the owner role. Do not set this to an email you do not control. Keep all secret keys out of GitHub.

## 3. Run locally

```bash
npm install
npm run dev
```

Open `http://localhost:3000`. Create an account, sign in, open the dashboard and try Paystack test deposits. A wallet settlement requires the SQL function in `supabase/schema.sql` to have been installed successfully.

## 4. Paystack webhook

Set the webhook in the Paystack dashboard to:

`https://YOUR-VERCEL-DOMAIN/api/paystack/webhook`

Use test mode first. The server verifies the `x-paystack-signature`, verifies the transaction with Paystack, checks currency and amount, and calls an idempotent database function to credit the wallet once. Test both the browser callback and repeated webhook deliveries before trusting the balance.

## 5. Push to GitHub

Create a **private** GitHub repository. From this folder run:

```bash
git init
git add .
git commit -m "Initial VisPlay website"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-PRIVATE-REPO.git
git push -u origin main
```

Do not commit `.env.local` or any secret keys.

## 6. Deploy to Vercel

1. Import the private GitHub repository into Vercel.
2. Add every variable from `.env.example` under **Project Settings → Environment Variables** for Preview and Production as appropriate.
3. Deploy. If variables change, redeploy so server functions receive the updated configuration.
4. Update `NEXT_PUBLIC_SITE_URL` to the deployed domain and redeploy.
5. Add the deployed domain to Supabase Auth's allowed redirect URLs.
6. Update the Paystack webhook to the deployed domain.

## Routes

- `/` — storefront
- `/signup`, `/login` — authentication
- `/dashboard` — viewer wallet and activity
- `/creator` — creator application
- `/admin` — owner dashboard (owner email must be configured)
- `/movie/last-horizon` and other demo movie routes — catalogue demo
- `/api/paystack/initialize` — authenticated deposit initialization
- `/api/paystack/verify` — authenticated transaction verification
- `/api/paystack/webhook` — signature-verified Paystack webhook

## Next production tasks

1. Integrate a real media provider (upload UI, large/resumable uploads, transcoding/adaptive streaming, signed playback access and a server-enforced five-minute preview).
2. Replace demo catalogue entries with licensed/owned movie files and real database-backed catalogue records.
3. Add server-enforced creator content management and complete admin moderation of users, movies, bundles, payouts and campaigns.
4. Implement bundle entitlements, watch history, likes/comments, notifications, reward policies and anti-abuse controls.
5. Integrate real livestream/premiere delivery, advertising delivery/analytics and a payout provider.
6. Run end-to-end tests and a security review before production use.
