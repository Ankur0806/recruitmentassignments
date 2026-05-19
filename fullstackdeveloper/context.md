# DT Growth Platform — CRM Node System
### Context Document for the Growth Charter Renderer Assignment

**Read this entire document before writing code.** It contains the domain knowledge, data model, rendering logic, and everything you need to build the app. Skimming will cost you more time than reading carefully.

---

## What DeepThought Does

DeepThought helps Indian manufacturing MSMEs become better-run organizations. We work with founders of ₹25Cr–500Cr manufacturing companies — pharmaceutical, automotive components, specialty chemicals, food processing, textiles — who have strong products but outgrown their execution infrastructure.

Our CRM isn't a sales tracker. It's a **classification engine**. As a company moves through our funnel (from first contact to engagement), we capture structured data about who they are, what they want to grow, and what's blocking that growth. Each data point is a **node** — a classification with exactly 8 possible values. From these nodes, documents like the **Growth Charter** are auto-generated.

---

## What Is a Node?

A node is a single classification dimension in the CRM. Every node has:

- **A node ID** — a short code (e.g., `D1`, `I3`, `K1`)
- **A name** — what it measures (e.g., "KPI Selection", "Improvement Ownership")
- **Exactly 8 options** — each is a real position the observer might find, not a scale from bad to good. Option 1 is the strongest DT fit signal; option 8 is the weakest or absent. But every option describes something genuine.
- **A scored value** — which of the 8 options best matches this company, determined by a DT consultant during conversations with the founder
- **(Optional) Companion fields** — free-text data (names, numbers, quotes) that provide the specific payload. The node classifies; the companion captures specifics.
- **(Optional) Verbatim fields** — the founder's actual words (`quote`) and what they structurally mean (`interpretation`). These power the "Founder's Voice" section of the Growth Charter.

**Example:** Node `D1` (KPI Selection) classifies which growth KPI the founder cares about. If the founder says "I need more leads," D1 is scored as value `1`. If they say "my conversion rate is terrible," D1 is scored as value `2`. The node doesn't record what they said — only which of 8 positions best matches. The companion/verbatim fields capture the specifics.

---

## What Is the Growth Charter?

The Growth Charter is a personalized document generated for each company in the CRM. It mirrors the founder's growth ambition, execution gaps, and infrastructure reality back to them — assembled from their own words and the structured data captured during conversations.

**It is not written by a human.** It is assembled from node data:

1. **Node values** determine the narrative — which text appears in each section
2. **Companion fields** fill in specifics — the founder's name, the actual revenue number, the specific KPI metric
3. **Verbatim quotes** add the founder's voice — their actual words alongside the structural interpretation

The charter has a **template** with `{{placeholder}}` markers. A **regex** finds each placeholder. A **render map** converts each node's scored value into readable text. The application replaces placeholders with the correct text from the data.

**The founder reads the Growth Charter and thinks: "I've never had my growth ambition laid out this clearly."** That's the experience you're building the renderer for.

---

## Your 10 Nodes

These are the nodes your app works with. Each has 8 options. You'll store them in MongoDB, look up their render text from the render map, and place that text into the Growth Charter template.

---

### K1: Decision-Maker Identification (Profile Node)

*How well is the primary decision-maker identified and accessible?*

| Value | Option |
|-------|--------|
| 1 | Direct relationship established — met the DM, they're engaged and actively participating |
| 2 | DM identified and contacted — responsive, scheduling or in early conversation |
| 3 | DM fully profiled from research — name, title, background known, not yet contacted |
| 4 | DM identified by name and role — basic profile known, background unclear |
| 5 | DM role known but specific person uncertain — multiple candidates |
| 6 | DM contacted through intermediary — not reached directly |
| 7 | DM identified but gated — assistant or family member screening access |
| 8 | Not identified — company structure unclear, don't know who makes decisions |

**Companion fields:** `name`, `title`, `background`
**Charter usage:** Options 1–4 → charter addresses founder by name. Options 5–8 → charter uses role title only or "Decision-maker: To be confirmed."

---

