# Panijog — Project Guide

> **Comprehensive developer onboarding document.** If you are an AI agent or a new
> developer picking up this project, read this file first. It describes the
> product, the repository, the architecture, the conventions, and the exact steps
> to build, preview, and deploy everything. A short overview lives in
> [`README.md`](./README.md); licensing in [`LICENSE`](./LICENSE).

---

## 1. What Panijog is

Panijog is an Android hydration-tracking app plus a static marketing website.

- **App:** native Android, Kotlin + Jetpack Compose, Room, Hilt, Firebase Auth +
  Firestore, Firebase App Check. Package `com.hanafi.panijog.hytdrk`.
  Version **1.0** (version code 1), released **6 November 2026**, minSdk 24
  (Android 7.0+), APK 30,351,362 bytes (28.9 MiB).
- **Website:** static HTML/CSS/vanilla JS hosted on Firebase Hosting at
  `https://panijog.web.app/`. No framework, no build step, no third-party
  scripts or fonts.
- **Developer:** Mohammadullah Al Hanafi (https://mohammadullah.web.app/).
- **Support/tip jar:** https://www.supportkori.com/hanafi (a "buy me a coffee"
  page, **not** a support desk).
- **Contact:** in-app / website contact form → private Firebase Firestore
  collection `contactMessages`.

The product philosophy: calm, local-first, honest scope. The website deliberately
separates features that ship in the APK from features that are only planned.

---

## 2. Repository layout

```
Panijog app/                          (repo root)
├── README.md                         short overview (this project)
├── LICENSE                           proprietary, all rights reserved
├── PROJECT_GUIDE.md                  this file
├── logo.png / app_icon.png           canonical brand assets (DO NOT edit)
├── doc/                              app-side notes & prompts (DO NOT edit)
│   ├── app_prompt.txt
│   ├── website_prompt.txt
│   ├── firebase.rules                the APP's rules (DO NOT edit)
│   ├── notes.txt, command.cmd, admin_panal_prompt.txt
├── admin/                            admin panel (separate, do not touch)
├── logo/                             extra brand files
└── website/                          EVERYTHING web-related lives here
    ├── index.html, features.html, download.html, about.html,
    │   contact.html, privacy-policy.html, terms.html, changelog.html,
    │   404.html, help.html, help-*.html
    ├── Panijog.apk                   the released APK (served for download)
    ├── cat-computer.gif              404 animation
    ├── robots.txt, sitemap.xml, site.webmanifest, .firebaserc, firebase.json
    ├── README.md                     website-specific notes
    ├── assets/
    │   ├── css/  base.css, components.css, pages.css, fonts.css
    │   ├── js/   main.js, contact.js, firebase-config.js
    │   ├── fonts/ 32 self-hosted WOFF2 (Plus Jakarta Sans, Hind Siliguri)
    │   ├── img/  logo, app_icon, favicons, og-image.png
    │   ├── icons/ (reserved)
    │   └── screenshots/ (empty; drop real screenshots here)
    └── firebase/
        └── firestore.rules           deployable rules (app + contactMessages)
```

### Hard rules (never break these)

- All website work happens **only** inside `website/`.
- Never edit `doc/`, `admin/`, root `logo.png`/`app_icon.png`, or
  `doc/firebase.rules`. Deployable rules live in `website/firebase/firestore.rules`.
- Never deploy (`firebase deploy`) without explicit user authorization.
- Never invent stats, ratings, testimonials, download counts, or release dates.
- No React/Vue/Angular/Next. HTML5 + CSS3 + vanilla JS ES6+. Inline SVG icons
  only; **no emoji icons**.
- Site content is English with Bangla brand accents (no bilingual toggle).

---

## 3. Tech stack

### App (not in this repo's website folder)
- Kotlin, Jetpack Compose, Room, Hilt
- Firebase Auth (Google), Cloud Firestore, Firebase App Check
- WorkManager-style local scheduling for reminders
- No Crashlytics, no Firebase Analytics, no ads SDK in 1.0

### Website
- Static HTML, CSS, vanilla JS. No build step.
- Fonts self-hosted (downloaded WOFF2 + `fonts.css`) — no Google Fonts request
  at runtime.
- CSP via `<meta>` tags and `firebase.json` headers (no inline scripts/styles).
- Firebase Hosting (site ID `panijog`), Firestore REST for the contact form.

---

## 4. App features

