# Apr 25 — Full Day Notes

## Session 1: HTTP Methods Recap + RAML Fragments, Resource Types, Traits, Types

### Topics Covered
- HTTP methods recap (GET/POST/PUT/PATCH/DELETE, and Upsert)
- RAML's four core reusability/validation concepts: **Fragments, Resource Types, Traits, Types**
- Walkthrough of a real RAML file
- API design requirements-gathering discussion

### HTTP Methods Recap
| Method | Purpose |
|---|---|
| **GET** | Retrieve data |
| **POST** | Create new data |
| **PUT** | Full **update** of a resource — can also **insert** if it doesn't exist (acts like an **Upsert**: update if exists, insert if not) |
| **PATCH** | **Partial** update of a resource |
| **DELETE** | Remove data |

- **Upsert** = "update or insert" — PUT effectively behaves this way when there's no existing data to update.

### Why RAML Reusability Constructs Exist
**Motivating scenario:** an API needs **30 resources**, and all 30 need to accept `POST`, the same media type, and similar header/body validation. Repeating that full configuration 30 times in one RAML file is wasteful and error-prone — RAML provides ways to factor out and reuse these chunks.

### 1. Fragments (a.k.a. "Collections")
- **Definition:** a **fragment is a reusable chunk of a RAML file, split out into its own separate file** and referenced from the main RAML.
- Sometimes called **"Collections"** in interviews — same underlying concept, different word choice.
- General mechanism: instead of repeating a block (methods, media types, etc.) inline many times, save it once in its own file and reference it wherever needed.

### 2. Resource Types
- **Definition:** the **resource-level reusability** piece — specifically for reusing **method definitions** across multiple resources.
- Use case: many resources all needing the same `POST` (or other method) behavior — define it once as a Resource Type, then apply it to each resource via `type: <resourceTypeName>` instead of rewriting the method block each time.
- Only really needed for **multi-resource APIs**; a single-resource API can just declare its method inline.

### 3. Traits
- **Definition:** used for **header-level validation** reusability.
- Example: common required headers like `Client-ID` and `Client-Secret` are defined once in a traits file (e.g. under a `commonHeaders` key), and applied to a resource via `is: [commonHeaders]`.
- Access/apply traits using the **`is`** keyword.
- Field requirement syntax:
  - `required: true` → header/field is mandatory.
  - A trailing **`?`** on a key name → marks it **optional** (equivalent to `required: false`).

### 4. Types (Data/Schema Validation)
- **Definition:** used to validate the **actual payload structure** (schema-level validation) — field names and data types (e.g. `integer` vs. `string`).
- If the incoming payload doesn't match the declared type/schema (missing required field, wrong type), RAML rejects it.
- Error code distinction:
  - **Wrong HTTP method** used → **405 Method Not Allowed**.
  - **Malformed/invalid payload** (schema mismatch) → **400 Bad Request**.
- String vs. number distinction in JSON/RAML examples: **double-quoted values are strings**; unquoted numeric values are **integers/numbers**.

### Example RAML Structure Walked Through
Top-level file sections:
- `baseUri` — mainly relevant for mock service enablement.
- RAML version declaration, `title`, `version`, `mediaType` (e.g. `application/json`), `protocols` (e.g. `HTTPS`).
- A `traits` folder containing a header-validation file (e.g. `Client-ID`, `Client-Secret` as required headers).
- A resource definition with:
  - `description` (optional, for readability/documentation — not mandatory).
  - `body` → `mediaType` → schema reference (a separate **Type** file) defining fields like `projectId` (integer, required), `projectName`, `projectNumber`, `salesforceOpportunityId` (all strings).
  - An **example** file showing a sample JSON payload matching that schema.

