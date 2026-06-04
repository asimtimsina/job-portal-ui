# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start Vite dev server
- `npm run build` — Production build
- `npm run lint` — ESLint (flat config, JS/JSX only)
- `npm run preview` — Preview production build

## Architecture

React 19 SPA using Vite 7, Tailwind CSS 4, and React Router 7. No TypeScript — plain JSX throughout.

### State Management

Two layers of React Context:

- **`src/context/`** — Core contexts: `AuthContext` (auth + dummy users + localStorage persistence), `JobContext` (applications, saved jobs, employer job CRUD), `ThemeContext`
- **`src/contexts/`** — Data-fetching contexts: `JobsDataContext` (cached job list with 5-min TTL), `CompaniesContext`

Provider nesting order (in App.jsx): AuthProvider → JobsDataProvider → JobProvider → CompaniesProvider → ThemeProvider

### Data Layer

Currently uses **mock data** with localStorage persistence — no real backend. Services in `src/services/` simulate async API calls using `delay()` from `src/utils/delay.js`. Data originates from `src/data/mockData.js`.

Key localStorage keys: `jobPortalUser`, `authToken`, `registeredUsers`, `globalPostedJobs`, `jobApplications_{userId}`, `savedJobs_{userId}`, `postedJobs_{userId}`.

### Routing & Roles

Three roles with route protection via `ProtectedRoute` component:
- **ROLE_JOB_SEEKER** — profile, applied-jobs, saved-jobs
- **ROLE_EMPLOYER** — post-job, employer/jobs, job-applicants/:jobId
- **ROLE_ADMIN** — admin/*, admin pages in `src/pages/admin/`

### Key Libraries

- Font Awesome + Lucide React for icons
- react-toastify for notifications

### ESLint

Flat config (`eslint.config.js`). The `no-unused-vars` rule ignores variables starting with uppercase or underscore (`varsIgnorePattern: '^[A-Z_]'`).

## Git Workflow

Default branch is `master`. Remote is `origin` (GitHub: `asimtimsina/job-portal-ui`).

### Branching Strategy

Trunk-based with short-lived feature branches off `master`:

- **`master`** — always deployable; never commit directly except for trivial docs/config tweaks (and only when explicitly requested).
- **Feature branches** — branch off latest `master`, keep scope tight, rebase or merge `master` in if they go stale (>2 days).
- **Naming** — `<type>/<short-kebab-summary>`, e.g. `feat/saved-jobs-pagination`, `fix/login-redirect-loop`, `chore/upgrade-vite-7`, `refactor/auth-context-split`, `docs/readme-setup`.
- **Lifetime** — delete the branch after the PR merges (locally + remote).

### Commit Strategy

Use [Conventional Commits](https://www.conventionalcommits.org/) with an imperative subject:

```
<type>(<optional-scope>): <subject ≤ 72 chars>

<optional body: the *why*, wrapped at ~72 cols>

<optional footer: BREAKING CHANGE:, Refs #123, Co-Authored-By: ...>
```

- **Types** — `feat`, `fix`, `chore`, `refactor`, `docs`, `style`, `test`, `perf`, `build`, `ci`, `revert`.
- **Scope (optional)** — area touched, e.g. `auth`, `jobs`, `admin`, `ui`, `context`.
- **Subject** — lowercase, no trailing period, imperative ("add", not "added"/"adds").
- **Atomic** — one logical change per commit; don't bundle unrelated edits.
- **Body** — explain the *why*, not the *what* (the diff shows the what). Skip for obvious changes.
- **Don't commit** — `node_modules`, `dist`, `.env*`, editor files, or any file containing secrets. `.gitignore` already covers most.

### PR Strategy

- **Always via PR** — even solo, open a PR rather than pushing to `master` directly. It gives a review surface and a deploy checkpoint.
- **Title** — same Conventional Commits format as the lead commit, ≤70 chars.
- **Body template** — `## Summary` (1–3 bullets on *what + why*) + `## Test plan` (checklist of how it was verified: `npm run lint`, `npm run build`, manual flows touched).
- **Size** — aim for <400 LOC diff; split larger work into stacked PRs when feasible.
- **Merge style** — **squash and merge** into `master` to keep history linear; the squash commit message should match Conventional Commits.
- **Before merging** — `npm run lint` and `npm run build` must pass; manually verify the feature in `npm run dev` for UI changes.
- **After merging** — delete the remote branch (GitHub option) and the local branch (`git branch -d <name>`).
- **Never** force-push to `master`; force-push to feature branches only with `--force-with-lease`.
