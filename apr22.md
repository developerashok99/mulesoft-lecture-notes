# Apr 22 — Full Day Notes

## Session 1: Quick Recap (Very Short Clip)

### Topics Covered
- Brief basics recap — this is a short (~3 minute) clip, likely the start of a session before the main recording begins.

### Notes
- Quick Q&A on MuleSoft fundamentals:
  - MuleSoft's purpose: **integrating applications/systems** (rather than building applications from scratch).
  - The IDE used for Mule development is **Anypoint Studio**.
  - Confirmed the version of Mule being used in the course: **Mule 4**.

*(This session is very short — most of the day's substantive content is in `apr22-02.md`.)*


---

## Session 2: Full Interview Rapid-Fire Recap

### Topics Covered
Rapid-fire recap covering nearly the entire course so far — a strong single-file interview cheat sheet.

### Scatter-Gather vs. Round-Robin
- **Scatter-Gather:** sends the **same data to multiple targets in parallel**, then aggregates responses.
- **Round-Robin:** sends data to targets **sequentially**, one at a time, cycling through them.

### Error Handling Levels
Three levels where error handling can be configured:
- **Flow level**
- **Global level**
- **Process level** (e.g. Try scope)

Behavior of the two global handler types:
- **On Error Continue** — flow **continues** after the error, returns **HTTP 200**.
- **On Error Propagate** — flow **stops/trips**, returns **HTTP 500**.

Best practice emphasized: **global handling alone is not sufficient** — combine it with **process-level** handling (e.g. a Try scope around risky logic) inside your main flow(s) for proper coverage. Recommended structure: **two main flows** — one for the primary logic, one acting as a process-level wrapper/try block.

### Batch Job vs. For Each (recap)
| | For Each | Batch Job |
|---|---|---|
| Threading | Single-threaded | Multi-threaded (**16 threads default**) |
| Execution | Sequential | Parallel |
| Default size | 1 | 100 (customizable based on record volume) |
| Stages | — | Batch Step, Batch Job, On Complete |
| Best for | Small volumes | Large volumes (lakhs/millions) — completes much faster |

### Securing Sensitive Data (Encryption)
- Use an encryption algorithm (e.g. **AES**) with a secure key (example given: 16-character key) to encrypt sensitive values (passwords, secrets) in property files.
- **Syntax to read an encrypted value:**
  - In a **connector config field**: `${secure::keyName}`
  - In a **Transform Message (DataWeave)**: `p('secure::keyName')`
- **Syntax to read a normal (non-encrypted) property value:** `${keyName}`

### Notifications & Messaging (recap)
- **Email/Notification connector** — used to trigger email alerts.
- **Anypoint MQ** — recap of Pub/Sub async model, Queue vs. Exchange, and how to create both in Anypoint Platform.

### Files & Object Store (recap)
- File reading follows **FIFO** (First In, First Out).
- **Object Store**: max retention 30 days; **Persistent** flag prevents data loss when the app goes into an unresponsive/downtime state; Object Store consumes **runtime memory**.

### DataWeave Functions — Must-Know List
Interview framing: **know at least 10 DataWeave functions cold** — failing to explain Transform Message functions is called out as a likely reason to not get selected. Core list mentioned:
- `map`
- range function (e.g. `[0 to 3]`)
- `sizeOf`
- `splitBy`
- `replace`
- `pluck`
- `flatten`
- date functions

### Other Recap Points
- **Throttling / Rate Limiting / SLA-based policies** — can be tested internally via MUnit rather than only via live calls.
- **Property files** — reading config dynamically from property files (including pointing to different property files per environment).
- **POM file** — holds the project's **Maven dependencies**.

### Interview Framing
- The mock interview discussed here runs ~25 minutes; if you can speak fluently and confidently across the topics above, that's described as more than sufficient to pass.
- Remaining topics for the course: **Fragments** (important, don't skip), **Bitbucket**, **Jenkins** — final topics closing out the training.

