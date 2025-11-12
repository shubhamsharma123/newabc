# newabc
abc 

Below are three complete prompts — one per assignment. Feel free to tweak the "Priority" or "DB" sections depending on whether you have PostgreSQL available.

Prompt 1 — Full-Text Search (Assignment 1)
Role:

You are an experienced backend engineer and database developer with knowledge of Python, FastAPI, SQLAlchemy, and PostgreSQL full-text search. You will implement a production-quality search feature and add tests.
Goal:

Add a search endpoint GET /books/search?q={query}&limit={n} that returns ranked book results searching title, description, and author. Use PostgreSQL tsvector + plainto_tsquery and weights; fallback to ilike when Postgres isn't available.
Context (repository-specific):

Repo root contains main.py and package api.
Book model is model.py with fields: id, title, description (may be newly added), author, publication_date, ISBN, cover_image, category, user_id.
Router for books: router.py.
Service for book logic: service.py.
Config: config.py exposes DATABASE_URL.
Pydantic schemas: schema.py.
No migration tool is present in the repo by default (you may add Alembic files if you choose).
There is an images folder used for cover images.
Requirements / Steps (explicit):

Inspect current Book model and schema. If description field is missing, add it to both model and schema and ensure ORM/Pydantic compatibility.
Implement search_books(db: Session, query: str, limit: int = 20) in service.py:
If DATABASE_URL indicates Postgres (startswith 'postgres' or 'postgresql'), run a raw SQL query using plainto_tsquery(:q) on a weighted tsvector composed from title (weight 'A'), description (weight 'B'), and author (weight 'C'). Order by ts_rank_cd desc and limit by limit. Return Pydantic BookResponse list, preserving the rank order.
Else, fallback to SQLAlchemy ilike across title, description, and author, limit results, and return BookResponse list.
Add a GET route /books/search in router.py that accepts q and limit query params and calls the service.
Add basic unit tests under test:
Test Postgres branch by mocking get_settings() returning a Postgres URL or use an actual test DB if available (preferred).
Test fallback branch using SQLite in-memory DB; seed a few books and assert search returns expected results.
Documentation: update README.md or add a short comment describing the endpoint and any DB prerequisites.
(Optional but recommended) Provide an Alembic migration script that:
Adds a description column to books.
Adds a materialized search_vector tsvector column and a GIN index.
Adds a trigger to update search_vector on insert/update.
Acceptance criteria:

GET /books/search returns JSON list of books matching the query, ranked by relevance when Postgres is used.
Works with Postgres and with fallback ilike (SQLite) for testing.
Tests cover happy-path search and fallback behavior.
No breaking changes to existing endpoints.
Security & constraints:

Protect file uploads and paths (no directory traversal). Reuse existing file handling logic.
Avoid SQL injection: use parameterized queries (bind params).
Keep changes minimal to existing code structure.
Files to edit (suggested):

model.py
schema.py
service.py
router.py
test/test_search.py (new)
README.md (update)
Example tests / queries:

Seed books: ("FastAPI in Action", "Great FastAPI guide", "Author A"), ("Deep Learning", "ML book", "Author B").
GET /books/search?q=FastAPI should return the FastAPI book first.
GET /books/search?q=Author%20B should return Deep Learning.
Prompt 2 — RBAC (Assignment 2)
Role:

You are an experienced backend engineer with expertise in authentication/authorization and Python web frameworks. Implement role-based access control and secure role management endpoints.
Goal:

Add role support to users (admin, editor, reader). Implement middleware/dependency to enforce role requirements and protect endpoints: admin-only delete, editor+ update, reader read-only. Add an endpoint to assign/update roles securely.
Context (repository-specific):

Current user model: model.py with id, name, email, password, books relationship.
Current auth middleware: security.py and router.py handle JWT tokens; JWTAuth adds request.user.
Current user service: service.py for creating users, validation helpers.
Requirements / Steps:

Extend User model:
Add roles column (string) storing roles as comma-separated values or use a separate roles relationship table. For quick implementation, use CSV string; for better normalization, add a roles table and association.
Update Pydantic schemas to include roles in responses and role update request.
Add DB migration (if possible) to add roles column or new table.
Implement a dependency or decorator require_roles(*allowed_roles) in a new module api/auth/permissions.py:
The dependency should read request.user (set by JWTAuth) and check user's roles.
If not authorized, raise 403 HTTPException.
Should be usable as dependencies=[Depends(require_roles('admin'))] in router definitions, or as a decorator for handlers.
Apply permissions:
Update router.py: delete endpoint -> admin only; update endpoint -> editor or admin; create/get -> reader+ (any authenticated).
Implement endpoint POST /users/{user_id}/roles (or /users/{user_id}/roles) to assign roles, protected by admin only.
Tests:
Create tests asserting admin can delete, editor can update, reader cannot delete/update.
Documentation: add RBAC section to README.
Acceptance criteria:

Role checks enforced, correct HTTP 403 for unauthorized attempts.
Role assignment endpoint only accessible by admin.
Tests demonstrating major role-based behaviors.
Files to edit:

model.py
schema.py
api/auth/permissions.py (new)
router.py (apply dependencies)
router.py (add roles endpoint)
test/test_rbac.py (new)
Prompt 3 — Recommendations (Assignment 3)
Role:

You are an experienced backend engineer/data engineer familiar with building simple recommendation logic, SQLAlchemy, and designing extensible schemas.
Goal:

Add a /recommendations/{user_id} endpoint that returns 5 recommended books based on a user's reading history (author and genre similarity).
Context (repo-specific):

Book model has category representing genre, and author string.
No reading_history table currently exists.
Requirements / Steps:

Add reading_history model/table model.py or in db:
Columns: id, user_id (FK), book_id (FK), read_timestamp (datetime).
Add a service service.py implementing:
get_recommendations(db, user_id, limit=5):
Retrieve books read by the user from reading_history.
Compute candidate books the user hasn’t read.
Score candidates: +2 if same author as any read book, +1 if same category as any read book. Optionally weight by read recency.
Return top limit books as BookResponse list.
Add endpoint GET /recommendations/{user_id} in router.py:
Should be protected so only the user themselves or admin can fetch recommendations (use require_roles from RBAC or check request.user.id).
Tests:
Seed reading_history for a user and ensure recommendations include books with matching authors/genres not in the user's history.
Cold-start: if user has no history, return popular or recent books (e.g., top 5 by created_time).
Documentation: add a short section describing recommendation logic and its extensibility.
Acceptance criteria:

Endpoint returns up to 5 unique recommendations.
Recommendations exclude books the user has already read.
Tests cover normal and cold-start cases.
Files to edit/add:

model.py
service.py
router.py
test/test_recommendations.py
Usage notes for AI agent:

If the repo uses migrations, create migration scripts. Otherwise, add schema changes and document them.
Keep changes modular (new modules under api/recommendation and api/auth/permissions).
Ensure all DB queries are efficient (use IN, index columns where needed).
Use parameterized queries when executing raw SQL.
