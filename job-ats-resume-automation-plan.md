# Automation Plan: ATS Resume Creation for Performance & Digital Marketing Roles

**Target roles:** Performance Marketing, Digital Marketing

**Target seniority:** Senior / 1+ year experience (recommended: make the exact range configurable, for example 1–3 years)

**Requested stack:** Google Antigravity as the development/orchestration workspace, GitHub for source control and CI/CD, Naukri as the target job platform

**Document purpose:** Define the complete architecture, requirements, credential model, phased execution plan, prompt library, testing strategy, and rollout controls for an automation that discovers suitable jobs and creates tailored, ATS-friendly resumes.

**Recommended operating principle:** Automate research, extraction, scoring, resume drafting, validation, and tracking first. Keep final job submission as a human-approved action until the platform's terms, account permissions, and technical interfaces clearly permit automated submission.

---

## 1. Executive Summary

The automation should:

1. Collect the candidate's master profile, experience, achievements, skills, links, and resume history.
2. Discover relevant Performance Marketing and Digital Marketing jobs from approved sources.
3. Extract and normalize each job description.
4. Score each job against the candidate profile.
5. Identify ATS keywords and role-specific requirements.
6. Generate a tailored resume for each approved job without inventing experience.
7. Run ATS, quality, factuality, and formatting checks.
8. Produce a review package containing the job, match score, resume, cover note, missing information, and recommended next action.
9. Optionally open Naukri for the user to review and submit manually, or use an officially supported API/integration if one is available and authorized.
10. Track applications, versions, outcomes, and improvement signals in a structured database or spreadsheet.

### Core design decision

Use a **human-in-the-loop application workflow**:

- Fully automate: job discovery, deduplication, parsing, scoring, keyword analysis, resume generation, validation, and reporting.
- Require user approval: final resume selection and any external application submission.
- Do not bypass CAPTCHA, rate limits, login protections, anti-bot controls, or Naukri access restrictions.

---

## 2. Scope and Non-Goals

### In scope

- Performance Marketing and Digital Marketing job discovery.
- Seniority filtering with configurable experience range.
- ATS-focused resume tailoring.
- Job-description parsing.
- Keyword and competency gap analysis.
- Resume generation in Markdown, DOCX, and PDF.
- GitHub-hosted source code and GitHub Actions automation.
- Optional scheduled runs.
- Application tracking and audit history.
- Naukri review-assist workflow.

### Out of scope for the first release

- Automatic mass application submission.
- CAPTCHA solving or bypassing.
- Creating false experience, certifications, metrics, employers, or education.
- Scraping behind login without explicit authorization.
- Sending unsolicited messages to recruiters.
- Automatically changing the candidate's Naukri profile.
- Buying paid scraping or job-board services without the user's approval.

---

## 3. Recommended Architecture

```text
                    +-----------------------------+
                    | Candidate Master Profile    |
                    | skills, facts, achievements |
                    +-------------+---------------+
                                  |
                                  v
+----------------+     +----------+-----------+     +-------------------+
| Job Sources    | --> | Ingestion & Parsing  | --> | Job Normalization |
| Naukri/manual  |     | HTML/API/export     |     | schema + dedupe    |
| approved APIs  |     +----------+-----------+     +---------+---------+
+----------------+                |                           |
                                  v                           v
                         +--------+---------+        +--------+---------+
                         | Relevance Scorer |        | Keyword/Gaps    |
                         +--------+---------+        +--------+---------+
                                  |                           |
                                  +-------------+-------------+
                                                v
                                    +-----------+------------+
                                    | Resume Tailoring Engine |
                                    | LLM + rules + templates |
                                    +-----------+------------+
                                                |
                                                v
                          +---------------------+--------------------+
                          | Validation & Quality Gate              |
                          | factuality, ATS, length, formatting     |
                          +---------------------+--------------------+
                                                |
                                                v
                           +-------------------+--------------------+
                           | Review Package / Approval Queue       |
                           | resume + cover note + score + source  |
                           +-------------------+--------------------+
                                               |
                               +---------------+----------------+
                               |                                |
                               v                                v
                    +----------+----------+          +----------+----------+
                    | User review / Naukri |          | Tracker / Analytics |
                    | manual submission    |          | status + outcomes   |
                    +---------------------+          +---------------------+
```

