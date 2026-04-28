# Assignment: Leadership Portfolio Redesign
### DeepThought — Product Management Internship

---

## About DeepThought

DeepThought is a B2B company that helps Indian manufacturing MSMEs become better-run organizations. We do this through two things:

1. **Execution consulting** — we place operating Fellows inside client organizations who work alongside the founder's team to structure daily operations, build accountability systems, and diagnose execution gaps.

2. **PDGMS** — an AI-powered SaaS platform that structures daily work so that leadership gets diagnostic data as a byproduct. Not a project management tool. An operating system for the organization.

PDGMS tracks **what people deliver** and **how they grow** — contribution and capability — as physics-grade data. Every number is verified. No trusted self-claims.

---

## About This Role

This is a Product Management role at DeepThought. You will work on PDGMS — specifically on the employee-facing modules that people interact with every day. Your job will involve:

- Translating complex backend data into clear, meaningful UI experiences
- Making design decisions that balance information density with readability
- Understanding how different user personas (employees, supervisors, HR, mentors) experience the same product surface
- Turning a data-heavy system into something people actually want to open every day

This assignment tests exactly that. You are given a real product page with real data — and a specific design problem we haven't solved yet.

---

## The Assignment

**Redesign the Leadership Portfolio page so it feels like a pitch deck, not a dashboard.**

The Leadership Portfolio is a monthly career document that PDGMS generates for every employee. It shows what they contributed and how they grew. Right now, it looks like a dashboard — metric widgets arranged in a grid, all with equal visual weight, optimized for monitoring. We want it to feel like a **personal career pitch deck** — narrative-first, story-driven, something an employee would be proud to show a mentor.

You are given:
- This document — full context on the system, the data model, the design problem, and what we've already tried
- **`data-model.ts`** — the actual TypeScript types from our codebase (every field that exists in the backend)
- **`mock-portfolio-data.json`** — realistic sample data for one employee's one month (with wins, misses, and constraints)

Your job: design a version of the Leadership Portfolio page that transforms the data into a story.

---

## Important: Read This Before Starting

> **We use AI tools extensively at DeepThought — Claude, Gemini, GitHub Copilot, and others. We encourage you to use them too. But there is a difference between using AI as a thinking partner and copy-pasting AI output without applying your mind.**
>
> **Most candidates we receive paste a prompt into ChatGPT, copy the output, and submit it as their work. The result is generic, unverified, and indistinguishable from every other submission. These get rejected within 30 seconds.**
>
> **This is why Part B of this assignment requires a hand-drawn wireframe submitted via Internshala chat.** A hand-drawn wireframe forces you to think through layout, hierarchy, and information architecture yourself — things that copy-paste cannot fake. It is the single fastest way for our recruiters to separate candidates who actually thought about the problem from those who didn't.
>
> **If you skip the hand-drawn wireframe, or send a digitally generated one instead, your application will not be reviewed.** We mean this literally. The wireframe is how we filter 90% of applications before spending quality time on the remaining 10%.

---

## The Assignment Has Two Parts

| Part | What | Submit Where |
|------|------|-------------|
| **Part A** | Design output: high-fidelity mockup or working prototype + design rationale document | Upload to GitHub or Google Drive. Share the link on Internshala. |
| **Part B** | Design thinking: hand-drawn wireframe + written explanation of your key design decisions | Send directly in the Internshala chat window. |

### Part A: The Design (GitHub / Google Drive)

Produce one of the following (pick whichever plays to your strength):
- **High-fidelity mockup** in Figma, Sketch, or any design tool — export as images or share a Figma link
- **Working HTML/CSS prototype** — static is fine, interactivity is a bonus but not required
- **Annotated wireframes** — lower fidelity is acceptable if the annotations explain every design decision clearly

Along with the design, include a **design rationale document** (1-2 pages) that explains:
- What design decisions you made and why
- Which of the 5 design challenges (Part 10) you chose to address, and your position on each
- What you would do differently with more time
- Any assumptions you made about user behavior or data

### Part B: Design Thinking (Internshala Chat)

Send directly in the Internshala chat window:

