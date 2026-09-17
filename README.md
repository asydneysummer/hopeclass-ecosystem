# HopeClass — экосистема для репетитора

**HopeClass** is a multi-tenant EdTech platform for tutors: student/parent/teacher cabinet (PWA), admin control plane, public visiting cards, real-time whiteboard, and desktop client. Modules share one product story but live in separate private repos for clarity and deploy boundaries.

> **Student testing (quiz)** runs on [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) — assigned from the HopeClass tutor cabinet via SSO; results return through webhook. Source: [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)*.

---

## Modules

| Module | Repository | Production |
|--------|------------|------------|
| **Core** — multi-tenant API, tutor PWA, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(private — request access)* | `*.hopeclass.ru`, `repetitor.hopeclass.ru`, `admin.hopeclass.ru` |
| **Visiting cards** — public link-in-bio pages | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(private — request access)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Online whiteboard** — lobby + Excalidraw editor | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(private — request access)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Desktop** — Electron client, offline sync | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(private — request access)* | Desktop releases |

---

## Architecture (high level)

```
┌─────────────────────────────────────────────────────────────┐
│ HopeClass ecosystem                                         │
│  tutor-app ←→ server (tenant DB per teacher) ←→ control-plane │
│  card-app (public) · desk-app (collab) · hopeclass-desktop   │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP + shared secrets
                            ▼
┌─────────────────────────────────────────────────────────────┐
│ Metod Orbita (separate repos)                               │
│  metod-orbita-shop: store + quiz module                     │
│  metod-orbita-courses: LMS (SSO via shop JWT)               │
└─────────────────────────────────────────────────────────────┘
```

---

## Cross-product integrations

| From | To | Mechanism |
|------|-----|-----------|
| HopeClass tutor-app | metod-orbita quiz | Test assignment, SSO launch |
| quiz-module | HopeClass API | Webhook with per-tenant key → quiz reports + push |
| metod-orbita-shop | HopeClass | Trial schools, ensure-user, visiting-card editor API |
| hopeclass-cards | metod-orbita-shop | Card theme presets sync |
| hopeclass-desk | HopeClass server | JWT auth, `/api/desk/*` |

---

## Tech stack (core)

- **Backend:** Node.js, Express, Prisma, PostgreSQL (per-tenant isolation via proxy)
- **Frontend:** React, Vite, Zustand, mobile-first PWA
- **Admin:** control-plane (tenant registry, provisioning, support)
- **Auth:** JWT bearer; multi-tenant routing by Host / `x-tenant-slug`

---

## Local development (core)

```bash
# API (multi-tenant)
cd server && MULTI_TENANT=true CONTROL_DATABASE_URL=... npm run dev

# Tutor PWA
cd tutor-app && npm run dev

# Control plane
cd control-plane && npm run dev
```

See each repo's README for env vars and deploy scripts.

---

## Related repos (Metod Orbita)

| Repo | Scope |
|------|-------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)* | Store, payments (Prodamus), quiz SPA + API, HopeClass bridge |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(private — request access)* | Course platform; SSO via shop JWT |

---

*Portfolio index · private source repos · author: Michael / asydneysummer*
