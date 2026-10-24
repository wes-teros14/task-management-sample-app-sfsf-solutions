# CLAUDE.md

Inherits all rules from global `~/.claude/CLAUDE.md`.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Part 1: Project and Architecture

## What this is

SAP sample extension app for SAP SuccessFactors, deployed to SAP BTP Cloud Foundry. It manages "employee onboarding" tasks stored as SuccessFactors `TodoEntryV2` entities. Intended as a proof of concept, not production code.

## Commands

* Build (from repo root): `mvn clean install`, produces `task-management-app/target/task-management-app.jar`
* Deploy: `cf push --vars-file vars.yml` (fill in `ID` and `REGION_HOST` in `vars.yml` first)
* One-time CF service setup (instance names must match `manifest.yml`):
  * `cf create-service destination lite destination`
  * `cf create-service sap-successfactors-extensibility api-access sap-successfactors-extensibility -c sap-successfactors-extensibility.json` (set `systemName` to the registered SFSF system)
  * `cf create-service xsuaa application xsuaa -c xsuaa.json`

There are no tests and no linter configured. The app cannot run meaningfully locally: `DestinationProvider` throws at startup unless a `sap-successfactors-extensibility` service is bound (via `VCAP_SERVICES`), and every endpoint requires an XSUAA JWT.

Toolchain: Java 8 (README says JDK 8 to 15), Spring Boot 2.2.1, SAP Cloud SDK 3.46.0, Lombok (needs IDE plugin for IDE builds).

## Architecture

Two CF apps (see `manifest.yml`):

1. **`approuter/`**: `@sap/approuter` (Node). Handles login via XSUAA and forwards all traffic (`xs-app.json`) to the `token-destination`, which points at the Java app with `forwardAuthToken: true`. Users always enter through this app.
2. **`task-management-app/`**: Spring Boot backend that also serves the SAPUI5 frontend from `src/main/resources/static/`.

### Backend request flow

`rest/*Controller` → `services/*Manager` (interfaces, impls in `services/impl`, package-private) → SAP Cloud SDK generated VDM services → SuccessFactors OData v2.

* **Generated OData client (VDM)**: the `odata-generator-maven-plugin` generates Java classes from `task-management-app/edmx/*.edmx` into `target/vdm` during `generate-sources`, under package `com.sap.core.extensions.taskmanagement.vdm`. These classes are not in source control; run a Maven build before the IDE can resolve them. To consume a new SFSF entity, add its EDMX to `edmx/` and register a `Default*Service` bean in `configurations/TaskManagementConfigurations.java` (service path `/odata/v2`).
* **Connectivity**: `destination/DestinationProvider` finds the bound service of type `sap-successfactors-extensibility`, uses its *service name* as the destination name, and loads it through the Destination service (`ScpCfDestinationLoader`). Every OData call does `.executeRequest(destinationProvider.getDestination())`.
* **Upsert**: completing a ToDo uses the SFSF `upsert` function import, which the VDM doesn't cover. `odata/impl/CustomODataToDoEntryServiceImpl` builds a raw `ODataRequestAction` with Gson and sets `__metadata.uri` manually (e.g. `TodoEntryV2(<id>M)`).
* **Security**: `SecurityConfiguration` makes everything authenticated, stateless, JWT-validated via XSUAA. `UserInfoImpl` reads the current user's logon name from the token; it is used as the SFSF user ID for filtering/creating ToDos.

### Domain conventions (ToDoEntityManagerImpl / TaskManagementController)

* Only ToDos with category `"41"` (Generic Intelligent Services) owned by the current user are used.
* Status `2` = active, `3` = completed.
* New ToDo names are formatted `"<full name> (<userId>)"`; `GET /v1/onboardings` filters out entries that don't match this pattern (`TO_DO_NAME_PATTERN`), so changing the naming format breaks listing.

### REST API

* `/v1/onboardings`: GET list, POST create, `PATCH /{id}/complete`, `DELETE /{id}`
* `/v1/users`: GET all, `/current`, `/{userId}`, `/{userId}/photo`

### Frontend

SAPUI5 app (namespace `ext.samples.taskmanagement`, bootstrapped from the OpenUI5 CDN in `index.html`). XML views in `view/`, controllers in `controller/`, i18n in `i18n/message-bundle.properties`. `util/Config.js` holds the API base path (`v1`), which is relative so it works behind the approuter.

# Part 2: Workflow Rules

## Q&A log: `docs/qa-log.html`

Whenever I ask a question about how the code works, why something is designed the way it is, or you
explain project logic to me, **append it to `docs/qa-log.html`** before finishing your turn. It is a
single self-contained HTML file, grouped by topic (not chronological), and it is the reference I use
when explaining this project to other people.

* Add the question in my words, and a **short** answer: a few sentences, not the full chat reply.
  Include a concrete example, file path or measured number wherever one exists.
* Put it under the existing topic heading it belongs to; add a new heading only if none fits, and
  update the table of contents when you do.
* **Record corrections prominently.** If an earlier answer of mine turns out to be wrong, do not
  quietly replace it. Mark it as a correction and say what was wrong, in the amber callout style
  already used. Those entries are the most valuable ones in the file: they stop the same wrong
  conclusion being reached twice.
