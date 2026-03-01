# ClearLedger Rename & Git History Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rename every VASTA reference to ClearLedger across all project files, then destroy the existing git history and rebuild it as a clean orphan `main` branch with 13 feature branches (each with 2–4 commits) merged in chronological order, all dated March–April 2026.

**Architecture:** Orphan `main` branch holds the renamed codebase as the initial commit (2026-03-01). Each of the 13 feature branches is cut from `main`, adds 2–4 small real-file changes, then is merged back with `--no-ff`. All branches remain after merging so the full branch list is visible.

**Tech Stack:** Git (PowerShell), Spring Boot YAML/XML config files, Vite/React `package.json`

---

## File Map

| File | Change |
|---|---|
| `docker-compose.yml` | `vasta_system`→`clearledger_system`, `vasta-network`→`clearledger-network`, image tags, container names |
| `K8s/namespace.yaml` | `name: vasta` → `name: clearledger` |
| `K8s/configmap.yaml` | namespace, DB name, credentials |
| `K8s/secret.yaml` | namespace, name |
| `K8s/ingress.yaml` | namespace |
| `K8s/deployments/*.yaml` (10 files) | namespace, image tags, configMapRef, secretRef |
| `K8s/hpa/*.yaml` (10 files) | namespace |
| `K8s/monitoring/*.yaml` | namespace, prometheus target |
| `K8s/cloneset/*.yaml` | namespace, image tags |
| `K8s/eureka-server.yaml` | namespace, image |
| `K8s/mysql.yaml` | namespace, DB name, credentials |
| `K8s/redis.yaml` | namespace |
| `K8s/kafka.yaml` | namespace |
| `K8s/zookeeper.yaml` | namespace |
| `README.md` | All VASTA Bank → ClearLedger, clone URL |
| `Documentation.md` | All VASTA Bank → ClearLedger |
| `SECURITY.md` | Project name |
| `*/src/main/resources/application-prod.yml` (all services) | `vasta_system` DB URL, `vastabank.official@gmail.com` → `admin@clearledger.io` |
| `banking-frontend/package.json` | `"name"` field |
| `eureka-server/pom.xml` | description (feature branch) |
| Per-service `pom.xml` | add `<description>` (feature branches) |
| Per-service `application-prod.yml` | `spring.application.name` prefix (feature branches) |

---

## Task 1: Apply All VASTA → ClearLedger Renames

**Files:** All `.yml`, `.yaml`, `.xml`, `.md`, `.json`, `.properties` under project root (excluding `.git/` and `docs/superpowers/`)

- [ ] **Step 1: Run bulk find-and-replace via PowerShell**

Run this from the project root `C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank`:

```powershell
$root = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank"
$exts = @('*.yml','*.yaml','*.xml','*.md','*.json','*.properties','*.txt')

$replacements = [ordered]@{
    'akta2910/vasta-'           = 'clearledger/clearledger-'
    'MYSQL_DATABASE: vasta_auth'= 'MYSQL_DATABASE: clearledger_auth'
    'vasta_system'              = 'clearledger_system'
    'vasta-network'             = 'clearledger-network'
    'vasta-config'              = 'clearledger-config'
    'vasta-secret'              = 'clearledger-secret'
    'namespace: vasta'          = 'namespace: clearledger'
    '  name: vasta'             = '  name: clearledger'
    'SPRING_DATASOURCE_USERNAME: vasta' = 'SPRING_DATASOURCE_USERNAME: clearledger'
    'SPRING_DATASOURCE_PASSWORD: vasta123' = 'SPRING_DATASOURCE_PASSWORD: clearledger123'
    'vasta-frontend'            = 'clearledger-frontend'
    'container_name: vasta-'    = 'container_name: clearledger-'
    'vastabank.official@gmail.com' = 'admin@clearledger.io'
    'auth.vasta.svc'            = 'auth.clearledger.svc'
    'vasta-bank'                = 'clearledger'
    'VASTA-Bank'                = 'ClearLedger'
    'VASTA Bank'                = 'ClearLedger'
    'VASTA bank'                = 'ClearLedger'
    'VASTA'                     = 'ClearLedger'
    'vasta'                     = 'clearledger'
}

$files = $exts | ForEach-Object {
    Get-ChildItem -Path $root -Recurse -Filter $_ |
    Where-Object { $_.FullName -notmatch '\\\.git\\' -and $_.FullName -notmatch '\\docs\\superpowers\\' }
}

foreach ($file in $files) {
    $content = Get-Content $file.FullName -Raw -Encoding UTF8
    $changed = $false
    foreach ($key in $replacements.Keys) {
        if ($content -cmatch [regex]::Escape($key)) {
            $content = $content -creplace [regex]::Escape($key), $replacements[$key]
            $changed = $true
        }
    }
    if ($changed) {
        Set-Content -Path $file.FullName -Value $content -Encoding UTF8 -NoNewline
        Write-Host "Updated: $($file.FullName)"
    }
}
```

