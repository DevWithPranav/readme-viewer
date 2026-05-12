# Mock Data API Requirements

This document maps the current frontend mock data in `mulearndashboard` to backend APIs that should replace it. It includes:

- Proposed endpoints needed to remove mock data from the frontend.
- Request body and response body examples for each endpoint.
- Existing backend endpoints that can already cover part of the requirement.

Base API prefix used by this backend: `/api/v1/`

Standard response wrapper used in examples:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Success"]
  },
  "response": {}
}
```

## Table of Contents

1. [Existing Useful Endpoints](#existing-useful-endpoints)
2. [Learner Home Dashboard](#learner-home-dashboard)
3. [Mentor Home Dashboard](#mentor-home-dashboard)
4. [Company Home Dashboard](#company-home-dashboard)
5. [Campus Admin Dashboard](#campus-admin-dashboard)
6. [Company Profile Extended Data](#company-profile-extended-data)
7. [Implementation Priority](#implementation-priority)

## Existing Useful Endpoints

These endpoints already exist in the backend and should be reused where possible before adding new APIs.

| Frontend mock area | Existing endpoint | Method | Usefulness | Notes |
|---|---|---:|---|---|
| Learner profile/current user | `/api/v1/dashboard/user/info/` | `GET` | Current user information | Useful for name, muid, roles, basic profile data. |
| Learner rank | `/api/v1/dashboard/profile/rank/<muid>/` | `GET` | User rank | Can power learner rank if response shape has rank. |
| Learner levels | `/api/v1/dashboard/profile/get-user-levels/` | `GET` | Current user's level data | Can help derive current learner level. |
| Learner levels by muid | `/api/v1/dashboard/profile/get-user-levels/<muid>/` | `GET` | Public/user level data | Useful for profile context. |
| Learner karma feed | `/api/v1/dashboard/profile/karma-feed/` | `GET` | User karma activity feed | Can be aggregated for weekly karma if needed. |
| Learner user circles | `/api/v1/dashboard/learningcircle/user-circles/` | `GET` | Current user's joined circles | Useful for active circle count. |
| Learning circle meetings | `/api/v1/dashboard/learningcircle/meeting/list/` | `GET` | Upcoming/current meetings | Useful for next meeting. |
| Public learning circle meetings | `/api/v1/dashboard/learningcircle/meeting/list-public/` | `GET` | Public meeting list | Can support quick browse. |
| Global student leaderboard | `/api/v1/leaderboard/students/` | `GET` | Learner leaderboard | Useful for rank/list views. |
| Mentor overview | `/api/v1/dashboard/mentor/overview/` | `GET` | Mentor user, persona, stats | Already covers core mentor stat values, but not deltas/ratings/completion rate. |
| Mentor sessions | `/api/v1/dashboard/mentor/sessions/` | `GET`, `POST` | Mentor session list/create | Can replace upcoming mentoring sessions. |
| Mentor session detail | `/api/v1/dashboard/mentor/sessions/<session_id>/` | `PATCH` | Update session status/attendance | Useful for completing/cancelling sessions. |
| Mentor mentees | `/api/v1/dashboard/mentor/mentees/` | `GET` | Mentee progress list | Can replace mentee progress cards with minor response additions. |
| Mentor availability | `/api/v1/dashboard/mentor/availability/` | `GET`, `POST` | Availability slots | Can replace availability data. |
| Mentor availability delete | `/api/v1/dashboard/mentor/availability/<slot_id>/` | `DELETE` | Remove slot | Useful for availability management. |
| Campus details | `/api/v1/dashboard/campus/campus-details/` | `GET` | Campus members, karma, rank | Useful for campus stat cards. |
| Campus student levels | `/api/v1/dashboard/campus/student-level/` | `GET` | Level distribution | Useful for funnel/level sections. |
| Campus weekly karma | `/api/v1/dashboard/campus/weekly-karma/` | `GET` | Last 7 days karma | Useful for campus trend metrics. |
| Campus IG chapters | `/api/v1/dashboard/campus/ig-chapters/` | `GET`, `POST` | Campus IG chapters | Useful for circle/IG health if enriched. |
| Campus events | `/api/v1/dashboard/campus/events/` | `GET` | Campus events | Useful for recent activity if event activity is enough. |
| Campus event distribution | `/api/v1/dashboard/campus/events/distribution/` | `GET` | Event tag distribution | Useful for campus engagement views. |
| Company profile | `/api/v1/dashboard/company/profile/` | `GET`, `POST`, `PATCH`, `DELETE` | Own company profile | Covers base profile only. Needs extended fields. |
| Public company profile | `/api/v1/dashboard/company/profile/public/<slug>/` | `GET` | Public company profile | Covers base public company profile only. Needs extended fields and jobs. |
| Company jobs | `/api/v1/dashboard/company/jobs/` | `GET` | Own company job list | Can replace part of company profile jobs and dashboard jobs count. |
| Company job details | `/api/v1/dashboard/company/jobs/<job_id>/details/` | `GET` | Job details with rules | Useful for profile/jobs pages. |
| Company job applications | `/api/v1/dashboard/company/jobs/<job_id>/applications/` | `GET` | Applicants for a job | Can power applications count per job. |
| Company learner discovery | `/api/v1/dashboard/company/learners/` | `GET` | Public learner discovery | Useful for talent pool count and filters. |
| Learner applications | `/api/v1/dashboard/company/applications/` | `GET` | Current learner's applications | Useful for learner-side job application status. |

## Learner Home Dashboard

Frontend mock files:

- `src/features/home/constants/mock-stats.ts`
- Exports: `MOCK_STATS`, `MOCK_NEXT_MEETING`, `MOCK_QUICK_ACTION_COUNTS`

### Proposed Endpoint: Learner Dashboard Summary

`GET /api/v1/dashboard/home/learner/summary/`

Purpose:

- Replace learner home stats, next meeting, and quick action counts.
- Aggregate existing profile, rank, learning circle, meeting, job, and leaderboard data into one dashboard response.

Authentication:

- JWT required.
- Current authenticated learner.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `timezone` | string | No | Client timezone for meeting display, default `Asia/Kolkata`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Learner dashboard summary fetched successfully"]
  },
  "response": {
    "stats": {
      "weekly_karma": 480,
      "total_karma": 12450,
      "rank": 42,
      "active_circles": 3,
      "streak_days": 6
    },
    "next_meeting": {
      "id": "meeting-uuid",
      "circle_id": "circle-uuid",
      "circle_name": "Web Development Circle",
      "title": "React State Management",
      "starts_at": "2026-05-14T13:30:00+05:30",
      "duration_minutes": 60,
      "meeting_link": "https://meet.example.com/react-state",
      "rsvp_status": "joined"
    },
    "quick_action_counts": {
      "circles": 3,
      "leaderboard_rank": 42,
      "job_openings": 18
    }
  }
}
```

