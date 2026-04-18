# DT Fellowship Assignment: The Daily Reflection Tree

## Build an AI-Powered End-of-Day Reflection Agent

**Duration:** 48 hours | **Difficulty:** Intermediate | **Tools:** Any AI tools, any language

---

## The Problem

Most employees end their workday without reflecting on *how* they worked — only *what* they did. Over time, this creates blind spots: reactive habits harden, entitlement creeps in, and self-centeredness narrows perspective. A simple daily reflection — guided by the right questions in the right sequence — can shift someone from unconscious drift to intentional growth.

Your job: build a **tree-driven conversational agent** that walks an employee through a structured end-of-day reflection. The tree determines what to ask based on how the person answers.

---

## The Three Axes of Reflection

Your reflection tree must explore three psychological axes. Each axis is a spectrum — the agent's job is to help the employee *locate themselves* on it honestly, not lecture them.

### Axis 1: Locus — Victim vs Victor

> "Why does this always happen to me?" vs "I chose how I responded."

**The psychology:**
- **Locus of Control** (Julian Rotter, 1954): People with an *internal* locus believe their actions shape outcomes. Those with an *external* locus believe circumstances, luck, or others control what happens to them.
- **Growth Mindset** (Carol Dweck, 2006): The belief that abilities can be developed through effort, strategy, and feedback — versus a *fixed mindset* that treats talent as innate and unchangeable.

**What the agent should surface:**
- When the employee describes a setback, do they attribute it to external forces ("the client changed requirements") or to something within their control ("I didn't clarify scope early enough")?
- The goal is not to force blame. It's to help the person see where they *did* have agency — even in genuinely difficult situations.
- The agent should gently reframe: "You mentioned X happened to you. Was there a moment where you had a choice? What did you choose?"

**Design considerations:**
- Don't be preachy. A reflection tool that moralizes gets closed on day two.
- The tree should branch based on how the employee describes their day — if they describe events passively ("things fell apart"), probe for agency. If they describe events actively ("I decided to..."), acknowledge and deepen.

---

### Axis 2: Orientation — Contribution vs Entitlement

> "What did I earn today?" vs "What did I give today?"

**The psychology:**
- **Psychological Entitlement** (Campbell et al., 2004): A stable belief that one deserves more than others — independent of actual contribution. Entitled employees focus on what the organization *owes* them.
- **Organizational Citizenship Behavior** (Organ, 1988): Discretionary effort beyond formal job requirements — helping colleagues, improving processes, volunteering for unglamorous work.

**What the agent should surface:**
- When reflecting on the day, does the employee talk about what they *received* (recognition, credit, resources) or what they *contributed* (helped someone, improved something, taught something)?
- Entitlement is often invisible to the person holding it. The agent doesn't accuse — it redirects attention: "You mentioned wanting more support from your manager. That's valid. Separately — was there a moment today where *you* supported someone else?"
- Track patterns: if the employee consistently frames the day around what they didn't get, the tree should gently surface this over multiple sessions.

**Design considerations:**
- The Contribution-Entitlement axis is sensitive. The tree must never shame.
- Use **contrast questions**: "What's one thing you expected today? What's one thing you gave that wasn't expected of you?"
- Allow the employee to sit with the asymmetry without the agent resolving it for them.

---

### Axis 3: Radius — Self-Centrism vs Altrocentrism

> "How did today affect *me*?" vs "How did today affect *us*?"

**The psychology:**
- **Self-Transcendence** (Maslow, 1969 — the often-overlooked peak above self-actualization): The shift from "What do I need?" to "What does the world need from me?" Maslow's later work argued that the healthiest humans orient outward.
- **Perspective-Taking** (Batson, 2011): The cognitive act of imagining another's experience. Not sympathy ("I feel sorry for you") but empathy-as-understanding ("I see what you're navigating").
- **Terror Management Theory** (Greenberg, Solomon, Pyszczynski): Much anxiety and emotional pain stems from self-focused rumination. Orienting toward others reduces existential dread — not by avoiding problems, but by contextualizing them within something larger.

**What the agent should surface:**
- When the employee describes stress or frustration, is the frame entirely self-referential ("I'm overwhelmed", "I didn't get to finish my work") or does it include others ("My delay affected the team's timeline", "I noticed a colleague struggling")?
- Most emotional pain at work is amplified by self-centeredness. The agent should help the employee zoom out: "You had a hard day. Who else was in the room? What were they dealing with?"
- The goal is **transcendence** — not selflessness, but the realization that contributing to something beyond yourself is the most reliable path to meaning and reduced suffering.

**Design considerations:**
- This axis often unlocks naturally when the first two are explored well. If someone sees their agency (Axis 1) and shifts to contribution (Axis 2), the radius of concern tends to expand on its own.
- Use **widening questions**: "How did this affect you?" → "How did this affect your team?" → "If you could help one person tomorrow based on what you learned today, who would it be?"

---

## Technical Requirements

### The Tree

You must build a **tree data structure** where:

1. **Nodes** represent conversational moments — a greeting, a question, a reflection prompt, an acknowledgment, a transition.
2. **Edges** represent transitions — determined by the employee's response or by conditions evaluated against accumulated context.
3. **Branching logic** is declarative — the tree *describes* the conversation flow; code *walks* it. Hardcoded `if/else` chains are not trees.