### Available in 1.0 (verified against the APK)
Daily dashboard with wave progress; quick + custom logging; 7 drink categories
(water, tea, coffee, milk, fresh juice, coconut water, other); daily goal with a
suggested estimate; reminders with quiet hours; boot rescheduling; 7-day
analytics (average + beverage breakdown); streaks, badges, virtual garden;
offline-first Room storage; Google sign-in + Firestore sync (account-scoped
rules); JSON/CSV export; account deletion; in-app privacy/terms; English +
Bangla; automatic dark theme.

### Planned, not in 1.0 (do not claim these ship)
Health Connect, weather suggestions, widgets, Quick Settings tile, Wear OS,
voice logging, family/multi-profile, biometric app lock, privacy dashboard,
calendar/history, custom vessel presets, snooze/skip, exact-time reminders,
crash reporting, Firebase Analytics, import/restore, remote push.

The website's `features.html` is the source of truth for this split.

### Permissions (from the manifest)
Internet + network state, notifications (optional), receive boot completed.
No location, microphone, camera, contacts, or broad storage. Biometric hardware
permissions are declared via AndroidX biometric but no biometric lock screen is
active in 1.0.

---

## 5. Website pages

| File | Purpose |
| --- | --- |
| `index.html` | Home: hero, features, how-it-works, CSS previews, FAQ, CTA |
| `features.html` | Available vs. clearly marked planned work |
| `download.html` | APK metadata, install steps, checksum, permissions |
| `help.html` | Help center index (cards) |
| `help-install.html` … `help-troubleshooting.html` | Individual help articles |
| `changelog.html` | Released versions only |
| `about.html` | Principles, stack, release facts |
| `contact.html` | Firebase-backed contact form + support links |
| `privacy-policy.html` | Data practices |
| `terms.html` | Licence, health disclaimer, liability |
| `license.html` | All-rights-reserved license page |
| `404.html` | Branded not-found page with `cat-computer.gif` |

Navigation (header) on every page: Home, Features, Download, Help, About,
Contact. Footer has Explore (incl. Help center), Legal, Support (Buy me a
coffee / Developer website / Send a message).

---

## 6. Design system (CSS)

Three stylesheets, loaded in order:

- `assets/css/fonts.css` — self-hosted `@font-face` (Plus Jakarta Sans, Hind
  Siliguri).
- `assets/css/base.css` — tokens, reset, typography, layout primitives,
  accessibility, motion, print, **custom scrollbars**.
- `assets/css/components.css` — buttons, badges, cards, header/footer, phone
  mockup, forms, FAQ, timeline, callout, reveal.
- `assets/css/pages.css` — page-level layouts (hero, download, legal, contact,
  about, changelog, 404, help pager).

Key tokens: `--brand` #0d63e8, `--cyan` #0891b2, `--bg` #f2f7ff, dark scheme
via `prefers-color-scheme`. Fonts: `--font-sans` (Plus Jakarta Sans),
`--font-bn` (Hind Siliguri). Container max width 1160px. Header is a floating
glass pill (sticky, rounded, blurred).

Class conventions: `.reveal` for scroll-in animation, `.card`, `.btn`,
`.phone`/`.mockup-label` for CSS phone previews labelled "Illustrative
preview". No inline styles or scripts anywhere.

---

## 7. JavaScript

- `assets/js/main.js` — sticky/floating header state, mobile nav (Escape to
  close, focus management), footer year, reveal-on-scroll (IntersectionObserver),
  copy-to-clipboard buttons, date formatting. No dependencies.
- `assets/js/contact.js` — ES module. Validates the form, honeypot check,
  45s cooldown via localStorage, POSTs to the Firestore REST API, honest
  success/error states. Imports `firebase-config.js`.
- `assets/js/firebase-config.js` — public Firebase web config (not a secret).

---

## 8. Firebase configuration

- Project ID: `gen-lang-client-0297092818`
- Web API key: `AIzaSyDrvaz-dW6R2M83VFkg16gxzeTX2zkSKo0`
- App ID: `1:178323036143:web:f8207b796fcf56d7ef481a`
- Auth domain: `gen-lang-client-0297092818.firebaseapp.com`
- Hosting site ID: `panijog` → `panijog.web.app`
- Firestore region (recommended): `asia-south1`

These identifiers are public by design; access is enforced by rules, not by
hiding the file.

### Firestore rules (`website/firebase/firestore.rules`)

Two rule sets:

1. App rules — user-scoped `users/{userId}` and `users/{userId}/intakes/{id}`,
   validated field types/ranges, owner-only read/write/delete.
2. Website contact — `contactMessages/{id}` allows **create only**, with field
   types and length limits (name 2–80, email ≤120 + regex, topic in enum,
   message 10–2000, page ≤120). Read/update/delete denied to all clients.
   Messages are read in the Firebase Console.

