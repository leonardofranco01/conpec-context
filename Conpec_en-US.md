# Conpec — General Context Document

> **Purpose:** Provide comprehensive context about Conpec for LLMs and AI agents assisting with document creation, strategy, scripts, presentations, and other tasks. This is the foundational context document — specialized documents (development, onboarding, knowledge management, etc.) will be derived from this base.

---

## 1. Company Overview

**Conpec** is a junior enterprise (Empresa Júnior / EJ) focused on computing, based at **Unicamp** (Universidade Estadual de Campinas), one of the top universities in Brazil and Latin America.

### Mission

Empower talent in a collaborative environment to transform Unicamp's excellence into unique computing solutions.

### Vision

Be the largest computing junior enterprise in Brazil, a reference in projects and culture.

### Values

| Value | Description |
|-------|-------------|
| **Excellence** | Pursuing high standards in everything we deliver |
| **Proactivity** | Taking initiative without waiting to be asked |
| **Nonconformity** | Challenging the status quo, always seeking improvement |
| **Curiosity** | Continuous desire to learn and explore |
| **Companionship** | Supporting each other as a team |
| **Determination** | Persisting through challenges to achieve goals |

### What Conpec Does

Conpec develops software solutions for external clients, including:

- **Websites** — institutional sites, internal management systems, automation tools
- **Mobile applications** — using React Native
- **Chatbots** — conversational AI solutions
- **Other software solutions** — tailored to client needs

---

## 2. Organizational Structure

Conpec currently has **53 members** organized into a hierarchical structure with three management tiers:

### Tiers

| Tier | Role | Responsibility |
|------|------|----------------|
| **Directorates** | Directors | Strategic management |
| **Management** | Managers (Gerentes) | Tactical management |
| **Operational** | Assessors & Trainees | Day-to-day execution |

### Directorates & Areas

#### Executive Presidency (Presidência Executiva)
- 1 Executive President
- Not considered an "area" — single-person directorate
- Currently also absorbing People & Management (G&G) strategic responsibilities due to vacancy

#### Market Area (Diretoria de Mercado)
- **Director:** 1
- **Marketing Management:** 1 manager, 2 assessors, 12 trainees
- **Commercial Management:** currently vacant
- **Total:** 16 members
- **Scope:** client acquisition, marketing campaigns, brand, commercial strategy

#### Projects Area (Diretoria de Projetos)
- **Director:** 1
- **Projects Management:** 2 managers, 15 assessors, 18 trainees
- **Total:** 36 members
- **Scope:** project execution, technical delivery, consulting, development

#### People & Management Area (Diretoria de Gente e Gestão — G&G)
- **Status:** directorate currently vacant
- Strategic scope absorbed by Executive Presidency
- Operational tasks distributed across the entire company
- **Scope (when active):** internal culture, member development, HR, organizational processes

---

## 3. Client Profile & Acquisition

### Client Profile

Conpec does not have a strictly defined ICP (Ideal Customer Profile). Clients are diverse:

- Startups
- Small and large companies
- Unicamp departments and academic units
- Various other organizations

### Client Acquisition

| Channel | Status |
|---------|--------|
| **Passive prospecting** | Active — Google Ads, social media campaigns, blog content |
| **Referrals** | Significant source of new clients |
| **Active prospecting** | Being structured within the Market area |

---

## 4. Project Lifecycle

A project at Conpec follows these stages:

```
Funnel Entry → Qualification → Diagnostic → Consulting → Prototyping → Development → Delivery → Maintenance
```

### Stage Details

| Stage | Owner | Description |
|-------|-------|-------------|
| **Funnel Entry** | Market | Lead enters the pipeline |
| **Qualification** | Market | Lead is evaluated for fit and viability |
| **Diagnostic** | Market + Projects | Deep dive into the lead's problem and needs (still a lead at this point) |
| **Consulting** | Projects | Begins after contract signing — requirements gathering, scope definition, technical analysis |
| **Prototyping** | Projects | UI/UX design and interactive prototyping in Figma |
| **Development** | Projects | Implementation using the tech stack, following Scrum methodology |
| **Delivery** | Projects | Final product handoff to the client |
| **Maintenance** | Projects | Post-delivery support and updates (if contracted) |

### Project Duration

| Complexity | Duration | Examples |
|------------|----------|----------|
| **Simple** | 2–4 months | Institutional websites, simple automations |
| **Complex** | 10–12 months | Internal management systems, unfamiliar functionalities |

---

## 5. Project Team Structure

Each project team (called a **squad**) consists of **4–6 members**:

