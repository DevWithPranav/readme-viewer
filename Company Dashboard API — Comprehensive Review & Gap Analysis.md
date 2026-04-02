# Company Dashboard API — Comprehensive Review & Gap Analysis

> **Date**: April 2, 2026  
> **Scope**: `api/dashboard/company/`, `api/dashboard/ig/`, `api/dashboard/events/` and related modules  
> **Baseline**: Company Dashboard Requirements Diagram (provided by user)

---

## 1. Requirements Summary (From Diagram)

The Company Dashboard specification defines **7 core feature areas**:

| # | Feature | Sub-features |
|---|---------|------------|
| **R1** | Post Gigs, Internships and Job Roles | Role, Proof-of-Work (PoW), Karma Reward |
| **R2** | Post Proof-of-Work Challenges | Standalone PoW challenge creation for learners |
| **R3** | Filter & View Learners | Filter by **Karma**, **Skill Badge**, **IG** |
| **R4** | Gig Engagement Analytics | Dashboard metrics on gig/job engagement |
| **R5** | Company Profile and Badges | Company profile page, engagement-based badges |
| **R6** | Event Request or IG Collaboration Form | Companies submit event/IG collaboration requests |

---

## 2. What Has Been Implemented

### 2.1 Job Posting System (Partial coverage of R1)

**Location**: [urls.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/urls.py) → [jobs_views.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/jobs_views.py)

| Endpoint | Method | Description | Status |
|----------|--------|-------------|--------|
| `company/jobs/` | GET | List paginated jobs for authenticated company user | ✅ Implemented |
| `company/jobs/create/` | POST | Create a new job posting | ✅ Implemented |
| `company/jobs/<job_id>/details/` | GET | Get single job details with rules | ✅ Implemented |
| `company/jobs/<job_id>/` | PATCH | Partial update of a job posting | ✅ Implemented |
| `company/jobs/<job_id>/` | DELETE | Soft-delete a job posting | ✅ Implemented |
| `company/jobs/<job_id>/rules/create/` | POST | Create eligibility rule (skill, IG, achievement) | ✅ Implemented |
| `company/jobs/<job_id>/rules/<rule_id>/` | PATCH | Update a specific job rule | ✅ Implemented |
| `company/jobs/<job_id>/rules/<rule_id>/delete/` | DELETE | Hard-delete a job rule | ✅ Implemented |

**Database Models**: [company.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/db/company.py)
- `Company` — company profile with status, industry, website, etc.
- `CompanyJob` — job postings with types: `Hybrid`, `Full-Time`, `Remote`, `Part-Time`, `Internship`, `Gig`
- `CompanyJobRule` — rule-based eligibility: `skill`, `interest_group`, `achievement`

> [!TIP]
> The job type choices include `Internship` and `Gig`, which partially satisfies R1's "Post Gigs, Internships and Job Roles" requirement.

**Strengths:**
- Clean `BaseCompanyJobView` with shared authorization logic
- N+1 query prevention via `get_optimized_jobs_with_rules()` — bulk-fetches related Skills, IGs, and Achievements
- Rule validation ensures referenced Skill/IG/Achievement actually exists in the DB
- Soft-delete pattern for jobs
- Proper pagination with search & sort support

---

### 2.2 Interest Group (IG) Management (Partial coverage of R6)

**Location**: [urls.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/ig/urls.py) → [dash_ig_view.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/ig/dash_ig_view.py)

| Endpoint | Method | Description | Status |
|----------|--------|-------------|--------|
| `ig/` | GET | List all IGs (paginated, searchable) | ✅ Implemented |
| `ig/` | POST | Create new IG (Admin only) | ✅ Implemented |
| `ig/<pk>/` | PUT | Full update of IG (Admin only) | ✅ Implemented |
| `ig/<pk>/` | DELETE | Delete an IG (Admin only) | ✅ Implemented |
| `ig/get/<pk>/` | GET | Get single IG details | ✅ Implemented |
| `ig/get/<pk>/` | PATCH | Partial update of IG (IG Lead or Admin) | ✅ Implemented |
| `ig/list/` | GET | Public IG listing (cached 10min) | ✅ Implemented |
| `ig/csv/` | GET | Download IG data as CSV | ✅ Implemented |
| `ig/request/` | GET | List IG creation requests (Company/Admin) | ✅ Implemented |
| `ig/request/` | POST | Submit IG creation request (Company/Admin) | ✅ Implemented |
| `ig/request/<pk>/` | PATCH | Update IG request status (Admin only) | ✅ Implemented |

