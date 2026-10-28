# Shared Agents and Components

Stack: **n8n** (workflows and orchestration), **FastAPI** (API layer), **AI-generated frontend** (one page per workflow).

## Design rules

- **Deep researcher:** built once as a shared sub-workflow, not one copy per workflow. Results are cached so the same job title is not researched again for every user.
- **CV parser:** outputs a fixed schema so every downstream workflow reads the CV the same way.
- **Summarizer:** one generic component with modes, instead of one prompt that does everything.
- **Scoring and progress math** run in code (FastAPI), not in the LLM. The LLM gives per-requirement judgments; code computes the totals.

## Core agents

| Agent | Purpose | Input | Output | Used by steps |
|---|---|---|---|---|
| **Deep researcher** | Finds job requirements for a role from several sources | Job title, level, region (optional) | Sourced list of required skills, tools, level and responsibilities, with source links | 2, 4, 7 |
| **CV parser** | Extracts structured data from the uploaded CV | CV file (PDF/DOCX) | JSON: personal info, skills, experience, projects, education, certifications, links | All (via the foundation) |
| **Summarizer** | Condenses gathered information for a given purpose | Raw text or JSON plus a mode | Short summary in the requested mode | 1, 2, 3, 6 |

**Summarizer modes:** `requirements` (merge many research results into one baseline), `cv_overview`, `gap_report` (list, summary, visual data), `progress_update`.

## Added agents

| Agent | Purpose | Used by steps |
|---|---|---|
| **Requirements normalizer** | Maps skills from the CV, job postings and research onto one skill taxonomy, so "JS", "JavaScript" and "ES6" count as the same skill | 1, 2, 4, 6 |
| **Link reader / enricher** | Fetches job posting text and reads GitHub projects linked in the CV (languages, README, activity) as evidence of skills | 2, 4, 5 |
| **Matcher / scorer** | Compares the parsed CV to the requirements baseline and produces a score with a breakdown per requirement | 1, 4 |
| **Improvement prioritizer** | Ranks CV additions and fixes by impact and effort | 3 |
| **Question generator** | Creates Q/A questions from the CV's claims, with an answer key | 5 |
| **Answer evaluator** | Grades the user's answers honestly against the CV claims and returns a verified-skill level per skill | 5, 6 |
| **Progress calculator** | Turns skill updates and Q/A results into a progress state against the goal | 6 |
| **Curriculum builder** | Builds an ordered learning plan from gaps, target and progress | 7 |
| **Validator** | Checks every agent's output against its schema and flags empty or low-confidence results before they reach the next step | All |

## Non-agent components

- **Cache/store:** keeps research results, parsed CVs and shared outputs (target career profile, gap report, progress state). Can be a database behind FastAPI.
- **Formatter:** turns gap and progress data into list, summary and chart-ready JSON for the frontend.

## Where the logic lives

- **n8n:** agents that call LLMs or search, orchestration, sub-workflow calls.
- **FastAPI:** the parser's file handling, schema validation, the cache, and deterministic parts such as score calculation and progress math.

## n8n nodes for reading links

| Link type | Node |
|---|---|
| GitHub repos, issues, files | GitHub node |
| GitHub Projects (v2 boards) | GraphQL node, or HTTP Request node with a GitHub token |
| Any web page (job postings, portfolios) | HTTP Request + HTML node |
| JavaScript-heavy pages | Scraping service (Firecrawl, Jina Reader, Browserless) via HTTP Request |
| Agent-driven reading | AI Agent node with HTTP Request Tool or GitHub Tool attached |

LinkedIn blocks scraping. For step 4, offer a paste-the-job-text fallback.

## Reuse map

| Scope | Components |
|---|---|
| Foundation | CV parser, validator, cache |
| Step 2 (target career) | Deep researcher, link reader, normalizer, summarizer |
| Step 4 (job match score) | Link reader, normalizer, matcher |
| Step 5 (Q/A test) | Question generator, answer evaluator |
| Steps 1, 3, 6, 7 | Stored outputs of earlier steps, plus their own agents (matcher, prioritizer, progress calculator, curriculum builder) |
