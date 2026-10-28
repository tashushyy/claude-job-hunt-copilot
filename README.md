# Claude Job Hunt Copilot

Run a focused job search with Claude. Every weekday morning a scheduled task searches LinkedIn (via web search) and the main job boards for fresh roles that match your profile, scores them against your resume, and writes an application pack for the best ones: a cover letter, a recruiter note and a CV tailored to the posting. Everything lands in a private tracker dashboard where you apply, save the documents as PDFs, and record your progress.

**You still apply yourself.** Claude never submits applications, sends messages or logs in to job sites for you. LinkedIn's terms ban automated activity, and an application has to come from you. Expect about 10 minutes a day: open the ready packs, apply, mark them Applied.

## What's in this repo

| Path | What it is |
|---|---|
| `PLAYBOOK.md` | The full process, including resume and LinkedIn optimisation, with prompts for each step. Start here. |
| `tracker/job-tracker.html` | The tracker dashboard. Claude publishes it as a private artifact for you. |
| `prompts/daily-search.md` | The scheduled-task prompt template that runs the morning search. |
| `config/profile.example.json` | The settings document the tracker and daily run read. |

## Requirements

- A Claude plan with **artifacts** and **scheduled tasks**, and web search turned on.
- Your resume as a **.docx** (formatting is preserved when CVs are tailored).
- Optional: the Claude desktop app (lets Claude edit LinkedIn with you in its built-in browser) and a Gmail or Outlook connector (lets the daily run mark interviews and rejections from replies).

The tracker uses the claude.ai artifact runtime (its database, file assets and downloads). It only works when published as a Claude artifact, not as a standalone web page.

## Setup (about 30 minutes, mostly in one chat with Claude)

**1. Profile and resume.** Fill in the candidate profile in `PLAYBOOK.md` §1, attach your resume .docx, and work through §2–§4 with Claude: stress-test the plan, update the resume, optimise LinkedIn.

**2. Publish the tracker.** Attach `tracker/job-tracker.html` and send:

> Publish this file as a private artifact with capabilities `{"db": {"rules": [{"path": "", "read": "admin", "write": "admin"}]}, "user": {}, "assets": {}, "downloads": true}` and icon "briefcase". Then pin it to my sidebar.

**3. Load your resume and settings.** With your final resume .docx attached, send:

> Upload my resume to the tracker three ways: plain text, PDF, and the .docx base64-encoded as a .txt file. Then write config/profile using `config/profile.example.json` from this repo, filled in with my profile and the three asset ids.

**4. Create the daily search.** Fill in the placeholders in `prompts/daily-search.md` (your name, the tracker URL, your location rule, boards, skills, page limit), then send:

> Create a scheduled task named "<Your name> job search" that runs weekdays at <time> in <time zone>, in the cloud, with this prompt. Fire it once now and check the first results with me.

**5. Make files land in Documents.** Create `Documents/Job Applications` and set it as your browser's download folder. Each saved pack is a date-stamped zip (`YYYY-MM-DD - Company - Role.zip`) holding the cover letter PDF, the tailored CV PDF, the recruiter note and the job details, so the folder sorts by date.

**6. Optional.** Connect Gmail or Outlook in Claude so replies update statuses automatically. On a Mac, open the tracker in Chrome and use ⋮ → Cast, save, and share → Install page as app to give it its own Dock icon.

## Daily routine

1. Read the morning notification.
2. Open the tracker, filter **Pack ready**, sort by fit.
3. Check location, visa and pay on the posting. Click **Save cover letter + CV**.
4. Apply on the company's own page. Send the recruiter note on LinkedIn.
5. Set the status to **Applied**, or **Skipped** if you pass on it.

If interviews stay under about 5% of applications after 20 or more, fix positioning and resume evidence (`PLAYBOOK.md` §2–§4) before applying more.

## Guardrails built into the prompts

- No auto-applying, messaging or logging in to job sites; no getting around blocks or bot checks.
- Nothing invented in cover letters or tailored CVs. Tailoring can reword and reorder, never add a tool, metric, employer, title or claim that isn't in your master CV.
- Visa and relocation are marked "Yes" only when the posting says so.
- Aggregator dates are checked against the company's own page, since old roles are often re-dated.
- The tracker is private to you. Your resume and job data never go into this repo.

## Customising

- Change targets, regions or packs per day in `config/profile` in the tracker; the next run picks them up.
- Add or remove job boards in step 3 of the scheduled-task prompt.
- Change the tracker's title and subtitle with `tracker_title` and `tracker_subtitle` in `config/profile`.

## License

MIT. See `LICENSE`.
