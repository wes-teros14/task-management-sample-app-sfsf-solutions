# Project memory

Last updated: 2026-10-24

## Purpose
- Fork of SAP sample `SAP/task-management-sample-app-sfsf-solutions`; used to learn BTP + SF side-by-side extensions, then build further POCs.
- Fork: `origin` = github.com/wes-teros14/task-management-sample-app-sfsf-solutions; `upstream` = SAP repo.

## Environment (verified 2026-10-23)
- BTP trial: org `63f955d7trial`, space `dev`, region `ap21` (`REGION_HOST: ap21.hana.ondemand.com`).
- SF system: `apisalesdemo8.successfactors.com` (demo, only system). Registered in BTP as "SFSF Demo".
- Test user: `davidm.epiuse` (David McMillan). XSUAA logon name == SF userId (required by app).
- Destination auth: `OAuth2SAMLBearerAssertion` (principal propagation), auto-created by `sap-successfactors-extensibility` instance.
- App URL: `https://task-management-web-S0024548536.cfapps.ap21.hana.ondemand.com`.

## Build / toolchain decisions
- Build JDK = Temurin 11 portable at `%USERPROFILE%\.jdks\jdk-11.0.32.1+1`; user-level `JAVA_HOME` set to it.
  - Why: JDK 26 breaks build (old Lombok from Spring Boot 2.2.1 + JDK 23+ no implicit annotation processing → missing VDM getters).
  - winget MSI install rejected: needs admin (exit 1602).
- Machine-level `JAVA_HOME` still points to non-existent `jdk-26`; harmless (user-level wins). Needs admin to fix.
- VS Code terminals keep stale env until VS Code fully restarted.

## Deploy decisions
- `manifest.yml`: `sap_java_buildpack` → `java_buildpack` + `JBP_CONFIG_OPEN_JDK_JRE: '{ jre: { version: 11.+ } }'`.
  - Why: `sap_java_buildpack` no longer exists on landscape; only `sap_java_buildpack_jakarta*` (for newer Java/Jakarta).
- Stack `cflinuxfs4` deprecated: new pushes end 2027-04-12, restage ends 2027-05-10; running apps unaffected. Move = `stack: cflinuxfs5` (java_buildpack there is v5.0.7, JRE config untested).
- `vars.yml` and `sap-successfactors-extensibility.json` hold personal values (S-user ID, system name): kept local, never committed (fork likely public).

## Tried and rejected
- ToDo category `"19"` instead of `"41"` (`ToDoEntityManagerImpl.java:32`): SF returned HTTP 500 on create. Reverted to `"41"`. Exact SF error never captured (app doesn't log SF response body). Assumption: SF API only supports creating category 41 — unverified.

## Known app behaviour (verified)
- Completed ToDos stay listed by design (UI shows "Completed").
- `GET /v1/users` returns all existing SF users, no paging → "onboarding" picks already-hired people.
- Created ToDo 257507 confirmed in SF via Postman, but not visible on David's SF home page — cause unclear.

## Docs
- `docs/qa-log.html`: Q&A log (per CLAUDE.md).
- `docs/visual/backend-request-flow.html`: app + BTP architecture explainer.
- `docs/visual/poc-ideas-data-quality-and-mdf.html`: detail for next POC options #3/#4.

## Future direction
- Next POC candidates (ranked): #1 real onboarding tracker, #3 data-quality checker (Job Scheduler), #4 custom MDF object app. Others: manager dashboard, event-driven (Event Mesh), AI assist, CAP rebuild.
- New apps: Spring Boot 3 / Java 17-21 / Cloud SDK 5, or CAP (works on trial). CAP reference: github.com/SAP-samples/cloud-sf-extension-cap-sample (exists; not reviewed).
- New app on same subaccount needs unique CF app names/routes and unique `xsappname` (current: `extension-app`); destination + extensibility instances can be shared.
