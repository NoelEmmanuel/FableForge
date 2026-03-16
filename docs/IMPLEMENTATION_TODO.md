# FableForge: Implementation todo (who does what)

**Personal project only**—no production or public use. Focus: learning multi-agent workflows and building the pipeline (orchestrator, continuity, planner, writer, inline editor, inspiration from online). This list is in execution order. **Done by AI (Cursor)** = implemented by the assistant. **Done by you** = your responsibility (accounts, keys, decisions, testing, content).

---

## Done by AI (Cursor)

| # | Task | Details |
|---|------|---------|
| 1 | **Project setup** | Next.js 14+ (App Router), TypeScript, ESLint, Drizzle config, env.example (no secrets). |
| 2 | **Database schema** | Drizzle schema for books, book_specs, characters, inspiration_sources, style_profiles, chapters, chapter_versions, generation_runs, artifacts; migrations. |
| 3 | **Supabase + Drizzle wiring** | DB client; run migrations against your Supabase project URL (you provide env). |
| 4 | **Auth** | Supabase Auth: sign-up/sign-in UI, session middleware, protect API routes and dashboard; optional profiles table. |
| 5 | **Books CRUD** | API: create/read/update/delete books. UI: book list, create book, edit title/slug, delete. |
| 6 | **Book spec (story bible)** | API: get/upsert book_spec (JSONB). UI: single story-bible editor (sections + key fields: genre, tone, POV). |
| 7 | **Characters** | API: CRUD characters for a book. UI: character list, add/edit/delete, name + role + short profile. |
| 8 | **Chapters** | API: list/create/update/delete/reorder chapters. UI: chapter list with reorder, create chapter, open in editor. |
| 9 | **Chapter editor shell** | TipTap editor: load chapter content, save (persist to chapters + chapter_versions), basic toolbar. |
| 10 | **Streamed chapter generation** | API: POST generate; load artifacts → planner (GPT-4o-mini) → writer (GPT-4o) stream; persist plan + run + content. Client: consume stream, append to editor, "Generating..." state. |
| 11 | **Artifacts** | Load/save continuity_summary, chapter_plan, chapter_memory (optional: character_state); retrieval by chapter order; used by writer and planner. |
| 12 | **Inline AI edit** | API: POST inline-edit (selection + action); single LLM call; return replacement text. Editor: selection toolbar or context menu (rewrite, expand, condense, more emotional, etc.); replace selection with response. |
| 13 | **Inspiration pipeline** | UI: paste, URL, or search (e.g. popular novel). API: create inspiration_source, trigger Inngest. Inngest: if URL/search, fetch sample (limit size); Inspiration agent analyzes with cheap model, write style_profile, mark done. UI: show status, attach profile to book; writer uses profile. |
| 14 | **Cost/token tracking** | Record each generation_run (input_tokens, output_tokens, model_used, cost_estimate); optional dashboard or readout. |

---

## Done by you

| # | Task | Details |
|---|------|---------|
| 1 | **Accounts and keys** | Create Supabase project (URL + anon key), OpenAI API key; optional: Vercel, Inngest. Add to `.env.local` (never commit). |
| 2 | **Run migrations** | After schema exists, run `drizzle-kit push` or `migrate` against your Supabase DB (or ask AI to run it once env is set). |
| 3 | **First user** | Sign up in the app or create a test user via Supabase dashboard. |
| 4 | **Content and decisions** | Fill at least one book (story bible, characters, outlines); add inspiration (paste, URL, or search) and confirm style profile. |
| 5 | **Testing and feedback** | Use the app to generate and edit chapters; note what’s wrong or missing. Observe how each agent runs and how artifacts hand off; tune for cost and quality. |

---

## Suggested order (AI work)

1. **Setup** → DB schema + migrations → **Auth**  
2. **Books + book spec + Characters** → **Chapters** list + reorder  
3. **Chapter editor shell** (load/save) → **Generate** (stream) + **artifacts**  
4. **Inline edit** → **Inspiration pipeline** → **Cost tracking**

The plan’s frontmatter `todos` (in the architecture plan) mirror these AI deliverables and can be checked off as each is completed.
