# Recruiter Review

Everything the recruiter does after an opportunity goes live: reviewing applications, scoring submissions, labeling candidates, and filtering.

---

## Applications Table (`ApplicationsTable.tsx`)

The primary recruiter surface. Rendered on `opportunities/[oppId]/applications/page.tsx`.

### Columns

| Column | Source | Sortable |
|--------|--------|----------|
| Candidate | `candidate.fullName` + `candidate.profilePicture` (avatar) | Yes (alpha) |
| Status | `application.status` — ACTIVE / WITHDRAWN / SHORTLISTED / REJECTED / INTERVIEW | Yes |
| Current round | `application.currentRound` / total rounds (e.g., "2 / 4") | Yes (numeric) |
| Variant | `variant.variantName` or "Baseline" | Yes |
| Labels | Array of `LabelBadge` components | No |
| Applied | `application.appliedAt` — relative time ("3 days ago") | Yes (date) |
| Last activity | Most recent `submission.submittedAt` or `submission.updatedAt` | Yes (date) |

Default sort: `appliedAt` descending (newest first).

### Filters

All filters are query-param driven (server-side filtering via `GET /opportunities/:id/applications`).

| Filter | Component | Param |
|--------|-----------|-------|
| Round | Dropdown: "All rounds" / Round 1 / Round 2 / ... | `?round=2` |
| Label | Dropdown populated from `opportunity.customLabels` + "No label" | `?label=Shortlisted` |
| Variant | Dropdown populated from process variants + "Baseline" | `?variant=clx...` |
| Status | Dropdown: ALL / ACTIVE / WITHDRAWN / SHORTLISTED / REJECTED / INTERVIEW | `?status=ACTIVE` |

Filters are combinable. Selecting multiple filters ANDs them.

### Pagination

- `?page=1&limit=50` — server-side pagination
- Show total count above table: "Showing 1-50 of 342 applications"
- Page size selector: 25 / 50 / 100

### Row click

Click a row -> navigate to `opportunities/[oppId]/applications/[appId]/page.tsx` (single candidate review).

---

## Candidate Review Panel (`CandidateReviewPanel.tsx`)

Full view of a single candidate's journey through the selection process.

### Header

```
[avatar] [full name]
[email]
Status: [ACTIVE]    Variant: [v1-baseline]    Applied: [May 1, 2026]
Labels: [Shortlisted] [Strong Thinker] [+ Add label]
```

### Round-by-round timeline

Vertical timeline. Each round is a section:

```
── Round 1: Technical Assignment ── [competence] ──────────
   Status: SUBMITTED (May 2, 2026 — 34 hours after unlock)
   
   [Submission content]
   ├── If freeform: rendered text
   ├── If guided: labeled sections (prompt label -> response)
   └── If response: reference material (collapsed) + candidate response
   
   [Candidate media]
   └── Google Drive link (if submitted) — opens in new tab
   
   [Self-assessment] (if applicable)
   └── Table: dimension | candidate rating | recruiter score (side by side)
   
   [Post-submission response] (if applicable)
   └── Rendered text
   
   [Evaluation scorecard]
   ├── If evaluations exist: show scores from each evaluator
   ├── If current user hasn't evaluated: show editable scorecard form
   └── Overall note per evaluator
   
── Round 2: Self-Reflection ── [reflection] ──────────
   Status: DRAFT (deadline in 14 hours)
   
   [Draft content — shown to recruiter in read-only, lighter styling]
   [No scorecard yet — round not submitted]
   
── Round 3: Peer Feedback ── [service] ──────────
   Status: LOCKED (not yet unlocked — candidate hasn't reached this round)
```

### What the recruiter sees per submission status

| Submission status | Recruiter sees |
|-------------------|---------------|
| `null` (not yet created) | "Not yet unlocked" — greyed out section |
| `DRAFT` | Draft content in light/italic styling. No scorecard. Shows deadline countdown |
| `SUBMITTED` | Full submission content. Scorecard available. Shows time-to-submit |

---

## Evaluation Scorecard (`EvaluationScorecard.tsx`)

Rendered inline within each round section of the candidate review panel. Uses dimensions from `roundTemplate.evaluationDimensions`.

### Layout (when scoring)

```
┌─ Evaluation: Technical Assignment ─────────────────┐
│                                                     │
│ Depth of thinking (1-5)                             │
│ "Is this genuine reflection or surface-level?"      │
│ [1] [2] [3] [4] [5]     Note: [____________]       │
│                                                     │
│ Authenticity (1-5)                                  │
│ "Does this feel like the candidate's own voice?"    │
│ [1] [2] [3] [4] [5]     Note: [____________]       │
│                                                     │
│ Overall note:                                       │
│ [______________________________________________]    │
│                                                     │
│ [Save evaluation]                                   │
└─────────────────────────────────────────────────────┘
```

Each dimension row:
- Label + description (from template)
- Score selector: numbered buttons 1 to `maxScore` (5 or 10). Active score highlighted
- Optional note field (max 500 chars)

Bottom: overall note textarea (no char limit for MVP) + save button.

### Layout (when viewing existing evaluations)

If the current recruiter has already evaluated, show their scores as editable (pre-filled). Save button changes to "Update evaluation."

If other recruiters have also evaluated, show their scores below in read-only cards:

```
── Your evaluation ──
[editable scorecard — pre-filled]

── Recruiter: Priya Sharma ── (May 3, 2026) ──
Depth of thinking: 4/5 — "Genuine but could go deeper"
Authenticity: 5/5
Overall: "Strong candidate. Clear voice."

── Recruiter: Ankit Mehta ── (May 3, 2026) ──
Depth of thinking: 3/5 — "Some generic phrasing"
Authenticity: 3/5
Overall: "Needs verification in interview."
```

