# AI-Assisted Job Hunt Playbook

A repeatable process for running a job search with Claude: sharpen the resume and LinkedIn profile, search fresh postings every weekday, score them, write application packs (cover letter, tailored CV, recruiter note), and track every application on a private dashboard.

It works for any candidate. Fill in the **Candidate profile** in section 1, then give Claude each section in order. Text in `{{double braces}}` is a placeholder.

> **What this does not do.** Claude never submits applications, sends messages, or logs in to job sites for the candidate. LinkedIn's terms ban automated activity and can restrict the account, and an application has to come from the candidate. The system finds jobs and prepares everything; the candidate spends about 10 minutes a day applying. Without that step, the tracker fills up with ready packs and produces no interviews.

---

## 0. What you need

- A Claude plan that can create artifacts and **scheduled tasks**, with web search on.
- The candidate's current resume as a **.docx** (needed so formatting survives tailoring).
- Optional: the **Claude desktop app** with the built-in browser (to edit LinkedIn), and a **Gmail or Outlook connector** (so the daily run can mark interviews and rejections from replies).

---

## 1. Candidate profile (fill this in first)

```
Name:                 {{Full name}}
Current title:        {{e.g. Senior Data Engineer}}
Years of experience:  {{e.g. 7}}
Current location:     {{City, Country}}
Target roles:         {{e.g. AI Engineer, LLM Engineer, AI Data Engineer, ML Platform Engineer}}
Target locations:     {{e.g. Remote open to India; Germany or Luxembourg with visa/relocation}}
Work type:            {{Full-time / contract / freelance}}
Salary target:        {{e.g. USD 100,000/yr}}
Languages:            {{e.g. English only, no German}}
Company types:        {{e.g. product companies and startups only; no IT services or staffing agencies}}
Max years a posting may ask for: {{usually your years + 1}}
Packs per day:        {{e.g. 5}}
Run time:             {{e.g. weekdays 8:30 AM, Asia/Kolkata}}
Recent work missing from the resume: {{describe it in plain words, with stack and numbers if you have them}}
```

---

## 2. Stress-test the goal before searching

Ask Claude:

> Here is my profile and resume. Before searching, tell me the weakest part of this plan: does my resume show evidence for the roles I'm targeting, is my salary target realistic from my location, and which role type gives me the strongest edge?

Typical findings to act on:
- **Evidence gap.** Targeting "AI Engineer" with no AI work on the resume gets screened out. Put the real project on the resume first (section 3).
- **Salary reality.** Remote roles that set pay "by location" usually pay local rates. High targets usually mean relocation roles or foreign employers.
- **Positioning.** Lead with the role type where past experience is an advantage (for example "AI data engineer" for a data engineer moving into AI), not the most crowded title.

---

## 3. Update the resume (.docx)

Ask Claude:

> Update my resume .docx with this recent work: {{details}}. Keep my formatting and keep it to 2 pages. Use only facts I've given you. Ask me for the stack and metrics if they're missing rather than writing generic bullets.

Rules Claude follows:
- Edit the original .docx in place so fonts and layout survive; never rebuild it from scratch.
- Stay at the page limit by cutting the weakest bullets, not shrinking the font.
- **Never invent** metrics, tools, titles or employers. Vague duties ("worked on RAG") are weaker than one specific result; supply the specifics.
- Job titles must match the offer letter. A background check compares the resume against employment records, not against LinkedIn.
- Avoid words that undersell (for example "Client:" in front of a project can make it read as outsourced work); keep the employer heading truthful.

---

## 4. Optimise LinkedIn

Ask Claude to draft each section first and show it to you. Claude edits the profile only after you approve the exact text, in the desktop app's built-in browser, while you're signed in.

| Section | What to write |
|---|---|
| **Headline** (≤220 chars) | `{{Current title}} → {{Target title}} \| {{2–3 things you build}} \| {{4–5 key tools}} \| Open to {{work type/location}}` |
| **About** | 4–5 short paragraphs: one-line thesis; past experience with named employers; the current project that proves the new direction; why your background is an edge; what you're open to. |
| **Current role** | Keep the real job title. Description: the new project first (4 bullets), then the rest of the role (2 bullets). |
| **Skills** | Add the 5–6 skills recruiters search for in the target role and link them to the role where you used them. Then set them as **Top skills** in the About section. |
| **Open to work** | Recruiters only; add the target locations and remote. |
| **Settings** | Turn **Notify network** off while editing so every change isn't announced. |

