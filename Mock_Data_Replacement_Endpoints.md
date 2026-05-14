# Mock Data Replacement Endpoints

This document lists the backend endpoints added to replace frontend mock data in `mulearndashboard`.

Base path: `/api/v1/`

Auth: all endpoints require JWT except public company profile/job endpoints.

## Learner Home

### Learner Dashboard Summary

Endpoint: `GET /api/v1/dashboard/home/learner/summary/`

Usage: replaces `MOCK_STATS`, `MOCK_NEXT_MEETING`, and `MOCK_QUICK_ACTION_COUNTS`.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `timezone` | string | No | Reserved for client timezone display. |

Request body: none.

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

### Learner Streak

Endpoint: `GET /api/v1/dashboard/home/learner/streak/`

Usage: standalone streak endpoint if the frontend needs to refresh streak data independently.

Request body: none.

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
    "last_activity_at": "2026-05-12",
    "activity_dates": ["2026-05-12", "2026-05-11"]
  }
}
```

## Mentor Home

### Mentor Dashboard Summary

Endpoint: `GET /api/v1/dashboard/mentor/overview/home-summary/`

Usage: replaces `MOCK_MENTOR_STAT_DELTAS`, `MOCK_UPCOMING_SESSIONS`, `MOCK_MENTEE_PROGRESS`, `MOCK_EXPERTISE_TAGS`, and `MOCK_MENTOR_NEXT_SESSION`.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `ig_id` | string | No | Overrides active mentor IG when allowed by persona context. |
| `timezone` | string | No | Reserved for client timezone display. |

Request body: none.

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
      "starts_at": "2026-05-15T17:00:00+05:30",
      "mode": "ONLINE",
      "meeting_link": "https://meet.example.com/session"
    },
    "stat_cards": [
      {
        "key": "active_mentees",
        "label": "Active mentees",
        "value": 18,
        "delta": 0,
        "delta_type": "neutral",
        "period": "30d"
      }
    ],
    "upcoming_sessions": [],
    "session_requests": [],
    "mentee_progress": [],
    "expertise_tags": ["React", "TypeScript"]
  }
}
```

Note: `session_requests` currently returns an empty list because no session request table/workflow exists yet.

## Company Home

### Company Dashboard Summary

Endpoint: `GET /api/v1/dashboard/company/home-summary/`

Usage: replaces `MOCK_COMPANY_QUICK_STATS`, `MOCK_COMPANY_STAT_CARDS`, `MOCK_LEVEL_DISTRIBUTION`, `MOCK_TOP_INTEREST_GROUPS`, `MOCK_VERIFIED_COMPANY_NAME`, and `MOCK_TOTAL_LEARNERS`.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `90d`, or `all`. Default `30d`. |
| `karma_min` | integer | No | Talent pool filter. |
| `karma_max` | integer | No | Talent pool filter. |
| `level_order_min` | integer | No | Talent pool filter. |
| `ig_ids` | string | No | Comma-separated IG UUIDs. |
| `interested_in_work` | boolean | No | Talent pool filter. |
| `interested_in_gig_work` | boolean | No | Talent pool filter. |

Request body: none.

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
      "total_views": 0,
      "applications": 216,
      "hired": 9
    },
    "stat_cards": [],
    "talent_pool": {
      "total_learners": 18420,
      "level_distribution": [],
      "top_interest_groups": []
    }
  }
}
```

### Company Talent Pool Analytics

Endpoint: `GET /api/v1/dashboard/company/talent-pool/analytics/`

Usage: standalone talent-pool analytics endpoint for filters and reports.

Query params: same talent pool filters as company dashboard summary.

Request body: none.

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
        "count": 6420,
        "percentage": 34.85
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

## Campus Admin Home

### Campus Dashboard Summary

Endpoint: `GET /api/v1/dashboard/campus/home-summary/`

Usage: replaces `MOCK_CAMPUS_FUNNEL`, `MOCK_CAMPUS_FUNNEL_MAX`, `MOCK_CIRCLE_HEALTH`, `MOCK_RECENT_ACTIVITY`, and `MOCK_CAMPUS_STAT_DELTAS`.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `60d`, or `90d`. Default `30d`. |

Request body: none.

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
      "college_name": "Example College",
      "campus_code": "EXC001",
      "campus_zone": "South Zone"
    },
    "stat_cards": [],
    "member_funnel": {
      "max": 312,
      "stages": []
    },
    "circle_health": [],
    "recent_activity": []
  }
}
```