### F2: Revenue Scale (Profile Node)

*What is the company's approximate annual revenue?*

| Value | Option |
|-------|--------|
| 1 | ₹500Cr+ — large mid-market |
| 2 | ₹200Cr–500Cr — upper mid-market, established operations |
| 3 | ₹100Cr–200Cr — scaling rapidly |
| 4 | ₹50Cr–100Cr — proven business model, ready to invest in growth infrastructure |
| 5 | ₹25Cr–50Cr — emerging, growth potential but limited budget |
| 6 | ₹10Cr–25Cr — early stage, founder-driven |
| 7 | Below ₹10Cr — too early for full engagement |
| 8 | Not disclosed |

**Companion fields:** `actualRevenue` (exact number if shared), `revenueSource` (public filings / founder-stated / estimated)
**Charter usage:** Options 1–7 → revenue context in Company Snapshot. Option 8 → "Revenue: Confidential."

---

### C7: Systems Maturity

*Has the company invested in operational systems — ERP, costing, MIS, digital infrastructure?*

| Value | Option |
|-------|--------|
| 1 | ERP fully implemented AND integrated across departments — real-time MIS dashboards, structured costing, automated reporting |
| 2 | ERP implemented in core functions (production, finance) — structured costing exists, some manual gaps remain |
| 3 | ERP implemented but partially adopted — some functions on system, others still on Excel or manual processes |
| 4 | Structured processes exist without ERP — formal SOPs, costing models, disciplined tracking through spreadsheets |
| 5 | Digital tools adopted piecemeal — Tally for accounting, Excel for production, WhatsApp for coordination, no integration |
| 6 | Systems investment planned or in progress — company recognizes the need, has budgeted or begun implementation |
| 7 | Runs on founder's memory and informal coordination — no structured tracking, no costing models |
| 8 | Actively resistant to systems — "we don't need software to run our business" |

**Companion fields:** `systemsInUse` (what specific tools/systems exist)
**Charter usage:** Systems context in Company Snapshot section.

---

### D1: KPI Selection

*What's the one KPI that would compound their growth most?*

| Value | Option |
|-------|--------|
| 1 | More leads into the pipeline |
| 2 | Better conversion rate of leads |
| 3 | Revenue per existing customer (upsell) |
| 4 | New offerings to existing customers (cross-sell) |
| 5 | Customer satisfaction and retention (NPS) |
| 6 | Turnaround time (TAT) |
| 7 | Quality and compliance |
| 8 | Margins — reducing waste |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The KPI headline in the Growth Ambition section.

---

### D2: A-to-B Clarity

*Where is this number today, where does it need to be?*

| Value | Option |
|-------|--------|
| 1 | Both A and B are precise — tracked rigorously, data-backed target |
| 2 | A is measured, B is a stretch target based on ambition |
| 3 | A is measured, B is what the market demands |
| 4 | A is felt but not tracked rigorously — B is clear |
| 5 | Both are approximate — a range, not a number |
| 6 | A is known, B depends on what's realistic once they start |
| 7 | B is defined from peer benchmarks — A is where they're falling short |
| 8 | First time putting a number on this — the gap was felt, never quantified |

**Companion fields:** `primaryMetric`, `currentValue`, `targetValue`, `measurementMethod`
**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The A-to-B framing in Growth Ambition with actual numbers from companions.

---

### D3: Business Unlock

*If this number moved to B — what does that unlock for the business?*

| Value | Option |
|-------|--------|
| 1 | Revenue crosses the next milestone — ₹50Cr, ₹100Cr, ₹200Cr |
| 2 | The growth model starts compounding — not just linear anymore |
| 3 | A new market or geography becomes viable |
| 4 | The business becomes investable or exit-ready |
| 5 | Profitability step-changes — growth starts paying for itself |
| 6 | The team scales — leadership layer emerges below the founder |
| 7 | Competitive position locks in — the window is theirs |
| 8 | The business becomes what the founder originally envisioned |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The "What this unlocks" section of Growth Ambition.

---

### D7: Founder Outcome

