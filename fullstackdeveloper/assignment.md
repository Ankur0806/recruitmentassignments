# Assignment: Growth Charter Renderer — PDGMS CRM Module
### DeepThought — Full-Stack Developer Internship

---

## Your Files

| # | File | What it is | When to read |
|---|------|-----------|-------------|
| 1 | **assignment.md** (this file) | The assignment brief — what to build, how to submit, how you'll be graded | Read first |
| 2 | **context.md** | Domain knowledge — the CRM node system, Growth Charter structure, rendering logic, regex patterns, LLM prompt | Read fully before writing any code |
| 3 | **seed-data.json** | Sample company with pre-filled node data — use this to seed your MongoDB and test your renderer | Load into your database |
| 4 | **render-map.json** | The value-to-text mapping for all 10 nodes — use this in your rendering logic | Load into your app |
| 5 | **sample-transcript.json** | A founder conversation transcript — use this for Part B (optional) | Read when you start Part B |

---

## About DeepThought

DeepThought is a B2B company that helps Indian manufacturing MSMEs become better-run organizations. We do this through two things:

1. **Execution consulting** — we place operating Fellows inside client organizations who work alongside the founder's team to structure daily operations, build accountability systems, and diagnose execution gaps.

2. **PDGMS** — an AI-powered SaaS platform that structures daily work so that leadership gets diagnostic data as a byproduct.

One of PDGMS's core modules is the **CRM** — a buyer-side funnel where every stage is built around classification nodes. As a company moves through the funnel, nodes are scored, companion data is captured, and documents like the **Growth Charter** are auto-generated from this data.

---

## About This Role

Every software developer at DeepThought is a **Full-Stack Builder**. That means:

- **Product thinking** — you read the domain, understand the why, and scope the MVP without someone handing you a spec
- **Frontend** — you render complex business documents from structured data
- **Backend** — you model data in MongoDB, build APIs, and wire the data flow
- **Business acumen** — you understand what you're building and why it matters to the user
- **AI integration** — you connect LLM APIs to production workflows (Part B)
- **Self-management** — you break the work into commits, track your own progress, and ship incrementally

This assignment tests all six. You are given a real CRM system, real node definitions, and a real document generation flow. Build a working app.

---

## The Problem

### What is the Growth Charter?

The Growth Charter is the key document in PDGMS's CRM funnel. It's a personalized narrative document that mirrors a company founder's growth ambition, execution gaps, and infrastructure reality back to them — written from their own words and data.

The Growth Charter is **not written manually**. It's **auto-generated from CRM nodes** — structured classification data scored during conversations with the founder. Each node classifies a specific dimension (what KPI the founder cares about, what broke in prior attempts, who should own the improvement, etc.). The charter template has placeholders that get replaced with the text that matches each node's current value.

### What You're Building

A fullstack web application that:

1. **Stores CRM node data in MongoDB** — the schema defines accounts and their scored nodes
2. **Renders a Growth Charter in the browser** — using a markdown template, a regex pattern, and a render map that converts node values into readable narrative text
3. **(Optional, Part B)** Extracts node values from a founder transcript by running an LLM call against Gemini

**The regex pattern, the charter template, the render map, and the MongoDB schema are all given to you.** Your job is to read the domain context, understand the data model, and wire everything together into a working app.

---

## Important: Read This Before Starting

> **We use AI tools extensively at DeepThought — Claude, Gemini, GitHub Copilot, and others. We encourage you to use them too for coding assistance. But there is a difference between using AI to help you write code and submitting a project you don't understand.**
>
> **This assignment is fundamentally a comprehension and execution test.** The technical pieces are straightforward — MongoDB, a template renderer, a REST API. The challenge is reading a dense business document (`context.md`), understanding what the data means, and wiring the right data into the right places.
>
> **If your Growth Charter renders but the content doesn't make sense — wrong text in wrong sections, placeholders unfilled, business logic misunderstood — that tells us you didn't read the context. The AI can write your code, but it can't read the domain for you.**

---

## Part A: Growth Charter Renderer (Required)

Build a web app that renders a Growth Charter from CRM node data stored in MongoDB.

### What the app does:

1. **Seed page or API route** — load `seed-data.json` into MongoDB (one account + its nodes)
2. **Charter page** — select an account, fetch its nodes from MongoDB, and render the Growth Charter

### How rendering works:

1. Fetch the account and all its nodes from MongoDB
2. Build a **placeholder map** — a flat key-value object where each key matches a `{{placeholder}}` in the charter template, and the value comes from either:
   - The account record (e.g., `companyName`)
   - A node's render text (looked up from `render-map.json` using the node's scored value)
   - A node's companion field (e.g., the founder's name from K1, the revenue from F2)
3. Apply the **regex** on the charter template to replace all `{{placeholders}}` with their values
4. Render the resulting markdown as HTML in the browser

**Everything you need — the template, the regex, the render map, the placeholder-to-data mapping — is documented in `context.md`.** Read it.

### The Founder's Voice section:

The charter includes a quote table ("What [Founder] Said / What It Means") built from **verbatim fields** on nodes. This section is dynamic — it renders one row per node that has a verbatim quote. Your app needs to iterate over nodes with verbatim data and build this table. This is the one section that can't be a simple regex replacement — you'll handle it in code.

### Technical Requirements