### Campus Member Funnel

Endpoint: `GET /api/v1/dashboard/campus/member-funnel/`

Usage: standalone replacement for campus funnel mock data.

Request body: none.

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
      }
    ]
  }
}
```

### Campus Circle Health

Endpoint: `GET /api/v1/dashboard/campus/circle-health/`

Usage: standalone replacement for circle health mock data.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `30d`, `60d`, or `90d`. Default `30d`. |

Request body: none.

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

### Campus Recent Activity

Endpoint: `GET /api/v1/dashboard/campus/recent-activity/`

Usage: standalone replacement for recent campus activity mock data.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `limit` | integer | No | Default `10`, max `50`. |

Request body: none.

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
        "id": "circle-uuid",
        "type": "circle_created",
        "title": "Python Circle created",
        "description": "Python Circle was created",
        "created_at": "2026-05-12T09:00:00+05:30",
        "actor": {
          "id": "user-uuid",
          "full_name": "Meera S",
          "muid": "meera@mulearn",
          "profile_pic": null
        },
        "metadata": {
          "circle_id": "circle-uuid",
          "circle_name": "Python Circle"
        }
      }
    ]
  }
}
```

## Company Profile Extended

### Own Company Profile

Endpoint: `GET /api/v1/dashboard/company/profile/`

Usage: existing own company profile endpoint. The response now includes extended fields and derived stats, so the frontend does not need a separate extended-details API.

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company extended profile fetched successfully"]
  },
  "response": {
    "id": "company-uuid",
    "name": "Example Technologies",
    "slug": "example-technologies",
    "description": "We build learning technology.",
    "industry_sector": "Education Technology",
    "website_link": "https://example.com",
    "location": "Kochi, Kerala",
    "founded_year": 2019,
    "remote_policy": "hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Python", "Django", "React"],
    "perks": ["Flexible work"],
    "testimonials": [],
    "gallery": [],
    "hire_count": 24,
    "alumni_count": 8,
    "avg_karma_of_hires": 2450,
    "campus_events_count": 0
  }
}
```

### Update Own Company Profile

Endpoint: `PATCH /api/v1/dashboard/company/profile/`

Usage: existing own company profile update endpoint. It now accepts extended profile fields along with the existing company fields.

Request body:

```json
{
  "founded_year": 2019,
  "remote_policy": "hybrid",
  "culture_text": "We value curiosity and ownership.",
  "tech_stack": ["Python", "Django", "React"],
  "perks": ["Flexible work"],
  "testimonials": [
    {
      "learner_name": "Rahul Krishna",
      "role": "Frontend Intern",
      "quote": "muLearn helped me find my first internship."
    }
  ],
  "gallery": [
    {
      "image_url": "https://example.com/image.png",
      "caption": "Campus hiring drive",
      "sort_order": 1
    }
  ]
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company profile updated successfully"]
  },
  "response": {
    "id": "company-uuid",
    "name": "Example Technologies",
    "slug": "example-technologies",
    "founded_year": 2019,
    "remote_policy": "hybrid"
  }
}
```

### Public Company Profile

Endpoint: `GET /api/v1/dashboard/company/profile/public/<slug>/`

Usage: existing public company profile endpoint. The response now includes public extended fields and derived stats.

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Public company profile fetched successfully"]
  },
  "response": {
    "id": "company-uuid",
    "name": "Example Technologies",
    "slug": "example-technologies",
    "description": "We build learning technology.",
    "industry_sector": "Education Technology",
    "website_link": "https://example.com",
    "location": "Kochi, Kerala",
    "founded_year": 2019,
    "remote_policy": "hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Python", "Django", "React"],
    "perks": ["Flexible work"],
    "testimonials": [],
    "gallery": [],
    "hire_count": 24,
    "alumni_count": 8,
    "avg_karma_of_hires": 2450,
    "campus_events_count": 0
  }
}
```

### Public Company Jobs By Slug

Endpoint: `GET /api/v1/dashboard/company/profile/public/<slug>/jobs/`

Usage: replaces `MOCK_PUBLIC_JOBS`.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `pageIndex` | integer | No | Page number. |
| `perPage` | integer | No | Page size. |
| `search` | string | No | Searches title, location, and job type. |
| `sortBy` | string | No | `createdAt` or `title`. |

Request body: none.

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
    "jobs": [],
    "pagination": {
      "count": 0,
      "totalPages": 0,
      "isFirst": true,
      "isLast": true
    }
  }
}
```