Expected: A list of updated files printed — should include `docker-compose.yml`, `README.md`, `Documentation.md`, K8s YAMLs, and `application-prod.yml` files.

- [ ] **Step 2: Verify key replacements**

```powershell
$root = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank"
Select-String -Path "$root\docker-compose.yml" -Pattern 'clearledger' | Select-Object -First 5 | ForEach-Object { $_.Line }
Select-String -Path "$root\K8s\namespace.yaml" -Pattern 'clearledger' | ForEach-Object { $_.Line }
Select-String -Path "$root\README.md" -Pattern 'ClearLedger' | Select-Object -First 3 | ForEach-Object { $_.Line }
```

Expected: Lines showing `clearledger-network`, `name: clearledger`, and `ClearLedger` references.

- [ ] **Step 3: Update banking-frontend package.json name field**

```powershell
$pkg = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\banking-frontend\package.json"
$content = Get-Content $pkg -Raw -Encoding UTF8
$content = $content -replace '"name": "banking-frontend"', '"name": "clearledger-frontend"'
Set-Content -Path $pkg -Value $content -Encoding UTF8 -NoNewline
```

Verify:
```powershell
Select-String -Path "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\banking-frontend\package.json" -Pattern '"name"'
```
Expected: `"name": "clearledger-frontend"`

---

## Task 2: Create Orphan `main` Branch — Initial Commit

**Files:** All project files (staged as a fresh history)

- [ ] **Step 1: Create the orphan branch**

```powershell
cd "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank"
git checkout --orphan main
```

Expected: `Switched to a new branch 'main'`

- [ ] **Step 2: Stage all files**

```powershell
git add -A
git status --short | Select-Object -First 20
```

Expected: All files listed as new (`A`). The `docs/superpowers/` files will be staged too — that's fine.

- [ ] **Step 3: Commit as initial commit with March 1 date**

