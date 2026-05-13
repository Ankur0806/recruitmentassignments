# ATS Module — Implementation Plan

## Context

DT receives 5,000-10,000 applicants every week depending on hiring intensity. ~2,000/week for SoftDev or Data Analytics roles, 200-300/week for Psychology or Business Analytics. The current process runs manually through Internshala.

### The Purpose: Transformation, Not Automation

The ATS is **not a transactional automation tool**. The selection process itself is designed to **transform candidates into the mindsets DT expects in Fellows** — servant leadership, cross-functional thinking, delayed gratification, and genuine critical thinking. The process is the product.

Every round, every piece of philosophy messaging, every leaderboard signal, and every self-feedback loop exists to make the candidate think deeper, reflect honestly, and demonstrate genuine effort. Candidates who complete all rounds sincerely have already begun their transformation before they even join.

### The Long-Term Vision: Hire90

This starts as a DT internal product. Eventually, it becomes a product that **DT clients use to run their own Fellowship programs for younger leaders** — this is the Hire90 "Ownership Hiring" vision.

Hire90 helps companies hire people who own outcomes (KPI ownership roles) through work simulation and evidence-based selection. The ATS is the platform that powers this:
- **Role Intelligence** (blueprints, personas, simulation scenarios) stored in PDGMS
- **Work Simulation** — candidates do the job before they get the job
- **90-Day Tracker** — selection filter, onboarding plan, and review framework in one
- **Multi-tenant** — each client org runs their own Fellowship program with their own selection processes

The DT Fellowship is the proof that the method works. If Role Intelligence can identify ownership capability in a 24-year-old with no track record, the method works. The ATS must be architected so it can serve this multi-tenant future.

**Stack:** Next.js 16 (App Router) + Express/NodeBB + PostgreSQL (Prisma 7.4.2) + MongoDB (auth)
**Auth:** NodeBB Google OAuth, separate `ats_session` cookie, 7-day inactivity expiry via CRON

---

## 1. Data Model (Prisma Schema)

Add to `prisma/schema.prisma`:

**Enums:**

```prisma
enum AtsRole {
  CANDIDATE
  RECRUITER
  DESIGNER
}

enum OpportunityStatus {
  DRAFT
  LIVE
  CLOSED
  ARCHIVED
}

enum ApplicationStatus {
  ACTIVE
  WITHDRAWN
  SHORTLISTED
  REJECTED
  INTERVIEW
}

enum SubmissionStatus {
  DRAFT
  SUBMITTED
}
```

**Tables:**

| Model | Key Fields | Purpose |
|-------|-----------|---------|
| `ats_users` | id, uid (NodeBB), email, googleId, fullName, atsRoles[], lastActiveAt, sessionToken, sessionExpiresAt | ATS-specific user with Google-only auth |
| `ats_round_templates` | id, name, guidelinesHtml, mediaUrl, submissionMode, guidedPrompts, referenceContent, candidateMediaRequirement, reflectionTrigger, selfAssessmentDimensions, evaluationDimensions, tags[] | Configurable round building blocks (7 features) |
| `ats_selection_processes` | id, name, description, isLocked, createdById | Assembled from round templates, locked when opportunity goes live |
| `ats_rounds` | id, selectionProcessId, roundTemplateId, orderIndex, deadlineHours, interRoundContent?, interRoundMediaUrl? | Ordered rounds with inter-round messaging |
| `ats_opportunities` | id, title, description, selectionProcessId, createdById, status, customLabels[], publishedAt | Job posting linked to a selection process |
| `ats_applications` | id, opportunityId, candidateId, variantId?, status, currentRound | One per candidate per opportunity, optionally assigned to a process variant |
| `ats_submissions` | id, applicationId, roundId, candidateId, content, guidedResponses, selfAssessmentRatings, candidateMediaUrl, status (DRAFT->SUBMITTED) | Supports freeform, guided, and self-assessment submissions |
| `ats_candidate_labels` | id, opportunityId, candidateId, labeledById, label, note? | Custom labels, unique per (opportunity, candidate, recruiter, label) |
| `ats_round_evaluations` | id, submissionId, evaluatorId, dimensionScores, overallNote | Recruiter scores against designer-defined evaluation dimensions |
| `ats_process_variants` | id, selectionProcessId, variantName, splitPercentage, roundOverrides | A/B testing — split candidates across process configurations |

**Full Prisma Models:**