Backend data sources:

- User info: `/api/v1/dashboard/user/info/`
- Rank: `/api/v1/dashboard/profile/rank/<muid>/`
- Karma feed: `/api/v1/dashboard/profile/karma-feed/`
- User circles: `/api/v1/dashboard/learningcircle/user-circles/`
- Meetings: `/api/v1/dashboard/learningcircle/meeting/list/`
- Public jobs: existing code has `PublicJobsListAPIView`, but it is not currently wired in `api/dashboard/company/jobs/urls.py`. Add a URL if the frontend needs unauthenticated/public job counts.

### Optional Endpoint: Learner Streak

`GET /api/v1/dashboard/home/learner/streak/`

Purpose:

- Calculate streak independently if it is too expensive to include in the dashboard summary.

Request body:

None.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Learner streak fetched successfully"]
  },
  "response": {
    "streak_days": 6,
    "last_activity_at": "2026-05-12T10:30:00+05:30",
    "activity_dates": [
      "2026-05-07",
      "2026-05-08",
      "2026-05-09",
      "2026-05-10",
      "2026-05-11",
      "2026-05-12"
    ]
  }
}
```

## Mentor Home Dashboard

Frontend mock file:

- `src/features/home/constants/mock-mentor.ts`
- Exports: `MOCK_MENTOR_STAT_DELTAS`, `MOCK_UPCOMING_SESSIONS`, `MOCK_SESSION_REQUESTS`, `MOCK_MENTEE_PROGRESS`, `MOCK_EXPERTISE_TAGS`, `MOCK_MENTOR_NEXT_SESSION`

### Existing Endpoint To Reuse: Mentor Overview

`GET /api/v1/dashboard/mentor/overview/`

Already returns:

- Mentor user details.
- Mentor profile and expertise.
- Active persona IG.
- Authorized IGs.
- Stats: total mentees, sessions conducted, pending task approvals, volunteer hours.

Current useful response shape:

```json
{
  "response": {
    "user": {
      "full_name": "Asha Nair",
      "muid": "asha@mulearn",
      "profile_pic": "https://..."
    },
    "mentor_profile": {
      "about": "Frontend mentor",
      "expertise": ["React", "TypeScript"],
      "reason": "I like mentoring learners",
      "volunteer_hours": 42,
      "mentor_tier": "NORMAL",
      "is_verified": true
    },
    "active_persona": {
      "active_persona": "mentor",
      "active_role_link_id": "role-link-uuid",
      "active_ig_id": "ig-uuid",
      "ig_name": "Web Development",
      "is_verified": true
    },
    "authorized_igs": [],
    "stats": {
      "total_mentees": 18,
      "sessions_conducted": 12,
      "pending_task_approvals": 5,
      "volunteer_hours": 42
    }
  }
}
```

Gap:

- Does not include dashboard deltas, average rating, completion rate, next session, or session requests.

### Proposed Endpoint: Mentor Dashboard Summary

`GET /api/v1/dashboard/mentor/home-summary/`

Purpose:

- Replace all mentor dashboard mock exports in one API call.
- Can internally reuse `MentorOverviewView`, session tables, availability slots, and mentee data.

Authentication:

- JWT required.
- `Mentor` role in active persona IG.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `ig_id` | string | No | Override active IG if the mentor has access. |
| `timezone` | string | No | Client timezone, default `Asia/Kolkata`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Mentor dashboard summary fetched successfully"]
  },
  "response": {
    "next_session": {
      "id": "session-uuid",
      "title": "Portfolio Review",
      "mentee_name": "Rahul Krishna",
      "mentee_muid": "rahul@mulearn",
      "starts_at": "2026-05-13T18:00:00+05:30",
      "mode": "ONLINE",
      "meeting_link": "https://meet.example.com/session"
    },
    "stat_cards": [
      {
        "key": "active_mentees",
        "label": "Active mentees",
        "value": 18,
        "delta": 4,
        "delta_type": "increase",
        "period": "last_30_days"
      },
      {
        "key": "hours_mentored",
        "label": "Hours mentored",
        "value": 42,
        "delta": 6,
        "delta_type": "increase",
        "period": "last_30_days"
      },
      {
        "key": "avg_rating",
        "label": "Avg rating",
        "value": 4.8,
        "delta": 0.2,
        "delta_type": "increase",
        "period": "last_30_days"
      },
      {
        "key": "completion_rate",
        "label": "Completion rate",
        "value": 92,
        "unit": "percent",
        "delta": 3,
        "delta_type": "increase",
        "period": "last_30_days"
      }
    ],
    "upcoming_sessions": [
      {
        "id": "session-uuid",
        "title": "Portfolio Review",
        "mentee": {
          "id": "user-uuid",
          "full_name": "Rahul Krishna",
          "muid": "rahul@mulearn",
          "profile_pic": "https://..."
        },
        "topic": "Frontend portfolio",
        "starts_at": "2026-05-13T18:00:00+05:30",
        "ends_at": "2026-05-13T19:00:00+05:30",
        "mode": "ONLINE",
        "status": "SCHEDULED"
      }
    ],
    "session_requests": [
      {
        "id": "request-uuid",
        "mentee": {
          "id": "user-uuid",
          "full_name": "Meera S",
          "muid": "meera@mulearn",
          "profile_pic": null
        },
        "topic": "Backend project review",
        "requested_at": "2026-05-12T09:15:00+05:30",
        "preferred_slots": [
          {
            "starts_at": "2026-05-15T17:00:00+05:30",
            "ends_at": "2026-05-15T18:00:00+05:30"
          }
        ],
        "status": "PENDING"
      }
    ],
    "mentee_progress": [
      {
        "user_id": "user-uuid",
        "full_name": "Rahul Krishna",
        "muid": "rahul@mulearn",
        "profile_pic": "https://...",
        "ig_karma": 760,
        "target_karma": 1000,
        "progress_percent": 76,
        "ig_level": "Level 2",
        "session_count": 4,
        "last_session_at": "2026-05-08T18:00:00+05:30"
      }
    ],
    "expertise_tags": ["React", "TypeScript", "Career Guidance"]
  }
}
```

