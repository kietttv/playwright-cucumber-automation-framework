# Test Suite — <KEY>: <Feature › Sub-feature> (v1)

**<Ticket summary as in Jira>**

| Field | Value |
|---|---|
| Epic | <EPIC-KEY> – <epic name> |
| Fix version | <fix version> |
| Reporter / PO | <name> |
| Assignee | <name> |
| Ticket status | <status> |
| Case IDs | <first ID> – <last ID> (repo test-ID scheme, e.g. `TC-2135` – `TC-2198`) |
| Total cases | <n> (<ready> ready · <partly> partly blocked · <blocked> fully blocked) |
| Design reference | <attachment names / Figma links / design spec paths> |

---

## 1. What this ticket delivers

<2–4 sentences in plain language. Add a table if the feature has several parts (e.g. link / who sees it / when enabled / where it goes).>

### What the design shows

<Bullets describing each design frame that matters for testing: layout, copy, states.>

---

## 2. Scope

**In scope:** <comma-separated list of behaviours covered>

**Out of scope:**

| Area | Covered by |
|---|---|
| <area> | <ticket key or "Not UI-testable — …"> |

---

## 3. Gaps & open questions (raise before execution)

| # | Gap | Blast radius | Working assumption | Blocks |
|---|---|---|---|---|
| **G1** | <what is missing or contradictory> | Low / Medium / High | <assumption the cases are written to> | <case numbers, or —> |

---

## 4. Test data needed

| # | Data | Used by |
|---|---|---|
| D1 | <account type / record in a specific state> | <case numbers> |

---

## 5. Test cases

> **Legend:** Priority is 🔴 High · 🟡 Medium · 🟢 Low. ⏸️ TBC means blocked or partly blocked by a gap.
> Unless stated otherwise, every case starts from: *<common precondition>*.

### A. <Group name>

---

**<TEST-ID, e.g. TC-2135> · Verify_<Area>_<Behaviour>** 🔴
*Scenario:* <one sentence>
*Steps:*
1. <action>
2. <action>

*Expected:*
- <observable result, with exact copy in quotes>

---

### Already covered elsewhere

| Behaviour | Covered by |
|---|---|
| <behaviour from the AC> | <existing case or test ID, and its suite or ticket> |

---

## 6. Quick reference

<Optional table summarising expected states, e.g. element × status → enabled / disabled / hidden.>

---

## 7. AC coverage

| # | Acceptance criterion | Case(s) |
|---|---|---|
| AC1 | <AC text, shortened> | 001, 002 |

---

## 8. Open items — routing

| # | Question | Owner | Cases waiting |
|---|---|---|---|
| G1 | <question to ask> | <PO / dev name> | <case numbers> |

---

## 9. Change log

### v1 — <dd Mmm yyyy>

Initial suite: <n> cases.
