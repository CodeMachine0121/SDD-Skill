---
name: api-test
description: >
  Generate Postman collection from .sdd/{feature}/ scenarios.
  Triggered by: "api test", "postman", "generate collection", "/api-test".
---

## Steps

**1. Credentials** — if BASE_URL or BEARER_TOKEN not provided, ask for both before anything else.

**2. Feature** — find `.sdd/` folders containing `PRD.md`, `SPEC.md`, or `*.feature`. Multiple → ask user which. None → stop: "Run `/prd` first."

**3. Read docs** from selected folder:
- `SPEC.md`/`*.feature` → all `Scenario:`/`Scenario Outline:` with Given/When/Then
- `PRD.md` → ACs for context
- `ARCH.md`/`API.md` → endpoints, methods, shapes

**4. Show scenario list** and wait for confirmation:
```
N scenarios for "<feature>":
1. [GET]  /users/:id  — Happy path
2. [POST] /users      — Missing field error
…
Generate? (yes / edit first)
```

**5. Generate** `.sdd/{feature}/postman-collection.json` (Postman Collection v2.1):
- Collection variables: `BASE_URL`, `BEARER_TOKEN`
- Collection-level bearer auth using `{{BEARER_TOKEN}}`
- URL: `{{BASE_URL}}<path>` — never hardcode base
- `Content-Type: application/json` on POST/PUT/PATCH
- Body from `When` steps — realistic non-sensitive placeholders
- Description: verbatim Given/When/Then
- Tests tab: `pm.response.to.have.status(<N>)` inferred from `Then`
- Group into folders by domain object or `Feature:` label

**6. Report:**
```
Written: .sdd/{feature}/postman-collection.json
Scenarios: N  |  Variables: BASE_URL, BEARER_TOKEN
Import: Postman → Import → Raw text
```

## Rules
- No credentials → no action
- URL always `{{BASE_URL}}...`, token always in variable
- Never generate without scenario list confirmation
- Only writes `postman-collection.json`
