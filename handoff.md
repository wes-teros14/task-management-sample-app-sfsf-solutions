# Handoff

Updated: 2026-10-23

## Done this session
- Build fixed (JDK 11), deployed both apps to trial CF (java_buildpack); app works end to end.
- Verified create → SF (ToDo 257507). Tried category 19 → SF 500 → reverted to 41.
- Wrote `docs/qa-log.html`, `docs/visual/backend-request-flow.html`, `docs/visual/poc-ideas-data-quality-and-mdf.html`.
- Pushed `CLAUDE.md` to fork `main` (b5f5d82).

## Current state
- Category `"41"` in source. Redeploy after revert assumed (user reported "working now") — unclear if `cf push` ran.
- Uncommitted: `manifest.yml` (buildpack fix), `CLAUDE.md` (lesson), `docs/`, `project_memory.md`, `handoff.md`.
- Never commit: `vars.yml`, `sap-successfactors-extensibility.json` (personal values).
- No `dev` branch exists yet (local or origin).

## Next steps
- User to choose next POC (#1, #3 or #4); then write spec to `tasks/todo.md` before coding.
- Optional: move to `cflinuxfs5` before 2027-04-12.

## Open questions
- Why ToDo 257507 isn't visible on David's SF home page.
- Push flow: create `dev` from `main`? (CLAUDE.md requires push to dev, then merge to main.)