### Hosting config (`website/firebase.json`)

- Serves the directory root (`.`), ignores tooling files.
- Clean URLs, no trailing slash.
- Security headers: CSP, X-Content-Type-Options, X-Frame-Options, Referrer-Policy,
  Permissions-Policy, COOP, HSTS.
- Cache: HTML `no-cache`; CSS/fonts/images 1 year immutable; JS 1 hour;
  APK 1 day with forced download content-type.

### Backend status (as of last build)

- The project had **no Firestore database** yet.
- `panijog.web.app` had **no Hosting site deployed**.
- The local Firebase CLI account (`muahanafib10403@gmail.com`) had **no access**
  to the project.

To finish the backend, sign in to the Firebase Console with the project owner and:

1. Create Firestore: Firestore Database → Create database → Production mode →
   location `asia-south1`.
2. Create Hosting site: Hosting → Add site → site ID `panijog`.
3. Deploy rules: `cd website` → `firebase login:add` (owner account) →
   `firebase deploy --only firestore:rules --project gen-lang-client-0297092818`.
4. Deploy site (only when authorized):
   `firebase deploy --only hosting --project gen-lang-client-0297092818`.

If browser requests to Firestore fail with 403 because of API-key restrictions,
open Google Cloud Console → APIs & Services → Credentials → the web API key, and
allow `panijog.web.app` (and `localhost` for testing) as HTTP referrers.

---

## 9. Contact form data flow

1. User fills the form on `contact.html`.
2. `contact.js` validates (name, email, topic, message, consent checkbox).
3. Honeypot field `website` must be empty; otherwise silently drop.
4. 45s cooldown via `localStorage` (`panijog.contact.lastSent`).
5. POST to
   `https://firestore.googleapis.com/v1/projects/{projectId}/databases/(default)/documents/contactMessages?key={apiKey}`
   with fields `name`, `email`, `topic`, `message`, `page`.
6. Firestore adds `createTime` metadata.
7. Success: honest success message. 403: points to the buy-me-a-coffee / help
   links. The site never reads messages back.

---

## 10. APK metadata (must stay in sync with the file in `website/`)

| Field | Value |
| --- | --- |
| File | `Panijog.apk` |
| Size | 30,351,362 bytes (28.9 MiB / 30.4 MB) |
| Version | 1.0 (version code 1) |
| Package | `com.hanafi.panijog.hytdrk` |
| Min Android | 7.0 (API 24) |
| Release date | 6 November 2026 |
| Download URL | `https://github.com/alhanafib/panijog/releases/tag/android-app` |
| SHA-256 | `401f029785a7fb0e7c4200c5393a1d694436a6a9d7431bf76dcac61af1b75136` |

If a new APK replaces it, update version, size, date, and SHA-256 on
`download.html`, `changelog.html`, `about.html`, and `README.md`.

---

## 11. Local development & preview

Any static server works:

```powershell
cd "E:\aworkbyhanafi\Panijog app\website"
python -m http.server 8080
# open http://localhost:8080/
```

or

```powershell
npx serve .
```

Opening `index.html` via `file://` mostly works, but module scripts and clipboard
permissions behave better over `http://localhost`.

For deterministic screenshots at specific widths, use headless Chrome:

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --screenshot=out.png --window-size=1440,900 http://localhost:8080/
```

---

## 12. Validation commands

```powershell
cd website
# HTML validation
npx html-validate@9.7.1 "*.html"

# JS syntax
node --check assets/js/main.js
node --check assets/js/contact.js