1. **A hand-drawn wireframe** — photo of a hand-drawn layout sketch showing the page structure, information hierarchy, and how the 6 sections are organized. It doesn't need to be beautiful. It needs to show that you thought through the spatial relationships yourself.

2. **3 key design decisions** — in text, explain the 3 most important choices you made and the tradeoff behind each one. Two sentences per decision is enough.

> **The hand-drawn wireframe is mandatory. Applications without it will not be reviewed.**

---

## Evaluation Criteria

1. **Information hierarchy** — Does the design have a clear signal at the top? Can you tell "how did I do?" in 3 seconds? Or is it metric soup?
2. **Story over data** — Does the narrative lead, with data as supporting evidence? Or is it a chart page with text footnotes?
3. **Design judgment** — Did you make clear tradeoffs (what to show, what to collapse, what to remove)? Or did you try to show everything at once?
4. **Data fidelity** — Does every element in your design map to a real field in `data-model.ts`? Or did you invent data the backend doesn't have?
5. **Portability** — Would this make sense to someone outside the organization? Or is it full of internal jargon?
6. **Constraint section handling** — How did you present "what blocked me" without making it feel like a complaint form?

---

## Logistics

- **Time:** You have 5 days from receiving this assignment.
- **Tools:** Use whatever design tools you want. AI is fine for ideation — but the hand-drawn wireframe and your rationale must be your own thinking.
- **Questions:** If something is unclear, make a reasonable assumption and state it.

### Submission Checklist

| # | Item | Where |
|---|------|-------|
| 1 | High-fidelity mockup or prototype | GitHub / Google Drive → link on Internshala |
| 2 | Design rationale document (1-2 pages) | Same repo/folder |
| 3 | **Hand-drawn wireframe** | **Photo in Internshala chat window** |
| 4 | 3 key design decisions (text) | Internshala chat window |

---

# DETAILED CONTEXT

Everything below is reference material for your design. Read it all before starting — the design constraints, experiments, and data model are essential.

---

## Part 1: The System — PDGMS, Aavahana, and the 5×5 Grid

### What is PDGMS?

**Performance Delivery & Growth Management System.** It is the platform where organizational work happens — from daily execution to annual strategy. It is structured on a **5×5 grid**: 5 horizontal layers (H1–H5: Goals → Strategy → Programs → Management → Execution) × 5 vertical authority levels (V1–V5: Operator → Manager → Program Lead → Strategic Director → Mandate Setter).

PDGMS is not a generic project management tool. It tracks **what people deliver** and **how they grow** — contribution and capability — as physics-grade data. Every number is verified. No trusted self-claims.

### What is Aavahana?

**Aavahana** ("Invocation" in Sanskrit) is the L1/L2 user-facing surface of PDGMS. It is where employees (V1 operators) and supervisors (V2 managers) do their daily work:

- **Plan** daily/weekly/monthly work
- **Report** what was done (daily report, weekly report, monthly report)
- **Track** career progression (Career Runrate)
- **Build** intellectual property (IP commits)
- **Claim** framework and process usage
- **Participate** in organizational rituals (auto-tracked)

Aavahana is the everyday loop. The employee opens it every day.

### What is Leadership Portfolio?

The Leadership Portfolio is a **monthly career artifact**. It is NOT a dashboard — dashboards are for monitoring. The portfolio is a **document the employee owns**. It is:

- **Generated** automatically when the employee submits their monthly report
- **Tagged** with a permanent UID that follows the person even after they leave the organization
- **Composed** of 6 fixed sections, each backed by physics data from the monthly report
- **Narrative-enhanced** — each section has a data summary AND a written narrative paragraph

The portfolio answers two questions:
1. **What did I contribute this month?** (delivery, IP, KPIs)
2. **How did I grow this month?** (frameworks, processes, rituals, career trajectory)

It is the one asset an employee keeps after leaving employment.

---

## Part 2: The 6 Axes — What Gets Measured

PDGMS tracks employee performance across **6 axes** split into two categories. These are the building blocks of everything in the system — reports, career scoring, and the Leadership Portfolio.

### 3 Contribution Axes (What You Produce)

#### 1. Deliverables

**What:** Discrete units of work shipped. Tracked via Event Loop tickets (similar to Jira tickets but within PDGMS).

