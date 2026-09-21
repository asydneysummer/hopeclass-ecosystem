# HopeClass — multi-tenant tutor ecosystem

**HopeClass** is a multi-tenant EdTech platform for tutors: student, parent, and teacher cabinet (PWA), admin control plane, public visiting cards, real-time whiteboard, and desktop client. Modules share one product story but live in separate private repos for deploy boundaries. Built and owned by Alexander ([asydneysummer](https://github.com/asydneysummer)).

**Production:** [*.hopeclass.ru](https://hopeclass.ru) (tenant subdomains), [repetitor.hopeclass.ru](https://repetitor.hopeclass.ru), [admin.hopeclass.ru](https://admin.hopeclass.ru), public cards at [hopeclass.ru/{slug}](https://hopeclass.ru), whiteboard at [desk.hopeclass.ru](https://desk.hopeclass.ru) and [desk.metod-orbita.ru](https://desk.metod-orbita.ru), student quizzes at [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz).

This repository (**[hopeclass-ecosystem](https://github.com/asydneysummer/hopeclass-ecosystem)**) is the public portfolio index for the HopeClass module family. Application source lives in the linked private repos below.

## Features

- Multi-tenant tutor SaaS with per-teacher PostgreSQL isolation
- Mobile-first PWA for teachers, students, and parents
- Control plane for tenant registry, provisioning, and support
- Public visiting cards at `hopeclass.ru/{slug}`
- Real-time whiteboard (Excalidraw) on desk hosts
- Electron desktop client with offline sync and white-label branding
- Quiz integration with Metod Orbita: assign from the tutor cabinet, launch via SSO, results via webhook

## Roles & capabilities

| Role | Can do |
|------|--------|
| **Teacher** | Use the tutor PWA on tenant hosts (`*.hopeclass.ru`, `repetitor.hopeclass.ru`); manage the tenant-scoped API; assign Metod Orbita quizzes (SSO launch to `metod-orbita.ru/quiz`); open the desk whiteboard; publish a visiting card at `hopeclass.ru/{slug}` |
| **Student** | Use the mobile-first PWA in the tutor tenant; take assigned quizzes on `metod-orbita.ru/quiz` via SSO; receive quiz outcomes synced back into HopeClass (webhook → reports and push) |
| **Parent** | Use parent-facing flows in the same tenant-scoped PWA (shared JWT auth and Host / `x-tenant-slug` routing) |
| **Admin (control plane)** | Operate tenant registry, provisioning, and support tooling on `admin.hopeclass.ru` (separate control-plane app, not tenant JWT) |

## Architecture / tech map

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

| From | To | Mechanism |
|------|-----|-----------|
| HopeClass tutor-app | metod-orbita quiz | Test assignment, SSO launch |
| quiz-module | HopeClass API | Webhook with per-tenant key → quiz reports + push |
| metod-orbita-shop | HopeClass | Trial schools, ensure-user, visiting-card editor API |
| hopeclass-cards | metod-orbita-shop | Card theme presets sync |
| hopeclass-desk | HopeClass server | JWT auth, `/api/desk/*` |

## Key engineering work

- **Multi-tenant PostgreSQL isolation** — one database per teacher behind a multi-tenant API proxy; routing by Host header and `x-tenant-slug`
- **Control plane** — tenant registry, provisioning, and support surfaces decoupled from tenant runtime (`admin.hopeclass.ru`)
- **Mobile-first PWA** — React + Vite tutor cabinet for teachers, students, and parents with JWT bearer auth
- **Visiting cards** — separately deployed public link-in-bio app at `hopeclass.ru/{slug}` with cross-product theme preset sync
- **Desk (Excalidraw)** — collaborative lobby + editor (`hopeclass-desk`) authenticated against HopeClass `/api/desk/*`
- **Electron offline** — desktop client with offline sync and white-label branding (`hopeclass-desktop`)
- **Quiz SSO and webhooks** — tutor assigns tests in-app; learners launch `metod-orbita.ru/quiz` via SSO; quiz module posts results with per-tenant webhook keys
- **Metod Orbita bridge** — shop integration for trial schools, user ensure, and visiting-card editor APIs; courses LMS SSO via shop JWT (`metod-orbita-courses`)

## Tech stack

- **Backend:** Node.js, Express, Prisma, PostgreSQL (per-tenant isolation via proxy)
- **Frontend:** React, Vite, Zustand, mobile-first PWA
- **Admin:** control-plane (tenant registry, provisioning, support)
- **Auth:** JWT bearer; multi-tenant routing by Host / `x-tenant-slug`
- **Desk:** Excalidraw-based collab app (separate repo)
- **Desktop:** Electron (separate repo)

## Repository layout

| Path | Role |
|------|------|
| `README.md` | Public employer-facing ecosystem index (this file) |

Runtime apps and services live in the module repositories listed below—not in this index repo.

## Local setup

There is no application code in **hopeclass-ecosystem**. Clone **[hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform)** *(private — request access)* and use its README / `.env.example` for env vars and deploy scripts.

Typical core dev commands (from **hopeclass-platform**):

```bash
# API (multi-tenant)
cd server && MULTI_TENANT=true CONTROL_DATABASE_URL=... npm run dev

# Tutor PWA
cd tutor-app && npm run dev

# Control plane
cd control-plane && npm run dev
```

Other modules (cards, desk, desktop, Metod Orbita) document their own setup in each repo.

## Related repositories

### HopeClass modules

| Module | Repository | Production |
|--------|------------|------------|
| **Core** — multi-tenant API, tutor PWA, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(private — request access)* | `*.hopeclass.ru`, `repetitor.hopeclass.ru`, `admin.hopeclass.ru` |
| **Visiting cards** — public link-in-bio pages | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(private — request access)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Online whiteboard** — lobby + Excalidraw editor | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(private — request access)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Desktop** — Electron client, offline sync | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(private — request access)* | Desktop releases |

### Metod Orbita (quiz & courses)

| Repo | Scope | Production |
|------|-------|------------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)* | Store, payments (Prodamus), quiz SPA + API, HopeClass bridge | [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) (quiz); shop surfaces per that repo |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(private — request access)* | Course platform; SSO via shop JWT | Per **metod-orbita-courses** deploy docs |

*Private portfolio index · request access to module source repos · author: Alexander / asydneysummer*

---

# HopeClass — экосистема для репетитора

**HopeClass** — мультитenant EdTech-платформа для репетиторов: кабинет ученика, родителя и преподавателя (PWA), control plane, публичные визитки, онлайн-доска и десктоп-клиент. Автор и владелец продукта: Alexander ([asydneysummer](https://github.com/asydneysummer)).

**Продакшен:** [*.hopeclass.ru](https://hopeclass.ru), [repetitor.hopeclass.ru](https://repetitor.hopeclass.ru), [admin.hopeclass.ru](https://admin.hopeclass.ru), визитки [hopeclass.ru/{slug}](https://hopeclass.ru), доска [desk.hopeclass.ru](https://desk.hopeclass.ru) / [desk.metod-orbita.ru](https://desk.metod-orbita.ru), квизы учеников — [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz).

Этот репозиторий — публичный портфолио-индекс экосистемы; исходники приложений — в связанных приватных репозиториях.

## Роли

| Роль | Возможности |
|------|-------------|
| **Преподаватель** | PWA на tenant-хостах; назначение квизов Metod Orbita (SSO); доска; визитка `hopeclass.ru/{slug}` |
| **Ученик** | PWA в tenant; прохождение квизов на `metod-orbita.ru/quiz`; результаты через webhook в HopeClass |
| **Родитель** | Родительские сценарии в том же tenant-scoped PWA |
| **Админ (control plane)** | Реестр tenant-ов, провижининг, поддержка на `admin.hopeclass.ru` |

## Модули

| Модуль | Репозиторий | Продакшен |
|--------|-------------|-----------|
| **Ядро** | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(приватный)* | `*.hopeclass.ru`, `repetitor.hopeclass.ru`, `admin.hopeclass.ru` |
| **Визитки** | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(приватный)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Доска** | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(приватный)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Десктоп** | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(приватный)* | Desktop-релизы |
| **Квизы (Metod Orbita)** | [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный)* | [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) |
| **Курсы (Metod Orbita)** | [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(приватный)* | По документации репозитория |

*Портфолио-индекс · запросите доступ к исходникам модулей*
