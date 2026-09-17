# HopeClass — Tutor Ecosystem

**HopeClass** is a multi-tenant EdTech platform for tutors: student/parent/teacher cabinet (PWA), admin control plane, public visiting cards, real-time whiteboard, and desktop client. Modules share one product story but live in separate private repos for clarity and deploy boundaries.

> **Student testing (quiz)** runs on [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) — assigned from the HopeClass tutor cabinet via SSO; results return through webhook. Source: [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)*.

## Features

- Multi-tenant tutor SaaS with per-teacher PostgreSQL isolation
- Mobile-first PWA for teachers, students, and parents
- Control plane for tenant registry, provisioning, and support
- Public visiting cards at `hopeclass.ru/{slug}`
- Real-time whiteboard (Excalidraw) at desk.hopeclass.ru
- Electron desktop client with offline sync and white-label branding
- Quiz integration with Metod Orbita via SSO and webhooks

## Modules

| Module | Repository | Production |
|--------|------------|------------|
| **Core** — multi-tenant API, tutor PWA, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(private — request access)* | `*.hopeclass.ru`, `repetitor.hopeclass.ru`, `admin.hopeclass.ru` |
| **Visiting cards** — public link-in-bio pages | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(private — request access)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Online whiteboard** — lobby + Excalidraw editor | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(private — request access)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Desktop** — Electron client, offline sync | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(private — request access)* | Desktop releases |

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

## Cross-product integrations

| From | To | Mechanism |
|------|-----|-----------|
| HopeClass tutor-app | metod-orbita quiz | Test assignment, SSO launch |
| quiz-module | HopeClass API | Webhook with per-tenant key → quiz reports + push |
| metod-orbita-shop | HopeClass | Trial schools, ensure-user, visiting-card editor API |
| hopeclass-cards | metod-orbita-shop | Card theme presets sync |
| hopeclass-desk | HopeClass server | JWT auth, `/api/desk/*` |

## Tech stack (core)

- **Backend:** Node.js, Express, Prisma, PostgreSQL (per-tenant isolation via proxy)
- **Frontend:** React, Vite, Zustand, mobile-first PWA
- **Admin:** control-plane (tenant registry, provisioning, support)
- **Auth:** JWT bearer; multi-tenant routing by Host / `x-tenant-slug`

## Setup

```bash
# API (multi-tenant)
cd server && MULTI_TENANT=true CONTROL_DATABASE_URL=... npm run dev

# Tutor PWA
cd tutor-app && npm run dev

# Control plane
cd control-plane && npm run dev
```

See each repo's README for env vars and deploy scripts.

## Related repos (Metod Orbita)

| Repo | Scope |
|------|-------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)* | Store, payments (Prodamus), quiz SPA + API, HopeClass bridge |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(private — request access)* | Course platform; SSO via shop JWT |

*Portfolio index · private source repos · author: Michael / asydneysummer*

---

# HopeClass — экосистема для репетитора

**HopeClass** — мультитenant EdTech-платформа для репетиторов: кабинет ученика/родителя/преподавателя (PWA), админ-панель, публичные визитки, онлайн-доска и десктоп-клиент. Модули образуют единый продукт, но вынесены в отдельные приватные репозитории для ясности границ деплоя.

> **Тестирование учеников (квизы)** работает на [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) — назначается из кабинета репетитора HopeClass через SSO; результаты возвращаются webhook-ом. Исходники: [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный — запросите доступ)*.

## Возможности

- Мультитenant SaaS для репетиторов с изоляцией PostgreSQL на каждого преподавателя
- Mobile-first PWA для учителей, учеников и родителей
- Control plane: реестр tenant-ов, провижининг, поддержка
- Публичные визитки на `hopeclass.ru/{slug}`
- Онлайн-доска (Excalidraw) на desk.hopeclass.ru
- Electron-клиент с офлайн-синхронизацией и white-label брендингом
- Интеграция квизов с Методикой Орбита через SSO и webhooks

## Модули

| Модуль | Репозиторий | Продакшен |
|--------|-------------|-----------|
| **Ядро** — мультитenant API, PWA репетитора, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(приватный — запросите доступ)* | `*.hopeclass.ru`, `repetitor.hopeclass.ru`, `admin.hopeclass.ru` |
| **Визитки** — публичные link-in-bio страницы | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(приватный — запросите доступ)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Онлайн-доска** — лобби + редактор Excalidraw | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(приватный — запросите доступ)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Десктоп** — Electron-клиент, офлайн-синхронизация | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(приватный — запросите доступ)* | Релизы для desktop |

## Архитектура (обзор)

```
┌─────────────────────────────────────────────────────────────┐
│ Экосистема HopeClass                                        │
│  tutor-app ←→ server (БД tenant на преподавателя) ←→ control-plane │
│  card-app (публичный) · desk-app (коллаб) · hopeclass-desktop │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP + shared secrets
                            ▼
┌─────────────────────────────────────────────────────────────┐
│ Методика Орбита (отдельные репозитории)                     │
│  metod-orbita-shop: магазин + модуль квизов                 │
│  metod-orbita-courses: LMS (SSO через JWT магазина)         │
└─────────────────────────────────────────────────────────────┘
```

## Межпродуктовые интеграции

| Откуда | Куда | Механизм |
|--------|------|----------|
| HopeClass tutor-app | квиз metod-orbita | Назначение теста, SSO-запуск |
| quiz-module | HopeClass API | Webhook с ключом tenant → отчёты + push |
| metod-orbita-shop | HopeClass | Пробные школы, ensure-user, API редактора визитки |
| hopeclass-cards | metod-orbita-shop | Синхронизация пресетов тем визиток |
| hopeclass-desk | HopeClass server | JWT-авторизация, `/api/desk/*` |

## Стек (ядро)

- **Backend:** Node.js, Express, Prisma, PostgreSQL (изоляция tenant через proxy)
- **Frontend:** React, Vite, Zustand, mobile-first PWA
- **Админка:** control-plane (реестр tenant-ов, провижининг, поддержка)
- **Auth:** JWT bearer; мультитenant-маршрутизация по Host / `x-tenant-slug`

## Установка

```bash
# API (мультитenant)
cd server && MULTI_TENANT=true CONTROL_DATABASE_URL=... npm run dev

# PWA репетитора
cd tutor-app && npm run dev

# Control plane
cd control-plane && npm run dev
```

Подробности по env и деплою — в README каждого репозитория.

## Связанные репозитории (Методика Орбита)

| Репозиторий | Назначение |
|-------------|------------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный — запросите доступ)* | Магазин, оплата (Prodamus), SPA + API квизов, мост HopeClass |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(приватный — запросите доступ)* | Платформа курсов; SSO через JWT магазина |

*Портфолио-индекс · приватные исходники · автор: Michael / asydneysummer*