**Lifecycle:** `in_progress` → `pending_qa` → `delivered`

**Key rule:** Only `delivered` (QA passed) counts for career scoring. Pending QA does not count.

**Who creates them:** TPM (L3) creates tickets and assigns them. Employee works on them. Status changes are auto-tracked.

**Employee does NOT manually report deliverables** — they flow from ticket state changes in the Event Loop.

**Example:** "Designed curriculum mapping matrix" — 1 deliverable, status: delivered.

#### 2. IP (Intellectual Property)

**What:** Original knowledge contributions. Uses a git-commit model — employee "commits" work tagged to an IP item.

**Cadence:** Weekly

**Two types:**
- **Program IP** — contributions to the program's shared IP. TPM approves.
- **MyZone IP** — personal IP (blog posts, tools, research). HR approves for career scoring.

**Format:** Number + Label — "4 × Blog Posts", "2 × Assessment Framework commits"

**Key rule:** L4 (Strategic Director) can withdraw IP claims if they don't meet quality standards.

**Example:** "Assessment Framework v2 — rubric engine" — 1 IP commit, verified.

#### 3. KPIs (Key Performance Indicators)

**What:** Shared program metrics. Multiple team members commit against the SAME KPI each month.

**Cadence:** Monthly

**How it works:**
1. Program defines shared KPIs (e.g., "Schools Visited", "Training Sessions Conducted")
2. Each KPI has a target (e.g., Target: 20 schools)
3. Employee enters their actual number monthly (e.g., Actual: 12 schools)
4. Employee marks status: hit / miss / partial / pending
5. **TPM verifies** with official before/after numbers — employee self-report is not final

**Format:** Number + Label + Unit — "12 × Schools Visited (schools)"

**Example:**
| KPI | Target | Actual | Status |
|-----|--------|--------|--------|
| Schools Visited | 20 | 12 | miss |
| Training Sessions | 8 | 8 | hit |

### 3 Capability Axes (How You Grow)

#### 4. Frameworks

**What:** Employee claims they applied a recognized framework during work.

**Cadence:** Daily (claimed in daily report)

**How it works:** Employee says "I applied First Principles thinking 3 times today" — this is a **claim** that requires **L2/TPM verification**. Until verified, it's pending.

**Format:** Number + Label — "3 × First Principles applied"

**Verification states:** `pending` → `verified` / `rejected`

#### 5. Processes

**What:** Same mechanic as Frameworks, but for organizational processes.

**Cadence:** Daily (claimed in daily report)

**Example:** "2 × Git Workflow followed", "1 × Sprint Planning process used"

**Same verification requirement** as frameworks.

#### 6. Rituals

**What:** Participation in organizational rituals. **Auto-tracked** — the employee does NOT claim these manually.

**Examples of rituals:**
- Daily plan created on time
- Standup (SD) attended
- Weekly report submitted
- LDI (Learning & Development Initiative) completed

**Feeds into UBS** (Unicorn Behavior Score) — an engagement score computed from ritual participation.

### Universal Rule

ALL 6 axes use **Number + Label** format. ALL require verification. No trusted self-claims. This is what makes the data "physics-grade" — every number has a source and a verifier.

---

## Part 3: Career Ladder & Scoring

### V-Levels (Authority Levels)

| Level | Role | Scope |
|-------|------|-------|
| V1 | Operator / Doer | Executes work |
| V2 | Senior / Manager | Breaks down and reviews work |
| V3 | Program Lead | Designs systems across teams |
| V4 | Strategic Director | Makes strategy to meet goals |
| V5 | Mandate Setter | Sets goals, reviews strategy |

These are NOT job titles — they are capability and authority scopes. An individual can operate at multiple V-layers.

### Career Runrate

The Career Runrate is a scoring system that tracks an employee's progression through career levels. It has 3 views:

1. **Planned** — AI suggests a 20-year trajectory at onboarding, supervisor edits
2. **Actual** — computed from the 6 axes (weighted composite score, 0–100)
3. **Projected** — regression analysis vs level eligibility dates

**Pace status:** `ahead` | `on_track` | `behind`

**Pure physics. No AI in scoring.** The composite score is a deterministic weighted calculation from the 6 axis scores.

### Composite Score

