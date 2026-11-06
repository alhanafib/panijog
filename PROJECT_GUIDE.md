# Panijog — Project Guide (A to Z)

> **Read this first.** If you are an AI agent or a new developer picking up this
> project, this file is the complete handover document: product, repository,
> architecture, conventions, Firebase setup, deployment, and QA. A short
> overview lives in [`README.md`](./README.md); licensing in
> [`LICENSE`](./LICENSE). Do not delete this file.

**Last updated:** 6 November 2026 (v1.0 + latest UI/nav fixes same day).

---

## Table of contents

1. [What Panijog is](#1-what-panijog-is)
2. [Repository layout](#2-repository-layout)
3. [Hard rules (never break)](#3-hard-rules-never-break)
4. [Tech stack](#4-tech-stack)
5. [App features (1.0)](#5-app-features-10)
6. [Website — structure and design](#6-website--structure-and-design)
7. [Admin panel — structure and design](#7-admin-panel--structure-and-design)
8. [Firebase project, configs, and rules](#8-firebase-project-configs-and-rules)
9. [Firestore collections reference](#9-firestore-collections-reference)
10. [Admin authorization (RBAC) and Super Admin setup](#10-admin-authorization-rbac-and-super-admin-setup)
11. [APK metadata and distribution](#11-apk-metadata-and-distribution)
12. [Local preview and QA tooling](#12-local-preview-and-qa-tooling)
13. [Validation checklist](#13-validation-checklist)
14. [Deployment procedures](#14-deployment-procedures)
15. [Current live status and remaining to-dos](#15-current-live-status-and-remaining-to-dos)
16. [Content editing / common tasks](#16-content-editing--common-tasks)

---

## 1. What Panijog is

Panijog (Bengali: পানিযোগ — roughly "water discipline") is a hydration-tracking
Android app plus three supporting deliverables, all under one repo:

1. **Android app** — native Kotlin + Jetpack Compose; version 1.0, released
   6 November 2026. Package `com.hanafi.panijog.hytdrk`.
2. **Marketing website** — static HTML/CSS/vanilla JS at
   <https://panijog.web.app/> (Firebase Hosting site `panijog`).
3. **Admin panel** — static HTML/CSS/JS dashboard at
   <https://panijog-admin-cp.web.app/> (Firebase Hosting site
   `panijog-admin-cp`), backed by the same Firestore project.
4. **Documentation** — this guide, README, LICENSE, and app-side notes in
   `doc/`.

Product philosophy: **calm, local-first, honest scope**. Never claim unshipped
features, never invent stats/ratings/testimonials/download counts. Download
counts shown in the admin dashboard are **manual counters** set by the operator
(Settings → "APK download count").

- **Developer:** Mohammadullah Al Hanafi (<https://mohammadullah.web.app/>)
- **Support/tip jar:** <https://www.supportkori.com/hanafi> (a "buy me a
  coffee" page — **not** a support desk; does not handle support cases)
- **Contact:** website contact form → Firestore collection `contactMessages`
  (read in Firebase Console)
- **Source repo:** <https://github.com/alhanafib/panijog>

---

## 2. Repository layout

```
Panijog app/                          (repo root)
├── .firebaserc                       default project = gen-lang-client-0297092818
├── firebase.json                     ROOT RULES DEPLOYER (firestore.rules only)
├── .gitignore
├── doc/                              APP-side files, docs, and rules
│   ├── Panijog.apk                   the released APK (binary source of truth)
│   ├── firestore.rules               ★ THE unified rules (app+website+admin)
│   ├── PROJECT_GUIDE.md              this file (synced copy in ../github/)
│   ├── README.md, LICENSE            short docs (synced copies in ../github/)
│   ├── firebase.rules                AI Studio app rules — DO NOT EDIT
│   ├── firebase.json                 AI Studio copy — DO NOT EDIT/DELETE
│   ├── firebase-blueprint.json       AI Studio schema — DO NOT EDIT/DELETE
│   ├── notes.txt, command.cmd        operator quick notes
│   └── admin_panal_prompt.txt,
│       app_prompt.txt, website_prompt.txt — original AI prompts (reference)
├── website/                          WEBSITE project (everything web)
│   ├── firebase.json                 hosting site "panijog", public "public/"
│   ├── .firebaserc, .gitignore
│   ├── README.md                     website-specific notes
│   └── public/                       DEPLOYABLE WEBSITE (79 files)
│       ├── index.html … 17 HTML pages
│       ├── 404.html, robots.txt, sitemap.xml, site.webmanifest,
│       │   google2b8b929fdefa4ff3.html (site verification)
│       ├── cat-computer.gif          404 page animation
│       └── assets/
│           ├── css/    base.css, components.css, pages.css, fonts.css
│           ├── js/     main.js, contact.js, firebase-config.js
│           ├── fonts/  32 self-hosted WOFF2 (Plus Jakarta Sans, Hind Siliguri)
│           ├── img/    logo, favicons, og-image
│           └── screenshots/  7 real app screenshots + README
├── admin/                            ADMIN PANEL project
│   ├── firebase.json                 hosting site "panijog-admin-cp", public "public/"
│   ├── firestore.indexes.json        composite index definitions
│   ├── .firebaserc, .gitignore
│   ├── README.md                     admin setup notes
│   └── public/                       DEPLOYABLE ADMIN
│       ├── login.html, dashboard.html … 17 HTML files (incl. index → dashboard
│       │   redirect)
│       └── assets/
│           ├── css/    style.css, fonts.css
│           ├── js/     firebase-config.js, auth.js, guards.js, boot.js,
│           │           shell.js, utilities.js, + one module per page
│           ├── fonts/  same 32 WOFF2 (copied from website)
│           ├── img/    logo, favicons
│           └── icons/  (svg assets)
├── github/                           copy for public repo pushes
│   ├── PROJECT_GUIDE.md, README.md, LICENSE   (synced from doc/)
└── logo/                             brand source files (user-provided;
                                      app_main_icon.png, logo.png, .ai, .jfif)
```

### Rules deployment — IMPORTANT

There is **one** rules file: `doc/firestore.rules`. Both site
`firebase.json` files point at it (`"rules": "../doc/firestore.rules"`), and a
root-level `firebase.json` + `.firebaserc` exist purely to deploy the rules
without an "outside project directory" error. Deploy rules from the **root**:

```
cd "Panijog app"
firebase deploy --only firestore:rules
```

or from `admin/` (which also deploys indexes):

```
cd admin
firebase deploy --only firestore:rules firestore:indexes
```

Never duplicate the rules into other files.

---

## 3. Hard rules (never break)

1. **File placement:** website deployable files → `website/public/`; website
   config/docs → `website/` (outside public). Same pattern for `admin/`.
   App-side files → `doc/`. Root `Panijog.apk` and `logo/` stay as working
   copies.
2. **Never edit or delete** the AI Studio files: `doc/firebase.rules`,
   `doc/firebase.json`, `doc/firebase-blueprint.json`, `doc/README.md`,
   `doc/LICENSE`. Also keep `doc/notes.txt`, `doc/command.cmd`,
   `doc/*_prompt.txt` (operator's own files).
3. **Never deploy without explicit user authorization** (`firebase deploy`).
4. **No fake data anywhere:** no invented stats, ratings, testimonials,
   download counts, or release dates. Empty analytics must show honest empty
   states. The dashboard "APK downloads" KPI is a manual counter
   (`appConfig/main.downloadCount`).
5. **Static only:** no React/Vue/Angular/Next, no build step, no npm bundler,
   no CDN scripts. HTML5 + CSS3 + vanilla ES modules.
6. **No emojis as UI icons** — inline SVG only (stroke style, `stroke-width
   ~1.8`, `currentColor`).
7. **No inline event handlers / no inline scripts/styles** (CSP friendly).
   Admin CSS may use `'unsafe-inline'` for styles only.
8. **Fonts self-hosted** (Plus Jakarta Sans + Hind Siliguri, 32 WOFF2, both
   sites). Never add a Google Fonts request.
9. **English UI, Bangla accents**; website has an EN/বাং language toggle.
   Bangla text renders in Hind Siliguri (also the fallback in `--font-sans`).
10. **html-validate must pass** on every site: `npx html-validate@9.7.1
    "*.html"` exit code 0.
11. **Accessibility:** keyboard focus visible, Escape closes menus/modals,
    `prefers-reduced-motion` respected, no horizontal overflow 320–1440 px.

---

## 4. Tech stack

### Android app
- Kotlin, Jetpack Compose, Room, Hilt
- Firebase Auth (Google sign-in), Cloud Firestore sync, Firebase App Check
- Local-first Room storage; JSON/CSV export; account deletion in-app
- No Crashlytics, no Firebase Analytics, no ads SDK in 1.0

### Website
- Static HTML/CSS/JS; no framework, no build step
- Self-hosted fonts; CSP via meta tag + hosting headers
- Firestore **REST API** for the contact form (no SDK scripts on website)
- Firebase Hosting, site `panijog`

### Admin panel
- Static HTML/CSS/JS; Firebase v10.12.0 modular SDK from `www.gstatic.com` CDN
- Firestore + Firebase Auth (email/password); no Cloud Functions (Spark plan)
- Firebase Hosting, site `panijog-admin-cp`

### Firestore security
- Single `rules_version = '2'` file, `doc/firestore.rules`, covering app,
  website contact form, and admin RBAC (details in §8–§9).

---

## 5. App features (1.0)

### Shipped in 1.0 (verified against the APK)
Daily dashboard with wave progress; quick + custom water logging; 7 drink
categories (water, tea, coffee, milk, fresh juice, coconut water, other); daily
goal with suggested estimate; reminders with quiet hours; boot rescheduling;
7-day analytics (average + beverage breakdown); streaks, badges, virtual
garden; offline-first Room storage; Google sign-in + Firestore sync
(owner-scoped rules); JSON/CSV export; account deletion; in-app privacy/terms;
English + Bangla; automatic dark theme.

### Planned, NOT in 1.0 (do not claim these ship)
Health Connect, weather suggestions, widgets, Quick Settings tile, Wear OS,
voice logging, family/multi-profile, biometric app lock, privacy dashboard,
calendar/history, custom vessel presets, snooze/skip, exact-time reminders,
crash reporting, Firebase Analytics, import/restore, remote push.

`website/features.html` is the source of truth for this split ("Available now"
= shipped; everything else explicitly "Planned").

### Permissions (from the manifest)
Internet + network state, notifications (optional), receive boot completed.
No location, microphone, camera, contacts, or broad storage.

---

## 6. Website — structure and design

### Pages (17)

| File | Purpose |
| --- | --- |
| `index.html` | Home: hero, features, how-it-works, preview, FAQ, CTA |
| `features.html` | Available vs. planned (honest split) |
| `download.html` | GitHub download flow, APK metadata, SHA-256, install steps |
| `help.html` | Help center index |
| `help-install/setup/account/data/reminders/troubleshooting.html` | 6 help articles |
| `changelog.html` | Released versions only |
| `about.html` | Principles, stack, release facts |
| `contact.html` | Firestore-backed contact form |
| `privacy-policy.html`, `terms.html`, `license.html` | Legal |
| `404.html` | Branded not-found + `cat-computer.gif` |

Navigation (header): page links **centred** (Home, Features, Help, About,
Contact) + the "Get Panijog" blue CTA placed immediately beside the language
and theme toggles on the right (2026-11-06 layout). The Download link itself
is intentionally not in the nav list; the download page stays reachable from
the CTA, footer "Explore" column, hero CTAs, and content links. On mobile
(≤900px) the CTA is hidden from the bar and the right cluster is hamburger +
language + theme, tightly grouped with small gaps. Footer: Explore / Legal /
Support (GitHub repo, Buy me a coffee, Developer site, Send a message).

### Download flow (GitHub-hosted APK)
Spark plan forbids serving `.apk` from Firebase Hosting, so the APK lives on
GitHub Releases:
<https://github.com/alhanafib/panijog/releases/tag/android-app>.
`download.html` explains: **Step 1** — open the release page; **Step 2** —
click the `Panijog.apk` asset under Assets. The GitHub button is highlighted
as "recommended". Verified against SHA-256 shown on the page (§11).

### CSS system (4 sheets, loaded in order)
- `fonts.css` — self-hosted `@font-face` (two families)
- `base.css` — tokens (`:root` custom properties), reset, typography,
  accessibility, motion, print; `--font-sans` = "Plus Jakarta Sans", "Hind
  Siliguri", …; `--font-bn` = "Hind Siliguri", …
- `components.css` — buttons, badges, cards, floating glass header, footer,
  phone mockups (`.shot-phone` frames real screenshots), forms, FAQ, timeline,
  callout, reveal
- `pages.css` — page-level layouts (hero, download shell, legal, contact,
  about, changelog, 404, help pager)

Theme system: `data-theme` attribute on `<html>`; three-state toggle
(white/dark/system, default system), persisted in `localStorage`; system
follows `prefers-color-scheme`. Floating glass pill header (sticky, blurred).

Language system (EN/বাং): `data-i18n` dictionary in `main.js`; switching sets
`html[lang]`, which swaps Hind Siliguri in. Grid/flex children get
`min-width: 0` and `overflow-wrap` to prevent overflow; long hashes/urls break
with `overflow-wrap: anywhere`.

### JS (3 files, external, CSP-safe)
- `main.js` — header state, mobile nav (Escape + focus management), theme
  toggle, i18n toggle, reveal-on-scroll (IntersectionObserver),
  copy-to-clipboard, footer year. No dependencies.
- `contact.js` — validates form, honeypot (`website` field must be empty),
  45 s cooldown via `localStorage` key `panijog.contact.lastSent`, POSTs to
  Firestore REST, honest success/error messages (403 suggests alternative
  channels). Imports `firebase-config.js`.
- `firebase-config.js` — public web config (public by design; rules protect data).

### Hosting headers (website firebase.json)
CSP (self-only), nosniff, DENY framing, strict referrer policy, locked-down
Permissions-Policy, COOP, HSTS. Cache: HTML no-cache; CSS/fonts/imgs 1 year
immutable; JS 1 hour. Clean URLs, no trailing slash.

---

## 7. Admin panel — structure and design

### Architecture
Single Firebase project as the website, but a separate Hosting site. Shell
pattern: each page is a thin HTML file (`<div id="app">` + `boot.js`).
`boot.js` renders the shell, enforces roles, then dynamically imports the
page module and calls `init(ctx)`.

```
login → auth.js (signInWithEmailAndPassword, no persistence=none? default local)
      → getAdminProfile(uid) from adminProfiles
      → guards.js requireAuth / requireRole (redirects login.html / 403.html)
      → shell.js renderShell (sidebar + topbar + #content)
      → <page>.module init()
```

### Files in `admin/public/assets/js/`
- `firebase-config.js` — public web app config (separate appId from website)
- `auth.js` — `signIn`, `logout`, `getAdminProfile`; login errors surface a
  clear message if Firestore is unreachable ("Could not reach the database.
  Create Firestore and deploy rules first.")
- `guards.js` — `requireAuth(redirect)`, `requireRole(profile, minRole)`
- `boot.js` — page → module/title/minRole mapping; handles `index.html`
  redirect
- `shell.js` — NAV list, SVG icon set, sidebar + topbar template, sidebar
  toggle (desktop collapse `.app-shell.sidebar-collapsed`, mobile drawer +
  backdrop), logout link, theme toggle, **layout: backdrop div is a sibling of
  `.app-shell`, never a grid child** (kept like this deliberately — see §13)
- `utilities.js` — `popup(...)` centered OK-dialog (replaced toasts; name
  `showToast` kept as alias for compatibility), formatting, skeleton helpers
- Page modules — `dashboard.js users.js user-details.js releases.js
  campaigns.js analytics.js feedback.js notifications.js content.js
  audit-logs.js settings.js admins.js profile.js login.js`

### Pages (16 + redirect)
login, dashboard, users, user-details, releases, campaigns, analytics,
feedback, notifications, content, audit-logs, settings, admins, profile, 403,
404 (plus `index.html` → dashboard redirect).

### Role matrix (`boot.js` MIN_ROLE)
Support+ → users, user-details, feedback · Admin+ → releases, campaigns,
notifications, content, audit-logs · Analyst+ → analytics · SuperAdmin →
settings, admins.

### Admin UI
Floating glass header + sidebar (260px) + content column; **collapsible nav
groups** (Overview / Management / Insights / System). Each group header is a
`<button class="sidebar-group-toggle">` with a chevron; the group containing
the current page is open by default, other groups start collapsed, and
toggle state persists in `localStorage` (`panijog.admin.navgroups`); links
live inside `.sidebar-group-items` (`display:none` until the group has
`is-open`). KPI cards, data
tables (`table-wrap` with horizontal scroll), badges, modals
(`modal-backdrop`), skeleton/empty/error states; responsive 320–1440 px.
Theme toggle (white/dark/system, default system) — same behaviour as the
website. Self-hosted fonts (same 32 WOFF2). Password show/hide toggle (eye
SVG) on the login card.

### Dashboard specifics
- KPIs: total users, new users (last 7 days), APK downloads (manual counter
  from `appConfig/main.downloadCount`), recent audit log entries.
- **Regression note:** `downloadCount` must be read from `appConfig/main`
  *before* the KPI grid template is built (older code referenced it before
  `let downloadCount` — TDZ error "Cannot access 'downloadCount' before
  initialization"; fixed 2026-11-06 by moving the `getDoc` read above the
  Promise.all block).
- Settings page: SuperAdmin only; edit "APK download count" and similar
  `appConfig/main` keys via a form; save → `appConfig/main.update`.

---

## 8. Firebase project, configs, and rules

### Project
- **Project ID:** `gen-lang-client-0297092818`
- **Project number:** `178323036143`
- **Plan:** Spark (no Cloud Functions, no custom claims, no paid hosting
  features)
- **Recommended Firestore region:** `asia-south1`
- **Hosting sites:** `panijog` (website), `panijog-admin-cp` (admin)

### Web app configs (public by design)
| App | apiKey | appId |
| --- | --- | --- |
| Website | `AIzaSyDrvaz-dW6R2M83VFkg16gxzeTX2zkSKo0` | `1:178323036143:web:f8207b796fcf56d7ef481a` |
| Admin | same apiKey | `1:178323036143:web:23de24a9aa20816cef481a` |

Security is enforced by Firestore rules and API-key referrer restrictions, not
by hiding the config.

### Rules deployment
Single file `doc/firestore.rules` (root `firebase.json` → `"rules":
"doc/firestore.rules"`; website/admin firebase.json → `../doc/firestore.rules`).
Admin `firebase.json` also indexes (`firestore.indexes.json`: auditLogs by
`timestamp` DESC; feedback by `status` ASC + `createdAt` DESC; releases by
`releaseDate` DESC; users by `createdAt` DESC).

### CSP notes
- Website pages: meta CSP, `script-src 'self'` only (no CDN).
- Admin pages: meta CSP allows `www.gstatic.com` + `apis.google.com` for the
  Firebase CDN SDK and `'unsafe-inline'` styles only.
- Hosting headers mirror the meta CSP on both sites.

### Cache policy (updated 2026-11-06)
- **Website:** CSS is `max-age=31536000, immutable` — so stylesheet links carry
  a version query (`assets/css/*.css?v=2`). **When CSS changes, bump the
  version** (`v=2` → `v=3`) on every page or returning visitors will keep the
  old styles for a year. JS is 1-hour cache (acceptable).
- **Admin:** JS and CSS are explicitly **`no-cache`** (headers added after the
  "Cannot access 'downloadCount' before initialization" fix was hidden by a
  stale cached dashboard.js). HTML is also `no-cache`. Admin traffic is tiny,
  so freshness beats caching here.

### API key protection (recommended, operator task)
Google Cloud Console → APIs & Services → Credentials → web API key →
HTTP referrers restriction:
`panijog.web.app`, `panijog-admin-cp.web.app`, `localhost` (for testing).

### AI Studio files (protected)
`doc/firebase.json`, `doc/firebase-blueprint.json`, `doc/firebase.rules` came
from Google AI Studio. `doc/firebase.json` targets a separate AI Studio
database — **never** overwrite production config with it.

---

## 9. Firestore collections reference

| Collection | Access | Purpose |
| --- | --- | --- |
| `users/{userId}` | owner; Support+ can read; SuperAdmin delete | App profile: userId, displayName, dailyGoalMl (500–10000), wake/sleepTimeMinutes |
| `users/{userId}/intakes/{id}` | owner; Support+ read | Intake record: amountMl 1–5000, category, timestamp (int), dateString (10 chars) |
| `adminProfiles/{uid}` | read self/admins; SuperAdmin write | role (SuperAdmin/Admin/Support/Analyst), status (active), permissions, takes audit fields |
| `appConfig/{key}` | any admin read; SuperAdmin write | Settings incl. `main.downloadCount` |
| `releases/{id}` | Admin+ write, SuperAdmin delete | Release metadata |
| `announcements/{id}` | Admin+ | In-app announcements |
| `feedback/{id}` | create any signed-in; Support+ read/update (status, priority, adminNotes, resolvedAt); SuperAdmin delete | User feedback |
| `supportCampaigns/{id}` | Admin+ write; SuperAdmin delete | Support campaign metadata |
| `analyticsAggregates/{id}` | any admin read; SuperAdmin write | Aggregate stats |
| `auditLogs/{id}` | Admin create (self-actions only, validated fields); **no update/delete** | Append-only audit trail |
| `deletionRequests/{id}` | create any signed-in; Support+ update; SuperAdmin delete | Account deletion workflow |
| `contactMessages/{id}` | **create only** (validated: name 2–80, email ≤120 + regex, topic enum, message 10–2000, page ≤120); Admin+ read; SuperAdmin update/delete | Website contact form |
| *(everything else)* | `match /{document=**} allow read, write: if false` | Deny by default |

Owner-scoped writes validate field types and ranges in rules — the app cannot
write malformed documents even if the client is compromised.

---

## 10. Admin authorization (RBAC) and Super Admin setup

- Admin status = **existence** of `adminProfiles/{uid}` (no custom claims —
  Spark plan has no Cloud Functions).
- Role rules: `isSuperAdmin()`, `isAdminOrAbove()` (Admin/SuperAdmin),
  `isSupportOrAbove()` (Support/Admin/SuperAdmin), plus any signed-in admin
  (`isAdmin()`) for Analyst-level reads.
- Privileged Auth-level actions (deleting a Firebase Auth user) are done in
  the Console; everything else works from the panel.

### Bootstrap the first Super Admin (operator steps)
1. Firebase Console → **Authentication** → Users → **Add user** (email +
   password). Copy the UID.
2. Firebase Console → **Firestore** → start collection `adminProfiles` →
   document **ID = UID** with fields:
   - `role: "SuperAdmin"`
   - `status: "active"`
   - `permissions: ["*"]`
   - `invitedBy: null` (optional)
   - `createdAt` / `updatedAt`: server timestamp
3. Sign in at <https://panijog-admin-cp.web.app/login.html> → Dashboard.
4. Create other admins from the **Admins** page (their Auth user must exist
   first; the Auth user itself is deleted only from the Console).

### Login troubleshooting checklist
If login fails after entering valid credentials, check in order:
(a) Firestore database exists (created in Console),
(b) rules are deployed (`firebase deploy --only firestore:rules`),
(c) `adminProfiles/{uid}` exists with `status: "active"`,
(d) API key referrers allow the admin domain (403 symptom).

---

## 11. APK metadata and distribution

| Field | Value |
| --- | --- |
| Local file | `doc/Panijog.apk` (canonical copy) |
| Size | 30,312,173 bytes (28.9 MiB / 30.4 MB) |
| Version | 1.0 (version code 1) |
| Package | `com.hanafi.panijog.hytdrk` |
| Min Android | 7.0 (API 24) |
| Release date | 6 November 2026 |
| Download URL | `https://github.com/alhanafib/panijog/releases/tag/android-app` |
| SHA-256 | `74bc9e04aba20486c7171c7b621ab2c46a6622e0a5111f48c08e7f8dcf554355` |

Verify: `Get-FileHash doc\Panijog.apk -Algorithm SHA256`.

If a new APK ships: replace `doc/Panijog.apk`, attach the asset to the GitHub
release, and update version/size/date/SHA-256 on `download.html`,
`changelog.html`, `about.html`, and README/docs. Search the repo for the old
64-hex string to catch every place.

Distribution path: website `download.html` → GitHub release page → `Panijog.apk`
asset (Spark plan cannot serve executable files from Hosting; hosting headers
still include `*.apk` rules for future-proofing but no APK is deployed there).

---

## 12. Local preview and QA tooling

### Quick static servers
```powershell
# website (port 4173)
cd "website\public"; npx serve -l 4173 .   # or python -m http.server 4173

# admin (port 4174)
cd "admin\public"; npx serve -l 4174 .     # login at http://127.0.0.1:4174/login.html
```

The temp QA scripts (path `C:\Users\RUBEL\AppData\Local\Temp\opencode\qa\`)
are disposable but reusable: `site-server.js` (site → 4173), `admin-server.js`
(admin → 4174, SPA-ish fallback to 404 page), `all-console.js` /
`admin-console.js` (headless Chrome console error sweep with puppeteer-core),
`overflow2.js` (responsive overflow check at 320/375/430/768/1024/1440),
audit scripts. They may be deleted anytime; they are not project files.

### HTML validation (must pass)
```powershell
cd "website\public"; npx --yes html-validate@9.7.1 "*.html"
cd "admin\public";   npx --yes html-validate@9.7.1 "*.html"
```

### JS syntax
```powershell
node --check assets/js/main.js          # website
node --check <file>                     # every admin public JS module
```

---

## 13. Validation checklist (latest full pass)

| Check | Website | Admin |
| --- | --- | --- |
| html-validate | 0 errors (17 pages) | 0 errors (17 files) |
| Console errors | 0 | 0 (favicon fixed on 403/404/index) |
| JS syntax | pass | pass (20 modules) |
| Horizontal overflow (320–1440) | none | none |
| Theme toggle | right side of header | right side of header |
| Fonts | self-hosted, Bangla→Hind Siliguri | self-hosted, Bangla→Hind Siliguri |
| Layout | floating glass header | sidebar left + topbar right (fixed) |

### Known layout pitfalls (do not regress)
1. **Admin shell grid:** `.app-shell` uses
   `grid-template-columns: var(--sidebar-w) 1fr`. Every child of `.app-shell`
   becomes a grid item. The `sidebar-backdrop` div must stay **outside**
   `.app-shell` (sibling before it) or the desktop layout breaks (sidebar
   shifts to the second column, header wraps below).
2. `.sidebar-backdrop { display: none; }` base rule (desktop); the ≤900px
   media query turns it into the mobile drawer backdrop.
3. `.sidebar` toggle button (☰) visible at all widths; desktop click collapses
   the whole sidebar column (`.app-shell.sidebar-collapsed`), mobile click
   opens the drawer.
4. `min-width: 0` on grid/flex children + `overflow-wrap` on long strings
   (hashes, emails) to prevent overflow.
5. **Login shows "Cannot access 'downloadCount' before initialization"** —
   fixed in code on 2026-11-06 (the `appConfig/main` read now runs before the
   KPI grid builds). If it still shows, the browser cached the old JS: hard
   refresh (Ctrl+F5). New visits are safe because admin assets are `no-cache`.
6. The `/403` URL pattern: the local admin server and Firebase Hosting
   `cleanUrls` both map it — keep `403.html` present.

---

## 14. Deployment procedures

**Only deploy after explicit user authorization.**

```powershell
# 1) Rules (once per rules change) — from repo root:
cd "Panijog app"
firebase deploy --only firestore:rules --project gen-lang-client-0297092818

# 1b) With indexes — from admin/:
cd admin
firebase deploy --only firestore:rules firestore:indexes --project gen-lang-client-0297092818

# 2) Website:
cd website
firebase deploy --only hosting:panijog --project gen-lang-client-0297092818
# → https://panijog.web.app

# 3) Admin:
cd admin
firebase deploy --only hosting:panijog-admin-cp --project gen-lang-client-0297092818
# → https://panijog-admin-cp.web.app
```

Rules deploy requires the root `firebase.json` (single firestore block,
`doc/firestore.rules`). Note: the Firebase CLI refuses rules outside the CLI
project directory, which is why every site config points at `../doc/…` and why
the root config exists.

### Auth Firebase CLI account
The local CLI was signed in with `muahanafib10403@gmail.com` (owner).

---

## 15. Current live status and remaining to-dos

### Live (deployed 6 Nov 2026)
- Website: <https://panijog.web.app/> — 17 pages, current fixes live.
- Admin: <https://panijog-admin-cp.web.app/> — layout + sidebar fixes live.
- Firestore rules: deployed (compiled clean after removing BOM and a stray
  `EOF` line).

### Operator to-dos (Firebase Console)
1. Verify Firestore database exists (Production mode, `asia-south1`).
2. Create the first Super Admin (steps in §10) — `adminProfiles/{uid}` with
   role/status/permissions.
3. Restrict the web API key referrers (§8).
4. Admin → Settings → "APK download count" → set (e.g., 10) → Save.
5. Read contact messages in Firestore Console (`contactMessages`).

### Honest limitations
- Analytics pages show empty states until real aggregates exist — this is by
  design (no fake data).
- Download counter is manual until server-side counting exists.
- ` Analyst` role works only for data reads; no separate Analyst UI beyond
  analytics/audit access.

---

## 16. Content editing / common tasks

| Task | Where |
| --- | --- |
| Change a color/token | website `assets/css/base.css` or admin `assets/css/style.css` `:root` |
| Add a website page | copy an existing HTML page, update header/footer active link, add to `sitemap.xml` |
| Add a help article | new `help-<slug>.html` + card on `help.html` + prev/next pager row + sitemap entry |
| Update APK metadata | `download.html`, `changelog.html`, `about.html`, README, this guide (§11) |
| Edit contact form | website `assets/js/contact.js` + `contactMessages` rules (§9) |
| Add admin nav item | `admin/public/assets/js/shell.js` NAV + ICONS, `boot.js` TITLE/MIN_ROLE/MODULE, new page HTML + module |
| Replace app screenshots | `website/public/assets/screenshots/` (7 JPEGs referenced in the home gallery + preview section) |
| Deploy rules | §14 commands |
| Deploy site (authorized) | §14 commands |

### Conventions recap
- English content, Bangla accents; `bn` uses Hind Siliguri
- Inline SVG icons only; no emoji
- No inline styles/scripts; no inline event handlers
- `<!DOCTYPE html>` uppercase; meta CSP; relative asset paths
- Every page: honest content, matching nav/footer, `sitemap.xml` coverage
- Footer copyright: "© 2026 Panijog. All rights reserved. Developed by
  Mohammadullah Al Hanafi." linked to the developer site

---

*End of guide. Keep this file in sync in both `doc/PROJECT_GUIDE.md` and
`github/PROJECT_GUIDE.md`.*
