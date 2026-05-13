# Candidate Submission Flow

This is the state machine for the candidate's experience on a single round.

---

## State machine

```
[Round unlocked]
     |
     v
  DRAFT ──save──> DRAFT (repeatable)
     |
     v
  Submit clicked
     |
     ├── Validate completeness (see rules below)
     |   ├── FAIL -> show errors, stay in DRAFT
     |   └── PASS -> continue
     |
     ├── Has self_assessment trigger?
     |   ├── YES -> show SelfAssessmentForm (candidate rates 1-5 on each dimension)
     |   |         -> on confirm, include ratings in submit payload
     |   └── NO -> skip
     |
     v
  SUBMITTED (locked, irreversible)
     |
     ├── Has post_submission_prompt trigger?
     |   ├── YES -> show PostSubmissionPrompt (follow-up question)
     |   |         -> candidate answers, saved to submission.postSubmissionResponse
     |   └── NO -> skip
     |
     ├── Has gemini_self_feedback trigger?
     |   ├── YES -> show "Get AI Feedback" button (available anytime after submit)
     |   └── NO -> skip
     |
     ├── Is there a next round?
     |   ├── YES -> show inter-round content (if configured) -> unlock next round
     |   └── NO -> show completion message
     |
     v
  [Next round or done]
```

---

## What renders on the submission page

The `RoundSubmissionPage.tsx` reads `template.submissionMode` and renders the matching editor:

```
if submissionMode === "freeform":
  render FreeformEditor
    - Single <textarea> with draft save
    - No structured prompts

if submissionMode === "guided":
  render GuidedEditor
    - For each guidedPrompt: render labeled <textarea>
    - Show charLimit counter if set
    - All fields save together on draft save

if submissionMode === "response":
  render ResponseEditor
    - Top section: RichContentRenderer for referenceContent
    - If referenceMediaUrl: MediaPlayer
    - Bottom section: FreeformEditor for candidate's response
```

### Always present (regardless of mode):

```
- DeadlineCountdown showing time until deadlineAt
- MediaPlayer for template.mediaUrl (designer's instructional voice/video)
- RichContentRenderer for template.guidelinesHtml
- CandidateMediaUpload if candidateMediaRequirement !== "none"
  - Shows Google Drive link input
  - Shows permission instructions: "Set sharing to 'Anyone with the link can view'"
  - If requirement starts with "required_": shows validation error on submit if empty
  - Label changes based on type: "Upload voice note" / "Upload photo" / "Upload video"
- Draft save button (always visible, saves current state)
- Submit button (validates, then locks)
```

---

## Submit validation rules

On submit, validate based on the round template's configuration:

| Condition | Rule |
|-----------|------|
| `submissionMode === "freeform"` | `content` must be non-empty, min 10 chars |
| `submissionMode === "guided"` | Every prompt in `guidedPrompts` must have a non-empty response. If `charLimit` is set, response must not exceed it |
| `submissionMode === "response"` | `content` must be non-empty, min 10 chars |
| `candidateMediaRequirement` starts with `required_` | `candidateMediaUrl` must be a non-empty URL |
| `reflectionTrigger === "self_assessment"` | Every dimension must have a rating (1-5 integer) |
| `reflectionTrigger === "post_submission_prompt"` | Not validated on submit — this is answered AFTER submit |

---

## Self-assessment flow (when `reflectionTrigger === "self_assessment"`)

1. Candidate clicks Submit
2. Content validation passes
3. **Before locking**, a modal or inline form appears: `SelfAssessmentForm.tsx`
4. Shows each dimension (label + description) with a 1-5 rating selector
5. All dimensions must be rated
6. Candidate confirms -> ratings are included in the submit API call
7. Submission locks

The self-assessment is NOT saveable in draft. It only appears at submit time. This forces the candidate to rate themselves based on what they actually wrote, not what they plan to write.

---

## Post-submission prompt flow (when `reflectionTrigger === "post_submission_prompt"`)