Each axis has a score (0–100). The composite score is a weighted average. Each career level (V1–V5) has minimum thresholds per axis. If an employee's scores exceed the thresholds for the next level, they're projected to be eligible.

---

## Part 4: The Data Available for Leadership Portfolio

The portfolio is assembled server-side from data the system already has. **You cannot invent data fields that don't exist.** This is the constraint.

### The Full Data Model

The complete TypeScript type definitions from our codebase are provided in a separate file: **`data-model.ts`**. This is the actual type system the frontend uses — not a simplified summary.

Key types you'll be working with:

| Type | What it contains | Relevance to portfolio |
|------|-----------------|----------------------|
| `LeadershipPortfolio` | The portfolio itself — 6 sections, UID, generation timestamp | The top-level object your page renders |
| `LeadershipPortfolioSection` | One section — id, title, dataPoints (key-value), narrative | Each of the 6 portfolio sections |
| `AxisScores` | 6 axis scores (0-100): deliverables, ip, kpis, frameworks, processes, rituals | Section 3 (Capability Growth) — the core measurement |
| `CareerRunrateData` | Planned trajectory, actual snapshots, projected trajectory, current/next level | Section 6 (Career Trajectory) — the career arc |
| `CareerRunrateSnapshotEnriched` | Snapshot + delta + direction (up/down/flat) + level change flag | History of score changes over time |
| `AxisGapItem` | Per-axis gap to next level: current score, required score, gap, cleared | What's blocking promotion eligibility |
| `TrajectoryDelta` | Months ahead/behind plan, direction | The "pace" signal |
| `RunwayNode` | Timeline node: achieved/current/projected/future, position %, confidence band | Visual career runway |
| `MonthlyReport` | Full monthly data: axis summaries, weekly breakdown, happiness, constraints | The source data that generates the portfolio |
| `MonthlyAxisSummary` | Deliverables, IP, KPIs, frameworks, processes, rituals — all monthly | Sections 2-4 of the portfolio |
| `MonthlyConstraintSummary` | Raised/resolved/open, resolution rate, avg days, recurring patterns, longest open | Section 5 (Constraint Patterns) |
| `DeliverableSummary` | Total, completed, in-progress, items with ticket IDs and statuses | Section 2 (Contribution Highlights) |
| `IPSummary` | Total commits, target commits, item-level detail | Section 2 (Contribution Highlights) |
| `MonthlyKPICommit` | Per-KPI: target value, actual value, unit, hit/miss/partial/pending | Section 4 (KPI Impact) |
| `UBSScore` / `UBSTrend` | Engagement score (weighted events), weekly trend | Feeds into ritual score |
| `Ticket` | Full ticket with lifecycle, state transitions, estimates vs actuals, QA criteria | Detail behind deliverables and constraints |

Read `data-model.ts` for the complete field-level definitions. Every field in the mock data file maps to a type in that file.

### Portfolio Response Shape (from backend)

```
{
  portfolioId: string
  uid: string                      // permanent — follows the person
  monthId: string                  // "YYYY-MM"
  orgId: string
  period: { from: string, to: string }
  generatedAt: string              // ISO datetime
  sections: [                      // always 6 sections
    { type: string, data: {...}, narrative: string | null }
  ]
}
```

### Section 1: Executive Summary

**Source:** Career snapshot (latest weekly snapshot)

```
{
  compositeScore: number       // weighted 6-axis composite, 0–100
  weekId: string               // e.g. "2026-W16"
  paceStatus: string           // 'ahead' | 'on_track' | 'behind'
  currentLevel: string         // e.g. "V3"
}
```

### Section 2: Contribution Highlights

**Source:** Verified IP commits (top 10) + Monthly report delivery summary

```
{
  ipCommitCount: number
  recentCommits: [
    { id: number, description: string, createdAt: string }
  ]
  totalDeliverables: number
  completed: number            // only QA-passed deliverables
  inProgress: number
  pendingQa: number
  notStarted: number
}
```

### Section 3: Capability Growth

**Source:** Career snapshot — 6-axis scores

```
{
  deliveryRate: number         // 0–100, delivered / total assigned
  ipScore: number              // 0–100, IP axis
  frameworkScore: number       // 0–100, framework mastery
  processScore: number         // 0–100, process adoption
  ritualScore: number          // 0–100, ritual participation
}
```