### Suggested components

| Layer | Recommended option | Notes |
|---|---|---|
| Development | Google Antigravity | Use as the coding and agent workspace; confirm its actual runtime, browser, secrets, and scheduled-job capabilities. |
| Source control | GitHub | Private repository recommended. |
| CI/CD | GitHub Actions | Scheduled discovery, tests, validation, and artifact generation. |
| Job acquisition | Official API, approved feed, user export, or manual URL first | Prefer official/authorized access. Apify is optional, not automatically appropriate. |
| Browser assistance | Playwright or a supported browser connector | Use only where permitted; do not bypass platform controls. |
| LLM | Approved model available in Google Antigravity or a server-side provider | Use structured JSON output and low-temperature generation for consistency. |
| Data store | SQLite for MVP; PostgreSQL/Supabase for multi-device production | Store source URLs, job hashes, resume versions, and statuses. |
| Document rendering | DOCX template plus PDF conversion | Keep an editable DOCX and a final PDF. |
| Notifications | Email, Slack, Telegram, or GitHub issue/comment | Optional; protect personal data. |
| Secrets | GitHub Actions Secrets / environment secret manager | Never commit passwords, tokens, cookies, or API keys. |

---

## 4. Functional Requirements

### 4.1 Candidate profile

The system needs a structured, factual candidate profile:

- Full name and preferred contact details.
- City, country, relocation preference, and work authorization.
- Target titles.
- Years of experience and employment dates.
- Employers, industries, products, budgets, channels, and markets.
- Paid media platforms: Google Ads, Meta Ads, LinkedIn Ads, etc.
- Analytics and measurement: GA4, GTM, Looker Studio, attribution, conversion tracking.
- SEO, CRM, lifecycle marketing, CRO, email, content, or other relevant skills.
- Verified achievements with numbers and context.
- Education and certifications.
- Portfolio, LinkedIn, website, and campaign case studies.
- Preferred work mode, salary range, notice period, and location.
- Skills the candidate does not have, so the automation does not claim them.

### 4.2 Job ingestion

Each job record should capture:

- Source platform.
- Original URL.
- Job title.
- Company.
- Location and remote/hybrid status.
- Salary, if available.
- Required experience.
- Responsibilities.
- Must-have skills.
- Nice-to-have skills.
- Tools/platforms.
- Industry.
- Application method.
- Date discovered.
- Job description text and a content hash.
- Evidence/source timestamp.

### 4.3 Resume generation

The system should generate:

- One-page resume when the experience level and content justify it.
- Two-page resume when needed for relevant senior experience.
- ATS-safe layout with standard headings.
- Tailored professional summary.
- Tailored skills section.
- Reordered and rewritten experience bullets based only on verified facts.
- Relevant projects or certifications.
- Optional concise cover note.
- A change log showing what was tailored.

### 4.4 Quality gates

A resume cannot enter the approval queue unless it passes:

- No unsupported claims.
- No invented metrics.
- No keyword stuffing.
- No missing contact details.
- No broken links.
- No duplicate bullets.
- No unusual tables, text boxes, icons, headers, or footers that may reduce ATS parsing quality.
- Reasonable length and readable formatting.
- Job-specific relevance above a configurable threshold.

---

## 5. Job Source Strategy: Naukri, Apify, and Alternatives

### 5.1 Preferred source order

1. Official Naukri API, partner feed, export, or integration—if available to the account and contractually allowed.
2. Naukri jobs manually supplied by the user via URLs or exported data.
3. A permitted browser-assisted workflow where the user is logged in and actively reviews results.
4. Other job sources with official APIs or clearly permitted feeds.
5. Third-party scraping only after checking the source's terms, robots guidance, rate limits, data-license restrictions, and the service's acceptable-use rules.