**Strengths:**
- `InterestGroupRequestAPI` enables companies to **request new IG creation** — directly supports R6
- Company role (`RoleType.COMPANY`) is explicitly included in permission checks
- Status workflow: `requested → active / rejected / cancelled`
- IG JSON fields (prerequisites, career_opportunities, leads, mentors) properly serialized/deserialized

---

### 2.3 Event System (Partial coverage of R6)

**Location**: [urls.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/events/urls.py)

| Endpoint | Method | Description | Status |
|----------|--------|-------------|--------|
| `events/manage/` | GET/POST | List/create events for organizer | ✅ Implemented |
| `events/manage/<event_id>/` | GET/PUT/PATCH/DELETE | Full event management | ✅ Implemented |
| `events/manage/<event_id>/publish/` | POST | Submit event for approval | ✅ Implemented |
| `events/manage/<event_id>/co-owners/` | GET/POST | Manage event co-owners | ✅ Implemented |
| `events/manage/<event_id>/collaborators/` | GET/POST | Invite IG/campus/company collaborators | ✅ Implemented |
| `events/manage/<event_id>/collaborators/<id>/accept/` | POST | Accept collaboration invite | ✅ Implemented |
| `events/manage/<event_id>/collaborators/<id>/reject/` | POST | Reject collaboration invite | ✅ Implemented |
| `events/company/<company_id>/` | GET | List events by a specific company | ✅ Implemented |
| `events/admin/<event_id>/approve/` | POST | Admin event approval | ✅ Implemented |

**Strengths:**
- Full collaboration workflow with `COLLAB_IG`, `COLLAB_CAMPUS`, `COLLAB_CAMPUS_IG`, `COLLAB_COMPANY` entity types
- `RoleType.COMPANY` included in `MANAGEABLE_ROLES` for event creation
- Proper approval pipeline: `DRAFT → PENDING_... → PUBLISHED`
- Supports "Event Request or IG Collaboration Form" (R6) partially

---

## 3. Critical Gap Analysis

### ❌ Gap Summary Matrix

| Req | Feature | Implementation Status | Severity |
|-----|---------|-----------------------|----------|
| **R1** | Post **Gigs** | ⚠️ Partial — `job_type='Gig'` exists but no gig-specific workflows | **Medium** |
| **R1** | Post **Internships** | ⚠️ Partial — `job_type='Internship'` exists but no internship-specific fields | **Medium** |
| **R1** | **Role** specification | ⚠️ Partial — job rules can reference skills/IGs/achievements but no explicit "Role" field | **Medium** |
| **R1** | **Proof-of-Work (PoW)** linked to jobs | ❌ **Missing** — No mechanism to attach PoW challenges to job postings | **High** |
| **R1** | **Karma Reward** for jobs | ⚠️ Partial — `min_karma` is set as eligibility, not as a reward mechanism | **High** |
| **R2** | Post **Proof-of-Work Challenges** | ❌ **Missing** — No company-facing API to create/manage PoW challenges | **Critical** |
| **R3** | Filter & View Learners by **Karma** | ❌ **Missing** — No API for companies to browse/filter learners | **Critical** |
| **R3** | Filter & View Learners by **Skill Badge** | ❌ **Missing** — No skill-badge-based learner search | **Critical** |
| **R3** | Filter & View Learners by **IG** | ❌ **Missing** — No IG-based learner filtering for companies | **Critical** |
| **R4** | **Gig Engagement Analytics** | ❌ **Missing** — Zero analytics endpoints | **Critical** |
| **R5** | **Company Profile** management | ❌ **Missing** — No CRUD API for company profile data | **High** |
| **R5** | **Badges** based on engagement | ❌ **Missing** — No badge/gamification system for companies | **High** |
| **R6** | **Event Request** form | ⚠️ Partial — Event creation requires `COMPANY` role but no dedicated request form | **Medium** |
| **R6** | **IG Collaboration** form | ⚠️ Partial — IG request API exists; event collaboration API exists | **Low** |

---

## 4. Detailed Gap Descriptions

### 4.1 🔴 CRITICAL: Proof-of-Work (PoW) Challenge System (R2)

> [!CAUTION]
> This is the single largest missing feature. The entire PoW challenge concept has zero implementation.

**What's needed:**
- A new model `CompanyPoWChallenge` for companies to define challenges
- CRUD endpoints under `company/pow-challenges/`
- Fields: title, description, submission criteria, evaluation rubric, karma reward, deadline, associated skills/IGs
- Submission tracking: learners submit proof, companies review and award karma
- Integration with the existing Task system (tasks have PoW elements via `TaskType`)