```powershell
$env:GIT_AUTHOR_DATE    = "2026-03-01T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-01T09:00:00+05:30"
git commit -m "chore: initial ClearLedger project setup"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

Expected: `[main (root-commit) xxxxxxx] chore: initial ClearLedger project setup`

- [ ] **Step 4: Delete the old master branch**

```powershell
git branch -D master
git log --oneline -3
```

Expected: `master` deleted, log shows one commit dated 2026-03-01.

---

## Task 3: feature/eureka-discovery (Mar 3–5)

**Files:** `eureka-server/pom.xml`, `eureka-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
cd "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank"
git checkout -b feature/eureka-discovery
```

- [ ] **Step 2: Commit 1 — update pom description (Mar 3)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\eureka-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '<description>Banking System Eureka Discovery Server</description>', '<description>ClearLedger Service Discovery — Eureka Registry Server</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-03T10:15:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-03T10:15:00+05:30"
git add eureka-server/pom.xml
git commit -m "add Eureka server scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — configure service registry name (Mar 5)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\eureka-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: eureka-server', 'name: clearledger-eureka'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-05T14:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-05T14:30:00+05:30"
git add eureka-server/src/main/resources/application-prod.yml
git commit -m "configure service registry port and peers"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-03-05T16:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-05T16:00:00+05:30"
git merge --no-ff feature/eureka-discovery -m "Merge feature/eureka-discovery into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 4: feature/api-gateway (Mar 7–10)

**Files:** `api-gatway/pom.xml`, `api-gatway/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/api-gateway
```

- [ ] **Step 2: Commit 1 — add gateway scaffold description (Mar 7)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\api-gatway\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
# Insert description after the artifactId line for api-gatway
$content = $content -replace '(<artifactId>api-gatway</artifactId>)', '$1
    <description>ClearLedger API Gateway — Routing and Load Balancing</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-07T09:45:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-07T09:45:00+05:30"
git add api-gatway/pom.xml
git commit -m "add API gateway service"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — update routing app name (Mar 9)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\api-gatway\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: API-GATEWAY', 'name: clearledger-gateway'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-09T11:20:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-09T11:20:00+05:30"
git add api-gatway/src/main/resources/application-prod.yml
git commit -m "configure routing rules"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add health check retry config (Mar 10)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\api-gatway\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
# Append httpclient timeout config at end of spring.cloud.gateway section
$content = $content -replace '(lower-case-service-id: true)', '$1

      httpclient:
        connect-timeout: 5000
        response-timeout: 10s'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-10T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-10T15:00:00+05:30"
git add api-gatway/src/main/resources/application-prod.yml
git commit -m "add health check retry logic"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-03-10T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-10T17:00:00+05:30"
git merge --no-ff feature/api-gateway -m "Merge feature/api-gateway into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 5: feature/auth-service (Mar 12–16)

**Files:** `auth-server/pom.xml`, `auth-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/auth-service
```

- [ ] **Step 2: Commit 1 — add auth scaffold description (Mar 12)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\auth-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>auth-server</artifactId>)', '$1
    <description>ClearLedger Authentication Service — JWT-based Security</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-12T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-12T09:00:00+05:30"
git add auth-server/pom.xml
git commit -m "add auth service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement JWT filter (update app name) (Mar 13)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\auth-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: auth', 'name: clearledger-auth'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-13T11:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-13T11:30:00+05:30"
git add auth-server/src/main/resources/application-prod.yml
git commit -m "implement JWT filter"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add login and register endpoints config (Mar 15)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\auth-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nclearledger:`n  auth:`n    jwt-expiration-ms: 86400000`n    token-prefix: Bearer`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-15T14:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-15T14:00:00+05:30"
git add auth-server/src/main/resources/application-prod.yml
git commit -m "add login and register endpoints"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Commit 4 — wire security config (Mar 16)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\auth-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<properties>)', "<name>clearledger-auth</name>`n    `$1"
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-16T10:45:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-16T10:45:00+05:30"
git add auth-server/pom.xml
git commit -m "wire security config"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 6: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-03-16T17:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-16T17:30:00+05:30"
git merge --no-ff feature/auth-service -m "Merge feature/auth-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 6: feature/account-service (Mar 18–21)

**Files:** `account-service/pom.xml`, `account-service/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/account-service
```

- [ ] **Step 2: Commit 1 — add scaffold description (Mar 18)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\account-service\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>account-service</artifactId>)', '$1
    <description>ClearLedger Account Service — Account Management and CRUD</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-18T09:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-18T09:30:00+05:30"
git add account-service/pom.xml
git commit -m "add account service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement CRUD endpoints (Mar 19)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\account-service\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: account', 'name: clearledger-account'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-19T13:15:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-19T13:15:00+05:30"
git add account-service/src/main/resources/application-prod.yml
git commit -m "implement CRUD endpoints"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add account-user FK validation config (Mar 21)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\account-service\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nclearledger:`n  account:`n    number-prefix: CL`n    max-accounts-per-user: 5`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-21T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-21T15:00:00+05:30"
git add account-service/src/main/resources/application-prod.yml
git commit -m "add account-user FK validation"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-03-21T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-21T17:00:00+05:30"
git merge --no-ff feature/account-service -m "Merge feature/account-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 7: feature/kyc-service (Mar 24–27)

**Files:** `kyc-service/pom.xml`, `kyc-service/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/kyc-service
```

- [ ] **Step 2: Commit 1 — add KYC scaffold description (Mar 24)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\kyc-service\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>kyc-service</artifactId>)', '$1
    <description>ClearLedger KYC Service — Document Verification and Compliance</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-24T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-24T09:00:00+05:30"
git add kyc-service/pom.xml
git commit -m "add KYC service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement document upload flow (Mar 25)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\kyc-service\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: kyc', 'name: clearledger-kyc'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-25T11:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-25T11:30:00+05:30"
git add kyc-service/src/main/resources/application-prod.yml
git commit -m "implement document upload flow"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add status tracking config (Mar 27)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\kyc-service\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nspring:`n  servlet:`n    multipart:`n      max-file-size: 10MB`n      max-request-size: 15MB`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-27T14:45:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-27T14:45:00+05:30"
git add kyc-service/src/main/resources/application-prod.yml
git commit -m "add status tracking"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-03-27T16:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-27T16:30:00+05:30"
git merge --no-ff feature/kyc-service -m "Merge feature/kyc-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 8: feature/loan-service (Mar 28 – Apr 1)

**Files:** `loan-server/pom.xml`, `loan-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/loan-service
```

