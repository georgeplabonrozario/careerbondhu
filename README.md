<!-- Replace BOT_LINK, LANDING_PAGE_LINK and the screenshot paths before publishing. -->

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,20,24&height=180&section=header&text=Career%20Bondhu&fontSize=44&fontColor=ffffff&fontAlignY=40&desc=AI%20career%20guidance%20on%20Telegram%20for%20Bangladeshi%20students&descAlignY=62&descSize=16" width="100%" alt="Career Bondhu"/>

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-F55036?style=for-the-badge)
![Claude](https://img.shields.io/badge/Claude_API-D97757?style=for-the-badge&logo=claude&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram_Bot-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

[**Open the bot**](BOT_LINK) &nbsp;|&nbsp; [**Landing page**](LANDING_PAGE_LINK)

</div>

## Why this exists

Many students and fresh graduates in Bangladesh write their first CV with no feedback. Career counselling is hard to reach and often costly. Career Bondhu puts a CV reviewer and a career chat assistant inside Telegram, where most of these users already spend their day.

I designed and built the whole system myself on n8n, with Supabase as the database and Groq and Claude as the language models.

## What it does

- **CV review.** The user sends a CV as a PDF. The bot extracts the text, runs it through an LLM, and sends back a structured review as a text file.
- **CV rewrite.** The `/rewrite` command produces an improved version. The user can edit or confirm it with inline buttons before it is saved.
- **Career chat.** An assistant named Sanu answers career questions. It runs on Llama 3.3 70B through Groq.
- **Referrals.** Users earn credits by inviting others. Credit counting runs in a Supabase RPC function so the numbers stay consistent.
- **Payments.** Paid services are verified before they unlock. Admins approve or reject payments from a dashboard connected live to Supabase.
- **Tiers.** A free tier and a paid tier (৳99).

## How it works

```
Telegram user
     |
     v
n8n webhook  --->  command router (/start, /rewrite, CV upload, chat)
                         |
        +----------------+------------------+
        |                |                  |
   CV review         Rewrite flow       Sanu chat
   PDF.co extract    Edit / Confirm     Groq (Llama 3.3)
   -> LLM chain      inline buttons     with stored context
        |                |                  |
        +----------------+------------------+
                         |
                         v
                  Supabase (PostgreSQL)
        users, cv_submissions, payment_verifications,
                   reviews, chat_leads
                         |
                         v
        Admin dashboard (HTML + live Supabase data)
```

## Tech stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| Database | Supabase (PostgreSQL) with RPC functions |
| Language models | Groq (Llama 3.3 70B), Anthropic Claude API |
| PDF text extraction | PDF.co |
| Messaging | Telegram Bot API |
| Landing page | GitHub Pages |
| Delivery and config | GitLab CI/CD, Container Registry, CI variables for API keys |

## Problems I had to solve

A few issues cost me real debugging time, so I am keeping the fixes here.

- **Groq only reads text.** A PDF cannot go straight into the model. Text is extracted first, then passed to the LLM.
- **One call for all CV fields.** Instead of many small prompts, a single LLM chain returns every field as one JSON object. This is faster and cheaper.
- **Broken Telegram messages.** AI-written text often contains characters that Telegram's Markdown parser rejects. All AI-generated messages are sent with parse mode set to none.
- **Telegram ID type mismatch.** IDs arrived as strings in some nodes and numbers in others. `parseInt()` fixes the lookups.
- **Data across workflow branches.** Data needed on a different branch is saved to Supabase first, then fetched again with a Get a Row node.
- **Referral credits drifting.** Credit updates moved into a Supabase RPC function, `increment_referral`, so updates happen in one place.

## Database

| Table | Purpose |
|---|---|
| `users` | Telegram profile, credits, referral data |
| `cv_submissions` | Uploaded CVs and parsed results |
| `payment_verifications` | Payment proofs and admin decisions |
| `reviews` | Generated CV reviews |
| `chat_leads` | Users who started a career chat |

## Screenshots

<!-- Add real screenshots. Recruiters look at these first. -->

| Bot conversation | Admin dashboard |
|---|---|
| ![Bot](docs/bot.png) | ![Dashboard](docs/dashboard.png) |

## Roadmap

These are planned, not finished.

- [ ] Job description matching with a `/match` command
- [ ] Chat assistant that knows the user's CV
- [ ] Re-engagement message for users with incomplete CV data
- [ ] LinkedIn job alerts for paid users

## Repository contents

```
workflows/     Exported n8n workflow JSON files (credentials removed)
dashboard/     Admin dashboard HTML
docs/          Screenshots and notes
```

## Setup note

Workflow files in this repo have all credentials removed. To run your own copy you need an n8n instance, a Supabase project, a Telegram bot token from BotFather, and API keys for Groq and PDF.co. Add them as credentials inside n8n, never inside the workflow files.

## Author

**George Joseph Rozario Plabon**
[LinkedIn](https://bd.linkedin.com/in/georgeplabonrozario) | [Email](mailto:george.plabon@gmail.com)
