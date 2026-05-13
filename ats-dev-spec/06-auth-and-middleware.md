# Auth and Middleware

Google OAuth login, session management, middleware chain, and CRON jobs. The ATS auth system is **completely separate** from the main PDGMS auth — different cookie, different user table, different session lifecycle.

---

## Why separate auth

The main PDGMS app uses NodeBB session cookies (`express.sid`) tied to organization membership and department roles. The ATS has different actors (candidates who have no org), different session rules (7-day inactivity expiry), and will eventually be multi-tenant (Hire90). Sharing auth would couple the ATS to PDGMS org structure.

The ATS still creates a NodeBB user (for future forum integration) but manages its own session independently.

---

## Google OAuth Flow

### Passport config

File: `src/middleware/ats-passport.js`

```javascript
const GoogleStrategy = require("passport-google-oauth20").Strategy;

const atsGoogleStrategy = new GoogleStrategy(
  {
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: `${process.env.API_BASE_URL}/api/v3/pdgms/ats/auth/google/callback`,
    scope: ["profile", "email"],
  },
  async (accessToken, refreshToken, profile, done) => {
    const email = profile.emails[0].value;
    const googleId = profile.id;
    const fullName = profile.displayName;
    const profilePicture = profile.photos?.[0]?.value || null;

    done(null, { email, googleId, fullName, profilePicture });
  }
);
```

Use the **same** Google OAuth app (same client ID/secret) as the main PDGMS login. The callback URL is different — Google allows multiple authorized redirect URIs on the same OAuth app.

### Login sequence (step by step)

```
1. User visits /ats/auth/login (frontend)
2. Clicks "Sign in with Google"
3. Frontend redirects browser to: /api/v3/pdgms/ats/auth/google
4. Backend handler calls passport.authenticate("google-ats", { scope: ["profile", "email"] })
5. Browser redirects to Google consent screen
6. User consents -> Google redirects to: /api/v3/pdgms/ats/auth/google/callback?code=...
7. Passport exchanges code for profile (email, name, picture, googleId)
8. Backend callback handler:

   a. Find AtsUser by googleId
      - EXISTS: update fullName, profilePicture, lastActiveAt
      - NOT EXISTS: create AtsUser with atsRoles: ["CANDIDATE"]
        - Also find-or-create NodeBB user by email (for future forum)
        - Store NodeBB uid on AtsUser

   b. Generate sessionToken = crypto.randomUUID()
   c. Set sessionExpiresAt = now + 7 days
   d. Save sessionToken + sessionExpiresAt on AtsUser

   e. Set cookie:
      res.cookie("ats_session", sessionToken, {
        secure: process.env.NODE_ENV === "production",
        httpOnly: false,
        sameSite: "lax",
        path: "/",
        maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days in ms
      });

   f. Redirect to /ats/candidate/dashboard (or /ats/recruiter/dashboard if has RECRUITER role)

9. Frontend AtsAuthProvider detects ats_session cookie on mount
10. Calls GET /api/v3/pdgms/ats/auth/me
11. Populates atsAuthAtom with user data
```

### Cookie: `ats_session`

| Property | Value | Reason |
|----------|-------|--------|
| `secure` | `true` in production | HTTPS only |
| `httpOnly` | `false` | Frontend needs to detect cookie presence to decide auth state |
| `sameSite` | `"lax"` | Allow cookie on top-level navigation (OAuth redirect) |
| `path` | `"/"` | Available to all routes |
| `maxAge` | 7 days | Hard max. Inactivity expiry is enforced server-side |

### Logout

```
1. Frontend calls POST /api/v3/pdgms/ats/auth/logout
2. Backend:
   a. Find AtsUser by sessionToken from cookie
   b. Set sessionToken = null, sessionExpiresAt = null
   c. Clear cookie: res.clearCookie("ats_session", { path: "/" })
3. Frontend clears atsAuthAtom, redirects to /ats/auth/login
```

---

## Middleware

### `ensureAtsAuth` (`src/middleware/ats-auth.js`)

Runs on every ATS route except `/ats/auth/*` and `/ats/opportunities/public`.

```javascript
async function ensureAtsAuth(req, res, next) {
  const token = req.cookies["ats_session"];

  if (!token) {
    return res.status(401).json({
      status: { code: 401, message: "Not authenticated" },
    });
  }

  const user = await prisma.atsUser.findUnique({
    where: { sessionToken: token },
  });

  if (!user) {
    return res.status(401).json({
      status: { code: 401, message: "Invalid session" },
    });
  }

  if (user.sessionExpiresAt && user.sessionExpiresAt < new Date()) {
    await prisma.atsUser.update({
      where: { id: user.id },
      data: { sessionToken: null, sessionExpiresAt: null },
    });
    return res.status(401).json({
      status: { code: 401, message: "Session expired" },
    });
  }

  // Refresh activity timestamp
  await prisma.atsUser.update({
    where: { id: user.id },
    data: { lastActiveAt: new Date() },
  });

  req.atsUser = {
    id: user.id,
    uid: user.uid,
    email: user.email,
    fullName: user.fullName,
    atsRoles: user.atsRoles,
    profilePicture: user.profilePicture,
  };

  next();
}
```