| Role | Count | Responsibilities |
|------|-------|------------------|
| **Squad Leader** | 1 | Acts as PM, Scrum Master, and Product Owner. Manages the project, facilitates ceremonies, communicates with the client |
| **Developers / Designers** | 3–5 | All squad members (except the leader) handle both development and design — there is no strict separation between dev and design roles |

> **Note:** Squad members are generalists — every member works on both code and design. This is a deliberate choice to develop well-rounded professionals.

---

## 6. Methodology

Conpec uses **modified Agile frameworks** adapted to the junior enterprise context.

### Scrum (Adapted)

| Ceremony | Frequency | Duration |
|----------|-----------|----------|
| **Sprint Planning** | Once per sprint | 30–45 minutes |
| **Dailys** | 2–3 times per week | 15–30 minutes max |
| **Sprint Review** | Once per sprint | 30–45 minutes |
| **Sprint Retrospective** | Not practiced | Omitted due to time constraints |

- **Sprint length:** 2–3 weeks
- No retrospective — time constraint decision

### Kanban

- **Tool:** Notion (dedicated pages per project)
- **Board structure:** backlog management with task definition, assignees, priority levels
- Used alongside Scrum for task tracking and visualization

---

## 7. Technology Stack

### Core Stack

| Layer | Technology |
|-------|-----------|
| **Language** | TypeScript |
| **Frontend** | React.js, Next.js |
| **Mobile** | React Native |
| **Backend** | Node.js |
| **Database / BaaS** | Supabase, Firebase |

### Libraries & Tools

| Category | Tool |
|----------|------|
| **CSS Framework** | Tailwind CSS |
| **UI Components** | Shadcn UI |
| **Validation** | Zod |
| **Design & Prototyping** | Figma |
| **Deployment & Hosting** | Vercel |
| **Version Control** | Git (with Gitflow branching strategy) |
| **Commit Convention** | Conventional Commits |
| **Project Management** | Notion (Kanban boards) |

### Code Standards

- **Branching strategy:** Gitflow (main, develop, feature/*, hotfix/*, release/*)
- **Commit messages:** Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, etc.)
- **Automated testing:** Not yet implemented (planned for future adoption)
- **Project documentation:** No standardized format yet (planned for future adoption)

---

## 8. Tools Ecosystem

| Purpose | Tool |
|---------|------|
| Design & Prototyping | Figma |
| Project Management | Notion |
| Code Hosting | GitHub (assumed — Gitflow usage) |
| Deployment | Vercel |
| Backend Services | Supabase, Firebase |
| Marketing / Ads | Google Ads |
| Communication | Slack |
| Knowledge Management & File Storage | Google Drive (Google Workspace) |

---

## 9. Key Context for AI Agents

When assisting Conpec, agents should be aware of:

### Culture & Environment

- **Junior enterprise context** — members are university students balancing academics and EJ work
- **Learning-oriented** — members join to develop professional skills, not just deliver projects
- **Generalist philosophy** — developers also design; no strict role separation within squads
- **Collaborative culture** — companionship and team support are core values
- **High turnover** — as a student organization, members rotate frequently (typical EJ cycle: 1–2 years)

### Constraints

- **Time-limited members** — students have classes, exams, and other commitments
- **Variable experience levels** — team ranges from trainees (beginners) to experienced assessors
- **No dedicated QA** — testing is manual and done by developers themselves
- **No standardized documentation** — knowledge transfer relies on informal processes
- **G&G directorate vacant** — people management is distributed, not centralized

### Opportunities & Planned Improvements

- Implementing automated testing practices
- Standardizing project documentation
- Structuring active prospecting in the commercial area
- Building specialized context documents for: development workflows, member onboarding, knowledge management

### Communication Style

- Internal communication is in **Brazilian Portuguese**
- Technical documentation and code are in **English**
- Agents should adapt language based on the task: Portuguese for internal documents/presentations, English for code and technical artifacts

---

## 10. Glossary

| Term | Meaning |
|------|---------|
| **EJ (Empresa Júnior)** | Junior Enterprise — student-run company linked to a university |
| **Unicamp** | Universidade Estadual de Campinas |
| **Squad** | Project team (4–6 members) |
| **Squad Leader** | Project lead acting as PM/SM/PO |
| **Assessor** | Experienced operational member |
| **Trainee** | New member in training/probation period |
| **G&G (Gente e Gestão)** | People & Management area |
| **Daily** | Short standup meeting (adapted — not daily) |
| **Conventional Commits** | Standardized commit message format (feat, fix, chore, etc.) |
| **Gitflow** | Branching model with main, develop, feature, hotfix, release branches |
| **BaaS** | Backend as a Service (Supabase, Firebase) |
| **ICP** | Ideal Customer Profile |

---
