# Search Queries for Job Scraper

<!-- SETUP: Customize these queries based on your skills, target roles, and location -->

## Installed portal CLIs (primary for `/scrape`)

`/scrape` discovers every portal skill under `.agents/skills/*/SKILL.md` and runs its CLI first. Shipped country-agnostic CLIs include `linkedin-search` and `freehire-search`; Danish demos and any skill you add with `/add-portal` are included the same way. You do **not** need a matching `site:` line below for those CLIs to run.

The `site:` query templates in this file are the **WebSearch fallback** — for portals without a CLI, company career pages, or when a CLI fails.

**Language scope:** write every query category in every language listed in your CLAUDE.md Languages table (typically 1-2, sometimes more). A posting requiring a language you have *not* declared, as a job condition, is excluded before scoring; a posting requiring a *higher level* than you declared in a language you *do* work in is flagged for your own judgment, not excluded — see `04-job-evaluation.md`'s Language Gate, the single source of truth for this rule. Translate each category's keywords rather than machine-translating word-for-word (e.g. "Frontend Developer" -> "Desarrollador Frontend", not a literal word-for-word translation) if you work in more than one language.

## Search Sites

Primary (candidate market):
- **linkedin.com/jobs** - LinkedIn (India + remote; also abroad remote)
- **wellfound.com** - Wellfound / AngelList (VC-backed startups)
- **naukri.com** - Naukri (India full-time)
- **indeed.com** / **in.indeed.com** - Indeed
- **ycombinator.com/jobs** - Y Combinator Jobs
- **remoteok.com** - RemoteOK
- **aijobs** boards (AIJobs.net or similar niche AI listings)
- **himalayas.app** - Himalayas (remote)

Note: Built-in CLI scrapers target Danish portals (Jobindex, Jobbank, Jobdanmark, Jobnet). For Mukesh's search, prefer LinkedIn CLI + manual/portal search on the boards above. Scaffold India/startup boards with `/add-portal` when needed.

Secondary (company career pages via Google):
- Direct Google searches with `site:` filters for known VC-backed AI startups

## Query Categories

Queries are grouped by priority. Write **each category in every language from your Languages table** (see Language scope above). Combine with location terms: `Remote`, `India`, `Hyderabad`, or `Remote Worldwide` where the site supports it.

### Priority 1: AI / LLM / Agentic Engineering (primary direction)

These match the strongest and most desired career direction.

```
site:linkedin.com/jobs "AI Engineer" Remote OR India
site:linkedin.com/jobs "LLM Engineer" Remote OR India
site:linkedin.com/jobs "Agentic AI" OR "AI Agent" Engineer Remote OR India
site:wellfound.com "AI Engineer" OR "LLM Engineer"
site:ycombinator.com/jobs "AI Engineer" OR "Machine Learning"
"AI Engineer" LangGraph OR LangChain OR RAG site:naukri.com
```

### Priority 2: Production GenAI / Applied ML (domain expertise)

These match production LLM, RAG, agents, recommendations, and ML infrastructure.

```
site:linkedin.com/jobs "Generative AI Engineer" OR "Applied AI Engineer" India OR Remote
site:linkedin.com/jobs RAG OR "retrieval augmented" Python Remote OR India
site:linkedin.com/jobs "ML Engineer" PyTorch FastAPI Remote OR India
site:himalayas.app "AI Engineer" OR "Machine Learning Engineer"
site:remoteok.com "AI" OR "LLM" OR "Machine Learning"
```

### Priority 3: Platform / Forward-Deployed / Founding AI (adjacent growth)

Adjacent roles that leverage end-to-end ownership and startup fit.

```
site:linkedin.com/jobs "AI Platform Engineer" Remote OR India
site:linkedin.com/jobs "Forward Deployed" AI OR "Forward-Deployed Engineer"
site:wellfound.com "Founding AI" OR "Founding Engineer" AI
site:ycombinator.com/jobs "Founding Engineer" AI OR ML
site:linkedin.com/jobs "Solutions Engineer" AI OR "Solutions AI Engineer"
```

### Priority 4: Broader / GTM-technical (wider net)

```
site:linkedin.com/jobs "GTM Engineer" OR "AI Solutions Engineer" Remote OR India
site:indeed.com "AI Engineer" India OR Remote
site:naukri.com "Machine Learning Engineer" Python LLM
site:linkedin.com/jobs "MLOps" OR "Production ML" Python India OR Remote
```

## Key skill filters (combine with titles)

Prefer postings mentioning: LLMs, AI Agents, RAG, LangChain, LangGraph, Python, PyTorch, FastAPI, production ML

## Location Filter

When evaluating results, verify logistics match preferences:
- **Ideal:** Remote (India or abroad)
- **Acceptable:** Hybrid/onsite in Hyderabad or other Indian cities with relocate willingness
- **Borderline:** Onsite abroad with relocation package (discuss case-by-case)
- **Too far / fail:** Unpaid relocation abroad with no remote option and weak fit otherwise

## Language Filter

Your working languages and levels are in CLAUDE.md's Languages table. When filtering scraped results, apply `04-job-evaluation.md`'s Language Gate: a posting requiring a language you haven't declared at all is excluded; a posting requiring a higher level than you declared in a language you do work in is not excluded, flag it clearly instead (see `job-scraper/SKILL.md`'s Step 3 "Quick Fit Assessment" for how the flag surfaces in `/scrape` output). Postings simply *written* in a language you don't work in, that don't require it on the job, are fine.

## Date Filter

Only include jobs posted within the last 14 days, or with an application deadline that has not yet passed. If a posting date cannot be determined, include it but flag as "date unknown".

## Target company profile

Prioritize VC-backed startups building production AI products. Flag roles that are AI-theater (no shipping) or maintenance-only.

## Compensation baseline (for ranking notes, not hard filters)

- India: ~₹1 lakh/month
- Remote abroad: ~USD 1,500/month
Strong equity / mentorship / conversion-track internships may justify flexibility.

## Adapting Queries

If the user specifies a focus area, select queries from the matching category and also generate 2-3 custom queries for that focus. For example:
- "/scrape agents" -> Priority 1 agentic queries + LangGraph/CrewAI custom queries
- "/scrape founding" -> Priority 3 founding/YC queries