*If this KPI moved and stayed — what changes for them as the founder?*

| Value | Option |
|-------|--------|
| 1 | I focus on vision and strategy — not chasing execution daily |
| 2 | The business runs without me in every room |
| 3 | My best people grow into the leaders I need them to be |
| 4 | I prove this company can compound — not just grow year to year |
| 5 | I create something worth more than my time in it |
| 6 | I build the leadership layer I've been wanting to build |
| 7 | I build what I set out to build when I started this company |
| 8 | I enjoy running this business again — it becomes energizing, not draining |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The personal stake — "What This Means for [Founder]" section.

---

### I3: Improvement Ownership

*Who holds singular, non-transferable accountability for driving improvement in this KPI — distinct from operational delivery responsibilities?*

| Value | Option |
|-------|--------|
| 1 | A dedicated person with no other responsibilities |
| 2 | A senior leader carving out time for it |
| 3 | Me (the founder) — but with a system supporting me |
| 4 | A small team focused only on improvement |
| 5 | Each department head owning their piece |
| 6 | Someone from outside who brings structure |
| 7 | The right person doesn't exist in my team yet |
| 8 | I know who, but they're buried in operations |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The "Who Should Own This" section of the Execution Gap.

---

### I9: Prior Attempt Learning

*If they've tried this before — what broke, and what needs to change this time?*

| Value | Option |
|-------|--------|
| 1 | Needed a dedicated person — not someone splitting time with operations |
| 2 | Lacked a method — just intent and energy, no structured approach |
| 3 | Needed shorter cycles — learned too late whether it was working |
| 4 | Needed external structure — the team couldn't build the framework alone |
| 5 | Lacked the founder's involvement in reviews — delegation without oversight |
| 6 | Started too large — should prove the concept small first |
| 7 | Tracked outcomes but not activities — work wasn't visible until results appeared |
| 8 | This is the first real attempt — no prior failure to learn from |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The "What Broke Before" section of the Execution Gap.

---

### I12: Intervention Type

*Is the process there and just needs to be faster — or does the process itself need to be figured out — or has nobody done this before?*

| Value | Option |
|-------|--------|
| 1 | Process works, just needs technology to go faster |
| 2 | Process works but manual bottlenecks slow it down |
| 3 | Process exists but isn't delivering results — needs diagnosis and redesign |
| 4 | Process was designed for a different scale — worked before, breaking now |
| 5 | Parts of the process exist, other parts are missing |
| 6 | No designed process — it's ad-hoc, relationship-driven, founder-dependent |
| 7 | Process improved before and hit a ceiling — fundamentally different approach needed |
| 8 | This function doesn't exist in the company — building from scratch |

**Verbatim fields:** `quote` + `interpretation`
**Charter usage:** The "What Type of Intervention" section of the Execution Gap.

---

## Companion Fields Summary

Not every node has companion fields. Here's the map:

| Node | Companion Fields | What They Carry |
|------|-----------------|-----------------|
| **K1** | `name`, `title`, `background` | Who the charter addresses — the decision-maker's identity |
| **F2** | `actualRevenue`, `revenueSource` | The exact revenue number and where it came from |
| **C7** | `systemsInUse` | What specific tools/systems the company has |
| **D2** | `primaryMetric`, `currentValue`, `targetValue`, `measurementMethod` | The A-to-B numbers — the actual KPI data |