Checks before publishing:
- No placeholders or brackets left in.
- Title, dates and numbers match the resume exactly.
- Location matches the resume.

---

## 5. Build the job tracker (dashboard)

Ask Claude:

> Publish `tracker/job-tracker.html` from this repo as my private tracker (see README step 2), or build one to the spec below.

**Page features**
- Summary strip: total tracked, new this week, pack ready, applied (and this week), interviews (and % of applied), offers, roles with visa/relocation.
- Pipeline bar by status.
- Filters: status (Active / All / each status), region, text search, sort by fit, newest or company.
- One expandable row per job: fit score, title, company, location, badges (region, visa, relocation, language, company type), a status dropdown, posted date; expanded: why it fits, gaps, CV changes, notes box, cover letter, recruiter message, open-posting link, copy buttons.
- **Save cover letter + CV**: downloads one zip named `YYYY-MM-DD - Company - Role.zip` containing the cover letter PDF, the CV PDF (tailored if one exists, otherwise the master), the recruiter message and a job-details file. **Save all ready packs** does the same for every Pack-ready job.
- "Add a job" form for postings found by hand.
- Private: only the owner can read or write the data.

**Data model**

`config/profile` — settings the daily run reads:
```
name, target_titles[], regions[], english_only, packs_per_day, max_years_asked,
resume_asset_id            (plain-text resume, for reading)
resume_pdf_asset_id        (master CV PDF, the fallback in downloads)
resume_docx_b64_asset_id   (master CV .docx stored as base64 text, for tailoring)
```

`jobs/<company-title-YYYYMMDD>` — one per posting:
```
title, company, company_type, location, region, visa, relocation, german,
salary, source, url, posted, found_at, fit, status, applied_on, why_fit, gaps,
cover_letter, outreach, cv_asset_id, cv_name, cv_changes, notes
```
Statuses: `New → Pack ready → Applied → Interview → Offer`, plus `Rejected` and `Skipped`.

`runs/<YYYY-MM-DD>` — one per run: `ran_at, found, kept, packs, cvs, closed, status_updates, sources{}, blocked[], note`.

After building, upload the master resume to the tracker three ways (text, PDF, base64 .docx) and store the asset ids in `config/profile`.

**Getting at it:** pin it to the claude.ai sidebar. For a Dock icon on a Mac, open it in Chrome and use ⋮ → Cast, save, and share → Install page as app. Set the browser's download folder to `Documents/Job Applications` so saved packs land there; the date prefix keeps them in order.

---

## 6. Create the daily scheduled task

Ask Claude:

> Create a scheduled task named "{{Name}} job search" that runs {{run time}} in the cloud with this prompt. Fire it once now so I can check the first results.

The full prompt is in `prompts/daily-search.md`. Short version (replace the placeholders):