```prisma
model AtsUser {
  id               String          @id @default(cuid())
  uid              Int             // NodeBB user ID
  email            String          @unique
  fullName         String
  googleId         String          @unique
  profilePicture   String?
  atsRoles         AtsRole[]
  lastActiveAt     DateTime        @default(now())
  sessionToken     String?         @unique
  sessionExpiresAt DateTime?
  createdAt        DateTime        @default(now())
  updatedAt        DateTime        @updatedAt

  applications     Application[]
  submissions      Submission[]
  evaluations      RoundEvaluation[]
  candidateLabels  CandidateLabel[] @relation("LabeledBy")
  labelsReceived   CandidateLabel[] @relation("LabeledCandidate")
  opportunities    Opportunity[]
  roundTemplates   RoundTemplate[]
  selectionProcesses SelectionProcess[]

  @@index([uid])
  @@index([sessionToken])
  @@index([lastActiveAt])
  @@map("ats_users")
}

model RoundTemplate {
  id                        String    @id @default(cuid())
  name                      String
  description               String?
  createdById               String
  createdAt                 DateTime  @default(now())
  updatedAt                 DateTime  @updatedAt

  // Guidelines — rich HTML instructions + optional voice/video from designer
  guidelinesHtml            String    @db.Text
  mediaUrl                  String?
  mediaType                 String?   // "audio" | "video" | null

  // Feature 1: Submission mode
  submissionMode            String    @default("freeform")  // "freeform" | "guided" | "response"
  guidedPrompts             Json?     // [{label, placeholder, charLimit?}] — used when mode is "guided"
  referenceContent          String?   @db.Text  // text/HTML shown to candidate — used when mode is "response"
  referenceMediaUrl         String?   // media shown to candidate — used when mode is "response"

  // Feature 2: Candidate media requirement
  candidateMediaRequirement String    @default("none")  // "none" | "optional_voice" | "required_voice" | "required_photo" | "required_video"

  // Feature 3: Reflection trigger (what happens after submit)
  reflectionTrigger         String    @default("none")  // "none" | "gemini_self_feedback" | "self_assessment" | "post_submission_prompt"
  selfAssessmentDimensions  Json?     // [{label, description}] — candidate rates themselves 1-5
  postSubmissionPrompt      String?   @db.Text  // follow-up question shown after submit

  // Feature 4: Evaluation scorecard (for recruiter review)
  evaluationDimensions      Json?     // [{label, description, scale}] — recruiter scores against these

  // Metadata
  tags                      String[]  // free-form: ["competence", "reflection", "service", "authenticity-filter"]

  createdBy                 AtsUser   @relation(fields: [createdById], references: [id])
  rounds                    Round[]

  @@map("ats_round_templates")
}

model SelectionProcess {
  id            String    @id @default(cuid())
  name          String
  description   String?
  createdById   String
  isLocked      Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  createdBy     AtsUser   @relation(fields: [createdById], references: [id])
  rounds        Round[]
  opportunities Opportunity[]
  variants      ProcessVariant[]

  @@map("ats_selection_processes")
}

model Round {
  id                   String    @id @default(cuid())
  selectionProcessId   String
  roundTemplateId      String
  orderIndex           Int
  deadlineHours        Int
  customGuidelinesHtml String?   @db.Text
  customMediaUrl       String?

  // Feature 5: Inter-round content (shown between this round and the next)
  interRoundContent    String?   @db.Text  // rich HTML message
  interRoundMediaUrl   String?             // voice/video from designer

  createdAt            DateTime  @default(now())
  updatedAt            DateTime  @updatedAt

  selectionProcess     SelectionProcess @relation(fields: [selectionProcessId], references: [id], onDelete: Cascade)
  roundTemplate        RoundTemplate    @relation(fields: [roundTemplateId], references: [id])
  submissions          Submission[]

  @@unique([selectionProcessId, orderIndex])
  @@map("ats_rounds")
}

model Opportunity {
  id                  String            @id @default(cuid())
  title               String
  description         String            @db.Text
  selectionProcessId  String
  createdById         String
  status              OpportunityStatus @default(DRAFT)
  publishedAt         DateTime?
  closedAt            DateTime?
  customLabels        String[]
  createdAt           DateTime          @default(now())
  updatedAt           DateTime          @updatedAt

  selectionProcess    SelectionProcess  @relation(fields: [selectionProcessId], references: [id])
  createdBy           AtsUser           @relation(fields: [createdById], references: [id])
  applications        Application[]
  candidateLabels     CandidateLabel[]

  @@map("ats_opportunities")
}

model Application {
  id             String            @id @default(cuid())
  opportunityId  String
  candidateId    String
  variantId      String?           // which process variant (for A/B testing)
  status         ApplicationStatus @default(ACTIVE)
  currentRound   Int               @default(1)
  appliedAt      DateTime          @default(now())
  updatedAt      DateTime          @updatedAt

  opportunity    Opportunity       @relation(fields: [opportunityId], references: [id])
  candidate      AtsUser           @relation(fields: [candidateId], references: [id])
  variant        ProcessVariant?   @relation(fields: [variantId], references: [id])
  submissions    Submission[]

  @@unique([opportunityId, candidateId])
  @@map("ats_applications")
}

model Submission {
  id                     String           @id @default(cuid())
  applicationId          String
  roundId                String
  candidateId            String
  content                String           @db.Text           // freeform text
  guidedResponses        Json?            // [{promptLabel, response}] — for guided mode
  selfAssessmentRatings  Json?            // [{dimension, rating}] — candidate self-rating
  postSubmissionResponse String?          @db.Text           // answer to post-submission prompt
  candidateMediaUrl      String?          // Google Drive link for required media
  status                 SubmissionStatus @default(DRAFT)
  unlockedAt             DateTime         @default(now())
  submittedAt            DateTime?
  deadlineAt             DateTime
  createdAt              DateTime         @default(now())
  updatedAt              DateTime         @updatedAt

  application            Application      @relation(fields: [applicationId], references: [id], onDelete: Cascade)
  round                  Round            @relation(fields: [roundId], references: [id])
  candidate              AtsUser          @relation(fields: [candidateId], references: [id])
  evaluations            RoundEvaluation[]

  @@unique([applicationId, roundId])
  @@index([candidateId])
  @@map("ats_submissions")
}

// Feature 6: Evaluation scorecard — recruiter scores per submission
model RoundEvaluation {
  id              String    @id @default(cuid())
  submissionId    String
  evaluatorId     String
  dimensionScores Json      // [{dimension, score, note?}]
  overallNote     String?   @db.Text
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  submission      Submission @relation(fields: [submissionId], references: [id], onDelete: Cascade)
  evaluator       AtsUser    @relation(fields: [evaluatorId], references: [id])

  @@unique([submissionId, evaluatorId])
  @@map("ats_round_evaluations")
}

// Feature 7: Process variants — A/B testing of process configurations
model ProcessVariant {
  id                  String    @id @default(cuid())
  selectionProcessId  String
  variantName         String
  splitPercentage     Int       // 0-100
  roundOverrides      Json?     // [{roundOrderIndex, overrides}] — per-round config tweaks
  createdAt           DateTime  @default(now())

  selectionProcess    SelectionProcess @relation(fields: [selectionProcessId], references: [id], onDelete: Cascade)
  applications        Application[]

  @@map("ats_process_variants")
}

model CandidateLabel {
  id             String    @id @default(cuid())
  opportunityId  String
  candidateId    String
  labeledById    String
  label          String
  note           String?
  createdAt      DateTime  @default(now())

  opportunity    Opportunity @relation(fields: [opportunityId], references: [id], onDelete: Cascade)
  candidate      AtsUser     @relation("LabeledCandidate", fields: [candidateId], references: [id])
  labeledBy      AtsUser     @relation("LabeledBy", fields: [labeledById], references: [id])

  @@unique([opportunityId, candidateId, labeledById, label])
  @@index([opportunityId, candidateId])
  @@map("ats_candidate_labels")
}
```