| Requirement | Detail |
|------------|--------|
| **Database** | MongoDB (local or Atlas free tier). Two collections: `accounts` and `nodes`. |
| **Backend** | Any language/framework — Node/Express, Python/FastAPI, your choice. Exposes API to fetch account + nodes. |
| **Frontend** | Any framework — React, Vue, Svelte, vanilla JS. Renders the Growth Charter in the browser. |
| **Rendering** | Use the regex pattern and render map provided. The charter must be generated from data, not hardcoded. |
| **No deployment required** | Everything runs locally. `README.md` must include setup instructions. |
| **Git history** | Incremental commits that show your thinking. One giant commit = your application will not be reviewed. |

### What Part A must produce:

When we run your app and select the seeded account (Sureflow Formulations), we should see a Growth Charter that looks like the **Sample Rendered Charter** in `context.md`. The content should match — correct founder name, correct revenue, correct KPI text, correct execution gap descriptions, correct quotes.

---

## Part B: Transcript Node Extraction (Optional — Bonus)

Given a founder conversation transcript, use an LLM to extract CRM node values automatically.

### What you add to the app:

1. **Transcript input** — a text area where the user pastes a founder transcript
2. **"Extract Nodes" button** — sends the transcript + the provided LLM prompt to Gemini's API
3. **Review output** — displays the extracted node values so the user can review before saving
4. **Save to MongoDB** — creates a new account with the extracted nodes

### What's given:

- The LLM prompt template is in `context.md` — it's complete and ready to use
- A sample transcript is in `sample-transcript.json` — use it to test
- The prompt tells the LLM exactly what to extract and in what format (JSON)

### What you figure out:

- **Getting a Gemini API key.** Google AI Studio offers free API keys. We're not going to walk you through the signup. If you can't figure out how to get an API key for a free service, that's a signal.
- **Making the API call.** Gemini has a REST API. The request format is documented in Google's API docs.
- **Parsing the LLM response.** The prompt asks for JSON output, but LLMs aren't always clean. Handle it.

### After extraction:

Once nodes are extracted and saved, the user should be able to view the Growth Charter for the new company — same renderer, different data.

---

## Tech Stack

**For this assignment:** Use whatever you're comfortable with. Any frontend framework, any backend language, MongoDB for the database. We care about the output, not the stack.

**At DeepThought:** If hired, you'd work with our production stack — **Next.js, Tailwind CSS, MongoDB, PostgreSQL, Redis, Prisma**. Familiarity with any of these is a plus but not required for the assignment.

---

## What We Don't Care About

- Visual polish (clean and functional is enough — no animations or gradients needed)
- Authentication or user management
- Mobile responsiveness (desktop-only is fine)
- Deployment
- Comprehensive error handling for edge cases

## What We Care About Deeply

- **Did you read `context.md`?** The Growth Charter should render with correct content in every section. Wrong text in wrong places = you didn't understand the data model.
- **Does the regex rendering work?** Load any account → charter renders from data, not hardcoded.
- **Is the data model correct?** Accounts and nodes in MongoDB, properly related, companions stored where they belong.
- **Does it actually run?** We will clone your repo, follow your README, seed the data, and check the charter output.
- **Can you explain what you built?** The code walkthrough video is where we assess comprehension.

---

## Submission

| # | Item | Where |
|---|------|-------|
| 1 | GitHub repo (public) with working code + README | Share link on Internshala |
| 2 | Commit history showing incremental development | Same repo |
| 3 | **Code walkthrough video (3-5 min)** — walk through your codebase, explain the data model, the rendering logic, and your product decisions | **Internshala chat** |

### What Your README Must Contain

1. **Setup instructions** — step-by-step commands to run the app (install deps, start MongoDB, seed data, start backend, start frontend). Assume the reader has Node/Python and MongoDB installed.
2. **Architecture overview** — one paragraph or a simple diagram: frontend, backend, MongoDB, how they connect, where the rendering happens.
3. **Which parts of `context.md` drove your data model** — show us you read it.
4. **What you'd improve with more time** — be specific. "Better UI" is vague. "Add a node editor so a DT consultant can score nodes directly instead of only seeding from JSON" is specific.

> **The code walkthrough video is mandatory. Applications without it will not be reviewed.** We need to hear you explain the system — what the nodes are, how the rendering works, why the charter looks the way it does. This cannot be faked with AI.

---

## Evaluation Criteria

| Criterion | Weight | What we're looking for |
|-----------|--------|----------------------|
| **Domain comprehension** | 30% | Does the Growth Charter render with correct, meaningful content? Does the data model reflect the node architecture described in `context.md`? |
| **Rendering logic** | 25% | Does the regex replacement work? Are render texts, companion fields, and verbatim quotes wired correctly? Would a different account with different node values produce a different charter? |
| **Backend + data model** | 20% | Is MongoDB used properly? Are accounts and nodes modeled correctly? Can data be seeded and fetched cleanly? |
| **Frontend usability** | 10% | Can a non-developer read the Growth Charter? Is the layout clear? |
| **Code walkthrough** | 10% | Can you explain what you built and why? Do you understand the domain or did you just wire code? |
| **Part B (bonus)** | +15% | Does the LLM extraction work? Can you go from transcript → nodes → rendered charter? |

---

## Logistics

- **Time:** You have **48 hours** from receiving this assignment.
- **Tools:** Use whatever you want — AI coding assistants, boilerplate generators, component libraries. The code walkthrough video proves whether you understand what you built.
- **Questions:** If something is unclear, make a reasonable assumption and state it in your README.
- **MongoDB:** Free local install or MongoDB Atlas free tier. Either works.
- **Gemini API (Part B only):** Free tier available from Google AI Studio. No cost.

---

Now open `context.md`. Read it fully before writing code. The domain context is the assignment.
