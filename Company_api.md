# Company Dashboard Comprehensive API Spec

Base path: `/api/v1/dashboard/company/`

Public tracking path: `/api/v1/public/`

Authentication: company-dashboard endpoints require `Authorization: Bearer <access_token>` and an active company profile unless explicitly marked public.

Common success envelope:

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

Common failure envelope:

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Validation failed"],
    "error_code": "VALIDATION_ERROR"
  },
  "response": {}
}
```

## Endpoint Summary

| Endpoint | Method | Usage Scenario |
|---|---:|---|
| `/api/v1/dashboard/company/home-summary/` | `GET` | Company sees jobs, applications, hires, profile status, and talent-pool snapshot. |
| `/api/v1/dashboard/company/profile/` | `GET` | Company views its own profile and verification status. |
| `/api/v1/dashboard/company/profile/` | `PATCH` | Company updates profile fields such as logo, description, tech stack, perks, and gallery. |
| `/api/v1/dashboard/company/jobs/` | `GET` | Company lists all posted jobs, gigs, internships, and opportunities. |
| `/api/v1/dashboard/company/jobs/create/` | `POST` | Company posts a job, internship, gig, or PoW-gated opportunity. |
| `/api/v1/dashboard/company/jobs/<job_id>/details/` | `GET` | Company opens one job with rules, linked task, and application summary. |
| `/api/v1/dashboard/company/jobs/<job_id>/` | `PATCH` | Company updates a job, closes it, or changes eligibility. |
| `/api/v1/dashboard/company/jobs/<job_id>/analytics/` | `GET` | Company reviews views, applications, shortlists, hires, and conversion rates. |
| `/api/v1/public/jobs/<job_id>/view/` | `POST` | Frontend records a public job view for analytics. |
| `/api/v1/dashboard/company/jobs/<job_id>/applications/` | `GET` | Company views applicants for one job. |
| `/api/v1/dashboard/company/jobs/<job_id>/applications/<app_id>/` | `PATCH` | Company changes application status through the hiring workflow. |
| `/api/v1/dashboard/company/learners/` | `GET` | Company discovers learners by karma, level, IG, achievement, skill, and work preference. |
| `/api/v1/dashboard/company/learners/<user_id>/shortlist/` | `POST` | Company saves a learner for follow-up or links them to a job. |
| `/api/v1/dashboard/company/shortlists/` | `GET` | Company views saved learners and follow-up status. |
| `/api/v1/dashboard/company/tasks/` | `GET` | Company lists its submitted PoW tasks and approval status. |
| `/api/v1/dashboard/company/tasks/submit/` | `POST` | Company submits a proof-of-work task for admin review. |
| `/api/v1/dashboard/company/badges/` | `GET` | Company views reputation badges earned on the platform. |
| `/api/v1/dashboard/company/engagement-score/` | `GET` | Company sees overall engagement score and improvement actions. |
| `/api/v1/dashboard/company/ig-collaboration-requests/` | `POST` | Company requests collaboration with an Interest Group. |
| `/api/v1/dashboard/company/events/request/` | `POST` | Company requests to host or co-host an event. |

## 1. Dashboard Home Summary

Endpoint: `GET /api/v1/dashboard/company/home-summary/`

Usage: Used on the first dashboard screen. The company gets a quick health snapshot: active jobs, applications, hires, task approvals, pending actions, and talent-pool highlights.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `90d`; default `30d`. |

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
      "name": "Acme Labs",
      "slug": "acme-labs",
      "status": "active",
      "logo": "https://cdn.example.com/acme-logo.png"
    },
    "stat_cards": [
      {
        "key": "jobs_posted",
        "label": "Jobs posted",
        "value": 8,
        "delta": 2,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "applications",
        "label": "Applications",
        "value": 126,
        "delta": 34,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "hired",
        "label": "Hired",
        "value": 7,
        "delta": 2,
        "delta_type": "increase",
        "period": "30d"
      },
      {
        "key": "pending_reviews",
        "label": "Pending reviews",
        "value": 14,
        "delta": 4,
        "delta_type": "neutral",
        "period": "30d"
      }
    ],
    "pending_actions": {
      "pending_task_reviews": 2,
      "unreviewed_applications": 14,
      "profile_completion_percentage": 82
    },
    "talent_pool": {
      "total_learners": 1240,
      "active_learners": 380,
      "top_interest_groups": [
        {
          "ig_id": "ig-uuid",
          "name": "Web Development",
          "learner_count": 420
        }
      ]
    }
  }
}
```