**Key constraints:**
- `ats_applications`: unique on [opportunityId, candidateId]
- `ats_submissions`: unique on [applicationId, roundId]
- `ats_rounds`: unique on [selectionProcessId, orderIndex]
- `ats_round_evaluations`: unique on [submissionId, evaluatorId]
- `SelectionProcess.isLocked` = true when any attached opportunity goes LIVE (enforced at API layer)

---

## 2. Seven Configurable Features in Round Templates

These are the knobs the designer turns when creating a round template. Each feature is a field on `RoundTemplate` that changes what the candidate sees and what data is collected.

### Feature 1: Submission Mode (`submissionMode`)

| Mode | What the candidate sees | When to use |
|------|------------------------|-------------|
| `freeform` | Single plain text box | Open-ended assignments |
| `guided` | Multiple labeled text fields defined by the designer | Force structure: "What did you build?" / "What did you struggle with?" / "What would you change?" |
| `response` | Reference material (text/media) displayed above the submission box | Candidate reacts to a case study, a story, or a previous candidate's work |

For `guided`: designer defines an array of prompts, each with a label, placeholder text, and optional character limit. Stored as `guidedPrompts` JSON. Candidate responses stored as `guidedResponses` JSON on the Submission.

For `response`: designer pastes reference text or uploads reference media. Stored as `referenceContent` and `referenceMediaUrl` on the template. The candidate sees this content and writes their response in the text box below.

### Feature 2: Candidate Media Requirement (`candidateMediaRequirement`)

| Setting | What happens |
|---------|-------------|
| `none` | No media required |
| `optional_voice` | Candidate may attach a voice note (Google Drive link) |
| `required_voice` | Must submit a voice note link |
| `required_photo` | Must upload a photo (hand-drawn diagram, whiteboard, etc.) |
| `required_video` | Must submit a video walkthrough |

Candidate submits a Google Drive link. The system validates a link exists when the requirement is `required_*`. Instructions for setting Drive permissions are displayed automatically.

### Feature 3: Reflection Trigger (`reflectionTrigger`)

What happens after the candidate submits:

| Setting | What happens |
|---------|-------------|
| `none` | Move to next round |
| `gemini_self_feedback` | "Get AI Feedback" button appears — copies prompt to clipboard, opens Gemini |
| `self_assessment` | Before locking, candidate rates themselves 1-5 on designer-defined dimensions |
| `post_submission_prompt` | A follow-up question appears after submission; candidate answers it separately |

`self_assessment`: Designer defines 3-5 dimensions (stored as `selfAssessmentDimensions` JSON). Candidate rates themselves before the submission locks. Recruiter sees self-ratings alongside the actual submission — the gap between self-perception and recruiter's score is a signal.

