# File Map

Every file to create for the ATS module. Grouped by layer. The "mirrors" column shows which existing PDGMS file the new file follows in structure and conventions.

---

## Backend

### Routes (`src/routes/write/pdgms/ats/`)

| File | Endpoints | Mirrors |
|------|-----------|---------|
| `auth.js` | POST /auth/google, GET /auth/google/callback, POST /auth/logout, GET /auth/me | `src/routes/write/pdgms/auth.js` |
| `templates.js` | POST/GET/GET:id/PUT/DELETE /templates, POST /templates/:id/media | `src/routes/write/pdgms/mimamsaka/` pattern |
| `processes.js` | POST/GET/GET:id/PUT/DELETE /processes, POST/PUT/DELETE /processes/:id/rounds, PATCH /processes/:id/rounds/reorder, POST /processes/:id/clone, POST/PUT/DELETE /processes/:id/variants, GET /processes/:id/analytics | `src/routes/write/pdgms/mimamsaka/` pattern |
| `opportunities.js` | POST/GET/GET:id/PUT /opportunities, PATCH /opportunities/:id/status, PATCH /opportunities/:id/labels, GET /opportunities/:id/applications, GET /opportunities/:id/applications/:appId, PATCH /opportunities/:id/applications/:appId, POST/DELETE /opportunities/:id/labels, POST/PUT /opportunities/:id/submissions/:subId/evaluate | `src/routes/write/pdgms/mimamsaka/` pattern |
| `candidate.js` | GET /opportunities/public, GET /opportunities/:id/public, POST /opportunities/:id/apply, GET /my/applications, GET /my/applications/:appId, GET /my/applications/:appId/rounds/:roundId, PUT /my/applications/:appId/rounds/:roundId, POST /my/applications/:appId/rounds/:roundId/submit, POST /my/applications/:appId/rounds/:roundId/feedback-prompt | New pattern (candidate-facing) |
| `leaderboard.js` | GET /opportunities/:id/leaderboard, GET /opportunities/:id/leaderboard/admin | New |

### Controllers (`src/controllers/write/pdgms/ats/`)

| File | Mirrors |
|------|---------|
| `auth.js` | Existing auth controllers |
| `templates.js` | `src/controllers/write/pdgms/mimamsaka/` |
| `processes.js` | `src/controllers/write/pdgms/mimamsaka/` |
| `opportunities.js` | `src/controllers/write/pdgms/mimamsaka/` |
| `candidate.js` | New |
| `leaderboard.js` | New |

### API layer (`src/api/pdgms/ats/`)

| File | Mirrors |
|------|---------|
| `auth.js` | Existing API helpers |
| `templates.js` | `src/api/pdgms/mimamsaka/` |
| `processes.js` | `src/api/pdgms/mimamsaka/` |
| `opportunities.js` | `src/api/pdgms/mimamsaka/` |
| `candidate.js` | New |
| `leaderboard.js` | New |
| `validations.js` | Zod schemas from `01-json-schemas.md` |

### Middleware (`src/middleware/`)

| File | Purpose | Mirrors |
|------|---------|---------|
| `ats-auth.js` | `ensureAtsAuth` — validates ats_session cookie, sets req.atsUser | `src/middleware/` existing auth |
| `ats-role.js` | `requireAtsRole(role)` factory — checks req.atsUser.atsRoles | New |
| `ats-passport.js` | Google OAuth strategy config for ATS callback URL | Existing passport config |

### CRON jobs (`src/pdgms/jobs/`)

| File | Schedule | Purpose |
|------|----------|---------|
| `ats-session-cleanup.js` | Every 6 hours | Expire inactive sessions (7-day inactivity) |
| `ats-deadline-check.js` | Every hour | Auto-submit expired drafts, unlock next round |

### Schema

| File | Change |
|------|--------|
| `prisma/schema.prisma` | Add all 10 ATS models + 4 enums (see `ats-module-plan.md` section 1) |

### Registration

| File | Change |
|------|--------|
| `src/pdgms/index.js` | Import and start both CRON jobs. Register ATS route files |

---

## Frontend — Pages (`src/app/ats/`)