**Impact**: Without PoW challenges, companies cannot assess learner competencies — a core value proposition of the platform.

---

### 4.2 🔴 CRITICAL: Filter & View Learners (R3)

> [!CAUTION]
> Companies currently have **no way to discover or search for learners** on the platform. This is a talent-discovery feature that is entirely absent.

**What's needed:**
- New API: `company/learners/` with GET supporting filters:
  - `?karma_min=` / `?karma_max=` — filter by karma range
  - `?skill_ids=` — filter by skill badges
  - `?ig_ids=` — filter by Interest Group membership
  - `?level=` — filter by learner level
  - `?location=` — geographic filter
- Pagination, sorting (by karma, level, activity)
- Individual learner profile view for companies (privacy-respecting)
- The existing `User` model has karma fields; `UserIgLink` tracks IG membership; skill badges exist via achievements

**Data sources already available:**
- `db.user.User` — has user profile data
- `UserIgLink` — IG membership (via `user_ig_link_ig`)
- `KarmaActivityLog` / user karma — available in the system
- `Skill` / `TaskSkillLink` — skill associations

---

### 4.3 🔴 CRITICAL: Gig Engagement Analytics (R4)

> [!WARNING]
> There are zero analytics endpoints in the entire company dashboard module.

**What's needed:**
- `company/analytics/` with sub-endpoints:
  - `company/analytics/overview/` — total jobs posted, active jobs, views, applications
  - `company/analytics/jobs/<job_id>/` — per-job metrics (views, applications, acceptance rate)
  - `company/analytics/trends/` — time-series data for engagement
  - `company/analytics/learner-match/` — how many learners match job criteria
- This requires tracking infrastructure:
  - Job view counts
  - Application/interest tracking (a `CompanyJobApplication` model)
  - Skill-match scoring

---

### 4.4 🔴 HIGH: Company Profile Management (R5)

**What exists:**
- The `Company` model has fields: `name`, `logo`, `description`, `industry_sector`, `website_link`, `email`, `slug`, `status`, `location`
- **But there is NO API to read/update these fields from the company dashboard**

**What's needed:**
- `company/profile/` — GET (view own profile), PUT/PATCH (update profile)
- `company/profile/public/<slug>/` — public-facing company profile
- Fields to manage: logo upload, description, industry, website, location
- Social links, portfolio/case studies
- Company verification status display

---

### 4.5 🔴 HIGH: Company Badges & Gamification (R5)

**What exists:**
- The `Achievement` model exists in the system (used for learner badges)
- Job rules can reference achievements

**What's needed:**
- New model: `CompanyBadge` or leverage existing `Achievement` for company-tier badges
- Badge criteria based on engagement metrics:
  - Number of jobs posted
  - Number of PoW challenges created
  - Number of learners hired/engaged
  - Event sponsorship count
- API: `company/badges/` — GET list of earned & available badges
- Badge display on company public profile

---

### 4.6 ⚠️ MEDIUM: Job Posting Enhancements (R1)

**Current gaps in the existing job system:**

1. **No distinction between Gig vs Internship vs Full-Time workflows:**
   - Gigs should have: duration, hourly rate, deliverables
   - Internships should have: stipend, duration, certificate provision
   - Currently all types share the same flat `CompanyJob` model

2. **No Karma Reward mechanism:**
   - `min_karma` is an input requirement, not a reward
   - Need: `karma_reward` field — karma awarded to learners who complete the gig/challenge
   - Integration with the karma system to actually credit learners

3. **No PoW linkage:**
   - Job postings cannot specify required PoW tasks
   - Need: `CompanyJobPoWLink` or a new `pow_challenge_id` FK on `CompanyJob`

4. **No application/interest tracking:**
   - Currently there is no `CompanyJobApplication` model
   - Learners cannot express interest or apply to jobs
   - Companies cannot shortlist or manage applicants

---

### 4.7 ⚠️ MEDIUM: Event Request Enhancement (R6)

**Current state:**
- Event creation is permitted for `RoleType.COMPANY` via `ManageEventListCreateAPI`
- IG collaboration is handled via `EventConnection` with `COLLAB_COMPANY` type
- IG request is handled via `InterestGroupRequestAPI`

**Gaps:**
- No dedicated "Event Request Form" for companies — they use the same event creation flow as other roles
- No company-specific event templates
- No direct IG collaboration request form outside the event context — companies should be able to propose IG collaboration without creating an event first

---

## 5. Code Quality Observations

### Issues Found

