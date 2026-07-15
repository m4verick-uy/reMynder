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

- **`develop`** branch — work and push here first. Vercel auto-deploys every push to a stable preview URL: `remynder-git-develop-m4vericks-projects.vercel.app`.
- **`main`** branch — production. Merging `develop` → `main` deploys to `remynder.vercel.app`.
- Both branches share the same Firebase project (`pendientes-2c0ea`) and Firestore database — there is no data isolation between preview and production. Test changes carefully; anything you do in the preview affects real task data.
- The preview URL is added to Firebase **Authorized Domains** so Google login works there.

```bash
git checkout develop
# make changes, commit, push
git push origin develop
# open remynder-git-develop-m4vericks-projects.vercel.app, verify in browser

# when satisfied:
git checkout main
git merge develop
git push origin main
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
