# Skills Repository 5-Minute Preflight Runbook

Use this before deep debugging.

## Skill Doc Standard Note

- Skill entrypoint is `SKILL.md`.
- This runbook is an operational convention (recommended), not a required skill file.
- SDK/API names can drift by version; validate current names against docs/raw-docs before release.

## 1) Confirm Integration Surface

- Confirm you are updating the correct product folder before editing any docs.
- Keep `SKILL.md` as the skill entrypoint in every skill directory.
- Use this runbook when adding/removing docs, links, references, or triggers across skills.

## 2) Confirm Required Credentials

- No runtime secrets are required for doc maintenance.
- If examples include credentials, ensure they reference `.env` keys, not hardcoded values.
- Never commit local absolute paths or machine-specific endpoints.

## 3) Confirm Lifecycle Order

1. Update target docs (`SKILL.md`, references, scenarios, troubleshooting) in the product folder.
2. Update cross-links from parent hubs (`skills/SKILL.md`, product root `SKILL.md`, or `general/SKILL.md`) if navigation changed.
3. Verify frontmatter (`name`, `description`, optional `triggers`) remains present and accurate.
4. Ensure no new orphan markdown files are left behind.

## 4) Confirm Event/State Handling

- Keep naming consistent with current product folder conventions.
- Keep related files (`references/environment-variables.md`, `architecture.md`, `runbook`) aligned when content moves.
- Track deprecations/renames explicitly so future updates are migration-safe.

## 5) Confirm Cleanup + Upgrade Posture

- Remove dead links after any file move or rename.
- Remove stale platform claims that no longer match current docs.
- Remove any machine-local absolute path references before finalizing.

## 6) Quick Probes

- Every product folder has a working `SKILL.md` entrypoint link from upstream hub docs.
- New files are reachable from at least one root/navigation file.
- Environment variable docs are present and referenced where expected.

## 7) Fast Decision Tree

- New page exists but no inbound links -> add links in parent `SKILL.md` and relevant hub/use-case docs.
- Routing confusion between products -> strengthen triggers and routing notes in `SKILL.md`.
- Conflicting docs after version updates -> keep old behavior in troubleshooting/deprecation notes, not main quick path.

## 8) Source Checkpoints

### Official docs

- https://developers.zoom.us/
- https://marketplace.zoom.us/
- https://devforum.zoom.us/

### Raw docs in repo

- `raw-docs/developers.zoom.us/docs/`
- `raw-docs/marketplacefront.zoom.us/sdk/`
- `tools/zoom-crawler/raw-docs/`
