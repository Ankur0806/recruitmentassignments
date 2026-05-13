# Designer Forms

Two builder surfaces: **Template Builder** (create/edit round templates) and **Process Builder** (assemble rounds into a selection process). Both are DESIGNER-only pages.

---

## Template Builder (`TemplateForm.tsx`)

Single form that creates or edits a `RoundTemplate`. The form has a fixed section (always visible) and conditional sections that appear based on the designer's choices.

### Fixed section (always visible)

| Field | Component | Validation |
|-------|-----------|------------|
| `name` | Text input | Required, 3-200 chars |
| `description` | Text input | Optional, max 500 chars |
| `guidelinesHtml` | Rich text editor (Tiptap or similar) | Required, non-empty |
| `mediaUrl` | `TemplateMediaUploader` | Optional. Upload or paste URL. Accepted: audio/*, video/* |
| `mediaType` | Auto-detected from upload | Set by uploader: `"audio"` or `"video"` or `null` |
| `tags` | `TagEditor` — chip input, free-form strings | Optional. Each tag max 50 chars |

### Submission mode selector (`SubmissionModeSelector.tsx`)

Radio group with three options. Selecting one shows its configuration panel, hides others.

```
[x] Freeform    — Candidate writes in a single text box
[ ] Guided      — Candidate answers structured prompts
[ ] Response    — Candidate responds to reference material
```

**When `guided` is selected:**

Show `GuidedPromptsEditor` — a sortable list of prompt fields.

| Sub-field | Component | Validation |
|-----------|-----------|------------|
| Prompt list | Sortable list (drag to reorder) | 2-10 prompts |
| Each prompt: `label` | Text input | Required, max 200 chars |
| Each prompt: `placeholder` | Text input | Required, max 500 chars |
| Each prompt: `charLimit` | Number input | Optional, min 50 if set |
| Add prompt button | Adds new row with generated cuid `id` | Disabled at 10 prompts |
| Remove prompt button | Removes row | Disabled at 2 prompts |

Each prompt gets an `id` (cuid) generated client-side when added. This ID persists through saves and is referenced by candidate `guidedResponses`.

**When `response` is selected:**

Show reference content fields.

| Sub-field | Component | Validation |
|-----------|-----------|------------|
| `referenceContent` | Rich text editor | At least one of referenceContent or referenceMediaUrl required |
| `referenceMediaUrl` | File upload or URL input | At least one of referenceContent or referenceMediaUrl required |

**When `freeform` is selected:**

No additional fields. `guidedPrompts`, `referenceContent`, `referenceMediaUrl` are cleared from the payload before save.

### Media requirement selector (`MediaRequirementSelector.tsx`)

Dropdown or radio group:

```
[x] None
[ ] Optional voice note
[ ] Required voice note
[ ] Required photo (hand-drawn diagram, whiteboard, etc.)
[ ] Required video walkthrough
```

Maps to `candidateMediaRequirement` enum values: `none | optional_voice | required_voice | required_photo | required_video`.

No conditional fields — selection is the value.

### Reflection trigger selector (`ReflectionTriggerSelector.tsx`)

Dropdown or radio group. Selecting one shows its configuration panel.

```
[x] None            — Candidate moves to next round after submit
[ ] Gemini feedback  — "Get AI Feedback" button after submit
[ ] Self-assessment  — Candidate rates themselves before locking
[ ] Post-submission prompt — Follow-up question after locking
```

**When `self_assessment` is selected:**

Show `SelfAssessmentDimensionEditor` — a list of dimension fields.

| Sub-field | Component | Validation |
|-----------|-----------|------------|
| Dimension list | List (no drag — order doesn't matter) | 2-5 dimensions |
| Each dimension: `label` | Text input | Required, max 100 chars |
| Each dimension: `description` | Text input | Required, max 300 chars |
| Add dimension button | Adds row with generated cuid `id` | Disabled at 5 dimensions |
| Remove dimension button | Removes row | Disabled at 2 dimensions |

**When `post_submission_prompt` is selected:**

| Sub-field | Component | Validation |
|-----------|-----------|------------|
| `postSubmissionPrompt` | Textarea | Required, non-empty, max 2000 chars |

**When `gemini_self_feedback` is selected:**

No additional fields. Show a preview of the prompt template (read-only) so the designer knows what the candidate will receive.

**When `none` is selected:**

No additional fields.

### Evaluation dimensions editor (`EvaluationDimensionEditor.tsx`)

Always visible (independent of reflection trigger). This configures what recruiters score against.

| Sub-field | Component | Validation |
|-----------|-----------|------------|
| Dimension list | List | 1-10 dimensions. Optional — form can save with 0 dimensions (no scorecard) |
| Each dimension: `label` | Text input | Required, max 100 chars |
| Each dimension: `description` | Text input | Required, max 300 chars |
| Each dimension: `maxScore` | Dropdown: 5 or 10 | Required per dimension |
| Add dimension button | Adds row with generated cuid `id` | Disabled at 10 dimensions |
| Remove dimension button | Removes row | Always available |

If no evaluation dimensions are defined, recruiters review with free-form notes only (no scorecard).

### Form layout (top to bottom)

```
1. Name + Description
2. Guidelines (rich text editor)
3. Media upload
4. ── Submission Mode ──
   [selector]
   [conditional: guided prompts / reference content / nothing]
5. ── Candidate Media ──
   [selector]
6. ── Reflection Trigger ──
   [selector]
   [conditional: self-assessment dimensions / post-submission prompt / gemini preview / nothing]
7. ── Evaluation Scorecard ──
   [dimension editor]
8. ── Tags ──
   [chip input]
9. [Save] [Cancel]
```

### Edit mode restrictions

If the template is used in a locked process:
- All fields are **read-only**
- Show banner: "This template is used in a live opportunity and cannot be edited. Clone the template to make changes."
- Save button hidden, Cancel → Back

---

## Process Builder (`ProcessBuilder.tsx`)

The process builder assembles round templates into an ordered selection process. Two main areas: the round list and the arc indicator.

### Process metadata

| Field | Component | Validation |
|-------|-----------|------------|
| `name` | Text input | Required, 3-200 chars |
| `description` | Textarea | Optional, max 1000 chars |

### Round list (`ProcessRoundList.tsx`)

A sortable (drag-and-drop) list of rounds. Each round row shows:

```
[drag handle] [orderIndex] [template name] [tags as chips] [deadline] [expand/collapse]
```

**Add round:**
- Button opens a template picker modal/drawer
- Template picker shows all templates as cards (`TemplateCard.tsx`) — name, submissionMode icon, tags, media indicator
- Search/filter by name or tag
- Select template -> adds round at the end of the list
- Set `deadlineHours` (required, number input, min 1)

**Each round's expanded config (`RoundConfigPanel.tsx`):**

| Field | Component | Validation |
|-------|-----------|------------|
| `deadlineHours` | Number input | Required, min 1 |
| `customGuidelinesHtml` | Rich text editor | Optional. Overrides template guidelines for this round only |
| `customMediaUrl` | File upload or URL | Optional. Overrides template media for this round only |
| `interRoundContent` | Rich text editor | Optional. Shown between this round and the next |
| `interRoundMediaUrl` | File upload or URL | Optional. Voice/video for inter-round message |
| Template preview | Read-only summary of the linked template's configuration | Non-editable |

**Reorder:**
- Drag-and-drop reorder
- On drop, frontend sends `PATCH /processes/:id/rounds/reorder` with the new ID order
- `orderIndex` values are reassigned server-side (1-based)

**Remove round:**
- Delete button per row
- Confirm dialog: "Remove Round {N}: {templateName}?"
- Process must not be locked

### Arc indicator (`ProcessArcIndicator.tsx`)

A horizontal bar or stacked bar that visualizes the tag composition of the assembled process.

```
Example for a 4-round process:
  Round 1: tags [competence]
  Round 2: tags [reflection, authenticity-filter]
  Round 3: tags [service]
  Round 4: tags [reflection]

Arc visualization:
  [competence: 1] [reflection: 2] [authenticity-filter: 1] [service: 1]
```

Implementation: count tags across all rounds, render as a proportional bar with colored segments. Each unique tag gets a deterministic color (hash tag name to hue). Shows tag name + count on hover.

Helps the designer see at a glance: "Is my process balanced? Am I testing what I think I'm testing?"

### Lock behavior

When any attached opportunity goes LIVE:
- `isLocked` = true
- Round list becomes read-only (no add, remove, reorder, or edit)
- Show banner: "This process is locked because it is used in a live opportunity. Clone this process to make a new version."
- Clone button remains available (creates unlocked copy)

---

## Variant Manager (`VariantManager.tsx`)

Accessed from within the process builder. Shows existing variants and allows creating new ones.

### Variant list

Each variant row shows:

```
[variant name] [split %] [N overrides] [edit] [delete]
```

### Create/edit variant form

| Field | Component | Validation |
|-------|-----------|------------|
| `variantName` | Text input | Required, 3-100 chars |
| `splitPercentage` | Number input (slider or input) | 1-100. Sum of all variants for this process must be <= 100 |
| `roundOverrides` | Per-round override editor | See below |

**Round override editor:**

Shows each round in the process. For each round, the designer can toggle overrides:

```
Round 1: Technical Assignment
  [ ] Override submission mode    -> [dropdown: freeform/guided/response]
  [ ] Override reflection trigger -> [dropdown: none/gemini/self-assessment/post-submission]
  [ ] Override media requirement  -> [dropdown: none/optional_voice/required_voice/...]
  [ ] Override inter-round content -> [rich text editor]
  [ ] Override inter-round media   -> [file upload]

Round 2: Self-Reflection
  [x] Override reflection trigger -> [self_assessment]  (changed from none)
  ...
```

Only toggled overrides are included in the `roundOverrides` JSON. Un-toggled fields inherit from the base round.

### Split percentage display

Show a visual bar of how traffic splits:

```
[v1-baseline: 50%] [v2-with-self-assessment: 50%] [unassigned: 0%]
```

If variants sum to less than 100%, the remaining percentage goes to the base process (no variant). If variants sum to exactly 100%, all candidates get a variant.

### Lock behavior

Same as process builder — variants cannot be created, edited, or deleted when the process is locked.

---

## Process Analytics (`ProcessAnalytics.tsx`)

Read-only dashboard accessible from the process builder (or as a separate page linked from it). Only meaningful after the process has applications.

### Sections

**Funnel breakdown (by round):**

| Column | Source |
|--------|--------|
| Round name | `roundTemplate.name` |
| Tags | `roundTemplate.tags` |
| Started | Count of submissions created for this round |
| Submitted | Count of submissions with status=SUBMITTED |
| Drop-off | Started - Submitted |
| Avg time to submit | Mean of (submittedAt - unlockedAt) in hours |
| Avg evaluation score | Mean of all dimension scores for this round |

Rendered as a table + a funnel visualization (horizontal bars shrinking left to right).

**Variant comparison (if variants exist):**

| Column | Source |
|--------|--------|
| Variant name | `variant.variantName` or "Baseline" |
| Applications | Count |
| Completed all rounds | Count where currentRound > total rounds |
| Completion rate | Completed / Applications |
| Avg total score | Mean of all evaluation scores across all rounds |

Rendered as a comparison table. Highlight the winning variant (highest completion rate) if sample size > 20.

**Tag aggregation:**

Group all rounds by tag, aggregate:
- Total submissions per tag
- Average score per tag
- Average time-to-submit per tag

Helps answer: "Are reflection rounds harder than competence rounds?"

---

## Template Card (`TemplateCard.tsx`)

Used in the template list page and the template picker (when adding a round to a process).

```
┌─────────────────────────────────────┐
│ [name]                              │
│ [description — truncated to 2 lines]│
│                                     │
│ Mode: [guided]  Media: [photo req]  │
│ Trigger: [self-assessment]          │
│ Scorecard: [3 dimensions]           │
│                                     │
│ [tag] [tag] [tag]                   │
│                                     │
│ Used in 3 processes                 │
│ Created May 1, 2026                 │
└─────────────────────────────────────┘
```

Icons for submissionMode, candidateMediaRequirement, reflectionTrigger. Tag chips with deterministic colors.