### Section 4: KPI Impact

**Source:** KPI commits + TPM verification records

```
{
  totalKpiCommits: number      // all KPI commits for the month
  verifiedCount: number        // commits where TPM verified met=true
}
```

### Section 5: Constraint Patterns

**Source:** Event Loop tickets of type "constraint" (top 20)

```
{
  totalConstraints: number
  resolved: number             // status = resolved or confirmed
  byType: {
    budget: number
    talent: number
    internal_support: number
    assumptions: number
    permissions: number
    unspecified: number
  }
}
```

**What are constraints?** Blockers that prevent work from getting done. Employees raise constraint tickets when they're stuck — "HOD resigned" (talent), "Budget not approved for math manipulatives" (budget). The system tracks how many get raised vs resolved.

### Section 6: Career Trajectory

**Source:** Career plan + Career snapshot projections

```
{
  ladderId: string                     // which career ladder
  milestones: [                        // 20-year trajectory
    { level: string, targetDate: string }
  ]
  projectedNextLevel: string | null    // e.g. "V3"
  projectedDate: string | null         // estimated eligibility date
}
```

### Additional Data Available (from Monthly Report, not portfolio endpoint)

These fields exist on the monthly report and could enrich the portfolio in future:

```
// Deliverable detail — individual items
[{ ticketId, label, ticketType, status, estimatedMinutes, actualMinutes }]

// Capability detail — individual claims
{ frameworks: [{ labelId, quantity, description }],
  processes: [{ labelId, quantity, description }],
  rituals: [...] }

// Time distribution — how hours were spent
{ operational, research, deepWork, coordination, meetings, planning }  // hours

// KPI detail — individual monthly actuals
[{ kpiId, personalContribution, quantity }]
```

### What Does NOT Exist

| Data | Status |
|------|--------|
| Employee name / photo / role in portfolio response | Not included — comes from user session/auth |
| Regex/template narrative engine | Not built yet — narratives are currently AI-generated or null |
| Monthly constraint pattern aggregation (recurring types) | Not built — only raw ticket counts |
| "Why" behind career pace (gap drivers, blocker reasons) | Not stored |
| Per-claim framework/process verification detail | No dedicated table |
| Timesheet data in portfolio | Exists on report but not fed into portfolio |

---

## Part 5: Design Constraints

### Hard Constraints (must follow)

1. **Read-only surface.** The portfolio has no inputs, no forms, no "Generate" button. It is assembled on monthly report submission. The page is a mirror — you look, you don't touch.

2. **Data must map to backend fields.** Every number shown must come from the data model in Part 4. You cannot show data the backend doesn't compute.

3. **No LLM on the frontend.** Narratives are generated server-side (regex/template decision trees, not AI). The frontend just renders what the backend returns.

4. **UID is sacred.** The portfolio UID must be visible. It is the permanent identifier that follows the employee.

5. **Month-scoped.** One portfolio per month. Navigation is prev/next month, not arbitrary date ranges.

6. **No jargon in the UI.** "IP Commits" → "Original Work" or "Knowledge Contributions". "QA Pending" → "Awaiting Review". "Rituals" → "Engagement". The audience is the employee and potentially their mentor — not a developer.

### Soft Constraints (strong preferences)

7. **Career artifact, not dashboard.** It should feel like a document you're proud of, not an analytics widget you're anxious about. Think portfolio you'd show a mentor, not a Jira board.

8. **Narrative is prominent.** The written paragraph is the throughput — not an afterthought below a data table. Story first, evidence second.

9. **Progressive disclosure.** The employee has 30 seconds between meetings. The top of the page answers "how did I do?" in one scan. Detail is available but not forced.

10. **Design tokens only.** If building in our stack: `bg-base-100`, `bg-base-200`, `border-base-300`, `text-base-content` with opacity modifiers. No hardcoded hex colors. DaisyUI components. If building outside our stack: use your own design system, just maintain the same visual hierarchy principles.

---

## Part 6: What We Learned from Experiments 1 & 2

We built two static prototype pages to test layout approaches. Both used mock data matching the backend shape.