### Proposed Endpoint: Mentor Session Request Action

`PATCH /api/v1/dashboard/mentor/session-requests/<request_id>/`

Purpose:

- Accept or decline pending session requests shown on `SessionRequestsCard`.

Request body:

```json
{
  "action": "accept",
  "starts_at": "2026-05-15T17:00:00+05:30",
  "ends_at": "2026-05-15T18:00:00+05:30",
  "meeting_link": "https://meet.example.com/session"
}
```

For decline:

```json
{
  "action": "decline",
  "decline_reason": "Not available at the requested time"
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Session request accepted successfully"]
  },
  "response": {
    "request_id": "request-uuid",
    "status": "ACCEPTED",
    "session_id": "session-uuid"
  }
}
```

## Company Home Dashboard

Frontend mock file:

- `src/features/home/constants/mock-company.ts`
- Exports: `MOCK_COMPANY_QUICK_STATS`, `MOCK_COMPANY_STAT_CARDS`, `MOCK_LEVEL_DISTRIBUTION`, `MOCK_TOP_INTEREST_GROUPS`, `MOCK_VERIFIED_COMPANY_NAME`, `MOCK_TOTAL_LEARNERS`

### Existing Endpoints To Reuse

- Own company profile: `GET /api/v1/dashboard/company/profile/`
- Company jobs: `GET /api/v1/dashboard/company/jobs/`
- Company learners: `GET /api/v1/dashboard/company/learners/`
- Job applications for each job: `GET /api/v1/dashboard/company/jobs/<job_id>/applications/`

