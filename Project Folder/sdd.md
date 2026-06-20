System Design Document (SDD)
EventPlan — School Organization Management System
Version: 1.0.0
Status: Active Development
Last Updated: May 2026

1. System Overview
EventPlan is a single-page application (SPA) built with a client-first architecture backed by Supabase as a Backend-as-a-Service (BaaS) provider. The system handles authentication, data persistence, authorization, and real-time-capable subscriptions through Supabase's managed infrastructure while the React frontend manages all UI rendering and local state.

Architecture Type
Client-BaaS Architecture — No custom server-side application layer. All logic lives in the React frontend and Supabase edge functions (if needed).

2. Architecture Diagram
┌─────────────────────────────────────────────────────────┐
│                    Client Layer (Browser)                │
│                                                         │
│   ┌─────────────┐   ┌─────────────┐   ┌────────────┐   │
│   │  React 19   │   │ TanStack    │   │ Auth       │   │
│   │  Components │──▶│ React Query │   │ Context    │   │
│   └──────┬──────┘   └─────────────┘   └────────────┘   │
│          │                                              │
│   ┌──────▼──────────────────────────────────────────┐  │
│   │              Service Layer                       │  │
│   │  authService │ recruitmentService │ eventService │  │
│   └──────────────────────┬───────────────────────────┘  │
└──────────────────────────┼──────────────────────────────┘
                           │ HTTPS / REST / WebSockets
┌──────────────────────────▼──────────────────────────────┐
│                    Supabase (BaaS)                       │
│                                                         │
│   ┌─────────────┐  ┌────────────┐  ┌────────────────┐  │
│   │  Auth       │  │ PostgreSQL │  │   Storage      │  │
│   │  (JWT)      │  │ (RLS)      │  │   (Files)      │  │
│   └─────────────┘  └────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────────┘
3. Technology Stack
3.1 Frontend
Technology	Version	Purpose
React	19.x	UI component framework
TypeScript	6.x	Type safety and code quality
Vite	8.x	Build tool and dev server
Tailwind CSS	4.x	Utility-first CSS framework
@tailwindcss/vite	4.x	Vite-native Tailwind integration
Lucide React	Latest	Premium icon library
TanStack React Query	5.x	Server-state management and caching
React Router DOM	6.x	Client-side routing
3.2 Backend (BaaS)
Technology	Version	Purpose
Supabase Auth	Managed	JWT-based authentication
Supabase Database	PostgreSQL 15	Primary relational data store
Supabase RLS	Native	Row-Level Security enforcement
Supabase Storage	Managed	File and document storage
3.3 Development Tooling
Tool	Purpose
ESLint	Static code analysis
Prettier	Code formatting
Vite HMR	Hot Module Replacement for fast iteration
4. Database Schema
4.1 Entity Relationship Diagram
profiles ──────────────────────────────────────────────────┐
   │                                                        │
   ├── recruitment_campaigns (created_by)                  │
   │       │                                               │
   │       └── applications (campaign_id, applicant_id) ──┘
   │
   ├── events (proposal_by, approved_by)
   │       │
   │       ├── event_committees (event_id, user_id)
   │       ├── event_registrations (event_id, user_id)
   │       ├── event_announcements (event_id, created_by)
   │       └── event_reports (event_id, submitted_by)
   │
   └── onboarding_templates (role_target)