### 5.2 Direct Naukri visit

A direct Naukri visit can be used for a **review-assist workflow**:

- Search using saved criteria.
- Open a job page.
- Copy or export the job description where allowed.
- Send the job URL/text to the tailoring engine.
- Generate a resume and recommendation.
- Return the user to Naukri for final review and submission.

The workflow should stop if it encounters CAPTCHA, a blocked page, a login challenge, an anti-automation message, or a rate-limit warning.

### 5.3 Apify

Apify may be used only if:

- The actor is authorized for the target source.
- Its terms permit the intended use.
- The source and actor do not prohibit automated collection.
- The rate, volume, and storage policy are acceptable.
- The user has approved any paid usage.
- Personal data is stored and deleted according to the user's policy.

Apify is **not required** for the MVP. Start with manual URLs or approved feeds. Add Apify only when the source and actor have been reviewed and the volume justifies it.

### 5.4 Recommended MVP job input

For the first version, support these inputs:

- Paste a job description into a form or Markdown file.
- Provide a public job URL.
- Upload a CSV/JSON export.
- Add a Naukri URL manually after the user has reviewed it.

This eliminates the highest-risk part—uncontrolled job-board scraping—while validating the resume-generation value quickly.

---

## 6. Credentials and Access Requirements

### 6.1 Required for the MVP

| Credential/access | Required? | Purpose |
|---|---:|---|
| GitHub account | Yes | Repository, issues, Actions, artifacts. |
| Private GitHub repository access | Yes | Store code and non-sensitive templates. |
| Google Antigravity access | Yes | Development/orchestration environment. |
| LLM access | Yes | Parse jobs and tailor resumes. Use the provider configured in the environment. |
| Candidate profile data | Yes | Source of factual resume content. |
| Naukri username/password | No for MVP | Avoid storing credentials; use manual login or an approved connector. |
| Naukri URL/job text | Yes for Naukri-specific tailoring | User supplies a job URL or text. |

### 6.2 Optional credentials

| Credential/access | When needed | Handling rule |
|---|---|---|
| Naukri official API token/OAuth | Only if Naukri provides it for the account/use case | Store as an encrypted secret; verify scopes and terms. |
| Apify API token | If approved Apify actor is used | Store in GitHub Actions Secrets; set spending and run limits. |
| Email SMTP/API | Notifications | Use a dedicated sender and minimum scopes. |
| Slack/Telegram token | Notifications | Use a private channel and minimum scopes. |
| Database URL | Hosted PostgreSQL/Supabase | Store as an environment secret. |
| Cloud storage credentials | Resume artifact storage | Use a private bucket and expiring links. |
| GitHub Personal Access Token | Only if `GITHUB_TOKEN` is insufficient | Prefer built-in `GITHUB_TOKEN`; use least privilege. |

### 6.3 Do not request or store

- Passwords in Markdown, source files, prompts, or chat.
- Browser session cookies in GitHub.
- CAPTCHA answers or bypass tokens.
- Full email inbox access unless essential and explicitly authorized.
- Broad GitHub organization administration scopes.
- Payment credentials.

### 6.4 Recommended GitHub Secrets

```text
LLM_API_KEY
LLM_API_BASE_URL                 # only if the selected provider requires it
APIFY_API_TOKEN                  # optional
DATABASE_URL                     # optional
NOTIFICATION_WEBHOOK_URL         # optional
SMTP_USERNAME                    # optional
SMTP_PASSWORD                    # optional
Naukri_API_TOKEN                 # only if officially available and approved
```

Use GitHub Environments such as `staging` and `production`, with approval protection on production.

---

## 7. Data Model

### CandidateProfile

```json
{
  "candidate_id": "candidate-001",
  "target_roles": ["Performance Marketing Manager", "Digital Marketing Manager"],
  "experience_years": 2,
  "location": "",
  "work_authorization": "",
  "summary_facts": [],
  "skills": [],
  "tools": [],
  "achievements": [
    {
      "statement": "",
      "metric": "",
      "time_period": "",
      "evidence": ""
    }
  ],
  "employment": [],
  "education": [],
  "certifications": [],
  "links": [],
  "negative_constraints": ["Do not claim tools not present in this profile"]
}
```