## 2. Company Profile

Endpoint: `GET /api/v1/dashboard/company/profile/`

Usage: Company views editable profile information, verification metadata, public page content, and employer-brand fields.

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company profile fetched successfully"]
  },
  "response": {
    "id": "company-uuid",
    "name": "Acme Labs",
    "slug": "acme-labs",
    "logo": "https://cdn.example.com/logo.png",
    "description": "A product engineering company.",
    "industry_sector": "Technology",
    "website_link": "https://acme.example",
    "email": "careers@acme.example",
    "location": "Kochi, Kerala",
    "status": "active",
    "company_size": "51-200",
    "linkedin_url": "https://linkedin.com/company/acme",
    "tech_stack": ["Python", "Django", "React"],
    "perks": ["Hybrid work", "Mentorship"],
    "testimonials": [
      {
        "name": "Riya Sharma",
        "role": "Backend Intern",
        "quote": "The PoW challenge helped me show real skills."
      }
    ],
    "gallery": [
      "https://cdn.example.com/acme-office.jpg"
    ],
    "verified_at": "2026-05-20T10:00:00+05:30",
    "rejection_reason": null
  }
}
```

Endpoint: `PATCH /api/v1/dashboard/company/profile/`

Usage: Company updates profile data shown to learners, partners, and admins.

Request body:

```json
{
  "description": "We build reliable fintech products.",
  "tech_stack": ["Python", "Django", "PostgreSQL", "React"],
  "perks": ["Flexible hours", "Learning budget"],
  "gallery": [
    "https://cdn.example.com/acme-team.jpg"
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
    "name": "Acme Labs",
    "description": "We build reliable fintech products.",
    "tech_stack": ["Python", "Django", "PostgreSQL", "React"],
    "updated_at": "2026-05-28T09:30:00+05:30"
  }
}
```

## 3. Job List

Endpoint: `GET /api/v1/dashboard/company/jobs/`

Usage: Company lists all opportunities it has posted, including active, draft, closed, and expired jobs.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `search` | string | No | Search by title, location, job type. |
| `status` | string | No | `Draft`, `Active`, `Closed`, `Expired`. |
| `job_type` | string | No | `Full-Time`, `Internship`, `Gig`, `Remote`, `Hybrid`, `Part-Time`. |
| `pageIndex` | integer | No | Page number. |
| `perPage` | integer | No | Items per page. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Jobs fetched successfully"]
  },
  "response": {
    "jobs": [
      {
        "id": "job-uuid",
        "title": "Backend Engineering Intern",
        "job_type": "Internship",
        "location": "Remote",
        "status": "Active",
        "salary_range": "15000-25000 INR",
        "min_karma": 1000,
        "min_level": 3,
        "requires_task_completion": true,
        "linked_task_info": {
          "id": "task-uuid",
          "hashtag": "#django-api",
          "title": "Build a REST API",
          "karma": 200
        },
        "application_count": 32,
        "created_at": "2026-05-10T10:00:00+05:30"
      }
    ],
    "pagination": {
      "count": 8,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false
    }
  }
}
```

## 4. Create Job, Internship, or Gig

Endpoint: `POST /api/v1/dashboard/company/jobs/create/`

Usage: Company posts a new hiring opportunity. It can optionally require a learner to complete a PoW task before applying.

Request body:

```json
{
  "title": "Backend Engineering Intern",
  "experience": "0-1 years",
  "job_description": "Build APIs, write tests, and document endpoints.",
  "job_type": "Internship",
  "location": "Remote",
  "salary_range": "15000-25000 INR",
  "min_karma": 1000,
  "min_level": 3,
  "duration_value": 3,
  "duration_unit": "months",
  "stipend": "20000 INR",
  "certificate_provided": true,
  "karma_reward": 500,
  "requires_task_completion": true,
  "linked_task_id": "task-uuid",
  "rules": [
    {
      "rule_type": "skill",
      "rule_type_id": "skill-uuid"
    },
    {
      "rule_type": "interest_group",
      "rule_type_id": "ig-uuid"
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
    "general": ["Job created successfully"]
  },
  "response": {
    "job_id": "job-uuid",
    "title": "Backend Engineering Intern",
    "status": "Active",
    "company_id": "company-uuid"
  }
}
```

## 5. Job Details

Endpoint: `GET /api/v1/dashboard/company/jobs/<job_id>/details/`

