# API Payloads

Base path: `/api/v3/pdgms/ats`
All responses follow: `{ status: { code, message }, response: { ... } }`

---

## Auth

### `POST /ats/auth/google`

No request body. Redirects to Google OAuth consent screen.

### `GET /ats/auth/google/callback?code=...`

No request body. Google redirects here. Backend:
1. Exchanges code for Google profile (email, name, picture, googleId)
2. Upserts AtsUser by googleId
3. Generates sessionToken (crypto.randomUUID)
4. Sets cookie `ats_session` with sessionToken
5. Redirects to `/ats/candidate/dashboard`

### `POST /ats/auth/logout`

No request body. Clears `ats_session` cookie, nulls sessionToken on AtsUser.

### `GET /ats/auth/me`

Response:
```json
{
  "id": "clx...",
  "email": "candidate@gmail.com",
  "fullName": "Harsh Kumar",
  "profilePicture": "https://lh3.googleusercontent.com/...",
  "atsRoles": ["CANDIDATE"]
}
```

---

## Designer — Templates

### `POST /ats/templates`

Request:
```json
{
  "name": "Career Roadmap Activity",
  "description": "Candidate generates a career roadmap with hand-drawn diagram",
  "guidelinesHtml": "<h2>Your task</h2><p>Create a career roadmap...</p>",
  "mediaUrl": "https://drive.google.com/...",
  "mediaType": "video",
  "submissionMode": "guided",
  "guidedPrompts": [
    { "id": "clx1", "label": "Your 10-year vision", "placeholder": "Where do you see yourself...", "charLimit": 2000 },
    { "id": "clx2", "label": "Why DT", "placeholder": "What about DT..." }
  ],
  "candidateMediaRequirement": "required_photo",
  "reflectionTrigger": "self_assessment",
  "selfAssessmentDimensions": [
    { "id": "clx3", "label": "Clarity", "description": "How clearly did you articulate your vision?" },
    { "id": "clx4", "label": "Self-awareness", "description": "How honest were you about your strengths and gaps?" }
  ],
  "evaluationDimensions": [
    { "id": "clx5", "label": "Depth of thinking", "description": "Is this genuine reflection or surface-level?", "maxScore": 5 },
    { "id": "clx6", "label": "Authenticity", "description": "Does this feel like the candidate's own voice?", "maxScore": 5 }
  ],
  "tags": ["reflection", "authenticity-filter"]
}
```

Response: the created RoundTemplate object with `id`, `createdAt`, `createdById`.

Validation:
- `name`: required, 3-200 chars
- `guidelinesHtml`: required, non-empty
- `submissionMode`: must be one of `freeform | guided | response`
- If `submissionMode === "guided"`: `guidedPrompts` required, 2-10 items
- If `submissionMode === "response"`: `referenceContent` or `referenceMediaUrl` required (at least one)
- If `submissionMode === "freeform"`: `guidedPrompts`, `referenceContent`, `referenceMediaUrl` are ignored
- If `reflectionTrigger === "self_assessment"`: `selfAssessmentDimensions` required, 2-5 items
- If `reflectionTrigger === "post_submission_prompt"`: `postSubmissionPrompt` required, non-empty
- `tags`: optional string array

### `GET /ats/templates`

Query params: none (returns all templates, designer-scoped later if needed)

Response:
```json
[
  {
    "id": "clx...",
    "name": "Career Roadmap Activity",
    "description": "...",
    "submissionMode": "guided",
    "candidateMediaRequirement": "required_photo",
    "reflectionTrigger": "self_assessment",
    "tags": ["reflection", "authenticity-filter"],
    "createdAt": "2026-05-01T...",
    "usedInProcesses": 3
  }
]
```

### `GET /ats/templates/:id`

Response: full RoundTemplate object with all fields.

### `PUT /ats/templates/:id`

Same body as POST. Cannot update if template is used in a locked process (return 409).

### `DELETE /ats/templates/:id`

No body. Cannot delete if template is used in any process (return 409).