`post_submission_prompt`: Designer writes a question that the candidate can only answer AFTER doing the work. Stored as `postSubmissionPrompt`. Candidate response stored as `postSubmissionResponse` on the Submission.

### Feature 4: Evaluation Scorecard (`evaluationDimensions`)

Designer defines dimensions the recruiter scores against when reviewing a submission. Each dimension has a label, description, and scale. Stored as `evaluationDimensions` JSON on the template.

When reviewing, the recruiter sees a scorecard UI instead of just free-form notes. Scores are stored in the `ats_round_evaluations` table per (submission, evaluator). Multiple recruiters can independently evaluate the same submission.

Aggregated scores feed into process analytics — "which round types produce higher scores on which dimensions?"

### Feature 5: Inter-Round Message (`interRoundContent` on Round)

Rich HTML + optional voice/video that the candidate sees between finishing one round and starting the next. Configured per-round in the process builder (not on the template — because the transition depends on sequence context).

The designer uses this to shift the candidate's frame before the next round. Could be philosophy messaging, framing for what comes next, or a voice note from a DT Fellow sharing their experience.

### Feature 6: Round Tags (`tags`)

Free-form string array on the template. Examples: `competence`, `reflection`, `service`, `authenticity-filter`, or any custom tag.

Used for:
- **Arc visualization** in the process builder — designer sees the balance of their assembled process at a glance
- **Process analytics** — drop-off rates, scores, and time-to-submit aggregated by tag
- **Recruiter filtering** — filter submissions by round tag in the review dashboard

Not enforced by the system — the designer decides what tags mean and how to use them.

### Feature 7: Process Variants (`ProcessVariant`)

A selection process can have 2-3 variants. Each variant can override specific round configurations (e.g., swap a `freeform` submission to `guided`, add a `self_assessment` trigger, change the inter-round message).

When an opportunity goes live with variants, candidates are randomly assigned based on `splitPercentage`. The recruiter dashboard shows which variant each candidate was assigned to. Analytics compare completion rate, scores, and drop-off by variant.

This is how the designer runs experiments. Change one variable, split traffic, compare outcomes, iterate.

---

## 3. Backend API — 5-Layer Pattern

Base path: `/api/v3/pdgms/ats/`

### File structure

```
src/routes/write/pdgms/ats/       -> auth.js, templates.js, processes.js, opportunities.js, candidate.js, leaderboard.js
src/controllers/write/pdgms/ats/  -> (mirrors routes)
src/api/pdgms/ats/                -> (mirrors routes)
src/middleware/ats-auth.js        -> ensureAtsAuth (checks ats_session cookie)
src/middleware/ats-role.js        -> requireAtsRole('DESIGNER') factory
src/pdgms/jobs/ats-session-cleanup.js  -> CRON: expire sessions
src/pdgms/jobs/ats-deadline-check.js   -> CRON: auto-submit on deadline
```

### Endpoints by actor

**Auth:**

| Method | Path | Description |
|--------|------|-------------|
| POST | `/ats/auth/google` | Initiate Google OAuth |
| GET | `/ats/auth/google/callback` | OAuth callback, create/find AtsUser, set ats_session cookie |
| POST | `/ats/auth/logout` | Clear session |
| GET | `/ats/auth/me` | Current user + roles |

**Designer** (requireAtsRole DESIGNER):

| Method | Path | Description |
|--------|------|-------------|
| POST | `/ats/templates` | Create round template |
| GET | `/ats/templates` | List all templates |
| GET | `/ats/templates/:id` | Get single template |
| PUT | `/ats/templates/:id` | Update template |
| DELETE | `/ats/templates/:id` | Delete template (only if unused) |
| POST | `/ats/templates/:id/media` | Upload voice/video for template |
| POST | `/ats/processes` | Create selection process |
| GET | `/ats/processes` | List all processes |
| GET | `/ats/processes/:id` | Get process with rounds |
| PUT | `/ats/processes/:id` | Update process (if not locked) |
| DELETE | `/ats/processes/:id` | Delete process (if not locked) |
| POST | `/ats/processes/:id/clone` | Clone process |
| POST | `/ats/processes/:id/rounds` | Add round to process |
| PUT | `/ats/processes/:id/rounds/:roundId` | Update round |
| DELETE | `/ats/processes/:id/rounds/:roundId` | Remove round |
| PATCH | `/ats/processes/:id/rounds/reorder` | Reorder rounds |
| POST | `/ats/processes/:id/variants` | Create process variant |
| PUT | `/ats/processes/:id/variants/:variantId` | Update variant |
| DELETE | `/ats/processes/:id/variants/:variantId` | Delete variant |
| GET | `/ats/processes/:id/analytics` | Process analytics (drop-off, scores by tag/variant) |

**Recruiter** (requireAtsRole RECRUITER):

