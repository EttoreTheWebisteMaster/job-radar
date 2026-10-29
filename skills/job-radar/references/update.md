# update — apply a new release to an existing Board

The Board's data lives in its database and survives a republish. Only the page and the scheduled task prompts change.

1. Find the Board (artifact titled "Job Radar Board") and read it with `Artifact` action read, so the publish is allowed.
2. Republish `assets/board.html` to the Board's URL (`url` set) with `capabilities` from `assets/capabilities.json`. Do not pass `icon`.
3. Read `meta/settings`. Add any field from `schema.md` that is missing, with its default, pinned with `if_version`. Never change a value the user set.
4. List scheduled tasks whose name starts with "Job Radar:". For each, compare its prompt with `assets/schedules.md` and update the prompt (and only the prompt) to the new template, keeping its schedule and the Board URL.
5. Reply in two lines: the release now running (`assets/release.json`), and what changed for the user (from the repository's CHANGELOG if available).
