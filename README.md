# AAFE Gala Guest List Tool: start here

© 2026 ERIMX BOKHUS team, all rights reserved, licensing to be decided with AAFE and Pursuit.


A companion tool for Asian Americans for Equality's annual gala. It sits on top of Give Lively and turns ticket purchases, sponsorship pledges, checks and direct deposits, and comps into one complete, editable guest list. It adds a private sponsor portal where sponsors name their guests, a staff workspace, badge exports, and gala-night check-in on iPads.

**Status:** this is the stable build (v1.0) as of November 2, 2026, built and tested on invented data only. It is handed to AAFE on November 10, 2026.

Built by team ERIMX BOKHUS (Pursuit AI Native Fellowship): Natalie Walker, Aisling Leiva-Davila, Cornell Robertson and Antonin Lesov. Shared with AAFE and its technical partners for the handoff.

## Documents

| | Document | File | What it covers |
| --- | --- | --- | --- |
| **01** | Backend: setup and reference | `prototype/docs/BACKEND.md` | Setting up a Supabase project, deploying the server functions and the apps, every function the apps call, Give Lively, Salesforce, retention. Starts with a diagram of how the parts connect |
| **02** | Running the tool | `docs/operations.md` | Free-plan limits, backups and restores, who can open the apps, and a checklist for gala night |
| **03** | Keys and secrets | `docs/keys-and-secrets.md` | Every key: what it can do, where it lives, how to rotate it, and what to change before real data |
| **04** | Security review | `docs/security-review.md` | What was checked, what was found and fixed, and the tests that prove it |
| **05** | Guest data handling | `docs/data-handling.md` | The data commitments made to AAFE: what is stored, what never is, who sees what, retention |
| **06** | Salesforce | `docs/salesforce-setup.md` | For AAFE and Idlewild Partners: the Salesforce file AAFE imports after the gala, a suggested column mapping, and the direct sync in the code (not connected) |
| **07** | The apps | `prototype/README.md`, `prototype/checkin-app/README.md`, `import-engine/README.md`, the `.env.example` files | Developer notes for the apps, the iPad check-in app and the import engine, with the settings files |

## Try it without installing

Double-click `prototype/OPEN-Staff-App.html`, `prototype/OPEN-Guest-Portal.html` or `prototype/OPEN-CheckIn-App.html`. Each is one self-contained file that runs the app offline on invented data (`prototype/README.md`, `prototype/checkin-app/README.md`).

## The parts

- **Three web apps** (React and TypeScript, built with Vite and Tailwind CSS): the staff app, the sponsor portal, and the iPad check-in app, which installs on an iPad's Home Screen and keeps working when the venue's internet drops.
- **Supabase:** Postgres with row-level security on every table, sign-in by emailed code, three server functions (the Give Lively sync, the email sender, and a Salesforce sync that isn't connected to a real Salesforce: document 06), and five scheduled jobs.
- **Hosting:** the apps are plain static files that any host can serve. The prototype serves them from Cloudflare Pages, behind Cloudflare Access.
- **Email:** Resend sends the app's emails. Sign-in codes go through the project's own SMTP sender.
- **Salesforce:** after the gala, a gala admin downloads the Salesforce file (who checked in, with organization and sponsor) for AAFE to import. Give Lively's own Salesforce app writes the online purchases.
- **Rules built into the code:** no payment processing and no card or bank details; staff approve every email before it sends; sponsor links and QR e-tickets are random, stored only as a hash, and expire; every edit is logged with its source; no AI anywhere.

## Checks

`npm ci`, then `node scripts/ci-local.mjs` at the repository's root runs every check on your own computer: the real-data guard, the secret scan, the import engine's tests, the lint, the frontend tests and demo-file rebuild, the backend tests, and the server functions' type check. It takes about 3 minutes; `--quick` takes about 15 seconds. Docker (for the secret scan) and Deno (for the server functions) are optional: a missing tool skips its check. The same checks are set up to run in GitHub Actions on every pull request (`.github/workflows/`).

## Before real data

Two lists say what changes before AAFE's real gala data goes in:

- **Running the tool** (`docs/operations.md`), "Before going live: what still points at the team".
- **Keys and secrets** (`docs/keys-and-secrets.md`), "Before real data or handoff".
