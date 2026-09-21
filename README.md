# HopeClass — Tutor Ecosystem

**HopeClass** is a multi-tenant EdTech platform for tutors: student/parent/teacher cabinet (PWA), control-plane admin, public visiting cards, real-time whiteboard, and desktop client. **Metod Orbita** is the commercial sibling line (store, quiz module, LMS). Modules share one product story but live in **separate private repositories** for clarity and deploy boundaries.

This repository is the **public portfolio index** — architecture, production links, role maps, and UI screenshots for employers and collaborators. Application source code is not published here; open a module repo link below and **request access** to review implementation.

> **Student testing (quiz)** runs on [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) — assigned from the HopeClass tutor cabinet via SSO; results return through webhook. Source: [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)*.

## Features

- **Multi-tenant tutor SaaS:** database-per-teacher PostgreSQL isolation, white-label domains on `*.hopeclass.ru`, shared login portal, dynamic PWA branding
- **Teacher workflows:** students and parents, invite links, subjects (algebra/geometry/physics), schedule (day/week + finance totals), topics and materials, homework assign/review, finance and receipts, groups, mock tests, quiz settings, notifications (in-app + Web Push)
- **Student & parent PWA:** role-specific dashboards, homework upload, topic materials, schedule; parents see linked children with read-only progress, homework, schedule, and billing (actions logged for teachers)
- **Control plane:** tenant registry, pause/activate, theme/assets, provisioning CLI, fleet migrations, support inbox backend
- **Public visiting cards** at `hopeclass.ru/{slug}` — link-in-bio layouts, blocks, theme presets synced to Metod shop
- **Online whiteboard (Desk):** lobby + Excalidraw editor on [desk.hopeclass.ru](https://desk.hopeclass.ru) and [desk.metod-orbita.ru](https://desk.metod-orbita.ru)
- **Desktop (Electron):** IDE-style teacher shell, offline SQLite snapshots, outbox replay, white-label installers on [releases.hopeclass.ru/desktop/](https://releases.hopeclass.ru/desktop/)
- **Metod Orbita shop:** catalog, Prodamus checkout, teacher/student accounts, HopeClass trial and visiting-card editor, quiz SPA at `/quiz`
- **Metod Orbita courses LMS:** [courses.metod-orbita.ru](https://courses.metod-orbita.ru) — modular courses, homework, coins, SSO from shop JWT

## Modules

| Module | Repository | Production |
|--------|------------|------------|
| **Core** — multi-tenant API, tutor PWA, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(private — request access)* | `*.hopeclass.ru`, [repetitor.hopeclass.ru](https://repetitor.hopeclass.ru), [admin.hopeclass.ru](https://admin.hopeclass.ru) |
| **Visiting cards** — public link-in-bio pages | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(private — request access)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Online whiteboard** — lobby + Excalidraw editor | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(private — request access)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Desktop** — Electron client, offline sync | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(private — request access)* | [releases.hopeclass.ru/desktop/](https://releases.hopeclass.ru/desktop/) |
| **Metod shop + quiz** | [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)* | [metod-orbita.ru](https://metod-orbita.ru), [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) |
| **Metod courses LMS** | [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(private — request access)* | [courses.metod-orbita.ru](https://courses.metod-orbita.ru) |

## Module detail — HopeClass Platform

**Product surface:** mobile-first PWA (`tutor-app`) for **teacher**, **student**, and **parent**; Express API (`server`) with `/api/*` and uploads; control-plane admin UI for fleet operations. Tenant resolved from `Host`, `x-tenant-slug`, or shared portal hosts. JWTs scoped with `tenantId`.

| Role | Capabilities |
|------|----------------|
| **teacher** | Full CRUD on students, parents, lessons, topics, homework, assignments, payments/expenses/receipts; invite and personal links (14-day TTL); groups; quiz settings; notifications; support settings; student subjects and archive |
| **student** | Own lessons, homework, topics, assignments; upload completion photos; topic/homework chat; profile updates; no access to other students |
| **parent** | Linked children (multi-child switch); read-only progress, schedule, homework, finance/receipts; parent–teacher messaging; `ParentLog` audit visible to teacher; cannot edit teacher data |
| **control-plane admin** | Not a tenant user — tenant CRUD, themes/assets, provisioning APIs, support inbox via control-plane session (separate from tutor JWT) |

**Engineering highlights:** database-per-tenant Prisma proxy; provisioning CLI; fleet migrations; quiz webhook ingestion from Metod; white-label manifest and VAPID per tenant.

## Module detail — HopeClass Cards

**Product surface:** static SPA at `hopeclass.ru/{slug}`; data from platform `GET /api/public/card/{slug}`; ten layout modes and block types (profile, links, social, CTA, gallery, YouTube, price, etc.); OG/meta tags; theme preset catalog with sync to Metod shop.

| Role | Capabilities |
|------|----------------|
| **Public visitor** | View published slug; follow outbound links; no login in this app |
| **Teacher / editor** | Edit card in HopeClass admin or Metod shop profile (persistence on platform API) |
| **Operator** | Build and deploy static assets via deploy scripts (hosting access required) |

## Module detail — HopeClass Desk

**Product surface:** login → board lobby → `/b/:id` Excalidraw editor; JWT + `x-tenant-slug`; API authorization on platform `/api/desk/*`.

| Role | Capabilities |
|------|----------------|
| **teacher** | List/create/rename/delete boards; assign subjects and student members; open editor; manage draw permissions |
| **student** | List assigned boards; collaborate on canvas; no lobby CRUD |
| **parent** | May authenticate; UI indicates boards are for teachers/students only (no board list/editor) |

**Engineering highlights:** embedded Excalidraw via iframe `postMessage`; Docker debrand stack; dual production hosts (shared HopeClass API vs Orbita-local API snapshot).

## Module detail — HopeClass Desktop

**Product surface:** Electron shell with activity bar, command palette, offline sync; installers and auto-update feeds per tenant slug.

| Role | Capabilities |
|------|----------------|
| **teacher** (primary) | Dashboard, students, schedule, topics, homework, chats, finance, groups, parents, textbooks (Polotno), quizzes, mock tests, receipts, subscription, settings |
| **student** | Dashboard, schedule, topics, homework, mock tests, quiz reports, support |
| **parent** | Dashboard, schedule, homework, teacher chats, finance, receipts, mock tests, quiz reports, support |

**Engineering highlights:** SQLite snapshots + outbox replay; `hopeclass://` file cache; branded release matrix; deep links.

## Module detail — Metod Orbita Shop (+ Quiz)

**Product surface:** shop SPA (catalog, cart, blog, profile, admin); quiz SPA at `/quiz/`; shop-server + quiz-module APIs behind nginx.

| Role | Capabilities |
|------|----------------|
| **Public (unauthenticated)** | Marketing home, catalog, product/bundle pages, cart UI, blog, reviews, about, announcements, legal pages, login/register, password-reset request, order status page, preview tokens for free materials |
| **customer** (teacher/buyer) | Purchase flow (Prodamus); profile tabs (orders, courses, favorites, preview library, HopeClass “My App”, documents, settings); manage linked students; HopeClass 14-day trial; visiting-card editor; quiz authoring and assignment; courses SSO links |
| **student** | Register with teacher phone; pending/approved teacher link; take quizzes; no HopeClass trial, no students tab, no card editor |
| **admin** | Full `/admin/*` — products, orders, users, promos, blog, mail, password-reset ops, student link requests; all customer capabilities |
| **quiz_admin** | Quiz admin UI/API only (shop login may redirect to `/quiz/`); no full shop admin middleware |

**Engineering highlights:** shared JWT between shop and quiz; Prodamus webhooks; `activateOrderCourses`; HopeClass trial/card/ensure-user bridge; HopeClass quiz SSO and result webhooks.

## Module detail — Metod Orbita Courses

**Product surface:** tile landing, SSO login, student dashboard (desktop + mobile), admin CMS; shop is identity and subscription source of truth.

| Role | Capabilities |
|------|----------------|
| **Public (unauthenticated)** | Landing tiles from public course catalog API; expanded tile view; login form; password-reset page |
| **teacher** (courses JWT — student listener) | Entitled courses, articles, homework submit, coin unlocks, push subscribe; device limits (2 desktop / 1 mobile) |
| **course_editor** | Admin course tree and article editors (`canEditCourses`); no homework review, students admin, or publish-to-shop |
| **admin** | Full admin tabs: courses CMS, homework review, students/devices, password-reset admin, publish course products to shop, coin admin |

Shop→courses role map: `admin`→`admin`, `quiz_admin`→`course_editor`, else→`teacher`.

## Screenshots

Production UI samples (no PII). Full-resolution assets live under [`docs/screenshots/`](docs/screenshots/).

### HopeClass

| Teacher dashboard | Schedule |
|:---:|:---:|
| ![Teacher dashboard](docs/screenshots/hopeclass/teacher-dashboard.png) | ![Teacher schedule](docs/screenshots/hopeclass/teacher-schedule.png) |

| Student dashboard | Parent dashboard |
|:---:|:---:|
| ![Student dashboard](docs/screenshots/hopeclass/student-dashboard.png) | ![Parent dashboard](docs/screenshots/hopeclass/parent-dashboard.png) |

| Control plane (tenants) | Desk lobby |
|:---:|:---:|
| ![Admin tenants](docs/screenshots/hopeclass/admin-tenants.png) | ![Desk lobby](docs/screenshots/hopeclass/desk-lobby.png) |

### Metod Orbita — shop ([metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop))

| Landing | Catalog |
|:---:|:---:|
| ![Shop landing](docs/screenshots/metod-shop/landing.png) | ![Catalog](docs/screenshots/metod-shop/catalog.png) |

| Blog | Profile |
|:---:|:---:|
| ![Blog](docs/screenshots/metod-shop/blog.png) | ![Profile](docs/screenshots/metod-shop/profile.png) |

| Admin — orders |
|:---:|
| ![Admin orders](docs/screenshots/metod-shop/admin-orders.png) |

### Metod Orbita — courses ([metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses))

| Public landing | Login |
|:---:|:---:|
| ![Courses landing](docs/screenshots/courses/landing-tiles.png) | ![Login](docs/screenshots/courses/login.png) |

| Student home | Course view |
|:---:|:---:|
| ![Student home](docs/screenshots/courses/student-home.png) | ![Student course](docs/screenshots/courses/student-course.png) |

| Article reader | Gated article |
|:---:|:---:|
| ![Student article](docs/screenshots/courses/student-article.png) | ![Article lock](docs/screenshots/courses/article-lock.png) |

| Mobile student |
|:---:|
| ![Mobile student](docs/screenshots/courses/mobile-student.png) |

| Admin — courses | Course editor |
|:---:|:---:|
| ![Admin courses](docs/screenshots/courses/admin-courses.png) | ![Admin editor](docs/screenshots/courses/admin-editor-courses.png) |

| Admin — students | Homework | Password reset |
|:---:|:---:|:---:|
| ![Admin students](docs/screenshots/courses/admin-students.png) | ![Admin homework](docs/screenshots/courses/admin-homework.png) | ![Password reset](docs/screenshots/courses/admin-password-reset.png) |

## Architecture (high level)

```
┌─────────────────────────────────────────────────────────────┐
│ HopeClass ecosystem                                         │
│  tutor-app ←→ server (tenant DB per teacher) ←→ control-plane │
│  card-app (public) · desk-app (collab) · hopeclass-desktop   │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP + shared secrets (integration)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│ Metod Orbita                                                │
│  metod-orbita.ru: shop-app + shop-server + quiz-app/module  │
│  courses.metod-orbita.ru: courses-app + courses-server      │
└─────────────────────────────────────────────────────────────┘
```

```
  metod-orbita.ru (nginx)
    /        → shop-app
    /quiz/   → quiz-app
    /api     → shop-server (Prodamus, HopeClass bridge, subscriptions)
    /api/quiz/ → quiz-module (tests, attempts, HopeClass SSO/webhooks)

  courses.metod-orbita.ru
    SPA + /api → courses-server (SSO exchange, content, homework, coins)
         └── subscription checks → shop API
```

## Metod Orbita — integration summary

| Repo | Scope | Production |
|------|-------|------------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) | Store, Prodamus, quiz SPA + API, HopeClass bridge, JWT SSO issuer | [metod-orbita.ru](https://metod-orbita.ru), [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) | LMS: course builder, student cabinet, homework, coins, publish-to-shop | [courses.metod-orbita.ru](https://courses.metod-orbita.ru) |

**Shop flows (evidence-backed):** cart → `POST /api/orders` → Prodamus pay → webhook → order paid → course subscription activation for course products; student registers with teacher phone → `StudentTeacher` pending → teacher approve/reject; preview tokens for free material samples.

**Courses flows:** shop login or `#sso=` exchange → courses JWT; access via shop subscriptions and `Course.shopProductId`; homework submit → admin review → coin awards → optional coin-priced article unlock; admin publish creates shop product and stores external id.

## Cross-product integrations

| From | To | Mechanism |
|------|-----|-----------|
| HopeClass tutor-app | metod-orbita quiz | Test assignment, HopeClass SSO launch (`typ: hopeclass_quiz`) |
| quiz-module | HopeClass API | Webhook with per-tenant key → quiz reports + push |
| metod-orbita-shop | HopeClass | Trial schools, ensure-user, visiting-card editor API, control DB lookups |
| hopeclass-cards | metod-orbita-shop | Theme preset sync (`card-presets.json`) |
| hopeclass-desk | HopeClass server | JWT auth, `/api/desk/*`, `/api/desk-live` WebSocket |
| metod-orbita-shop | metod-orbita-courses | JWT SSO (`POST /api/auth/exchange`); subscription and course-subscriber APIs |
| metod-orbita-shop | HopeClass quiz | Shared JWT; internal purchase-aware assignment helpers |

## Tech stack

| Area | Stack |
|------|--------|
| **HopeClass core** | Node.js, TypeScript, Express, Prisma, PostgreSQL; React 18, Vite, Zustand PWA; control-plane admin |
| **HopeClass cards** | React, Vite, nginx static + API proxy to platform |
| **HopeClass desk** | React, Vite, Excalidraw Docker stack, nginx, WebSocket room server |
| **HopeClass desktop** | Electron 33, React 19, SQLite (better-sqlite3), electron-updater |
| **Metod shop + quiz** | React, Vite, Tailwind, Zustand; Express, Prisma, PostgreSQL (shop + quiz DBs); Prodamus |
| **Metod courses** | React, Vite, Tailwind, Framer Motion; Express, Prisma, SQLite; shop REST for SSO/subscriptions |

**Auth patterns:** JWT bearer; HopeClass multi-tenant Host / `x-tenant-slug`; shop and quiz share JWT secret; courses trusts shop token exchange.

## Setup

Portfolio index only — clone **private** module repos after access is granted.

**HopeClass platform (typical local):**

```bash
cd server && cp .env.example .env && MULTI_TENANT=true npm run dev
cd tutor-app && npm install && npm run dev
cd control-plane && npm install && npm run dev
```

**HopeClass cards:** `cd card-app && npm install && npm run dev` (proxies `/api` to platform).

**HopeClass desk:** `cd desk-app && npm install && npm run dev` (requires platform API + optional Excalidraw stack).

**HopeClass desktop:** `cd hopeclass-desktop && npm install && npm run dev`.

**Metod shop + quiz:** configure each package from `.env.example`; align JWT between shop-server and quiz-module; `npm run dev` in each app/server.

**Metod courses:** `courses-server` + `courses-app` with `JWT_SECRET` matching shop and `SHOP_API_URL` pointing at running shop API.

Do not commit real credentials; use placeholders from each repo’s `.env.example`.

## Related repos

| Repository | Scope |
|------------|-------|
| [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(private — request access)* | Multi-tenant API, tutor PWA, control plane |
| [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(private — request access)* | Public visiting cards |
| [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(private — request access)* | Desk lobby + Excalidraw deploy |
| [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(private — request access)* | Electron client |
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(private — request access)* | Store, Prodamus, quiz, HopeClass bridge |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(private — request access)* | LMS; SSO via shop JWT |

*Portfolio index · private source repos · author: Alexander / [asydneysummer](https://github.com/asydneysummer)*

---

# HopeClass — экосистема для репетитора

**HopeClass** — мультитenant EdTech-платформа для репетиторов: кабинет ученика/родителя/преподавателя (PWA), control plane, публичные визитки, онлайн-доска и desktop-клиент. **Методика Орбита** — коммерческая линейка (магазин, модуль квизов, LMS). Модули образуют единый продукт, но вынесены в **отдельные приватные репозитории**.

Этот репозиторий — **публичный портфолио-индекс**: архитектура, production-ссылки, карта ролей и скриншоты UI. Исходный код приложений здесь не публикуется; откройте ссылку на модуль ниже и **запросите доступ** для ревью реализации.

> **Тестирование учеников (квизы)** — [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz): назначается из кабинета репетитора HopeClass через SSO; результаты возвращаются webhook-ом. Исходники: [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный — запросите доступ)*.

## Возможности

- **Мультитenant SaaS для репетиторов:** изоляция PostgreSQL на преподавателя, white-label на `*.hopeclass.ru`, общий портал входа, брендинг PWA
- **Кабинет преподавателя:** ученики и родители, invite-ссылки, предметы (алгебра/геометрия/физика), расписание (день/неделя + суммы по тарифам), темы и материалы, домашние задания, финансы и квитанции, группы, пробные тесты, настройки квизов, уведомления (in-app + Web Push)
- **PWA ученика и родителя:** отдельные дашборды; ученик загружает выполненные работы; родитель видит привязанных детей (прогресс, расписание, ДЗ, оплаты только на чтение; действия логируются для преподавателя)
- **Control plane:** реестр tenant-ов, пауза/активация, темы и ассеты, CLI провижининга, миграции по флоту, бэкенд поддержки
- **Публичные визитки** на `hopeclass.ru/{slug}` — link-in-bio, блоки, пресеты тем с синхронизацией в магазин Методики
- **Онлайн-доска (Desk):** лобби + редактор Excalidraw на [desk.hopeclass.ru](https://desk.hopeclass.ru) и [desk.metod-orbita.ru](https://desk.metod-orbita.ru)
- **Desktop (Electron):** оболочка в стиле IDE для учителя, офлайн SQLite, outbox, white-label установщики на [releases.hopeclass.ru/desktop/](https://releases.hopeclass.ru/desktop/)
- **Магазин Методики Орбита:** каталог, оплата Prodamus, аккаунты teacher/student, trial HopeClass и редактор визитки, квизы на `/quiz`
- **LMS курсов:** [courses.metod-orbita.ru](https://courses.metod-orbita.ru) — модульные курсы, ДЗ, монетки, SSO через JWT магазина

## Модули

| Модуль | Репозиторий | Продакшен |
|--------|-------------|-----------|
| **Ядро** — API, PWA репетитора, control plane | [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(приватный — запросите доступ)* | `*.hopeclass.ru`, [repetitor.hopeclass.ru](https://repetitor.hopeclass.ru), [admin.hopeclass.ru](https://admin.hopeclass.ru) |
| **Визитки** — публичные link-in-bio | [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(приватный — запросите доступ)* | [hopeclass.ru/{slug}](https://hopeclass.ru) |
| **Онлайн-доска** — лобби + Excalidraw | [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(приватный — запросите доступ)* | [desk.hopeclass.ru](https://desk.hopeclass.ru), [desk.metod-orbita.ru](https://desk.metod-orbita.ru) |
| **Desktop** — Electron, офлайн-синхронизация | [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(приватный — запросите доступ)* | [releases.hopeclass.ru/desktop/](https://releases.hopeclass.ru/desktop/) |
| **Магазин + квизы** | [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный — запросите доступ)* | [metod-orbita.ru](https://metod-orbita.ru), [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) |
| **LMS курсов** | [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(приватный — запросите доступ)* | [courses.metod-orbita.ru](https://courses.metod-orbita.ru) |

## Модуль — HopeClass Platform

**Поверхность продукта:** PWA (`tutor-app`) для **teacher**, **student**, **parent**; API Express (`server`); админка control plane. Tenant по `Host` / `x-tenant-slug` / shared portal. JWT с `tenantId`.

| Роль | Возможности |
|------|-------------|
| **teacher** | Полное управление учениками, родителями, уроками, темами, ДЗ, финансами, ссылками (TTL 14 дней), группами, квизами, уведомлениями, настройками поддержки |
| **student** | Свои уроки, ДЗ, темы; загрузка фото выполненных работ; чаты; без доступа к чужим данным |
| **parent** | Привязанные дети (переключение); прогресс, расписание, ДЗ, финансы на чтение; переписка с учителем; логи `ParentLog` у преподавателя |
| **админ control-plane** | Не tenant-user: реестр tenant-ов, темы, провижининг, поддержка (отдельная сессия control-plane) |

**Инженерия:** database-per-tenant + Prisma proxy; CLI провижининга; миграции по флоту; webhook квизов; white-label manifest и VAPID.

## Модуль — HopeClass Cards

**Поверхность:** SPA `hopeclass.ru/{slug}`; `GET /api/public/card/{slug}`; 10 макетов; блоки и OG/meta; пресеты тем → sync в shop.

| Роль | Возможности |
|------|-------------|
| **Публичный посетитель** | Просмотр опубликованного slug без входа |
| **Репетитор / редактор** | Редактирование в админке HopeClass или профиле магазина |
| **Оператор** | Сборка и деплой статики |

## Модуль — HopeClass Desk

**Поверхность:** вход → лобби досок → `/b/:id` Excalidraw; JWT + `x-tenant-slug`; авторизация на `/api/desk/*` платформы.

| Роль | Возможности |
|------|-------------|
| **teacher** | CRUD досок, предметы, участники, редактор, права на рисование |
| **student** | Доступные доски, совместное рисование |
| **parent** | Вход возможен; досок в UI нет (только teacher/student) |

**Инженерия:** Excalidraw в iframe + `postMessage`; Docker debrand; два prod-хоста (общий API HopeClass vs локальный API на Orbita).

## Модуль — HopeClass Desktop

**Поверхность:** Electron, offline sync, white-label релизы.

| Роль | Возможности |
|------|-------------|
| **teacher** | Дашборд, ученики, расписание, темы, ДЗ, чаты, финансы, группы, родители, учебники (Polotno), квизы, mock tests, подписка, настройки |
| **student** | Дашборд, расписание, темы, ДЗ, mock tests, отчёты квизов, поддержка |
| **parent** | Дашборд, расписание, ДЗ, чаты, финансы, квитанции, mock tests, отчёты квизов, поддержка |

**Инженерия:** SQLite snapshots + outbox; протокол `hopeclass://`; матрица брендированных сборок.

## Модуль — Metod Orbita Shop (+ Quiz)

**Поверхность:** shop SPA + quiz SPA `/quiz/` + shop-server + quiz-module.

| Роль | Возможности |
|------|-------------|
| **Публичный (без входа)** | Лендинг, каталог, товар/набор, корзина, блог, отзывы, about, объявления, legal, login/register, сброс пароля, статус заказа, preview бесплатных материалов |
| **customer** | Покупки (Prodamus); профиль (заказы, курсы, избранное, preview, HopeClass, документы, настройки); ученики; trial HopeClass 14 дней; редактор визитки; авторинг квизов; SSO в courses |
| **student** | Регистрация с телефоном учителя; связь pending/approved; прохождение квизов; без trial HopeClass и без редактора визитки |
| **admin** | Полная `/admin/*`: товары, заказы, пользователи, промо, блог, почта, сброс паролей, заявки учеников |
| **quiz_admin** | Только админка квизов; без полной админки магазина |

**Инженерия:** общий JWT shop/quiz; Prodamus; `activateOrderCourses`; мост HopeClass; SSO квизов HopeClass и webhooks результатов.

## Модуль — Metod Orbita Courses

**Поверхность:** плиточный лендинг, SSO, кабинет слушателя (desktop/mobile), админ CMS; магазин — identity и подписки.

| Роль | Возможности |
|------|-------------|
| **Публичный (без входа)** | Плитки курсов, разворот плитки, форма входа, страница сброса пароля |
| **teacher** (роль слушателя в JWT courses) | Купленные курсы, статьи, ДЗ, unlock за монетки, push; лимиты устройств (2 desktop / 1 mobile) |
| **course_editor** | Редактирование дерева курсов и статей; без проверки ДЗ и publish в shop |
| **admin** | Все вкладки: CMS, проверка ДЗ, студенты/устройства, сброс паролей, публикация продукта в shop, монетки |

Маппинг shop→courses: `admin`→`admin`, `quiz_admin`→`course_editor`, иначе→`teacher`.

## Скриншоты

Образцы production UI (без персональных данных). Файлы: [`docs/screenshots/`](docs/screenshots/).

### HopeClass

| Дашборд преподавателя | Расписание |
|:---:|:---:|
| ![Teacher dashboard](docs/screenshots/hopeclass/teacher-dashboard.png) | ![Teacher schedule](docs/screenshots/hopeclass/teacher-schedule.png) |

| Дашборд ученика | Дашборд родителя |
|:---:|:---:|
| ![Student dashboard](docs/screenshots/hopeclass/student-dashboard.png) | ![Parent dashboard](docs/screenshots/hopeclass/parent-dashboard.png) |

| Control plane (tenant-ы) | Лобби Desk |
|:---:|:---:|
| ![Admin tenants](docs/screenshots/hopeclass/admin-tenants.png) | ![Desk lobby](docs/screenshots/hopeclass/desk-lobby.png) |

### Методика Орбита — магазин ([metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop))

| Лендинг | Каталог |
|:---:|:---:|
| ![Shop landing](docs/screenshots/metod-shop/landing.png) | ![Catalog](docs/screenshots/metod-shop/catalog.png) |

| Блог | Профиль |
|:---:|:---:|
| ![Blog](docs/screenshots/metod-shop/blog.png) | ![Profile](docs/screenshots/metod-shop/profile.png) |

| Админ — заказы |
|:---:|
| ![Admin orders](docs/screenshots/metod-shop/admin-orders.png) |

### Методика Орбита — курсы ([metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses))

| Публичный лендинг | Вход |
|:---:|:---:|
| ![Courses landing](docs/screenshots/courses/landing-tiles.png) | ![Login](docs/screenshots/courses/login.png) |

| Домашняя студента | Курс |
|:---:|:---:|
| ![Student home](docs/screenshots/courses/student-home.png) | ![Student course](docs/screenshots/courses/student-course.png) |

| Статья | Заблокированная статья |
|:---:|:---:|
| ![Student article](docs/screenshots/courses/student-article.png) | ![Article lock](docs/screenshots/courses/article-lock.png) |

| Mobile студент |
|:---:|
| ![Mobile student](docs/screenshots/courses/mobile-student.png) |

| Админ — курсы | Редактор курса |
|:---:|:---:|
| ![Admin courses](docs/screenshots/courses/admin-courses.png) | ![Admin editor](docs/screenshots/courses/admin-editor-courses.png) |

| Админ — студенты | Домашние задания | Сброс пароля |
|:---:|:---:|:---:|
| ![Admin students](docs/screenshots/courses/admin-students.png) | ![Admin homework](docs/screenshots/courses/admin-homework.png) | ![Password reset](docs/screenshots/courses/admin-password-reset.png) |

## Архитектура (обзор)

```
┌─────────────────────────────────────────────────────────────┐
│ Экосистема HopeClass                                        │
│  tutor-app ←→ server (БД tenant на преподавателя) ←→ control-plane │
│  card-app (публичный) · desk-app · hopeclass-desktop        │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTP + интеграционные секреты
                            ▼
┌─────────────────────────────────────────────────────────────┐
│ Методика Орбита                                             │
│  metod-orbita.ru: shop + quiz                              │
│  courses.metod-orbita.ru: LMS                               │
└─────────────────────────────────────────────────────────────┘
```

```
  metod-orbita.ru (nginx)
    /        → shop-app
    /quiz/   → quiz-app
    /api     → shop-server
    /api/quiz/ → quiz-module

  courses.metod-orbita.ru
    SPA + /api → courses-server → проверки подписок в shop API
```

## Методика Орбита — интеграции

| Репозиторий | Назначение | Продакшен |
|-------------|------------|-----------|
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) | Магазин, Prodamus, квизы, мост HopeClass, SSO | [metod-orbita.ru](https://metod-orbita.ru), [metod-orbita.ru/quiz](https://metod-orbita.ru/quiz) |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) | LMS, publish в shop, монетки | [courses.metod-orbita.ru](https://courses.metod-orbita.ru) |

**Потоки магазина:** корзина → заказ → Prodamus → webhook → оплачен → активация подписки на курс; ученик с телефоном учителя → pending → approve/reject; preview-токены для бесплатных фрагментов.

**Потоки courses:** SSO/exchange → JWT courses; доступ по подпискам shop и `shopProductId`; ДЗ → проверка admin → монетки → unlock статей за монетки; publish создаёт продукт в shop.

## Межпродуктовые интеграции

| Откуда | Куда | Механизм |
|--------|------|----------|
| HopeClass tutor-app | квиз metod-orbita | Назначение теста, SSO HopeClass |
| quiz-module | HopeClass API | Webhook с ключом tenant → отчёты + push |
| metod-orbita-shop | HopeClass | Trial, ensure-user, API редактора визитки |
| hopeclass-cards | metod-orbita-shop | Синхронизация пресетов тем |
| hopeclass-desk | HopeClass server | JWT, `/api/desk/*`, WebSocket desk-live |
| metod-orbita-shop | metod-orbita-courses | SSO (`POST /api/auth/exchange`), подписки |
| metod-orbita-shop | HopeClass quiz | Общий JWT; internal assignment helpers |

## Стек

| Область | Стек |
|---------|------|
| **Ядро HopeClass** | Node.js, TypeScript, Express, Prisma, PostgreSQL; React, Vite, Zustand PWA; control-plane |
| **Визитки** | React, Vite, nginx + proxy к platform |
| **Desk** | React, Vite, Excalidraw Docker, nginx, WebSocket |
| **Desktop** | Electron 33, React 19, SQLite, electron-updater |
| **Shop + quiz** | React, Vite, Tailwind, Zustand; Express, Prisma, PostgreSQL; Prodamus |
| **Courses** | React, Vite, Tailwind, Framer Motion; Express, Prisma, SQLite; REST shop |

**Auth:** JWT; multi-tenant Host / `x-tenant-slug`; shop/quiz — общий секрет JWT; courses — exchange shop token.

## Установка

Только индекс портфолио — клонируйте **приватные** репозитории после выдачи доступа.

**HopeClass platform:**

```bash
cd server && cp .env.example .env && MULTI_TENANT=true npm run dev
cd tutor-app && npm install && npm run dev
cd control-plane && npm install && npm run dev
```

**HopeClass cards:** `cd card-app && npm install && npm run dev`

**HopeClass desk:** `cd desk-app && npm install && npm run dev`

**HopeClass desktop:** `cd hopeclass-desktop && npm install && npm run dev`

**Metod shop + quiz:** `.env.example` в каждом пакете; JWT shop-server = quiz-module; `npm run dev`.

**Metod courses:** `courses-server` + `courses-app`; `JWT_SECRET` как в shop; `SHOP_API_URL` на работающий shop API.

Не коммитьте реальные секреты — только плейсхолдеры из `.env.example`.

## Связанные репозитории

| Репозиторий | Назначение |
|-------------|------------|
| [hopeclass-platform](https://github.com/asydneysummer/hopeclass-platform) *(приватный — запросите доступ)* | API, PWA, control plane |
| [hopeclass-cards](https://github.com/asydneysummer/hopeclass-cards) *(приватный — запросите доступ)* | Публичные визитки |
| [hopeclass-desk](https://github.com/asydneysummer/hopeclass-desk) *(приватный — запросите доступ)* | Desk + Excalidraw |
| [hopeclass-desktop](https://github.com/asydneysummer/hopeclass-desktop) *(приватный — запросите доступ)* | Desktop-клиент |
| [metod-orbita-shop](https://github.com/asydneysummer/metod-orbita-shop) *(приватный — запросите доступ)* | Магазин, Prodamus, квизы, HopeClass |
| [metod-orbita-courses](https://github.com/asydneysummer/metod-orbita-courses) *(приватный — запросите доступ)* | LMS, SSO через shop |

*Портфолио-индекс · приватные исходники · автор: Александр / [asydneysummer](https://github.com/asydneysummer)*