| Method | Path | Description |
|--------|------|-------------|
| POST | `/ats/opportunities` | Create opportunity |
| GET | `/ats/opportunities` | List opportunities |
| GET | `/ats/opportunities/:id` | Get single opportunity |
| PUT | `/ats/opportunities/:id` | Update opportunity (metadata only) |
| PATCH | `/ats/opportunities/:id/status` | Change status (DRAFT -> LIVE -> CLOSED) |
| PATCH | `/ats/opportunities/:id/labels` | Update custom labels list |
| GET | `/ats/opportunities/:id/applications` | List all applications with filters |
| GET | `/ats/opportunities/:id/applications/:appId` | Single application with submissions |
| POST | `/ats/opportunities/:id/labels` | Apply label to candidate |
| DELETE | `/ats/opportunities/:id/labels/:labelId` | Remove label |
| POST | `/ats/opportunities/:id/submissions/:subId/evaluate` | Submit evaluation scorecard |
| PUT | `/ats/opportunities/:id/submissions/:subId/evaluate` | Update evaluation |
| GET | `/ats/opportunities/:id/leaderboard/admin` | Full leaderboard with all names |

**Candidate** (requireAtsRole CANDIDATE):

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ats/opportunities/public` | List LIVE opportunities |
| GET | `/ats/opportunities/:id/public` | Opportunity details + process overview |
| POST | `/ats/opportunities/:id/apply` | Apply to opportunity |
| GET | `/ats/my/applications` | List my applications |
| GET | `/ats/my/applications/:appId` | My application with submissions |
| PUT | `/ats/my/applications/:appId/rounds/:roundId` | Save draft |
| POST | `/ats/my/applications/:appId/rounds/:roundId/submit` | Lock & submit (one-shot) |
| POST | `/ats/my/applications/:appId/rounds/:roundId/feedback-prompt` | Generate Gemini feedback prompt |
| GET | `/ats/opportunities/:id/leaderboard` | Public leaderboard |

### Middleware

```javascript
// ats-auth.js — checks 'ats_session' cookie, validates against AtsUser.sessionToken
// Sets req.atsUser = { id, uid, email, atsRoles, ... }
// Updates lastActiveAt on every authenticated request
// Returns 401 if invalid/expired

// ats-role.js — factory function
// requireAtsRole('DESIGNER') returns middleware checking req.atsUser.atsRoles
// Returns 403 if missing role

// Route registration example:
setupApiRoute(router, 'post', '/templates',
  [ensureAtsAuth, requireAtsRole('DESIGNER')],
  controllers.createTemplate
);
```

---

## 4. Auth Flow — Separate Google Login

### Login sequence

1. User visits `/ats/auth/login`
2. Clicks "Sign in with Google"
3. Frontend redirects to `/api/v3/pdgms/ats/auth/google`
4. Backend initiates Google OAuth (using existing passport-google-oauth20)
5. Google redirects to `/api/v3/pdgms/ats/auth/google/callback`
6. Backend:
   - Validates Google token
   - Finds or creates `AtsUser` record (by googleId)
   - If no NodeBB user exists for this email, creates one (or links to existing)
   - Generates `sessionToken` (crypto.randomUUID), stores on AtsUser
   - Sets `sessionExpiresAt` = now + 7 days
   - Sets `ats_session` cookie with the session token
   - Redirects to `/ats/candidate/dashboard`
7. Frontend AtsAuthProvider detects cookie, calls `/ats/auth/me`, populates atsAuthAtom

### Session management (frontend)

```typescript
// Jotai atoms — src/lib/store/ats-auth.ts
export const atsAuthAtom = atomWithStorage<AtsAuthState>("ats_auth", {
  user: null,
  isAuthenticated: false,
  loading: false,
});

// Hook — src/lib/hooks/ats/use-ats-auth.ts
// Checks for 'ats_session' cookie (not 'express.sid')
// Calls AtsAuthService.me()
// No organization/department context
// Returns atsRoles, hasAtsRole(role) checker
```

### Cookie config

```typescript
export const ATS_COOKIE_CONFIG = {
  secure: process.env.NODE_ENV === "production",
  httpOnly: false,
  sameSite: "lax",
  path: "/",
  maxAge: 60 * 60 * 24 * 7, // 7 days
};
```

---

## 5. Frontend — Route Structure

```
src/app/ats/
  layout.tsx                    -> ATS root layout (own auth provider, no PDGMS sidebar)
  page.tsx                      -> Landing / redirect

  auth/
    login/page.tsx              -> Google-only login (DT philosophy on left, login on right)
    callback/page.tsx           -> OAuth callback handler

  candidate/
    layout.tsx                  -> Candidate sidebar layout
    dashboard/page.tsx          -> My applications overview
    opportunities/page.tsx      -> Browse live opportunities
    opportunities/[oppId]/page.tsx -> Opportunity detail + apply
    applications/[appId]/page.tsx  -> Application progress view
    applications/[appId]/rounds/[roundId]/page.tsx -> Round submission page
    applications/[appId]/leaderboard/page.tsx -> Opportunity leaderboard

  recruiter/
    layout.tsx                  -> Recruiter sidebar layout
    dashboard/page.tsx          -> Recruiter home
    opportunities/page.tsx      -> Manage opportunities
    opportunities/create/page.tsx -> Create opportunity
    opportunities/[oppId]/page.tsx -> Opportunity detail
    opportunities/[oppId]/applications/page.tsx -> Review all applications
    opportunities/[oppId]/applications/[appId]/page.tsx -> Single candidate review
    opportunities/[oppId]/leaderboard/page.tsx -> Admin leaderboard

  designer/
    layout.tsx                  -> Designer sidebar layout
    dashboard/page.tsx          -> Designer home
    templates/page.tsx          -> Manage round templates
    templates/create/page.tsx   -> Create template
    templates/[templateId]/page.tsx -> Edit template
    processes/page.tsx          -> Manage selection processes
    processes/create/page.tsx   -> Assemble rounds from templates
    processes/[processId]/page.tsx -> Edit/view process