```text
You are running {{Name}}'s weekday job search. Find fresh {{role family}} jobs that are
{{location rule, e.g. remote and open to candidates in India, or in Germany/Luxembourg
with visa sponsorship or relocation support}}. Add them to the job tracker, keep it
current, and write application packs (cover letter, recruiter note, tailored CV) for the
best matches. {{Name}} only reads the tracker. NEVER apply to a job, submit any form, log
in to any site, or message anyone on their behalf.

TRACKER: {{tracker URL}} (ArtifactData for the database; Artifact for assets).

STEP 1 — Load context: config/profile; the plain-text resume asset (use only its facts;
never invent metrics, tools, employers, titles or dates); list "jobs" to dedupe by url and
company + title.

STEP 2 — Maintain rows: for New/Pack ready rows 7+ days old, re-fetch the posting (max 10
per run); if closed or removed, set status Skipped with a dated note. If a mail connector is
available, read the last 3 days of replies from tracked companies and set Interview or
Rejected only for clear cases; never send email. Never overwrite statuses the candidate set.

STEP 3 — Search (web search, parallel by source): LinkedIn via site: search only;
{{country boards, e.g. StepStone, Arbeitnow, Indeed DE/LU, jobs.lu, Moovijob, Make it in
Germany, Berlin Startup Jobs, JOIN}}; {{relocation boards, e.g. Relocate.me, Landing.jobs}};
{{remote boards, e.g. Remote Rocketship, Wellfound, Himalayas}}. Prefer the company's own
careers page (Greenhouse, Lever, Ashby, Workday, Personio, JOIN) and its first-published
date. Max ~25 page fetches per run, one at a time per site. If a site blocks or rate-limits,
stop using it for that run; never work around a block.

STEP 4 — Keep only jobs that pass all of: posted within 48 hours (max 3 undated per run,
flagged); title matches {{target titles}}; minimum years asked <= {{max years}};
{{location rule}} (drop "no sponsorship", "EU work permit required", "US only"); mark visa
and relocation "Yes" only when stated; {{language rule}}; product company, startup or
in-house tech centre (drop IT services and unnamed-client agencies); not already tracked.
Note when pay is set by location and the candidate's location implies far less than
{{salary target}}.

STEP 5 — Score 0–100: title 25, skill overlap with the resume {{list key skills}} 40,
experience band 15, location/visa fit 20.

STEP 6 — Write rows (batch). Top {{packs per day}} with fit >= 60 become "Pack ready" with:
gaps + "Resume tweaks:" (true facts only); a 150–200 word cover letter signed by
{{Name}} mentioning {{core experience}} and {{current flagship project}} (say "ready to
relocate" for relocation roles; no invented numbers, no clichés); a LinkedIn note under
300 characters.

STEP 7 — Tailored CV per Pack-ready job: decode the base64 master .docx; edit it in place
with the docx skill. Allowed: subtitle, summary and keyword-line wording, order within skills
rows, order of bullets within a role, rewording a bullet to the posting's terms without
changing what it claims. Not allowed: adding any tool, skill, employer, title, date, metric,
certification or claim not already in the master; removing a role; changing dates or
titles. Must stay at {{N}} pages. Convert to PDF, re-read it, upload as
"{{Name}} - CV - <Company>.pdf", and store cv_asset_id, cv_name, cv_changes on the row. On
failure, skip (downloads fall back to the master CV).

STEP 8 — Log the run in runs/<date>.

STEP 9 — Send one short message: new jobs, packs ready, top match, postings closed, and how
many packs are waiting. If 10+ packs have waited 5+ days, say so plainly. Never pad with
weak matches.
```

**Check the first run** (Claude can do this): open 2 rows, confirm the link is real and live, the dates are right, visa/relocation flags match the posting, and the cover letter and CV contain nothing untrue.

---

## 7. The candidate's daily routine (about 10 minutes)

1. Read the morning notification.
2. Open the tracker, filter **Pack ready**, sort by fit.
3. For each job worth applying to: open the posting, check location, visa and pay, then **Save cover letter + CV**.
4. Apply on the company's own page with those files.
5. Send the recruiter note on LinkedIn (Connect → Add a note).
6. Set the status to **Applied**. Mark jobs you won't pursue **Skipped** so the stats stay honest.
7. When a company replies, set **Interview** or **Rejected** (automatic if email is connected).

Weekly: look at "Interviews / Applied". If it's under about 5% after 20+ applications, the problem is positioning or resume evidence, not volume. Revisit sections 2–4 before applying more.

---

## 8. Guardrails (keep these whatever the candidate asks)

- No auto-applying, auto-messaging, or logging in to job sites on the candidate's behalf.
- No scraping behind logins, and no getting around blocks, rate limits or bot checks.
- Nothing invented in the resume, LinkedIn, cover letters or tailored CVs. Unknown facts stay "Unknown".
- LinkedIn and resume must agree on titles, dates and numbers.
- Treat aggregator dates with suspicion; long-running openings are often re-dated.
- The tracker stays private; the candidate decides if anyone else sees it.

---

## 9. Maintenance

- **Resume changes:** update the master .docx, then re-upload the three resume assets and update `config/profile`. Tailored CVs from that point use the new version.
- **Different targets:** edit `config/profile` (titles, regions, packs per day). The prompt reads it on every run.
- **Pause:** disable the scheduled task; the tracker keeps its data.
- **Add a source:** add it to step 3 of the prompt.