### JobRecord

```json
{
  "job_id": "sha256-of-source-and-title",
  "source": "naukri",
  "url": "",
  "title": "",
  "company": "",
  "location": "",
  "employment_type": "",
  "experience_min": null,
  "experience_max": null,
  "description_raw": "",
  "must_have_skills": [],
  "nice_to_have_skills": [],
  "tools": [],
  "responsibilities": [],
  "discovered_at": "",
  "content_hash": "",
  "source_evidence": ""
}
```

### ApplicationRecord

```json
{
  "application_id": "",
  "job_id": "",
  "resume_version": "",
  "match_score": 0,
  "status": "review",
  "user_decision": "pending",
  "submitted_at": null,
  "outcome": null,
  "notes": ""
}
```

---

## 8. Match Scoring Model

Use a transparent score rather than an unexplained LLM decision.

| Factor | Weight |
|---|---:|
| Target title and role family | 20% |
| Required skills match | 25% |
| Platform/tool match | 15% |
| Experience range fit | 15% |
| Industry/function relevance | 10% |
| Location/work-mode fit | 5% |
| Evidence-backed achievement relevance | 10% |

### Example thresholds

- **80–100:** Strong fit; generate tailored resume automatically for review.
- **65–79:** Potential fit; generate only if the user has enabled medium-confidence jobs.
- **50–64:** Keep in research queue; no resume by default.
- **Below 50:** Reject or archive.

The score should always show its contributing factors and missing requirements.

---

## 9. Phase-by-Phase Execution Plan

## Phase 0 — Confirm scope and compliance

**Goal:** Finalize what the automation is allowed to do.

### Tasks

- Confirm target role titles.
- Confirm exact experience range.
- Confirm locations, remote preference, salary, notice period, and work authorization.
- Confirm whether the user wants only resume creation or also application tracking.
- Review Naukri terms, account permissions, and any approved integration.
- Decide whether final submission is manual or officially supported.
- Define data retention and deletion period.

### Deliverable

A signed-off configuration file such as `config/target-profile.yaml`.

### Exit criteria

- No ambiguity about target roles.
- No plan to bypass platform controls.
- Human approval is defined for final submission.

---

## Phase 1 — Repository and environment setup

**Goal:** Create a reproducible GitHub project.

### Suggested repository structure

```text
ats-resume-automation/
├── README.md
├── pyproject.toml
├── .env.example
├── .gitignore
├── config/
│   ├── target-profile.example.yaml
│   ├── scoring.yaml
│   └── policy.yaml
├── data/
│   ├── candidate-profile.example.json
│   └── sample-jobs/
├── prompts/
│   ├── parse-job.md
│   ├── score-job.md
│   ├── tailor-resume.md
│   ├── validate-resume.md
│   └── cover-note.md
├── src/
│   ├── ingest/
│   ├── parse/
│   ├── score/
│   ├── resume/
│   ├── validate/
│   └── tracker/
├── templates/
│   ├── resume.md.j2
│   └── cover-note.md.j2
├── tests/
├── outputs/.gitkeep
└── .github/workflows/
    ├── test.yml
    └── scheduled-discovery.yml
```

### Recommended implementation choices

- Python for parsing, scoring, document generation, and tests.
- Pydantic or JSON Schema for structured data validation.
- Jinja2 for deterministic templates.
- SQLite for the MVP.
- Playwright only for permitted browser-assisted review, not for bypassing protections.
- DOCX generation with a stable template and PDF conversion in CI.

### Exit criteria

- Repository is private.
- Branch protection is enabled.
- Secrets are configured outside the repository.
- Local test command works from a clean checkout.

---

## Phase 2 — Build the candidate knowledge base

**Goal:** Create the single source of truth for resume facts.

### Tasks

