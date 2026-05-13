# Leaderboard

Two views of the same data: a **candidate view** (public-ish, limited info) and a **recruiter view** (full names, labels, click-through). Both are per-opportunity.

---

## Data queries

### Round breakdown (shared by both views)

```sql
-- Prisma raw query or aggregated findMany
-- For each round in the opportunity's selection process:

SELECT
  r."orderIndex",
  rt."name" AS "roundName",
  rt."tags",
  COUNT(DISTINCT s."id") AS "candidatesStarted",
  COUNT(DISTINCT s."id") FILTER (WHERE s."status" = 'SUBMITTED') AS "candidatesCompleted"
FROM ats_rounds r
JOIN ats_round_templates rt ON r."roundTemplateId" = rt."id"
LEFT JOIN ats_submissions s ON s."roundId" = r."id"
  AND s."applicationId" IN (
    SELECT id FROM ats_applications WHERE "opportunityId" = $1
  )
WHERE r."selectionProcessId" = (
  SELECT "selectionProcessId" FROM ats_opportunities WHERE id = $1
)
GROUP BY r."orderIndex", rt."name", rt."tags"
ORDER BY r."orderIndex";
```

### Total applicants

```sql
SELECT COUNT(*) AS "totalApplicants"
FROM ats_applications
WHERE "opportunityId" = $1;
```

### Final shortlist (candidate view)

Only candidates whose `application.status === "SHORTLISTED"`:

```sql
SELECT
  u."fullName" AS "candidateName"
FROM ats_applications a
JOIN ats_users u ON a."candidateId" = u."id"
WHERE a."opportunityId" = $1
  AND a."status" = 'SHORTLISTED'
ORDER BY u."fullName";
```

No email, no profile picture, no scores — just the name. This is the only place candidate names are visible to other candidates.

### Current candidate position (candidate view, authenticated)

```sql
SELECT
  a."currentRound",
  (SELECT COUNT(*) FROM ats_applications WHERE "opportunityId" = $1) AS "totalApplicants",
  (SELECT COUNT(*) FROM ats_applications
   WHERE "opportunityId" = $1 AND "currentRound" >= a."currentRound") AS "candidatesAtSameRoundOrBeyond"
FROM ats_applications a
WHERE a."opportunityId" = $1
  AND a."candidateId" = $2;
```

### Full candidate list (recruiter admin view)

```sql
SELECT
  a."id" AS "applicationId",
  u."fullName",
  u."email",
  u."profilePicture",
  a."status",
  a."currentRound",
  a."appliedAt",
  v."variantName",
  (
    SELECT json_agg(json_build_object('label', cl."label", 'labeledBy', lu."fullName", 'note', cl."note"))
    FROM ats_candidate_labels cl
    JOIN ats_users lu ON cl."labeledById" = lu."id"
    WHERE cl."candidateId" = u."id" AND cl."opportunityId" = $1
  ) AS "labels"
FROM ats_applications a
JOIN ats_users u ON a."candidateId" = u."id"
LEFT JOIN ats_process_variants v ON a."variantId" = v."id"
WHERE a."opportunityId" = $1
ORDER BY a."currentRound" DESC, a."appliedAt" ASC;
```

---

## API endpoints

### `GET /ats/opportunities/:id/leaderboard` (candidate view)

Auth: `ensureAtsAuth` + `requireAtsRole("CANDIDATE")`

Response:

```json
{
  "opportunityId": "clx...",
  "opportunityTitle": "DT Fellowship - Software Developer",
  "totalApplicants": 342,
  "roundBreakdown": [
    {
      "orderIndex": 1,
      "roundName": "Technical Assignment",
      "tags": ["competence"],
      "candidatesStarted": 342,
      "candidatesCompleted": 198
    },
    {
      "orderIndex": 2,
      "roundName": "Self-Reflection",
      "tags": ["reflection"],
      "candidatesStarted": 198,
      "candidatesCompleted": 89
    },
    {
      "orderIndex": 3,
      "roundName": "Peer Feedback",
      "tags": ["service"],
      "candidatesStarted": 89,
      "candidatesCompleted": 31
    }
  ],
  "finalShortlist": [
    { "candidateName": "Harsh Kumar" },
    { "candidateName": "Priya Sharma" }
  ],
  "myPosition": {
    "currentRound": 2,
    "candidatesAtSameRoundOrBeyond": 120
  }
}
```

`myPosition` is only populated if the authenticated candidate has an application for this opportunity. `null` otherwise.

`finalShortlist` is empty until the recruiter marks candidates as SHORTLISTED.

### `GET /ats/opportunities/:id/leaderboard/admin` (recruiter view)

Auth: `ensureAtsAuth` + `requireAtsRole("RECRUITER")`

Response:

```json
{
  "opportunityId": "clx...",
  "opportunityTitle": "DT Fellowship - Software Developer",
  "totalApplicants": 342,
  "roundBreakdown": [
    {
      "orderIndex": 1,
      "roundName": "Technical Assignment",
      "tags": ["competence"],
      "candidatesStarted": 342,
      "candidatesCompleted": 198
    }
  ],
  "candidates": [
    {
      "applicationId": "clx...",
      "fullName": "Harsh Kumar",
      "email": "harsh@gmail.com",
      "profilePicture": "https://...",
      "status": "ACTIVE",
      "currentRound": 3,
      "appliedAt": "2026-05-01T...",
      "variantName": "v1-baseline",
      "labels": [
        { "label": "Shortlisted", "labeledBy": "Priya Sharma", "note": null }
      ]
    }
  ]
}
```

The `candidates` array contains every applicant with full details. Sorted by: currentRound descending (furthest along first), then appliedAt ascending (earliest first among same round).

---

## Frontend components

### `LeaderboardView.tsx` (candidate)

Page: `candidate/applications/[appId]/leaderboard/page.tsx`

Layout:

```
┌─────────────────────────────────────────────────┐
│ DT Fellowship - Software Developer              │
│ 342 applicants                                  │
│                                                 │
│ ── Your Progress ──                             │
│ You are in Round 2.                             │
│ 120 candidates are at Round 2 or beyond.        │
│                                                 │
│ ── Funnel ──                                    │
│ ┌─────────────────────────────────────────┐     │
│ │ Round 1: Technical Assignment    342→198│     │
│ │ ████████████████████████████████████████│     │
│ ├─────────────────────────────────────────┤     │
│ │ Round 2: Self-Reflection         198→89 │     │
│ │ ██████████████████████                  │     │
│ ├─────────────────────────────────────────┤     │
│ │ Round 3: Peer Feedback            89→31 │     │
│ │ █████████                               │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ── Final Shortlist ──                           │
│ (Names appear here once candidates are          │
│  shortlisted by recruiters)                     │
│                                                 │
│ Harsh Kumar                                     │
│ Priya Sharma                                    │
│                                                 │
│ ── Philosophy ──                                │
│ "The selection process is not a filter.         │
│  It is a mirror. If you did the work            │
│  sincerely, you already grew."                  │
└─────────────────────────────────────────────────┘
```

**Funnel visualization:**
- Horizontal bars, proportional width to `candidatesStarted`
- Each bar shows: round name, started count -> completed count
- Tag chips next to round name
- Candidate's current round highlighted (different color or border)

**Your Progress section:**
- Only shown if `myPosition` is not null
- "You are in Round {N}. {M} candidates are at Round {N} or beyond."

**Final Shortlist:**
- Only names. No scores, no round details, no profile pictures
- If empty: "No candidates have been shortlisted yet."
- Philosophy quote below the shortlist (from `src/config/ats/philosophy.ts`)

### `AdminLeaderboardView.tsx` (recruiter)

Page: `recruiter/opportunities/[oppId]/leaderboard/page.tsx`

Same funnel visualization as candidate view, plus:

**Full candidate table below the funnel:**

| Column | Content |
|--------|---------|
| Avatar + Name | Profile picture + full name |
| Email | Candidate email |
| Status | Application status badge |
| Current round | Round number / total |
| Variant | Variant name or "Baseline" |
| Labels | Label badges |
| Applied | Date |

Click a row -> navigate to candidate review panel (`opportunities/[oppId]/applications/[appId]`).

**Filters** (same as ApplicationsTable):
- Round, label, variant, status dropdowns
- Combined with the funnel: clicking a funnel bar could pre-filter to that round

---

## Caching

### Candidate view

```typescript
const { data } = useQuery({
  queryKey: ["leaderboard", opportunityId],
  queryFn: () => AtsLeaderboardService.getCandidateLeaderboard(opportunityId),
  staleTime: 60_000,    // 60 seconds
  refetchOnWindowFocus: true,
});
```

60-second stale time means candidates won't hammer the server refreshing, but the data stays reasonably fresh.

### Recruiter view

```typescript
const { data } = useQuery({
  queryKey: ["leaderboard-admin", opportunityId],
  queryFn: () => AtsLeaderboardService.getAdminLeaderboard(opportunityId),
  staleTime: 30_000,    // 30 seconds
  refetchOnWindowFocus: true,
});
```

30-second stale time for recruiters — they need fresher data when actively reviewing.

### No WebSocket for MVP

Real-time updates (candidate just submitted, new application came in) are not needed for MVP. React Query's stale-while-revalidate pattern is sufficient. If the recruiter needs to see the latest, they refresh or wait 30 seconds.

---

## Service layer

### `ats-leaderboard.service.ts`

```typescript
const ATS_BASE = "/api/v3/pdgms/ats";

export const AtsLeaderboardService = {
  getCandidateLeaderboard: (opportunityId: string) =>
    fetch(`${ATS_BASE}/opportunities/${opportunityId}/leaderboard`, {
      credentials: "include",
    }).then((res) => res.json().then((data) => data.response)),

  getAdminLeaderboard: (opportunityId: string) =>
    fetch(`${ATS_BASE}/opportunities/${opportunityId}/leaderboard/admin`, {
      credentials: "include",
    }).then((res) => res.json().then((data) => data.response)),
};
```
