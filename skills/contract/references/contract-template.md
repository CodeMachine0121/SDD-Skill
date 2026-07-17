# Contract Traceability Matrix — {feature}

Contract: {PRD.md | BRIEF.md}
Design map: {ARCH.md | none}
Implementation: {path}
Oracle: Acceptance Criteria ({N} clauses)

## Clauses

The `Spec-expected` column holds the business-observable oracle from Phase 2; the
concrete artifact it bridges to (via UL-MAP/ARCH) is what the audit columns check.

| ID | Clause | Spec-expected (oracle) | Impl | Test | Test audit | Code audit | Status |
|----|--------|------------------------|------|------|------------|------------|--------|
| AC-1 | <verbatim> | Checkout is rejected, showing "Invalid amount" | svc.ts:42 | order.test.ts:t_neg | asserts-oracle | produces-oracle | ✅ conforms |
| AC-2 | <verbatim> | ... | — | — | no-test | not-implemented | ❌ gap |
| AC-3 | <verbatim> | "Invalid amount" is shown | svc.ts:50 | order.test.ts:t_neg2 | shallow | produces-oracle | 🟠 mis-asserted |
| BR-1 | <verbatim> | Order total = subtotal − discount | order.ts:80 | order.test.ts:t_total | asserts-oracle | diverges | 🔴 violation |

## Orphans (code with no clause)

| Code | Description | Verdict |
|------|-------------|---------|
| util.ts:90 | <what it does> | undocumented / **violation (out-of-scope)** |

## Summary

- Conforms: X/N clauses ✅ (P%)
- Violations: list of 🔴 IDs  (code produces the wrong outcome)
- Mis-asserted: list of 🟠 IDs  (green test asserts the wrong/weaker thing)
- Partial: list of 🟡 IDs  (no test asserts the oracle)
- Gaps: list of ❌ IDs
- Unclear: list of ❔ IDs
- Orphans: count