Gap:

- No single company dashboard analytics endpoint.
- No total views field for company jobs/profile.
- No hired count aggregation endpoint.
- No talent pool level distribution/top interest group aggregate endpoint.

### Proposed Endpoint: Company Dashboard Summary

`GET /api/v1/dashboard/company/home-summary/`

Purpose:

- Replace company dashboard mock data with real company, job, application, and learner discovery aggregates.

Authentication:

- JWT required.
- Company role and active company profile required.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `90d`, or `all`. Default `30d`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company dashboard summary fetched successfully"]
  },
  "response": {
    "company": {
      "id": "company-uuid",
      "name": "Example Technologies",
      "slug": "example-technologies",
      "status": "active",
      "logo": "https://..."
    },
    "quick_stats": {
      "jobs_posted": 12,
      "total_views": 3480,
      "applications": 216,
      "hired": 9
    },
    "stat_cards": [
      {
        "key": "jobs_posted",
        "label": "Jobs posted",
        "value": 12,
        "delta": 2,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "total_views",
        "label": "Total views",
        "value": 3480,
        "delta": 410,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "applications",
        "label": "Applications",
        "value": 216,
        "delta": 34,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "hired",
        "label": "Hired",
        "value": 9,
        "delta": 1,
        "delta_type": "increase",
        "period": "30d"
      }
    ],
    "talent_pool": {
      "total_learners": 18420,
      "level_distribution": [
        {
          "level": "L1",
          "count": 6420,
          "percentage": 34.85,
          "color": "#3B82F6"
        },
        {
          "level": "L2",
          "count": 5100,
          "percentage": 27.69,
          "color": "#10B981"
        },
        {
          "level": "L3",
          "count": 3810,
          "percentage": 20.68,
          "color": "#F59E0B"
        },
        {
          "level": "L4",
          "count": 2120,
          "percentage": 11.51,
          "color": "#EF4444"
        },
        {
          "level": "L5+",
          "count": 970,
          "percentage": 5.27,
          "color": "#8B5CF6"
        }
      ],
      "top_interest_groups": [
        {
          "ig_id": "ig-uuid",
          "name": "Web Development",
          "learner_count": 4200,
          "total_karma": 982000
        }
      ]
    }
  }
}
```

### Proposed Endpoint: Company Talent Pool Analytics

`GET /api/v1/dashboard/company/talent-pool/analytics/`

Purpose:

- Provide a reusable analytics endpoint for talent distribution, independent from dashboard summary.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `ig_ids` | string | No | Comma-separated IG UUIDs. |
| `level_order_min` | integer | No | Minimum learner level order. |
| `karma_min` | integer | No | Minimum karma. |
| `karma_max` | integer | No | Maximum karma. |
| `interested_in_work` | boolean | No | Filter learners interested in work. |
| `interested_in_gig_work` | boolean | No | Filter learners interested in gig work. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Talent pool analytics fetched successfully"]
  },
  "response": {
    "total_learners": 18420,
    "level_distribution": [
      {
        "level_id": "level-uuid",
        "level_name": "Level 1",
        "level_order": 1,
        "count": 6420
      }
    ],
    "top_interest_groups": [
      {
        "ig_id": "ig-uuid",
        "name": "Web Development",
        "learner_count": 4200,
        "total_karma": 982000
      }
    ]
  }
}
```

