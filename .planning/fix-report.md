# Fix Report: Agentic Guide

> **Date:** 2026-05-12
> **Scope:** All 20 chapter files + reference files
> **Source:** REVIEW.md findings + PLAN.md structural audit

---

## Summary

6 issues identified, 5 resolved, 1 skipped (low priority), 2 informational.

---

## Issues Table

| # | Issue | Root Cause | Files Changed | Verification | Status |
|---|---|---|---|---|---|
| 1 | Missing 'On Your Hardware' callouts (5 chapters) | Part 3/4 chapters written before callout convention was established | `03-advanced/03-greenfield-brownfield.md`, `03-advanced/04-refactor-migrate.md`, `03-advanced/05-patterns.md`, `03-advanced/06-long-term.md`, `04-daily-driver/04-quality-checks.md` | Grep confirmed all chapters now contain `## On Your Hardware` | Resolved |
| 2 | Chapter 2 too short, missing Common Pitfalls section | Initial draft was abbreviated | `01-beginner/02-debug-fix.md` | Added ~140 lines: 'When the Agent Makes Things Worse' section, 'Common Pitfalls' (3 items), expanded 'On Your Hardware' tiered callout | Resolved |
| 3 | README title mismatch vs revised plan | REVIEW.md suggested "AI Coding on a Budget" vs existing "AI Coding on Constrained Systems" | N/A | Decision: existing title is more descriptive and audience-appropriate | Skipped |
| 4 | Missing Project A/B/C references in Part 4 exercises | Part 4 exercises were generic, not tied to running projects | `04-daily-driver/01-your-setup.md`, `04-daily-driver/02-models-quantization.md`, `04-daily-driver/03-performance.md`, `04-daily-driver/04-quality-checks.md`, `04-daily-driver/05-trust-and-safety.md` | Grep confirmed Project A/B/C references now present in all Part 4 exercises | Resolved |
| 5 | Chapter 8 prerequisite link error | Self-referencing prerequisite ("Completed Chapters 6-7 and Chapter 8") | `02-understand/03-context.md` | Changed to "Completed Chapters 6-7" | Resolved |
| 6 | Chapter 9 missing Common Pitfalls | False positive — section already present (`## Common Pitfalls` at line 364) | N/A | Verified section exists with 3 items | No fix needed |
| 7 | Archive files lack cross-references to new chapters | Content moved from `archive/` to chapters without update notes | `archive/agents.md`, `archive/constrained-systems.md`, etc. | Informational; files archived intentionally | Informational |

---

## Remaining Gaps

### Content-Level (Not Fixed — Require Writing New Content)

1. **Running Projects A/B/C need starter code** — The companion repo mentioned in PLAN.md doesn't exist yet. Project A (CLI tool) starter code should be created before Phase 1 shipping.

2. **Diagrams missing** — PLAN.md calls for ASCII/Mermaid diagrams in chapters (agent loops, architectures, token budgets). No diagrams exist in current chapter files.

3. **Cross-chapter links sparse** — Chapters reference each other by number ("see Chapter 3") but lack actual markdown links to files.

4. **Reference section incomplete** — `reference/prompt-templates.md` and `reference/troubleshooting.md` are stubs. PLAN.md calls for these to be "copy-paste ready."

5. **Archive cross-references** — The 7 files in `archive/` contain the source research. They should have notes linking to the chapter files that consumed their content.

6. **Website not set up** — PLAN.md specifies MkDocs Material or Astro. No site configuration exists yet.

---

## Recommended Next Steps

### Phase 1 (Priority: High — For v0.1 Shipping)
1. Create companion repo skeleton with Project A starter code
2. Add Mermaid diagrams to chapters 0, 2, 5, 6, 7, 10, 11
3. Add markdown cross-links between chapters (prerequisites, further reading)

### Phase 2 (Priority: Medium — Before v0.2)
4. Flesh out reference section (prompt templates, troubleshooting)
5. Add archive cross-reference notes
6. Set up MkDocs site configuration

### Phase 3 (Priority: Low — Post v1.0)
7. Video walkthroughs for exercises
8. Community/feedback collection mechanism

---

## Verification Method

- **Grep searches** for structural elements (`## On Your Hardware`, `Project [ABC]`, `## Common Pitfalls`) across all chapter files
- **Manual review** of prerequisite links and cross-references
- **Line count comparison** for expanded chapters (Chapter 2: +140 lines)