# Custom audit (missing targets, duplicate ids, CSP, viewport, JSON validity)
node C:\Users\RUBEL\AppData\Local\Temp\opencode\audit-site.js
```

Responsive check widths: 320, 375, 430, 768, 1024, 1440 px — no horizontal
overflow. Keyboard accessible, `prefers-reduced-motion` respected.

---

## 13. Conventions

- **No fake data.** Never add ratings, download counts, testimonials, release
  dates, or "Play Store" claims.
- `features.html` is the source of truth: "Available now" = shipped in the APK;
  everything else is "Planned".
- Download page must match the APK actually in the folder.
- Do not edit `doc/firebase.rules`, `doc/*`, `admin/*`, or root brand images.
- All website files stay in `website/`; app files stay in `doc/`/app project.
- Icons are inline SVG; no emoji icons.
- Fonts are self-hosted; no Google Fonts request at runtime.
- HTML uses `<!DOCTYPE html>` uppercase, meta CSP, relative asset paths.

---

## 14. Content editing notes

- Help center articles live in `help-*.html` (one per article) plus the index
  `help.html`. Internal links use real file names, not anchors.
- To add a new help article: create `help-<slug>.html` from an existing one,
  add a card on `help.html`, add a row to the help pager (prev/next), and add
  the URL to `sitemap.xml`.
- CSS phone mockups are labelled "Illustrative preview". Replace with real
  screenshots from `assets/screenshots/` when available (see its README).
- The footer copyright reads "© 2026 Panijog. All rights reserved. Developed by
  Mohammadullah Al Hanafi." (linked to the developer website).

---

## 15. Files that may be deleted / are optional

- `assets/icons/` — empty, reserved for future SVG assets.
- `assets/screenshots/` — empty until real screenshots are added (keep README).
- Temp files in the OS temp dir are not part of the project.

Do not delete `Panijog.apk`, `cat-computer.gif`, fonts, or any page listed above.

---

## 16. Quick reference: common tasks

| Task | Where |
| --- | --- |
| Change a color/token | `assets/css/base.css` `:root` |
| Add a new page | Copy an existing page, update nav/footer active state, add to sitemap |
| Update APK metadata | `download.html`, `changelog.html`, `about.html`, `README.md` |
| Edit contact form | `assets/js/contact.js` + `firebase/firestore.rules` |
| Deploy rules | `firebase deploy --only firestore:rules` |
| Deploy site | `firebase deploy --only hosting` (needs authorization) |

---

*Last updated: 6 November 2026. Maintained alongside version 1.0.*


---

## Admin Panel

A separate admin dashboard lives in `admin/`. It is a static site (HTML/CSS/JS)
plus Firestore rules and indexes. It shares the same Firebase project but uses a
separate Hosting site `panijog-admin-cp` (`panijog-admin-cp.web.app`).

### Authorization model (no Cloud Functions / no Blaze)
- Admin status = existence of `adminProfiles/{uid}` document.
- Role read via `get()` in Firestore rules: SuperAdmin / Admin / Support / Analyst.
- Privileged Auth-level actions (disable Firebase Auth user) require the Console
  or a local Admin SDK script.
- Audit logs are append-only.

### Pages
`login.html`, `dashboard.html`, `users.html`, `user-details.html`,
`releases.html`, `campaigns.html`, `analytics.html`, `feedback.html`,
`notifications.html`, `content.html`, `audit-logs.html`, `settings.html`,
`admins.html`, `profile.html`, `403.html`, `404.html`.

### Theme
White / dark / system toggle (default system), persisted in localStorage.

## Website updates (latest)

- Google site verification meta tag added to all pages.
- Theme toggle (white/dark/system) added to header.
- Floating glass header.
- GitHub repository + GitHub Releases download links added to download page and footer.
- SHA-256 updated to `401f029785a7fb0e7c4200c5393a1d694436a6a9d7431bf76dcac61af1b75136`
  (APK size 30,351,362 bytes / 28.9 MiB).
- `public/` folder reorganization: deployable files in `website/public/`,
  config/rules/docs outside.
- Root leftover `firebase.json`, `.firebaserc`, `.firebase/` removed.
- `license.html` page added; footer links to it.
- Help center split into index + 6 article pages.

## File organization rule
- Website deployable files → `website/public/`.
- Website config/rules/docs → `website/` (outside public).
- Admin deployable files → `admin/public/`.
- Admin config/rules/functions/docs → `admin/` (outside public).
- App files → `doc/` / app project.


---

## Unified Firestore Rules (app + website + admin)

As of the latest update, the **same unified rules file** is used for all three
components (app, website, admin panel) because they share one Firebase project
(`gen-lang-client-0297092818`). The canonical copy lives in three places (kept
in sync):

- `website/firebase/firestore.rules` (deployable via website firebase.json)
- `admin/firestore.rules` (deployable via admin firebase.json)
- `doc/firestore.rules` (reference copy, do not delete)

The rules cover:
- App: `users/{userId}`, `users/{userId}/intakes/{intakeId}` (owner-scoped).
- Website: `contactMessages/{id}` (create-only, validated).
- Admin: `adminProfiles`, `appConfig`, `releases`, `announcements`,
  `feedback`, `supportCampaigns`, `analyticsAggregates`, `auditLogs`
  (append-only), `deletionRequests`.
- Admin authorization via `adminProfiles/{uid}` existence + role (no Cloud
  Functions / no Blaze plan).

### Google AI Studio files in doc/
The following files were added from Google AI Studio and are kept as reference:
- `doc/firebase-blueprint.json` — entity schema (UserProfile, IntakeRecord).
- `doc/firebase.json` — points to a separate AI Studio database
  (`ai-studio-android-panijog-...`); **not** the production database.
  Do not overwrite the production `firebase.json` files with this.
- `doc/README.md`, `doc/LICENSE` — copies for the doc folder.

### Notes on the production database
Production project: `gen-lang-client-0297092818`. Firestore region recommended:
`asia-south1`. Hosting sites: `panijog` (website) and `panijog-admin-cp`
(admin).


---

## Website Structure (public/ folder)

Deployable files live in `website/public/`. Config/rules/docs stay in `website/`:

```
website/
├── public/              <- Firebase Hosting public dir
│   ├── index.html … license.html, 404.html
│   ├── Panijog.apk, cat-computer.gif
│   ├── robots.txt, sitemap.xml, site.webmanifest
│   └── assets/ (css, js, fonts, img, screenshots)
├── firebase/
│   └── firestore.rules  <- unified (app + website + admin)
├── firebase.json        (public = "public")
├── .firebaserc
└── README.md
```

## Website Features (latest additions)

- **Theme toggle**: white / dark / system (default system), persisted in localStorage, in header.
- **Floating glass header**: sticky rounded glass pill.
- **Google site verification**: meta tag on all pages.
- **GitHub links**: footer (all pages) + download page "Download from GitHub (recommended)" button.
- **SHA-256**: `401f029785a7fb0e7c4200c5393a1d694436a6a9d7431bf76dcac61af1b75136` (APK 30,351,362 bytes / 28.9 MiB).
- **License page**: `license.html`.
- **Help center**: index + 6 article pages with pager.

## Admin Panel

Location: `admin/`. Static site + unified Firestore rules + indexes.

- **Hosting site**: `panijog-admin-cp` → `panijog-admin-cp.web.app`.
- **Auth**: Firebase Auth email/password; admin status = `adminProfiles/{uid}` existence.
- **RBAC**: SuperAdmin / Admin / Support / Analyst via Firestore rules (no Cloud Functions).
- **Theme**: white/dark/system toggle (default system).
- **Pages**: login, dashboard, users, user-details, releases, campaigns, analytics, feedback, notifications, content, audit-logs, settings, admins, profile, 403, 404.
- **UI**: floating glass header, sidebar, data tables, KPI cards, modals, toasts, skeleton/empty/error states, responsive 320–1440.

### How to create a Super Admin
1. Firebase Console → Authentication → Add user (email + password). Note the UID.
2. Firestore → `adminProfiles` collection → document ID = UID → fields:
   `role: "SuperAdmin"`, `status: "active"`, `permissions: ["*"]`,
   `invitedBy: null`, `createdAt: serverTimestamp()`.
3. Login at `panijog-admin-cp.web.app/login.html`.
4. From Admins page, create other admin profiles (user must exist in Auth first).

## Super Admin creation (summary)
Firebase Console → Auth (add user) → Firestore `adminProfiles/{uid}` with role SuperAdmin/active → login.

## File organization rule
- Website deployable → `website/public/`.
- Website config/rules/docs → `website/`.
- Admin deployable → `admin/public/`.
- Admin config/rules/docs → `admin/`.
- App files → `doc/` / app project.
- No unnecessary duplicates; root `Panijog.apk` and `logo/` are kept as working copies.

## Validation
- Website: 17 pages, html-validate 0 errors, console 0 errors, no horizontal overflow.
- Admin: 15 pages, html-validate 0 errors, console 0 functional errors.
- Unified rules: 3 locations, identical.


## UI/UX notes (latest)
- Theme toggle moved to the right side of the header (website + admin).
- Toast notifications replaced with professional centered popup dialogs (admin panel).
- Admin login page redesigned (centered card, focus rings, loading state).
- Admin shell: sidebar flex column, aligned topbar/content, responsive tables.


## Latest updates
- Admin dashboard now shows an APK download counter (manual, set in Settings).
- Admin panel fonts (Plus Jakarta Sans, Hind Siliguri) are now self-hosted (copied from website).
- Theme toggle moved to the right side of the header (website + admin).
- Toast notifications replaced with professional centered popup dialogs (admin).


## Screenshots
Real app screenshots (7 JPEGs) are in `website/public/assets/screenshots/` and displayed in a gallery on the home page (`index.html`). Filenames: 01-home-dashboard, 02-add-water, 03-analytics, 04-achievements, 05-settings, 06-settings, 07-authentication.
