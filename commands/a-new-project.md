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

## Step 2: Project Choice

### Choose a starting point

Ask:

> **How would you like to start?**
>
> 1. **Sample promo page** — A working single-page personal site with profile card, weather, social links, AI chat, timeline, and music widget. You'll customize it with your info, then modify or add modules from there.
>
> 2. **Empty project** — A clean slate with the full stack wired up (Hono, DaisyUI, HTMX, Alpine.js, D1, R2, Workers AI) but no modules. Good for building something completely new.

Store the choice as `PROJECT_MODE` (`sample` or `empty`).

### Project name

**If sample mode:**

Ask:

> What should we call your project? This becomes the folder name and URL. A short kebab-case name works best — for example, `my-site`, `dj-ammon`, `jane-codes`.

**If empty mode:**

Ask:

> What do you want to build? Describe it in a few sentences — who it's for, what it does, what the core experience feels like.

Then ask:

> What should the project be called?

Suggest a kebab-case name derived from their description (e.g., `daily-checkin`, `recipe-finder`). Let the user accept or choose a different name.

Store the final name as `PROJECT_NAME`.

---

## Step 3: Scaffold

Create the project directory at `~/Development/PROJECT_NAME/`.

**Primary path:** Download the starter template from GitHub:

```bash
npx -y degit ammonhaggerty/ai-coding-primer/starter ~/Development/PROJECT_NAME
```

After downloading, replace placeholder values in the template:

- In `package.json`: change `"name": "starter"` to `"name": "PROJECT_NAME"`
- In `wrangler.toml`: change `name = "starter"` to `name = "PROJECT_NAME"`, `database_name = "starter-db"` to `database_name = "PROJECT_NAME-db"`, `bucket_name = "starter-bucket"` to `bucket_name = "PROJECT_NAME-bucket"`

**If empty mode**, also strip the sample modules:

1. Delete the `src/modules/` directory
2. Delete `src/system-prompt.ts`
3. Delete the `public/` directory
4. Replace `src/index.ts` with the minimal version (see **Empty index.ts** below)
5. Replace `schema.sql` with the empty version (see **Empty schema.sql** below)

Then skip to **Step 5: Configure**.

**If sample mode**, proceed to **Step 4: Personalize**.

---

**Fallback path:** If the download fails (no internet, GitHub unavailable, or `npx` error), generate the project files from scratch using the templates below. The fallback always produces an empty-mode project.

### Fallback `package.json`

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
    "@cloudflare/workers-types": "^4",
    "wrangler": "^4"
  }
}
```

### Fallback `wrangler.toml`

```toml
name = "PROJECT_NAME"
main = "src/index.ts"
assets = { directory = "./public" }
compatibility_date = "2024-12-01"
compatibility_flags = ["nodejs_compat"]

[[d1_databases]]
binding = "DB"
database_name = "PROJECT_NAME-db"
database_id = "placeholder-will-be-updated"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "PROJECT_NAME-bucket"

[ai]
binding = "AI"
```

### Fallback `src/layout.ts`

```ts
export function layout(title: string, content: string): string {
  return `<!DOCTYPE html>
<html lang="en" data-theme="emerald">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>${title}</title>
  <link href="https://cdn.jsdelivr.net/npm/daisyui@5" rel="stylesheet" type="text/css" />
  <link href="https://cdn.jsdelivr.net/npm/daisyui@5/themes.css" rel="stylesheet" type="text/css" />
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
  <script src="https://unpkg.com/htmx.org@2"></script>
  <script src="https://unpkg.com/alpinejs@3" defer></script>
</head>
<body class="min-h-screen bg-base-200">
  <main class="max-w-3xl mx-auto p-5 flex flex-col gap-5">
    ${content}
  </main>
</body>
</html>`;
}
```

### <a id="empty-index"></a>Empty `index.ts` (used by both empty mode and fallback)

```ts
import { Hono } from 'hono';
import { layout } from './layout';