### `requireAtsRole` (`src/middleware/ats-role.js`)

Factory function. Returns middleware that checks `req.atsUser.atsRoles`.

```javascript
function requireAtsRole(...roles) {
  return (req, res, next) => {
    const userRoles = req.atsUser?.atsRoles || [];
    const hasRole = roles.some((role) => userRoles.includes(role));

    if (!hasRole) {
      return res.status(403).json({
        status: { code: 403, message: `Requires role: ${roles.join(" or ")}` },
      });
    }

    next();
  };
}
```

### Route registration pattern

Use the existing `setupApiRoute` helper:

```javascript
// src/routes/write/pdgms/ats/templates.js

const { setupApiRoute } = require("../../../../helpers");
const ensureAtsAuth = require("../../../../middleware/ats-auth");
const { requireAtsRole } = require("../../../../middleware/ats-role");
const controllers = require("../../../../controllers/write/pdgms/ats/templates");

module.exports = function (params) {
  const router = params.router;

  setupApiRoute(router, "post", "/templates",
    [ensureAtsAuth, requireAtsRole("DESIGNER")],
    controllers.createTemplate
  );

  setupApiRoute(router, "get", "/templates",
    [ensureAtsAuth, requireAtsRole("DESIGNER")],
    controllers.listTemplates
  );

  // ... etc
};
```

### Middleware chain order

For a typical protected designer route:

```
Request -> cookieParser -> ensureAtsAuth -> requireAtsRole("DESIGNER") -> controller
```

For candidate public routes (browse opportunities):

```
Request -> cookieParser -> controller (no auth middleware)
```

For candidate authenticated routes:

```
Request -> cookieParser -> ensureAtsAuth -> requireAtsRole("CANDIDATE") -> controller
```

---

## Role assignment

New users are created with `atsRoles: ["CANDIDATE"]` by default. DESIGNER and RECRUITER roles are assigned manually (database update or admin endpoint — no self-service role assignment in MVP).

A user can hold multiple roles (e.g., a DT team member might be both RECRUITER and DESIGNER). The frontend renders different sidebars and navigation based on the highest role present:
- DESIGNER > RECRUITER > CANDIDATE (priority order for default landing page)
- Role switcher in the sidebar if multiple roles held

### Future: admin endpoint for role assignment

Not in MVP scope, but plan the data model for it:

```
PATCH /ats/admin/users/:userId/roles
Body: { "atsRoles": ["CANDIDATE", "RECRUITER"] }
Requires: a future ADMIN role or direct DB access
```

---

## Frontend Auth

### Jotai store (`src/lib/store/ats-auth.ts`)

```typescript
import { atom } from "jotai";
import { atomWithStorage } from "jotai/utils";

interface AtsUser {
  id: string;
  email: string;
  fullName: string;
  profilePicture: string | null;
  atsRoles: ("CANDIDATE" | "RECRUITER" | "DESIGNER")[];
}

interface AtsAuthState {
  user: AtsUser | null;
  isAuthenticated: boolean;
  loading: boolean;
}

export const atsAuthAtom = atomWithStorage<AtsAuthState>("ats_auth", {
  user: null,
  isAuthenticated: false,
  loading: true,
});

export const atsUserAtom = atom((get) => get(atsAuthAtom).user);

export const atsRolesAtom = atom((get) => get(atsAuthAtom).user?.atsRoles ?? []);
```

### Auth hook (`src/lib/hooks/ats/use-ats-auth.ts`)

```typescript
export function useAtsAuth() {
  const [authState, setAuthState] = useAtom(atsAuthAtom);

  useEffect(() => {
    const hasCookie = document.cookie.includes("ats_session=");
    if (!hasCookie) {
      setAuthState({ user: null, isAuthenticated: false, loading: false });
      return;
    }
    AtsAuthService.me()
      .then((user) =>
        setAuthState({ user, isAuthenticated: true, loading: false })
      )
      .catch(() =>
        setAuthState({ user: null, isAuthenticated: false, loading: false })
      );
  }, []);

  const hasRole = (role: "CANDIDATE" | "RECRUITER" | "DESIGNER") =>
    authState.user?.atsRoles.includes(role) ?? false;

  const logout = async () => {
    await AtsAuthService.logout();
    setAuthState({ user: null, isAuthenticated: false, loading: false });
  };

  return { ...authState, hasRole, logout };
}
```

### Auth guard (`AtsAuthGuard.tsx`)

Wraps protected pages. Redirects to `/ats/auth/login` if not authenticated. Optionally checks for a required role.

```typescript
interface Props {
  children: React.ReactNode;
  requiredRole?: "CANDIDATE" | "RECRUITER" | "DESIGNER";
}

export function AtsAuthGuard({ children, requiredRole }: Props) {
  const { isAuthenticated, loading, hasRole } = useAtsAuth();
  const router = useRouter();

  if (loading) return <LoadingSpinner />;

  if (!isAuthenticated) {
    router.replace("/ats/auth/login");
    return null;
  }

  if (requiredRole && !hasRole(requiredRole)) {
    router.replace("/ats/candidate/dashboard");
    return null;
  }

  return <>{children}</>;
}
```

