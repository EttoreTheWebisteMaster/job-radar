# find-jobs — the nightly search

Finds new postings that fit the user's CVs and saves them to the Board. Rules are in `rulebook.md`; read it first. This file lists the steps.

1. **Load state.** With `ArtifactData`, read `meta/settings`, `meta/goals`, `meta/profile`, `meta/learning` and `meta/lastRun`, and list the `deleted` collection once.
   - If `meta/profile` is missing, run step 1 of `find-companies.md` (build the profile from the `cvs` collection) and continue.
   - Count roles first seen in the last 7 days with fit 60+ inside the settings. If that averages under `dailyTarget` a night, shift budget as the rulebook's Daily goal says.
   - If `meta/learning` exists, plan tonight's budget with it (rulebook, Learning).
   - Do not read the `cvs` collection otherwise; `meta/profile` is enough for scoring.

2. **Company sweep.** Query `companies` for status "hot", then the oldest `lastChecked` (empty first), up to `companiesPerNight`. Skip tier E unless "hot", status "no-fit", and companies whose `notes` start with "No careers page" and were checked in the last 30 days. For each company:
   - Fetch its `careers` page (rulebook, Fetching).
   - Keep every title that matches `meta/profile.titles` and passes the location settings. Each fitting role becomes its own document. For tier P, match calls on profile and activities and apply the Public calls rules.
   - A role whose id or URL is already in `jobs`, or whose id is in `deleted`, is not new. A known role missing from a list that loaded fine becomes "closed" only if its status is new, near-miss, older or unlisted? and `status` is not in its `edited`. A call past its `deadline` becomes "closed" the same way. Never touch roles the user acted on.
   - Update the company: `lastChecked` today, `openRoles`, `verified` (true if the page loaded), and `careers`, `site`, `city`, `mode`, `scope`, `notes` when found. Never write a field in its `edited`, and never `status` or `contact`.

3. **Sources sweep.** Rulebook, Sources.

4. **Open-web sweep.** Up to `searchesPerNight` searches split as the rulebook's Search rotation says, weighted by `meta/learning`.
   - Aggregators are for discovery only.
   - A company found that is not in `companies` and not in `deleted` gets added: name, tier, city, mode, scope, modeNote, what, roles, site (full https homepage), careers if it loaded, verified, status "watch", `edited: []`.
   - When a new role is at a company that already has an open role on the Board, or the posting lists other roles, check that company's careers page for every other fitting role in this run.

5. **Score.** Open only postings that are new, relevant by title, and inside the settings, up to `postingsPerNight`. Verify and score each once per CV profile (rulebook, Verification and Scoring).

6. **Save.** Write each new role to `jobs` with every field in `schema.md` a run may write (add the company first if missing). Status "new" if fit ≥ 60, "near-miss" if 50-59, "older" if stale; skip below 50. `firstSeen` today, `edited: []`. Update `meta/lastRun` {date, mode "jobs", checked, newRoles, newCompanies}. Batch writes where you can.

7. **Reply, short — it is the push notification.** In `meta/settings.language`:
   - Line 1: "New match: <role> at <company> (<fit>)" if any new role has fit ≥ 70; else "Deadline soon: …" if a public call with fit ≥ 60 closes within 10 days; else "No new roles above 70; checked N companies, added M."
   - Then a table of new roles with fit ≥ 60, home area first, same company together: Role | Company | Where (city · mode) | Posted / Deadline | Fit | Link, with one line each on fit and the main gap.
   - One line counting sources that failed and public calls dropped for degree, if any. No empty sections.
