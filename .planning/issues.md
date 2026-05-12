# Tracked Issues

> Derived from REVIEW.md analysis and structural audit.

---

## Issue 1: Missing "On Your Hardware" Callouts (4 chapters)

**Severity:** High — Content Principles require every chapter to have a hardware-aware callout.

**Affected chapters:**
- `03-advanced/03-greenfield-brownfield.md` (Chapter 12)
- `03-advanced/04-refactor-migrate.md` (Chapter 13)
- `03-advanced/05-patterns.md` (Chapter 14)
- `03-advanced/06-long-term.md` (Chapter 15)
- `04-daily-driver/04-quality-checks.md` (Chapter 19)

**Fix:** Add "On Your Hardware" section to each chapter.

---

## Issue 2: Chapter 2 (Debug) Too Short — Missing Common Pitfalls Section

**Severity:** Medium — REVIEW.md recommends each chapter have "Common pitfalls" section. Chapter 2 is only 127 lines vs ~350+ for peers.

**Affected:** `01-beginner/02-debug-fix.md`

**Fix:** Add Common Pitfalls section and expand "When the agent makes things worse" content.

---

## Issue 3: README Title Doesn't Match Revised Plan

**Severity:** Low — README says "AI Coding on Constrained Systems" but revised plan title is "AI Coding on a Budget".

**Affected:** `README.md`

**Fix:** Update title to match revised plan.

---

## Issue 4: Missing Project A/B/C References in Part 3 and Part 4

**Severity:** Medium — PLAN.md specifies running projects threaded across chapters. Part 3/4 exercises don't consistently reference Project A/B/C.

**Affected:** All Part 3 and Part 4 chapters

**Fix:** Add Project A/B/C references to exercises where missing.

---

## Issue 5: Chapter 8 (Context) Prerequisite Link Error

**Severity:** Low — Prerequisites section references "Chapter 8" for itself.

**Affected:** `02-understand/03-context.md:21`

**Fix:** Correct prerequisite reference.

---

## Issue 6: No "Common Pitfalls" in Chapter 9 (What Not to Build)

**Severity:** Low — Has "Common Pitfalls" section but named differently.

**Affected:** `02-understand/04-what-not-to-build.md`

**Status:** Actually present as "Common Pitfalls" at line 339. No fix needed.

---

## Issue 7: Archive Files Not Referenced from Chapters

**Severity:** Low — REVIEW.md notes archive files contain strong research that should be cross-referenced.

**Affected:** All chapters

**Status:** Informational — chapters reference `reference/` but not `archive/`.