| File | Auth | Role | Description |
|------|------|------|-------------|
| `layout.tsx` | — | — | ATS root layout. Own `AtsAuthProvider`, no PDGMS sidebar |
| `page.tsx` | — | — | Landing/redirect |
| `auth/login/page.tsx` | None | — | Google-only login. DT philosophy on left panel, login button on right |
| `auth/callback/page.tsx` | None | — | OAuth callback handler. Reads code param, redirects |
| `candidate/layout.tsx` | AtsAuth | CANDIDATE | Candidate sidebar layout |
| `candidate/dashboard/page.tsx` | AtsAuth | CANDIDATE | My applications overview |
| `candidate/opportunities/page.tsx` | AtsAuth | CANDIDATE | Browse live opportunities |
| `candidate/opportunities/[oppId]/page.tsx` | AtsAuth | CANDIDATE | Opportunity detail + apply button |
| `candidate/applications/[appId]/page.tsx` | AtsAuth | CANDIDATE | Application progress view (round timeline) |
| `candidate/applications/[appId]/rounds/[roundId]/page.tsx` | AtsAuth | CANDIDATE | Round submission page |
| `candidate/applications/[appId]/leaderboard/page.tsx` | AtsAuth | CANDIDATE | Candidate leaderboard view |
| `recruiter/layout.tsx` | AtsAuth | RECRUITER | Recruiter sidebar layout |
| `recruiter/dashboard/page.tsx` | AtsAuth | RECRUITER | Recruiter home (active opps, pending reviews) |
| `recruiter/opportunities/page.tsx` | AtsAuth | RECRUITER | Manage opportunities list |
| `recruiter/opportunities/create/page.tsx` | AtsAuth | RECRUITER | Create opportunity form |
| `recruiter/opportunities/[oppId]/page.tsx` | AtsAuth | RECRUITER | Opportunity detail + stats |
| `recruiter/opportunities/[oppId]/applications/page.tsx` | AtsAuth | RECRUITER | Applications table with filters |
| `recruiter/opportunities/[oppId]/applications/[appId]/page.tsx` | AtsAuth | RECRUITER | Single candidate review panel |
| `recruiter/opportunities/[oppId]/leaderboard/page.tsx` | AtsAuth | RECRUITER | Admin leaderboard |
| `designer/layout.tsx` | AtsAuth | DESIGNER | Designer sidebar layout |
| `designer/dashboard/page.tsx` | AtsAuth | DESIGNER | Designer home |
| `designer/templates/page.tsx` | AtsAuth | DESIGNER | Template list |
| `designer/templates/create/page.tsx` | AtsAuth | DESIGNER | Template builder form |
| `designer/templates/[templateId]/page.tsx` | AtsAuth | DESIGNER | Edit template |
| `designer/processes/page.tsx` | AtsAuth | DESIGNER | Process list |
| `designer/processes/create/page.tsx` | AtsAuth | DESIGNER | Process builder |
| `designer/processes/[processId]/page.tsx` | AtsAuth | DESIGNER | Edit/view process + analytics |

**Total pages: 27**

---

## Frontend — Components (`src/components/ats/`)

### Auth (`auth/`)

| Component | Used by | Spec ref |
|-----------|---------|----------|
| `AtsAuthGuard.tsx` | All protected layouts | `06-auth-and-middleware.md` |
| `GoogleLoginButton.tsx` | Login page | `06-auth-and-middleware.md` |

### Shared (`shared/`)

| Component | Used by | Spec ref |
|-----------|---------|----------|
| `AtsSidebar.tsx` | All role layouts | New — role-configurable nav |
| `DtPhilosophyBanner.tsx` | Login, empty states, leaderboard | `ats-module-plan.md` section 9 |
| `RoundProgressIndicator.tsx` | Candidate application view | `03-submission-flow.md` |
| `RichContentRenderer.tsx` | Guidelines, inter-round content | `03-submission-flow.md` |
| `MediaPlayer.tsx` | Template media, inter-round media | `03-submission-flow.md` |
| `DeadlineCountdown.tsx` | Submission page | `03-submission-flow.md` |
| `InterRoundMessage.tsx` | Between rounds | `03-submission-flow.md` |

### Candidate (`candidate/`)

| Component | Used by | Spec ref |
|-----------|---------|----------|
| `OpportunityCard.tsx` | Opportunities browse page | New |
| `OpportunityList.tsx` | Opportunities browse page | New |
| `ApplicationProgressCard.tsx` | Candidate dashboard | New |
| `RoundSubmissionPage.tsx` | Round submission page | `03-submission-flow.md` |
| `FreeformEditor.tsx` | Submission page (freeform mode) | `03-submission-flow.md` |
| `GuidedEditor.tsx` | Submission page (guided mode) | `03-submission-flow.md` |
| `ResponseEditor.tsx` | Submission page (response mode) | `03-submission-flow.md` |
| `SelfAssessmentForm.tsx` | Submit flow (self_assessment trigger) | `03-submission-flow.md` |
| `PostSubmissionPrompt.tsx` | Post-submit (post_submission_prompt trigger) | `03-submission-flow.md` |
| `GeminiFeedbackButton.tsx` | Submitted round view | `03-submission-flow.md` |
| `CandidateMediaUpload.tsx` | Submission page (when media required) | `03-submission-flow.md` |
| `LeaderboardView.tsx` | Candidate leaderboard page | `07-leaderboard.md` |

### Recruiter (`recruiter/`)