- Collect the current resume and LinkedIn profile manually.
- Convert each role into structured employment data.
- Record every metric with its source/evidence.
- Separate verified facts from aspirational language.
- Add skills with proficiency and evidence.
- Add negative constraints, such as tools or domains not used.
- Create a fact registry so generated resumes can be audited.

### Exit criteria

- Every generated achievement is traceable to a candidate-provided fact.
- Missing metrics are marked as missing rather than guessed.
- The user approves the master profile.

---

## Phase 3 — Implement job ingestion

**Goal:** Accept safe, repeatable job inputs.

### MVP inputs

- Paste text.
- Upload Markdown, TXT, DOCX, CSV, or JSON.
- Enter a public URL.
- Add a manually reviewed Naukri URL.

### Optional later inputs

- Approved API.
- Approved job feed.
- Authorized Apify actor.
- Browser-assisted collection with explicit user action.

### Tasks

- Normalize whitespace and HTML.
- Remove navigation and unrelated page content.
- Detect title, company, location, experience, salary, and skills.
- Create a content hash.
- Deduplicate jobs by source URL, company/title, and description similarity.
- Preserve raw source text for auditability.

### Exit criteria

- The same job is not processed repeatedly.
- Source URL and timestamp are preserved.
- Ingestion fails safely when content is unavailable.

---

## Phase 4 — Parse and score jobs

**Goal:** Produce a transparent job-fit recommendation.

### Tasks

- Extract required and preferred skills.
- Normalize synonyms, for example `GA4` and `Google Analytics 4`.
- Detect seniority and minimum experience.
- Identify platform requirements such as Google Ads, Meta Ads, GA4, GTM, CRM, SEO, and reporting.
- Calculate the deterministic score.
- Ask the LLM for a short explanation and missing requirements.
- Store both raw extraction and normalized output.

### Exit criteria

- Score is reproducible from stored inputs.
- A reviewer can understand why the job was accepted or rejected.
- The LLM cannot override hard constraints without an explicit policy.

---

## Phase 5 — Generate tailored resumes

**Goal:** Create job-specific, ATS-safe documents.

### Rules

- Use only candidate-profile facts.
- Prefer the job's terminology when it truthfully matches the candidate's experience.
- Do not copy the job description into the resume.
- Do not add skills merely because the job asks for them.
- Keep bullets concise and outcome-oriented.
- Preserve chronological clarity.
- Use standard headings: Summary, Skills, Experience, Education, Certifications, Projects.
- Keep contact details consistent.
- Generate a resume version ID and a tailoring change log.

### Output package

```text
outputs/<application-id>/
├── resume.md
├── resume.docx
├── resume.pdf
├── cover-note.md
├── match-report.json
├── validation-report.json
└── change-log.md
```

### Exit criteria

- Documents render successfully.
- Resume contains no unsupported facts.
- Output is ready for human review.

---

## Phase 6 — Validation and ATS quality gate

**Goal:** Reject low-quality or unsafe outputs before review.

### Automated checks

- Required sections present.
- Name and contact details present.
- No placeholder tokens such as `{{name}}`.
- No unsupported tool or certification claims.
- No fabricated numbers.
- No excessive keyword repetition.
- Bullet length and readability within limits.
- PDF text extraction contains expected content.
- Links are valid or clearly marked.
- File names are safe and descriptive.
- PDF has no blank pages or overlapping text.

### Human review checklist

- Does the resume sound like the candidate?
- Are the most relevant achievements visible in the top half?
- Is the role level accurate?
- Are metrics defensible?
- Are missing requirements clearly disclosed?
- Is the design simple enough for ATS parsing?

### Exit criteria

- At least 90% of test resumes pass automated checks.
- No factuality-critical defect passes the gate.

---

## Phase 7 — Review queue and Naukri workflow

**Goal:** Put the user in control of final decisions.

### Review card contents

- Job title and company.
- Source URL.
- Match score and score breakdown.
- Top matched requirements.
- Missing or uncertain requirements.
- Resume preview/download links.
- Cover note.
- Suggested decision: Apply, Save, Reject, or Needs information.
- Disclosure of whether the job was sourced manually, by API, or by an approved collector.