### Experiment 1: Dashboard Style

**Layout:** Signal headline (metric concatenation) → two equal cards (Contributions + Capability) → 3 collapsed evidence sections → 4 placeholder cards.

**What worked:**
- 3-layer progressive disclosure (signal → stories → evidence)
- Capability bars with threshold marker at 60
- Collapsed evidence sections
- Full-width reading column (not 2-col grid for main content)

**What didn't:**
- No employee identity — anonymous, could be anyone's report
- Signal headline was metric soup: "42 of 54 delivered · 14 IP commits · 4/5 axes above threshold"
- Narratives were stat dumps, not stories
- Jargon everywhere: "IP Commits", "QA Pending", "axis scores"
- All cards had equal visual weight — no hierarchy
- Felt like Jira, not a career portfolio

### Experiment 2: Pitch Deck Style

**Layout:** Identity bar (name, role, program) → signal sentence ("Strong delivery month — 78% completion. Growing in 4 of 5 areas.") → delivery story card → growth story card → 2 collapsed evidence sections → 4 placeholder cards.

**What changed:**
- Employee name + role at top — "this is MY document"
- Signal is a human sentence with an adjective (Strong/Solid/Mixed), not metric concatenation
- Narratives rewritten as stories, not stat readouts
- 2 large stat cards (Delivered + Original Work) instead of 4 small ones
- Jargon replaced: "Original Work", "Knowledge", "Engagement", "Awaiting Review"
- 2 evidence collapses instead of 3 (merged deliverable breakdown + IP log)

**Open questions from both experiments:**
1. Should the signal be a sentence or something more visual (a score ring, a grade, a color)?
2. Is narrative-first (story above data) the right order, or do some people want numbers first?
3. Are 6 sections too many for a monthly portfolio? Should some be collapsed or removed?
4. Does the threshold line on capability bars (at 60) communicate clearly?
5. Should the portfolio feel more like a resume section, a report card, or a pitch slide?
6. Where does the employee photo/avatar belong (if at all)?

---

## Part 7: The 6 Portfolio Sections — What Story Each Tells

| Section | Question it answers | Data it uses | Tone |
|---------|-------------------|-------------|------|
| Executive Summary | "How am I doing overall?" | Composite score, pace status, current level | Confidence or urgency |
| Contribution Highlights | "What did I ship?" | Deliverables completed, IP commits, top items | Pride in output |
| Capability Growth | "How am I developing?" | 5 axis scores vs threshold | Growth awareness |
| KPI Impact | "Did I hit my targets?" | KPI actuals vs targets, verification rate | Accountability |
| Constraint Patterns | "What blocked me?" | Constraints raised, resolved, recurring types | Honest about obstacles |
| Career Trajectory | "Where am I headed?" | Career plan milestones, projected level, pace | Forward momentum |

**For this design exercise:** Sections 2 (Contribution Highlights) and 3 (Capability Growth) have the richest data. Start there. The other 4 have thinner data — they can be simpler cards or placeholders.

---

## Part 8: Actors and Audience

**Who creates the portfolio?** The system — automatically, when the employee submits their monthly report. No manual generation step.

**Who reads the portfolio?**
- **The employee themselves** — primary audience. Monthly self-reflection.
- **The employee's supervisor (V2)** — review conversations, feedback.
- **A mentor or career advisor** — the portfolio is the "show your work" document.
- **HR** — career progression decisions.
- **The employee's future self** — years later, looking back at what they built.