### Self-assessment comparison

When the round has `reflectionTrigger === "self_assessment"`, show the candidate's self-ratings alongside the recruiter's scores:

```
| Dimension         | Candidate self-rating | Your score | Gap |
|-------------------|-----------------------|------------|-----|
| Clarity           | 4                     | 2          | -2  |
| Self-awareness    | 3                     | 4          | +1  |
```

Gap column is color-coded:
- Negative (candidate rated higher than recruiter) — amber
- Positive (candidate rated lower) — neutral
- Zero — green

The gap is a diagnostic signal, not a judgment. A candidate who consistently over-rates themselves shows low self-awareness. A candidate who under-rates shows either humility or low confidence — the recruiter interprets.

### No evaluation dimensions defined

If the round template has no `evaluationDimensions`, the scorecard section shows only:

```
Overall note:
[______________________________________________]
[Save note]
```

No dimension scoring — just a free-form recruiter note.

---

## Label Manager (`LabelManager.tsx`)

### Applying labels

In the candidate review header, next to existing labels:

```
Labels: [Shortlisted] [Strong Thinker] [+ Add label]
```

Click "+ Add label" -> dropdown of available labels from `opportunity.customLabels`, excluding labels the current recruiter has already applied to this candidate.

Selecting a label optionally shows a note field:

```
Label: Interview Ready
Note (optional): [Schedule call this week — strong all 3 rounds]
[Apply]
```

Calls `POST /opportunities/:id/labels` with `candidateId`, `label`, `note`.

### Removing labels

Each label badge the current recruiter created shows an "x" on hover:

```
[Shortlisted ×]  [Strong Thinker ×]  [Interview Ready] (by Priya — not removable by you)
```

Only the recruiter who applied the label can remove it. Labels from other recruiters are shown without the remove button — tooltip shows who applied it.

### Label Badge (`LabelBadge.tsx`)

```typescript
props: {
  label: string;
  labeledBy: string;      // recruiter name
  note?: string;          // shown in tooltip on hover
  isOwn: boolean;         // true if current user applied it
  onRemove?: () => void;  // only if isOwn
}
```

Colors: deterministic from label text (hash to hue, same approach as tag colors). `customLabels` on the opportunity define the available labels — these are not system-wide, they're per-opportunity.

---

## Variant Display

### In the applications table

Variant column shows the variant name as a badge. "Baseline" for candidates not assigned to any variant.

### In the candidate review panel

Header shows variant name. If the candidate is in a variant with round overrides, show which rounds are overridden:

```
Variant: v2-with-self-assessment
  Round 2 override: reflectionTrigger changed to self_assessment
```

This helps the recruiter understand why a candidate's Round 2 experience differs from baseline.

### In process analytics

Variant comparison table (see `04-designer-forms.md` → Process Analytics section). The recruiter can see completion rates and average scores per variant.

---

## Recruiter Dashboard (`recruiter/dashboard/page.tsx`)

Landing page after recruiter login.

### Sections

**Active opportunities:**

Cards for each opportunity with status=LIVE:

```
┌─────────────────────────────────────┐
│ DT Fellowship - Software Developer  │
│ Published May 1, 2026               │
│                                     │
│ 342 applicants                      │
│ 89 in Round 1 | 31 in Round 2 | 19 in Round 3 │
│                                     │
│ 12 pending review (submitted, not yet evaluated) │
│ 5 labeled "Interview Ready"         │
│                                     │
│ [Review applications]               │
└─────────────────────────────────────┘
```

**Pending reviews count:**

Across all opportunities, count submissions where:
- `status === "SUBMITTED"`
- No `RoundEvaluation` exists from the current recruiter for this submission

Show as: "You have 47 submissions to review across 3 opportunities."

**Recently evaluated:**

List of the recruiter's last 10 evaluations with candidate name, round, and score summary. Click -> opens candidate review panel.

---

## Opportunity Detail (`recruiter/opportunities/[oppId]/page.tsx`)

### Sections

**Overview:**
- Title, description, status, published date
- Linked selection process name + round count
- Custom labels list
- Status transition buttons: "Go Live" (DRAFT->LIVE), "Close Applications" (LIVE->CLOSED), "Archive" (CLOSED->ARCHIVED)

**Quick stats:**
- Total applicants
- Applications by status (ACTIVE / WITHDRAWN / SHORTLISTED / REJECTED / INTERVIEW)
- Round-by-round funnel (same data as process analytics, scoped to this opportunity)

**Navigation:**
- [Review applications] -> applications table
- [Leaderboard (admin)] -> admin leaderboard
- [Process analytics] -> process analytics (if DESIGNER role also held)

---

## Status Transitions (Recruiter Actions on Applications)

MVP does not include bulk status changes. Recruiter changes application status from the candidate review panel:

```
Status: [ACTIVE ▾]
  -> ACTIVE
  -> SHORTLISTED
  -> REJECTED
  -> INTERVIEW
  -> WITHDRAWN (only if candidate initiated — recruiter cannot set this)
```

Status change calls `PATCH /opportunities/:id/applications/:appId` (not yet in the API spec — add this endpoint):

```json
{
  "status": "SHORTLISTED"
}
```

Validation:
- WITHDRAWN can only be set by the candidate (return 403 if recruiter attempts)
- All other transitions are valid from any status (recruiter has full discretion)