All D-nodes and I-nodes (D1, D2, D3, D7, I3, I9, I12) can carry **verbatim fields**: `quote` (the founder's actual words) and `interpretation` (the structural insight behind the quote). These power the Founder's Voice section.

---

## How the Growth Charter Is Rendered

### Step 1: The Template

The Growth Charter template is a markdown string with `{{placeholder}}` markers. Here is the full template your app uses:

```markdown
# Growth Charter
## {{companyName}}
### Prepared for {{founderName}}, {{founderTitle}}
*{{founderBackground}}*

---

### Company Snapshot

{{companyName}} is a {{businessDescription}}.

| | |
|---|---|
| **Revenue** | {{revenueRender}} ({{revenueActual}}, {{revenueSource}}) |
| **Systems** | {{systemsRender}}. *{{systemsDetail}}* |

---

### Growth Ambition

{{companyName}}'s highest-leverage growth opportunity is **{{kpiRender}}**.

**Where we are today:**
{{abClarityRender}}
- **Current:** {{currentValue}}
- **Target:** {{targetValue}}

**What this unlocks:**
{{unlockRender}}

---

### What This Means for {{founderName}}

{{outcomeRender}}

---

### The Execution Gap

**Who should own this improvement:**
{{ownershipRender}}

**What broke in previous attempts:**
{{priorAttemptRender}}

**What type of intervention is needed:**
{{interventionRender}}

---

### The Founder's Voice

{{verbatimTable}}

---

*Growth Charter generated from CRM nodes | {{companyName}} | {{generatedDate}}*
```

### Step 2: The Regex

The regex finds every `{{placeholder}}` in the template and replaces it with the corresponding value from a placeholder map:

```
/\{\{(\w+)\}\}/g
```

In JavaScript:
```javascript
const rendered = template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
  return placeholders[key] ?? match;
});
```

### Step 3: The Placeholder Map

You build this map from the MongoDB data. Each key matches a `{{placeholder}}` in the template. The value comes from one of three sources:

| Placeholder | Source | How to get it |
|-------------|--------|---------------|
| `companyName` | Account record | `account.companyName` |
| `businessDescription` | Account record | `account.businessDescription` |
| `founderName` | K1 companion | `k1Node.companion.name` |
| `founderTitle` | K1 companion | `k1Node.companion.title` |
| `founderBackground` | K1 companion | `k1Node.companion.background` |
| `revenueRender` | Render map | `renderMap.F2[f2Node.value]` |
| `revenueActual` | F2 companion | `f2Node.companion.actualRevenue` |
| `revenueSource` | F2 companion | `f2Node.companion.revenueSource` |
| `systemsRender` | Render map | `renderMap.C7[c7Node.value]` |
| `systemsDetail` | C7 companion | `c7Node.companion.systemsInUse` |
| `kpiRender` | Render map | `renderMap.D1[d1Node.value]` |
| `abClarityRender` | Render map | `renderMap.D2[d2Node.value]` |
| `currentValue` | D2 companion | `d2Node.companion.currentValue` |
| `targetValue` | D2 companion | `d2Node.companion.targetValue` |
| `unlockRender` | Render map | `renderMap.D3[d3Node.value]` |
| `outcomeRender` | Render map | `renderMap.D7[d7Node.value]` |
| `ownershipRender` | Render map | `renderMap.I3[i3Node.value]` |
| `priorAttemptRender` | Render map | `renderMap.I9[i9Node.value]` |
| `interventionRender` | Render map | `renderMap.I12[i12Node.value]` |
| `generatedDate` | System | Current date |

### Step 4: The Render Map

The render map converts a node's scored value (1–8) into readable narrative text. Each node has 8 entries — one per possible value. The complete render map is in `render-map.json`. Here are three examples:

**D1 (KPI Selection) — what the growth lever is:**

| Value | Render Text |
|-------|-------------|
| 1 | pipeline generation — increasing the volume of qualified leads entering the sales funnel |
| 2 | conversion optimization — improving the rate at which existing leads become paying customers |
| 3 | revenue deepening — growing wallet share from the existing customer base through upselling |
| 4 | portfolio expansion — introducing new offerings to existing customers through cross-selling |
| 5 | customer retention — strengthening satisfaction and loyalty to reduce churn and increase lifetime value |
| 6 | speed of delivery — reducing turnaround time to serve more customers with the same capacity |
| 7 | quality and compliance — elevating product standards and regulatory adherence to protect market access |
| 8 | margin improvement — reducing operational waste to increase profitability without growing revenue |

**I3 (Improvement Ownership) — who owns this:**

| Value | Render Text |
|-------|-------------|
| 1 | A dedicated person with no other responsibilities — singular focus on driving this KPI |
| 2 | A senior leader is carving out time for it — capable but splitting attention with other priorities |
| 3 | The founder will drive it personally — with a system supporting them rather than carrying the load alone |
| 4 | A small team focused only on improvement — a dedicated unit that doesn't get pulled into operations |
| 5 | Each department head owns their piece — distributed ownership with coordination needed |
| 6 | Someone from outside brings structure — external capability to design what the organization can't build alone |
| 7 | The right person doesn't exist in the organization yet — the role needs to be created or hired for |
| 8 | The founder knows who should drive it, but that person is buried in day-to-day operations — their time is consumed by routine delivery, leaving no bandwidth for systematic improvement |

**I12 (Intervention Type) — what type of fix is needed:**

| Value | Render Text |
|-------|-------------|
| 1 | The process works, it just needs technology to go faster — the logic stays the same, manual steps get automated |
| 2 | The process works but manual bottlenecks slow it down — technology replaces specific manual steps without changing process design |
| 3 | The process exists but isn't delivering results — specific steps need diagnosis and redesign |
| 4 | The process was designed for a different scale — what worked at ₹50Cr is breaking at ₹200Cr |
| 5 | Parts of the process exist, other parts are missing — the gaps need to be built, not the whole system |
| 6 | No designed process exists for this function — it's ad-hoc, relationship-driven, or founder-dependent |
| 7 | The process has been improved before and hit a ceiling — a fundamentally different approach is needed |
| 8 | This function doesn't exist in the company — it's being built from scratch |

The full render map for all 10 nodes is in `render-map.json`. Load it into your app.

### Step 5: The Founder's Voice (Verbatim Table)

The `{{verbatimTable}}` placeholder is special — it can't be a simple regex replacement because it's a dynamic table built from multiple nodes.

Your code should:
1. Iterate over all nodes for the account
2. Filter for nodes that have verbatim data (`quote` and `interpretation` fields are not empty)
3. Build a markdown table:

```markdown
| What [Founder Name] Said | What It Means |
|---|---|
| "[quote from node 1]" | [interpretation from node 1] |
| "[quote from node 2]" | [interpretation from node 2] |
| ... | ... |
```

4. Insert this table into the template in place of `{{verbatimTable}}`

Do the verbatim table replacement **before** the main regex pass (since the table itself might contain `{{` patterns that shouldn't be replaced), or handle it separately.

---

## Sample Rendered Charter

This is what the Growth Charter should look like for the sample company in `seed-data.json` (Sureflow Formulations). When you run your app, the output should match this content:

---

> # Growth Charter
> ## Sureflow Formulations
> ### Prepared for Rajesh Kapoor, Managing Director
> *M.Pharm, MBA (XLRI), 22 years in pharma manufacturing*
>
> ---
>
> ### Company Snapshot
>
> Sureflow Formulations is a pharmaceutical formulations manufacturer specializing in APIs and intermediates for the domestic market.
>
> | | |
> |---|---|
> | **Revenue** | ₹50–100 Crore annual revenue — proven business model with room to grow (₹85 Crore, founder-stated) |
> | **Systems** | Digital tools adopted piecemeal — accounting software, spreadsheets for planning, messaging apps for coordination, but no integration. *Tally for accounting, Excel for production planning, WhatsApp for coordination* |
>
> ---
>
> ### Growth Ambition
>
> Sureflow Formulations' highest-leverage growth opportunity is **pipeline generation — increasing the volume of qualified leads entering the sales funnel**.
>
> **Where we are today:**
> The gap between today and the target is felt but not tracked rigorously — the target is clear, the baseline is approximate
> - **Current:** ~12 qualified leads/month (sales team estimate)
> - **Target:** 30 qualified leads/month
>
> **What this unlocks:**
> Revenue crosses the next major milestone — ₹50Cr, ₹100Cr, or ₹200Cr — a threshold that changes the company's competitive position
>
> ---
>
> ### What This Means for Rajesh
>
> The business runs without the founder in every room — operational independence from the founder's constant presence
>
> ---
>
> ### The Execution Gap
>
> **Who should own this improvement:**
> The founder knows who should drive it, but that person is buried in day-to-day operations — their time is consumed by routine delivery, leaving no bandwidth for systematic improvement
>
> **What broke in previous attempts:**
> Previous attempts lacked a method — there was intent and energy but no structured, repeatable approach
>
> **What type of intervention is needed:**
> No designed process exists for this function — it's ad-hoc, relationship-driven, or founder-dependent
>
> ---
>
> ### The Founder's Voice
>
> | What Rajesh Said | What It Means |
> |---|---|
> | "The thing I want to fix is leads. We get maybe 12 qualified inquiries a month." | Pipeline volume is the identified bottleneck — founder sees lead generation as the highest-leverage KPI |
> | "We get maybe 12 qualified inquiries a month... I want to get to 30 per month." | Baseline is approximate (felt, not tracked), target is precise |
> | "If we cross 100 crore, everything changes. We become visible to institutional buyers." | Revenue milestone unlock — ₹100Cr is the threshold that changes competitive positioning |
> | "I want this business to run without me in every room. Right now if I take a week off, the pipeline stops." | Founder seeks operational independence — the business must function without the founder's constant presence |
> | "My sales head, Priya, she's the one who should be driving this. But she's buried." | The right owner is identified but has no protected bandwidth — routine operations consume all improvement capacity |
> | "The problem wasn't the campaigns — we didn't have a method. We just threw money at it and hoped." | Previous failure was structural (no method) not resource-based — founder has correct diagnosis |
> | "There's no designed process for lead generation. It's all ad-hoc." | No process exists — needs to be built from scratch, not optimized or repaired |

---

## MongoDB Schema

Two collections. Keep it simple.

### `accounts` collection

```json
{
  "_id": "ObjectId",
  "companyName": "string (required)",
  "businessDescription": "string",
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

### `nodes` collection

```json
{
  "_id": "ObjectId",
  "accountId": "ObjectId (ref: accounts, required)",
  "nodeId": "string (required — e.g., 'D1', 'I3', 'K1')",
  "name": "string (required — human-readable name)",
  "value": "number (required — 1 to 8)",
  "companion": {
    "// node-specific free-text fields": "object, optional"
  },
  "verbatim": {
    "quote": "string (optional — founder's actual words)",
    "interpretation": "string (optional — structural meaning)"
  },
  "scoredAt": "Date",
  "scoredBy": "string"
}
```

**Index:** `{ accountId: 1, nodeId: 1 }` — unique. One value per node per account.

---

## Part B: Node Extraction from Transcripts

In the DT workflow, nodes are scored by a consultant after a conversation with the founder. The consultant listens, then maps what they heard to the closest option for each node.

Part B automates this: given a transcript, an LLM extracts node values.

### The LLM Prompt

Use this prompt with the Gemini API. Replace `{{TRANSCRIPT}}` with the actual transcript text.

```
You are a CRM analyst for DeepThought, a B2B consulting company that helps manufacturing MSMEs grow. Given a conversation transcript with a company founder, extract values for the following CRM nodes. Each node has 8 possible options — pick the one that best matches what the founder described.

NODES:

D1 — KPI Selection (which number, if it moved, would change their business most):
1. More leads into the pipeline
2. Better conversion rate of leads
3. Revenue per existing customer (upsell)
4. New offerings to existing customers (cross-sell)
5. Customer satisfaction and retention
6. Turnaround time
7. Quality and compliance
8. Margins — reducing waste

D2 — A-to-B Clarity (where is the number today, where does it need to be):
1. Both A and B are precise — tracked rigorously, data-backed target
2. A is measured, B is a stretch target
3. A is measured, B is what the market demands
4. A is felt but not tracked — B is clear
5. Both are approximate — a range, not a number
6. A is known, B depends on what's realistic
7. B defined from benchmarks, A is where they fall short
8. First time quantifying this gap

D3 — Business Unlock (what moves if the KPI moves):
1. Revenue crosses a major milestone
2. Growth model starts compounding
3. New market or geography becomes viable
4. Business becomes investable or exit-ready
5. Profitability step-changes
6. Team scales — leadership layer emerges
7. Competitive position locks in
8. Business becomes what the founder envisioned

D7 — Founder Outcome (what changes for the founder personally):
1. Focus on vision and strategy, not daily execution
2. Business runs without me in every room
3. Best people grow into leaders
4. Company proves it can compound
5. Creates something worth more than my time
6. Builds the leadership layer
7. Builds what I originally set out to build
8. Enjoys running the business again

I3 — Improvement Ownership (who should drive this):
1. Dedicated person, no other responsibilities
2. Senior leader carving out time
3. The founder, with a supporting system
4. A small dedicated team
5. Each department head owns their piece
6. Someone external brings structure
7. Right person doesn't exist yet
8. Right person exists but is buried in operations

I9 — Prior Attempt Learning (what broke before):
1. Needed a dedicated person
2. Lacked a method
3. Needed shorter review cycles
4. Needed external structure
5. Lacked founder involvement in reviews
6. Started too large
7. Tracked outcomes not activities
8. First real attempt

I12 — Intervention Type (what kind of fix):
1. Process works, needs tech acceleration
2. Process works, manual bottlenecks
3. Process exists but not delivering
4. Process built for different scale
5. Parts exist, parts missing
6. No designed process — ad-hoc
7. Process hit a ceiling, need new approach
8. Function doesn't exist yet

K1 — Decision-Maker (who is the founder/leader):
Extract name, title, and background if mentioned.

F2 — Revenue Scale:
1. ₹500Cr+  2. ₹200–500Cr  3. ₹100–200Cr  4. ₹50–100Cr
5. ₹25–50Cr  6. ₹10–25Cr  7. Below ₹10Cr  8. Not disclosed
Extract actual revenue if mentioned.

C7 — Systems Maturity:
1. Full ERP integrated  2. ERP in core functions  3. ERP partial
4. Structured but no ERP  5. Piecemeal digital tools  6. Systems planned
7. Founder memory only  8. Actively resistant
Extract specific systems if mentioned.

INSTRUCTIONS:
- For each node, return the option number (1-8) that best matches the transcript
- Include the specific quote that supports your classification
- If a node cannot be determined, set value to null and note "not surfaced"
- For K1, F2, and C7, also extract companion field data
- For D2, extract currentValue and targetValue if mentioned

OUTPUT FORMAT (respond with valid JSON only, no other text):
{
  "account": {
    "companyName": "extracted company name",
    "businessDescription": "one-line description of what the company does"
  },
  "nodes": {
    "D1": { "value": 1, "evidence": "quote from transcript" },
    "D2": { "value": 4, "evidence": "quote", "companion": { "primaryMetric": "...", "currentValue": "...", "targetValue": "..." } },
    "D3": { "value": 1, "evidence": "quote" },
    "D7": { "value": 2, "evidence": "quote" },
    "I3": { "value": 8, "evidence": "quote" },
    "I9": { "value": 2, "evidence": "quote" },
    "I12": { "value": 6, "evidence": "quote" },
    "K1": { "value": 3, "companion": { "name": "...", "title": "...", "background": "..." } },
    "F2": { "value": 4, "companion": { "actualRevenue": "...", "revenueSource": "founder-stated" } },
    "C7": { "value": 5, "companion": { "systemsInUse": "..." } }
  }
}

TRANSCRIPT:
{{TRANSCRIPT}}
```

### Calling the Gemini API

Get a free API key from **Google AI Studio** (aistudio.google.com). The Gemini API has a REST endpoint. Here's the shape of the call:

```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=YOUR_API_KEY

{
  "contents": [{
    "parts": [{ "text": "YOUR_PROMPT_WITH_TRANSCRIPT_HERE" }]
  }]
}
```

The response will contain the generated text in `response.candidates[0].content.parts[0].text`. Parse the JSON from that text.

### Sample Transcript

A sample transcript for testing is in `sample-transcript.json`. It's a different company than the seed data — so Part B, if implemented, should create a new account and render a new charter.
