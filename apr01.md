# Apr 01 — Full Day Notes

## Session 1: Interview Prep Recap + Circuit Breaker Logic

### Topics Covered
- Quick recap of interview-relevant topics
- Circuit Breaker logic (why and how)

### Interview Topics Recap
Quick-fire list of things to be ready to explain in interviews:
- Circuit Breaker logic
- Anypoint MQ: queues and exchanges
- DataWeave functions
- Securing sensitive info in property files (secure properties)
- Ways to handle errors in Mule (error handling strategies)
- **Message structure: Mule 3 vs Mule 4**
  - Mule 3: `inbound properties` / `outbound properties` → Mule 4: unified `attributes`
  - Mule 3: three kinds of variables → Mule 4: single `flow variable` (`vars`)
  - `payload` concept is the same in both

### Why Circuit Breaker Logic?
Problem scenario without circuit breaker, using Anypoint MQ:
- A message sits in a queue; a flow picks it up and tries to deliver it to a target system.
- Target system is down (e.g. for 2 hours).
- Delivery fails → flow goes to error handling → sends **NACK** (don't remove message from queue).
- Same message gets picked up again immediately → fails again → NACK again → infinite retry loop.
- At scale (thousands/lakhs of messages), this retry storm can:
  - Flood the target system with repeated pointless requests.
  - Overload the Mule app itself (excess requests vs. available workers/vCores).
  - Push the app into an **unresponsive / hung / undeployed state**.
  - Cause side effects like duplicate notification emails being fired repeatedly.

**Circuit Breaker logic exists to stop this retry storm** by temporarily "tripping" (pausing) the flow instead of hammering a known-down target system.

### Circuit Breaker Configuration
Three key settings (configured on the QoS/error-handling side of the connector, e.g. in "Edit inline"):

| Setting | Meaning |
|---|---|
| **Error Type(s)** | Which errors count toward tripping the breaker (e.g. `HTTP:TIMEOUT`, `HTTP:CONNECTIVITY`, unauthorized, etc.) — comma-separated |
| **Threshold** | How many times the *same* error must occur consecutively before the flow trips (a "peak limit" count) |
| **Trip Timeout** | How long the flow stays tripped (paused) once the threshold is hit |

#### How it behaves
1. Target system starts failing with a configured error type.
2. Once that error has occurred `threshold` times in a row, the flow **trips** — it stops consuming new messages for `trip timeout` duration.
3. After the timeout expires, the breaker releases **one** message as a test:
   - If it succeeds → all other queued messages are released normally.
   - If it fails → the flow trips again for another full timeout period.
4. This repeats until the target system recovers, avoiding wasted repeated calls while it's down.

**Example:** threshold = 5, trip timeout = 5 minutes. If the target throws the same timeout error 5 times in a row, the flow pauses for 5 minutes before testing again with a single message.

Only the **configured error types** trigger this behavior — any other (unexpected) error is not controlled by the breaker and will execute normally.

### Related Scenario: Bad Request / Invalid Payload
If the *message itself* is invalid (e.g. malformed JSON that the target system's validation always rejects), retrying it via Circuit Breaker alone doesn't fully solve the problem — retrying a bad payload 1000 times still never succeeds. This leads into the next topic: **Dead Letter Queue (DLQ)** and **max re-delivery attempts**, covered in the next session.

### Action Items
- Be ready to explain Circuit Breaker logic (error types, threshold, trip timeout) fluently for interviews.


---

## Session 2: Dead Letter Queue, Circuit Breaker Q&A, Intro to Object Store

### Topics Covered
- Dead Letter Queue (DLQ) in Anypoint MQ
- Circuit Breaker logic — recap/Q&A
- Task assigned: implement Circuit Breaker logic
- Introduction to Object Store (teaser for next session)

### Dead Letter Queue (DLQ)
Handles the "bad/invalid payload that will never succeed" case that plain Circuit Breaker retries can't fix.

- A **DLQ is a separate queue bound to your main queue** (e.g. main queue `test.queue` bound to `test.dlq`).
- Setting: **Max Re-delivery Attempts** — default is **10**.
- Behavior: the same message is re-delivered up to the max attempts (e.g. 10 times); on the **next** attempt after that (11th), the message is:
  - Removed from the main queue.
  - Pushed into the bound **Dead Letter Queue** for storage/inspection.
- You can customize the max re-delivery count (e.g. set to 5 → message moves to DLQ after 5 failed reads, not 10).
- Once in the DLQ, you can inspect the payload manually (via "Message Browser" → "Get Message") to see what bad data was sent, and decide how to handle it (fix and resubmit to target, log for investigation, etc.).
- Purpose: prevents endlessly retrying a message that can never succeed (e.g. permanently malformed payload) while still preserving it for manual review instead of silently dropping it.

#### Threshold vs. Max Re-delivery Attempts (clarifying Q&A)
These are two independent, complementary controls:
- **Circuit Breaker Threshold** — how many times the *same error type* must occur before the **flow** trips (pauses processing entirely for `trip timeout`).
- **Max Re-delivery Attempts (DLQ setting)** — how many times a *specific message* can be re-read from the queue before it's moved to the DLQ.

Example walked through: MQ max re-delivery = 5, Circuit Breaker threshold = 10 → the message-level DLQ rule (5) will kick in before the flow-level circuit breaker threshold (10) is reached, since they count independently.

### Task Assigned
- Implement Circuit Breaker logic end-to-end using **manual acknowledgment mode** (ACK/NACK) on a queue subscriber:
  1. Publish ~10 messages to a queue.
  2. Point the flow at an intentionally invalid/unreachable target so it fails.
  3. Capture the real error type from the console logs.
  4. Configure that error type in the Circuit Breaker (with threshold = 5 or 10, trip timeout = 5 minutes as practice values).
  5. Observe: flow trips, pauses, and messages queue up; verify DLQ picks up messages that exceed max re-delivery attempts.
- Rationale: this is a common, detailed interview question — need to be able to explain it end-to-end, not just conceptually.

### Teaser: Object Store (next topic)
Brief intro before wrapping up — to be covered in depth in the Apr 07 sessions:
- Object Store is a **connector/component** for storing data inside the Mule runtime itself.
- Question posed to the class: *"What connectors have we used so far?"*
  1. HTTP connector (request + listener)
  2. Anypoint MQ connector
  3. *(Next)* Object Store connector

### Logistics Note
- No live session the next day — self-practice day, especially for the Circuit Breaker task.
- Recordings for the prior 2–3 days were pending upload (mentioned as being uploaded right after this session).