### Auth service (`src/lib/services/ats/ats-auth.service.ts`)

```typescript
const ATS_BASE = "/api/v3/pdgms/ats";

export const AtsAuthService = {
  me: () => fetch(`${ATS_BASE}/auth/me`, { credentials: "include" })
    .then((res) => {
      if (!res.ok) throw new Error("Not authenticated");
      return res.json().then((data) => data.response);
    }),

  logout: () => fetch(`${ATS_BASE}/auth/logout`, {
    method: "POST",
    credentials: "include",
  }),

  getGoogleLoginUrl: () => `${ATS_BASE}/auth/google`,
};
```

---

## CRON Jobs

Both jobs registered in `src/pdgms/index.js` using the `cron` package (v4.3.4).

### Session cleanup (`src/pdgms/jobs/ats-session-cleanup.js`)

**Schedule:** Every 6 hours (`0 */6 * * *`)

```javascript
const cron = require("cron");

function startAtsSessionCleanup() {
  const job = new cron.CronJob("0 */6 * * *", async () => {
    const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);

    const expired = await prisma.atsUser.updateMany({
      where: {
        lastActiveAt: { lt: sevenDaysAgo },
        sessionToken: { not: null },
      },
      data: {
        sessionToken: null,
        sessionExpiresAt: null,
      },
    });

    if (expired.count > 0) {
      console.log(`[ATS] Cleaned ${expired.count} expired sessions`);
    }
  });

  job.start();
  return job;
}
```

Why 7-day inactivity (not 7-day absolute): a candidate actively working on a submission shouldn't be logged out mid-draft. Every authenticated request updates `lastActiveAt`. The session only expires if the user hasn't touched the ATS in 7 days.

### Deadline check (`src/pdgms/jobs/ats-deadline-check.js`)

**Schedule:** Every hour (`0 * * * *`)

```javascript
function startAtsDeadlineCheck() {
  const job = new cron.CronJob("0 * * * *", async () => {
    const now = new Date();

    // Find all DRAFT submissions past their deadline
    const expired = await prisma.submission.findMany({
      where: {
        status: "DRAFT",
        deadlineAt: { lt: now },
      },
      include: {
        application: {
          include: {
            opportunity: {
              include: {
                selectionProcess: { include: { rounds: true } },
              },
            },
          },
        },
        round: true,
      },
    });

    for (const submission of expired) {
      await prisma.$transaction(async (tx) => {
        // 1. Auto-submit the draft as-is
        await tx.submission.update({
          where: { id: submission.id },
          data: {
            status: "SUBMITTED",
            submittedAt: now,
          },
        });

        // 2. Find next round
        const rounds = submission.application.opportunity.selectionProcess.rounds;
        const currentOrder = submission.round.orderIndex;
        const nextRound = rounds.find((r) => r.orderIndex === currentOrder + 1);

        // 3. Advance application and create next submission
        if (nextRound) {
          await tx.application.update({
            where: { id: submission.applicationId },
            data: { currentRound: nextRound.orderIndex },
          });

          const deadlineAt = new Date(
            now.getTime() + nextRound.deadlineHours * 60 * 60 * 1000
          );

          await tx.submission.create({
            data: {
              applicationId: submission.applicationId,
              roundId: nextRound.id,
              candidateId: submission.candidateId,
              content: "",
              status: "DRAFT",
              unlockedAt: now,
              deadlineAt,
            },
          });
        }
        // If no next round: application stays ACTIVE at final round
      });
    }

    if (expired.length > 0) {
      console.log(`[ATS] Auto-submitted ${expired.length} expired drafts`);
    }
  });

  job.start();
  return job;
}
```

### Registration

```javascript
// src/pdgms/index.js (add to existing startup)
const { startAtsSessionCleanup } = require("./jobs/ats-session-cleanup");
const { startAtsDeadlineCheck } = require("./jobs/ats-deadline-check");

// Inside the startup function:
startAtsSessionCleanup();
startAtsDeadlineCheck();
```

---

## Public routes (no auth)

These routes skip `ensureAtsAuth`:

| Route | Purpose |
|-------|---------|
| `GET /ats/opportunities/public` | Browse live opportunities (no login required) |
| `GET /ats/opportunities/:id/public` | Opportunity detail (no login required) |
| `POST /ats/auth/google` | Initiate OAuth |
| `GET /ats/auth/google/callback` | OAuth callback |

Everything else requires `ensureAtsAuth` at minimum.

---

## Environment variables

| Variable | Used by | Example |
|----------|---------|---------|
| `GOOGLE_CLIENT_ID` | Passport Google strategy | `123456789.apps.googleusercontent.com` |
| `GOOGLE_CLIENT_SECRET` | Passport Google strategy | `GOCSPX-...` |
| `API_BASE_URL` | OAuth callback URL construction | `https://pdgms.dtgrowthteams.com` |
| `NODE_ENV` | Cookie secure flag | `production` |

No new env vars needed — these already exist for the main PDGMS Google login.
