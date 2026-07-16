# Search Queries for Job Scraper

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

Queries are grouped by priority. Combine with location terms: `Remote`, `India`, `Hyderabad`, or `Remote Worldwide` where the site supports it.

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
