---
name: daily-verified-job-search
description: Daily, evidence-backed job discovery and dashboard refresh. Use when the user wants an AI agent to analyze a resume, find current jobs, verify application links, rank opportunities, refresh a saved job set, or run a recurring job-search automation.
---

# Daily Verified Job Search

Use this skill to maintain a current, evidence-backed job shortlist from a resume or candidate profile. The workflow is portable across Claude, Manus, ChatGPT-style agents, and other tool-using systems. Adapt the web-search, browser, storage, scheduler, and dashboard-update steps to the host platform.

## Operating rules

- Treat the resume and the user's target preferences as the source of truth for fit.
- Search current web pages, not remembered listings or search snippets alone.
- Verify every included role on the live listing page and its application destination before including it.
- Prefer the employer's own careers page and application system. Use a reputable job platform only when the listing is live, the employer is identifiable, and the application flow is accessible.
- Exclude expired, closed, duplicate, suspicious, senior-only, irrelevant, or materially mislocated listings.
- Never fabricate a job, requirement, salary, location, application URL, verification result, or recruiter identity.
- Never submit an application, message a recruiter, or make another external commitment without explicit user approval.
- Preserve uncertainty beside the affected job. Use “not stated” or “unknown” instead of inference.
- During refreshes, remove stale roles first, then add newly verified roles. Do not retain a role merely because it appeared in an earlier run.

## Inputs

Collect or infer these inputs before searching:

1. Resume file or structured candidate profile.
2. Target geography, work mode, role families, seniority, and eligibility constraints.
3. Existing job dataset, if this is a refresh.
4. Dashboard or storage destination, if one exists.
5. Refresh cadence and timezone, if automation is requested.

If a material input is missing, ask one focused clarification question. Otherwise proceed with conservative assumptions and state them in the result.

## Resume analysis

Extract and summarize:

- Technical skills grouped by language, frontend, backend, data, cloud, testing, tools, and domain.
- Formal experience versus project-based experience, with dates and ownership level.
- Education, graduation date, current student status, and work authorization when stated.
- Flagship projects, deployment evidence, public repositories, outcomes, and measurable signals.
- Certifications, competitive-programming evidence, open-source work, leadership, and communication strengths.
- Differentiators, missing proof, eligibility constraints, and likely seniority.

Recommend exactly five role families unless the user asks for another number. Rank them by demonstrated evidence, not generic popularity.

## Discovery and verification

Search in batches by role family and employer type. Prefer a mix of direct company pages, strong startups, MNCs, and reputable job platforms. For each candidate listing:

1. Open the live listing page.
2. Confirm the exact title, employer, location, work mode, experience requirement, current/open wording, and material skills.
3. Open the application destination. Confirm it is reachable and is an official employer/ATS flow, a reputable platform application, or an explicitly provided official recruiting email.
4. Record the evidence source and checked date.
5. Compare requirements with the resume. Separate demonstrated matches from assumptions.
6. Deduplicate by normalized employer + title + location + requisition/application URL.
7. Assign `Apply First`, `Good Match`, or `Stretch Role` and a fit score from 0–100.

A role is included only when `verified_current=true`, `duplicate_or_excluded=false`, the application destination is credible, and the role is relevant to the user's current eligibility. If fewer roles survive than requested, report the actual count and do not pad the set.

## Fit scoring

Use a consistent rubric and explain material deductions:

- 30 points: direct technical stack overlap.
- 20 points: seniority, student/fresher, graduation, and location eligibility.
- 20 points: demonstrated project or work evidence relevant to the role.
- 15 points: core fundamentals and interview readiness.
- 10 points: domain, product, or company relevance.
- 5 points: likelihood of success based on missing mandatory requirements.

A mandatory unmet requirement can lower the score sharply or force `Stretch Role`, even when the stack match is strong.

## Refresh behavior

On every scheduled run:

1. Load the prior job dataset and candidate profile.
2. Re-open each stored listing and application URL.
3. Mark closed, expired, missing, duplicate, suspicious, or materially changed roles for removal.
4. Search for replacement roles using current queries and the same eligibility rules.
5. Verify replacements independently; never reuse old evidence as proof of current availability.
6. Recalculate fit scores, categories, Top 10 ranking, resume improvements, and recruiter messages.
7. Atomically replace the visible job set, preserving only the newest verified records.
8. Store `lastRefreshedAt`, `verifiedAt`, `listingUrl`, `applyUrl`, `sourceType`, `verificationEvidence`, and a stable deduplication key.
9. If the refresh fails, do not silently present stale data as current. Show the last successful refresh timestamp and the failure reason.

## Output schema

Return or persist records with this shape:

```json
{
  "lastRefreshedAt": "RFC3339 timestamp",
  "candidate": {
    "name": "...",
    "seniority": "...",
    "targetRoles": ["..."],
    "strengths": ["..."],
    "gaps": ["..."]
  },
  "jobs": [
    {
      "id": "stable-deduplication-key",
      "title": "...",
      "company": "...",
      "location": "...",
      "experienceRequired": "...",
      "summary": "...",
      "listingUrl": "https://...",
      "applyUrl": "https://...",
      "sourceType": "official-careers | official-ats | reputable-platform | official-email",
      "verifiedCurrent": true,
      "verifiedAt": "RFC3339 timestamp",
      "verificationEvidence": "...",
      "category": "Apply First | Good Match | Stretch Role",
      "fitScore": 0,
      "whyMatch": "...",
      "missingSkills": "...",
      "priorityRank": 1,
      "resumeImprovements": ["..."]
    }
  ],
  "topFiveRecruiterMessages": [
    {"jobId": "...", "message": "..."}
  ]
}
```

## Dashboard and automation adapters

For a dashboard, expose search, filters for category/company/location/experience/role, fit-score indicators, direct apply buttons, verification dates, why-match details, missing skills, resume improvements, and recruiter messages. Keep application links exact and open them in a new tab.

For scheduling, use the host's recurring scheduler. For a daily 9:00 AM India schedule, use the platform's Asia/Kolkata or Asia/Calcutta timezone and its six-field cron equivalent of `0 0 9 * * *` when that syntax is supported. In Claude or another tool without a built-in scheduler, pair this skill with cron, GitHub Actions, Zapier/Make, or the platform's scheduled-task feature. The scheduler should invoke the same refresh prompt and provide access to the resume, prior JSON dataset, and dashboard update endpoint.

## Final response

Report the number of verified active jobs, the refresh timestamp, the five target role families, the Top 10, and any evidence limits. Link the dashboard or JSON output when available. Make clear that the workflow searches and prepares opportunities but does not apply automatically.
