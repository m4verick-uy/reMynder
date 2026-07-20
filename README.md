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

This project has no automatic Git-triggered deploys (there's no Vercel↔GitHub webhook
connected) — deploys have always been manual via the Vercel CLI. The dev environment follows
the same pattern:

- **`develop`** branch — work here first. Deploy with `vercel` (no flag) to push a **preview**
  deployment, then run `vercel alias set <preview-url> remynder-dev.vercel.app` once to keep a
  stable, bookmarkable dev URL: **https://remynder-dev.vercel.app**. Subsequent `vercel` runs
  need the alias re-pointed the same way if you want `remynder-dev.vercel.app` to reflect the
  latest preview.
- **`main`** branch — production. Merge `develop` → `main`, then run `vercel --prod` to deploy to
  `remynder.vercel.app`.
- Both branches share the same Firebase project (`pendientes-2c0ea`) and Firestore database — no
  data isolation between dev and production. Anything you do at `remynder-dev.vercel.app` affects
  real task data.
- `remynder-dev.vercel.app` is added to Firebase **Authorized Domains** so Google login works there.

```bash
git checkout develop
# make changes, commit
vercel                # deploy preview
vercel alias set <preview-url-from-output> remynder-dev.vercel.app
# open remynder-dev.vercel.app, verify in browser

# when satisfied:
git checkout main
git merge develop
git push origin main
vercel --prod         # deploy to remynder.vercel.app
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
