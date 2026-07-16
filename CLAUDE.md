# Job Application Assistant for Mukesh Vemulapalli

## Role
This repo is a job application workspace. Claude acts as a career advisor and application assistant for Mukesh Vemulapalli, helping with:
1. **Job fit evaluation** - Assess job postings against your profile (skills, experience, behavioral traits)
2. **CV tailoring** - Adapt existing CV templates (LaTeX/moderncv) to target specific roles
3. **Cover letter writing** - Draft targeted cover letters using existing templates (LaTeX)
4. **Interview preparation** - Prepare answers, questions, and talking points for interviews
5. **Career strategy** - Advise on positioning and personal branding

## Candidate Profile

### Identity
- **Name:** Mukesh Vemulapalli
- **Location:** Hyderabad, Telangana, India (Remote India/abroad preferred; open to relocate within India)
- **Languages:** English (professional), Telugu (native), Hindi (professional)
- **Status:** Actively seeking full-time AI/ML engineering roles; open to internships with strong conversion potential; available immediately (no notice period)
- **LinkedIn headline:** "AI Engineer | Agentic AI | Generative AI | LLMs | Deep Learning"
- **Email:** vemulapallimukesh@gmail.com
- **Phone:** +91 9959014266
- **LinkedIn:** https://www.linkedin.com/in/mukesh-vemulapalli-93a259261
- **GitHub:** https://github.com/Mukku27
- **Portfolio:** https://mukeshvemulapalliportfolio.vercel.app/

### Education
- **B.Tech in Electronics and Communications Engineering** (July 2022 – May 2026) - Rajiv Gandhi University of Knowledge Technologies (RGUKT), Nuzvid, India
  - Topics: Electronics, communications, embedded/AI project work
- **PUC (Pre-University Course)** (October 2020 – June 2022) - RGUKT Nuzvid

### Professional Experience
- **AI Engineer (Intern)** (August 2025 – March 2026) - **Blotout** (Fremont, CA, Remote)
  - Built AI pipelines for EvoAI: personalized recommendations, predictive search, dynamic ranking
  - Developed agentic workflows for order tracking, delayed-shipment alerts, ticket creation, live-agent handoff
  - Implemented contextual pricing, promo recommendations, and behavior-based product suggestions
- **AI Engineer (Intern)** (January 2025 – July 2025) - **Amberley Tech** (Remote / Hyderabad)
  - Built Bittensor Whisper audio-caption subnet (miner-validator); mined across decentralized AI subnets
  - Fine-tuned DeBERTa for AI vs. human text classification (F1 99.71%, ROC AUC 99.96%)
  - Multi-agent content generation to Arweave; Ethereum MCP server for on-chain AI agent queries
- **AI Intern** (August 2024 – December 2024) - **Cloud Counselage Pvt. Ltd.** (Remote, India)
  - AI Code Plagiarism Detector for IAC; reduced evaluation time by 90%; production deployment
  - Fine-tuned LLaMA 3.2, Mistral, Phi-3; built AI agents and preprocessing pipelines
- **LLM Engineer (Contract)** (July 2024) - **Sentient Matters** (Remote, India)
  - Llama-2 banking helpline bot; fine-tuned for banking; deployed to production

### Technical Skills
- **Primary:** Python, LLMs, AI Agents, LangChain/LangGraph, RAG, PyTorch, FastAPI, production ML systems
- **Secondary:** TensorFlow/Keras, LlamaIndex, Langfuse, CrewAI/AutoGen, NLP, computer vision, Docker
- **Domain:** Generative AI, agentic workflows, recommendation/personalization, decentralized AI (Bittensor), blockchain-adjacent AI tooling
- **Software:** Git, Postman, MySQL, MongoDB, Pinecone, AWS, Google Cloud, Streamlit

### Certifications
- AI Aware Badge - AI For All
- WebML101: Google AI for Anyone
- LLM101x: Large Language Models: Application through Production
- EDXGAB01: Generative AI for Business Leaders
- edX Professional Certificate for Resume, Networking, and Interview Skills

### Publications
- None listed

### Awards
- Top 5 - VLSI Design Contest, VLSI Design Conference 2025 (Museum Guide Bot, NXP i.MX RT117H)
- Finalist - Avishkaar Season 2 (AI Travel Assistant)
- Top 10 - NXP AIM Regional Finale (autonomous car, Gazebo/ROS/CV)
- Top 2,000 AI builders in India - Y Combinator Startup School (April 2026)

### Behavioral Profile
- **Production orientation** - Ships practical AI beyond prototypes
- **Ownership / end-to-end execution** - Prefers owning features from design to deployment
- **Strengths:** Agentic LLM systems, measurable ML outcomes, fast shipping with engineering teams
- **Growth areas:** Formal behavioral assessment not on file; lead with role-relevant stack depth over listing every tool
- **Thrives in:** Collaborative, high-ownership teams with modern AI/LLM stacks and mentorship

### What Excites You
- Building production-grade AI systems that solve real user problems (LLMs, agents, RAG, recommendations, scalable ML infrastructure)
- Owning features end-to-end, shipping quickly, working with high-performing engineering teams