```

### Frontend files to create

**State & Auth:**
- `src/lib/store/ats-auth.ts` — Jotai atoms (atsAuthAtom, atsUserAtom, atsRolesAtom)
- `src/lib/hooks/ats/use-ats-auth.ts` — mirrors use-auth.ts pattern

**Services:**
- `src/lib/services/ats/ats-auth.service.ts`
- `src/lib/services/ats/ats-template.service.ts`
- `src/lib/services/ats/ats-process.service.ts`
- `src/lib/services/ats/ats-opportunity.service.ts`
- `src/lib/services/ats/ats-candidate.service.ts`
- `src/lib/services/ats/ats-review.service.ts`
- `src/lib/services/ats/ats-leaderboard.service.ts`

**Types & Validation:**
- `src/types/ats/ats.types.ts` — all TypeScript types and enums
- `src/validations/ats/ats.validations.ts` — Zod schemas

**Config:**
- `src/config/routes/ats.ts` — route definitions
- `src/config/ats/philosophy.ts` — DT philosophy messages

### Key components

```
src/components/ats/
  auth/
    AtsAuthGuard.tsx            -> Route guard for ATS session + role
    GoogleLoginButton.tsx       -> Google OAuth button

  shared/
    AtsSidebar.tsx              -> Role-configurable sidebar
    DtPhilosophyBanner.tsx      -> Reusable DT values messaging
    RoundProgressIndicator.tsx  -> Visual round progress tracker
    RichContentRenderer.tsx     -> Renders HTML guidelines safely
    MediaPlayer.tsx             -> Audio/video player for instructions
    DeadlineCountdown.tsx       -> Countdown timer for deadlines
    InterRoundMessage.tsx       -> Displays inter-round content between rounds

  candidate/
    OpportunityCard.tsx         -> Card for browsing opportunities
    OpportunityList.tsx         -> Grid of OpportunityCards
    ApplicationProgressCard.tsx -> Application status + current round
    RoundSubmissionPage.tsx     -> Core submission page (renders based on submissionMode)
    FreeformEditor.tsx          -> Plain text editor with draft save (submissionMode: freeform)
    GuidedEditor.tsx            -> Multiple labeled text fields (submissionMode: guided)
    ResponseEditor.tsx          -> Reference material + text box (submissionMode: response)
    SelfAssessmentForm.tsx      -> 1-5 rating on designer-defined dimensions (reflectionTrigger: self_assessment)
    PostSubmissionPrompt.tsx    -> Follow-up question after submit (reflectionTrigger: post_submission_prompt)
    GeminiFeedbackButton.tsx    -> Copy-to-clipboard + open Gemini (reflectionTrigger: gemini_self_feedback)
    CandidateMediaUpload.tsx    -> Google Drive link input with permission instructions
    LeaderboardView.tsx         -> Candidate-facing leaderboard

  recruiter/
    OpportunityForm.tsx         -> Create/edit opportunity form
    ApplicationsTable.tsx       -> Table with filters (filterable by round tag, variant, label)
    CandidateReviewPanel.tsx    -> Single candidate's submissions
    EvaluationScorecard.tsx     -> Recruiter scores against designer-defined dimensions
    LabelManager.tsx            -> Apply/remove labels
    LabelBadge.tsx              -> Visual label display

  designer/
    TemplateForm.tsx            -> Create/edit round template (all 6 configurable features)
    SubmissionModeSelector.tsx  -> Choose freeform/guided/response + configure prompts/reference
    MediaRequirementSelector.tsx -> Choose candidate media requirement
    ReflectionTriggerSelector.tsx -> Choose reflection trigger + configure dimensions/prompt
    EvaluationDimensionEditor.tsx -> Define evaluation scorecard dimensions
    TagEditor.tsx               -> Add/remove round tags
    TemplateCard.tsx            -> Template preview card showing configured features
    TemplateMediaUploader.tsx   -> Upload voice/video
    ProcessBuilder.tsx          -> Drag-and-drop process assembly with arc visualization
    ProcessRoundList.tsx        -> Ordered list of rounds with inter-round content editor
    ProcessArcIndicator.tsx     -> Visual summary of round tags in the process
    RoundConfigPanel.tsx        -> Configure round (deadline, inter-round message, overrides)
    VariantManager.tsx          -> Create/manage A/B test variants
    ProcessAnalytics.tsx        -> Drop-off, scores, time-to-submit by tag and variant