### Naukri process

1. User opens the job URL.
2. User verifies the role is still active and legitimate.
3. User reviews the generated resume.
4. User submits manually through Naukri, or uses an officially supported submission integration.
5. User marks the application status in the tracker.
6. Automation stores the final submitted resume version and timestamp.

### Exit criteria

- No automatic submission occurs without an approved and policy-compliant integration.
- Every application has a resume version and source URL.

---

## Phase 8 — Tracking and analytics

**Goal:** Learn which resume versions and job types perform best.

### Track

- Jobs found.
- Jobs accepted/rejected.
- Applications submitted.
- Resume version used.
- Interview stage.
- Rejection/response reason when known.
- Time from discovery to submission.
- Match score versus outcome.
- Keyword patterns associated with callbacks.

### Reporting

Create a weekly report with:

- Number of relevant jobs discovered.
- Number of resumes generated.
- Number approved and submitted.
- Conversion to recruiter response/interview.
- Common missing skills.
- Resume changes recommended from outcomes.

Do not let the system automatically rewrite the master profile based on one rejection. Require evidence across multiple outcomes.

---

## Phase 9 — Deploy and schedule with GitHub

**Goal:** Run repeatable, observable workflows.

### GitHub Actions workflows

#### `test.yml`

Trigger on pull requests and pushes:

- Install dependencies.
- Run unit tests.
- Validate schemas.
- Render a sample resume.
- Run PDF text extraction checks.
- Upload test artifacts.

#### `scheduled-discovery.yml`

Run on a conservative schedule, such as once daily or a few times per week:

- Pull only approved sources.
- Ingest new jobs.
- Deduplicate.
- Score jobs.
- Generate review packages only above the threshold.
- Create a GitHub issue or send a private notification.
- Never submit applications.

#### Manual dispatch

Support `workflow_dispatch` with inputs:

- Source URL.
- Job ID.
- Candidate profile version.
- Minimum match score.
- Output format.

### Operational controls

- Set concurrency limits.
- Add timeouts.
- Cache dependencies carefully.
- Do not print secrets.
- Redact candidate PII in logs.
- Use artifact retention limits.
- Add production environment approval.
- Log source, version, and model metadata.

---

## 10. Prompt Library

The prompts below are starting templates. Put them in the repository, version them, and test them against fixed examples.

### 10.1 Parse a job description

```text
SYSTEM:
You are a structured job-description parser. Extract only information supported by the supplied job text. Do not infer missing details. Return valid JSON matching the requested schema.

USER:
Parse this job description for a candidate targeting Performance Marketing and Digital Marketing roles.

JOB TEXT:
{{job_text}}

Return:
{
  "title": "",
  "company": "",
  "location": "",
  "employment_type": "",
  "experience_min": null,
  "experience_max": null,
  "salary": "",
  "must_have_skills": [],
  "nice_to_have_skills": [],
  "platforms_and_tools": [],
  "responsibilities": [],
  "education_requirements": [],
  "certifications": [],
  "keywords": [],
  "uncertainties": []
}
```

### 10.2 Score job fit

```text
SYSTEM:
You are a conservative job-fit analyst. Score the job against the candidate profile using the supplied weights. Never treat an unverified skill as present. Return JSON only.

USER:
CANDIDATE PROFILE:
{{candidate_profile_json}}

PARSED JOB:
{{parsed_job_json}}

SCORING WEIGHTS:
{{scoring_config_json}}

Return:
{
  "overall_score": 0,
  "score_breakdown": {
    "title_role_fit": 0,
    "required_skills": 0,
    "platform_tools": 0,
    "experience_fit": 0,
    "industry_fit": 0,
    "location_fit": 0,
    "achievement_relevance": 0
  },
  "matched_requirements": [],
  "missing_requirements": [],
  "uncertain_requirements": [],
  "recommendation": "apply_review|save|reject",
  "reason": ""
}
```

### 10.3 Tailor the resume

