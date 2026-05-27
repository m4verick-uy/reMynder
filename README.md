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