4.2 Table Definitions
profiles
id            uuid  PK (references auth.users)
full_name     text  NOT NULL
email         text  NOT NULL
avatar_url    text  NULLABLE
role          enum  (super_admin | officer | committee_member | member | applicant)
created_at    timestamptz
updated_at    timestamptz
recruitment_campaigns
id            uuid  PK
title         text  NOT NULL
description   text  NOT NULL
type          enum  (member | officer)
status        enum  (draft | active | closed)
start_date    timestamptz NULLABLE
end_date      timestamptz NULLABLE
created_by    uuid  FK → profiles.id
created_at    timestamptz
applications
id                          uuid  PK
campaign_id                 uuid  FK → recruitment_campaigns.id
applicant_id                uuid  FK → profiles.id
status                      enum  (pending | screening | interview | approved | rejected)
answers                     jsonb (key-value of question → answer)
resume_url                  text  NULLABLE
feedback                    text  NULLABLE
onboarding_status           enum  (not_started | in_progress | completed)
onboarding_steps_completed  jsonb (array of completed step IDs)
created_at                  timestamptz
updated_at                  timestamptz
UNIQUE (campaign_id, applicant_id)
onboarding_templates
id           uuid  PK
title        text  NOT NULL
description  text  NULLABLE
role_target  enum  (super_admin | officer | committee_member | member | applicant)
steps        jsonb Array<{ id, title, description, required: boolean }>
created_at   timestamptz
events
id              uuid  PK
title           text  NOT NULL
description     text  NOT NULL
date            timestamptz NOT NULL
location        text  NOT NULL
status          enum  (proposal_draft | proposal_pending | approved | rejected | ongoing | completed | cancelled)
proposal_by     uuid  FK → profiles.id NULLABLE
approved_by     uuid  FK → profiles.id NULLABLE
approval_notes  text  NULLABLE
max_attendees   integer NULLABLE
created_at      timestamptz
updated_at      timestamptz
event_committees
id          uuid  PK
event_id    uuid  FK → events.id ON DELETE CASCADE
user_id     uuid  FK → profiles.id ON DELETE CASCADE
role        enum  (head | coordinator | member)
department  text  NOT NULL
created_at  timestamptz
UNIQUE (event_id, user_id)
event_registrations
id                uuid  PK
event_id          uuid  FK → events.id ON DELETE CASCADE
user_id           uuid  FK → profiles.id ON DELETE CASCADE
status            enum  (registered | cancelled)
attendance_status enum  (absent | present | excused)
registered_at     timestamptz
attended_at       timestamptz NULLABLE
UNIQUE (event_id, user_id)
event_announcements
id          uuid  PK
event_id    uuid  FK → events.id ON DELETE CASCADE
title       text  NOT NULL
content     text  NOT NULL
created_by  uuid  FK → profiles.id NULLABLE
created_at  timestamptz
event_reports
id                  uuid  PK
event_id            uuid  FK → events.id UNIQUE NOT NULL
summary             text  NOT NULL
budget_spent        numeric(12,2)
documentation_url   text  NULLABLE
submitted_by        uuid  FK → profiles.id NULLABLE
submitted_at        timestamptz
5. Row-Level Security (RLS) Policies
5.1 Policy Matrix
Table	SELECT	INSERT	UPDATE	DELETE
profiles	Public (all)	Trigger only	Own profile	Admin
recruitment_campaigns	Active/closed public; drafts = officer+	Officer+	Officer+	Officer+
applications	Own (applicant) OR Officer+	Own (applicant)	Officer+	Restricted
onboarding_templates	All	Officer+	Officer+	Officer+
events	Approved/ongoing public OR Officer+	Officer+	Officer+	Officer+
event_committees	All	Officer+ OR committee head	Officer+	Officer+
event_registrations	Own OR Officer+	Own	Own OR Officer+	Own
event_announcements	Member+ OR registered attendee	Officer+ OR committee head	Officer+	Officer+
event_reports	Member+	Officer+ OR committee head	Officer+	Officer+
5.2 Key RLS Rules (SQL Snippets)
-- Applicants can only see their own applications
CREATE POLICY "applicant_own_view" ON applications FOR SELECT
USING (auth.uid() = applicant_id);

-- Officers and admins can see all applications
CREATE POLICY "officer_full_access" ON applications FOR ALL
USING (EXISTS (
  SELECT 1 FROM profiles WHERE id = auth.uid() AND role IN ('officer', 'super_admin')
));

-- Events: members see only approved events
CREATE POLICY "public_approved_events" ON events FOR SELECT
USING (status IN ('approved', 'ongoing', 'completed') OR EXISTS (
  SELECT 1 FROM profiles WHERE id = auth.uid() AND role IN ('officer', 'super_admin')
));
6. Authentication Flow
User visits app
      │
      ▼
No session?──Yes──▶ AuthPage (Login / Register)
      │                    │
      │              Submit form
      │                    │
      │         Supabase Auth / Mock Auth
      │                    │
      │         Session returned + Profile fetched
      │                    │
      ▼                    ▼
AuthContext stores user profile ──▶ MainLayout renders
      │
      ▼
