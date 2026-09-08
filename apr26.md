# Apr 26 — Full Day Notes

## Session 1: Introduction to Bitbucket (Code Repositories)

### Topics Covered
- Why code repositories matter
- Bitbucket basics: branching model
- Creating a repository and branch, cloning via Git Bash

### Why Use a Code Repository?
Motivating scenario: two developers (e.g. working for the same client) are collaborating on one project. If one suddenly goes on leave, where is their work? If it only lives on their local machine/Studio, the team is stuck.

**A code repository (Bitbucket, GitHub, GitLab, etc.) solves this by:**
- Storing code centrally so any team member can **clone** it and continue work.
- Providing **tracking/history** — who added what, who last committed, via commits and pull requests.
- Enabling proper **code review** via Pull Requests before merging changes.

> Bitbucket and GitHub are the most commonly used code repositories in industry; this concept applies to **any** technology/language, not just Mule.

### Branching Model
- Every new (empty) repository starts with a single default branch: **`master`**.
- **Convention across the industry:** `master` holds **production-ready code only**.
- Since new/untested work shouldn't go straight to `master`, create a separate working branch — e.g. **`develop`** — for code that hasn't been tested yet.
- Branches are created via the repo's **Branches** section (e.g. "Create branch" button) — a new branch starts as a copy of whichever branch it was branched from.

### Hands-On: Setting Up and Cloning a Repo
1. Create an **empty repository** in Bitbucket (e.g. named `dummy`) — note: the local project name and the Bitbucket repository name don't need to match.
2. Create a new branch (e.g. `develop`) via Bitbucket's UI.
3. Locally, create an empty folder to hold cloned code.
4. Open **Git Bash** in that folder and clone the specific branch:
   ```
   git clone -b <branch-name> <repository-URL>
   ```
   - The `-b` flag specifies which branch to clone (without it, you'd get `master`, which should stay empty/production-only at this stage).
   - Example: `git clone -b develop <url>`
   - You'll be prompted for Bitbucket credentials (an app password, not your regular login password, may be required).

### Task Assigned
- Create a Bitbucket account.
- Create a "dummy" project locally in Anypoint Studio.
- Push it to Bitbucket (full workflow continued in `apr26-02.md`).


---

## Session 2: Git Push Workflow + Pull Requests

### Topics Covered
- The core Git Bash push workflow (4 commands)
- Cloning and continuing someone else's work
- Pull Requests — merging `develop` into `master`

### The 4-Command Git Push Workflow
After making changes to a project (e.g. adding a Logger), push them to Bitbucket using:

1. **`git status`** — shows which files have changed (untracked/modified files typically shown in red).
2. **`git add .`** — stages all changed files for commit (staged files typically shown in green when re-checking `git status`).
3. **`git commit -m "<message>"`** — commits the staged changes locally with a descriptive message (e.g. `"my first project"`, `"modified project - add a logger"`).
4. **`git push`** — pushes the committed changes up to the remote Bitbucket repository/branch.

After pushing, you can see the change history in Bitbucket's **Commits** view — additions/removals are highlighted (e.g. green for added lines), along with who made the change and when.

### Continuing a Teammate's Work (the real-world payoff)
Demonstrated end-to-end: one person pushes code → a second person **clones the same branch**, imports it into **Anypoint Studio**, makes further changes (e.g. one more Logger), and pushes again using the same 4-command flow.
- To bring in someone else's existing code changes locally before continuing: replace your local `src` folder with the freshly cloned one (or otherwise sync), then continue editing and repeat the add/commit/push cycle.
- This is the direct payoff of using a shared repository: **anyone can pick up where another person left off**, with full history of who changed what.

### Task Assigned
1. Create a Bitbucket account (if not already done).
2. Create a dummy project in Anypoint Studio.
3. Push it to Bitbucket.
4. Clone it again (simulating a teammate picking it up).
5. Add one more change (e.g. a Logger).
6. Push again.

### Pull Requests (PRs)
Used to **merge one branch into another** (e.g. `develop` → `master`) in a controlled, reviewed way — you don't push directly into `master`.

- In Bitbucket: **Pull Requests → Create Pull Request**, selecting the source branch (`develop`) and destination branch (`master`).
- You assign **reviewers** (e.g. a lead or manager) — creating the PR typically triggers a notification/email to them.
- The reviewer inspects the changes, and either **approves and merges**, or requests changes.
- This review gate before merging into `master` is standard practice — it's how teams ensure only reviewed, tested code reaches the production branch.