1. Submission locks successfully
2. Page shows the `postSubmissionPrompt` text from the template
3. A new text area appears for the candidate to answer
4. Candidate writes and clicks "Save response"
5. Saved to `submission.postSubmissionResponse` via a PATCH or included in the submit response flow
6. This is a one-shot save (no draft mode for this field)
7. After saving, the next round unlocks

The post-submission prompt is answered AFTER the main submission is locked. The candidate cannot change their submission based on the prompt — that's the point.

---

## Inter-round content flow

After a submission is locked and any reflection steps are complete:

1. Check if the current round has `interRoundContent` or `interRoundMediaUrl`
2. If yes: show `InterRoundMessage.tsx` component
   - Renders rich HTML content
   - Plays voice/video if media URL exists
   - Shows a "Continue to Round N" button
3. If no: skip directly to next round

The inter-round message is displayed once (when transitioning). If the candidate navigates back to their application overview and then to the next round, they don't see it again.

---

## Gemini feedback flow (when `reflectionTrigger === "gemini_self_feedback"`)

Available anytime after submission is SUBMITTED. Not blocking — doesn't gate the next round.

1. Candidate sees "Get AI Feedback" button on the submitted round view
2. Clicks it
3. Frontend calls `POST /ats/my/applications/:appId/rounds/:roundId/feedback-prompt`
4. Backend returns `{ promptText: "..." }`
5. Frontend calls `navigator.clipboard.writeText(promptText)`
6. Frontend calls `window.open("https://gemini.google.com/app", "_blank")`
7. Toast: "Prompt copied to clipboard. Paste it in Gemini to get feedback."

---

## Deadline expiry (handled by CRON, not frontend)

When `deadlineAt` passes and submission is still DRAFT:

1. CRON job (`ats-deadline-check`, runs every hour) finds these submissions
2. Sets status = SUBMITTED, submittedAt = now (auto-submits whatever draft exists)
3. Creates next round's Submission record (unlocks next round)
4. `selfAssessmentRatings` stays null (missed the window)
5. `postSubmissionResponse` stays null

The candidate sees their draft was auto-submitted when they next visit.

---

## Component props reference

### `FreeformEditor`
```typescript
props: {
  value: string;
  onChange: (value: string) => void;
  minLength?: number;  // for submit validation
  disabled?: boolean;  // true when SUBMITTED
}
```

### `GuidedEditor`
```typescript
props: {
  prompts: GuidedPrompt[];
  responses: GuidedResponse[];
  onChange: (responses: GuidedResponse[]) => void;
  disabled?: boolean;
}
```

### `ResponseEditor`
```typescript
props: {
  referenceContent?: string;   // HTML
  referenceMediaUrl?: string;
  value: string;               // candidate's response text
  onChange: (value: string) => void;
  disabled?: boolean;
}
```

### `SelfAssessmentForm`
```typescript
props: {
  dimensions: SelfAssessmentDimension[];
  onSubmit: (ratings: SelfAssessmentRating[]) => void;
  onCancel: () => void;
}
// Renders as modal or inline form at submit time
```

### `PostSubmissionPrompt`
```typescript
props: {
  prompt: string;  // the question text
  onSave: (response: string) => void;
  disabled?: boolean;  // true after saved
}
```

### `CandidateMediaUpload`
```typescript
props: {
  requirement: "none" | "optional_voice" | "required_voice" | "required_photo" | "required_video";
  value: string | null;  // Google Drive URL
  onChange: (url: string) => void;
  disabled?: boolean;
}
// Shows appropriate label: "Upload hand-drawn photo" / "Upload voice note" / etc.
// Shows Drive permission instructions when requirement !== "none"
```

### `GeminiFeedbackButton`
```typescript
props: {
  applicationId: string;
  roundId: string;
  disabled?: boolean;  // true if not yet submitted
}
```

### `InterRoundMessage`
```typescript
props: {
  content?: string;     // rich HTML
  mediaUrl?: string;    // voice/video
  nextRoundName: string;
  onContinue: () => void;
}
```
