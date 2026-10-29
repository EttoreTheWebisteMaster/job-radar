# Setup — the agent protocol

You are setting up Job Radar for the person you are talking to. Run the phases in order. Each phase ends with a checkpoint; do not start the next phase until it passes. Keep the user informed with a task list (one task per phase) when the task tool exists.

Tone: a sharp recruiter-friend, not a form. Ask few questions, propose answers from the CV, let the user correct. Use AskUserQuestion when available (max 4 questions per call, recommended option first), otherwise ask in plain text. Speak the user's language from their first message.

Nothing in this phase list is irreversible until Phase 4. Before Phase 4 you show one plan and get one clear "yes".

---

## Phase 0 — Preflight (silent, ~1 minute)

1. **Get the files.** If this skill is installed, its folder already has everything. Otherwise clone the release named in the user's link or in `assets/release.json`:
   `git clone --depth 1 --branch <version> https://github.com/<repo>.git job-radar` (fall back to the default branch if the tag is missing). Work from `job-radar/skills/job-radar/`.
2. **Check tools.** You need: `Artifact` (publish with `capabilities`), `ArtifactData`, web search and page fetching, and a scheduling tool (`create_trigger` or similar; search with ToolSearch). Nice to have: the **Tavily** connector (better page reading; the Board's "Check and fill in" button uses it) and push notifications.
3. **Check for an existing Board.** List the user's artifacts. If one is titled "Job Radar Board", stop and ask: update it (run `update.md`), or start a second one.

Checkpoint: if Artifact, ArtifactData or scheduling is missing, tell the user in one line what is missing and that Job Radar needs claude.ai with artifacts and scheduled tasks, then stop. If only Tavily is missing, note it and continue; offer to connect it at the end.

## Phase 1 — The CV (first question, always)

Open with one short paragraph: what Job Radar will do (finds roles every night, scores them against their CV, keeps them on a private board, reminds them to apply), and that setup takes about 10 minutes. Then ask for the CV:

> "Send me your CV (PDF, Word or pasted text). If you keep different CVs for different kinds of roles, send them all."

When the files arrive:
1. Read every CV fully. Extract plain text for each.
2. Build a draft understanding (do not save yet): name of the role direction(s), dated jobs with years, stack split into "used in a dated job" vs "projects/courses only", education and whether it is a degree, languages and levels, the city on the CV, seniority.
3. Decide how many **CV profiles** to propose. One profile per CV the user sent. If they sent one CV but it clearly supports two different directions (for example frontend and UX/UI), propose splitting into two profiles scored separately from the same text, and say why in one line. Never propose more than 4.

Checkpoint: at least one CV read. If a file cannot be read, say which and ask for a paste.

## Phase 2 — Tailor (2 rounds of questions, max)

**Round 1: what and where.** Ask, with your proposals pre-filled from the CV:
- **Target roles** — "From your CV I'd search for: <3-6 titles>. Right?" (options: yes / add more / change)
- **CV profiles** — "I'll score every role against: <profile list with one-line focus each>." (options: yes / merge into one / add one)
- **Home base and modes** — "You're based in <city from CV>. Which do you accept?" (multi-select: on-site at home, hybrid at home, remote)
- **Beyond home** — "Remote roles must be open to people in: <country> / all of Europe / anywhere" and "Would you move? no / only to some cities / anywhere in Europe"

**Round 2: rules and rhythm.** Ask only what the CV did not answer:
- **Contract types** — permanent, fixed-term, freelance (multi-select)
- **Daily goal** — applications per weekday (recommend 3 for a first job search while employed, 5 if searching full time)
- **Language** for notes, reviews and notifications (default: the language they write in) and search languages (default: the local language plus English)
- **Optional extras** (multi-select): a niche sector to watch (ask which), public-sector and research calls, companies or keywords to exclude, a maximum years-of-experience cutoff

Also infer without asking: `timezone` from the city, `nearby` towns (propose 3-6 commuter towns and show them in the plan), `skipInternships` true unless the CV is a student's.

Checkpoint: every field of `meta/settings` in `schema.md` has a value or a deliberate empty.

## Phase 3 — The plan (one message, one "yes")

Show a compact plan the user can approve in one tap:

```
Job Radar plan
Searching for: <titles>
Scored against: <profile A> · <profile B>
Where: <city> (+ <nearby>) on-site/hybrid · remote open to <scope> · relocation <rule>
Contracts: <list> · Goal: <n>/weekday · Language: <lang>
Extras: <niche / public calls / excludes / max years or "none">

I will now:
1. Create your private Job Radar Board (a page only you can open)
2. Save your CV text and settings on it
3. Build your profile and find the first companies (~5 min)
4. Schedule: find jobs nightly at 04:4x, find companies Sundays 02:3x,
   evening reminder weekdays 18:0x, weekly review Sundays 19:1x (your time)
```

Ask: "Go?" Options: Go / change something. Loop back only on what they change.

Checkpoint: explicit yes.

## Phase 4 — Build

1. **Publish the Board.** `Artifact` publish with `file_path` = `assets/board.html`, `capabilities` = the JSON in `assets/capabilities.json`, `icon` "radar", `description` "Your job search: roles found every night, scored against your CVs." If the publish refuses the `mcp` block (Tavily or Claude Code Remote not connected), publish again without that server and note that the related buttons will be inactive. Keep the returned URL: it is the **Board URL**.
2. **Write the settings and seeds** with one `ArtifactData` batch (new documents need no `if_version`):
   - `meta/settings` from Phases 2-3 (schema in `schema.md`), `updated` today
   - `meta/goals` {dailyTarget, weekdaysOnly: true, learn: true, updated}
   - one `cvs/<key>` per profile: {key, label, focus, fileName, text, updated}. Several profiles may share the same text.
   - every document in `assets/seed/sources.json` into `sources`, plus 2-4 job boards that matter in the user's country (status "untested")
3. **Build the profile and first companies now**: run the `find-companies` mode in this conversation (`references/find-companies.md`). It writes `meta/profile` and the first companies.

Checkpoint: `ArtifactData` list of `meta`, `cvs`, `sources` and `companies` returns the documents just written. If a write failed, retry it once, then tell the user exactly what is missing.

## Phase 5 — Schedule

Create the four scheduled tasks in `assets/schedules.md`. For each: fill `{BOARD_URL}`, `{REPO}`, `{VERSION}` (from `assets/release.json`), use the user's `timezone` in `CRON_TZ=`, turn on push notifications, and name it exactly as written ("Job Radar: …" — the Board finds its tasks by that prefix). If this skill is installed in the user's account, the prompt uses the skill; otherwise it uses the clone line already in the template. Do not create duplicates: list existing tasks first and skip names that exist.

Checkpoint: list the scheduled tasks and confirm all four exist and are enabled. Say in one sentence which approval setting they got; if runs will ask for approval, tell the user to switch each task to automatic approval in its settings, because nobody is there at 4 am to approve.

## Phase 6 — Hand-off

1. `Artifact` open the Board.
2. Send a short message:
   - what was set up (one line each: Board, profile, N companies, 4 schedules)
   - the daily loop: "Each morning open the Board → Today. Apply to the top role, mark it Applied. Skip what's not for you — the radar learns from both."
   - two Board features: paste any job link into the bar at the top to score it; "Check and fill in" on a role re-reads the posting
   - what's missing, if anything (Tavily not connected, approvals needed)
3. Offer one next step: run the first `find-jobs` now instead of waiting for tonight. If yes, run it in this conversation.

Never paste the Board URL in chat unless asked; the artifact card carries it.
