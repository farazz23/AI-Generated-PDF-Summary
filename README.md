<div align="center">

# 📄 AI-Powered PDF Summarizer

### Transform any PDF into a clear, engaging, emoji-rich summary in seconds — powered by OpenAI & Google Gemini.

[![Next.js](https://img.shields.io/badge/Next.js-15-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-v4-38bdf8?style=for-the-badge&logo=tailwindcss)](https://tailwindcss.com/)
[![Neon DB](https://img.shields.io/badge/Neon-Serverless_PG-00e599?style=for-the-badge&logo=postgresql)](https://neon.tech/)
[![Clerk](https://img.shields.io/badge/Clerk-Auth-6c47ff?style=for-the-badge&logo=clerk)](https://clerk.com/)
[![Stripe](https://img.shields.io/badge/Stripe-Payments-008cdd?style=for-the-badge&logo=stripe)](https://stripe.com/)

</div>

---

## 🧭 Table of Contents

- [✨ The Story](#-the-story)
- [🔭 Feature Overview](#-feature-overview)
- [🏗️ System Architecture](#️-system-architecture)
- [🌊 Data Flow Walkthrough](#-data-flow-walkthrough)
- [🗄️ Database Schema](#️-database-schema)
- [🧱 Project Structure](#-project-structure)
- [🧰 Tech Stack](#-tech-stack)
- [⚙️ Environment Variables](#️-environment-variables)
- [🚀 Getting Started](#-getting-started)
- [💳 Stripe & Payments Setup](#-stripe--payments-setup)
- [🤖 AI Provider Strategy](#-ai-provider-strategy)
- [🛡️ Authentication & Authorization](#️-authentication--authorization)
- [📦 Deployment](#-deployment)
- [🤝 Contributing](#-contributing)

---

## ✨ The Story

> *"I just got a 200-page research paper and my deadline is in 3 hours."*

We have all been there. The project was born out of a simple frustration: **long documents take too long to read**, and existing tools give you a wall of bullet points that are not much better than the original.

**AI PDF Summarizer** is a full-stack SaaS application that lets users:

1. **Upload** any PDF
2. **Extract** its text content automatically
3. **Summarize** it via AI into a structured, emoji-rich, human-readable format
4. **Save & revisit** summaries from a personal dashboard
5. **Subscribe** to a plan for extended usage

The AI does not just compress — it *transforms*. It writes like a social media content expert: structured sections, emojis that match context, key highlights, pro tips, and a bottom-line takeaway. Your brain actually *wants* to read it.

---

## 🔭 Feature Overview

| Feature | Description |
|---|---|
| 📤 **PDF Upload** | Upload PDFs via UploadThing CDN — fast, secure, direct browser-to-CDN |
| 🤖 **AI Summarization** | OpenAI GPT-3.5-Turbo as primary, Google Gemini 1.5 Flash as automatic fallback |
| 📊 **Personal Dashboard** | View, browse, and manage all your saved summaries |
| 📖 **Summary Detail Page** | Rich markdown render with word count and original file link |
| 🗑️ **Delete Summaries** | Remove any summary with one click (owner-only action) |
| 💳 **Stripe Subscriptions** | Basic ($9/mo · 5 PDFs) and Pro ($19/mo · Unlimited) plans |
| 🔐 **Clerk Authentication** | Social logins, email/password, JWT-protected routes |
| 🚦 **Upload Limits** | Enforced per-plan PDF limits, checked server-side before processing |
| 🌙 **Dark / Light Mode** | Fully themed via `next-themes` |
| ✨ **Animations** | Smooth entrance animations powered by Framer Motion |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT BROWSER                             │
│                                                                     │
│   Landing Page ──► Sign Up/In ──► Dashboard ──► Upload ──► Summary │
└──────────────────────────────┬──────────────────────────────────────┘
                               │  HTTPS
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     NEXT.JS 15 APP (Vercel)                         │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │  App Router  │  │  Middleware  │  │    Server Actions         │  │
│  │  (RSC + SSR) │  │  (Clerk JWT) │  │  upload-action.ts         │  │
│  └──────────────┘  └──────────────┘  │  summary-action.ts        │  │
│                                      └──────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                       API Routes                             │  │
│  │   /api/payments  (Stripe Webhook)                            │  │
│  │   /api/uploadthing (UploadThing CDN handler)                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐
│   Neon Postgres │  │   UploadThing   │  │  Clerk Auth      │
│  (Serverless)   │  │   (CDN/Storage) │  │  (JWT + Webhooks)│
│                 │  │                 │  │                  │
│  users          │  │  PDF Files      │  │  User Identity   │
│  pdf_summaries  │  │  (hosted URLs)  │  │  Sessions        │
│  payment        │  └─────────────────┘  └──────────────────┘
└─────────────────┘
          │
          │  reads file URL → extracts text → sends to AI
          ▼
┌─────────────────────────────────────┐
│          AI LAYER (lib/)            │
│                                     │
│  langchain.ts ──► PDFLoader         │
│  (fetches PDF, extracts text)       │
│              │                      │
│              ▼                      │
│   openai.ts (Primary)               │
│   GPT-3.5-Turbo ── rate limit? ───► │
│                                     │
│   geminiai.ts (Fallback)            │
│   Gemini 1.5 Flash                  │
└─────────────────────────────────────┘
          │
          ▼
┌─────────────────┐
│  Stripe Webhook │
│  /api/payments  │
│                 │
│  checkout.      │
│  session.       │
│  completed ───► │  update users table
│                 │  create payment record
│  subscription.  │
│  deleted   ───► │  set status = cancelled
└─────────────────┘
```

---

## 🌊 Data Flow Walkthrough

Here is exactly what happens from the moment a user hits **"Upload"** to seeing their summary.

```
USER CLICKS UPLOAD
      │
      ▼
1. 📤 UploadThing  (browser → CDN)
      PDF file is uploaded directly from the browser to UploadThing CDN.
      A public file URL is returned (e.g. https://utfs.io/f/xyz.pdf)
      │
      ▼
2. 🔒 Server Action: generatePDFText()   [action/upload-action.ts]
      Receives the file URL.
      Calls fetchAndExtractPdfText(fileUrl) via LangChain PDFLoader.
      LangChain fetches the remote PDF, parses it page by page,
      and joins all page content into a single text string.
      │
      ▼
3. 🤖 Server Action: generatePDFSummary()   [action/upload-action.ts]
      │
      ├── PRIMARY: generateSummaryFromOpenAI(pdfText)   [lib/openai.ts]
      │     Sends text + SUMMARY_SYSTEM_PROMPT to GPT-3.5-Turbo.
      │     Returns structured markdown with emojis.
      │
      └── FALLBACK (on OpenAI 429 rate limit): generateSummaryFromGeminiAI(pdfText)   [lib/geminiai.ts]
            Sends same text to Gemini 1.5 Flash.
            Same prompt template, consistent structured output.
      │
      ▼
4. 💾 Server Action: storePDFSummaryAction()   [action/upload-action.ts]
      Verifies user identity via Clerk auth().
      Inserts a new row in pdf_summaries:
        { user_id, original_file_url, summary_text, title, file_name }
      Revalidates the /summaries/[id] cache path.
      │
      ▼
5. 🚀 Redirect → /summaries/[id]
      Summary detail page renders the saved markdown.
      Shows word count, creation date, link to original file.
      │
      ▼
6. 📊 Dashboard  (/dashboard)
      getSummaries(userId) → lists all past summaries.
      SummaryCard components with title, date, and delete button.
```

### 💳 Payment Flow

```
USER CLICKS "Buy Plan"
      │
      ▼
1. Stripe Checkout (Stripe-hosted page)
      User enters card details on Stripe's secure checkout page.
      │
      ▼
2. Stripe fires: checkout.session.completed  →  POST /api/payments
      │
      ▼
3. handleCheckoutSessionCompleted()   [lib/payments.ts]
      Retrieves session + line_items from Stripe API.
      createOrUpdateUser() → upserts row in users table.
      createPayment()      → inserts row in payment table.
      │
      ▼
4. users.status = 'active', priceId stored.
      hasReachedUploadLimits() now checks against their plan tier.
      Basic: 5 PDFs/month   |   Pro: 1000 PDFs/month

USER CANCELS SUBSCRIPTION
      │
      ▼
5. Stripe fires: customer.subscription.deleted  →  POST /api/payments
      │
      ▼
6. handleSubscriptionDeleted()
      Sets users.status = 'cancelled' in the database.
```

---

## 🗄️ Database Schema

> **Database:** Neon Serverless PostgreSQL
> **Driver:** `@neondatabase/serverless` (HTTP-based, edge-compatible, no persistent connections)

```sql
-- Users (created/updated via Stripe webhooks)
CREATE TABLE users (
  id          UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(250) UNIQUE NOT NULL,
  fullname    VARCHAR(250) NOT NULL,
  customer_id VARCHAR(250) UNIQUE NOT NULL,   -- Stripe Customer ID
  price_id    VARCHAR(250) NOT NULL,           -- Stripe Price ID (plan tier)
  status      VARCHAR(50)  DEFAULT 'inactive', -- 'active' | 'inactive' | 'cancelled'
  created_at  TIMESTAMPTZ  DEFAULT NOW(),
  updated_at  TIMESTAMPTZ  DEFAULT NOW()
);

-- PDF Summaries (core content table)
CREATE TABLE pdf_summaries (
  id                UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           VARCHAR(255) NOT NULL,            -- Clerk User ID
  original_file_url TEXT         NOT NULL,            -- UploadThing CDN URL
  summary_text      TEXT         NOT NULL,            -- AI-generated markdown
  status            VARCHAR(255) DEFAULT 'completed',
  title             TEXT,                              -- Formatted from filename
  file_name         TEXT,
  created_at        TIMESTAMPTZ  DEFAULT NOW(),
  updated_at        TIMESTAMPTZ  DEFAULT NOW()
);

-- Payments (audit trail of Stripe transactions)
CREATE TABLE payment (
  id                UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  amount            INTEGER      NOT NULL,              -- Amount in cents
  status            VARCHAR(255) NOT NULL,              -- Stripe payment status
  stripe_payment_id VARCHAR(255) UNIQUE NOT NULL,       -- Stripe session ID
  price_id          VARCHAR(255) NOT NULL,
  user_email        VARCHAR(255) NOT NULL REFERENCES users(email),
  created_at        TIMESTAMPTZ  DEFAULT NOW(),
  updated_at        TIMESTAMPTZ  DEFAULT NOW()
);
```

> `updated_at` is auto-maintained by a shared PostgreSQL trigger (`update_updated_at_column`) applied to all three tables. See [`schema.sql`](./schema.sql) for the full trigger definitions.

---

## 🧱 Project Structure

```
AI-Generated-PDF-Summary/
│
├── app/                           # Next.js 15 App Router
│   ├── page.tsx                   # Public landing page
│   ├── layout.tsx                 # Root layout (fonts, theme, Clerk provider)
│   ├── globals.css                # Global styles + Tailwind v4 config
│   │
│   ├── (logged-in)/               # Route group — all auth-protected pages
│   │   ├── layout.tsx             # Shared layout for authenticated routes
│   │   ├── dashboard/             # /dashboard — summary list + stats
│   │   ├── upload/                # /upload — PDF upload form
│   │   └── summaries/[id]/        # /summaries/:id — summary detail view
│   │
│   ├── sign-in/                   # Clerk sign-in page
│   ├── sign-up/                   # Clerk sign-up page
│   │
│   └── api/
│       ├── payments/route.ts      # Stripe webhook handler
│       └── uploadthing/           # UploadThing file API route
│
├── action/                        # Next.js Server Actions
│   ├── upload-action.ts           # generatePDFText, generatePDFSummary, storePDFSummaryAction
│   └── summary-action.ts          # deleteSummaryAction
│
├── components/
│   ├── common/                    # Shared UI (BgGradient, Navbar, Footer, etc.)
│   ├── home/                      # Landing page sections
│   │   ├── HeroSection.tsx
│   │   ├── DemoSection.tsx
│   │   ├── HowItWorks.tsx
│   │   ├── PricingSection.tsx
│   │   └── CTASection.tsx
│   ├── dashboard/                 # Dashboard UI components
│   │   ├── SummaryCard.tsx
│   │   ├── DeleteButton.tsx
│   │   └── EmptySummaryState.tsx
│   ├── summaries/                 # Summary detail viewer components
│   └── upload/                    # Upload form + progress indicator
│
├── lib/                           # Core integrations & server-only utilities
│   ├── dbConnection.ts            # Neon Postgres connection singleton
│   ├── langchain.ts               # PDF text extraction via LangChain PDFLoader
│   ├── openai.ts                  # OpenAI GPT-3.5 summary generation
│   ├── geminiai.ts                # Google Gemini 1.5 Flash fallback
│   ├── payments.ts                # Stripe checkout & subscription event handlers
│   ├── summaries.ts               # DB queries: get summaries, get by ID, upload count
│   └── users.ts                   # DB queries: plan lookup, upload limit check
│
├── utils/
│   ├── constants.ts               # Pricing plans, Framer Motion animation variants
│   ├── prompts.ts                 # SUMMARY_SYSTEM_PROMPT (the AI instruction template)
│   ├── summary-helper.ts          # Markdown/emoji parsing utilities for the viewer
│   ├── formatFileNameAsTitle.ts   # Converts filenames to human-readable titles
│   ├── uploadthing.ts             # UploadThing client helper
│   └── helpers.ts                 # isDev environment flag
│
├── schema.sql                     # Full database schema — run once on Neon
├── middleware.ts                  # Clerk route protection middleware
├── next.config.ts
├── tailwind.config.ts
└── vercel.json
```

---

## 🧰 Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Framework** | [Next.js 15](https://nextjs.org/) + Turbopack | App Router, RSC, Server Actions — full-stack in one repo |
| **Language** | TypeScript 5 | Type safety end-to-end across client and server |
| **Styling** | Tailwind CSS v4 | Utility-first, design tokens, dark mode support |
| **Animations** | Framer Motion (`motion`) | Spring physics, stagger animations, micro-interactions |
| **Auth** | [Clerk](https://clerk.com/) | Drop-in auth with JWT middleware, social logins |
| **Database** | [Neon](https://neon.tech/) Serverless PostgreSQL | Edge-compatible, no persistent connections needed |
| **File Storage** | [UploadThing](https://uploadthing.com/) | Type-safe file uploads, CDN-hosted files |
| **AI (Primary)** | [OpenAI](https://openai.com/) GPT-3.5-Turbo | Fast, cost-effective summarization |
| **AI (Fallback)** | [Google Gemini](https://ai.google.dev/) 1.5 Flash | Automatic failover when OpenAI rate-limits |
| **PDF Parsing** | [LangChain](https://js.langchain.com/) PDFLoader | Remote PDF fetch + page-by-page text extraction |
| **Payments** | [Stripe](https://stripe.com/) + Webhooks | Subscription billing, checkout sessions |
| **Validation** | [Zod](https://zod.dev/) | Runtime schema validation |
| **UI Components** | [Radix UI](https://www.radix-ui.com/) | Accessible, unstyled headless primitives |
| **Icons** | [Lucide React](https://lucide.dev/) | Consistent, tree-shakeable icon set |
| **Toasts** | react-hot-toast + Sonner | User feedback on actions |
| **Deployment** | [Vercel](https://vercel.com/) | Zero-config Next.js deployment with edge network |

---

## ⚙️ Environment Variables

Create a `.env.local` file in the root of the project. **Never commit this file.**

```env
# ─────────────────────────────────────────────
# CLERK — Authentication
# https://dashboard.clerk.com → API Keys
# ─────────────────────────────────────────────
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# ─────────────────────────────────────────────
# NEON — Serverless PostgreSQL
# https://console.neon.tech → Connection String
# ─────────────────────────────────────────────
DATABASE_URL=postgresql://user:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require

# ─────────────────────────────────────────────
# OPENAI — Primary AI provider
# https://platform.openai.com/api-keys
# ─────────────────────────────────────────────
OPENAI_API_KEY=sk-...

# ─────────────────────────────────────────────
# GOOGLE GEMINI — Fallback AI provider
# https://aistudio.google.com/app/apikey
# ─────────────────────────────────────────────
GEMINIAI_API_KEY=AIza...

# ─────────────────────────────────────────────
# UPLOADTHING — File storage & CDN
# https://uploadthing.com/dashboard → API Keys
# ─────────────────────────────────────────────
UPLOADTHING_TOKEN=...

# ─────────────────────────────────────────────
# STRIPE — Payments & subscriptions
# https://dashboard.stripe.com/apikeys
# ─────────────────────────────────────────────
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** >= 20 or **Bun** (recommended)
- A [Neon](https://neon.tech) account (free tier works)
- A [Clerk](https://clerk.com) account (free tier works)
- An [UploadThing](https://uploadthing.com) account
- An [OpenAI](https://platform.openai.com) API key
- A [Google AI Studio](https://aistudio.google.com) API key
- A [Stripe](https://stripe.com) account (test mode)

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/AI-Generated-PDF-Summary.git
cd AI-Generated-PDF-Summary
```

### 2. Install dependencies

```bash
bun install
# or
npm install
```

### 3. Set up environment variables

```bash
cp .env.example .env.local
# Edit .env.local with your actual credentials
```

### 4. Initialize the database

Run the schema against your Neon database (one-time setup).

Use the Neon SQL editor in your dashboard, or run via psql:

```bash
psql $DATABASE_URL -f schema.sql
```

### 5. Start the development server

```bash
bun dev
# or
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 💳 Stripe & Payments Setup

### Forwarding Webhooks Locally

Stripe needs to reach your local server to test the payment flow.

**Option 1: Stripe CLI (recommended)**

```bash
# Install: https://stripe.com/docs/stripe-cli
stripe listen --forward-to localhost:3000/api/payments
```

Copy the printed `whsec_...` secret into `.env.local` as `STRIPE_WEBHOOK_SECRET`.

**Option 2: localtunnel (already in package.json)**

```bash
bun run tunnel
# Runs: lt --port 3000
# Copy the public URL → set as Stripe webhook endpoint in dashboard
```

### Creating Products in Stripe

In your [Stripe Dashboard](https://dashboard.stripe.com/test/products):

1. Create **Basic** product — $9/month → copy `price_id`
2. Create **Pro** product — $19/month → copy `price_id`
3. Update `utils/constants.ts` with the correct `priceId` and `paymentLink` for each plan.

---

## 🤖 AI Provider Strategy

The app implements a **primary + automatic fallback** pattern for resilience:

```
generatePDFSummary()
       │
       ├── 1. Try OpenAI GPT-3.5-Turbo
       │        ✅ Success  →  return summary
       │        ❌ Rate Limited (HTTP 429)  →
       │
       └── 2. Try Google Gemini 1.5 Flash
                ✅ Success  →  return summary
                ❌ Fails    →  throw error to caller
```

**Why this design?**
- OpenAI `gpt-3.5-turbo` is fast and cost-effective for most workloads
- Gemini Flash is free-tier friendly and absorbs rate-limit spikes gracefully
- Both providers receive the **exact same** `SUMMARY_SYSTEM_PROMPT` — outputs are structurally identical regardless of which provider runs
- The caller (upload form) never needs to know which AI was used

**The Prompt** (`utils/prompts.ts`) instructs the AI to act as a "social media content expert," outputting structured markdown sections: Title, Key Highlights, Why It Matters, Main Points, Pro Tips, Key Terms, and Bottom Line — all enriched with contextual emojis.

---

## 🛡️ Authentication & Authorization

Authentication is handled entirely by **Clerk**, with server-side enforcement via Next.js Middleware.

### Protected Routes

```typescript
// middleware.ts
const isProtectedRoutes = createRouteMatcher([
  '/dashboard(.*)',
  '/summaries(.*)',
  '/upload(.*)',
])
// Unauthenticated requests → auto-redirect to /sign-in
```

### Owner-Only Operations

Delete actions verify the authenticated user owns the resource at the database level:

```sql
DELETE FROM pdf_summaries
WHERE id = $summaryId AND user_id = $clerkUserId
-- Even if someone guesses a UUID, they cannot delete another user's summary
```

### Upload Limit Enforcement

Before any summarization begins, the server checks plan limits:

```typescript
// lib/users.ts
const { hasReachedLimits, uploadLimit } = await hasReachedUploadLimits(userId);
// Basic plan: 5 PDF limit   |   Pro plan: 1000 PDF limit
// Checked entirely server-side — cannot be bypassed from the client
```

---

## 📦 Deployment

The app is pre-configured for **Vercel** deployment via `vercel.json`.

### Deploy to Vercel

Connect your GitHub repository to Vercel for automatic deployments, or use the CLI:

```bash
npm i -g vercel
vercel --prod
```

### Required Vercel Environment Variables

Add all variables from the [Environment Variables](#️-environment-variables) section under **Vercel Dashboard → Settings → Environment Variables**.

### Stripe Webhook in Production

1. Go to [Stripe Dashboard → Webhooks](https://dashboard.stripe.com/webhooks)
2. Add endpoint: `https://your-app.vercel.app/api/payments`
3. Subscribe to events:
   - `checkout.session.completed`
   - `customer.subscription.deleted`
4. Copy the webhook signing secret → add to Vercel as `STRIPE_WEBHOOK_SECRET`

---

## 🤝 Contributing

Contributions are welcome! Here is how to get started:

1. **Fork** the repository
2. Create a feature branch: `git checkout -b feat/your-feature-name`
3. Make your changes following the code style below
4. Test your changes locally with `bun dev`
5. Commit: `git commit -m "feat: add export to PDF feature"`
6. Push and open a **Pull Request**

### Code Style Guidelines

- All new files must be **TypeScript** with explicit types — no `any` unless absolutely necessary
- **Server-only code** (DB queries, AI calls, Stripe) belongs in `lib/` or `action/` — never import these in client components
- **UI components** go in `components/` under the appropriate subdirectory
- Use **Server Actions** for all data mutations — avoid creating new API routes unless necessary (e.g., webhooks)
- Never expose API keys or secrets in client-side code
- Keep DB queries in `lib/summaries.ts` or `lib/users.ts` — do not write inline SQL in components or actions

---

<div align="center">

Built with love using Next.js 15, OpenAI, Google Gemini, Clerk, Neon, Stripe & UploadThing.

</div>