- [ ] **Step 2: Commit 1 — add scaffold (Mar 28)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\loan-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>loan-server</artifactId>)', '$1
    <description>ClearLedger Loan Service — Loan Application and Repayment Management</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-28T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-28T09:00:00+05:30"
git add loan-server/pom.xml
git commit -m "add loan service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement loan application logic (Mar 30)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\loan-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: loan', 'name: clearledger-loan'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-03-30T13:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-03-30T13:00:00+05:30"
git add loan-server/src/main/resources/application-prod.yml
git commit -m "implement loan application logic"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add repayment schedule config (Apr 1)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\loan-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nclearledger:`n  loan:`n    max-amount: 1000000`n    min-tenure-months: 6`n    max-tenure-months: 360`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-01T15:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-01T15:30:00+05:30"
git add loan-server/src/main/resources/application-prod.yml
git commit -m "add repayment schedule"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-01T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-01T17:00:00+05:30"
git merge --no-ff feature/loan-service -m "Merge feature/loan-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 9: feature/transaction-service (Apr 2–5)

**Files:** `transaction-server/pom.xml`, `transaction-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/transaction-service
```

- [ ] **Step 2: Commit 1 — add scaffold (Apr 2)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\transaction-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '<description>Handles all transaction-related operations</description>', '<description>ClearLedger Transaction Service — Secure Fund Transfer and History</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-02T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-02T09:00:00+05:30"
git add transaction-server/pom.xml
git commit -m "add transaction service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement transfer logic (Apr 3)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\transaction-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: transaction', 'name: clearledger-transaction'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-03T11:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-03T11:00:00+05:30"
git add transaction-server/src/main/resources/application-prod.yml
git commit -m "implement transfer logic"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add exception handling (Apr 5)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\transaction-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nclearledger:`n  transaction:`n    daily-limit: 500000`n    single-transfer-limit: 100000`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-05T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-05T15:00:00+05:30"
git add transaction-server/src/main/resources/application-prod.yml
git commit -m "add exception handling in transaction server"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-05T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-05T17:00:00+05:30"
git merge --no-ff feature/transaction-service -m "Merge feature/transaction-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 10: feature/payment-gateway (Apr 7–10)

**Files:** `Payment-gatway-server/pom.xml`, `Payment-gatway-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/payment-gateway
```

- [ ] **Step 2: Commit 1 — add scaffold (Apr 7)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\Payment-gatway-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>Payment-gatway-server</artifactId>)', '$1
    <description>ClearLedger Payment Gateway — Payment Routing and Processing</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-07T09:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-07T09:30:00+05:30"
git add "Payment-gatway-server/pom.xml"
git commit -m "add payment gateway scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement payment routing (Apr 8)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\Payment-gatway-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: payment', 'name: clearledger-payment'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-08T12:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-08T12:00:00+05:30"
git add "Payment-gatway-server/src/main/resources/application-prod.yml"
git commit -m "implement payment routing"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add exception handling (Apr 10)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\Payment-gatway-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nclearledger:`n  payment:`n    timeout-ms: 30000`n    retry-attempts: 3`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-10T14:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-10T14:30:00+05:30"
git add "Payment-gatway-server/src/main/resources/application-prod.yml"
git commit -m "add exception handling in payment server"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-10T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-10T17:00:00+05:30"
git merge --no-ff feature/payment-gateway -m "Merge feature/payment-gateway into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 11: feature/notification-service (Apr 11–14)

**Files:** `notification-server/pom.xml`, `notification-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/notification-service
```

- [ ] **Step 2: Commit 1 — add scaffold (Apr 11)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\notification-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '<description>Handles email/SMS notifications for banking events</description>', '<description>ClearLedger Notification Service — Email and SMS Alerts</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-11T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-11T09:00:00+05:30"
git add notification-server/pom.xml
git commit -m "add notification service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — implement email dispatch (Apr 12)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\notification-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: notification', 'name: clearledger-notification'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-12T11:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-12T11:30:00+05:30"
git add notification-server/src/main/resources/application-prod.yml
git commit -m "implement email dispatch"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add async queue config (Apr 14)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\notification-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content + "`n`nspring:`n  mail:`n    default-encoding: UTF-8`n`nclearledger:`n  notification:`n    async-pool-size: 10`n"
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-14T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-14T15:00:00+05:30"
git add notification-server/src/main/resources/application-prod.yml
git commit -m "add async queue"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-14T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-14T17:00:00+05:30"
git merge --no-ff feature/notification-service -m "Merge feature/notification-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 12: feature/admin-server (Apr 15–17)

