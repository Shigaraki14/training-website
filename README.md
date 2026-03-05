## AI + MuleSoft Enterprise Training – Master Reference

This project is a single‑page, self‑study reference site for a **30‑day AI + MuleSoft enterprise training curriculum**.  
It is implemented as a static HTML application with rich in‑browser behavior (notes, progress tracking, auth overlay).

---

### 1. Project Structure

- **`index.html`**
  - Full SPA containing:
    - Sidebar navigation (phases, days, study progress bar)
    - Main content (training days, topics, quizzes, notes)
    - Quick Reference Glossary panel
    - Theme toggle (dark/light)
    - Keyboard shortcuts (`/`, `J/K`, `G`, `Esc`)
    - LocalStorage‑backed notes and completion tracking
    - Scroll progress bar and “Back to top” button
  - Embedded **Auth Overlay**:
    - Login, Signup, and Forgot Password flows
    - Uses **Firebase** (Auth + Firestore) via CDN JS SDK
    - Also stores a simple session key in `localStorage` (`nexus_session`).

- **No build step / bundler**  
  Everything runs directly in the browser; open `index.html` to use the app.

---

### 2. Running the Project

You can run the project in either of these ways:

- **Quick local preview (no server needed)**
  - On Windows: double‑click `index.html` to open it in your browser.
  - Or drag `index.html` into a browser window.
  - This is sufficient for most content/UX review.

- **Via a simple static server (recommended for Firebase features)**
  - Example with `npx` (Node.js installed):
    - Open a terminal in the project folder.
    - Run:
      ```bash
      npx serve .
      ```
    - Open the printed URL (e.g. `http://localhost:3000`) in your browser.

---

### 3. Firebase Configuration

The auth overlay script in `index.html` expects Firebase configuration to be injected via global variables if you want **real** auth:

- `__firebase_config`: JSON stringified Firebase config, e.g.:
  ```js
  window.__firebase_config = JSON.stringify({
    apiKey: "YOUR_KEY",
    authDomain: "YOUR_DOMAIN",
    projectId: "YOUR_PROJECT_ID"
    // ...other fields as needed
  });
  ```
- `__app_id`: optional app identifier used in Firestore paths (default: `"ai-mulesoft-training"`).
- `__initial_auth_token`: optional Firebase custom token for signed‑in sessions.

If these globals are **not** supplied, the script falls back to a demo config:

- Anonymous Firebase Auth is attempted.
- Firestore is used best‑effort for the simple “registry” (email/password/Q&A).

> **Important:** The current implementation stores email, password, and security answers in Firestore documents in **plaintext** for demo/training use. Do **not** deploy this as‑is in production. For real deployments:
> - Use Firebase Auth’s built‑in email/password flows or SSO.
> - Never store passwords or security answers in Firestore directly.

---

### 4. Authentication Overlay – Behavior Summary

All of the auth UI and logic is contained in `index.html`:

- **Login**
  - Fields: email, password.
  - Special training account shortcut:
    - Email: `rghv@training.com`
    - Password: `Training@123`
    - If used, the code ensures a Firestore record exists (when available) and sets `localStorage.nexus_session`.
  - Normal login:
    - Looks up a Firestore document keyed by email.
    - Compares stored `password` field.
    - On success, sets `localStorage.nexus_session` and hides the overlay.

- **Signup**
  - Fields: username, email, password, security question, answer.
  - Performs basic validations (required fields, email regex, min password length).
  - Creates a Firestore document with those fields and `createdAt`.

- **Forgot Password**
  - Step 1: user enters email → registry lookup; shows stored security question.
  - Step 2: user answers question; answer is matched (case‑insensitive).
  - Step 3: user sets a new password; Firestore `password` field is updated.

- **Session Handling**
  - `nexus_session` in `localStorage` is used as a simple “is logged in” flag.
  - On Firebase `onAuthStateChanged`, if a user exists and a session key is present, the overlay is dismissed automatically.

---

### 5. LocalStorage Usage

The app uses `localStorage` for a few features:

- **`ai_notes`** – per‑topic notes:
  - Keys: topic IDs (e.g. `t01`, `t02`, `tA01`).
  - Values: user‑entered free‑text notes.
  - Notes are auto‑loaded on page load and flagged in the UI.

- **`ai_prog`** – study progress:
  - Keys: day IDs (e.g. `DAY 01`, `DAY 02`, `AUTH 01`).
  - Values: booleans indicating completion.
  - Used to show:
    - Checked “Mark Complete” boxes
    - Completed styling in day sections and sidebar
    - Progress bar value in the sidebar.

- **`theme`** – UI theme:
  - Values: `"dark"` or `"light"`.
  - Used to set the `data-theme` attribute on `<html>`.

- **`nexus_session`** – simple session:
  - Value: email used during login.
  - Used only to skip the auth overlay client‑side.

---

### 6. Keyboard Shortcuts & UX

From anywhere in the main page (not focused in an input/textarea):

- `/` – focus the search bar.
- `Esc` – clear search and close glossary.
- `J` / `K` – navigate between day sections (scroll).
- `G` – open the glossary side panel.

The study sections (`.ds`) and cards (`.tc`) use expand/collapse behaviors and search highlighting, all implemented inline in `index.html`.

---

### 7. Making Code Changes & Keeping This README Updated

**Rule:** Whenever you change functionality or behavior, update this `README.md` accordingly so it remains a reliable reference.

When you:

- Add, remove, or significantly change **features** (e.g., new auth provider, new storage keys, new keyboard shortcuts)
- Change **Firebase/Auth** architecture (e.g., switch to Firebase email/password or SSO, move registry logic to a backend)
- Alter **data storage** strategy (e.g., stop using plaintext passwords, add hashing, or move from Firestore to another store)
- Introduce **new files** (e.g., separate CSS/JS files, build tooling, or a backend)

Then:

- Update the relevant sections above:
  - **Project Structure** – new/renamed files.
  - **Firebase Configuration** and **Authentication Overlay** – any auth flow or security changes.
  - **LocalStorage Usage** – any new/changed keys.
  - Add a short note of what changed and why (1–3 sentences).

This README is intended to be the **single source of truth** for how the training app works and how to run/configure it.