| Issue | Location | Severity |
|-------|----------|----------|
| `print()` statements used for debugging in production code | [jobs_views.py:264, 290, 292, etc.](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/jobs_views.py#L264-L292) | Medium |
| Bare `except Exception` catching all errors — hides bugs | [jobs_views.py:310, 459, 518](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/jobs_views.py#L310) | High |
| `timezone.now()` imported from `pytz` but used incorrectly in `DeleteJobRuleAPIView` (line 813) | [jobs_views.py:813](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/jobs_views.py#L813) | Low |
| `managed = False` on `CompanyJob` but `managed = True` on `CompanyJobRule` — inconsistent migration strategy | [company.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/db/company.py) | Medium |
| Commented-out code remains in serializers and views | Multiple files | Low |
| No input sanitization for job descriptions (XSS risk if rendered) | [jobs_views.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/company/jobs/jobs_views.py) | Medium |
| Missing `__init__.py` in `api/dashboard/company/` directory | Directory structure | Low |

---

## 6. Missing Features — Prioritized Backlog

### Priority 1 — Critical (Must-have for MVP)

| # | Feature | Description | Estimated New Files |
|---|---------|-------------|-------------------|
| 1 | **Learner Discovery API** | `company/learners/` — filter/search by karma, skill badge, IG | New view + serializer |
| 2 | **Company Profile API** | `company/profile/` — CRUD for company profile data | New view + serializer |
| 3 | **PoW Challenge System** | `company/pow-challenges/` — full CRUD + submission tracking | New model + view + serializer |
| 4 | **Gig Analytics API** | `company/analytics/` — engagement metrics dashboard data | New view + serializer |

### Priority 2 — High (Required for full functionality)

| # | Feature | Description |
|---|---------|-------------|
| 5 | **Job Application Tracking** | `CompanyJobApplication` model + APIs for learner applications |
| 6 | **Karma Reward on Jobs** | `karma_reward` field + integration with karma credit system |
| 7 | **Company Badge System** | Engagement-based badges for companies |
| 8 | **PoW → Job Linkage** | Connect PoW challenges to specific job postings |

### Priority 3 — Medium (Enhancement)

| # | Feature | Description |
|---|---------|-------------|
| 9 | **Gig-specific fields** | Duration, hourly rate, deliverables for gig type |
| 10 | **Internship-specific fields** | Stipend, duration, certificate for internship type |
| 11 | **Event Request Form** | Dedicated company event request workflow |
| 12 | **IG Standalone Collaboration** | Company-to-IG collaboration outside event context |

### Priority 4 — Low (Polish)

| # | Feature | Description |
|---|---------|-------------|
| 13 | **Company Public Profile** | Public-facing company page via slug |
| 14 | **Job Applicant Shortlisting** | Shortlist, interview stage tracking |
| 15 | **Notification Integration** | Email/webhook hooks for application status changes |

---

## 7. Coverage Heatmap

```
REQUIREMENT               COVERAGE
═══════════════════════════════════════════
R1: Post Jobs/Gigs        ██████░░░░  60%  — Job CRUD works, missing PoW/Karma reward
R2: PoW Challenges        ░░░░░░░░░░   0%  — COMPLETELY MISSING
R3: Filter Learners       ░░░░░░░░░░   0%  — COMPLETELY MISSING
R4: Gig Analytics         ░░░░░░░░░░   0%  — COMPLETELY MISSING
R5: Profile & Badges      █░░░░░░░░░  10%  — Model exists but no API
R6: Event/IG Collab       ██████░░░░  60%  — Event + IG request exist, needs polish
═══════════════════════════════════════════
OVERALL ESTIMATED:        ██░░░░░░░░  ~22%
```

---

## 8. Recommendations

> [!IMPORTANT]
> **Only ~22% of the Company Dashboard requirements are implemented.** Three out of six core features (R2, R3, R4) have zero implementation. The platform currently functions primarily as a job posting board for companies, rather than the full-featured company engagement dashboard described in the requirements.

### Immediate Next Steps

1. **Company Profile API** — Quickest win; model already exists, just needs views/serializers
2. **Learner Discovery API** — High-impact; data already exists in the system (User, Karma, IG links, Skills)
3. **PoW Challenge System** — Core platform differentiator; requires new model design
4. **Analytics Foundation** — Start with basic counts; add tracking infrastructure incrementally

### Architecture Consideration

> [!WARNING]
> The current `CompanyJob` model uses `managed = False` (assuming an external DB migration strategy) while `CompanyJobRule` doesn't. Any new models (PoW, Application, Analytics) need to follow a consistent migration strategy. Verify with the team whether to use `managed = True` or `False`.
