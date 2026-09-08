# Apr 18 — Full Day Notes

## Session 1: Circuit Breaker — Full Hands-On Rebuild (Part 1)

### Topics Covered
- Building the Circuit Breaker project from scratch on Anypoint Platform
- Publish/Subscribe flow with Anypoint MQ
- Target Variable recap
- Configuring and testing Circuit Breaker live
- Interview Q&A: async decoupling insight (200 OK vs. actual delivery)

### Setting Up Anypoint MQ from Scratch
1. In Anypoint Platform, create a **Client App** (e.g. named "Circuit") to get a **Client ID** and **Client Secret** — used to authenticate the MQ connector config. Test the connection to confirm credentials work.
2. Create a **Queue** (e.g. `CircuitQ`) in a chosen region (e.g. Ireland).
3. Build two flows:
   - **Publish flow** — Listener → Logger → **Publish** to `CircuitQ`.
   - **Subscribe flow** — **Subscribe** from `CircuitQ` → Logger → Request to target system → Logger.

#### Queue vs. Exchange (recap, applied practically)
- If you publish directly to a **Queue**, only that one queue receives the message.
- If you publish to an **Exchange** instead, you can **bind multiple queues** to that exchange — every bound queue receives a copy, enabling one-to-many delivery/subscription.

#### Other config notes
- **Purge** (on a queue) deletes **all messages** currently in that queue — useful for resetting test state between runs.
- **Default acknowledgment mode is Auto** — switched to **Manual** here so the flow can explicitly ACK/NACK based on downstream success/failure.

### Target Variable (recap, applied here)
- When calling the target system via a Request connector, set the response into a **Target Variable** (Advanced tab) rather than letting it overwrite `payload`.
- This keeps the original **attributes/payload** (e.g. the MQ delivery token) accessible separately from the target system's response.

### Manual ACK/NACK Logic
- On successful delivery to target → **ACK** (message removed from queue).
- On failure (e.g. invalid target credentials configured deliberately for this test) → **NACK** (message stays in queue, gets redelivered) — this is what will trigger the Circuit Breaker once it repeats enough times.

### Configuring the Circuit Breaker
- Error type: **HTTP:CONNECTIVITY** (matching the deliberately-broken target credentials used for testing).
- **Threshold:** 5 (flow trips after 5 consecutive occurrences of this error).
- **Trip timeout:** short value for testing (e.g. 1–2 minutes).

### Deploying & Testing
1. Package and deploy the app to **CloudHub** via **Runtime Manager** (upload the built artifact).
2. Trigger via Postman → publish a message to the queue.
3. Observe in Runtime Manager logs:
   - MQ token/ack details appear in the logs as the subscriber consumes the message.
   - Repeated **HTTP:CONNECTIVITY** failures logged — confirmed to occur **exactly 5 times** (matching the threshold) before the flow **trips**.
   - After tripping, no further pickup attempts occur until the trip timeout elapses.

### Important Interview Insight: Async Decoupling
A subtlety walked through live: when Postman receives a **200 OK** immediately after publishing to the queue, that only confirms the message was **successfully accepted into the queue** — it does **not** mean the message was successfully delivered to the downstream target system.
- The publish and the subsequent target delivery happen on **different threads/at different times** (decoupled, asynchronous).
- The source/client has no visibility into what happens after the message enters the queue.
- **Interview framing:** if asked "the client got a 200 response, so why didn't the target receive the data?" — the correct explanation is that the 200 confirms successful **publish to the queue**, not successful **end-to-end delivery**; the actual delivery failure (e.g. HTTP connectivity issue) needs to be diagnosed separately via the Mule application's own logs (e.g. in Runtime Manager).

### Interview Q&A Snippets (mentioned, from a startup interview)
- How do you design an API?
- What is RAML, and what does it stand for?
- How do you deploy an application (CloudHub / Runtime Manager)?
- Difference between vCore and Worker.
- What connectors have you used? Why **SFTP** over **FTP**? (Answer: SFTP is the **secure**, encrypted variant — preferred for security reasons.)
- What's the biggest technical challenge you've faced, and how did you solve it?

### Continues in Part 2
Dead Letter Queue is added to this same project next (see `apr18-02.md`) to combine Circuit Breaker + DLQ end-to-end.


---

## Session 2: Circuit Breaker + Dead Letter Queue Combined (Part 2)

### Topics Covered
- Adding a DLQ to the Circuit Breaker project
- Disambiguating "max redelivery attempts" vs. Circuit Breaker "threshold"
- Communicating failure scenarios to non-technical stakeholders

### Adding a Dead Letter Queue
1. Create a new queue, e.g. `CircuitDLQ`.
2. Bind it to the main queue (`CircuitQ`) as its **Dead Letter Queue**.
3. Set **max redelivery attempts** on the main queue (e.g. tested with a small value like 2, then adjusted to 10 in discussion) — this controls how many times **one specific message** can be redelivered before it's moved to the DLQ.

### Clarifying: Max Redelivery Attempts vs. Circuit Breaker Threshold
These operate at **different levels** and are easy to confuse:

| Setting | Level | What it counts | Effect when hit |
|---|---|---|---|
| **Max Redelivery Attempts** (Queue/DLQ setting) | Per **message** | How many times *that specific message* has been redelivered from the queue | Message is moved out of the main queue into the **DLQ** |
| **Circuit Breaker Threshold** | Per **flow** | How many *consecutive occurrences of the same error type* the flow has seen (regardless of which message) | The **entire flow trips** (pauses consuming) for the trip-timeout duration |

**Worked example:** max redelivery = 10, threshold = 5.
- The flow will trip (pause) after **5** consecutive matching errors — this happens first, regardless of the per-message redelivery cap.
- A given message's individual redelivery count keeps accumulating across trips; once *that message* has been redelivered its max (10) times, it gets pushed to the DLQ — separate from however many times the flow itself has tripped and resumed in the meantime.
- These two counters are independent and serve different purposes: threshold protects the **flow/target system** from being hammered; max redelivery protects against **one bad message** looping forever.

### Communicating This to a Non-Technical Stakeholder
Practiced explaining a realistic support scenario:
- Stakeholder observation: "Postman/the client got a 200 response, so why didn't the target system receive the data?"
- **Correct explanation:** the 200 confirms the message was successfully **published/accepted into the queue** — it does not confirm delivery to the target. The actual failure is downstream: an **HTTP connectivity issue reaching the target system**. This needs to be framed clearly as *"there is a problem with the target system's availability,"* not as a bug in the Mule integration itself, when that's what the logs show.
- Emphasized as a common, realistic interview/on-the-job communication skill — translating technical failure details (async queue decoupling, HTTP connectivity errors) into a clear statement of *what actually went wrong and where*.

### Task / Wrap-up
- Review the recording to fully internalize the Circuit Breaker + DLQ interaction — flagged as one of the most confusing topics if not practiced hands-on.
- Purge test queue messages before the next session to reset state.

