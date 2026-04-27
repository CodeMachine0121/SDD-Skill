# 📔 Ubiquitous Language Map

**Project:** ____________________
**Bounded Context:** ____________________
**Maintainer:** ____________________
**Last Updated:** YYYY-MM-DD

---

## 1. Nouns & Concepts
*Records entities, value objects, attributes and their correspondence between code and real business.*

| Domain Term | Technical Name | User-Facing Label | Definition & Business Rules | Status |
| :--- | :--- | :--- | :--- | :--- |
| *(correct canonical name)* | *(name in code / DB)* | *(label shown in UI or reports)* | *(business logic, constraints, invariants)* | *(Archeology / Confirmed / To Be Deleted)* |
| | | | | |
| | | | | |

---

## 2. Actions & Processes
*Records business operations, function logic, and their corresponding business actions.*

| Business Action | Technical Method | Trigger | Business Impact | Notes |
| :--- | :--- | :--- | :--- | :--- |
| *(action as a domain expert describes it)* | *(function / method name in code)* | *(when this logic executes)* | *(what state changes or notifications result)* | |
| | | | | |
| | | | | |

---

## 3. Ambiguities & Conflicts
*Records cases where the same technical term means different things in different modules, or multiple terms refer to the same concept.*

| Ambiguous Term | Meaning in Context A | Meaning in Context B | Resolution |
| :--- | :--- | :--- | :--- |
| *(ambiguous variable or term)* | *(meaning in module X)* | *(meaning in module Y)* | *(recommended rename or disambiguation)* |
| | | | |
| | | | |

---

## 4. External & Enum Mapping
*Records magic numbers/strings in code and their real business meaning.*

| Category | Code Value / Key | Domain Label | Description |
| :--- | :--- | :--- | :--- |
| *(e.g., Status Code)* | *(e.g., 1, 2, 3 or "A", "B")* | *(e.g., Pending, Completed)* | |
| | | | |
| | | | |

---

## Quick Start Guide
1. **Archeology** — read source code; fill `Technical Name` with raw names found in the codebase.
2. **Mapping** — check UI screens or ask business stakeholders; fill `Domain Term` with the correct canonical name.
3. **Refine** — add business rules (e.g., "this field cannot be negative", "this action must occur after checkout").
4. **Sync** — this document is the single authoritative dictionary for all future renaming, refactoring, and new documentation.