```

---

## 6. Gemini Self-Feedback (reflection trigger)

Activated when the designer sets `reflectionTrigger: "gemini_self_feedback"` on a round template. Zero API cost.

Candidate flow:
1. Candidate submits a round that has Gemini feedback enabled
2. "Get AI Feedback" button appears on the submitted round
3. Backend generates prompt combining:
   - Context prompt (system framing)
   - Round guidelines (HTML stripped to plain text)
   - Candidate's submission content (freeform text or concatenated guided responses)
4. Frontend copies prompt to clipboard via `navigator.clipboard.writeText()`
5. Opens `https://gemini.google.com/app` via `window.open()`
6. Toast: "Prompt copied! Paste it in Gemini to get feedback."

### Prompt template (server-side)

```
You are reviewing a candidate's submission for a recruitment round.

=== ROUND CONTEXT ===
Round: {roundTemplate.name}
Guidelines given to candidate:
{roundTemplate.guidelinesHtml stripped to plain text}

=== CANDIDATE'S SUBMISSION ===
{submission.content or formatted guidedResponses}

=== YOUR TASK ===
Provide constructive feedback on this submission. Focus on:
1. Clarity and completeness of the response
2. Evidence of genuine thinking (vs. generic/AI-generated responses)
3. Specific areas for improvement
4. What was done well

Be honest, specific, and constructive. Do not give a score.
Focus on helping the candidate improve their thinking and communication.
```

---

## 7. Leaderboard

### Candidate view (public)

- Total applicant count
- Round-by-round counts: "342 applied -> 89 completed Round 1 -> 31 completed Round 2 -> 19 in final shortlist"
- Names shown ONLY for final shortlisted candidates
- Candidate sees their own position: "You are in Round 3. 42 candidates started. 18 reached Round 3."
- DT philosophy messaging baked in

### Recruiter view (admin)

- Everything in candidate view plus:
- Full candidate names at every round
- Labels applied to each candidate
- Click-through to individual application review
- Filter by label, round, submission status

### Data structure

```typescript
interface LeaderboardData {
  opportunityId: string;
  opportunityTitle: string;
  totalApplicants: number;
  roundBreakdown: Array<{
    roundOrder: number;
    roundName: string;
    candidatesAtRound: number;
    candidatesCompleted: number;
  }>;
  finalShortlist: Array<{
    candidateName: string;
    candidateCity?: string;
    candidateRole?: string;
  }>;
}
```

### Caching

React Query with staleTime 60s (candidate) / 30s (recruiter). No WebSocket for MVP.

---

## 8. CRON Jobs

| Job | File | Schedule | Logic |
|-----|------|----------|-------|
| Session Cleanup | `src/pdgms/jobs/ats-session-cleanup.js` | Every 6 hours | Find AtsUser where lastActiveAt < now - 7 days, clear sessionToken + sessionExpiresAt |
| Deadline Check | `src/pdgms/jobs/ats-deadline-check.js` | Every hour | Find Submissions where status=DRAFT AND deadlineAt < now. Auto-submit (lock as-is). Create next round's Submission record (unlock next round) |

Both use `cron` v4.3.4, registered in startup sequence at `src/pdgms/index.js`.

---

## 9. DT Philosophy Integration

Two layers of philosophy messaging:

**Layer 1 — Platform defaults** in `src/config/ats/philosophy.ts`, rendered by `DtPhilosophyBanner.tsx`. These are DT's own defaults for the login page, empty states, and system-level touchpoints:

| Touchpoint | Default message |
|-----------|---------|
| Login page | "The DT Fellowship is not a job. It is a mutual commitment to building something that doesn't exist yet." |
| Opportunity browse | Frame opportunities as "growth paths" not "job listings" |
| Empty state (no applications) | "Your journey begins when you choose to start." |
| Submission confirmed | "Your work is now visible. Thank you for your effort." |
| Deadline approaching | "Deadlines exist because delayed gratification is a skill, not a punishment." |

**Layer 2 — Process-level messaging** configured by the designer per selection process. These use the configurable features:

| Feature used | Where it appears |
|-------------|-----------------|
| Inter-round content (Feature 5) | Between rounds — designer crafts transition messages, philosophy framing, voice notes |
| Round template media | Before each round — designer's voice/video setting context |
| Leaderboard message | Per-process configurable text displayed on the leaderboard page |
| Welcome message | Per-opportunity configurable text shown after applying |

This two-layer approach means DT's values are baked into the platform, but each selection process (and eventually each Hire90 client) can layer their own messaging on top.

---

## 10. Phase Plan

### Phase 1 — MVP (5-6 weeks)

