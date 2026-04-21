# deepdown — cloud bot handoff

This zip is a complete, **type-clean and production-buildable** Next.js 14 + Supabase codebase for deepdown.io. Verified by passing `tsc --noEmit` with zero errors and `next build` succeeding with all 9 routes generated.

## First-time setup (for your cloud bot)

### 1. Unzip and install

```bash
unzip deepdown.zip
cd deepdown
npm install
```

### 2. Create Supabase project

1. Go to [supabase.com](https://supabase.com) and create a new project
2. In the Supabase SQL Editor, paste and run the contents of `supabase/migrations/20260420000001_initial_schema.sql`
3. Go to Settings → API and copy the three keys you'll need

### 3. Configure environment

Copy `.env.example` to `.env.local` and fill in:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
SUPABASE_SERVICE_ROLE_KEY=eyJhbG...
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### 4. Configure auth providers (Supabase dashboard → Authentication)

**Email/magic link** — already enabled by default. No action needed.

**Google OAuth** (optional but recommended):
1. In Google Cloud Console, create OAuth 2.0 credentials (Web application)
2. Add `https://your-project.supabase.co/auth/v1/callback` to Authorized redirect URIs
3. In Supabase: Authentication → Providers → Google → paste Client ID + Secret → save
4. For localhost dev, also add `http://localhost:3000/auth/callback` to your Google OAuth redirect URIs

### 5. Run dev

```bash
npm run dev
```

Open `http://localhost:3000`. You should be able to sign up, take assessments, see your archetype, and share your profile.

## Deploying to production (Vercel)

1. Push the repo to GitHub
2. Import in Vercel
3. Add all four env vars from `.env.local` to Vercel project settings
4. Update `NEXT_PUBLIC_SITE_URL` to your production domain (e.g., `https://deepdown.io`)
5. In Supabase → Authentication → URL Configuration, add your production URL to Site URL and Redirect URLs
6. In Google Cloud Console, add the production callback URL to authorized redirect URIs
7. Point `deepdown.io` DNS at Vercel

## Project structure

```
src/
├── app/                          # Next.js App Router pages
│   ├── page.tsx                  # Landing page
│   ├── login/ signup/            # Auth pages
│   ├── auth/callback/            # OAuth / magic-link callback
│   ├── app/                      # Authenticated area (middleware-protected)
│   │   ├── page.tsx              # Dashboard
│   │   ├── profile/              # User's archetype + lenses
│   │   └── assessment/[frameworkId]/  # Assessment intro + runner + complete
│   ├── s/[token]/                # Public share view
│   └── actions/assessments.ts    # Server actions (start, save, complete, share)
├── components/                   # Button, Logo, ProgressBar, QuestionRenderer, AssessmentRunner
├── lib/
│   ├── frameworks/               # THE CORE — all 6 frameworks + synthesis
│   │   ├── types.ts              # Question + Answer + Result types
│   │   ├── archetype.ts          # MBTI-inspired (24 likert)
│   │   ├── nine.ts               # Enneagram-inspired (27 forced-choice)
│   │   ├── ikigai.ts             # Purpose (20 dual-slider)
│   │   ├── strengths.ts          # StrengthsFinder-inspired (6 ranking rounds)
│   │   ├── frequencies.ts        # 7 Frequencies (3 scenarios × 7 options)
│   │   ├── love_language.ts      # Love Languages (dual ranking)
│   │   ├── synthesis.ts          # Composes unified archetype
│   │   └── index.ts
│   ├── supabase/                 # client.ts, server.ts, service.ts, types.ts
│   └── utils.ts
└── middleware.ts                 # Auth guard for /app routes

supabase/migrations/
└── 20260420000001_initial_schema.sql   # 11 tables, RLS, triggers, seed data
```

## What's included in v1

- ✅ 6 assessments with 142 real questions, real scoring
- ✅ Synthesis engine (combines 6 results → "The [Qualifier] [Role]")
- ✅ Auth: email magic link + Google OAuth
- ✅ Dashboard, profile page, share links
- ✅ Full DB schema with RLS, triggers, seed data
- ✅ Schema supports workspaces (B2B) and relationships (couples) — UI is v2

## Intentionally deferred (for v2+)

- LLM companion (user said hold off)
- Stripe / payments (build when ready to charge)
- Couples compatibility UI (schema exists; build `/app/partners`)
- Team/workspace UI (schema exists; build workspace switcher + team pages)

## Where to edit common things

| Want to change... | Edit |
|---|---|
| Questions in a framework | `src/lib/frameworks/{framework}.ts` |
| How archetypes are composed | `src/lib/frameworks/synthesis.ts` (the qualifier/role lookups) |
| Brand colors | `tailwind.config.ts` |
| Question UI formats | `src/components/QuestionRenderer.tsx` |
| Profile hero layout | `src/app/app/profile/page.tsx` |
| Landing page copy | `src/app/page.tsx` |
| Database schema | edit the SQL migration, re-run in Supabase |

## Verified

- ✅ `npx tsc --noEmit` — 0 errors
- ✅ `npx next build` — compiled successfully, all 9 routes generated
