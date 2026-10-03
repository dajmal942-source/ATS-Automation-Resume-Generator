# ATS Resume Automation Engine

> **Target Roles:** Performance Marketing, Digital Marketing, Growth Marketing (Senior / 1–4 years experience)  
> **Core Operating Principle:** Human-in-the-loop workflow. Automated discovery, scoring, tailoring, validation, and review packaging — final submission on Naukri remains human-controlled. Strict factuality: zero fabricated claims, zero unverified metrics.

---

## 🚀 Quickstart (Works Out-Of-The-Box)

The system is configured to run **immediately in high-fidelity offline deterministic mode**. You do not need API keys to start testing, scoring jobs, or generating ATS-safe resumes.

### 1. Environment Setup
```bash
# Clone or open the workspace
cd C:\Users\user\Documents\ATS-Resume-Automation

# Create virtual environment and install dependencies
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Check Credentials & Configuration Status
```bash
python -m src.cli check-env
```

### 3. Run Sample Marketing Jobs
```bash
# Run single sample (Performance Marketing Manager)
python -m src.cli run-sample 01-perf-mktg-manager

# Run all 6 test fixtures (includes high-fit, low-fit, and conflicting jobs)
python -m src.cli run-sample all
```

### 4. Interactive Review Queue
```bash
# View applications ready for review
python -m src.cli review

# Record your decision (e.g. after manually applying on Naukri)
python -m src.cli decide <application-id> apply --notes "Submitted on Naukri"
```

### 5. View Pipeline Analytics & Skill Gaps
```bash
python -m src.cli report
```

---

## 🔑 How to Add API Keys & Credentials Later

When you are ready to connect live AI models (Gemini, OpenAI, Claude) or job platform tokens, simply edit your `.env` file:

```bash
# Copy example if not already done
copy .env.example .env
```

Open `.env` in any text editor and fill in your keys:

```dotenv
# Choose your preferred provider: "gemini", "openai", "anthropic", or leave as "offline"
LLM_PROVIDER=gemini

# Google Gemini (Recommended for Google Antigravity)
GEMINI_API_KEY=AIzaSy...
GEMINI_MODEL=gemini-1.5-pro

# OpenAI (Alternative)
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o

# Anthropic (Alternative)
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-3-5-sonnet-20241022

# Naukri Integration (Official tokens if authorized for your account)
NAUKRI_API_TOKEN=
NAUKRI_CLIENT_ID=

# Apify Web Scraping (Optional)
APIFY_API_TOKEN=

# Notification Webhooks (Slack, Telegram, or SMTP)
NOTIFICATION_WEBHOOK_URL=
```

Once pasted, running `python -m src.cli check-env` will confirm that live LLM mode is active.

---

## 🛠️ CLI Command Reference

| Command | Description | Example |
|---|---|---|
| `check-env` | Verifies configured credentials and system paths | `python -m src.cli check-env` |
| `run-sample` | Runs pipeline against fixture jobs in `data/sample-jobs/` | `python -m src.cli run-sample 01-perf-mktg-manager` |
| `ingest --file` | Ingests job from JSON, Markdown, TXT, or CSV file | `python -m src.cli ingest --file data/sample-jobs/02-digital-mktg-lead.json` |
| `ingest --url` | Fetches and processes public job posting URL | `python -m src.cli ingest --url "https://..."` |
| `ingest --text` | Ingests pasted job description text | `python -m src.cli ingest --text "..." --title "..." --company "..."` |
| `review` | Displays the human approval queue | `python -m src.cli review` |
| `decide` | Records review decision (`apply`, `applied`, `save`, `reject`) | `python -m src.cli decide app-1eacd9c5 apply` |
| `report` | Generates summary report with common missing skill gaps | `python -m src.cli report` |

---

## 📦 Output Review Package Structure

For every job meeting the match threshold (default: `>= 75/100`), the engine generates an isolated review package in `outputs/<application-id>/`:

```text
outputs/app-1eacd9c5-202610031815/
├── resume.md              # ATS-compliant Markdown resume
├── resume.docx            # Professional single-column ATS Word document
├── resume.pdf             # Searchable ATS-optimized PDF (clean typography)
├── cover-note.md          # Tailored concise cover letter (< 180 words)
├── match-report.json      # Transparent score breakdown and matched skills
├── validation-report.json # ATS quality gate & factuality verification audit
└── change-log.md          # Audit log of tailored bullet points and disclosed gaps
```

---

## 🎯 Match Scoring Engine

Scoring is completely transparent and deterministic before any LLM explanation:

| Factor | Weight | Evaluation Method |
|---|---:|---|
| **Target Title & Family** | **20%** | Exact or token overlap with candidate's target roles (Performance, Digital, Growth Marketing) |
| **Required Skills Match** | **25%** | Cross-matched against candidate skills & tools corpus with synonym mapping (SEM, PPC, PMax, etc.) |
| **Platforms & Tools Match** | **15%** | Specific alignment with Google Ads, Meta Ads, GA4, GTM, Looker Studio, etc. |
| **Experience Range Fit** | **15%** | Evaluates minimum/maximum years required vs candidate's factual tenure (2.5 years) |
| **Industry Relevance** | **10%** | D2C, E-commerce, B2B SaaS, and Growth Tech focus |
| **Location & Work Mode** | **5%** | Bengaluru, Mumbai, Delhi NCR, Remote, or Hybrid preferences |
| **Achievement Relevance** | **10%** | Quantifiable business outcomes (CAC reduction, ROAS scaling, server-side tracking) |

### Rejection & Negative Constraints
If a job strictly requires skills listed in the candidate's `negative_constraints` (e.g. Salesforce Marketing Cloud, Marketo, DV360, or 5+ years experience), the system flags the conflict and automatically assigns a `reject` recommendation.

---

## 🛡️ ATS Quality Gate & Factuality Checks

Every generated document must pass automated validation:
1. **Fact Registry Audit:** Verifies that every employer, title, metric, and skill traces to `data/candidate-profile.json`.
2. **Negative Constraint Enforcement:** Prevents the resume from claiming unverified or prohibited tools.
3. **Template Token Check:** Ensures no raw `{{placeholder}}` strings remain.
4. **Keyword Stuffing Detector:** Flags repetitive word patterns.
5. **Layout & Standard Headings:** Ensures standard headings (*Professional Summary*, *Core Competencies & Skills*, *Professional Experience*, *Education & Certifications*) with single-column layouts and zero floating text boxes.

---

## 🧪 Testing

Run the full pytest suite (16 comprehensive unit & end-to-end acceptance tests):

```bash
pytest -v
```

---

## 🔄 CI/CD Automation (GitHub Actions)

- **`test.yml`:** Runs automated schema checks, unit tests, and sample artifact generation on every commit and PR.
- **`scheduled-discovery.yml`:** Scheduled daily discovery pipeline that ingests jobs, scores them, and generates review packages.
- **`manual-tailor.yml`:** Allows manual triggering via GitHub Actions workflow dispatch with a URL or pasted text.