**Minimum node types:**

| Type | Purpose | Behavior |
|------|---------|----------|
| `start` | Session opening | Auto-advances after greeting |
| `question` | Ask the employee something | Waits for response, presents options or free text |
| `decision` | Internal branching | Evaluates a condition, picks a child — no user-visible output |
| `reflection` | Surface an insight | Displays a reframe or observation; user-paced (they click "continue" when ready) |
| `bridge` | Transition between axes | Brief connecting statement, auto-advances |
| `end` | Session close | Summary or closing message |

You may add more node types. Creativity here is encouraged.

### Conditions and Branching

Each decision node should evaluate a **condition** against the conversation state to determine which child to visit. Examples:

```
condition: "response contains passive language"
condition: "self-references > other-references"  
condition: "axis1.score < 3"
condition: "mentioned a colleague by name"
condition: "frustration detected"
```

You decide how to implement condition evaluation — regex, keyword matching, sentiment analysis, LLM classification, or a scoring system. The key constraint: **branching must be driven by the tree, not by imperative code.**

### State Accumulation

As the conversation progresses, the agent should accumulate state:

- **Selections**: What the employee chose at each question
- **Scores or signals**: Per-axis indicators (e.g., internal vs external locus count)
- **Narrative**: Any free-text the employee provides
- **History**: Which nodes were visited, in what order

This state feeds back into condition evaluation, creating an adaptive conversation.

### The Conversation

The agent should feel like a **thoughtful conversation**, not a survey. Techniques:

- **Interpolation**: Use the employee's own words back to them. "You said '{their phrase}' — tell me more about that."
- **Pacing**: Not every node needs a response. Some are just acknowledgments. Some are pauses.
- **Warmth without sycophancy**: The agent respects the employee. It doesn't praise every answer. It doesn't say "Great reflection!" after obvious statements.
- **Progressive depth**: Start easy ("How was your day?"), go deeper ("Where did you have a choice today?"), arrive at insight ("Who benefited from your work today?").

---

## Deliverables

### 1. The Tree Definition (data, not code)

Provide your tree in a structured, human-readable format — JSON, YAML, TSV, or any format where the tree structure is *visible as data*. We should be able to read your tree and understand the entire conversation flow without running any code.

Minimum: **20 nodes** across all three axes, with at least **5 decision points** that branch based on employee responses.

### 2. The Agent (working code)

A runnable program that:
- Loads the tree from the data file (not hardcoded)
- Walks the tree, rendering each node appropriately
- Accepts user input at question nodes
- Evaluates conditions at decision nodes
- Accumulates state throughout the conversation
- Produces a **reflection summary** at the end

Interface: CLI is fine. Web UI is a bonus but not required. Voice is a bonus.

**Language:** Any. Python, TypeScript, Go, Rust — whatever you're productive in.

### 3. A Sample Conversation (transcript)

Run the agent yourself. Provide a full transcript showing:
- The conversation flow for at least two different "personas" — one who tends toward victim/entitled/self-centric, and one who tends toward victor/contributing/altrocentric
- How the tree branched differently for each
- The reflection summary each received

### 4. A Brief Write-Up (max 2 pages)

- What psychological model did you use to score or classify responses?
- How did you design the branching — what trade-offs did you make?
- What would you change with more time?
- What surprised you while building this?

---

## Evaluation Criteria

| Criterion | Weight | What we're looking for |
|-----------|--------|----------------------|
| **Tree design** | 30% | Is the tree thoughtful? Do branches feel purposeful, not arbitrary? Does the conversation flow naturally? |
| **Psychological grounding** | 25% | Do the questions actually surface the three axes? Is the reframing genuine or performative? |
| **Technical execution** | 20% | Does the code work? Is the tree truly data-driven? Is the separation between tree (data) and walker (code) clean? |
| **Conversation quality** | 15% | Does it feel like talking to a wise colleague, not a chatbot? Is there warmth without sycophancy? |
| **Surprise factor** | 10% | Did you do something we didn't expect? A novel node type, an elegant scoring system, a beautiful visualization of the tree, a voice interface — anything that shows you went beyond the spec. |

---

## Hints

- Start by writing the tree on paper. Draw the branches. Walk through it as if you were the employee. *Then* code it.
- The hardest part is writing good questions. Spend time here. A technically perfect tree with shallow questions will score lower than a rough prototype with questions that make people think.
- LLMs are excellent at classifying sentiment and detecting passive vs active language. Use them at decision nodes if you want — but the *tree structure* should be yours, not generated.
- The three axes are not independent. Someone who sees their agency (Axis 1) naturally starts thinking about contribution (Axis 2), which naturally widens their radius of concern (Axis 3). Your tree can exploit this progression.
- Read the original papers if you have time. Dweck's "Mindset", Rotter's locus of control scale, Maslow's 1969 paper on self-transcendence. Understanding the source material will make your questions sharper.

---

## Submission

- A Git repository (GitHub/GitLab) with:
  - The tree data file
  - The agent code
  - The sample transcripts
  - The write-up
- Send the repo link to the email provided in your application.

**Deadline:** 5 days from receipt of this assignment.

---

*This assignment has no single correct answer. We're looking for how you think about human psychology, how you translate that thinking into structure, and whether you can build something that makes a person pause and reflect. The best submissions will be the ones where the builder clearly used the tool on themselves.*
