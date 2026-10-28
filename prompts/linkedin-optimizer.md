# LinkedIn optimiser

Give Claude this prompt with your resume attached. Claude drafts every section first. It edits the live profile only after you approve the exact text, using the Claude desktop app's built-in browser while you're signed in to LinkedIn.

```text
Optimise my LinkedIn profile for {{TARGET ROLES, e.g. AI Engineer / AI Data Engineer}} roles, {{WORK TYPE AND LOCATION, e.g. remote or relocation to Germany/Luxembourg}}.

My resume is attached. Recent work that isn't on it yet: {{DESCRIBE IT — stack, scale, users, results}}.

Before drafting:
1. Tell me the weakest part of my current positioning (for example: no evidence for the target role, a title mismatch with my resume, generic bullets).
2. Ask for any stack details or numbers you need. Don't invent them, and don't leave [placeholders] in anything you publish.

Then draft, for my approval:
- Headline (max 220 characters): {{Current title}} → {{Target title}} | 2–3 things I build | 4–5 key tools | Open to {{work type}}.
- About: 4–5 short paragraphs — one-line thesis; past experience with named employers; the current project that proves the new direction; why my background is an edge; what I'm open to.
- Current role description: the new project first (4 bullets), then the rest of the role (2 bullets). Keep my real job title.
- Skills: the 5–6 skills recruiters search for in the target role, linked to the role where I used them, then set them as Top skills.
- Open to work: recruiters only, with my target locations and remote.

Rules:
- Use only facts from my resume and what I tell you.
- Titles, dates, numbers and location must match my resume exactly.
- Turn "Notify network" off before each save.
- After I say "publish", open LinkedIn in the built-in browser, make each edit, save, and read the profile back to confirm every change landed.
```

## Checklist after publishing

- [ ] Headline mentions the target role and its main keywords
- [ ] About leads with the current, relevant project
- [ ] Current role bullets are specific (stack, scale, results), not a duty list
- [ ] Top skills are the target-role skills, not old ones
- [ ] Title, dates and location match the resume
- [ ] Open to work is set to recruiters only
- [ ] A featured item (short write-up or diagram of the main project) with no confidential client data