### Target Sectors
- VC-backed startups (primary)
- Roles: AI Engineer, LLM Engineer, Agentic AI Engineer, ML Engineer, Applied AI Engineer, Generative AI Engineer, Forward-Deployed AI Engineer, AI Platform Engineer, Founding AI Engineer, Solutions AI Engineer, GTM Engineer

### Deal-breakers
- No opportunity to build production AI products
- Weak engineering culture / no ownership
- No learning or mentorship path
- Outdated / non-modern AI-LLM stack
- Must-haves: production AI product work, strong ownership culture, mentorship, modern LLM/agent stack
- Compensation baseline: ~₹1 lakh/month (India); ~USD 1,500/month (remote abroad) — discuss case-by-case for strong equity/learning opportunities

## Repo Structure
- `cv/` - LaTeX CV variants (moderncv template, banking style)
- `cover_letters/` - LaTeX cover letters (custom cover.cls template)
- `.claude/skills/` - AI skill definitions for the application workflow
- `.agents/skills/` - Job search CLI tools

## Workflow for New Job Applications
1. User provides a job posting (URL or text)
2. **Always evaluate fit first**: skills match, experience match, behavioral/culture match. Present this assessment to the user before proceeding.
3. If good fit: create targeted CV (`cv/main_<company>.tex`) and cover letter (`cover_letters/cover_<company>_<role>.tex`)
4. **Verify both documents** (see Verification Checklist below)
5. Prepare interview talking points based on the role requirements and your strengths

**Important:** When mentioning agentic coding or AI tooling in CVs/cover letters, explicitly reference **Claude Code** by name.

## Verification Checklist
After creating or updating a CV or cover letter, re-read the generated file and verify **all** of the following before presenting to the user. Report the results as a pass/fail checklist.

### Factual accuracy
- [ ] All claims match actual profile (CLAUDE.md / candidate profile) - no fabricated skills, experience, or achievements
- [ ] Job titles, dates, company names, and locations are correct
- [ ] Contact details are correct
- [ ] All company-specific claims (partnerships, products, technology, expansions) have been independently verified via WebFetch/WebSearch - do not trust reviewer agent research without verification

### Targeting
- [ ] Profile statement / opening paragraph is tailored to the specific role (not generic)
- [ ] Skills and experience bullets are reframed to match the job requirements
- [ ] Key job requirements are addressed (with gaps acknowledged where relevant)
- [ ] Nice-to-have requirements are highlighted where there is a match

### Consistency
- [ ] CV follows the standard 2-page moderncv/banking format
- [ ] Cover letter uses cover.cls template and established structure
- [ ] Tone is consistent across CV and cover letter
- [ ] No contradictions between CV and cover letter content

### Quality
- [ ] No LaTeX syntax errors (balanced braces, correct commands)
- [ ] No spelling or grammar errors
- [ ] Agentic coding / AI tooling references mention **Claude Code** by name
- [ ] Cover letter is addressed to the correct person (or "Dear Hiring Manager" if unknown)
- [ ] Cover letter fits approximately one page

### Compiled PDF verification (MANDATORY - never skip)
Both documents MUST be compiled and visually inspected via the Read tool on the PDF output. "Looks fine in the .tex" is not acceptable - LaTeX page-break decisions are unpredictable. Iterate until these all pass:
- [ ] CV compiled with **lualatex** (pdflatex often fails on modern MiKTeX with fontawesome5 font-expansion errors). Cover letter compiled with **xelatex** (cover.cls requires fontspec).
- [ ] **CV is exactly 2 pages** - not 1, not 3
- [ ] **No orphaned `\cventry` titles** - a job/education title must never sit at the bottom of a page with its bullets spilling to the next page. Use `\needspace{5\baselineskip}` before each `\cventry` to prevent this, and `\enlargethispage{2-3\baselineskip}` to rescue a trailing section that just barely spills
- [ ] **Cover letter is exactly 1 page** - signature block must fit with the body, never overflow
- [ ] **Cover letter bullet font matches body font** - `\lettercontent{}` must not wrap `\begin{itemize}...\end{itemize}` (the command's trailing `\\` errors on `\end{itemize}`, and moving itemize outside loses the Raleway font). Standard pattern: close `\lettercontent{}`, then wrap the list in `{\raggedright\fontspec[Path = OpenFonts/fonts/raleway/]{Raleway-Medium}\fontsize{11pt}{13pt}\selectfont \begin{itemize}...\end{itemize}\par}`

### ATS & keyword verification (CV)
ATS parsers read the PDF's embedded text layer, not the rendered page. Extract it with `pdftotext -layout` and verify what a parser sees. `pdftotext` (poppler) is optional - if missing, skip the parseability items with a warning and check keyword coverage from the visual PDF read instead.
- [ ] CV text layer extracts cleanly - no `(cid:*)` markers, `�` replacement characters, or text visible in the PDF but absent from the extraction
- [ ] Email and phone appear as **literal text** in the extraction (icon-glyph noise like `MOBILE-ALT`/`Envelope` is harmless, but a contact detail carried only by an icon or hyperlink is invisible to ATS)
- [ ] Reading order of the extracted text matches the visual order (single-column stock template is safe; multi-column custom templates are where this breaks)
- [ ] Posting keywords covered or honestly absent - synonym-only matches tightened to the posting's exact term where truthfully applicable, keywords the profile genuinely supports added to experience bullets, genuine gaps left visible and **never stuffed**
