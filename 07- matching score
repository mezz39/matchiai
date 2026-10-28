# Match Score System (Step 4)

The design basis for the job match score: how a CV is compared to one or many job postings and turned into a score the user can trust and act on.

**Why this document matters:** the match score is the product's moment of value, and the north-star metric (advancement, see [04-metrics.md](04-metrics.md)) is calculated from it. If the score is wrong or not trusted, every step built on it is wrong too ([03-defining-friction.md](03-defining-friction.md)).

## 1. Source and the flip

Source: Indeed Career Guide, [scoring system for resumes](https://ca.indeed.com/career-advice/career-development/scoring-system-for-resumes).

The article is written for **employers** who screen many resumes for one job. Matchlai is for **job seekers** who test one CV against one or many jobs. The mechanics carry over, but the purpose is reversed:

| | Employer screening (article) | Matchlai (seeker) |
|---|---|---|
| Goal | Pick candidates to interview | Show the user where they stand and what to fix |
| Scale | Many resumes, one job | One CV, one or many jobs |
| Output | Rank and shortlist | Score with an explanation and next actions |
| Who sets weights | The employer | The job posting, adjusted for the user's target |

### What we take from the article

| Idea in the article | How Matchlai uses it |
|---|---|
| Minimum vs preferred qualifications | **Must-have** and **nice-to-have** requirements (section 3) |
| Numerical method: rank items, give 1 to 5 points | **Weights** on every requirement (section 3) |
| Scorecard with qualifications across the top | **Score breakdown table** per job (section 7) |
| Same criteria for every candidate, to reduce bias | Same scoring rules for every user and every job; the formula is fixed and versioned |
| Scan for listed items instead of reading closely | Requirement checklist, extracted first, then matched |
| Common criteria: experience, education, certifications, technical skills, software, location, dates | **Requirement categories** (section 3) |

### What we deliberately fix

The article lists three weaknesses of resume scoring. Each maps to a design decision:

| Weakness in the article | Fix in Matchlai |
|---|---|
| **Overly rigid matching:** a skill described in different words gets no credit | Skill **normalizer** with synonyms and related skills; partial credit (section 4) |
| **False positives and negatives** | Evidence for every match, a confidence value, and calibration against real CVs (sections 5 and 9) |
| **Limited customization** | Weights adjustable per job and per user target (section 3) |

## 2. Pipeline

```
CV (parsed JSON)  ──┐
                    ├─► Normalizer ─► Requirement matching (LLM judgments) ─► Score engine (code) ─► Score report
Job posting(s)  ────┘        ▲                                                   ▲
 (link reader or pasted      │                                                   │
  text, or researched        Skill taxonomy                         Weights, formula version
  baseline for a role)
```

Where each step runs (see [06-agents-and-components.md](06-agents-and-components.md)):

| Step | Component | Runs in |
|---|---|---|
| 1. Get job text | Link reader, or pasted text fallback | n8n |
| 2. Extract requirements | LLM, fixed schema | n8n |
| 3. Normalize skills | Normalizer agent plus taxonomy | n8n and FastAPI |
| 4. Judge each requirement against the CV | Matcher agent (LLM) | n8n |
| 5. Validate the output | Validator | FastAPI |
| 6. Compute the score | Score engine | **FastAPI (code only)** |
| 7. Explain and format | Summarizer, formatter | n8n |

**Rule:** the LLM never calculates the score. It only gives a match level and evidence per requirement. Code computes the totals, so the same inputs always give the same score.

## 3. Requirements: classes, categories, weights

Every requirement extracted from a posting gets three attributes.

**Class** (from the article's minimum and preferred method)

- **Must-have:** essential. A missing must-have is shown as a gap and lowers the score sharply, but it does not zero it. The user still needs the full picture. *(The article discards the resume; the seeker product explains instead.)*
- **Nice-to-have:** raises the score but never blocks.

**Category** (from the article's common criteria)

| Category | Examples |
|---|---|
| Technical skills | Python, SQL, requirements elicitation |
| Tools and software | n8n, Jira, Figma |
| Experience | Years in role, type of work, seniority |
| Education | Degree, field |
| Certifications and licences | PMP, AWS |
| Domain knowledge | PropTech, finance |
| Soft skills | Communication, leadership |
| Constraints | Location, language, work authorization |

**Weight, 1 to 5** (the article's numerical method)

| Weight | Meaning |
|---|---|
| 5 | Core to the job; named in the title or repeated in the posting |
| 4 | Required and prominent |
| 3 | Required |
| 2 | Mentioned once, preferred |
| 1 | Minor or implied |

Default weights come from the posting: must-have status, how often it appears, and where it appears (title, "requirements" section, or "nice to have" section). They can be overridden per job.

## 4. Matching a requirement to the CV

For each requirement, the matcher returns a **match level** and **evidence**. Levels use partial credit so that different wording and adjacent skills are not scored as zero.

| Level | Credit | When |
|---|---|---|
| Exact | 1.0 | The CV states it clearly, with context (used in a project or role) |
| Strong | 0.8 | Clear equivalent or a stronger related skill (for example, "PostgreSQL" for "SQL") |
| Partial | 0.5 | Related or lower level (for example, "basic" use, or a similar tool) |
| Mentioned | 0.25 | Listed with no supporting context (skill list only) |
| Missing | 0 | No evidence |

Notes:

- "Mentioned" is separate from "Exact" on purpose. A skill listed without any project or role evidence is weaker proof, which also answers the honesty problem that step 5 (Q/A test) exists to check.
- Experience and education requirements use rules, not the LLM. For example, years of experience are computed from parsed dates, and the credit for 3 of 5 required years is `3 / 5 = 0.6`.
- Constraints (location, language, work authorization) are checked as pass or fail, and shown separately from the score. See section 6.

## 5. Evidence and confidence

Every requirement row must carry evidence, or the validator rejects it.

```json
{
  "requirement": "SQL",
  "class": "must",
  "category": "technical_skill",
  "weight": 4,
  "match_level": "strong",
  "credit": 0.8,
  "evidence": [
    {"cv_section": "experience[1]", "quote": "Designed PostgreSQL schemas for ..."}
  ],
  "confidence": 0.85
}
```

**Confidence** (0 to 1) is a diagnostic about the data, not about the user. It drops when:

- the CV parse was incomplete or low quality
- the posting text was short, vague, or pasted partially
- the requirement was extracted from a researched baseline, not an actual posting
- the evidence is a keyword hit with no context

The report shows an overall confidence next to the score. A score with low confidence is labelled as such, and the report says what would raise it (for example, "paste the full job text").

## 6. The score formula

Version the formula (`score_version`) so that scores from different versions are never compared silently.

```
category_score = Σ (weight × credit) / Σ weight          (inside each category)

base_score = Σ (weight × credit) / Σ weight × 100        (over all scored requirements)

must_have_coverage = (must-haves with credit ≥ 0.5) / (all must-haves)

final_score = base_score × must_have_penalty

must_have_penalty = 0.6 + 0.4 × must_have_coverage       (range 0.6 to 1.0)
```

Design choices:

- **Weighted average, not a plain count,** so one core skill matters more than several minor ones.
- **Must-have penalty** is a soft version of the article's "set aside resumes lacking minimum qualifications". A missing must-have pulls the score down by up to 40% but never makes it zero.
- **Constraints** (location, language, work authorization) are not in the score. They appear as separate **pass, fail, or unknown** flags next to it, because a number cannot honestly say "you are a 90 match but you cannot work in this country".
- **Unknown requirements** (those the matcher could not judge) are excluded from the calculation and lower the confidence value. They are never counted as missing.

**Bands** (labels only; the number is always shown with them):

| Score | Label | Suggested next action |
|---|---|---|
| 80 to 100 | Strong match | Apply; tailor wording |
| 60 to 79 | Good match | Apply after closing the top 1 to 2 gaps |
| 40 to 59 | Partial match | Close must-have gaps first (feeds step 3) |
| 0 to 39 | Weak match | Consider a nearer role or a learning plan (step 7) |

These cutoffs are a starting point. Calibrate them in section 9.

## 7. Score report

**Per job**

| Requirement | Class | Weight | Match | Credit | Evidence |
|---|---|---|---|---|---|
| SQL | Must | 4 | Strong | 0.8 | "Designed PostgreSQL schemas..." |
| Stakeholder interviews | Must | 5 | Missing | 0 | None found |
| Jira | Nice | 2 | Mentioned | 0.25 | Skills list only |

Plus: final score and band, category subscores, must-have coverage, constraints flags, overall confidence, and the **top 3 gaps ranked by score impact** (the weight of the requirement times its missing credit).

**Many jobs:** a comparison table, with one row per job and one column per category, sorted by final score. Show the requirements that are missing in most of the jobs, because those are the highest-value fixes across the whole job search.

Report formats (list, summary, visual) are produced by the formatter component. The visual is a bar per category plus a gap list.

## 8. Worked example

Job requirements (simplified):

| # | Requirement | Class | Weight | CV match | Credit | Weight × credit |
|---|---|---|---|---|---|---|
| 1 | Requirements elicitation | Must | 5 | Exact | 1.0 | 5.0 |
| 2 | SQL | Must | 4 | Strong | 0.8 | 3.2 |
| 3 | Stakeholder management | Must | 4 | Missing | 0 | 0 |
| 4 | Jira | Nice | 2 | Mentioned | 0.25 | 0.5 |
| 5 | PropTech domain | Nice | 3 | Partial | 0.5 | 1.5 |

- Σ weight = 5 + 4 + 4 + 2 + 3 = 18
- Σ (weight × credit) = 5.0 + 3.2 + 0 + 0.5 + 1.5 = 10.2
- base_score = 10.2 / 18 × 100 = **56.7**
- Must-haves with credit ≥ 0.5: 2 of 3, so coverage = 0.667
- must_have_penalty = 0.6 + 0.4 × 0.667 = 0.867
- final_score = 56.7 × 0.867 = **49.1**, band: partial match

Top gap: stakeholder management. Impact = 4 × 1.0 = 4 weight points, the largest single loss. If the user adds a relevant project and the credit becomes 0.8, the coverage becomes 1.0, the penalty becomes 1.0, and the final score rises to about **74** (13.4 / 18 × 100 = 74.4). This is the kind of statement the report should make: "adding evidence for stakeholder management would raise this match from 49 to about 74."

## 9. Validation and calibration

Because this score drives the north-star metric, it must be checked against reality, not just against itself.

1. **Gold set.** Collect 30 to 50 real CV and job pairs. Have a human (a recruiter or you) rate each pair as strong, partial, or weak. Compare the system's bands to the human labels. Track agreement and the false positive and false negative rates.
2. **Parser check.** On the same CVs, compare parsed fields to the real CV text. A wrong parse produces a wrong score, so measure parse accuracy separately.
3. **Stability.** Run the same CV and job through the matcher several times. The per-requirement levels should agree. If they do not, the score is noise. Lower the temperature, tighten the prompt, or use a rubric.
4. **Sensitivity.** Change one skill on the CV and check that the score moves in the expected direction and by a sensible amount.
5. **Outcome link.** When users report interviews or offers, compare them to scores (see the caveat in [04-metrics.md](04-metrics.md)). A good score should predict more interviews.
6. **Bias check.** Remove name, photo, age, and other personal details from what the matcher sees. The score must depend on skills, experience, and education only.

Record the evidence level of each finding as **observed**, **inferred**, or **assumed**, as in the friction method.

## 10. How the score feeds other steps

| Step | Uses |
|---|---|
| 1 Gap analysis | Requirements with credit below 1, ranked by impact |
| 3 What to improve | The same ranking, with effort estimates added |
| 5 Q/A test | "Mentioned" and "Partial" skills are the best candidates for verification questions |
| 6 Progress tracking | Baseline and current score; the formula and `score_version` must stay fixed between measurements |
| 7 Learning plan | Missing and partial must-haves become learning goals |

Goal progress % from [04-metrics.md](04-metrics.md): `(current − baseline) / (target − baseline) × 100`. This is only meaningful when baseline and current scores use the **same formula version and the same job or baseline**. If either changes, re-score the baseline before comparing.

## 11. FastAPI sketch

| Method and path | Purpose |
|---|---|
| `POST /v1/scores` | Body: `cv_id`, list of `job_id` or job text. Returns `score_id` and the full report. |
| `GET /v1/scores/{score_id}` | Returns a stored report |
| `GET /v1/users/{user_id}/scores?job_id=` | Score history for step 6 |
| `POST /v1/jobs/parse` | Body: link or pasted text. Returns structured requirements for review |

Stored per score: `cv_id`, `job_id`, `score_version`, requirement rows (with evidence and confidence), `final_score`, `confidence`, timestamp.

Let users **review the extracted requirements before scoring** (edit class and weight, remove wrong items). This costs one extra screen but removes the most common cause of a distrusted score: the system scoring requirements the user does not think the job has.

## 12. Open decisions

- Should weights be only auto-extracted, or can users edit them? (Recommended: editable, as in section 11.)
- Are the credit values (1.0, 0.8, 0.5, 0.25) and the must-have penalty (0.6 floor) acceptable? Calibrate them on the gold set before fixing them.
- Is the score for **one posting** or for a **role** (researched baseline)? Both use the same engine. A role-level score has lower confidence, because the baseline is not a real posting.
- Which constraints are hard filters, and which are only shown as flags?
- How do we score seniority: from years of experience, titles, or both?
- Which skill taxonomy should the normalizer use: build our own, or start from an open one such as ESCO or O*NET?