## Campus Admin Dashboard

Frontend mock file:

- `src/features/home/constants/mock-campus.ts`
- Exports: `MOCK_CAMPUS_FUNNEL`, `MOCK_CAMPUS_FUNNEL_MAX`, `MOCK_CIRCLE_HEALTH`, `MOCK_RECENT_ACTIVITY`, `MOCK_CAMPUS_STAT_DELTAS`

### Existing Endpoints To Reuse

- Campus details: `GET /api/v1/dashboard/campus/campus-details/`
- Student level distribution: `GET /api/v1/dashboard/campus/student-level/`
- Weekly karma: `GET /api/v1/dashboard/campus/weekly-karma/`
- Campus IG chapters: `GET /api/v1/dashboard/campus/ig-chapters/`
- Campus events: `GET /api/v1/dashboard/campus/events/`
- Campus event distribution: `GET /api/v1/dashboard/campus/events/distribution/`

Gap:

- No member funnel endpoint with registered/onboarded/active/L2+/circle lead stages.
- No circle health endpoint with sessions per month and active/slow/inactive status.
- No combined recent activity feed.
- Existing campus stat endpoints do not include deltas in the card format the frontend expects.

### Proposed Endpoint: Campus Dashboard Summary

`GET /api/v1/dashboard/campus/home-summary/`

Purpose:

- Replace all campus dashboard mock data in one API response.

Authentication:

- JWT required.
- Campus Lead or Lead Enabler.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `90d`, or `all`. Default `30d`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Campus dashboard summary fetched successfully"]
  },
  "response": {
    "campus": {
      "org_id": "org-uuid",
      "college_name": "Example College of Engineering",
      "campus_code": "ECE001",
      "campus_zone": "South Zone"
    },
    "stat_cards": [
      {
        "key": "active_members",
        "label": "Active members",
        "value": 180,
        "delta": 12,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "total_karma",
        "label": "Karma",
        "value": 45200,
        "delta": 4800,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "active_circles",
        "label": "Circles",
        "value": 7,
        "delta": 1,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "rank",
        "label": "Campus rank",
        "value": 5,
        "delta": 2,
        "delta_type": "increase",
        "period": "30d"
      }
    ],
    "member_funnel": {
      "max": 312,
      "stages": [
        {
          "key": "registered",
          "label": "Registered",
          "count": 312,
          "percentage": 100
        },
        {
          "key": "onboarded",
          "label": "Onboarded",
          "count": 240,
          "percentage": 76.92
        },
        {
          "key": "active",
          "label": "Active",
          "count": 180,
          "percentage": 57.69
        },
        {
          "key": "level_2_plus",
          "label": "Level 2+",
          "count": 95,
          "percentage": 30.45
        },
        {
          "key": "circle_lead",
          "label": "Circle Lead",
          "count": 14,
          "percentage": 4.49
        }
      ]
    },
    "circle_health": [
      {
        "circle_id": "circle-uuid",
        "circle_name": "Python Circle",
        "member_count": 42,
        "sessions_per_month": 4,
        "last_session_at": "2026-05-08T17:30:00+05:30",
        "status": "active"
      }
    ],
    "recent_activity": [
      {
        "id": "activity-uuid",
        "type": "level_up",
        "title": "Anjana reached Level 2",
        "description": "Anjana earned 320 karma in Web Development",
        "created_at": "2026-05-12T10:10:00+05:30",
        "actor": {
          "id": "user-uuid",
          "full_name": "Anjana S",
          "muid": "anjana@mulearn"
        },
        "metadata": {
          "level": "Level 2",
          "karma": 320
        }
      }
    ]
  }
}
```

### Proposed Endpoint: Campus Member Funnel

`GET /api/v1/dashboard/campus/member-funnel/`

Purpose:

- Reusable endpoint for funnel-only views and reports.

Request body:

None.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Campus member funnel fetched successfully"]
  },
  "response": {
    "max": 312,
    "stages": [
      {
        "key": "registered",
        "label": "Registered",
        "count": 312,
        "percentage": 100
      },
      {
        "key": "onboarded",
        "label": "Onboarded",
        "count": 240,
        "percentage": 76.92
      },
      {
        "key": "active",
        "label": "Active",
        "count": 180,
        "percentage": 57.69
      },
      {
        "key": "level_2_plus",
        "label": "Level 2+",
        "count": 95,
        "percentage": 30.45
      },
      {
        "key": "circle_lead",
        "label": "Circle Lead",
        "count": 14,
        "percentage": 4.49
      }
    ]
  }
}
```