**Who does NOT read the portfolio?**
- External clients or customers
- The general public
- Other employees at the same level (it's personal, not competitive)

The tone should be: **professional, personal, dignified**. Not corporate-stiff, not casual-slack. Think annual review document that you're actually proud to read.

---

## Part 9: The Core Design Problem — Dashboard to Pitch Deck

This is the single most important section. Everything above is context. This is the problem.

**The Leadership Portfolio currently looks like a dashboard. We want it to feel like a pitch deck.**

Not a literal pitch deck with slides. But the same transformation in *mindset*:

| Dashboard mindset (current) | Pitch deck mindset (target) |
|---|---|
| Metric widgets arranged in a grid | Narrative blocks with visual hierarchy |
| All cards have equal weight | One clear signal dominates, detail is secondary |
| "What are my numbers?" | "What's my story this month?" |
| Monitoring tool you check anxiously | Career artifact you'd show a mentor or future employer |
| Data-first — story is a footnote below the chart | Story-first — data is the evidence that supports the story |
| Anonymous — could be anyone's report | Personal — your name, your identity, your document |
| Optimized for scanning many metrics | Optimized for reading one narrative with proof |
| Feels like Jira or Google Analytics | Feels like a deck you'd present to someone you respect |

**What "pitch deck for a career month" means concretely:**

When a founder walks into a VC meeting, they don't open a dashboard. They present a deck that says: here's what we did, here's why it matters, here's where we're going, here's what's in the way. The data is there — but it serves the story, not the other way around.

The Leadership Portfolio should do the same for an employee's month:
- **Here's what I shipped** (contribution, with specific proof)
- **Here's how I grew** (capability scores, with context)
- **Here's what blocked me** (constraints, honestly named)
- **Here's where I'm headed** (career trajectory, with momentum)

The employee should be able to hand this to a mentor, a new manager, or their future self — and it should tell a coherent story in 60 seconds of reading.

**The enemy is metric soup.** When every number has equal visual weight, nothing communicates. The portfolio should have one unmistakable signal at the top ("Strong delivery month — growing in 4 of 5 areas") and everything else supports or explains that signal.

---

## Part 10: Design Challenges — Pick Your Battles

These are the open problems we haven't solved. You don't need to solve all of them — but your design should take a clear position on at least 3.

### Challenge 1: The Signal (most important)

What does the employee see in the first 3 seconds? Options explored so far:
- A sentence: "Strong delivery month — 78% completion. Growing in 4 of 5 areas."
- A composite score ring (72/100)
- A grade (A- / B+ / C)
- A color-coded status bar
- Something else entirely

The sentence worked better than the score ring in Experiment 2 — but it still felt like a stat readout, not a story. How do you make the first impression feel *personal and meaningful* rather than clinical?

### Challenge 2: Story vs Data Ordering

Should each section lead with the narrative paragraph and follow with data? Or lead with data and follow with interpretation? Experiment 2 tried narrative-first. It felt better for the employee reading about themselves — but felt odd for the supervisor reviewing the portfolio (supervisors wanted numbers first).

Can one layout serve both audiences? Or does the portfolio need two modes?

### Challenge 3: Six Sections — Too Many?

Six sections is a lot for a monthly document. Are all six equally important? Should some be:
- Always visible (contribution, capability, career trajectory)
- Collapsed by default (constraints, KPI detail)
- Merged (executive summary + career trajectory = one section?)
- Removed entirely for months where data is thin

How do you maintain the 6-axis completeness of the data model without overwhelming the reader?

### Challenge 4: The Constraint Section — Honesty Without Anxiety

Constraints (blockers, things that went wrong) are the hardest section to design. If you make them too prominent, the portfolio feels like a complaint document. If you hide them, you lose the diagnostic value — constraints are how the organization learns.

How do you present "here's what blocked me" in a way that feels like honest professional communication, not whining?

### Challenge 5: Portability and Identity

The portfolio follows the employee via a permanent UID — even after they leave the organization. This means:
- It needs to make sense outside the organization's context
- It should feel like *the employee's document*, not the company's report on the employee
- The identity (name, role, program, month) needs to be prominent enough that it reads as a personal artifact

How do you balance organizational context (the 5x5 grid, V-levels, program names) with portability (readable by someone who's never heard of PDGMS)?

---

## Part 11: Sample Data & Data Model

Two reference files are provided:

1. **`data-model.ts`** — The actual TypeScript type definitions from our codebase. Every interface, enum, and field that exists in the backend. This is the source of truth for what data is available. You don't need to write code against it — it's here so you know exactly what you can render.

2. **`mock-portfolio-data.json`** — A complete mock portfolio for one employee (Rajan Iyer, V2 Production Planning Lead at a pharma company, March 2026). This is realistic data with a mix of wins and misses — not a perfect-score showcase. Your design must render this data.

Use this data — don't invent your own numbers. The constraint is intentional: the design must work with the data the system actually produces.