Usage: Company opens a single job to view eligibility rules, task gate, applicants summary, and current status.

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Job details fetched successfully"]
  },
  "response": {
    "id": "job-uuid",
    "title": "Backend Engineering Intern",
    "job_type": "Internship",
    "status": "Active",
    "requires_task_completion": true,
    "linked_task_info": {
      "id": "task-uuid",
      "title": "Build a REST API",
      "hashtag": "#django-api"
    },
    "rules": [
      {
        "id": "rule-uuid",
        "rule_type": "skill",
        "rule_type_id": "skill-uuid",
        "rule_name": "Django"
      }
    ],
    "application_summary": {
      "applied": 22,
      "shortlisted": 7,
      "accepted": 2,
      "rejected": 4,
      "withdrawn": 1
    }
  }
}
```

## 6. Update Job

Endpoint: `PATCH /api/v1/dashboard/company/jobs/<job_id>/`

Usage: Company updates an active job, closes applications, changes eligibility, or edits job details.

Request body:

```json
{
  "status": "Closed",
  "job_description": "Applications are closed. Shortlisted candidates will be contacted.",
  "min_karma": 1200
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Job updated successfully"]
  },
  "response": {
    "job_id": "job-uuid",
    "status": "Closed",
    "updated_at": "2026-05-28T09:45:00+05:30"
  }
}
```

## 7. Job Analytics

Endpoint: `GET /api/v1/dashboard/company/jobs/<job_id>/analytics/`

Usage: Company reviews the hiring funnel for one job: views, unique views, applications, shortlist count, hires, and conversion rates.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `7d`, `30d`, `90d`, `all`; default `30d`. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Job analytics fetched successfully"]
  },
  "response": {
    "job": {
      "id": "job-uuid",
      "title": "Backend Engineering Intern",
      "status": "Active"
    },
    "funnel": {
      "views": 320,
      "unique_views": 210,
      "applications": 45,
      "shortlisted": 12,
      "accepted": 3,
      "rejected": 8,
      "withdrawn": 2
    },
    "conversion": {
      "view_to_apply_rate": 14.06,
      "apply_to_shortlist_rate": 26.67,
      "apply_to_hire_rate": 6.67
    },
    "trend": [
      {
        "date": "2026-05-21",
        "views": 42,
        "applications": 5
      }
    ]
  }
}
```

## 8. Public Job View Tracking

Endpoint: `POST /api/v1/public/jobs/<job_id>/view/`

Usage: Called by the public job detail page when a learner opens the page. The backend should deduplicate repeated views from the same user/session within a short time window.

Authentication: optional. If authenticated, associate view with user. If anonymous, associate with session ID or fingerprint.

Request body:

```json
{
  "source": "public_jobs",
  "referrer": "landing_page",
  "session_id": "session-abc-123",
  "device_type": "mobile"
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Job view tracked successfully"]
  },
  "response": {
    "job_id": "job-uuid",
    "counted": true,
    "view_id": "view-uuid"
  }
}
```

## 9. Job Applications

Endpoint: `GET /api/v1/dashboard/company/jobs/<job_id>/applications/`

