# weekly-review — Sunday evening

Updates what the radar learned and writes the week's review. Never invent a number: everything comes from the `jobs` collection.

1. Read `rulebook.md` (Learning and Sunday review). Read `meta/goals`, `meta/settings`, `meta/learning`, `sources` (for names) and `jobs` (limit 1000). Use dates in `meta/settings.timezone`. This week = Monday to today.
2. Compute this week's numbers and the all-time Learning numbers per dimension and group, with the rulebook's formulas, plus `titleSignals`.
3. Write `meta/learning` (set; pin with `if_version` if it exists): updated, by "weekly run", overall rates, boosts, notes (1-2 sentences), titleSignals.
4. Week id = ISO week, e.g. `2026-W44`. If `reviews/<week>` exists with `by` "you", update only its `stats`. Otherwise set it with week, from, to, stats, createdAt, by "weekly run", and the review text (rulebook, Sunday review).
5. Update `meta/lastRun` {date, mode "review"}.
6. Reply with one line in `language` — it is the push notification. Example: "Week: 18/25 applications, 3 goal days, 1 interview. Home-area roles work best. Read the review on Job Radar."
