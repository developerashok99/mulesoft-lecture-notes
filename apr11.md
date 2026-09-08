# Apr 11 — Full Day Notes

## Session 1: Mock Interview Recap + DataWeave Practice + Global Error Handling

### Topics Covered
- Mock interview debrief (policies, Anypoint MQ, Circuit Breaker, DataWeave)
- DataWeave practice problems
- Cron expression for scheduled jobs
- Global Error Handling + MUnit for error paths

### Mock Interview Debrief
Recap of a practice interview and the expected answers:

**Policies applied to APIs:**
- Client ID Enforcement policy (secures API access via client ID/secret)
- Basic Authentication policy

**Anypoint MQ:**
- Queues and Exchanges are created in **Anypoint Platform** (MQ section).
- Be ready to explain/demo: creating a Queue, creating an Exchange, binding multiple Queues to a single Exchange.

**Handling a queued message when the target system is down (recap):**
1. Subscribe to the queue in **manual** acknowledgment mode.
2. On successful delivery to the target → send **ACK** (removes message from queue).
3. On failure → send **NACK** (message stays in queue, gets redelivered) — this is what feeds into Circuit Breaker behavior.

### DataWeave Practice Problems
Worked examples using `payload` transformations:

1. **Filter employees by city:**
   ```
   payload filter ($.city == "Bangalore")
   ```
2. **Concatenate name + ID into one field:**
   ```
   payload map (item) -> {
     nameId: item.name ++ item.id
   }
   ```
3. **Extract first 4 characters of a string** — use the **range/substring approach**, e.g. `name[0 to 3]`.
4. **Remove duplicate records** — use **`distinctBy`**:
   ```
   payload distinctBy ($.someField)
   ```

Instructor's framing: for any DataWeave requirement, identify — filter? map/transform shape? substring/range? dedupe? — and pick the matching operator.

### Cron Expression Exercise
Requirement: run a scheduler **only on weekdays (Mon–Fri), at 12:00 PM**, not on weekends.
- Approach: build a Cron expression selecting Monday–Friday and the 12:00 hour, explicitly excluding Saturday/Sunday.
- Interview tip: be able to construct Cron expressions live, not just recognize them.

### Global Error Handling + MUnit for Error Paths
Extends the MUnit work from Apr 08 to explicitly test **error/exception handling**, not just the happy path.

#### Setting up Global Error Handling
1. Create a **Global Configuration** for error handling (a reusable, project-wide error handler rather than per-flow).
2. This lets you define how the app responds to error types across all flows without repeating logic in each one.

#### Testing error handling with MUnit
- In the MUnit test, use the **mocked connector's "Error" option** to deliberately **raise an error** (e.g. force the mocked requester to throw an error) instead of returning a normal mocked response.
- You can raise a **generic/ANY** error type to hit a catch-all global handler, or a **specific error type** (e.g. `HTTP:TIMEOUT`, `HTTP:CONNECTIVITY`) to route to a specific handler branch if your global config has multiple typed handlers.
- Build **one MUnit suite per meaningful error scenario** (e.g. one for ANY/global, others for specific error types) to get full coverage of the error-handling logic, similar to how you built suites for the happy path.
- Run via right-click on `src/test/munit` → **MUnit → Run Tests** to execute all suites and check coverage %.

### Task Assigned
- Implement **global error handling** in the project.
- Write MUnit tests for **both**:
  1. The main (happy path) flow.
  2. The error handling path (deliberately raising errors via mocks).
- Framed as a high-priority interview topic — practice, don't just understand in theory.


---

## Session 2: Task Recap (Error Handling + MUnit) — Short Session

### Topics Covered
- Recap/clarification of the error-handling + MUnit task from Session 1

### Task Recap
- Everyone should implement error handling in the main flow (per the global error handling approach from Session 1).
- Next session: come prepared with laptops — each person will implement Transform Message (DataWeave) logic live, one by one, via screen share.
- Deliverable: **two MUnit test suites** —
  1. One covering the main flow (happy path).
  2. One covering the error-handling path.

### Q&A Clarifications
- **"Can you throw an error from the source (listener)?"** — No. Errors can only be deliberately raised/mocked at the **project/connector level** (e.g. on a Requester), not from the Listener/source side.
- **"If the request is mocked, how does it still show as connected/covered?"** — Mocking a connector doesn't disconnect it from the flow graph; it just intercepts the call and returns a controlled (mocked) outcome instead of making a real call. The flow's structure — and therefore MUnit's coverage measurement — still treats it as part of the executed path.