Usage: Company reviews candidates who applied to one job.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `status` | string | No | `applied`, `shortlisted`, `accepted`, `rejected`, `withdrawn`. |
| `search` | string | No | Search by learner name or muID. |
| `pageIndex` | integer | No | Page number. |
| `perPage` | integer | No | Items per page. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Applications fetched successfully"]
  },
  "response": {
    "job": {
      "id": "job-uuid",
      "title": "Backend Engineering Intern"
    },
    "applicants": [
      {
        "id": "application-uuid",
        "status": "applied",
        "cover_note": "I completed the Django PoW challenge.",
        "applicant_id": "user-uuid",
        "full_name": "Riya Sharma",
        "muid": "riya-sharma@mulearn",
        "district": "Thrissur",
        "karma": 3200,
        "level": {
          "id": "level-uuid",
          "name": "Explorer",
          "level_order": 4
        },
        "created_at": "2026-05-26T12:00:00+05:30"
      }
    ],
    "pagination": {
      "count": 45,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false
    }
  }
}
```

Endpoint: `PATCH /api/v1/dashboard/company/jobs/<job_id>/applications/<app_id>/`

Usage: Company moves an application through the hiring workflow.

Request body:

```json
{
  "status": "shortlisted",
  "review_note": "Strong PoW submission and relevant project work."
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Application shortlisted successfully"]
  },
  "response": {
    "application_id": "application-uuid",
    "applicant_id": "user-uuid",
    "new_status": "shortlisted",
    "reviewed_by": "company-user-uuid",
    "reviewed_at": "2026-05-28T10:00:00+05:30"
  }
}
```

## 10. Learner Discovery With Skill Filter

Endpoint: `GET /api/v1/dashboard/company/learners/`

Usage: Company discovers learners by karma, level, IG, achievement, skills, and work preference.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `search` | string | No | Search by name, muID, or district. |
| `karma_min` | integer | No | Minimum karma. |
| `karma_max` | integer | No | Maximum karma. |
| `level_order_min` | integer | No | Minimum level order. |
| `ig_ids` | string | No | Comma-separated IG IDs. |
| `achievement_ids` | string | No | Comma-separated achievement IDs. |
| `skill_ids` | string | No | Comma-separated skill IDs. |
| `skill_match` | string | No | `any` or `all`; default `any`. |
| `interested_in_work` | boolean | No | Only learners opted in for work. |
| `interested_in_gig_work` | boolean | No | Only learners opted in for gig work. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Learners fetched successfully"]
  },
  "response": {
    "learners": [
      {
        "id": "user-uuid",
        "full_name": "Riya Sharma",
        "muid": "riya-sharma@mulearn",
        "district": "Thrissur",
        "karma": 3200,
        "level": {
          "id": "level-uuid",
          "name": "Explorer",
          "level_order": 4
        },
        "interest_groups": [
          {
            "id": "ig-uuid",
            "name": "Web Development"
          }
        ],
        "skills": [
          {
            "id": "skill-uuid",
            "name": "Django"
          }
        ],
        "badges": [
          {
            "id": "achievement-uuid",
            "title": "API Builder"
          }
        ],
        "interested_in_work": true,
        "interested_in_gig_work": true
      }
    ],
    "pagination": {
      "count": 50,
      "totalPages": 5,
      "isNext": true,
      "isPrev": false
    }
  }
}
```

## 11. Shortlist Learner

Endpoint: `POST /api/v1/dashboard/company/learners/<user_id>/shortlist/`

Usage: Company saves a learner for follow-up. The shortlist may optionally be linked to a specific job.

Request body:

```json
{
  "job_id": "job-uuid",
  "note": "Strong backend profile and completed Django task.",
  "tags": ["backend", "django", "high-karma"]
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Learner shortlisted successfully"]
  },
  "response": {
    "shortlist_id": "shortlist-uuid",
    "learner": {
      "id": "user-uuid",
      "full_name": "Riya Sharma",
      "muid": "riya-sharma@mulearn"
    },
    "job_id": "job-uuid",
    "status": "active",
    "created_at": "2026-05-28T10:05:00+05:30"
  }
}
```

## 12. Shortlist List

Endpoint: `GET /api/v1/dashboard/company/shortlists/`

Usage: Company views saved learners, filters by job or status, and tracks follow-up state.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `job_id` | string | No | Filter shortlisted learners for a job. |
| `status` | string | No | `active`, `contacted`, `archived`. |
| `search` | string | No | Search learner name or muID. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Shortlists fetched successfully"]
  },
  "response": {
    "shortlists": [
      {
        "id": "shortlist-uuid",
        "status": "active",
        "note": "Strong backend profile.",
        "tags": ["backend", "django"],
        "learner": {
          "id": "user-uuid",
          "full_name": "Riya Sharma",
          "muid": "riya-sharma@mulearn",
          "karma": 3200,
          "level_order": 4
        },
        "job": {
          "id": "job-uuid",
          "title": "Backend Engineering Intern"
        },
        "created_at": "2026-05-28T10:05:00+05:30"
      }
    ],
    "pagination": {
      "count": 12,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false
    }
  }
}
```

## 13. Company Task List

Endpoint: `GET /api/v1/dashboard/company/tasks/`

Usage: Company tracks submitted PoW tasks and admin approval status.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `approval_status` | string | No | `pending`, `approved`, `rejected`. |
| `search` | string | No | Search by title or hashtag. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Tasks fetched successfully"]
  },
  "response": {
    "tasks": [
      {
        "id": "task-uuid",
        "title": "Build a REST API",
        "hashtag": "#django-api",
        "karma": 200,
        "approval_status": "pending",
        "rejection_reason": null,
        "ig_name": "Web Development",
        "active": false,
        "created_at": "2026-05-28T10:10:00+05:30"
      }
    ],
    "pagination": {
      "count": 2,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false
    }
  }
}
```