| Component | Used by | Spec ref |
|-----------|---------|----------|
| `OpportunityForm.tsx` | Create/edit opportunity | New |
| `ApplicationsTable.tsx` | Applications page | `05-recruiter-review.md` |
| `CandidateReviewPanel.tsx` | Single candidate review page | `05-recruiter-review.md` |
| `EvaluationScorecard.tsx` | Within candidate review | `05-recruiter-review.md` |
| `LabelManager.tsx` | Within candidate review header | `05-recruiter-review.md` |
| `LabelBadge.tsx` | Applications table + review panel | `05-recruiter-review.md` |
| `AdminLeaderboardView.tsx` | Recruiter leaderboard page | `07-leaderboard.md` |

### Designer (`designer/`)

| Component | Used by | Spec ref |
|-----------|---------|----------|
| `TemplateForm.tsx` | Create/edit template pages | `04-designer-forms.md` |
| `SubmissionModeSelector.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `MediaRequirementSelector.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `ReflectionTriggerSelector.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `EvaluationDimensionEditor.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `TagEditor.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `TemplateCard.tsx` | Template list + template picker | `04-designer-forms.md` |
| `TemplateMediaUploader.tsx` | Within TemplateForm | `04-designer-forms.md` |
| `ProcessBuilder.tsx` | Process create/edit pages | `04-designer-forms.md` |
| `ProcessRoundList.tsx` | Within ProcessBuilder | `04-designer-forms.md` |
| `ProcessArcIndicator.tsx` | Within ProcessBuilder | `04-designer-forms.md` |
| `RoundConfigPanel.tsx` | Within ProcessRoundList (expanded row) | `04-designer-forms.md` |
| `VariantManager.tsx` | Within ProcessBuilder | `04-designer-forms.md` |
| `ProcessAnalytics.tsx` | Process detail page | `04-designer-forms.md` |

**Total components: 41**

---

## Frontend — Services (`src/lib/services/ats/`)

| File | Endpoints covered | Mirrors |
|------|-------------------|---------|
| `ats-auth.service.ts` | /auth/* | `src/lib/services/auth.service.ts` |
| `ats-template.service.ts` | /templates/* | `src/lib/services/mimamsaka/` pattern |
| `ats-process.service.ts` | /processes/*, /processes/:id/rounds/*, /processes/:id/variants/* | `src/lib/services/mimamsaka/` pattern |
| `ats-opportunity.service.ts` | /opportunities/* (recruiter endpoints) | `src/lib/services/mimamsaka/` pattern |
| `ats-candidate.service.ts` | /opportunities/public, /my/applications/*, submit, draft save, feedback-prompt | New |
| `ats-review.service.ts` | /opportunities/:id/submissions/:subId/evaluate, /opportunities/:id/labels/* | New |
| `ats-leaderboard.service.ts` | /opportunities/:id/leaderboard, /leaderboard/admin | New |

**Total services: 7**

---

## Frontend — State & Hooks

### Store (`src/lib/store/`)

| File | Contents | Mirrors |
|------|----------|---------|
| `ats-auth.ts` | `atsAuthAtom`, `atsUserAtom`, `atsRolesAtom` | `src/lib/store/auth.ts` |

### Hooks (`src/lib/hooks/ats/`)

| File | Exports | Mirrors |
|------|---------|---------|
| `use-ats-auth.ts` | `useAtsAuth()` — auth state, hasRole, logout | `src/lib/hooks/shared/use-auth.ts` |

---

## Frontend — Types & Validation

### Types (`src/types/ats/`)

| File | Contents |
|------|----------|
| `ats.types.ts` | All TypeScript interfaces and enums: AtsUser, AtsRole, RoundTemplate, SelectionProcess, Round, Opportunity, Application, Submission, RoundEvaluation, CandidateLabel, ProcessVariant, GuidedPrompt, SelfAssessmentDimension, EvaluationDimension, GuidedResponse, SelfAssessmentRating, DimensionScore, RoundOverride, LeaderboardData, all status enums |

### Validation (`src/validations/ats/`)

| File | Contents |
|------|----------|
| `ats.validations.ts` | Zod schemas for all JSON field types + form-level schemas for template create/edit, process create, opportunity create, submission draft/submit, evaluation submit | 

---

## Frontend — Config

### Routes (`src/config/routes/`)

| File | Contents |
|------|----------|
| `ats.ts` | Route path constants: `ATS_ROUTES.AUTH.LOGIN`, `ATS_ROUTES.CANDIDATE.DASHBOARD`, etc. |

### Philosophy (`src/config/ats/`)

| File | Contents |
|------|----------|
| `philosophy.ts` | Default DT philosophy messages keyed by touchpoint (login, browse, empty state, submission confirmed, deadline approaching) |

---

## Summary

| Layer | New files |
|-------|-----------|
| Backend routes | 6 |
| Backend controllers | 6 |
| Backend API | 7 |
| Backend middleware | 3 |
| Backend CRON | 2 |
| Prisma schema | 1 (modified) |
| Frontend pages | 27 |
| Frontend components | 41 |
| Frontend services | 7 |
| Frontend store | 1 |
| Frontend hooks | 1 |
| Frontend types | 1 |
| Frontend validations | 1 |
| Frontend config | 2 |
| **Total** | **106** |