* Skip purely mechanical exchanges ("run this command", "what did that error say"). The file is for
  explanations that would otherwise have to be re-derived, not a transcript.

## Self-learning

When I correct you or you catch yourself making a mistake, before continuing, add the lesson as a one-line rule under #LESSONS so it never happens again.

## LESSONS

* Before calling backend behaviour a gap/bug, check how the frontend consumes it (e.g. completed ToDos are listed on purpose; UI shows "Completed").

## .env Files

Never read `.env` files directly. Assume they contain sensitive credentials.

If you need configuration information, use another approach first, such as inspecting the application's configuration code, identifying the expected environment variable names, or asking me for the required value instead of accessing the `.env` file.

## Deciding the Code Approach

When choosing an implementation, prioritize correctness, maintainability, readability, and long term quality over execution time or development effort. Do not reject a better solution solely because it is more complex or would take longer to implement. Since you are an AI, do not use human effort estimates as a deciding factor unless I explicitly ask you to optimize for development time or implementation cost.

## Session memory: `project_memory.md` and `handoff.md`

Both live in project root folder. Always read and write them there, don't create copies elsewhere. Together they replace re-explaining project history at the start of every session.

**`project_memory.md`** is the durable record. **`handoff.md`** is the current snapshot. They're not interchangeable: something belongs in one or the other, never both, and never neither.

### At the start of every session

1. Read `project_memory.md` and `handoff.md` before doing anything else, before opening any code, before answering the user's request.
2. Treat `handoff.md` as the most recent snapshot. Pick up exactly where it says work left off, don't re-derive it from git log or guesswork.
3. If something in the handoff is unclear, stale, or conflicts with the new request, ask before proceeding rather than silently picking an interpretation.

### At the end of every session, or whenever asked to "update memory" / "wrap up"

Update **both** files:

**`project_memory.md`: edit, don't append.**

* Long-term knowledge only: key decisions and *why* they were made, established conventions/naming/architecture, permanent project facts (goals, constraints, stack, stakeholders), and things tried and rejected (with the reason, so they don't get re-proposed later).
* Revise stale entries in place. An endlessly growing append-only log is not this file's job. If a decision changed, edit the entry that described the old one.
* Organize by topic/section, not chronologically. Keep a `Last updated: [date]` line at the top.
* Never put transient status here (e.g. "currently working on X"). That's `handoff.md`'s job, and putting it here means it never gets cleaned up.

**`handoff.md`: overwrite, don't append.**

* This file reflects only the *current* moment. Every update replaces its previous contents; it is never a running history.
* Include: date/time of update, what was just done this session, current state (what's working, what's broken, what's in progress), the immediate next step(s), and any open question the next session needs answered before continuing.
* A handful of short bullets per section, no paragraphs, no restating what's already durable enough to belong in `project_memory.md` instead.

### Git push at session end

After updating both memory files:

1. Push the current branch to dev.
2. Merge dev into main and push main too.
3. Always ask for confirmation before pushing. Never push automatically.

### Style rules for both

* Concise. No filler, no repeating content that already lives in the other file.
* Bullets and short headers, not prose.
* Never invent information to fill a gap. Mark anything uncertain as "unclear" rather than guessing, since a wrong guess here compounds every session that reads it afterward.

## Visual Explanations

Create a self contained HTML file under `/docs/visual/` whenever an explanation
involves several moving parts, layered structure, a sequence of steps, or
relationships that are hard to follow as plain prose. Do this proactively
without waiting to be asked.

**Rules**

* One file per topic. Descriptive kebab case names, e.g.
  `auth-token-lifecycle.html`, `deployment-environments-compared.html`
* Single file only. Inline all CSS and JS. No external stylesheets, CDN links,
  build steps, or asset folders. The file must render correctly when opened
  directly from disk with no network access.
* The visual is the primary artifact. Keep supporting prose short and place it
  around the diagram.
* Overwrite the existing file when revisiting a topic rather than creating a
  versioned copy.
* Reply in chat with the file path plus a two or three sentence summary. Do not
  duplicate the full explanation in the chat response.

**Skip this for** single concept questions, quick syntax lookups, and anything
fully answerable in a short chat reply.

## Proof of Concept Files

POC HTML files go in `/temporary_files`, not in `docs/` or the app source tree.

## Reporting Information

* Report to me extremely concisely. Sacrifice grammar for brevity.
* Never drop negations, numbers, or warnings. "Not deployed" must not become "deployed".
* Keep code, commands, and error messages exact.

## Handling ambiguity

If a request is ambiguous:

* **Expensive to undo** (API or schema changes, renaming public interfaces,
  edits across multiple files): ask exactly ONE clarifying question first.
* **Cheap to undo** (filenames, formatting, small choices): decide, proceed,
  and state the assumption in one line.
* **Specs and planning:** ask as many questions as needed, in one block.

## When stuck

* After two failed attempts, stop. Do not try a third.
* Tell me what's blocking you and what you've already ruled out.

## Evidence over assumptions

* Don't trust assumptions when verifiable information exists. Check first, then act.
* State all assumptions used in the response.
* Separate **verified evidence** (cite the source) from **reasoning or inference**.
* Evidence before confidence.