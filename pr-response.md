# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill at the end -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to follow the project's verb_to_noun naming convention, matching `add_to_collection()`. Updated the one call site in `routes/watchlist/watchlist.py` — both the import and the call inside `add_film()`.
**How I verified:** Ran `grep -rn "save_to_watchlist" . --include="*.py"` across the project (excluding `.venv`) to confirm no references to the old name remained — it returned nothing. Then ran `pytest tests/ -v`; all tests still pass.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to `add_to_watchlist()` mirroring the pattern in `add_to_collection()`. Before inserting, it queries `WatchlistEntry.query.filter_by(user_id=user_id, film_id=film_id).first()`; if a row already exists it raises a new `AlreadyInWatchlistError` instead of committing a second entry. I defined `AlreadyInWatchlistError` as a watchlist-specific exception, paralleling the collection service's `AlreadyInCollectionError`, rather than reusing the collection exception, so the error names stay domain-accurate.
**How I verified:** In a Python shell against an in-memory DB, I added the same film twice for one user; the first call succeeded and the second raised `AlreadyInWatchlistError` as intended. Then ran `pytest tests/ -v` to confirm no existing tests broke. (A dedicated automated test follows in Comment 3.)

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` following the fixture and assertion structure of `tests/test_collection.py`. Added `test_add_to_watchlist_nonexistent_film_raises`, the direct equivalent of `test_add_to_collection_nonexistent_film_raises`: it calls `add_to_watchlist` with a film_id that doesn't exist and asserts `FilmNotFoundError` is raised. Reused the same `app`, `sample_user`, and `sample_film` in-memory fixtures.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` (passes) and `pytest tests/ -v` (all 6 pass — 4 collection + 2 watchlist).

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## Stretch — Second test
I added a second watchlist test beyond what the review requested: `test_add_to_watchlist_duplicate_raises`. I chose the duplicate-entry edge case because deduplication (Comment 2) is the watchlist's most behavior-critical guard — a silent duplicate would corrupt the list and isn't caught by the nonexistent-film test. The test adds the same film twice for one user, asserts `AlreadyInWatchlistError` is raised on the second call, and confirms exactly one row exists in the database afterward, matching the assertion style of `test_add_to_collection_duplicate_raises`.

## PR Description
<!-- Written at the end -->

## Commit History
<!-- Screenshot of git log --oneline -->
