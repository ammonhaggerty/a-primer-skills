---
name: a-new-project
description: Scaffold a new full-stack project from the AI Primer starter template with Cloudflare Workers, D1, R2, Hono, and DaisyUI
---

# /a-new-project

Scaffold a new full-stack project. This command is repeatable — run it every time a new project is needed.

Follow all seven steps in order. Complete each step before moving to the next.

---

## Step 1: Workspace Check

Detect the operating system and check the current working directory.

- **Mac/Linux:** Check if the current directory is `~/Development`.
- **Windows:** Check if the current directory is `%USERPROFILE%\Development`.

If not in the workspace directory:

- Explain the convention: "Keeping all projects in one place makes them easy to find and gives Claude consistent workspace awareness."
- Check whether `~/Development` (or the Windows equivalent) exists. If it does not, offer to create it.
- Navigate to the workspace directory before proceeding.

---

## Step 2: Project Intent

Gather two pieces of information from the user.

**First, ask:**

> What do you want to build? Describe it in a few sentences — who it's for, what it does, what the core experience feels like.

Wait for a response.

**Then ask:**

> What should the project be called?

Suggest a kebab-case name derived from their description (e.g., `daily-checkin`, `recipe-finder`, `team-standup`). Let the user accept the suggestion or choose a different name. Store the final name as `PROJECT_NAME` for use in all subsequent steps.

---

## Step 3: Scaffold

Create the project directory at `~/Development/PROJECT_NAME/`.

**Primary path:** Check if the AI Primer starter template exists at `~/Development/ai-coding-primer/starter/`. If it exists, copy its contents into the new project directory.

**Fallback path:** If the template directory does not exist, generate the project files from scratch. Create every file listed below.

### `package.json`

```json
{
  "name": "PROJECT_NAME",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "wrangler deploy"
  },
  "dependencies": {
    "hono": "^4"
  },
  "devDependencies": {
    "wrangler": "^3",
    "@cloudflare/workers-types": "^4"
  }
}
```

Replace `PROJECT_NAME` with the actual project name.

### `wrangler.toml`

```toml
name = "PROJECT_NAME"
main = "src/index.ts"
compatibility_date = "2024-12-01"
compatibility_flags = ["nodejs_compat"]

[[d1_databases]]
binding = "DB"
database_name = "PROJECT_NAME-db"
database_id = "placeholder-will-be-updated"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "PROJECT_NAME-bucket"
```

Replace `PROJECT_NAME` with the actual project name. The `database_id` placeholder will be replaced in Step 4 after creating the real D1 database.

### `src/index.ts`

Create a basic Hono application:

```ts
import { Hono } from 'hono';
import { layout } from './layout';

type Bindings = {
  DB: D1Database;
  BUCKET: R2Bucket;
};

const app = new Hono<{ Bindings: Bindings }>();

app.get('/', (c) => {
  const content = `
    <div class="hero min-h-[60vh]">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1 class="text-5xl font-bold">PROJECT_TITLE</h1>
          <p class="py-6">PROJECT_DESCRIPTION</p>
          <a href="/api/hello" class="btn btn-primary">Test API</a>
        </div>
      </div>
    </div>
  `;
  return c.html(layout('Home', content));
});

app.get('/api/hello', (c) => {
  return c.json({ message: 'Hello from PROJECT_NAME!' });
});

export default app;
```

Replace `PROJECT_TITLE` with a human-readable version of the project name. Replace `PROJECT_DESCRIPTION` with a one-sentence version of the user's project description from Step 2. Replace `PROJECT_NAME` with the actual project name.

### `src/layout.ts`

Create a shared HTML layout function:

```ts
export function layout(title: string, content: string): string {
  return `<!DOCTYPE html>
<html lang="en" data-theme="emerald">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>${title}</title>
  <link href="https://cdn.jsdelivr.net/npm/daisyui@5/dist/full.min.css" rel="stylesheet" />
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/htmx.org@2"></script>
  <script src="https://unpkg.com/alpinejs@3" defer></script>
</head>
<body class="min-h-screen bg-base-200">
  <div class="navbar bg-base-100 shadow-sm">
    <div class="flex-1">
      <a class="btn btn-ghost text-xl" href="/">PROJECT_TITLE</a>
    </div>
  </div>
  <main class="container mx-auto p-4">
    ${content}
  </main>
</body>
</html>`;
}
```

Replace `PROJECT_TITLE` with the human-readable project name.

### `schema.sql`

```sql
-- Database schema for PROJECT_NAME
-- Add table definitions here, then apply with:
--   wrangler d1 execute PROJECT_NAME-db --local --file=./schema.sql
--   wrangler d1 execute PROJECT_NAME-db --remote --file=./schema.sql

-- Example table (uncomment and modify):
-- CREATE TABLE IF NOT EXISTS items (
--   id INTEGER PRIMARY KEY AUTOINCREMENT,
--   name TEXT NOT NULL,
--   description TEXT,
--   created_at DATETIME DEFAULT CURRENT_TIMESTAMP
-- );
```