### Proposed Endpoint: Campus Circle Health

`GET /api/v1/dashboard/campus/circle-health/`

Purpose:

- Replace `MOCK_CIRCLE_HEALTH`.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `period` | string | No | `30d`, `60d`, or `90d`. Default `30d`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Campus circle health fetched successfully"]
  },
  "response": {
    "data": [
      {
        "circle_id": "circle-uuid",
        "circle_name": "Python Circle",
        "ig_id": "ig-uuid",
        "ig_name": "Python",
        "member_count": 42,
        "sessions_per_month": 4,
        "last_session_at": "2026-05-08T17:30:00+05:30",
        "status": "active"
      }
    ]
  }
}
```

Suggested status rules:

| Status | Suggested rule |
|---|---|
| `active` | At least 2 sessions in the selected period and last session within 30 days. |
| `slow` | At least 1 session in the selected period but not active. |
| `inactive` | No sessions in the selected period. |

### Proposed Endpoint: Campus Recent Activity

`GET /api/v1/dashboard/campus/recent-activity/`

Purpose:

- Replace `MOCK_RECENT_ACTIVITY`.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `limit` | integer | No | Default `10`, max `50`. |
| `types` | string | No | Comma-separated activity types. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Campus recent activity fetched successfully"]
  },
  "response": {
    "data": [
      {
        "id": "activity-uuid",
        "type": "circle_created",
        "title": "AI Circle created",
        "description": "AI Circle was created by Meera S",
        "created_at": "2026-05-12T09:00:00+05:30",
        "actor": {
          "id": "user-uuid",
          "full_name": "Meera S",
          "muid": "meera@mulearn",
          "profile_pic": null
        },
        "metadata": {
          "circle_id": "circle-uuid",
          "circle_name": "AI Circle"
        }
      }
    ]
  }
}
```

## Company Profile Extended Data

Frontend mock file:

- `src/features/company-jobs/constants/mock-company-profile.ts`
- Exports: `MOCK_COMPANY_EXTENDED`, `MOCK_PUBLIC_JOBS`

Mock fields:

- `founded_year`
- `remote_policy`
- `culture_text`
- `tech_stack`
- `perks`
- `hire_count`
- `alumni_count`
- `avg_karma_of_hires`
- `campus_events_count`
- `testimonials`
- Public jobs fallback

### Existing Endpoints To Reuse

Own company profile:

`GET /api/v1/dashboard/company/profile/`

Public company profile:

`GET /api/v1/dashboard/company/profile/public/<slug>/`

Own company jobs:

`GET /api/v1/dashboard/company/jobs/`

Gap:

- The existing `Company` serializer only returns base company fields.
- Public profile does not return extended profile sections.
- No model/API currently documents gallery, testimonials, hiring stats, or public jobs by company slug.
- `PublicJobsListAPIView` exists in code but is not wired in `api/dashboard/company/jobs/urls.py`.

### Proposed Endpoint: Own Company Extended Profile

`GET /api/v1/dashboard/company/profile/extended/`

Purpose:

- Replace `MOCK_COMPANY_EXTENDED` for company users viewing their own profile.

Authentication:

- JWT required.
- Company role and company ownership required.

Request body:

None.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company extended profile fetched successfully"]
  },
  "response": {
    "company": {
      "id": "company-uuid",
      "name": "Example Technologies",
      "slug": "example-technologies",
      "logo": "https://...",
      "description": "We build learning technology.",
      "industry_sector": "Education Technology",
      "website_link": "https://example.com",
      "location": "Kochi, Kerala",
      "company_size": "51-200"
    },
    "extended": {
      "founded_year": 2019,
      "remote_policy": "hybrid",
      "culture_text": "We value curiosity, ownership, and learner-first engineering.",
      "tech_stack": ["Python", "Django", "React", "PostgreSQL"],
      "perks": ["Flexible work", "Learning budget", "Mentorship"],
      "hire_count": 24,
      "alumni_count": 8,
      "avg_karma_of_hires": 2450,
      "campus_events_count": 6
    },
    "testimonials": [
      {
        "id": "testimonial-uuid",
        "learner_name": "Rahul Krishna",
        "learner_muid": "rahul@mulearn",
        "role": "Frontend Intern",
        "quote": "muLearn helped me find my first internship.",
        "profile_pic": "https://...",
        "hired_at": "2026-01-10"
      }
    ],
    "gallery": [
      {
        "id": "gallery-uuid",
        "image_url": "https://...",
        "caption": "Campus hiring drive",
        "sort_order": 1
      }
    ]
  }
}
```

### Proposed Endpoint: Update Own Company Extended Profile

`PATCH /api/v1/dashboard/company/profile/extended/`

Purpose:

- Allow company users to update extended fields after onboarding.

Request body:

```json
{
  "founded_year": 2019,
  "remote_policy": "hybrid",
  "culture_text": "We value curiosity, ownership, and learner-first engineering.",
  "tech_stack": ["Python", "Django", "React", "PostgreSQL"],
  "perks": ["Flexible work", "Learning budget", "Mentorship"]
}
```

Field notes:

| Field | Type | Required | Notes |
|---|---|---:|---|
| `founded_year` | integer | No | Should be between `1800` and current year. |
| `remote_policy` | string | No | Suggested values: `remote`, `hybrid`, `in_office`, `flexible`. |
| `culture_text` | string | No | Suggested max length: `2000`. |
| `tech_stack` | array of strings | No | Suggested max `30` items. |
| `perks` | array of strings | No | Suggested max `30` items. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company extended profile updated successfully"]
  },
  "response": {
    "company_id": "company-uuid",
    "updated_fields": [
      "founded_year",
      "remote_policy",
      "culture_text",
      "tech_stack",
      "perks"
    ]
  }
}
```

### Proposed Endpoint: Public Company Profile Extended

`GET /api/v1/dashboard/company/profile/public/<slug>/extended/`

Purpose:

- Replace `MOCK_COMPANY_EXTENDED` in public company profile view.
- Include public jobs so `MOCK_PUBLIC_JOBS` can be removed.

Authentication:

- Public, no JWT required.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `include_jobs` | boolean | No | Default `true`. |
| `jobs_limit` | integer | No | Default `6`, max `20`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Public company extended profile fetched successfully"]
  },
  "response": {
    "company": {
      "id": "company-uuid",
      "name": "Example Technologies",
      "slug": "example-technologies",
      "logo": "https://...",
      "description": "We build learning technology.",
      "industry_sector": "Education Technology",
      "website_link": "https://example.com",
      "location": "Kochi, Kerala"
    },
    "extended": {
      "founded_year": 2019,
      "remote_policy": "hybrid",
      "culture_text": "We value curiosity, ownership, and learner-first engineering.",
      "tech_stack": ["Python", "Django", "React", "PostgreSQL"],
      "perks": ["Flexible work", "Learning budget", "Mentorship"],
      "hire_count": 24,
      "alumni_count": 8,
      "avg_karma_of_hires": 2450,
      "campus_events_count": 6
    },
    "testimonials": [
      {
        "id": "testimonial-uuid",
        "learner_name": "Rahul Krishna",
        "role": "Frontend Intern",
        "quote": "muLearn helped me find my first internship.",
        "profile_pic": "https://..."
      }
    ],
    "jobs": [
      {
        "id": "job-uuid",
        "title": "Frontend Intern",
        "job_type": "internship",
        "location": "Remote",
        "salary_range": "INR 15,000 - INR 25,000",
        "min_karma": 1000,
        "min_level": 2,
        "status": "Active",
        "created_at": "2026-05-10T10:00:00Z"
      }
    ]
  }
}
```

### Proposed Endpoint: Public Company Jobs By Slug

`GET /api/v1/dashboard/company/profile/public/<slug>/jobs/`

Purpose:

- Reusable public jobs endpoint for a company profile page.

Request body:

None.

Query params:

| Param | Type | Required | Description |
|---|---|---:|---|
| `pageIndex` | integer | No | Page number. |
| `perPage` | integer | No | Items per page. |
| `search` | string | No | Search by job title/location/type. |
| `sortBy` | string | No | Suggested values: `createdAt`, `title`. |

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Public company jobs fetched successfully"]
  },
  "response": {
    "company": {
      "id": "company-uuid",
      "name": "Example Technologies",
      "slug": "example-technologies"
    },
    "jobs": [
      {
        "id": "job-uuid",
        "title": "Frontend Intern",
        "job_type": "internship",
        "location": "Remote",
        "salary_range": "INR 15,000 - INR 25,000",
        "min_karma": 1000,
        "min_level": 2,
        "karma_reward": 500,
        "duration_value": 3,
        "duration_unit": "months",
        "stipend": "INR 20,000",
        "certificate_provided": true,
        "rules": [
          {
            "id": "rule-uuid",
            "rule_type": "skill",
            "rule_type_id": "skill-uuid",
            "rule_name": "React"
          }
        ],
        "created_at": "2026-05-10T10:00:00Z",
        "updated_at": "2026-05-10T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 12,
      "totalPages": 2,
      "isFirst": true,
      "isLast": false
    }
  }
}
```

## Implementation Priority

Recommended backend work order:

1. Add `GET /api/v1/dashboard/home/learner/summary/`.
   - Highest value because it removes three learner mock exports in one call.
   - Reuse existing profile, rank, circle, meeting, and job data.

2. Add `GET /api/v1/dashboard/mentor/home-summary/`.
   - Existing mentor APIs already cover much of the data.
   - Add only missing aggregates: deltas, average rating, completion rate, next session, requests.

3. Add `GET /api/v1/dashboard/company/home-summary/` and `GET /api/v1/dashboard/company/talent-pool/analytics/`.
   - Needed for `#job-stats` and `#talent-pool`.
   - May require job/profile view tracking and hire aggregation.

4. Add campus dashboard endpoints.
   - `GET /api/v1/dashboard/campus/home-summary/`
   - `GET /api/v1/dashboard/campus/member-funnel/`
   - `GET /api/v1/dashboard/campus/circle-health/`
   - `GET /api/v1/dashboard/campus/recent-activity/`
   - Covers `#funnel`, `#circle-health`, and `#activity-feed`.

5. Add company extended profile endpoints.
   - `GET/PATCH /api/v1/dashboard/company/profile/extended/`
   - `GET /api/v1/dashboard/company/profile/public/<slug>/extended/`
   - `GET /api/v1/dashboard/company/profile/public/<slug>/jobs/`
   - Covers `#company-profile-extended`, `#company-gallery-model`, `#company-testimonials-model`, and `#company-hire-tracking`.

## Backend Model Notes

Some proposed endpoints may need new models or fields:

| Requirement | Suggested backend storage |
|---|---|
| Company founded year, remote policy, culture text, tech stack, perks | Add fields to `Company` or create `CompanyExtendedProfile` one-to-one model. |
| Company gallery | Create `CompanyGalleryImage` with `company`, `image_url`, `caption`, `sort_order`, `is_active`. |
| Company testimonials | Create `CompanyTestimonial` with `company`, optional `learner`, `learner_name`, `role`, `quote`, `profile_pic`, `is_public`. |
| Hire tracking | Use `CompanyJobApplication.status = accepted` as base. Add `hired_at` if accepted does not mean hired. |
| Job/profile views | Add `CompanyJobView` and/or `CompanyProfileViewEvent`, or daily aggregate counters. |
| Mentor rating/completion rate | Add mentor session feedback/rating model, or derive completion rate from `MentorshipSession.status`. |
| Session requests | Add `MentorshipSessionRequest` if learner-request flow does not already exist. |
| Campus activity feed | Either aggregate from existing events/logs or create append-only `CampusActivity` model. |
