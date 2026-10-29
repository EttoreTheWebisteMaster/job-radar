# find-companies — the weekly company hunt

Rebuilds the profile from the CVs and grows the Board's company list. Rules are in `rulebook.md`; read it first.

1. **Build the profile.** List the `cvs` collection and read every CV's `text`.
   - Write `meta/profile` (schema in `schema.md`):
     - `headline`: one line naming the user's directions, e.g. "Frontend / full-stack developer"
     - `titles`: target job titles in every `searchLanguages`, from the CVs' headlines and roles, plus public-call profiles the CVs qualify for when `publicCalls` is on
     - `stack`: each skill "job" if a dated job shows it, else "project"
     - `datedYears`, `education`, `qualifications` (which degree-gated roles this admits or excludes), `languages`
     - `notClaimable`: requirements common in this market (including the niche and public calls when on) that the CVs do not show
     - `updated` today
   - Update only fields that changed, pinned with `if_version`.
   - If no CV can be read, keep the existing profile, say so in the reply, and continue.
   - Read `meta/settings` and list `deleted` once.

2. **Find companies.** Up to `searchesPerWeek` searches aimed at company lists, careers pages and calls pages, never news or funding articles. Use location words from the settings. Rotate, home city first:
   - product companies and startups in the home area (A)
   - design and web agencies in the home area (B)
   - companies in the user's country hiring remote (C, only when remote is accepted)
   - EU remote-first companies that hire in the user's country (D, only when remote is accepted)
   - European companies hiring with relocation in the tech hubs (R, only when relocation is on)
   - companies in `niche` near home and remote in the country (S, only when set)
   - public research bodies, universities and administrations near home via their calls pages (P, only when `publicCalls`)
   - ATS company pages (`site:jobs.lever.co`, `site:join.com`, `site:jobs.ashbyhq.com` with the home city or country) to surface employers
   - up to `directoriesPerWeek` sources whose `use` includes "Companies"

3. **Add.** For each candidate:
   - Skip it if its slug is in `companies` or in `deleted` (`companies--<slug>`).
   - Skip companies that can only hire outside the settings.
   - Fetch its careers page (rulebook, Fetching).
   - Add up to `newCompaniesPerWeek`, at least a third in the home area when enough exist, with: name, tier, status "watch", what, roles that fit the profile, site (always set), careers (or ""), city, mode, scope, modeNote, ats, public jobs email if shown, verified, `edited: []`.
   - Consultancies and staffing firms are tier X; relocation employers R; niche S; public bodies P.

4. **Verify.** Query up to `verifyPerWeek` companies with `verified` false, preferring empty `careers` or `site`. Find and fetch each careers page; set `careers`, `site`, `verified`, `lastChecked`, but never a field in `edited`. If a company clearly cannot hire for the profile, set status "no-fit" with a one-line note, unless `status` is in `edited`. If no careers page exists, start `notes` with "No careers page".

5. **Save and reply.** Update `meta/lastRun` {date, mode "companies", newCompanies}. Reply in 3 lines or fewer, in `language`: profile updated or not; companies added (names, home area first); companies verified and any marked no-fit.
