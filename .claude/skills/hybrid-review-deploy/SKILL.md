---
name: hybrid-review-deploy
description: >
  Deploy workflow for the Hybrid Real Estate client review page. Automatically provides the correct git commit and push command with the full directory path after EVERY code change to the hybrid-review-site. Load this skill at the start of ANY session involving the review page — editing HTML, updating webhooks, changing copy, fixing bugs, or adding features. Trigger on: "review page", "hybrid-review-site", "index.html", "Make webhook", "star rating page", "feedback form", "client review", or any edit to the review page code.
---

# Hybrid Review Page — Deploy Skill

## Project context

- **Repo location (Mac — full path):** `/Users/elliot/Library/CloudStorage/GoogleDrive-elliot@hybridrealestate.com.au/My Drive/~Management Access/Claude Playing /Hybrid Business Development /hybrid-review-site`
- **File to edit:** `index.html` (was previously `Hybrid_RE_Review_Page.html`)
- **Live URL:** Cloudflare Pages — auto-deploys on every `git push`
- **Worker URL (old):** `https://hybrid-review-page.elliot-410.workers.dev`
- **Make webhook:** already set in `MAKE_WEBHOOK` constant in `index.html`
- **Notion database:** ⭐ Client Feedback — Hybrid Real Estate (`6852ed228dcb45b78c946edf8b3fcefd`)

## After EVERY code change — always output this

After making any edit to the review page, always end your response with this exact block so Elliot can copy-paste it straight into Terminal:

```bash
cd '/Users/elliot/Library/CloudStorage/GoogleDrive-elliot@hybridrealestate.com.au/My Drive/~Management Access/Claude Playing /Hybrid Business Development /hybrid-review-site' && git add . && git commit -m "DESCRIBE THE CHANGE HERE" && git push
```

Replace `DESCRIBE THE CHANGE HERE` with a short plain-English description of what changed (e.g. `"Update positive section copy"`, `"Add contact checkbox"`, `"Fix Google redirect timing"`).

**Why this matters:** Elliot is learning git and shouldn't have to remember the path or commands. Providing it every time removes friction and prevents the common mistake of forgetting to push after making changes.

## If you see a `.git/index.lock` error

This file gets left behind if a previous git operation was interrupted. Clear it with:

```bash
rm '/Users/elliot/Library/CloudStorage/GoogleDrive-elliot@hybridrealestate.com.au/My Drive/~Management Access/Claude Playing /Hybrid Business Development /hybrid-review-site/.git/index.lock'
```

Then re-run the commit/push command above.

## Key constants in index.html

```javascript
const GOOGLE_URL     = 'https://g.page/r/CQnkPfXXznBEEBM/review';
const MAKE_WEBHOOK   = 'https://hook.eu1.make.com/fk4593xb9kxi0gc9juovbmlg9d122l4v';
const FALLBACK_EMAIL = 'team@hybridrealestate.com.au';
const REDIRECT_MS    = 5000;
```

## Webhook payload types

The page fires three event types to the Make webhook:

| type | when | key fields |
|------|------|------------|
| `star_tap` | immediately on star click | `session_id`, `rating`, `rating_label` |
| `google_click` | just before redirect to Google | `session_id` |
| `form_submit` | on 1–3 star form submission | `session_id`, `name`, `feedback`, `area`, `email`, `contact_requested` |

All events share `session_id` (UUID generated on page load) so Make can link them in Notion.

## Notion database schema

Columns: Client Name, Rating (1–5 stars), Feedback, Area, Client Email, Status, Session ID, Contact Requested (checkbox), Google Click (checkbox), Team Notes, Attn To, Submitted.

## Make.com scenario structure

Three routes off a single webhook based on `type`:
1. `star_tap` → Create Notion record
2. `form_submit` → Search Notion by `session_id` → Update record with form fields
3. `google_click` → Search Notion by `session_id` → Tick Google Click checkbox

## Brand colours

- Emerald (header bg): `#0D5C4A`
- Fezandie (logo, accents): `#4ECBA8`
- Slate (body text): `#2D3035`
- Mist (page bg): `#F2F6F5`

Logo: pulled from Webflow CDN — `https://cdn.prod.website-files.com/652e5372d14b6af0be5ee017/652e5372d14b6af0be5ee043_Hybrid_Workmark_Fezandie.png`
