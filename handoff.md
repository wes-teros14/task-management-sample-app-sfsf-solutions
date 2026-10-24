# Handoff

Updated: 2026-10-24

## Done this session
- Build fixed (JDK 11), deployed both apps to trial CF (java_buildpack); app works end to end.
- Verified create → SF (ToDo 257507). Tried category 19 → SF 500 → reverted to 41.
- Wrote `docs/qa-log.html`, `docs/visual/backend-request-flow.html`, `docs/visual/poc-ideas-data-quality-and-mdf.html`.
- Pushed to fork: `CLAUDE.md` (b5f5d82), then all work (51de206); `dev` branch created.

## Current state
- Category `"41"` in source. Redeploy after revert assumed (user reported "working now") — unclear if `cf push` ran.
- All work committed (51de206) and pushed to fork `dev` and `main` (fast-forward).
- Never commit: `vars.yml`, `sap-successfactors-extensibility.json` (personal values).

## Next steps
- User to choose next POC (#1, #3 or #4); then write spec to `tasks/todo.md` before coding.
- Optional: move to `cflinuxfs5` before 2027-04-12.

## Open questions
- Why ToDo 257507 isn't visible on David's SF home page.