| Week | Scope |
|------|-------|
| 1-2 | Prisma schema migration (all 10 tables). ATS auth system (Google OAuth, session CRON, ensureAtsAuth + requireAtsRole middleware, auth endpoints). Frontend: ATS layout, auth pages, AtsAuthProvider, useAtsAuth hook, route definitions |
| 2-3 | Designer flow: RoundTemplate CRUD with all 6 configurable features (submission mode, media requirement, reflection trigger, evaluation dimensions, tags, inter-round content). SelectionProcess CRUD with round assembly, reorder, clone, arc visualization. Frontend: template form with feature selectors, process builder with arc indicator |
| 3-4 | Recruiter + Candidate core: Opportunity CRUD + status transitions + lock mechanism. Application + Submission endpoints supporting all 3 submission modes (freeform, guided, response) + self-assessment + post-submission prompt + candidate media. Labels + evaluation scorecard. Frontend: opportunity management, candidate browse + apply, round submission page (rendering by mode), recruiter review with scorecard |
| 4-6 | Leaderboard (candidate + recruiter views). Gemini feedback. Process variants + A/B assignment. Process analytics (drop-off, scores by tag/variant). Deadline CRON. Inter-round messaging. Philosophy touchpoints throughout. Polish |

### Phase 2 — Enhancements (post-MVP)

- Email/push notifications (round unlock, deadline approaching, shortlisted)
- Recruiter analytics dashboard (funnel conversion, drop-off, time-per-round)
- Rich text / file upload submissions
- Bulk recruiter actions (label, advance, reject)
- Opportunity auto-close CRON
- Public opportunity embed for external sites
- Template versioning
- Candidate and recruiter collaborative notes

### Phase 3 — Multi-Tenant / Hire90 (future)

This is the phase where the ATS becomes a product DT clients use to run their own Fellowship programs:

- **Org-scoped tenancy** — each client org gets their own selection processes, opportunities, candidates, leaderboards
- **Role Intelligence integration** — link selection processes to Hire90 Role Blueprints (KPI ownership roles, persona traits, success criteria)
- **Work Simulation rounds** — round templates that simulate real business scenarios from the client's context
- **90-Day Tracker integration** — the selection process output feeds into the onboarding tracker
- **White-label / co-branded** — client's branding on their Fellowship program
- **Client admin role** — client decision-makers can view candidates, participate in selection
- **Cross-hire learning** — PDGMS stores role intelligence so the next hire for the same type of role doesn't start from zero

### Phase 4 — Intelligence (future)

- AI authenticity scoring (detect LLM-generated submissions, internal use only)
- Submission quality signals for recruiters
- Candidate matching to opportunities based on ownership behavior signals
- Process optimization from historical data
- Fellowship candidate → ownership behavior scoring (maps to Hire90 evidence-based selection)

---

## 11. Verification Plan

| Test | Steps |
|------|-------|
| Auth | Login with Google -> verify ats_session cookie -> call /me -> verify roles -> wait 7 days inactivity -> verify session expired |
| Template Features | Create template with each submission mode (freeform, guided, response) -> verify form renders correctly. Set each reflection trigger -> verify behavior on submit. Set media requirement -> verify validation |
| Designer | Create template -> create process with 3 rounds -> verify ordering -> add inter-round content -> clone process -> verify clone is independent |
| Arc Visualization | Assemble process with tagged rounds -> verify arc indicator shows tag distribution |
| Lock | Attach process to opportunity -> publish opportunity -> verify process.isLocked = true -> verify cannot edit process |
| Candidate Apply | Browse opportunities -> apply -> verify Round 1 unlocked with deadline |
| Freeform Submission | Save draft -> re-save -> submit (lock) -> verify cannot resubmit -> verify Round 2 unlocks -> verify inter-round message appears |
| Guided Submission | Verify labeled text fields render -> fill all fields -> submit -> verify guidedResponses stored correctly |
| Response Submission | Verify reference material displays -> write response -> submit -> verify content stored |
| Self-Assessment | Submit round with self_assessment trigger -> verify rating form appears before lock -> verify ratings stored -> verify recruiter sees ratings alongside submission |
| Post-Submission Prompt | Submit round with post_submission_prompt trigger -> verify follow-up question appears -> answer -> verify stored |
| Candidate Media | Set required_photo -> submit without link -> verify validation fails -> submit with Drive link -> verify passes |
| Evaluation Scorecard | Recruiter reviews submission with evaluation dimensions -> fill scorecard -> verify stored in ats_round_evaluations -> second recruiter evaluates independently |
| Labels | Multiple recruiters label same candidate -> verify independent labels -> one rejects, other accepts -> candidate still gets interview |
| Variants | Create 2 variants (50/50 split) -> apply with 10 candidates -> verify ~5 assigned to each variant -> verify recruiter sees variant assignment |
| Leaderboard | Apply with multiple candidates -> verify counts update -> shortlist some -> verify only shortlisted names visible to candidates |
| Gemini | Submit a round with gemini_self_feedback trigger -> click Get Feedback -> verify clipboard has prompt -> verify Gemini tab opens |
| Deadline | Create submission -> let deadline pass -> verify auto-submitted -> verify next round unlocked |
| Analytics | Run candidates through process with tagged rounds and variants -> verify drop-off, scores, and time-to-submit aggregated by tag and variant |
