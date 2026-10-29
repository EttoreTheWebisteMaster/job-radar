# Board data schema

The Board is a private claude.ai artifact with a database (`db` capability). Everything Job Radar knows lives here, so a scheduled run needs only the Board URL. Read and write it with the `ArtifactData` tool (load it with ToolSearch). Pin every write to an existing document with `if_version`.

Dates are `YYYY-MM-DD` in the user's time zone (`meta/settings.timezone`). Ids are deterministic slugs: lowercase, `a-z0-9-`, max 80 chars.

## meta (collection `meta`)

### meta/settings — written by setup, edited by the user on the Board
| field | type | meaning |
|---|---|---|
| homeCity | string | City the user lives in, e.g. "Lyon" |
| country | string | Country, e.g. "France" |
| nearby | string[] | Towns that count as home |
| modes | ("on-site"\|"hybrid"\|"remote")[] | Work modes accepted |
| remoteFrom | "Country"\|"Europe"\|"Anywhere" | Remote roles must be open to people living in… |
| relocate | "home"\|"cities"\|"europe" | On-site/hybrid outside home: never, only `otherCities`, anywhere in Europe |
| otherCities | string[] | Used when relocate is "cities" |
| language | string | Language for notes, reviews and notifications, e.g. "Italian" |
| searchLanguages | string[] | Languages to write search queries in, e.g. ["Italian","English"] |
| timezone | IANA string | e.g. "Europe/Paris" |
| maxYears | number\|null | Drop postings requiring more years than this; null = no limit |
| contracts | string[] | Accepted: "permanent", "fixed-term", "freelance" |
| skipInternships | boolean | Default true |
| cvs | {key, label, focus}[] | One entry per CV, e.g. {key:"FRONTEND", label:"Frontend", focus:"frontend developer CV"}. Keys are UPPERCASE A-Z. |
| niche | string | Optional sector for tier S, e.g. "space and satellites", "climate tech", "games". Empty = no tier S |
| publicCalls | boolean | Look for public-sector calls and research jobs (tier P) |
| exclude | string[] | Companies or keywords to never show |
| runSizes | object | Optional overrides, see rulebook "Run sizes" |
| updated | date | |

### meta/goals — edited by the user on the Board
`dailyTarget` (number, default 5), `weekdaysOnly` (boolean, default true), `learn` (boolean, default true), `updated`.

### meta/profile — written by `find-companies` from the CVs
| field | type | meaning |
|---|---|---|
| headline | string | One line, e.g. "Frontend / full-stack developer" |
| titles | string[] | Target job titles in every `searchLanguages` |
| stack | {skill: "job"\|"project"} | "job" only if a dated job shows it |
| datedYears | {skill: number} | Years of dated professional use |
| education | string | |
| qualifications | string | Which degree-gated roles this admits or excludes |
| languages | string[] | With level |
| notClaimable | string[] | Common market requirements the CVs do not show |
| updated | date | |

### meta/learning — written by `weekly-review`
`updated`, `by`, `overall` {chooseRate, replyRate}, `boosts` {dimension: {group: integer -6..6}}, `notes`, `titleSignals` {apply: string[], skip: string[]}.

### meta/lastRun
`date`, `mode` ("jobs"\|"companies"\|"review"), `checked`, `newRoles`, `newCompanies`.

## cvs — one document per CV, written by setup
id = the CV key in lowercase. Fields: `key`, `label`, `focus`, `fileName`, `text` (full plain text extracted from the CV), `updated`.
The CV text is the only source of truth about the user. Nothing outside it is claimable.

## companies
id = slug of the name. Fields: `name`, `tier` (A B C D R S P E X), `status` ("watch"\|"hot"\|"applied"\|"contacted"\|"no-fit"), `what` (one line), `roles` (string), `site` (https homepage), `careers` (https URL or ""), `ats`, `email`, `city`, `mode`, `scope`, `modeNote`, `verified` (bool), `lastChecked`, `openRoles` (number), `notes`, `contact` (user only), `edited` (string[] of fields the user changed — never overwrite these).

## jobs
id = `<company-slug>--<role-slug>` (public calls: `<ente-slug>--<call-number>`). Fields:
`companyId`, `company`, `tier`, `role`, `url`, `city`, `mode` ("on-site"\|"hybrid"\|"remote"\|""), `scope`, `modeNote`, `contract`, `posted`, `deadline`, `firstSeen`, `cv` (a CV key), `fit` (0-100), `cvScores` {KEY: 0-100}, `flags` (one string of bracketed tags), `source` (a `sources` id, "company", "web" or "pasted"), `status`, `note`, `edited`.

Status values. Written by runs: "new", "near-miss", "older", "unlisted?", "closed". Set by the user only: "later", "skipped", "applied", "interview", "offer", "rejected".
User-only fields runs never write: `appliedAt`, `repliedAt`, `skippedAt`, `interviewAt`, `salary`, `followedUpAt`, `snoozeUntil`, `kit`, `checkedAt`, `contact`.

## sources
id = slug. Fields: `name`, `url`, `kind` (Job board, Aggregator, Remote board, Startup board, Design board, Staffing agency, Public calls, ATS, Company directory), `use` (string[]: "Jobs", "Companies"), `status` ("works"\|"partial"\|"untested"\|"blocked"), `lastTried`, `notes`, `found` (number of roles saved from it).

## deleted
id = `<collection>--<doc id>`, e.g. `companies--acme`. The user deleted it on purpose. Never re-add it.

## reviews
id = ISO week, e.g. `2026-W44`. Fields: `week`, `from`, `to`, `stats` {applied, goalDays, workdays, skipped, replies, interviews, found}, `text`, `createdAt`, `by` ("weekly run"\|"you").
