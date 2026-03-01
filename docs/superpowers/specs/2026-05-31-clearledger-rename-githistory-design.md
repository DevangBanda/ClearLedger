# ClearLedger — Rename & Git History Design

**Date:** 2026-05-31  
**Status:** Approved

---

## Overview

Rename the project from VASTA-Bank to **ClearLedger** across every file and source directory, then rebuild the git history from scratch using an orphan `main` branch with 13 feature branches merged in chronologically — all dated March–April 2026.

---

## Section 1: Rename Plan

All occurrences of `VASTA-Bank`, `VASTA`, `vasta`, `Vasta` are replaced with `ClearLedger`, `clearledger`, or `clear-ledger` as required by context.

| Target | Change |
|---|---|
| `pom.xml` (root + each service) | `groupId`, `artifactId`, `<name>` fields |
| `application.properties` / `application.yml` per service | `spring.application.name` |
| Java source files | Package declarations `com.vasta.*` → `com.clearledger.*`, directory structures renamed to match |
| `docker-compose.yml` | Service names, container names, image tags |
| `K8s/` manifests | `app:`, `name:`, label values |
| `banking-frontend/` | `package.json` name, any config referencing VASTA |
| `README.md` | Full rewrite as ClearLedger |
| `SECURITY.md` | Project name references |
| `.gitignore`, `.dockerignore` | Any VASTA references |

The on-disk folder name (`VASTA-Bank/`) is left unchanged — git does not care about the parent folder.

---

## Section 2: Git History Structure

### Strategy
- Create an orphan `main` branch (zero prior history)
- Apply all rename changes and commit as: `chore: initial ClearLedger project setup`
- Create 13 feature branches off that initial commit
- Each branch gets 2–4 realistic commits
- Branches are merged back to `main` sequentially, earliest → latest
- All commit dates fall within **March 1 – April 30, 2026**

### Branch & Commit Schedule

| Branch | Commits | Approx dates |
|---|---|---|
| `feature/eureka-discovery` | `add Eureka server scaffold` · `configure service registry port and peers` | Mar 3–5 |
| `feature/api-gateway` | `add API gateway service` · `configure routing rules` · `add health check retry logic` | Mar 7–10 |
| `feature/auth-service` | `add auth service scaffold` · `implement JWT filter` · `add login and register endpoints` · `wire security config` | Mar 12–16 |
| `feature/account-service` | `add account service scaffold` · `implement CRUD endpoints` · `add account-user FK validation` | Mar 18–21 |
| `feature/kyc-service` | `add KYC service scaffold` · `implement document upload flow` · `add status tracking` | Mar 24–27 |
| `feature/loan-service` | `add loan service scaffold` · `implement loan application logic` · `add repayment schedule` | Mar 28 – Apr 1 |
| `feature/transaction-service` | `add transaction service scaffold` · `implement transfer logic` · `add exception handling` | Apr 2–5 |
| `feature/payment-gateway` | `add payment gateway scaffold` · `implement payment routing` · `add exception handling` | Apr 7–10 |
| `feature/notification-service` | `add notification service scaffold` · `implement email dispatch` · `add async queue` | Apr 11–14 |
| `feature/admin-server` | `add admin server scaffold` · `configure dashboard endpoints` | Apr 15–17 |
| `feature/frontend` | `add React frontend scaffold` · `connect to API gateway` · `add login and dashboard views` | Apr 18–22 |
| `feature/docker` | `add Dockerfiles per service` · `add docker-compose with all services` | Apr 23–25 |
| `feature/k8s` | `add K8s manifests scaffold` · `configure deployments and services` · `add ingress and config maps` | Apr 26–30 |

### Merge strategy
Each branch is merged into `main` with `git merge --no-ff` to preserve branch topology. Merge commit messages follow the pattern: `Merge feature/X into main`.

---

## Section 3: Execution Approach

1. **Apply all renames** in the working tree (find & replace across all files, rename Java source directories)
2. **Create orphan branch**: `git checkout --orphan main-clean`, stage all files, commit as initial commit with date `2026-03-01`
3. **Delete old master** and rename `main-clean` → `main`
4. **For each feature branch** (in schedule order):
   - `git checkout -b feature/X`
   - Make a small, real change to relevant service files (e.g. touch a config value, update a comment in a relevant file) per commit — commits must not be empty
   - Set `GIT_AUTHOR_DATE` and `GIT_COMMITTER_DATE` per commit
   - `git checkout main && git merge --no-ff feature/X`
5. All 13 branches remain in the local repo (not deleted after merge) so the branch list is visible

---

## Out of Scope

- Pushing to GitHub (user decides when/whether to push)
- Changing the on-disk parent folder name
- Modifying CI/CD pipeline files beyond renaming VASTA references