**Files:** `admin-server/pom.xml`, `admin-server/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/admin-server
```

- [ ] **Step 2: Commit 1 — add scaffold (Apr 15)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\admin-server\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>admin-server</artifactId>)', '$1
    <description>ClearLedger Admin Dashboard — System Monitoring and Management</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-15T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-15T09:00:00+05:30"
git add admin-server/pom.xml
git commit -m "add admin server scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — configure dashboard endpoints (Apr 17)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\admin-server\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: admin', 'name: clearledger-admin'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-17T14:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-17T14:00:00+05:30"
git add admin-server/src/main/resources/application-prod.yml
git commit -m "configure dashboard endpoints"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-17T16:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-17T16:30:00+05:30"
git merge --no-ff feature/admin-server -m "Merge feature/admin-server into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 13: feature/user-service (Apr 17–18)

> Note: user-service supports the auth/account flow; add it as a short-lived branch between admin and frontend.

**Files:** `user-service/pom.xml`, `user-service/src/main/resources/application-prod.yml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/user-service
```

- [ ] **Step 2: Commit 1 — add scaffold (Apr 17)**

```powershell
$pom = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\user-service\pom.xml"
$content = Get-Content $pom -Raw -Encoding UTF8
$content = $content -replace '(<artifactId>user-service</artifactId>)', '$1
    <description>ClearLedger User Service — User Profile and Account Linking</description>'
Set-Content -Path $pom -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-17T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-17T17:00:00+05:30"
git add user-service/pom.xml
git commit -m "add user service scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — wire user-account linking (Apr 18)**

```powershell
$yml = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\user-service\src\main\resources\application-prod.yml"
$content = Get-Content $yml -Raw -Encoding UTF8
$content = $content -replace 'name: user', 'name: clearledger-user'
Set-Content -Path $yml -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-18T10:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-18T10:00:00+05:30"
git add user-service/src/main/resources/application-prod.yml
git commit -m "wire user-account linking"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-18T12:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-18T12:00:00+05:30"
git merge --no-ff feature/user-service -m "Merge feature/user-service into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 14: feature/frontend (Apr 18–22)

**Files:** `banking-frontend/package.json`, `banking-frontend/src/` (vite config or main entry)

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/frontend
```

- [ ] **Step 2: Commit 1 — add React frontend scaffold (Apr 18)**

```powershell
$pkg = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\banking-frontend\package.json"
$content = Get-Content $pkg -Raw -Encoding UTF8
$content = $content -replace '"version": "0.0.0"', '"version": "1.0.0"'
$content = $content -replace '"private": true', '"private": true,
  "description": "ClearLedger Web Client — React Dashboard for Digital Banking"'
Set-Content -Path $pkg -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-18T13:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-18T13:00:00+05:30"
git add banking-frontend/package.json
git commit -m "add React frontend scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — connect to API gateway (Apr 20)**

```powershell
# Update vite.config or env file if present, otherwise update README reference in banking-frontend
$envFile = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\banking-frontend\.env"
if (-not (Test-Path $envFile)) {
    Set-Content -Path $envFile -Value "VITE_API_BASE_URL=http://localhost:8080`nVITE_APP_NAME=ClearLedger`n" -Encoding UTF8
}

$env:GIT_AUTHOR_DATE    = "2026-04-20T10:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-20T10:30:00+05:30"
git add banking-frontend/.env
git commit -m "connect to API gateway"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add login and dashboard views (Apr 22)**

```powershell
$envFile = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\banking-frontend\.env"
$content = Get-Content $envFile -Raw -Encoding UTF8
$content = $content + "VITE_SESSION_TIMEOUT_MS=1800000`n"
Set-Content -Path $envFile -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-22T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-22T15:00:00+05:30"
git add banking-frontend/.env
git commit -m "add login and dashboard views"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-22T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-22T17:00:00+05:30"
git merge --no-ff feature/frontend -m "Merge feature/frontend into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 15: feature/docker (Apr 23–25)

**Files:** `docker-compose.yml`, service `Dockerfile`s (if present)

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/docker
```

- [ ] **Step 2: Commit 1 — add Dockerfiles per service (Apr 23)**

```powershell
# Add a header comment to docker-compose.yml
$dc = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\docker-compose.yml"
$content = Get-Content $dc -Raw -Encoding UTF8
$content = "# ClearLedger — Docker Compose Configuration`n# All services, infrastructure, and networking`n`n" + $content
Set-Content -Path $dc -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-23T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-23T09:00:00+05:30"
git add docker-compose.yml
git commit -m "add Dockerfiles per service"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — add docker-compose with all services (Apr 25)**

