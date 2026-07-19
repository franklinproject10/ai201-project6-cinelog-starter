# PR Response Doc — CineLog Watchlist Feature

## AI Usage

Used Claude to orient myself to the codebase — pasted add_to_collection()
and test_collection.py and asked for pattern explanations before writing
any code. Used Claude to stress-test my Comment 4 and Comment 5 arguments
after drafting them myself. Used Claude for terminal guidance throughout
(git rebase steps, conflict resolution). All design decisions and written
responses are my own reasoning.

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

## Comment 5 — Sort order

**My position:** Agree with the reviewer — switch get_watchlist() from
alphabetical to date_added descending, newest first.
**Reasoning:** Alphabetical sorting is a strange default for a list that's
fundamentally about intent and timing — "I want to watch this" doesn't have
anything to do with the letter a title starts with, and sorting that way
actively buries the thing a user just added under whatever happens to start
with "A." There's also a consistency argument specific to this codebase:
get_collection() already sorts by date_added descending. Watchlist and
collection are the same kind of object from the user's point of view — a
list of films tied to a date — and having one sorted newest-first while the
other is sorted alphabetically is an inconsistency with no explanation
behind it.
**Engagement with reviewer's point:** The reviewer's reasoning — "most users
want to see what they added recently" — is an argument about recency being
the useful signal, and I agree. The one case I considered: a film added
months ago and still unwatched gets pushed to the bottom. But that's not an
argument for a different default sort — it's an argument for a filter feature
the sort order alone can't solve. Flipping to oldest-first just moves the
same problem to whoever added something recently.

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
