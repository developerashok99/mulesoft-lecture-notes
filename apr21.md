# Apr 21 — Full Day Notes

## Session 1: On-Premise vs Cloud vs VPC, Platform Access Control

### Topics Covered
- Anypoint Platform access management (admin-controlled roles)
- Manual deployment vs. pipeline (CI/CD) deployment
- Cloud vs. On-Premise deployment models
- Proxies and firewalls for cloud ↔ on-premise connectivity
- VPC (Virtual Private Cloud)

### Platform Access Management
- Real organizations restrict access to Anypoint Platform features (Runtime Manager, API Manager, Anypoint MQ, Design Center, Exchange, Monitoring) — access is **admin-controlled**, unlike a personal trial account where everything is open.
- To get access to a given area, you request it from the platform **admin**.

### Manual Deployment vs. Pipeline Deployment
- **Manual deployment:** build the app in Studio, package it, and manually upload the JAR/artifact to the environment via Runtime Manager. Simple but not standard practice.
- **Pipeline deployment (CI/CD):** deployments go through an automated pipeline (e.g. Jenkins) — the standard, expected approach in most real organizations. Manual deployment is mentioned as a sign of a team **not** following mature engineering standards.

### Cloud vs. On-Premise
| | Cloud | On-Premise |
|---|---|---|
| Hosting | MuleSoft's Anypoint Platform runs on **AWS** | Deployed within the **company's own private server/network** |
| Accessibility | Accessible globally via the internet | Accessible only **within the company's private network** |
| Security | Secured via credentials/platform controls | Secured by network isolation — external requests are blocked entirely |
| Typical use | New/modern development | **Legacy applications** — companies often keep old, stable systems on-premise rather than risk migrating them |

- Same Anypoint tooling/UI is used either way — the difference is *where the runtime actually executes* (AWS-hosted vs. the company's own servers).
- On-premise deployments still use the same Runtime Manager concepts, but there's no "Worker / vCore" cloud billing model — resource availability depends on the company's own server capacity, which is free to use as-is (no per-resource cloud cost).

### Connecting Cloud ↔ On-Premise: Proxies & Firewalls
On-premise systems can't be reached directly from the open internet — a firewall blocks unrecognized traffic, only allowing **whitelisted URLs** through.

| Direction | Proxy type needed |
|---|---|
| **On-premise → Cloud** | **Forward Proxy** |
| **Cloud → On-premise** | **Reverse Proxy** |

- A proxy/firewall in this context acts as a controlled pass-through layer — it doesn't modify data, it just decides whether to allow a request through based on whitelisted source/target URLs.
- This whitelisting and firewall configuration is typically owned by the **DevOps/network team**, not the Mule developer — but you're expected to know the concept and correct terminology for interviews.

### VPC (Virtual Private Cloud)
Introduced by MuleSoft (~2.5 years before this recording) specifically to avoid the proxy/firewall complexity above.

- Enabled via a checkbox in the Anypoint Platform environment settings ("Turn your environment to VPC").
- Turning it on changes the **backend server pool** your app deploys to — those VPC-designated servers can connect **directly** to on-premise systems, without needing forward/reverse proxy setup.
- **Critical gotcha:** apps deployed in a VPC environment must use **port 8091** for their HTTP listener (instead of the usual 8081/8082). Using the wrong port results in a **502 Bad Gateway** error.
- Practical interview/on-the-job tip: always find out whether your organization's Anypoint setup is **Cloud**, **VPC**, or **On-Premise** — it directly determines whether you need proxies and which port convention to use.

### Key Takeaway
> Cloud = AWS-hosted, internet-accessible. On-premise = company's own server, network-isolated, needs forward/reverse proxies to bridge to cloud systems. VPC = MuleSoft's answer to skip the proxy dance, but forces port 8091.


---

## Session 2: Interview Confidence Coaching + Wrap-up

### Topics Covered
- DataWeave exercise wrap-up (from Session 1)
- General interview strategy/mindset coaching
- Logistics: remaining topics, platform access window

### DataWeave Exercise — Key Correction
When the requirement asks for a filtered/mapped **array** of results (e.g. employee names matching a condition), the output must:
- Remain an **array** (`[...]`), not be collapsed into a single object — a common mistake was returning one object instead of an array of matches.
- Match exact expected JSON structure: correct key names, correct value types, valid syntax.

### Interview Strategy Coaching
Core message: **you don't need to know everything — you need to confidently and thoroughly explain what you *do* know.**

- No interviewer expects 100% correct answers on every question; different engineers have exposure to different systems (some know Salesforce, some NetSuite, etc.) — that's normal and fine.
- What matters is **presenting your actual experience fully and clearly**: if you've used Anypoint MQ, Pub/Sub, and Circuit Breaker, explain them **in depth**, not superficially — that's what builds interviewer confidence in you.
- Don't worry about impressing with breadth; impress with **depth on what you've actually practiced**.
- This mindset applies beyond MuleSoft interviews — general interview advice.

### Logistics
- Remaining topics for the training: **RAML** (next), **Fragments**, **Bitbucket**, **Jenkins** — flagged as the final topics closing out the course.
- Reminder: the personal Anypoint trial organization/platform access is only available for a limited window (~20 days from setup) — use it to practice Anypoint MQ and other platform features while it lasts, since most companies' own environments don't expose everything as freely to trainees.