Replace `PROJECT_NAME` with the actual project name.

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "lib": ["ESNext"],
    "types": ["@cloudflare/workers-types"],
    "jsx": "react-jsx",
    "jsxImportSource": "hono/jsx",
    "noEmit": true,
    "isolatedModules": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### `.gitignore`

```
node_modules/
.dev.vars
.wrangler/
dist/
```

### `.dev.vars`

```
# Local environment variables for wrangler dev
# Add secrets and config values here (this file is gitignored)
# Example:
# API_KEY=your-secret-key
```

---

## Step 4: Configure

### Update `wrangler.toml` with the actual project name

Verify that all `PROJECT_NAME` placeholders in `wrangler.toml` have been replaced with the real project name.

### Create Cloudflare resources

Run the following commands and capture their output:

```bash
wrangler d1 create PROJECT_NAME-db
```

Parse the `database_id` from the output.

```bash
wrangler r2 bucket create PROJECT_NAME-bucket
```

Update `wrangler.toml` with the actual `database_id` returned from the D1 creation command.

If either command fails due to authentication, run `wrangler login` first and retry. If `wrangler` is not installed, run `npm install -g wrangler` first.

### Generate `CLAUDE.md`

Create `CLAUDE.md` in the project root. Tailor it to the project using the user's description from Step 2.

```markdown
# PROJECT_TITLE

PROJECT_DESCRIPTION_FROM_STEP_2

## Tech Stack

- **Runtime:** Cloudflare Workers
- **Framework:** Hono
- **Styling:** Tailwind CSS + DaisyUI (loaded via CDN, no build step)
- **Interactivity:** HTMX + Alpine.js (loaded via CDN, no build step)
- **Database:** Cloudflare D1 (SQLite)
- **File Storage:** Cloudflare R2

## Commands

- `npm run dev` — Start the local development server (runs `wrangler dev`)
- `npm run deploy` — Deploy to production on Cloudflare Workers

## Project Memory

Document all plans, architectural decisions, and session summaries in `docs/`. Use `docs/plans/` for feature plans and `docs/log/` for session notes and decisions.

At the start of each session and after context compaction, check recent entries in `docs/log/` and `docs/plans/` to restore context about in-progress work.

## Conventions

- Use DaisyUI component classes for all UI elements (buttons, cards, forms, navigation, modals)
- Use HTMX for server interactions (form submissions, dynamic content loading, partial page updates)
- Use Alpine.js for client-side state (toggles, dropdowns, counters, visibility)
- Use Hono for all routing and API endpoints
- Keep all styles in HTML via Tailwind/DaisyUI classes — no separate CSS files
- All CDN dependencies are loaded in `src/layout.ts` — no build step or bundler needed
```

Replace `PROJECT_TITLE` and `PROJECT_DESCRIPTION_FROM_STEP_2` with the actual values.

### Create the docs directory structure

```
docs/
  plans/     (for feature plans and design documents)
  log/       (for session summaries and decisions)
  README.md
```

Create `docs/README.md` with:

```markdown
# Project Documentation

This folder is the project's persistent memory. Plans go in `plans/`, session notes and decisions go in `log/`. Claude checks these at the start of each session.
```

Create empty `.gitkeep` files in `docs/plans/` and `docs/log/` so Git tracks the empty directories.

---

## Step 5: Version Control

### Initialize Git

```bash
git init
git add -A
git commit -m "Initial scaffold from AI Primer starter template"
```

### Create GitHub repository

```bash
gh repo create PROJECT_NAME --private --source=. --push
```

If the `gh` command is not installed:

1. Offer to install it:
   - **Mac:** `brew install gh`
   - **Linux:** Follow the instructions at https://github.com/cli/cli/blob/trunk/docs/install_linux.md
   - **Windows:** `winget install --id GitHub.cli`
2. After installing, authenticate with `gh auth login` and follow the prompts.
3. Retry the `gh repo create` command.

If SSH keys or GitHub authentication are not configured, walk through the setup:

1. Run `gh auth login`.
2. Select GitHub.com, SSH protocol, and follow the interactive prompts.
3. Retry the `gh repo create` command after authentication succeeds.

---

## Step 6: Run Locally

Install dependencies and start the development server:

```bash
npm install
npm run dev
```

After the server starts, tell the user:

> Open **http://localhost:8787** in your browser to see the project running.
>
> This is the project running on your machine. Only you can see it right now — it is not on the internet yet.

If `wrangler dev` fails, check common causes:

- Port 8787 already in use — another `wrangler dev` may be running. Stop it or use `wrangler dev --port 8788`.
- Missing dependencies — run `npm install` again.
- Authentication issues — run `wrangler login`.

---

## Step 7: Nudge

After confirming the project is running, share these closing messages:

> The project is running locally. When ready to put it on the internet, just say "deploy my project" and Claude will handle the rest.

Remind the user of the build loop:

> **Dream it** — imagine what should exist.
> **Describe it** — tell Claude, in plain English, what you want.
> **Build it** — Claude writes the code and connects the pieces.
> **React to it** — look at what Claude built and notice what is off.
> **Refine it** — describe the adjustments, Claude makes them.
> **Ship it** — one command, it is live.

Then suggest a next step:

> Start by describing a feature you want to add. Be specific — "add a form where users can submit their name and email" works better than "add a form."