type Bindings = {
  DB: D1Database;
  BUCKET: R2Bucket;
  AI: Ai;
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

### <a id="empty-schema"></a>Empty `schema.sql` (used by both empty mode and fallback)

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

### Fallback `tsconfig.json`

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

### Fallback `.gitignore`

```
node_modules/
.dev.vars
.wrangler/
dist/
```

### Fallback `.dev.vars`

```
# Local environment variables for wrangler dev
# Add secrets and config values here (this file is gitignored)
```

### Fallback: create `public/` directory

```bash
mkdir -p public
```

---

## Step 4: Personalize (sample mode only)

Skip this step entirely if `PROJECT_MODE` is `empty`.

This step gathers the user's info and customizes the sample project. Ask questions one at a time — do not batch them.

### 4a: Name

Ask:

> What's your name? This appears on the profile card.

Store as `USER_NAME`.

### 4b: Location

Ask:

> What city are you in? This shows on your profile and powers the weather widget. For example: "Oakland, CA" or "London, UK".

Store as `USER_LOCATION`.

### 4c: Social links

The sample project includes a link tree module. Each link has an icon, display label, and URL.

**Check if GitHub username is available** by running `gh api user --jq .login 2>/dev/null` or checking `git config --global user.name`. If found, pre-fill it.

Present the available platforms:

> Let's set up your social links. Pick a platform to add (or say "done" when finished):
>
> 1. GitHub
> 2. LinkedIn
> 3. Instagram
> 4. Threads
> 5. Discord
> 6. Discogs
> 7. Substack
> 8. Other (custom URL)

When the user picks a platform, ask for their handle:

> What's your [Platform] handle? (e.g., `/yourname`)

If they picked **GitHub** and you already found a username, suggest it: "I found your GitHub username: `USERNAME`. Use this one?"

If they picked **Other**, ask for both a label and URL:

> What should the link text say? (e.g., "mysite.com")
> What's the full URL? (e.g., "https://mysite.com")

After each link is added, ask:

> Add another link? Pick a platform or say "done".

**Build the link URLs from handles using these patterns:**

| Platform  | Icon key   | URL pattern                                    |
|-----------|------------|------------------------------------------------|
| GitHub    | `github`   | `https://github.com/HANDLE`                    |
| LinkedIn  | `linkedin` | `https://linkedin.com/in/HANDLE`               |
| Instagram | `instagram`| `https://instagram.com/HANDLE`                 |
| Threads   | `threads`  | `https://threads.net/@HANDLE`                   |
| Discord   | `discord`  | `https://discord.com/users/HANDLE`              |
| Discogs   | `discogs`  | `https://discogs.com/user/HANDLE`               |
| Substack  | `substack` | `https://substack.com/@HANDLE`                  |
| Other     | `link`     | Full URL as provided                            |

Store all links. After the user says "done", generate the `schema.sql` INSERT statements. Replace the existing seed data block in `schema.sql` with:

```sql
-- Your social links
INSERT OR IGNORE INTO links (icon, label, url, sort_order) VALUES
  ('ICON_KEY', '/HANDLE', 'FULL_URL', 1),
  ('ICON_KEY', '/HANDLE', 'FULL_URL', 2);
```

Use the handle as the label (prefixed with `/` for social platforms, or the custom label text for "Other" links). Assign `sort_order` sequentially.

### 4d: Profile photo

Ask:

> Want to add a profile photo? You can **drag and drop** a JPG or PNG image right into this terminal window. This is a good trick to know — Claude can see images you drop in!
>
> Drop an image now, or say "skip" to use a default avatar.

**If the user provides an image:**

1. Resize it to 200x200 pixels for the profile card:
   - **Mac:** `sips -z 200 200 SOURCE_PATH --out ~/Development/PROJECT_NAME/public/avatar.png`
   - **Linux:** `convert SOURCE_PATH -resize 200x200^ -gravity center -extent 200x200 ~/Development/PROJECT_NAME/public/avatar.png`
2. Confirm: "Profile photo saved!"

**If the user says skip:**

Generate a simple initials-based SVG avatar and save it as `public/avatar.png`. Use the first letter of their name on a colored background. Or, leave the existing `avatar.png` from the template as-is.

### 4e: Weather API key

The weather widget needs a free API key from WeatherAPI.com.

Ask:

> The profile card shows live local weather. This uses a free API key from WeatherAPI.com. Want to set this up now, or skip it for later?

**If yes:**

Walk through the setup:

> Here's how to get your free API key:
>
> 1. Open **https://www.weatherapi.com/signup.aspx** in your browser
> 2. Create a free account (email + password — no credit card needed)
> 3. After signing in, your API key is shown on the dashboard at **https://www.weatherapi.com/my/**
> 4. Copy the key and paste it here

Wait for the user to paste their key. Store as `WEATHER_API_KEY`.

**If skip:** The weather section will simply not appear — the profile card handles this gracefully when no API key is set.

### 4f: Write `.dev.vars`

Generate the `.dev.vars` file with all collected values:

```
# Local environment variables (this file is gitignored)
PROFILE_NAME=USER_NAME
PROFILE_LOCATION=USER_LOCATION
WEATHER_API_KEY=THEIR_KEY_OR_BLANK
WEATHER_LOCATION=USER_LOCATION
```

If the user skipped the weather API key, omit the `WEATHER_API_KEY` and `WEATHER_LOCATION` lines (or leave them commented out).

---

## Step 5: Configure

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

If D1 creation fails due to authentication, run `wrangler login` first and retry. If `wrangler` is not installed, run `npm install -g wrangler` first.

**If R2 bucket creation fails** with an error like "Please enable R2 through the Cloudflare Dashboard" (error code 10042), R2 hasn't been activated on the account. Tell the user:

"R2 Object Storage needs to be enabled in your Cloudflare dashboard. Go to **dash.cloudflare.com**, click **R2 Object Storage** in the left sidebar, and follow the activation steps. It requires a credit card on file, but the free tier is generous — 10GB storage and 10 million reads per month at no charge. Once it's enabled, let me know and I'll retry."

Wait for confirmation, then retry `wrangler r2 bucket create PROJECT_NAME-bucket`. The project cannot proceed without R2 — it's needed for file storage and deployment.

### Generate `CLAUDE.md`

Create `CLAUDE.md` in the project root. Tailor it to the project:

**If sample mode:**

```markdown
# PROJECT_TITLE

A personal single-page site with modular cards: profile with weather, social links, AI chat, timeline, and music widget.

## Tech Stack

- **Runtime:** Cloudflare Workers
- **Framework:** Hono
- **Styling:** Tailwind CSS v4 + DaisyUI 5 (CDN, no build step)
- **Interactivity:** HTMX + Alpine.js (CDN, no build step)
- **Animations:** GSAP (CDN)
- **Database:** Cloudflare D1 (SQLite)
- **File Storage:** Cloudflare R2
- **AI:** Cloudflare Workers AI (Llama 3.2 3B)
- **Fonts:** JetBrains Mono (body), Fraunces (headings)
- **Icons:** Bootstrap Icons, Weather Icons
- **Theme:** DaisyUI `emerald` (light) / `forest` (dark)

## Modules

Each module is a self-contained file in `src/modules/`:

- `profile.ts` — Name, location, avatar, live weather from WeatherAPI
- `chat.ts` — Terminal-style AI chat with conversation history (D1 sessions)
- `links.ts` — Social link tree from D1 database
- `timeline.ts` — Timeline of AI history with DaisyUI timeline component
- `listening.ts` — Mixcloud embed widget

## Environment Variables (`.dev.vars`)

- `PROFILE_NAME` — Display name on profile card
- `PROFILE_LOCATION` — Location shown on profile card
- `WEATHER_API_KEY` — Free key from weatherapi.com (optional)
- `WEATHER_LOCATION` — Location for weather lookup (defaults to PROFILE_LOCATION)

## Commands

- `npm run dev` — Start local dev server (runs `wrangler dev`)
- `npm run deploy` — Deploy to Cloudflare Workers
- Schema migration: `wrangler d1 execute PROJECT_NAME-db --local --file=./schema.sql`

## Conventions

- Write a plan before building anything non-trivial — always
- Use DaisyUI component classes for all UI elements
- Use HTMX for server interactions
- Use Alpine.js for client-side state
- Keep all styles in HTML via Tailwind/DaisyUI classes — no separate CSS files
- All CDN dependencies loaded in `src/layout.ts` — no build step
- DaisyUI 5: `card-border` doesn't work, use `border border-base-300`
- Cookies: `secure: false` in dev, `secure: true` in production

## Planning

For any non-trivial feature, write a plan to `docs/plans/` before writing code. Do not implement until the user reviews and approves the plan.

## Project Memory

Save session summaries, decisions, and progress to `docs/log/`. Check `docs/log/` and `docs/plans/` at session start to restore context.
```

**If empty mode:**

```markdown
# PROJECT_TITLE

PROJECT_DESCRIPTION_FROM_STEP_2

## Tech Stack

- **Runtime:** Cloudflare Workers
- **Framework:** Hono
- **Styling:** Tailwind CSS + DaisyUI 5 (CDN, no build step)
- **Interactivity:** HTMX + Alpine.js (CDN, no build step)
- **Database:** Cloudflare D1 (SQLite)
- **File Storage:** Cloudflare R2
- **AI:** Cloudflare Workers AI

## Commands

- `npm run dev` — Start local dev server (runs `wrangler dev`)
- `npm run deploy` — Deploy to Cloudflare Workers

## Conventions

- Write a plan before building anything non-trivial — always
- Use DaisyUI component classes for all UI elements
- Use HTMX for server interactions
- Use Alpine.js for client-side state
- Keep all styles in HTML via Tailwind/DaisyUI classes — no separate CSS files
- All CDN dependencies loaded in `src/layout.ts` — no build step
- DaisyUI 5: `card-border` doesn't work, use `border border-base-300`
- Cookies: `secure: false` in dev, `secure: true` in production

## Planning

For any non-trivial feature, write a plan to `docs/plans/` before writing code. Do not implement until the user reviews and approves the plan.

## Project Memory

Save session summaries, decisions, and progress to `docs/log/`. Check `docs/log/` and `docs/plans/` at session start to restore context.
```

Replace `PROJECT_TITLE` and `PROJECT_DESCRIPTION_FROM_STEP_2` with actual values.

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

## Step 6: Version Control

### Initialize Git

```bash
git init
git add -A
git commit -m "Initial scaffold from AI Primer starter template"
```

Your project is now tracked by Git locally. Every change from here on can be saved and undone.

### Create GitHub repository

If the user ran primer-setup, GitHub CLI (`gh`) is already installed and authenticated. Create a private repository and push:

```bash
gh repo create PROJECT_NAME --private --source=. --push
```

If `gh` is not installed or not authenticated (user skipped primer-setup), walk through the setup:

1. Install `gh`: **Mac:** `brew install gh` / **Linux:** see https://github.com/cli/cli/blob/trunk/docs/install_linux.md / **Windows:** `winget install --id GitHub.cli`
2. Authenticate: `gh auth login` — select GitHub.com, HTTPS, and login via browser
3. Retry: `gh repo create PROJECT_NAME --private --source=. --push`

If the user does not have a GitHub account, offer to skip: "When you want to back up your project, just say 'help me set up GitHub' and I'll walk you through it."

---

## Step 7: Run & Explore

### Install dependencies and apply schema

```bash
npm install
```

**If sample mode** (or if `schema.sql` has table definitions):

```bash
wrangler d1 execute PROJECT_NAME-db --local --file=./schema.sql
```

This creates the database tables and seeds the social links locally.

### Start the development server

```bash
npm run dev
```

After the server starts, tell the user:

> Open **http://localhost:8787** in your browser to see your project running.
>
> This is running on your machine — only you can see it right now. When you're ready to put it on the internet, say "deploy my project" and Claude will handle the rest.

If `wrangler dev` fails, check common causes:

- Port 8787 already in use — another `wrangler dev` may be running. Stop it or use `wrangler dev --port 8788`.
- Missing dependencies — run `npm install` again.
- Authentication issues — run `wrangler login`.

### What's next

**If sample mode:**

> You're looking at a working personal site! Here's what you can do from here:
>
> - **Modify any module** — Change the timeline content, swap the Mixcloud embed for your own, update your links
> - **Remove a module** — If you don't need the timeline or music widget, just say so
> - **Add a new module** — Ideas: books you're reading (Google Books API for covers), a photo gallery, a blog feed, a contact form
> - **Upgrade the AI chat** — Right now it uses the free Llama 3.2 model. You can add a Claude API key for a big upgrade, or train it for a completely different purpose
> - **Change the look** — Swap fonts, try a different DaisyUI theme, adjust the layout
>
> What would you like to change first?

**If empty mode:**

Remind the user of the build loop:

> **Dream it** — imagine what should exist.
> **Describe it** — tell Claude, in plain English, what you want.
> **Build it** — Claude writes the code and connects the pieces.
> **React to it** — look at what Claude built and notice what is off.
> **Refine it** — describe the adjustments, Claude makes them.
> **Ship it** — one command, it is live.

Then suggest a next step:

> Start by describing a feature you want to add. Be specific — "add a form where users can submit their name and email" works better than "add a form."
>
> Claude will write a plan first and ask you to review it before building anything. This is by design — reviewing a plan takes seconds, undoing a wrong assumption takes much longer.
