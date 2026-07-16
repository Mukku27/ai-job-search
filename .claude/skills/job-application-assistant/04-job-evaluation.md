# Job Evaluation Framework

## Scoring Dimensions

Evaluate each job posting against these five dimensions:

### 1. Technical Skills Match (0-100)
How well do the required/preferred skills align with the candidate's capabilities?

| Score | Meaning |
|-------|---------|
| 80-100 | Core requirements are primary skills |
| 60-79 | Most requirements match, 1-2 gaps that are learnable |
| 40-59 | Partial match, significant upskilling needed |
| 0-39 | Fundamental mismatch |

**Strong match areas:** Python, LLMs, AI agents / multi-agent systems, RAG, LangChain / LangGraph, PyTorch, FastAPI, production ML / LLM deployment, recommendation & personalization, NLP
**Moderate match areas:** TensorFlow/Keras, LlamaIndex, Langfuse, CrewAI/AutoGen, computer vision, Docker, vector DBs (Pinecone), AWS/GCP, Streamlit dashboards, blockchain-adjacent AI (Bittensor, Arweave, Ethereum MCP)
**Weak match areas:** Heavy frontend product engineering, traditional backend-only roles without AI, pure research without deployment path, years of senior staff IC experience expectations, enterprise Java-only stacks

### 2. Experience Match (0-100)
Does work history align with what they're looking for?

| Score | Meaning |
|-------|---------|
| 80-100 | Direct experience in the same domain and role type |
| 60-79 | Related experience, transferable skills clear |
| 40-59 | Adjacent experience, would need to make the case |
| 0-39 | Unrelated experience |

**Strong:** Production LLM apps, agentic workflows, GenAI personalization/recommendations, fine-tuning and evaluation, FastAPI/Python ML services
**Moderate:** Decentralized AI / crypto-adjacent AI tooling, embedded/edge AI contest work, contract/internship-length shipping cycles
**Entry-level:** Roles requiring 5+ years industry tenure, people management, large-scale distributed systems ownership without mentorship

### 3. Behavioral/Culture Fit (0-100)
Does the role and company culture match the behavioral profile?

| Score | Meaning |
|-------|---------|
| 80-100 | Culture strongly matches behavioral preferences |
| 60-79 | Mixed signals but mostly compatible |
| 40-59 | Some friction areas |
| 0-39 | Significant culture mismatch |

**Red flags to research:** Department disorganization, work dominated by maintenance over development, poor chemistry with leadership, culture mismatches. Check reviews, media coverage, LinkedIn connections, and network contacts for insider perspective.

Strong culture signals: ownership, shipping production AI, mentorship, modern LLM/agent stacks, high-performing eng teams.
Friction signals: no product-building, rigid bureaucracy, prototype-only AI theater, zero mentorship.

### 4. Location & Logistics (Pass/Fail + Notes)
- Remote (India or abroad): PASS
- Relocate within India: PASS
- On-site only outside India requiring relocation abroad without remote option: FLAG (discuss; open to remote abroad)
- Requires unpaid notice / delayed start: usually PASS (candidate available immediately)

### 5. Career Alignment & Motivation (0-100)
Does this role advance career goals and contain tasks that energize?

| Score | Meaning |
|-------|---------|
| 80-100 | Strongly aligned with career direction, clear growth path |
| 60-79 | Good role but only partially aligned with long-term goals |
| 40-59 | Decent job but doesn't build toward career goals |
| 0-39 | Dead end or backwards step |

**Career goals:**
- Full-time AI/ML engineering with production LLM, agents, RAG, recommendations, or ML infrastructure
- Grow into founding / forward-deployed / platform AI roles at VC-backed startups
- Own features end-to-end and ship with strong engineering teams

**Motivation filter:** Evaluate not just whether you *can* do the tasks, but whether the tasks will *energize* you. Consider:
- Tasks that energize: production AI products, agents/RAG/LLMs, end-to-end ownership, fast shipping, modern stacks, mentorship
- Tasks that drain: maintenance-only work, no AI product building, outdated stacks, zero ownership
- Non-task factors: engineering culture, autonomy, learning path, startup pace

**Life situation alignment:** Consider personal constraints:
- **Security**: Baseline ~₹1 lakh/month India or ~USD 1,500/month remote abroad; flexible for exceptional equity/learning
- **Flexibility**: Remote preferred; open to relocate in India; available immediately
- **Professional development:** Mentorship and modern AI/LLM learning are must-haves

### 6. Salary Benchmark (Optional)

If the salary lookup tool is configured (`salary_data.json` exists), look up the company:
```
python salary_lookup.py "<Company Name>" --json
```

If a city is known from the posting, add `--city "<City>"` to narrow results.

Present findings as:
```
### Salary Benchmark
| Metric | Value |
|--------|-------|
| [Category] index | XX.X (+/-X.X% vs baseline) |
| Overall index | XX.X (+/-X.X% vs baseline) |
```

Interpret results relative to the baseline defined in the data file's metadata. For index-based data, higher typically means above-market compensation.

If the salary tool is not configured, skip this section. Candidate self-reported baseline: ~₹1L/month India; ~USD 1.5k/month remote abroad.

## Output Format

Present the evaluation as:

```
## Job Fit Evaluation: [Role] at [Company]

| Dimension | Score | Notes |
|-----------|-------|-------|
| Technical Skills | XX/100 | [brief note] |
| Experience Match | XX/100 | [brief note] |
| Behavioral Fit | XX/100 | [brief note] |
| Location | PASS/FAIL | [brief note] |
| Career Alignment | XX/100 | [brief note] |

**Overall Score: XX/100** (weighted average of scored dimensions)

### Verdict: [Strong Fit / Good Fit / Moderate Fit / Weak Fit / Poor Fit]

### Key Strengths for This Role
- [bullet points]

### Gaps to Address
- [bullet points]

### Recommendation
[1-2 sentences: apply/skip/apply with caveats]

### Company Research Checklist
- [ ] Checked company website (mission, values, recent news)
- [ ] Checked review sites (Glassdoor, Jobindex, etc.)
- [ ] Checked LinkedIn for team size, recent hires, connections
- [ ] Checked media for restructuring, growth, or workplace issues
- [ ] Identified network contacts who may know the team/manager
```

## Weighting
- Technical Skills: 30%
- Experience Match: 25%
- Behavioral Fit: 15%
- Career Alignment: 30%

(Location is pass/fail, not weighted)

## Thresholds
- **Strong Fit** (75+): Definitely apply, tailor everything
- **Good Fit** (60-74): Apply, address gaps in cover letter
- **Moderate Fit** (45-59): Consider carefully, discuss with user
- **Weak Fit** (30-44): Probably skip unless strategic reasons
- **Poor Fit** (<30): Skip

## Pre-Application: Call the Employer (Best Practice)

Before writing the application, consider whether the candidate should call the contact person listed in the posting. **Only call if there are substantive questions** - never call just to "be remembered."

### When to Suggest Calling
- The posting has unclear or ambiguous requirements
- It's unclear which competencies are essential vs. nice-to-have
- The role description is vague about day-to-day tasks
- There's a named contact person who invites questions

### Good Questions to Ask
- "What are the primary challenges in this role?"
- "How is time typically divided across the listed responsibilities?"
- "Which competencies are most critical for success in this position?"
- "What does success look like in the first 6-12 months?"

### Rules for the Call
- Prepare a 30-second "elevator pitch" about your background in case they ask
- The call's purpose is **gathering information**, not delivering a pitch
- Take notes - use what you learn to tailor the application
- Reference the conversation naturally in the cover letter ("After speaking with [name], I was especially drawn to...")