Endpoint: `POST /api/v1/dashboard/company/tasks/submit/`

Usage: Company submits a challenge/PoW task to be reviewed by admins before it becomes available to learners.

Request body:

```json
{
  "title": "Build a REST API",
  "hashtag": "#django-api",
  "description": "Create a documented Django REST API with authentication and tests.",
  "karma": 200,
  "ig_id": "ig-uuid",
  "type_id": "task-type-uuid",
  "channel_id": "channel-uuid",
  "level_id": "level-uuid",
  "skill_ids": ["skill-uuid-1", "skill-uuid-2"]
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Task submitted for admin review"]
  },
  "response": {
    "id": "task-uuid",
    "title": "Build a REST API",
    "hashtag": "#django-api",
    "approval_status": "pending",
    "active": false
  }
}
```

## 14. Company Badges

Endpoint: `GET /api/v1/dashboard/company/badges/`

Usage: Company views reputation badges earned through verification, hiring activity, mentoring, events, and PoW contributions.

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company badges fetched successfully"]
  },
  "response": {
    "badges": [
      {
        "id": "badge-verified-company",
        "title": "Verified Company",
        "description": "Company profile has been verified by muLearn.",
        "status": "earned",
        "earned_at": "2026-05-20T10:00:00+05:30"
      },
      {
        "id": "badge-active-hirer",
        "title": "Active Hirer",
        "description": "Posted at least 3 active opportunities and reviewed applicants.",
        "status": "in_progress",
        "progress": {
          "current": 2,
          "target": 3
        }
      }
    ]
  }
}
```

## 15. Engagement Score

Endpoint: `GET /api/v1/dashboard/company/engagement-score/`

Usage: Company sees a single health score based on jobs, tasks, reviews, hires, event collaborations, and learner engagement.

Query params:

| Param | Type | Required | Notes |
|---|---|---:|---|
| `period` | string | No | `30d`, `90d`, `all`; default `90d`. |

Request body: none.

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Company engagement score fetched successfully"]
  },
  "response": {
    "score": 78,
    "grade": "good",
    "period": "90d",
    "components": [
      {
        "key": "job_activity",
        "label": "Job activity",
        "score": 22,
        "max_score": 25
      },
      {
        "key": "application_review",
        "label": "Application review",
        "score": 18,
        "max_score": 25
      },
      {
        "key": "community_contribution",
        "label": "Community contribution",
        "score": 20,
        "max_score": 25
      },
      {
        "key": "learner_outcomes",
        "label": "Learner outcomes",
        "score": 18,
        "max_score": 25
      }
    ],
    "recommended_actions": [
      "Review 6 pending applications",
      "Submit one PoW challenge for Web Development IG"
    ]
  }
}
```

## 16. IG Collaboration Request

Endpoint: `POST /api/v1/dashboard/company/ig-collaboration-requests/`

Usage: Company requests collaboration with an Interest Group for a campaign, challenge, webinar, internship track, or hiring activity.

Request body:

```json
{
  "ig_id": "ig-uuid",
  "title": "Backend API Challenge",
  "description": "We want to run a Django challenge with the Web Development IG.",
  "campaign_type": "challenge",
  "expected_start_date": "2026-06-10",
  "expected_end_date": "2026-06-25",
  "target_learners": 100,
  "contact_email": "partners@acme.example"
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["IG collaboration request submitted successfully"]
  },
  "response": {
    "request_id": "collaboration-request-uuid",
    "status": "pending",
    "ig_id": "ig-uuid",
    "submitted_at": "2026-05-28T10:15:00+05:30"
  }
}
```

## 17. Company Event Request

Endpoint: `POST /api/v1/dashboard/company/events/request/`

Usage: Company requests to host or co-host an event with muLearn, a campus, or an Interest Group.

Request body:

```json
{
  "title": "Career Talk: Building Production APIs",
  "description": "A technical career session for backend learners.",
  "event_type": "webinar",
  "preferred_date": "2026-06-20",
  "target_audience": ["Web Development", "Backend"],
  "mode": "online",
  "speaker_name": "Jane Doe",
  "speaker_designation": "Engineering Manager",
  "contact_email": "events@acme.example"
}
```

Response body:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Event request submitted successfully"]
  },
  "response": {
    "request_id": "event-request-uuid",
    "status": "pending_review",
    "title": "Career Talk: Building Production APIs",
    "submitted_at": "2026-05-28T10:20:00+05:30"
  }
}
```

