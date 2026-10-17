# AGENTS.md - Django URL Shortener Project Guidelines

## 1. Project Overview
You are building a high-performance, secure, and extensible **URL Shortener** web application using **Django**. The system should handle URL redirection with minimal latency, support custom short codes, track click analytics (IP, user agent, timestamps), and include optional expiration dates.

---

## 2. Tech Stack & Versions
- **Python:** 3.11+
- **Framework:** Django 5.x
- **Database:** PostgreSQL (primary data store)
- **Caching / Analytics Queue:** Redis (for caching redirects and handling click metrics asynchronously)
- **Testing:** `pytest`, `pytest-django`, `factory_boy`
- **Containerization:** Docker & Docker Compose

---

## 3. Architecture & Core Models

### `ShortURL` Model
- `id`: UUID (Primary Key)
- `original_url`: URLField (Supports long URLs up to 2048 chars)
- `short_code`: CharField (Unique, indexed, max length 10-15 chars. Used in the redirect path: `domain.com/<short_code>`)
- `custom_code`: Boolean (True if user-defined, False if auto-generated)
- `created_at`: DateTimeField (auto_now_add=True)
- `expires_at`: DateTimeField (Nullable/Optional)
- `is_active`: BooleanField (Default: True)
- `click_count`: PositiveIntegerField (Default: 0 - denormalized counter for quick stats)

### `ClickAnalytics` Model
- `short_url`: ForeignKey to `ShortURL` (on_delete=CASCADE)
- `clicked_at`: DateTimeField (auto_now_add=True)
- `ip_address`: GenericIPAddressField (Nullable for privacy compliance)
- `user_agent`: TextField (Nullable)
- `referer`: URLField (Nullable)

---

## 4. Coding Standards & Best Practices
1. **Clean Code & PEP 8:** Follow standard Python/Django style guidelines. Use descriptive variable names and keep views thin (business logic belongs in services/managers).
2. **Type Hints:** Use type annotations for all functions, methods, and service classes.
3. **Database Performance:** 
   - Always index fields used in frequent lookups (e.g., `short_code`).
   - Use `select_related` and `prefetch_related` where necessary to avoid N+1 query problems.
4. **Security:**
   - Validate and sanitize all input URLs against malformed schemes and malicious patterns (prevent open redirects or SSRF vectors).
   - Implement rate limiting on URL creation endpoints to prevent abuse/spam.
5. **Error Handling:** Gracefully handle non-existent or expired short codes with standard HTTP 404 responses.

---

## 5. Implementation Milestones (Step-by-Step)

Follow these phases sequentially when building or expanding the project:

### Phase 1: Project Setup & Core Configuration
- [ ] Initialize Django project and app structure (`urls`, `core`, `analytics`).
- [ ] Configure PostgreSQL and Redis connections in `settings.py` via environment variables (`django-environ`).
- [ ] Set up base settings for security, static files, and logging.

### Phase 2: Database Models & Migrations
- [ ] Implement `ShortURL` and `ClickAnalytics` models with proper indexes and constraints.
- [ ] Write custom model managers or utility functions for generating unique Base62 short codes (e.g., encoding primary keys or generating secure random alphanumeric strings).
- [ ] Generate and apply initial database migrations.

### Phase 3: Core Business Logic & Redirection Engine
- [ ] Build the URL creation service (handling validation, collision checks for auto-generated codes, and custom alias availability).
- [ ] Implement the lightning-fast redirect view:
  - Check Redis cache first for the mapping.
  - Fall back to PostgreSQL if cache misses, then populate cache.
  - Asynchronously or synchronously log click analytics.
  - Issue an HTTP `302 Found` (or `301 Moved Permanently` depending on configuration) redirect.

### Phase 4: API / Web Interface & Analytics
- [ ] Build clean Django templates (or Django REST Framework endpoints if building an API-first service) for shortening URLs.
- [ ] Build an analytics dashboard view showing total clicks, geographic/referrer trends, and creation history for a given short code.

### Phase 5: Testing & Quality Assurance
- [ ] Write unit and integration tests using `pytest` covering:
  - Short code generation uniqueness and collision handling.
  - Redirection logic and expiration checks.
  - API endpoint validation and rate limiting.
- [ ] Ensure test coverage exceeds 80%.

---

## 6. Git, Commit, & Pull Request Guidelines

### Branching Strategy
- **`main` / `master`**: Production-ready code only.
- **Feature Branches**: Name feature branches systematically:
  - `feat/short-code-generator`
  - `fix/redis-cache-invalidation`
  - `test/analytics-endpoints`

### Commit Message Standards
Follow the **Conventional Commits** specification (`type(scope): description`).

- **Allowed Types:**
  - `feat`: A new feature or endpoint.
  - `fix`: A bug fix or patch.
  - `test`: Adding or updating test cases.
  - `refactor`: Code changes that neither fix a bug nor add a feature.
  - `docs`: Documentation updates.
  - `chore`: Maintenance tasks, dependency updates, or Docker changes.
- **Rules:**
  - Keep the summary line under **50 characters**.
  - Use the **imperative mood** in the title (e.g., `feat(core): add Base62 encoding utility`, NOT `added` or `adds`).
  - Commits must be **atomic**—one logical change per commit. Avoid mixing formatting/refactoring with new feature logic.

### Pull Request (PR) Requirements
When creating or proposing a PR, use the following standardized format:

```markdown
## Summary
- Brief 2-3 sentence overview of what changed and why.

## Key Changes
- [ ] Added Base62 short code encoder in `core/utils.py`.
- [ ] Implemented Redis caching layer in redirect view.
- [ ] Added unit tests covering collision handling for short codes.

## How to Test
1. Run `pytest` to execute test suite.
2. Spin up Redis via `docker-compose up redis`.
3. Submit a POST request to `/api/v1/shorten/` and verify redirect response at `http://localhost:8000/<short_code>`.

## Checklist
- [ ] Code follows project PEP 8 and typing standards.
- [ ] Unit and integration tests pass locally.
- [ ] No migrations missing for database changes.
- [ ] Updated documentation/README if needed.