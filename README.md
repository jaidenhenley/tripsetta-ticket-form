# Tripsetta Ticket Form

A single-page web form that lets the team file dev tickets into the Tripsetta **Developer Items** Notion board from any device. You describe a bug or idea in plain words and Claude turns it into a clean title and description, classifies it, and creates the Notion page for you. No Notion login required.

[View the live form](https://jaidenhenley.github.io/tripsetta-ticket-form/)

## Stack

HTML, vanilla JavaScript, Supabase Edge Functions (Deno/TypeScript), Anthropic Claude API, Notion API, GitHub Pages

## How it works

A team member describes a bug or feature idea in a single text box. On submit, the page POSTs the raw text to a Supabase Edge Function, which calls Claude to structure it, then creates a page in the Developer Items Notion database and returns a link to it. The form only holds the front end; all keys and logic live server-side.

## Architecture

**Static front end.** The entire UI is one `index.html` file with no build step or dependencies. It renders the form, collects the submission, and shows the result inline with a link to the new Notion page. This repo contains only that file.

**Backend on Supabase.** The `create-ticket` Edge Function (in the main app's Supabase project, not this repo) is the engine. It holds the Notion token and Anthropic key as server-side secrets so they are never exposed to the browser.

**Claude structuring.** The function sends the raw submission to the Claude Messages API using structured outputs, with a JSON schema whose Type, Platform, and Workflow fields are enum-locked to the exact Notion options. This guarantees Claude can only return values that already exist on the board. It writes a concise title, a clean description, and the three tags.

**Manual overrides.** The form has an optional Advanced section with dropdowns for Type, Platform, and Workflow. Left on Auto, they defer to Claude. If set, the chosen value overrides Claude's pick, but only when it is a valid schema value.

**Graceful fallback.** If the Claude call fails or no key is configured, the ticket is still created from the raw text (title is the first line, Type defaults to Feedback, Platform and Workflow default to All). The form notes when AI was skipped.

**Notion write.** The function creates the page with the structured title as the Notion title property, the details as the page body, the three tags as multi-selects, Status set to Not started, and Date Requested set to today. The submitter's name, if provided, is stamped into the Notion Notes field.

**Anti-spam gate.** The function fails closed: it refuses all requests unless a shared secret is set, and every request must send a matching header. The form carries this value in `FORM_SECRET`. It is intentionally client-side and only deters random bots, so the real protection is keeping the form URL within the team.

## Config

Everything is in the config block near the bottom of `index.html`:

* `FUNCTION_URL` — the Supabase `create-ticket` function endpoint
* `FORM_SECRET` — must match the `FORM_SHARED_SECRET` secret set on the function

Commit a change to `index.html` and GitHub Pages redeploys automatically.

## Hosting

Served via GitHub Pages from the `main` branch.

1. **Settings → Pages**
2. Source: **Deploy from a branch** → `main` → `/ (root)`
3. Save; the form goes live within a minute or two

## Setup

```
git clone https://github.com/<your-username>/tripsetta-ticket-form.git
```

Open `index.html` to edit. No build step. Push to `main` to deploy.

## Notes

The form renders immediately, but submissions fail until the backend secrets (`NOTION_TOKEN`, `NOTION_DATABASE_ID`, `FORM_SHARED_SECRET`) are configured on the Supabase function by whoever owns the Supabase project. No credentials are stored in this repo.

## Developer

Jaiden Henley | [Portfolio](https://jaidenhenley.github.io/JaidenHenleyPort/) | [LinkedIn](https://www.linkedin.com/in/jaiden-henley) | [jaidenhenleydev@gmail.com](mailto:jaidenhenleydev@gmail.com)
