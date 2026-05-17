# SentinelAuth — AI-Based Secure Login System with Behavioral Biometrics

A full-stack web application that authenticates users with **password + behavioral biometrics** (keystroke dynamics, mouse movement, typing speed). An AI/statistical model continuously verifies user identity and flags suspicious logins.

> College project implementation — built on **Lovable Cloud** (TanStack Start + PostgreSQL + Edge Functions). The architecture mirrors the Express+MongoDB spec; only the runtime differs.

---

## ✨ Features

- **Home / Register / Login / Dashboard / Admin / Model Evaluation** pages
- **Behavioral capture**: keystroke dwell & flight times, typing speed, mouse velocity & path length, click count, touch pressure
- **AI risk engine**: Z-score deviation against per-user baseline (Welford's online algorithm) + Gemini LLM reasoning
- **Confidence score** (0–100%) per login attempt
- **Suspicious-login alert banner** + auto sign-out when risk ≥ 80
- **Typing pattern graph** (Recharts) on dashboard
- **Model evaluation page**: accuracy, precision, recall, F1, confusion matrix, risk distribution
- **Admin dashboard** with full login-attempt log
- **JWT auth + bcrypt password hashing** (managed by Lovable Cloud)
- **Row-Level Security (RLS)** on every table
- **Input validation** with Zod
- **Dark cyber-themed UI**, fully responsive

---

## 🏗️ Architecture

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│  React Frontend │──────▶│ TanStack Server Fns  │──────▶│  PostgreSQL DB  │
│ (TanStack Start)│       │  + Edge Function     │       │  (RLS policies) │
└────────┬────────┘       │  (analyze-login)     │       └─────────────────┘
         │                └──────────┬───────────┘
         │ captures                  │
         ▼                           ▼
   BiometricCollector          Gemini AI (LLM)
   (keystroke + mouse)        + Z-score model
```

### ML Workflow
1. Capture 11 behavioral features at login.
2. Update running mean/stddev via **Welford's algorithm** (`biometric_baselines`).
3. Compute Z-score deviation per feature.
4. Aggregate into weighted risk score (0–100).
5. Gemini explains the deviation in plain English.
6. Decision: `<60` allow · `60–79` flag · `≥80` block + sign-out.

---

## 🗄️ Database Schema (ER)

```
auth.users (managed)
   │
   ├── profiles            (id, username, display_name)
   ├── user_roles          (user_id, role: admin|user)
   ├── biometric_baselines (user_id PK, features JSONB, sample_count)
   └── login_attempts      (id, user_id, email, success, blocked,
                            risk_score, risk_level, features JSONB,
                            ai_analysis, user_agent, created_at)
```

All tables have RLS: users see only their own rows; admins see all.

---

## 📁 Folder Structure

```
src/
  routes/          # File-based routes (TanStack)
    index.tsx      # Home
    signup.tsx     # Register + baseline capture
    login.tsx      # Login + biometric verification
    dashboard.tsx  # User dashboard + typing graph
    admin.tsx      # Admin threat monitor
    evaluation.tsx # ML accuracy / confusion matrix
  lib/
    biometrics.ts  # BiometricCollector class
    auth-context.tsx
  integrations/supabase/  # auto-generated client
supabase/
  functions/analyze-login/  # AI risk-scoring edge function
  migrations/               # SQL schema + RLS policies
```

---

## 🧪 Test Credentials

| Role  | Email                  | Password    |
|-------|------------------------|-------------|
| User  | demo@sentinel.test     | Demo1234!   |
| Admin | admin@sentinel.test    | Admin1234!  |

### Test Cases
1. **Baseline build** — log in 4–5× with a consistent rhythm. Sample count grows; risk stays low.
2. **Anomaly detection** — log in while typing extremely slow or with unusual mouse movement. Risk score spikes; suspicious banner appears.
3. **Auto-block** — sustained anomaly (risk ≥ 80) forces sign-out and creates a `blocked=true` row in `login_attempts`.
4. **Admin view** — admin sees all attempts at `/admin`.
5. **Model report** — `/evaluation` shows accuracy, precision, recall, F1 + confusion matrix.

---

## 🚀 Running Locally

This project runs on Lovable Cloud — no manual backend setup needed.

```bash
bun install
bun run dev
```

Visit `http://localhost:5173`.

For deployment, use the **Publish** button in Lovable (frontend → CDN, edge functions → auto-deployed).

---

## 🔐 Security Checklist

- ✅ Passwords hashed (bcrypt, managed by auth provider)
- ✅ JWT session tokens
- ✅ RLS on all tables
- ✅ Roles in separate `user_roles` table (prevents privilege escalation)
- ✅ Zod input validation on all forms
- ✅ HTTPS by default
- ✅ Service-role key never exposed to the client

---

## 📊 Evaluation Metrics

Computed live on `/evaluation` from each user's actual login history:
- **Accuracy** = (TP+TN) / total
- **Precision** = TP / (TP+FP)
- **Recall** = TP / (TP+FN)
- **F1** = 2·P·R / (P+R)
- Threshold: `risk_score ≥ 60` predicts anomaly; ground truth = `blocked OR !success`.

---

Built for academic demonstration of behavioral-biometric authentication.
