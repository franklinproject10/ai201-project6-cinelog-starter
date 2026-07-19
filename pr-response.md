# PR Response Doc — CineLog Watchlist Feature

## AI Usage

<!-- Fill in at the end -->

## Comment 1 — Rename

**What I did:** Renamed save_to_watchlist() to add_to_watchlist() in
services/watchlist_service.py and updated the call site in
routes/watchlist/watchlist.py.
**How I verified:** grep -r "save_to_watchlist" --include="\*.py" returned
nothing. pytest tests/ -v — all 4 tests still pass.

## Comment 2 — Deduplication

**What I did:** Added AlreadyInWatchlistError exception class to
watchlist_service.py. Added duplicate check in add_to_watchlist() following
the same pattern as add_to_collection() — query for existing entry, raise
AlreadyInWatchlistError if found.
**How I verified:** pytest tests/ -v — all 4 tests still pass.

## Comment 3 — Missing test

**What I did:** Created tests/test_watchlist.py with
test_add_to_watchlist_nonexistent_film_raises, following the same fixture
and assertion structure as test_add_to_collection_nonexistent_film_raises
in test_collection.py.
**How I verified:** pytest tests/test_watchlist.py -v — 1 passed.
pytest tests/ -v — all 5 tests pass.

## Comment 4 — Default visibility

**My position:** Keep public=True as the default.
**Reasoning:** The question isn't really "is public visibility good or bad" in the abstract — it's "what is CineLog actually for." The README calls it a community film tracking app, and a watchlist is the one piece of user data whose entire value depends on being seen by other people. A private-by-default watchlist is just a personal to-do list that happens to live inside a social app — nobody discovers anything from it, nobody gets a recommendation out of it, and the "community" part of "community film tracking app" doesn't touch it at all. If the goal is a platform where people find films through what other people are into, the watchlist has to be visible by default, because almost nobody will go find a visibility toggle before they've even used the feature once. Defaulting to private doesn't protect users so much as it quietly switches the feature off for everyone who never opens settings — which, realistically, is most people.
**Tradeoff acknowledged:** The real cost here is that "public by default" doesn't distinguish between a harmless watchlist add (a blockbuster everyone's seen) and one that reveals something a user didn't mean to share — a documentary tied to a health issue, a film with political or religious associations they'd rather not broadcast to followers. That user isn't a hypothetical edge case, they're just someone who added a film without thinking about who could see their list, because nothing in the add flow told them it mattered. I'm not going to pretend defaulting to public is risk-free — it isn't. What I think it is, is the right default for the common case, with the responsibility on us to make the exception easy: giving add_to_watchlist() an explicit public parameter so a user (or the UI on their behalf) can opt a specific entry out at the moment they add it, rather than discovering later that it was visible the whole time.

## Comment 5 — Sort order

**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase

**What conflicted:** .gitignore had an add/add conflict — both my branch
and main added a .gitignore at the same time with slightly different
ordering. WatchlistEntry was also missing from models.py after the rebase
— the UUID refactor on main replaced the whole file and dropped it.
**How I resolved it:** Merged both .gitignore versions keeping all entries.
Restored WatchlistEntry to models.py with film_id as db.String(36) instead
of db.Integer to match the post-refactor UUID convention.
**How I verified no conflict remains:** pytest tests/ -v — all 5 tests pass.
git log --oneline shows linear history with no merge commits.

## PR Description

<!-- Written at the end -->
