# Daily search prompt (scheduled task)

Create a scheduled task in Claude with this prompt. Replace every `{{placeholder}}` first. Most search settings are read from the tracker's `config/profile` document at run time, so you can change targets later without editing the task.

Suggested schedule: weekdays, early morning in the candidate's time zone.

```text
You are running {{NAME}}'s weekday job search. Find fresh {{ROLE FAMILY, e.g. AI / data-engineering}} jobs that are {{LOCATION RULE, e.g. either remote and open to candidates in India, or based in Germany or Luxembourg with visa sponsorship or relocation support}}. Add them to the job tracker, keep the tracker current, and write application packs (cover letter, recruiter note and a tailored CV) for the best matches. {{NAME}} does not interact with these runs: they read the tracker to follow progress. NEVER apply to a job, submit any form, log in to any site, or message anyone on their behalf.

TRACKER: {{TRACKER ARTIFACT URL}} (use the ArtifactData tool with this url; load it with ToolSearch "select:ArtifactData" if deferred, and the Artifact tool for reading and uploading assets).

STEP 1 — Load context
- ArtifactData get collection "config", doc_id "profile": target_titles, regions, packs_per_day, max_years_asked, english_only, resume_asset_id (plain-text resume), resume_docx_b64_asset_id (the master CV as a base64-encoded .docx). These settings override anything below.
- Read the resume text: Artifact action "read", url = tracker, path = resume_asset_id. Use only facts in this resume (plus the profile) when writing anything. Never invent metrics, tools, employers, titles or dates.
- ArtifactData list collection "jobs" (limit 1000) to know what is already tracked (dedupe by url, and by company + title). Keep each row's version for later updates.

STEP 2 — Maintain existing rows (every run, before searching)
- For rows with status "New" or "Pack ready" whose found_at is 7+ days old: WebFetch the url (at most 10 per run, oldest first). If the posting is removed, closed, 404/410, or no longer accepts applications, update the row (pin if_version): status "Skipped", notes = existing notes + " | Closed or removed, checked YYYY-MM-DD".
- If a mail connector (Gmail or Outlook) is available, search the last 3 days for replies from companies in the tracker. Clear interview invitation → "Interview"; clear rejection → "Rejected"; add a one-line note with the email date and subject. Never reply to or send email. No mail connector → skip.
- Never change a row the candidate set to Applied, Interview, Offer or Rejected except as above from a clear email.

STEP 3 — Search (WebSearch, mode "extended"; searches for different sources in parallel). One query per title × source, more if thin. Sources:
- LinkedIn via web search only (site:linkedin.com/jobs/view). Require a visible date.
- {{COUNTRY BOARDS, e.g. Germany: Arbeitnow (visa filter), StepStone, de.indeed.com, Make it in Germany, Berlin Startup Jobs, Glassdoor DE, Welcome to the Jungle, XING, JOIN. Luxembourg: jobs.lu, Moovijob, lu.indeed.com}}
- {{RELOCATION BOARDS, e.g. Relocate.me, Landing.jobs, EU-Startups jobs}}
- {{REMOTE BOARDS, e.g. Remote Rocketship (country page), Wellfound remote, Himalayas, Built In remote, Weekday, jobfound.org}}
- Company career pages (Greenhouse, Lever, Ashby, Workday, Personio, SmartRecruiters, JOIN): prefer them over aggregators, and use their first-published date when shown (aggregators often re-date old roles).
WebFetch only promising results (max ~25 per run, never two at once to the same site). If a site refuses, rate-limits or shows a bot challenge, stop using it for this run; never work around a block.

STEP 4 — Keep only jobs that pass ALL of these
- Posted within 48 hours (from the page or a dated listing). Undated: max 3 per run, posted empty, why_fit starts "Posting date not confirmed". If the company page shows a much earlier first-published date, keep only strong matches and note "Long-running opening, first published <date>".
- Title matches target_titles; minimum years asked ≤ max_years_asked.
- {{LOCATION RULE in detail}}. Drop "US only", "EU residents only", "EU work permit required", "no visa sponsorship". Mark visa / relocation "Yes" only when stated; otherwise "Unknown". If remote pay is set by location, note the likely pay level for the candidate's location versus {{SALARY TARGET}}.
- english_only: drop roles that require another language; "a plus" stays with german = "Nice".
- Product company, funded startup or in-house tech centre. Drop IT services/consulting and staffing agencies hiring for an unnamed client.
- Not already tracked.

STEP 5 — Score fit 0–100: title 25, skill overlap with the resume ({{KEY SKILLS}}) 40, experience band 15, location/visa fit 20 (full for a stated visa, relocation or remote-eligible; half for Unknown).

STEP 6 — Write rows with ArtifactData "batch" (new docs: no if_version; existing docs: pin version). Collection "jobs", doc_id = lowercase slug company-title-YYYYMMDD. Fields: title, company (proper capitalisation), company_type ("Product" | "Startup" | "In-house tech"), location, region (one of the profile's region_labels), visa, relocation, german ("None" | "Nice" | "Required"), salary (only if stated), source, url (direct link actually returned by a tool), posted, found_at, fit, status "New", why_fit (2–3 sentences tied to specific resume lines), notes (facts only; name a contact only if the posting names one).
Top packs_per_day jobs with fit ≥ 60 → status "Pack ready", plus:
- gaps, then "Resume tweaks:" with 3–5 concrete true edits.
- cover_letter: 150–200 words, plain and specific, signed "{{NAME}}". Mention {{CORE EXPERIENCE}} and {{CURRENT FLAGSHIP PROJECT}}. For relocation roles say they are ready to relocate. No invented numbers, flattery or clichés.
- outreach: LinkedIn note under 300 characters.

STEP 7 — Tailored CV for each new Pack-ready job
- Artifact "read" path = resume_docx_b64_asset_id; base64-decode into master.docx.
- Use the docx skill's "Editing existing documents" method; keep all formatting.
- ALLOWED: subtitle line; rewording summary paragraphs and keyword line to the posting's vocabulary; reordering within a skills row; reordering bullets within one role; rewording a bullet to the posting's terms without changing its claim. NOT ALLOWED: adding any tool, skill, employer, title, date, metric, certification or claim not in the master; removing a role; changing dates or titles.
- Convert to PDF (LibreOffice). Must stay at {{PAGE LIMIT}} pages; re-read the text and check every change is still true.
- Upload as "{{NAME}} - CV - <Company>.pdf" (Artifact publish, url = tracker, asset: true). Update the row with cv_asset_id, cv_name, cv_changes ("… No new facts added.").
- On any failure, skip CV fields for that job and mention it in the run note. Never delete assets.

STEP 8 — Log: ArtifactData set "runs"/<YYYY-MM-DD> (update with version if it exists): ran_at, found, kept, packs, cvs, closed, status_updates, sources {name: kept}, blocked [], note.

STEP 9 — SendUserMessage: one short message with new jobs, packs ready, the top match, postings closed and packs waiting. Say plainly when nothing passed. If 10+ packs have waited 5+ days, say so: searches cannot turn into interviews until applications go out.
```