```text
SYSTEM:
You tailor resumes for ATS compatibility while preserving factual accuracy. You may rewrite, reorder, condense, and emphasize facts from the candidate profile. You must not invent employers, dates, tools, certifications, responsibilities, metrics, budgets, or results. If a job requirement is missing, do not claim it; report it in the gap list.

USER:
TARGET JOB:
{{parsed_job_json}}

CANDIDATE PROFILE:
{{candidate_profile_json}}

MATCH REPORT:
{{match_report_json}}

Create:
1. A concise professional summary.
2. A targeted skills section using only supported skills.
3. Reordered experience bullets emphasizing relevant evidence.
4. A short gap list.
5. A change log explaining what was emphasized.

Return JSON:
{
  "summary": "",
  "skills": [],
  "experience": [],
  "education": [],
  "certifications": [],
  "gap_list": [],
  "change_log": [],
  "factuality_notes": []
}
```

### 10.4 Validate a tailored resume

```text
SYSTEM:
You are a strict resume quality auditor. Compare the tailored resume against the candidate fact registry and job description. Flag any unsupported claim, invented metric, misleading wording, keyword stuffing, missing section, or ATS risk. Return JSON only.

USER:
FACT REGISTRY:
{{fact_registry_json}}

JOB DESCRIPTION:
{{job_text}}

TAILORED RESUME:
{{resume_json}}

Return:
{
  "pass": true,
  "factuality_errors": [],
  "unsupported_claims": [],
  "ats_warnings": [],
  "keyword_stuffing_warnings": [],
  "missing_sections": [],
  "readability_warnings": [],
  "required_fixes": []
}
```

### 10.5 Generate a concise cover note

```text
SYSTEM:
Write a short professional cover note for a job application. Use only verified candidate facts. Do not exaggerate or claim experience not present. Keep it specific to the role and under 180 words.

USER:
CANDIDATE PROFILE:
{{candidate_profile_json}}

JOB:
{{parsed_job_json}}

MATCH REPORT:
{{match_report_json}}

Return:
{
  "subject": "",
  "body": "",
  "claims_used": []
}
```

### 10.6 Human review assistant

```text
You are the final review assistant. Do not submit anything. Show the user:
- the job title, company, and source URL;
- the match score and why it was calculated;
- the strongest matched requirements;
- missing or uncertain requirements;
- the generated resume version;
- any factuality or ATS warnings;
- a clear recommendation: Apply, Save, Reject, or Needs information.
Wait for an explicit user decision before changing the application status.
```

### 10.7 Google Antigravity build prompt

```text
Build a private, testable ATS resume-tailoring application for Performance Marketing and Digital Marketing roles.

Constraints:
- Use a human-in-the-loop workflow.
- Do not automate Naukri submission.
- Do not bypass CAPTCHA, login protections, rate limits, or access controls.
- Accept manual job URLs/text and approved feeds first.
- Use structured schemas for candidate profiles, jobs, match reports, resumes, and applications.
- Never invent candidate facts or metrics.
- Store secrets only in environment variables or the platform secret manager.
- Generate Markdown, DOCX, and PDF outputs.
- Include unit tests, sample fixtures, logging, and a clear README.
- Add a deterministic scoring layer before any LLM explanation.
- Add a factuality and ATS validation gate.
- Make source, timestamp, model version, and resume version auditable.

Start by proposing the repository structure, data schemas, API boundaries, and test plan. Do not write production code until these are reviewed.
```

---

## 11. Testing Strategy

### Unit tests

- Job-title normalization.
- Experience-range parsing.
- Skill synonym mapping.
- Duplicate detection.
- Match-score calculation.
- Resume versioning.
- Filename generation.
- Secret redaction.

### Fixture tests

Maintain anonymized sample jobs for:

- Performance Marketing Manager.
- Digital Marketing Manager.
- Growth Marketing Specialist.
- A clearly irrelevant role.
- A role with missing experience information.
- A role with conflicting requirements.

### Factuality tests

