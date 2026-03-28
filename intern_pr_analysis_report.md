# Intern Selection — Pull Request Code Quality & Assurance Report

> **Organization:** GTech μLearn  
> **Project:** mulearnbackend (Django REST Framework)  
> **Assessment Tasks:** Issue [#2651](https://github.com/gtech-mulearn/mulearnbackend/issues/2651) (College Sorting Bug Fix) + Issue [#2658](https://github.com/gtech-mulearn/mulearnbackend/issues/2658) (IG Task Summary API)  
> **Report Date:** March 28, 2026  
> **Reviewed By:** Automated Code Quality Analysis

---

## 1. Executive Summary

Three candidates submitted pull requests implementing two backend tasks for the μLearn platform: (1) fixing a college dashboard sorting bug where `sortBy=org` sorted by UUID instead of organization name, and (2) building a new IG Task Activity Summary API endpoint with date filtering and contributor aggregation.

This report provides a **line-by-line critical review** of each candidate's code changes, evaluating correctness, performance, security, Django/DRF best practices, Git hygiene, and production readiness. Each PR was reviewed against the same evaluation rubric to ensure fair comparison.

| Candidate | GitHub Handle | PRs | Submitted |
|---|---|---|---|
| **User 1** | [joshnajojo12](https://github.com/joshnajojo12) | [#2676](https://github.com/gtech-mulearn/mulearnbackend/pull/2676) + [#2675](https://github.com/gtech-mulearn/mulearnbackend/pull/2675) | Mar 18, 2026 |
| **User 2** | [NayanUnni95](https://github.com/NayanUnni95) | [#2673](https://github.com/gtech-mulearn/mulearnbackend/pull/2673) | Mar 17, 2026 |
| **User 3** | [Sidharth-sk-809](https://github.com/Sidharth-sk-809) | [#2671](https://github.com/gtech-mulearn/mulearnbackend/pull/2671) | Mar 17, 2026 |

---

## 2. Task 1 — College Sorting Bug Fix (Issue #2651)

### 2.1 Problem Statement

The `CollegeApi.get()` endpoint's `sortBy=org` parameter sorted colleges by the `org` foreign key's UUID (primary key) rather than the organization's human-readable `title` field. The `sort_fields` dictionary only had one entry (`'org': 'org'`) and lacked support for `level` and `created_at` fields.

### 2.2 Expected Solution

Map `org` to Django's double-underscore traversal `org__title`, and add `level` and `created_at` as sortable fields.

### 2.3 Comparative Review

All three candidates applied the **identical fix** to [api/dashboard/college/college_view.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/college/college_view.py):

```python
sort_fields={
    'org': 'org__title',
    'level': 'level',
    'created_at': 'created_at',
}
```

**Assessment:** This is the correct and minimal fix. The `CommonUtils.get_paginated_queryset()` utility handles ascending/descending via `-` prefix and silently ignores unsupported fields.

| Review Point | User 1 | User 2 | User 3 |
|---|:---:|:---:|:---:|
| Correct `org__title` mapping | ✅ | ✅ | ✅ |
| Added `level` sort field | ✅ | ✅ | ✅ |
| Added `created_at` sort field | ✅ | ✅ | ✅ |
| No unnecessary changes | ✅ | ✅ | ❌ (see §5.3) |
| Clean diff | ✅ | ✅ | ✅ |

> [!NOTE]
> Since all three candidates produced identical code for Task 1, differentiation is primarily based on Task 2 implementation and overall PR discipline.

---

## 3. Task 2 — IG Task Summary API (Issue #2658)

### 3.1 Requirements

Build a GET endpoint returning aggregated task activity for an Interest Group:
- Total tasks completed (count of `KarmaActivityLog` entries)
- Total karma awarded (sum of `karma` field)
- Unique contributors (distinct user count)
- Top 5 contributors by karma
- Optional `from_date` / `to_date` query parameter filtering
- Proper error handling for invalid IG ID and invalid date formats
- Role-based access control (ADMIN, FELLOW, ASSOCIATE)

### 3.2 User 1 — `joshnajojo12` | PR [#2676](https://github.com/gtech-mulearn/mulearnbackend/pull/2676)

**Files Changed:** `dash_ig_view.py`, `dash_ig_serializer.py`, `urls.py` (3 files)

#### 3.2.1 View Implementation — Critical Analysis

```python
class IGTaskSummaryAPI(APIView):
    authentication_classes = [CustomizePermission]

    @role_required([RoleType.ADMIN.value, RoleType.FELLOW.value, RoleType.ASSOCIATE.value])
    def get(self, request, ig_id):
```

**✅ Authentication & Authorization:** Correctly uses `CustomizePermission` + `@role_required` decorator matching the codebase convention. Access restricted to appropriate roles.

```python
        ig = (
            InterestGroup.objects.filter(id=ig_id)
            .values("id", "name", "code")
            .first()
        )
```

**✅ Efficient IG Lookup:** Uses `.values("id", "name", "code").first()` — fetches only necessary columns as a dictionary, avoiding full model hydration. This is the most efficient approach among all candidates.

```python
        from_date = self._parse_date(from_param) if from_param else None
        if from_param and from_date is None:
            return CustomResponse(
                general_message="Invalid date format. Use YYYY-MM-DD"
            ).get_failure_response()
        # ... same pattern for to_date ...

        if from_date and to_date and from_date > to_date:
            return CustomResponse(
                general_message="from_date cannot be after to_date"
            ).get_failure_response()
```

**✅ Date Validation Excellence:** Three-stage validation:
1. Format validation via `_parse_date()` helper
2. Individual error responses for each invalid date
3. **Logical validation** — `from_date > to_date` check

> [!IMPORTANT]
> User 1 is the **only** candidate who validates that `from_date` is not after `to_date`. This is a common edge case that would produce confusing empty results in production if not caught.

```python
        filters = Q(task__ig_id=ig_id)
        if from_date:
            filters &= Q(created_at__date__gte=from_date)
        if to_date:
            filters &= Q(created_at__date__lte=to_date)

        activity_qs = KarmaActivityLog.objects.filter(filters)
```

**✅ Clean Query Composition:** Uses Django `Q` objects for composable filter building. Single queryset is reused for both aggregation and top contributors.

```python
        summary_values = activity_qs.aggregate(
            total_tasks=Count("id"),
            total_karma=Coalesce(Sum("karma"), 0),
            unique_contributors=Count("user", distinct=True),
        )
```

**✅ Optimal Aggregation:** Single `aggregate()` call produces all three metrics in **one SQL query**. Uses `Coalesce(Sum("karma"), 0)` to handle `NULL` at the database level — this is the correct Django ORM practice. `Count("user", distinct=True)` efficiently counts unique contributors.

```python
        top_contributors_queryset = (
            activity_qs.filter(user__isnull=False)
            .values("user__full_name", "user__muid")
            .annotate(karma_earned=Coalesce(Sum("karma"), 0))
            .order_by("-karma_earned", "user__full_name")[:5]
        )
```

**✅ Deterministic Ordering:** Secondary sort on `user__full_name` ensures stable ordering when two users have equal karma — critical for consistent pagination and API responses. Other candidates missed this.

**✅ Null User Filtering:** `user__isnull=False` prevents `None` entries in top contributors.

```python
    @staticmethod
    def _parse_date(value: str):
        try:
            return datetime.strptime(value, "%Y-%m-%d").date()
        except ValueError:
            return None
```

**✅ Helper Method:** Clean `@staticmethod` with type hint. Returns `None` sentinel on failure — simple and effective.

#### 3.2.2 Serializer Design

```python
class IGTopContributorSerializer(serializers.Serializer):
    full_name = serializers.CharField()
    muid = serializers.CharField()
    karma_earned = serializers.IntegerField()

class IGTaskSummarySerializer(serializers.Serializer):
    ig_id = serializers.CharField()
    ig_name = serializers.CharField()
    ig_code = serializers.CharField()
    total_tasks_completed = serializers.IntegerField()
    total_karma_awarded = serializers.IntegerField()
    unique_contributors = serializers.IntegerField()
    top_contributors = IGTopContributorSerializer(many=True)
    date_range = serializers.DictField(child=serializers.DateField(allow_null=True))
```

**✅ Proper Serializer Usage:** Uses DRF serializers for response validation with `serializer.is_valid(raise_exception=True)`. Separate `IGTopContributorSerializer` for nested contributor data.

**⚠️ Minor Issue:** `date_range = serializers.DictField(child=serializers.DateField(...))` — using `DateField` as child of `DictField` means the serialized output will be `date` objects, not strings. Could cause serialization issues with some clients.

#### 3.2.3 URL Registration

```python
path('ig/<str:ig_id>/task-summary/', dash_ig_view.IGTaskSummaryAPI.as_view()),
```

**⚠️ Redundant Path Prefix:** Since these URLs are already included under `dashboard/ig/`, the full path becomes `/api/v1/dashboard/ig/ig/<ig_id>/task-summary/` — the `ig/` is duplicated. This is a functional but cosmetic issue that would show up in API documentation.

#### 3.2.4 User 1 — Summary

| Aspect | Rating | Notes |
|---|:---:|---|
| Correctness | **9/10** | Handles all edge cases; minor URL duplication |
| ORM Efficiency | **10/10** | Single aggregate query, `Coalesce`, deterministic sort |
| Error Handling | **10/10** | Only candidate with `from_date > to_date` validation |
| Code Style | **8/10** | Clean but no docstrings in view method |
| Security | **9/10** | Proper auth, no injection vectors, validates all inputs |
| PR Discipline | **9/10** | Separate PRs per task — excellent Git workflow |

---

### 3.3 User 2 — `NayanUnni95` | PR [#2673](https://github.com/gtech-mulearn/mulearnbackend/pull/2673)

**Files Changed:** `college_view.py`, `dash_ig_view.py`, `dash_ig_serializer.py`, `urls.py`, `Dashboard_IG.md` (5 files)

#### 3.3.1 View Implementation — Critical Analysis

```python
    def get(self, request, ig_id):
        """Return a task activity summary for a given Interest Group.

        Supports optional date range filtering via `from_date` and `to_date`
        query params (format: YYYY-MM-DD). Returns aggregate zeros — not a
        failure — when there is no activity in the requested range.
        """
```

**✅ Excellent Docstring:** Clearly documents behavior, parameters, and the important design decision that empty results return zeros (not errors). This is production-level documentation.

```python
        ig = InterestGroup.objects.filter(id=ig_id).first()
        if not ig:
            return CustomResponse(
                general_message="Interest Group not found"
            ).get_failure_response()
```

**⚠️ Full Model Hydration:** Uses `.first()` which loads the full `InterestGroup` model. Functionally correct but slightly less efficient than User 1's `.values().first()`. Accesses `ig.id`, `ig.name`, `ig.code` later which works fine.

**⚠️ Missing HTTP Status Code:** `.get_failure_response()` without explicit `status_code=404` — defaults to 400, which is semantically incorrect for "not found."

```python
        try:
            if from_date_str:
                from_date = datetime.strptime(from_date_str, "%Y-%m-%d")
            if to_date_str:
                to_date = datetime.strptime(to_date_str, "%Y-%m-%d")
        except ValueError:
            return CustomResponse(
                general_message="Invalid date format. Use YYYY-MM-DD"
            ).get_failure_response()
```

**⚠️ Redundant Parsing:** `datetime.strptime()` returns a `datetime` object, but later does `.date()` for filtering. User 1 directly parses to `date` — cleaner.

**⚠️ Single Error Message:** If both dates are invalid, reports only first error. Acceptable trade-off but less user-friendly.

```python
        logs = KarmaActivityLog.objects.filter(
            task__ig=ig,
            user__isnull=False,
        ).select_related("user", "task")
```

**⚠️ Unnecessary `select_related()`:** The queryset later uses `.values()` and `.aggregate()` which generate their own SQL — the `select_related("user", "task")` JOIN is wasted, adding unnecessary database overhead. This suggests a misunderstanding of when `select_related` is beneficial.

**⚠️ Pre-filters `user__isnull=False`:** Excludes null-user records from the base queryset. This means `total_tasks_completed` and `total_karma_awarded` will not count entries with null users — which may or may not be the intended behavior depending on requirements.

```python
        totals = logs.aggregate(
            total_tasks_completed=Count("id"),
            total_karma_awarded=Sum("karma"),
            unique_contributors=Count("user", distinct=True),
        )
```

**✅ Single Aggregate Query:** Good — one DB call for all metrics.

**⚠️ Missing `Coalesce`:** `Sum("karma")` returns `None` when no records match. Relies on Python-level `or 0` fallback later — functionally correct but not best practice. `Coalesce(Sum("karma"), 0)` handles this at the DB level.

```python
        top_contributors = [
            {
                "full_name": row["user__full_name"],
                "muid": row["user__muid"],
                "karma_earned": row["karma_earned"],
            }
            for row in (
                logs.values("user__full_name", "user__muid")
                .annotate(karma_earned=Sum("karma"))
                .order_by("-karma_earned")[:5]
            )
        ]
```

**✅ Correct Top Contributors:** Proper GROUP BY via `.values().annotate()` pattern. Slicing `[:5]` applies `LIMIT 5` at SQL level.

**⚠️ Non-deterministic Ordering:** Only sorts by `-karma_earned`. Users with equal karma will appear in arbitrary database order — results may vary between requests.

```python
        serializer = IGTaskSummarySerializer(data={...})
        serializer.is_valid(raise_exception=True)

        return CustomResponse(
            general_message="Task summary fetched successfully",
            response=serializer.validated_data,
        ).get_success_response()
```

**✅ Uses `validated_data`:** Returns `serializer.validated_data` instead of `serializer.data` — technically more correct as it represents the cleaned/validated output.

#### 3.3.2 Serializer Design

```python
class IGTaskSummarySerializer(serializers.Serializer):
    """Read-only serializer documenting the IGTaskSummaryAPI response shape."""

    class ContributorSerializer(serializers.Serializer):
        full_name = serializers.CharField()
        muid = serializers.CharField()
        karma_earned = serializers.IntegerField()

    # ... fields ...
    top_contributors = ContributorSerializer(many=True)
    date_range = serializers.DictField(child=serializers.CharField(allow_null=True))
```

**✅ Nested Serializer:** `ContributorSerializer` is nested inside `IGTaskSummarySerializer` — cleaner namespace, indicates it's only used here.

**✅ Docstring:** Serializer documents its purpose.

**⚠️ Missing Newline:** File doesn't end with a newline — fails POSIX compliance and will show diff noise.

#### 3.3.3 API Documentation

```markdown
## Endpoint: `<str:ig_id>/task-summary/`
- Brief: Get task activity summary for a specific Interest Group.
- Path params: `str:ig_id`
- Request body example (JSON): ...
```

**✅ API Docs Updated:** User 2 is the **only** candidate who updated `Dashboard_IG.md` with the new endpoint documentation. This shows awareness that API changes require documentation updates.

**⚠️ Documentation Error:** Lists `from_date` and `to_date` in a "Request body example" JSON block, but these are **query parameters** for a GET request, not request body fields. This could mislead frontend developers.

#### 3.3.4 User 2 — Summary

| Aspect | Rating | Notes |
|---|:---:|---|
| Correctness | **8/10** | Works correctly; missing 404 status, null-user pre-filter |
| ORM Efficiency | **7/10** | Wasted `select_related`, no `Coalesce` |
| Error Handling | **7/10** | No `from_date > to_date` check, default 400 for not-found |
| Code Style | **9/10** | Excellent docstrings, nested serializer, clean formatting |
| Security | **9/10** | Proper auth, input validation |
| PR Discipline | **8/10** | Single PR for both tasks, good descriptions, checklist |
| Documentation | **10/10** | Only candidate to update API docs |

---

### 3.4 User 3 — `Sidharth-sk-809` | PR [#2671](https://github.com/gtech-mulearn/mulearnbackend/pull/2671)

**Files Changed:** 10 files (3 modified, 4 added, 7 deleted)

> [!CAUTION]
> **Critical Issue:** This PR deletes 7 core project configuration files from the `mulearnbackend/` directory. This alone would make the PR un-mergeable and would break the entire application in production.

#### 3.4.1 Deleted Files — Severity Analysis

| Deleted File | Impact | Severity |
|---|---|:---:|
| [mulearnbackend/__init__.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/__init__.py) | Breaks Celery task discovery | 🔴 Critical |
| [mulearnbackend/settings.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/settings.py) | Entire Django configuration lost (DB, cache, middleware, etc.) | 🔴 Critical |
| [mulearnbackend/urls.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/urls.py) | All URL routing destroyed | 🔴 Critical |
| [mulearnbackend/wsgi.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/wsgi.py) | Production WSGI deployment breaks | 🔴 Critical |
| [mulearnbackend/asgi.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/asgi.py) | WebSocket/async deployment breaks | 🔴 Critical |
| [mulearnbackend/celery.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/celery.py) | Background task processing breaks | 🔴 Critical |
| [mulearnbackend/middlewares.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/middlewares.py) | Error handling, IP binding, API signature verification lost | 🔴 Critical |
| [mulearnbackend/routing.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/mulearnbackend/routing.py) | WebSocket routing lost | 🟡 High |

**Root Cause Assessment:** This appears to be caused by improper Git operations — likely rebasing or resetting against an incorrect base branch, or force-pushing after accidental file deletion. This indicates a **lack of Git proficiency** that is concerning for production-level work.

#### 3.4.2 Extra Files Added to Repository Root

| Added File | Issue |
|---|---|
| `FIX_SUMMARY.md` (126 lines) | Documentation file committed to repo root — should be in PR description or wiki |
| `IG_TASK_SUMMARY_API_TESTING.md` (213 lines) | Testing guide in repo root — should be in `/docs` or PR description |
| `IG_TASK_SUMMARY_TEST_DATA.sql` (71 lines) | Raw SQL test data in repo root — should use Django fixtures or be in `/tests` |
| `IG_TASK_SUMMARY_TEST_DATA_GUIDE.md` (261 lines) | Data guide in repo root — should accompany test fixtures, not live in root |

**Assessment:** While demonstrating thoroughness, committing 671 lines of documentation and test data to the repository root violates standard project structure conventions. These files would pollute the root directory and should either live in PR descriptions, a `/docs` directory, or as Django test fixtures.

#### 3.4.3 View Implementation — Critical Analysis

```python
class IGTaskSummaryAPI(APIView):
    """API to fetch task activity summary for an Interest Group."""
    authentication_classes = [CustomizePermission]

    @role_required([RoleType.ADMIN.value, RoleType.FELLOW.value, RoleType.ASSOCIATE.value])
    def get(self, request, ig_id):
        """
        Get task activity summary for an Interest Group.

        Query Parameters:
            - from_date (optional): Start date in YYYY-MM-DD format
            - to_date (optional): End date in YYYY-MM-DD format

        Returns:
            - ig_id, ig_name, ig_code, total_tasks_completed,
              total_karma_awarded, unique_contributors,
              top_contributors, date_range
        """
```

**✅ Comprehensive Docstrings:** Both class-level and method-level docstrings with full parameter documentation. Best documentation among all candidates.

```python
        ig = InterestGroup.objects.filter(id=ig_id).first()
        if not ig:
            return CustomResponse(
                general_message="Interest Group not found"
            ).get_failure_response(status_code=404, http_status_code=404)
```

**✅ Correct 404 Status:** Properly returns HTTP 404 with both `status_code` and `http_status_code` parameters.

```python
        date_filter = Q()
        date_range_response = {
            "from_date": from_date,
            "to_date": to_date,
        }
```

**✅ Q Object Initialization:** Starts with empty `Q()` and builds conditionally — clean pattern.

**✅ Original Strings in Response:** Stores original `from_date`/`to_date` strings for the response before parsing — avoids re-serialization issues.

```python
        karma_logs = KarmaActivityLog.objects.filter(task__ig_id=ig_id).filter(date_filter)

        # Count tasks completed
        total_tasks_completed = karma_logs.count()

        # Sum karma awarded
        total_karma_result = karma_logs.aggregate(total=Sum("karma"))
        total_karma_awarded = total_karma_result.get("total") or 0

        # Count unique contributors
        unique_contributors = karma_logs.filter(
            user_id__isnull=False
        ).values("user_id").distinct().count()
```

**⚠️ Multiple DB Queries:** Executes **3 separate database queries** (`count()`, `aggregate()`, `filter().values().distinct().count()`) instead of a single `aggregate()` call. This is approximately 3x the database load compared to User 1 and User 2's implementations.

**Performance comparison:**
```
User 1: 1 aggregate() + 1 annotate()     = 2 queries
User 2: 1 aggregate() + 1 annotate()     = 2 queries
User 3: count() + aggregate() + count()  = 3 queries + 1 annotate() = 4 queries total
```

**⚠️ Missing `Coalesce`:** Same issue as User 2 — relies on Python `or 0` instead of DB-level null handling.

```python
        # Get top 5 contributors
        top_contributors = (
            karma_logs
            .filter(user_id__isnull=False)
            .values("user_id", "user__full_name", "user__muid")
            .annotate(karma_earned=Sum("karma"))
            .order_by("-karma_earned")[:5]
        )
```

**⚠️ Extra `user_id` in VALUES:** Includes `user_id` in `.values()` grouping — while not incorrect, it's unnecessary for the response and adds a field to the GROUP BY clause. User 1 and User 2 correctly omit it.

**⚠️ Non-deterministic Ordering:** Same issue as User 2 — no secondary sort key.

#### 3.4.4 Missing Serializer

**⚠️ No Serializer Validation:** The response is built as a raw dictionary and returned directly without DRF serializer validation. While functional, this means:
- No type checking on response fields
- No validation that the response matches the expected schema
- Inconsistent with Django REST Framework conventions
- Other candidates use serializers for output validation

#### 3.4.5 Unused Import

```python
from db.user import User  # Never used in the code
```

**⚠️ Dead Import:** `User` is imported but never referenced — indicates leftover code from development/experimentation.

#### 3.4.6 User 3 — Summary

| Aspect | Rating | Notes |
|---|:---:|---|
| Correctness | **7/10** | Core logic works; no serializer validation |
| ORM Efficiency | **5/10** | 4 DB queries instead of 2; unnecessary fields in GROUP BY |
| Error Handling | **8/10** | Proper 404, date validation; no range validation |
| Code Style | **8/10** | Excellent docstrings and comments; unused import |
| Security | **9/10** | Proper auth, input validation |
| PR Discipline | **2/10** | Deleted 7 core files; 4 extra files in repo root |
| Git Proficiency | **2/10** | Destructive changes indicate poor Git skills |

---

## 4. Cross-Cutting Analysis

### 4.1 ORM & Database Performance

| Metric | User 1 | User 2 | User 3 |
|---|:---:|:---:|:---:|
| Total DB queries (summary) | 1 | 1 | 3 |
| Total DB queries (top contributors) | 1 | 1 | 1 |
| **Total queries** | **2** | **2** | **4** |
| Uses `Coalesce()` for null-safe aggregation | ✅ | ❌ | ❌ |
| Avoids unnecessary JOINs | ✅ | ❌ (`select_related` wasted) | ✅ |
| Deterministic sort ordering | ✅ (secondary sort) | ❌ | ❌ |
| Efficient IG lookup | ✅ (`.values().first()`) | ⚠️ (full model) | ⚠️ (full model) |

### 4.2 Error Handling & Input Validation

| Scenario | User 1 | User 2 | User 3 |
|---|:---:|:---:|:---:|
| Invalid IG ID → 404 | ✅ (explicit 404) | ⚠️ (default 400) | ✅ (explicit 404) |
| Invalid date format | ✅ | ✅ | ✅ |
| `from_date > to_date` | ✅ | ❌ | ❌ |
| Missing auth token | ✅ | ✅ | ✅ |
| Wrong role | ✅ | ✅ | ✅ |
| No activity in range → 200 with zeros | ✅ | ✅ | ✅ |

### 4.3 Code Quality & Django Conventions

| Convention | User 1 | User 2 | User 3 |
|---|:---:|:---:|:---:|
| Uses DRF Serializer for response | ✅ | ✅ | ❌ |
| View docstrings | ❌ | ✅ (excellent) | ✅ (excellent) |
| Serializer docstrings | ❌ | ✅ | N/A |
| No unused imports | ✅ | ✅ | ❌ (`User`) |
| File ends with newline | ✅ | ❌ | ✅ |
| Follows existing code patterns | ✅ | ✅ | ✅ |
| API documentation updated | ❌ | ✅ | ❌ (separate file) |

### 4.4 Git & PR Discipline

| Criteria | User 1 | User 2 | User 3 |
|---|:---:|:---:|:---:|
| Separate PRs per task | ✅ (best practice) | ❌ (combined) | ❌ (combined) |
| Clean diff (no unrelated changes) | ✅ | ✅ | ❌ (7 deleted files) |
| Descriptive PR title | ✅ | ✅ | ✅ |
| Linked issues | ✅ | ✅ | ✅ |
| Testing evidence | ✅ (Postman) | ✅ (Postman + edge cases) | ✅ (Postman + test data) |
| No repo pollution | ✅ | ✅ | ❌ (4 extra root files) |

---

## 5. Final Scoring Matrix

| Category (Weight) | User 1 — joshnajojo12 | User 2 — NayanUnni95 | User 3 — Sidharth-sk-809 |
|---|:---:|:---:|:---:|
| **Code Correctness** (20%) | 18/20 | 16/20 | 14/20 |
| **ORM & Performance** (15%) | 15/15 | 11/15 | 8/15 |
| **Error Handling** (15%) | 15/15 | 11/15 | 12/15 |
| **Code Quality & Style** (10%) | 8/10 | 9/10 | 7/10 |
| **Django/DRF Best Practices** (10%) | 9/10 | 9/10 | 6/10 |
| **Git & PR Discipline** (15%) | 14/15 | 12/15 | 3/15 |
| **Documentation** (10%) | 7/10 | 10/10 | 7/10 |
| **Production Readiness** (5%) | 5/5 | 4/5 | 1/5 |
| **TOTAL** | **91/100** | **82/100** | **58/100** |

---

## 6. Final Recommendation

### 🥇 Rank 1 — User 1 (`joshnajojo12`) — **91/100** — Strongly Recommended

The strongest candidate from a **pure engineering standpoint**. Demonstrates deep understanding of Django ORM optimization (`Coalesce`, `Q` objects, `.values().first()`), comprehensive edge case handling (date range validation), and excellent Git workflow discipline (separate PRs per task). Code is production-ready with minimal changes needed. The only improvement area is adding docstrings.

### 🥈 Rank 2 — User 2 (`NayanUnni95`) — **82/100** — Recommended

A well-rounded candidate who balances code quality with professional documentation practices. The only candidate to update API docs — showing real-world awareness that code changes have downstream impacts. Minor technical gaps (wasted `select_related`, missing `Coalesce`, incorrect HTTP status for 404) are easily coachable. Would benefit from deeper ORM optimization training.

### 🥉 Rank 3 — User 3 (`Sidharth-sk-809`) — **58/100** — Not Recommended

While demonstrating enthusiasm through extensive documentation (FIX_SUMMARY.md, testing guides, SQL seed data), the **accidental deletion of 7 core project files** is a disqualifying concern for production work. This indicates insufficient Git proficiency, which is a fundamental skill for collaborative backend development. Additionally, the code uses 4 DB queries where 2 suffice, skips serializer validation, and has unused imports. The supplementary files, while well-intentioned, were committed to the repository root rather than proper locations.

---

> [!NOTE]
> This report evaluates code quality as demonstrated in the submitted PRs. It does not assess communication skills, domain knowledge, learning potential, or other factors that may be relevant for intern selection. The scoring reflects observable technical quality only.
