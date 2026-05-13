# ATS Dev Spec — Index

Read `../ats-module-plan.md` first for product context. These files are the execution spec.

| # | File | What it covers | Read when |
|---|------|---------------|-----------|
| 01 | [json-schemas.md](01-json-schemas.md) | Exact shapes for all JSON fields (guidedPrompts, selfAssessmentDimensions, evaluationDimensions, dimensionScores, guidedResponses, selfAssessmentRatings, roundOverrides) | Before writing any model, API, or form code |
| 02 | [api-payloads.md](02-api-payloads.md) | Request body and response body for every endpoint, grouped by actor | Before writing any route/controller/API layer code |
| 03 | [submission-flow.md](03-submission-flow.md) | Candidate submission state machine — what renders when, branching by submissionMode and reflectionTrigger, validation rules | Before building the submission page |
| 04 | [designer-forms.md](04-designer-forms.md) | Template builder and process builder — form fields, conditional UI, validation | Before building the designer pages |
| 05 | [recruiter-review.md](05-recruiter-review.md) | Review workflow — evaluation scorecard, labels, filtering, variant display | Before building the recruiter review pages |
| 06 | [auth-and-middleware.md](06-auth-and-middleware.md) | Google OAuth flow, session cookie, middleware chain, CRON jobs | Before writing auth or middleware code |
| 07 | [leaderboard.md](07-leaderboard.md) | Leaderboard SQL queries, response shape, candidate vs recruiter view | Before building the leaderboard |
| 08 | [file-map.md](08-file-map.md) | Every file to create — backend and frontend — with the parent it mirrors or extends | Before starting any file creation |
