# Apr 07 — Full Day Notes

## Session 1: Object Store Deep Dive

### Topics Covered
- Why/when to use Object Store
- Object Store vs. Anypoint MQ (limits comparison)
- Object Store operations
- Hands-on: store & retrieve POC
- Real use case: caching static data to avoid repeated external calls

### Why Object Store?
Object Store lets you persist data **inside the Mule runtime itself**, so you don't have to keep re-fetching the same unchanging ("static") data from an external system every time you need it.

Classic use case: your flow needs some **static/constant data** (e.g. from Salesforce) on every request, but that data itself rarely changes. Instead of calling the external system on every single request:
- Fetch it once (or on a schedule) and **store** it in Object Store.
- On subsequent requests, **retrieve** it from Object Store instead of hitting the external system again.
- Benefits: fewer unnecessary external calls, resilience if the external system is temporarily down, better performance.

### Object Store vs. Anypoint MQ — Key Limits

| | Anypoint MQ | Object Store |
|---|---|---|
| Max payload size | 10 MB | 10 MB (same limit) |
| Max retention | 7 days | **30 days** (default, configurable) |
| Consumes | — | **Runtime memory** (of the worker) |

- Both have the same 10 MB per-message/payload size limit.
- Object Store can hold data much longer (30 days default vs. MQ's 7 days), and that duration (TTL) is customizable per use case (e.g. 1 day, 2 days, 10 days) based on business requirements.
- Because Object Store data lives in runtime memory, it directly consumes the worker's memory allocation — a design tradeoff to keep in mind.

### Object Store Operations
Available operations in the Mule palette:
- **Store** — save a value under a key
- **Retrieve** — read back a value by key
- **Remove** — delete a specific key
- **Retrieve All Keys** — list all stored keys
- **Contains** — check if a key exists
- **Clear** — wipe all stored entries

#### Store / Retrieve mechanics
- Data is stored as a **key → value** pair (similar conceptually to a variable name → value).
- To **store**: give a *key* name and the value (e.g. the payload) to persist.
- To **retrieve**: reference the *same key* name to get the value back.
- Config also lets you set the retention duration — default is 30 days if left unset; can be customized.

### Hands-on POC walkthrough
1. Built a simple flow: HTTP Listener → **Store** (key = e.g. `car`) → stores incoming payload (e.g. `{car, price}`).
2. Built a second flow: HTTP Listener (different path, e.g. `/getcar`) → **Retrieve** (same key) → returns the previously stored payload, *without* needing any new input data.
3. Demonstrated: POST data once to the store endpoint, then GET from the retrieve endpoint with no body — the stored value comes back correctly.
4. Store and Retrieve can be used **across different flows and even different applications** within the same organization, as long as they use the **same key name** (since the data lives in the shared runtime-level store, shown as accessible org-wide on the platform in this trial account setup). Using distinct key names avoids collisions between unrelated data.

### Real-World Use Case Discussed
**Problem:** A flow needs to combine data from:
- The source system (dynamic, changes per request — e.g. `car`, `price`)
- An external system like Salesforce (**static** — e.g. `country`, always `"India"` in the example)

...then send the combined payload to a target system.

**Naive approach:** call Salesforce on every single request just to get a value that never changes — wasteful, and fails entirely if Salesforce is temporarily down.

**Better approach using Object Store:**
1. A separate **scheduled flow** (e.g. every 30 days, matching Object Store's default TTL) periodically calls Salesforce once and **stores** the static result under a key (e.g. `data`).
2. The **main request flow** (triggered per incoming request) **retrieves** that key from Object Store instead of calling Salesforce directly.
3. Result: Salesforce is called roughly once per schedule interval instead of once per request — far fewer external calls, and immunity to brief Salesforce outages.

This pattern — **cache static/reference data locally via Object Store, refresh it on a schedule** — is the main practical takeaway.

### Key Takeaway
> Object Store's core purpose: avoid unnecessary repeated calls to external systems for data that doesn't change often, by caching it inside Mule itself.


---

## Session 2: Object Store Use Case, Target Variable, Mule 3 vs Mule 4 Terms, MUnit Intro

### Topics Covered
- Full end-to-end Object Store use case (source + external + target systems)
- Target Variable concept (avoiding payload overwrite)
- More Mule 3 → Mule 4 terminology mapping
- Object Store "persistent" flag (teaser)
- Introduction to MUnit testing (why it matters)

### End-to-End Scenario: Combining Multiple Data Sources
**Requirement:** Build a flow that:
1. Receives `car` and `price` from the source system (dynamic data, via POST).
2. Calls an external app (e.g. Salesforce-like system) via a GET request to fetch `country` (static data — always the same, e.g. `"India"`).
3. Combines `car`, `price`, and `country` into one payload.
4. Sends the combined payload to a target system.

**Initial (naive) design:** call the external app directly inside the main request flow every time — flagged by the instructor as a bad approach, since `country` is static and doesn't need to be fetched on every request.

**Improved design (Object Store caching pattern):**
- A separate scheduled flow calls the external app periodically and **stores** the static value in Object Store.
- The main flow **retrieves** the cached value instead of calling the external app directly.
- Reduces unnecessary external calls and avoids failures if the external app is temporarily down.

### Problem Encountered: Payload Overwrite
When calling a second connector (e.g. the external app) inside a flow, its response **overwrites the current `payload`** by default — so the original source-system data (e.g. `car`, `price`) is lost by the time you reach the Transform step, unless handled properly.

#### Solution: Target Variable
- Use the connector's **Advanced → Target Variable** option.
- This stores the connector's response into a **named variable** instead of overwriting `payload`.
- The original `payload` remains untouched, and the new response is accessible via `vars.<variableName>`.
- In the Transform Message step, you then reference both: the original `payload` fields (e.g. `payload.car`, `payload.price`) and the variable (e.g. `vars.countryVar`) to build the combined output.

#### Terminology: Mule 3 vs Mule 4
| Mule 3 | Mule 4 |
|---|---|
| Message Enricher | **Target Variable** |
| Secure Property Placeholder | Secure Property (Config) |
| Poll scope | **Scheduler** |

> These Mule 3 → Mule 4 naming/concept changes come up frequently in interviews — know them cold.

### Object Store: "Persistent" Flag (teaser)
Briefly mentioned that Object Store has a **persistent** setting affecting whether stored data survives application restarts/unresponsive states — full explanation deferred to the next session (see `apr08-01.md`).

### Introduction to MUnit Testing
Motivation discussed (details/hands-on covered in `apr08-01.md` and `apr08-02.md`):
- Testing a flow via Postman sends **real requests to the real target system** every time — risky and wasteful if you're testing repeatedly (could be triggering thousands of real calls, some of which might not even be accepted, e.g. bad test payloads).
- **MUnit** lets you test a Mule flow's internal logic **without** actually connecting to real source/target systems.
- Key idea: **mock** the connectors (stop them from making real calls) and manually supply expected test payloads.

### Task Assigned
- Rebuild the flow to remove the direct external-app call from the main request path and instead retrieve the cached value from Object Store (per the caching pattern above).
- Use a scheduled flow (e.g. every 30 days) to refresh the cached value.

