# ReMynder

Personal task management app with Google authentication. Built with vanilla web technologies — no frameworks, no build step.

**Live:** [pendientes-six.vercel.app](https://pendientes-six.vercel.app)

---

## Stack

| Layer    | Technology                                  |
|----------|---------------------------------------------|
| Frontend | HTML + CSS + JavaScript (vanilla, no build) |
| Auth     | Firebase Authentication — Google OAuth 2.0  |
| Database | Cloud Firestore (NoSQL, real-time)          |
| Hosting  | Vercel                                      |
| Fonts    | DM Sans + DM Mono (Google Fonts)            |

No backend. No npm. No bundler. A single `index.html` the browser runs directly.

---

## Project Structure

```
reMynder/
├── web/
│   ├── index.html        # Full app: HTML + CSS + JS in one file
│   └── assets/
├── docs/
│   └── documentacion.md  # Technical documentation
├── .agents/              # Agent context and task files
├── .github/
└── CLAUDE.md             # Shared context for all agents working on this project
```

---

## Local Development

Firebase Auth blocks `file://` — run a local HTTP server:

```bash
python3 -m http.server 3000
# or
npx serve .
```

Then add `localhost` to **Authorized Domains** in Firebase Console.

---

## Dev → Prod Workflow

**As of 2026-07-20, this project has automatic Git-triggered deploys.** Vercel's project is
connected to GitHub (`m4verick-uy/reMynder`) with **Production Branch = `main`**. Every push
triggers a build automatically — no manual `vercel`/`vercel --prod` needed anymore.

- **`develop`** branch — work here first. Every push auto-deploys a Preview at
  **https://remynder-dev.vercel.app** (registered as a project Domain bound to `gitBranch: develop`
  via `POST /v10/projects/{id}/domains`, so it always tracks the latest `develop` deploy with no
  manual re-aliasing).
- **`main`** branch — production. Merging `develop` → `main` and pushing auto-deploys
  `remynder.vercel.app`.
- Both branches share the same Firebase project (`pendientes-2c0ea`) and Firestore database — no
  data isolation between dev and production. Anything you do at `remynder-dev.vercel.app` affects
  real task data.
- `remynder-dev.vercel.app` is added to Firebase **Authorized Domains** so Google login works there.
- **Before connecting Git integration, always double-check Settings → Git → Production Branch is
  `main`.** If it were ever misconfigured to `develop`, every dev push would deploy straight to
  production.

```bash
git checkout develop
# make changes, commit, push
git push origin develop
# Vercel auto-deploys — open remynder-dev.vercel.app, verify in browser

# when satisfied:
git checkout main
git merge develop
git push origin main   # Vercel auto-deploys to remynder.vercel.app
```

---

## Roadmap

- [ ] User-defined categories (currently hardcoded)
- [ ] Task editing
- [ ] Due dates and reminders
- [ ] Native iOS app
- [ ] Freemium model

ReMynder is the flagship product of a software company being built in Uruguay, aimed at Apple-quality personal productivity tools.

---

## License

Private — all rights reserved.