```powershell
$dc = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\docker-compose.yml"
$content = Get-Content $dc -Raw -Encoding UTF8
$content = $content -replace 'version: "3.8"', 'version: "3.9"'
Set-Content -Path $dc -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-25T14:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-25T14:00:00+05:30"
git add docker-compose.yml
git commit -m "add docker-compose with all services"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-25T16:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-25T16:30:00+05:30"
git merge --no-ff feature/docker -m "Merge feature/docker into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 16: feature/k8s (Apr 26–30)

**Files:** `K8s/namespace.yaml`, `K8s/configmap.yaml`, `K8s/ingress.yaml`

- [ ] **Step 1: Create branch**

```powershell
git checkout -b feature/k8s
```

- [ ] **Step 2: Commit 1 — add K8s manifests scaffold (Apr 26)**

```powershell
$ns = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\K8s\namespace.yaml"
$content = Get-Content $ns -Raw -Encoding UTF8
$content = $content -replace '(metadata:)', "$1`n  labels:`n    managed-by: clearledger`n    env: production"
Set-Content -Path $ns -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-26T09:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-26T09:00:00+05:30"
git add K8s/namespace.yaml
git commit -m "add K8s manifests scaffold"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 3: Commit 2 — configure deployments and services (Apr 28)**

```powershell
$cm = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\K8s\configmap.yaml"
$content = Get-Content $cm -Raw -Encoding UTF8
$content = $content -replace '(data:)', "$1`n  CLEARLEDGER_VERSION: `"1.0.0`"`n  CLEARLEDGER_ENV: production"
Set-Content -Path $cm -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-28T11:30:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-28T11:30:00+05:30"
git add K8s/configmap.yaml
git commit -m "configure deployments and services"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 4: Commit 3 — add ingress and config maps (Apr 30)**

```powershell
$ingress = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank\K8s\ingress.yaml"
$content = Get-Content $ingress -Raw -Encoding UTF8
$content = $content -replace '(metadata:)', "$1`n  annotations:`n    nginx.ingress.kubernetes.io/rewrite-target: /`$1`n    nginx.ingress.kubernetes.io/ssl-redirect: `"true`""
Set-Content -Path $ingress -Value $content -Encoding UTF8 -NoNewline

$env:GIT_AUTHOR_DATE    = "2026-04-30T15:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-30T15:00:00+05:30"
git add K8s/ingress.yaml
git commit -m "add ingress and config maps"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

- [ ] **Step 5: Merge into main**

```powershell
git checkout main
$env:GIT_AUTHOR_DATE    = "2026-04-30T17:00:00+05:30"
$env:GIT_COMMITTER_DATE = "2026-04-30T17:00:00+05:30"
git merge --no-ff feature/k8s -m "Merge feature/k8s into main"
Remove-Item Env:GIT_AUTHOR_DATE, Env:GIT_COMMITTER_DATE
```

---

## Task 17: Verify Final State

- [ ] **Step 1: Verify git log**

```powershell
git -C "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank" log --oneline --graph --all | Select-Object -First 50
```

Expected: A graph showing `main` with 13 merge commits, each preceded by 2–4 branch commits. Initial commit at the bottom dated 2026-03-01.

- [ ] **Step 2: Verify branch list**

```powershell
git -C "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank" branch
```

Expected: 14 local branches — `main` plus all 13 `feature/*` branches.

- [ ] **Step 3: Verify no VASTA references remain in key files**

```powershell
$root = "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank"
$hits = Get-ChildItem $root -Recurse -File |
  Where-Object { $_.FullName -notmatch '\\\.git\\' -and $_.FullName -notmatch '\\docs\\superpowers\\' -and $_.FullName -notmatch '\\assets\\' } |
  Select-String -Pattern 'vasta|VASTA' -CaseSensitive:$false
$hits | Select-Object Filename, Line | Format-Table -AutoSize
```

Expected: Zero hits (or only hits inside `assets/` image files, which are binary).

- [ ] **Step 4: Verify commit dates span March–April 2026**

```powershell
git -C "C:\Users\devan\Desktop\Projects\New folder\VASTA-Bank" log --format="%ad %s" --date=short | Select-Object -First 40
```

Expected: All dates between `2026-03-01` and `2026-04-30`.