Role extracted from profile
      │
      ├── super_admin / officer ──▶ Full sidebar (Screening, Proposals)
      ├── committee_member ────────▶ Events + Committee tabs
      ├── member ─────────────────▶ Events + Profile tabs
      └── applicant ──────────────▶ Recruitment + Onboarding tabs
6.1 Mock/Offline Mode
When VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY are absent, the system automatically switches to Sandbox Mode:

Auth is simulated against localStorage profiles
All CRUD operations persist to localStorage under ep_mock_* keys
The resetMockData() utility (available in browser console) force-resets seed data
UI shows a "Sandbox Storage Active" badge in the sidebar
7. State Management Architecture
7.1 Global State (React Context)
Context	Manages
AuthContext	Authenticated user profile, loading state, signIn/signOut methods
ToastContext	Global notification queue (success, error, warning, info)
7.2 Server State (TanStack React Query)
React Query is configured via queryClient.ts for:

staleTime: 5 minutes — prevents unnecessary re-fetches
retry: 1 — fails fast on network errors
refetchOnWindowFocus: false — prevents background data refreshes
7.3 Local Component State
All form state, modal visibility, loading flags, and pagination live in useState at the component level to keep side effects localized.

8. Frontend Folder Structure
src/
├── assets/                  # Static brand assets
├── components/
│   ├── layouts/             # MainLayout, AuthLayout
│   └── ui/                  # Button, Card, Input, Modal, Badge, Toast
├── context/
│   ├── AuthContext.tsx       # Auth session management
│   └── ToastContext.tsx      # Global notification system
├── hooks/                   # useAuth, useToast (re-exports from context)
├── lib/
│   ├── supabaseClient.ts     # Supabase JS client initialization
│   └── queryClient.ts        # TanStack Query client config
├── pages/
│   ├── auth/                # AuthPage
│   ├── dashboard/           # DashboardOverview (role-adaptive)
│   ├── recruitment/         # RecruitmentPage, OnboardingPage
│   ├── admin/               # ScreeningPage (Kanban)
│   ├── events/              # EventsPage, EventProposalsPage, EventDashboard
│   └── profile/             # ProfilePage
├── services/
│   ├── authService.ts        # Auth CRUD (Supabase + mock)
│   ├── recruitmentService.ts # Campaign + Application CRUD
│   ├── eventService.ts       # Event + Committee + Registration CRUD
│   └── mockData.ts           # Default mock seed data + resetMockData()
├── types/
│   └── index.ts              # All TypeScript interfaces and enums
├── utils/                   # Date formatters, validators
├── App.tsx                  # Route declarations, provider composition
├── index.css                # Tailwind directives + custom utility classes
└── main.tsx                 # React DOM bootstrap
9. API Service Layer Design
Each service file follows the Adapter Pattern: it exposes identical async methods regardless of whether the system is in mock mode or live Supabase mode.

// Pattern used across all services
const myService = {
  isMockMode: authService.isMockMode,

  async getSomething(): Promise<Something[]> {
    if (this.isMockMode) {
      // Read from localStorage
    } else {
      // Query Supabase
    }
  }
}
This ensures that swapping to a live backend only requires setting environment variables — zero code changes needed in components.

10. Deployment Architecture
10.1 Production Stack
GitHub Repository
      │
      ▼
Vercel / Netlify (Static Hosting)
  • npm run build → dist/
  • Environment Variables: VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY
      │
      ▼
Supabase Cloud Project
  • Auth: Supabase Auth (JWT)
  • Database: PostgreSQL + RLS
  • Storage: Supabase Buckets (for docs)
10.2 Environment Variables
Variable	Required	Description
VITE_SUPABASE_URL	For live mode	Supabase project REST API URL
VITE_SUPABASE_ANON_KEY	For live mode	Supabase anonymous public key
Without these variables, the app runs in Sandbox Mode automatically.

11. Security Considerations
Risk	Mitigation
Unauthorized data access	RLS policies enforced at DB level for every table
Role escalation	Role changes only permitted via server-side triggers or officer-initiated flows
Exposed API keys	Supabase anon key is designed to be public; all sensitive access is RLS-controlled
XSS	React's JSX escaping prevents injection; no dangerouslySetInnerHTML usage
CSRF	Supabase JWT auth is stateless; no session cookies used