### `POST /ats/templates/:id/media`

Multipart form upload. Field name: `file`. Accepted: audio/*, video/*. Max 100MB.
Returns: `{ mediaUrl: "...", mediaType: "audio" | "video" }`.

---

## Designer — Processes

### `POST /ats/processes`

Request:
```json
{
  "name": "DT Fellowship - SoftDev Track",
  "description": "3-round process: assignment, reflection, peer feedback"
}
```

Response: created SelectionProcess with `id`, no rounds yet.

### `POST /ats/processes/:id/rounds`

Request:
```json
{
  "roundTemplateId": "clx...",
  "orderIndex": 1,
  "deadlineHours": 48,
  "interRoundContent": "<p>You've completed Round 1. Before starting Round 2, watch this message from a DT Fellow.</p>",
  "interRoundMediaUrl": "https://drive.google.com/..."
}
```

Validation:
- `roundTemplateId`: must exist
- `orderIndex`: must not conflict with existing rounds (unique constraint)
- `deadlineHours`: required, min 1
- Process must not be locked (return 409)

### `PATCH /ats/processes/:id/rounds/reorder`

Request:
```json
{
  "order": ["roundId1", "roundId2", "roundId3"]
}
```

Sets orderIndex 1, 2, 3 based on array position. Process must not be locked.

### `POST /ats/processes/:id/clone`

No body. Clones the process with all rounds. The clone is unlocked. Returns the new process.

### `POST /ats/processes/:id/variants`

Request:
```json
{
  "variantName": "v2-with-self-assessment",
  "splitPercentage": 50,
  "roundOverrides": [
    {
      "roundOrderIndex": 2,
      "overrides": {
        "reflectionTrigger": "self_assessment"
      }
    }
  ]
}
```

Validation:
- All variant splitPercentages for a process must sum to <= 100
- roundOrderIndex must exist in the process
- Process must not be locked

### `GET /ats/processes/:id/analytics`

Response:
```json
{
  "processId": "clx...",
  "totalApplications": 342,
  "roundBreakdown": [
    {
      "orderIndex": 1,
      "templateName": "Technical Assignment",
      "tags": ["competence"],
      "started": 342,
      "submitted": 198,
      "dropOff": 144,
      "avgTimeToSubmitHours": 36.2,
      "avgEvaluationScore": 3.4
    }
  ],
  "variantBreakdown": [
    {
      "variantId": "clx...",
      "variantName": "v1-baseline",
      "applications": 171,
      "completedAll": 12,
      "completionRate": 0.07
    }
  ]
}
```

---

## Recruiter — Opportunities

### `POST /ats/opportunities`

Request:
```json
{
  "title": "DT Fellowship - Software Developer",
  "description": "<p>Build systems that make companies run better...</p>",
  "selectionProcessId": "clx...",
  "customLabels": ["Shortlisted", "On Hold", "Interview Ready", "Strong Reject"]
}
```

Validation:
- `selectionProcessId`: must exist, must have at least 1 round
- `customLabels`: optional string array, each label max 50 chars

### `PATCH /ats/opportunities/:id/status`

Request:
```json
{
  "status": "LIVE"
}
```

Transitions:
- `DRAFT -> LIVE`: locks the attached SelectionProcess (sets isLocked=true). Cannot revert.
- `LIVE -> CLOSED`: stops accepting new applications. Existing applications continue.
- `CLOSED -> ARCHIVED`: soft archive.

Returns 409 if transition is invalid.

### `GET /ats/opportunities/:id/applications`

Query params:
- `?round=2` — filter by current round
- `?label=Shortlisted` — filter by label
- `?variant=clx...` — filter by variant
- `?status=ACTIVE` — filter by application status
- `?page=1&limit=50` — pagination

Response:
```json
{
  "total": 342,
  "page": 1,
  "limit": 50,
  "applications": [
    {
      "id": "clx...",
      "candidate": {
        "id": "clx...",
        "fullName": "Harsh Kumar",
        "email": "harsh@gmail.com",
        "profilePicture": "..."
      },
      "status": "ACTIVE",
      "currentRound": 2,
      "variantName": "v1-baseline",
      "labels": [
        { "label": "Shortlisted", "labeledBy": "Recruiter Name", "createdAt": "..." }
      ],
      "appliedAt": "2026-05-01T..."
    }
  ]
}
```

### `GET /ats/opportunities/:id/applications/:appId`

Response: full application with all submissions, each submission includes:
- `content` (or `guidedResponses` if guided mode)
- `selfAssessmentRatings` (if applicable)
- `postSubmissionResponse` (if applicable)
- `candidateMediaUrl` (if applicable)
- `evaluations` array (all recruiter evaluations for this submission)
- Round template info (name, tags, evaluationDimensions)

### `POST /ats/opportunities/:id/submissions/:subId/evaluate`

Request:
```json
{
  "dimensionScores": [
    { "dimensionId": "clx5", "label": "Depth of thinking", "score": 4, "note": "Genuine reflection on failure" },
    { "dimensionId": "clx6", "label": "Authenticity", "score": 3 }
  ],
  "overallNote": "Strong self-awareness but the career roadmap lacks specificity in years 5-10."
}
```

Validation:
- Every dimensionId from the round's evaluationDimensions must have a score
- Score must be 1 to dimension.maxScore
- Creates or updates RoundEvaluation (upsert on [submissionId, evaluatorId])

### `POST /ats/opportunities/:id/labels`

Request:
```json
{
  "candidateId": "clx...",
  "label": "Interview Ready",
  "note": "Strong across all 3 rounds. Schedule call."
}
```

Validation:
- `label` must be in the opportunity's `customLabels` array
- Unique constraint: one label type per recruiter per candidate per opportunity

### `DELETE /ats/opportunities/:id/labels/:labelId`

No body. Only the recruiter who created the label can delete it.

---

## Candidate

### `GET /ats/opportunities/public`

No auth required. Returns all LIVE opportunities with:
```json
[
  {
    "id": "clx...",
    "title": "DT Fellowship - Software Developer",
    "description": "...",
    "totalRounds": 3,
    "totalApplicants": 342,
    "publishedAt": "2026-05-01T..."
  }
]
```

### `POST /ats/opportunities/:id/apply`

No body. Creates Application (status=ACTIVE, currentRound=1) and first Submission (status=DRAFT, deadlineAt computed). If variants exist, randomly assigns based on splitPercentage.

Response:
```json
{
  "applicationId": "clx...",
  "currentRound": 1,
  "variantId": "clx...",
  "submission": {
    "id": "clx...",
    "roundId": "clx...",
    "status": "DRAFT",
    "unlockedAt": "2026-05-01T...",
    "deadlineAt": "2026-05-03T..."
  }
}
```

### `GET /ats/my/applications/:appId`

Response:
```json
{
  "id": "clx...",
  "opportunity": { "id": "...", "title": "..." },
  "status": "ACTIVE",
  "currentRound": 2,
  "rounds": [
    {
      "orderIndex": 1,
      "templateName": "Technical Assignment",
      "tags": ["competence"],
      "submission": {
        "id": "clx...",
        "status": "SUBMITTED",
        "submittedAt": "2026-05-02T..."
      }
    },
    {
      "orderIndex": 2,
      "templateName": "Self-Reflection",
      "tags": ["reflection"],
      "submission": {
        "id": "clx...",
        "status": "DRAFT",
        "deadlineAt": "2026-05-04T..."
      },
      "interRoundContent": "<p>Before starting this round...</p>",
      "interRoundMediaUrl": "https://..."
    },
    {
      "orderIndex": 3,
      "templateName": "Peer Feedback",
      "tags": ["service"],
      "submission": null
    }
  ]
}
```

### `GET /ats/my/applications/:appId/rounds/:roundId`

Returns the full round detail for the candidate to work on:
```json
{
  "round": {
    "id": "clx...",
    "orderIndex": 2,
    "deadlineHours": 48
  },
  "template": {
    "name": "Self-Reflection",
    "guidelinesHtml": "<h2>Reflect on your journey...</h2>",
    "mediaUrl": "https://...",
    "mediaType": "video",
    "submissionMode": "guided",
    "guidedPrompts": [
      { "id": "clx1", "label": "What did this process reveal?", "placeholder": "...", "charLimit": 2000 },
      { "id": "clx2", "label": "What would you change?", "placeholder": "..." }
    ],
    "candidateMediaRequirement": "required_photo",
    "reflectionTrigger": "self_assessment",
    "selfAssessmentDimensions": [
      { "id": "clx3", "label": "Honesty", "description": "How honest were you with yourself?" }
    ],
    "referenceContent": null,
    "referenceMediaUrl": null
  },
  "submission": {
    "id": "clx...",
    "status": "DRAFT",
    "content": "my draft so far...",
    "guidedResponses": [
      { "promptId": "clx1", "label": "What did this process reveal?", "response": "draft answer..." }
    ],
    "selfAssessmentRatings": null,
    "postSubmissionResponse": null,
    "candidateMediaUrl": null,
    "unlockedAt": "2026-05-02T...",
    "deadlineAt": "2026-05-04T..."
  }
}
```

### `PUT /ats/my/applications/:appId/rounds/:roundId` (save draft)

Request:
```json
{
  "content": "my updated draft text",
  "guidedResponses": [
    { "promptId": "clx1", "label": "What did this process reveal?", "response": "updated answer..." }
  ],
  "candidateMediaUrl": "https://drive.google.com/..."
}
```

Validation:
- Submission must be in DRAFT status (return 409 if SUBMITTED)
- Deadline must not have passed (return 409 if expired)
- No completeness validation on draft — partial saves allowed

### `POST /ats/my/applications/:appId/rounds/:roundId/submit` (lock)

Request:
```json
{
  "content": "final text",
  "guidedResponses": [
    { "promptId": "clx1", "label": "...", "response": "final answer" },
    { "promptId": "clx2", "label": "...", "response": "final answer" }
  ],
  "selfAssessmentRatings": [
    { "dimensionId": "clx3", "label": "Honesty", "rating": 4 }
  ],
  "postSubmissionResponse": "My answer to the follow-up question...",
  "candidateMediaUrl": "https://drive.google.com/..."
}
```

Validation (strict on submit):
- Submission must be in DRAFT status
- Deadline must not have passed
- If `submissionMode === "freeform"`: `content` required, min 10 chars
- If `submissionMode === "guided"`: every prompt must have a non-empty response, charLimits enforced
- If `submissionMode === "response"`: `content` required, min 10 chars
- If `candidateMediaRequirement` starts with `required_`: `candidateMediaUrl` required, must be a URL
- If `reflectionTrigger === "self_assessment"`: every dimension must have a rating 1-5
- If `reflectionTrigger === "post_submission_prompt"`: `postSubmissionResponse` required, min 10 chars

On success:
1. Set status = SUBMITTED, submittedAt = now
2. Increment application.currentRound
3. If next round exists: create new Submission (status=DRAFT, unlockedAt=now, deadlineAt=now+deadlineHours)
4. If no next round: application stays ACTIVE (completed all rounds)

Response:
```json
{
  "submitted": true,
  "nextRound": {
    "orderIndex": 3,
    "templateName": "Peer Feedback",
    "submissionId": "clx...",
    "deadlineAt": "2026-05-06T...",
    "interRoundContent": "<p>Before this final round...</p>",
    "interRoundMediaUrl": "https://..."
  }
}
```

If last round: `"nextRound": null`.

### `POST /ats/my/applications/:appId/rounds/:roundId/feedback-prompt`

No request body. Returns the Gemini prompt text:
```json
{
  "promptText": "You are reviewing a candidate's submission...\n\n=== ROUND CONTEXT ===\n..."
}
```

Only available if the round's reflectionTrigger is `gemini_self_feedback` and submission status is `SUBMITTED`. Returns 403 otherwise.