### API Design: Requirements-Gathering Discipline
Before writing any RAML, get **explicit confirmation** from the business/source stakeholder on:
- Final API name and resource names (don't guess/finalize unilaterally).
- Exact fields expected in the payload, and their data types.
- Which fields are mandatory vs. optional (drives `required: true` vs. `?` in RAML).
- Source system's data shape vs. what the **target system actually needs** — not all source fields necessarily need to be mapped/sent onward; extra/unneeded fields can be filtered out during transformation.
- **Connectivity pattern** to both source and target systems — is it API-based? A different protocol? This determines integration design choices early.

General process: **finalize requirements → design RAML → import into Anypoint Studio → build the flow logic** (mapping/transformation, connectivity).

Process hygiene reminder: track all requirement confirmations and decisions via **email or a Jira/Scrum board** so there's a documented trail — don't rely on verbal-only confirmations.

### Logistics
- Remaining topics: **Bitbucket** (next session) and **Jenkins**.
- Action item before the next session: create a **Bitbucket account** and install **Git Bash**.


---

## Session 2: Deployment Practices, Runtime Properties, Logging, Connectors Q&A

### Topics Covered
- Updating RAML vs. updating Mule flows safely
- Runtime Manager settings: auto-update, auto-restart
- Runtime properties vs. application properties (precedence)
- Logging levels and categories
- Code review / standards discussion
- Connector ecosystem and documentation-first approach

### Updating RAML vs. Mule Flows
- **RAML updates** are done in **Design Center**.
- **Mule flow updates**: never touch production directly.
  1. Deploy the updated application to a **non-production (sandbox/lower) environment** first.
  2. Test thoroughly there.
  3. Only after it's verified working, promote/deploy the change to **production**.

### Runtime Manager Settings Explained
- **Auto-update (e.g. scheduled for the 25th of each month):** if left unchecked/default, the platform can auto-update automatically on schedule. Recommended practice: don't blindly rely on this — proactively check whether an available update could impact your application before it auto-applies.
- **"Automatically restart application when not responding":** a separate app-level setting — if checked, an app that becomes unresponsive due to heavy load will attempt to **auto-restart/redeploy** instead of staying stuck in an undeployed state.
  - **Important distinction:** this is a **different concept** from Object Store's **Persistent** flag. Auto-restart is about the *application's* availability; Object Store Persistent is about whether *stored data* survives an unresponsive/downtime event. They're unrelated settings that both relate to "what happens when the app goes down," which is why they're easy to confuse.

### Runtime Properties vs. Application Properties (Precedence)
- Properties can be defined in two places:
  1. **Application properties** — packaged with the app (e.g. in `src/main/resources`), e.g. `http.port=8081`.
  2. **Runtime Manager properties** — set at the deployment/environment level (e.g. `http.port=8082`).
- **Runtime Manager properties take precedence** and override the packaged application properties when both are set.
- **Practical use case:** if a credential (e.g. a password) expires unexpectedly, you can update it directly via **Runtime Manager properties** without repackaging and redeploying the whole application — a fast, low-risk hotfix path for urgent credential rotations. The proper long-term fix is still to update the actual property file and redeploy, but this offers an immediate stopgap.

### Logging
- Standard log levels: **INFO, DEBUG, WARN, ERROR**.
- Logging can also be scoped by **category** (e.g. only show logs for a specific component/module) rather than just by level.
- Log levels and categories can be configured directly in **Runtime Manager** settings for a deployed app (no redeploy needed to change verbosity).
- Discussion note: setting log level to `ERROR` on a category where errors are handled gracefully (e.g. caught and not truly "erroring") means those events won't be visible in that category's logs — so log-level choice should match what you actually intend to observe.
- General industry practice mentioned: most teams run with a mix of **INFO** (and **DEBUG** when actively troubleshooting) rather than raw debug-only, balancing monitoring visibility against log volume — practice varies by organization/maturity.

### Code Review & Engineering Standards
- Teams without proper pipelines/MUnit/code repositories are considered to be **not following industry standards** — this varies a lot by company maturity.
- **Code review by architects** before production deployment checks: connector choices, flow design quality, and suggests more efficient/faster approaches where applicable.
- Whether junior engineers get to give feedback/improve legacy code depends on workload and team culture — not universally expected.

### Connector Ecosystem
- **Anypoint Exchange has 200+ connectors** available (Salesforce, NetSuite, OpenAir, Magento, and many more, each with their own specific operations).
- You don't need to know every connector — but should be genuinely proficient in the ones you actually work with.
- It's possible to **build custom connectors** for Mule, though this — along with formal **architecture** roles — typically requires separate certification/examination (framed as a more senior/specialized career path).

### Best Practice: Documentation-First for New Connectors
- Every published connector comes with **documentation** — check it **before** attempting configuration.
- Recommended approach when integrating a new/unfamiliar system's connector:
  1. Read the documentation to understand available operations and expected configuration.
  2. Attempt implementation based on that understanding.
  3. If issues arise, iterate (trial and error is acceptable *after* consulting docs, not instead of them).
  4. If a connectivity issue still can't be resolved, escalate to the connector/system's support or vendor.
- Blindly opening a connector and guessing at configuration without reading docs first is called out as a bad habit to avoid.