- Candidate has no Google Ads experience: generated resume must not claim Google Ads.
- Candidate has a metric without evidence: output must mark it uncertain.
- Job requires a certification not in profile: it must appear in the gap list.
- Job has an ambiguous title: system must flag uncertainty.

### Security tests

- Secrets do not appear in logs.
- Pull requests from untrusted forks cannot access production secrets.
- Private candidate data is not uploaded to public artifacts.
- Generated documents do not include hidden metadata that was not intended.

### Acceptance test

Given a fixed candidate profile and a fixed job description:

- The same inputs produce the same deterministic score.
- The output contains only verified facts.
- The resume renders to readable PDF.
- The review package includes source URL, score, warnings, and version.
- No external application is submitted.

---

## 12. Security, Privacy, and Compliance

- Keep the repository private.
- Use data minimization: store only information needed for the workflow.
- Encrypt the database and backups if hosted.
- Define retention, such as deleting raw job pages after 90 days while retaining application records.
- Restrict access to candidate PII.
- Redact contact details from logs where possible.
- Use least-privilege tokens.
- Rotate tokens if exposed.
- Never place credentials in prompts or Markdown files.
- Respect Naukri's terms, applicable privacy requirements, and the rights of job posters and candidates.
- Do not use the system to misrepresent qualifications.

---

## 13. Rollout Plan

### Release 0 — Manual MVP

- Candidate profile JSON.
- Paste job description or URL.
- Parse, score, tailor, validate.
- Generate Markdown and PDF.
- User manually applies on Naukri.

### Release 1 — GitHub automation

- Private GitHub repository.
- Pull-request tests.
- Manual workflow dispatch.
- Review package as GitHub artifact or private issue.
- SQLite tracking.

### Release 2 — Approved job feeds

- Add an authorized API/feed.
- Daily discovery workflow.
- Deduplication and notifications.
- Human approval queue.

### Release 3 — Browser-assisted review

- Open approved job URLs in a browser.
- Let the user inspect the job and take over when login is required.
- Do not bypass CAPTCHA or submit automatically.

### Release 4 — Analytics and optimization

- Track response and interview outcomes.
- Report common skill gaps.
- Improve templates and scoring based on multiple outcomes.

---

## 14. Definition of Done

The first production-ready version is complete when:

- A private GitHub repository exists with documented setup instructions.
- The candidate profile is structured and approved.
- At least five sample job fixtures pass ingestion and scoring tests.
- A job can be entered manually and produce a tailored resume package.
- The generated resume is available in Markdown, DOCX, and PDF.
- Automated validation blocks unsupported claims.
- Secrets are stored outside source control.
- Naukri submission remains manual unless an approved official integration is verified.
- Every output has a source URL, timestamp, match score, resume version, and validation report.
- The user can reproduce a result from the stored inputs.

---

## 15. Immediate Next Actions

1. Confirm the exact target titles and experience range.
2. Prepare the candidate master profile and fact registry.
3. Create a private GitHub repository.
4. Configure Google Antigravity with the repository and environment secrets.
5. Implement manual job-text input first.
6. Build parsing, deterministic scoring, tailoring, and validation.
7. Generate five sample resumes and manually review them.
8. Add GitHub Actions tests and manual dispatch.
9. Add Naukri URLs only as user-reviewed inputs.
10. Evaluate an official Naukri integration or approved feed before considering any collection automation.
11. Add scheduling only after the manual workflow is reliable.
12. Keep final application submission human-approved.

---

## 16. Important Assumptions to Validate

- “Google Antigravity” refers to the intended development/orchestration environment and supports the required runtime or can connect to GitHub.
- The user's Naukri account and intended workflow permit the planned level of access.
- The candidate can provide factual experience details and evidence for achievements.
- The LLM provider used by the environment has an appropriate privacy and data-retention posture.
- GitHub Actions is acceptable for processing resume data; if not, use a private runtime and store only sanitized artifacts in GitHub.

**Recommended starting point:** Build Release 0 and Release 1 first. They deliver the core value—high-quality, truthful, ATS-tailored resumes—without making the project dependent on uncertain scraping access or risky automated submissions.
