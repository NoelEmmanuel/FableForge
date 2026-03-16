# FableForge: Cost Breakdown (Personal Project)

This document covers **cost breakdown** for FableForge as a **personal project only**—no production or public use. The goal is to keep spending low while learning multi-agent workflows and using inspiration from online (paste, URL, or search for popular novels’ tone/style).

---

## Personal use: monthly cost (approximate)

| Item | Cost | Notes |
|------|------|--------|
| **Vercel** | $0 | Hobby plan: enough for local or personal deploy. |
| **Supabase** | $0 | Free tier: 500MB DB, 50K MAU, 2 projects. |
| **Inngest** | $0 | Free tier: 25K step runs/month. |
| **OpenAI API** | $5–25 | Depends on usage (see below). |
| **Domain (optional)** | $0–1 | Only if you use a custom domain. |
| **Total** | **~$5–25/month** | Effectively just OpenAI. |

---

## OpenAI usage (cost-conscious design)

Pricing (GPT-4o: $2.50/1M input, $10/1M output; GPT-4o-mini: $0.15/1M input, $0.60/1M output):

- **One full chapter generation** (planner + writer): ~$0.03–0.05 per chapter (~2.5k words).
- **One inline edit**: ~$0.005–0.01 per edit.
- **One inspiration analysis** (paste or URL/search sample): ~$0.001–0.003 per run (cheap model + limited sample size).
- **Continuity agent** (after chapter): ~$0.001–0.002 per chapter if run async.

If you generate **30 chapters** in a month, **20 inline edits**, and **3 inspiration runs** (e.g. URL or search for novel samples):

- Chapters: 30 × ~$0.04 ≈ **$1.20**
- Inline: 20 × ~$0.007 ≈ **$0.15**
- Inspiration: 3 × ~$0.002 ≈ **$0.01**
- **Total OpenAI ≈ $1.50–3/month** for light use.

Heavier use (e.g. 80 chapters + many rewrites + more inspiration from longer URL samples) might land in the **$10–25/month** range. **Keeping cost down:** use GPT-4o-mini for planner, continuity, and inspiration; limit inspiration sample size (e.g. 10–20k chars); reuse artifacts so you never send the full book.

---

## What you pay with your time

- **You:** Accounts (Supabase, OpenAI, optionally Vercel/Inngest), env setup, testing, content (story bibles, outlines), and optionally URL/search setup for inspiration (e.g. fetch from a URL or a search API).
- **AI (Cursor):** Implementation per the plan’s todo list.

---

## Summary

| Scope | Approx. monthly cost |
|-------|----------------------|
| **Personal project** | **$5–25** (mostly OpenAI; infra free tier) |

No Phase 2 or market feasibility in scope for this build—purely personal use and learning multi-agent workflows.
