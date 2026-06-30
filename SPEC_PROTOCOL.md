# SPEC BOOK PROTOCOL

> **Version:** 1.0
> **Last Updated:** 2026-06-30
> **Purpose:** Governs how chapters are created, filled, reviewed, locked, and how sessions are run. This file + `SPEC_STATE.md` are the only two files you need to read to resume work at any time.

---

## 1. CHAPTER LIFECYCLE

Every chapter moves through 5 stages. Each stage has explicit entry criteria, allowed actions, and exit criteria. A chapter cannot skip stages.

### Stage 1: CREATED
> **Meaning:** Template file exists, no decisions filled yet.

**Entry Criteria:**
- Chapter is next in the write order (all dependency chapters are at least IN PROGRESS)
- Template file created from `chapter_template.md`

**Allowed Actions:**
- Read dependency chapters to understand incoming constraints
- Read cross-chapter impact register for any impacts flagged from other chapters

**Exit Criteria → SCOPED:**
- [ ] 15-30 questions identified and written to §4 (Open Questions)
- [ ] Cross-chapter impacts from dependencies reviewed and noted
- [ ] Scope boundaries (§1.1 and §1.2) filled in

---

### Stage 2: SCOPED
> **Meaning:** Questions are identified, dependencies verified. Ready for grilling.

**Exit Criteria → IN PROGRESS:**
- [ ] First grilling session completed (at least 1 question resolved)

---

### Stage 3: IN PROGRESS
> **Meaning:** Active grilling. Sessions are happening, decisions are being made.

**Exit Criteria → COMPLETE:**
- [ ] All open questions resolved (§4 is empty)
- [ ] All decisions have: context, options, decision, implementation contract, acceptance criteria
- [ ] Design patterns section (§4) filled
- [ ] Agent instructions (§5) filled — Always / Ask Before / Never tiers
- [ ] No TODOs, no "TBD", no placeholders remaining

---

### Stage 4: REVIEWED
> **Meaning:** Self-review passed. Cross-chapter consistency verified.

**Self-Review Checklist:**
- [ ] Every decision has a concrete implementation contract (not prose)
- [ ] Every acceptance criterion is binary testable (pass/fail, not subjective)
- [ ] Every file path mentioned matches the folder structure in Ch 00
- [ ] Every TypeScript interface is complete (no `any`, no `// TODO`)
- [ ] Agent instructions (§5) cover all decisions in the chapter
- [ ] Anti-patterns section lists at least one "never do" per major decision
- [ ] Cross-chapter impact register has no "Pending" items

**Exit Criteria → LOCKED:**
- [ ] Self-review checklist passed
- [ ] All cross-chapter impacts propagated and acknowledged
- [ ] No contradictions with dependency chapters
- [ ] `SPEC_STATE.md` updated to reflect LOCKED status

---

### Stage 5: LOCKED
> **Meaning:** Frozen. Implementation-ready. No changes without formal unlock.

**Unlock Process:**
1. Create a CHANGE REQUEST entry at the top of the chapter file
2. State: what's changing, why, which decisions are affected
3. Propagate impact to all dependent chapters
4. Affected dependent chapters drop from LOCKED → REVIEWED
5. Re-review and re-lock all affected chapters
6. Update `SPEC_STATE.md`

---

## 2. SESSION PROTOCOL

### Session Types

| Type | Purpose | Duration |
|------|---------|----------|
| **Cold-Start** | Read state, determine next action | 5 min |
| **Grilling** | Active decision-making on a chapter | 30-60 min |
| **Review** | Self-review or cross-chapter review | 20-40 min |
| **Propagation** | Push cross-chapter impacts to dependent chapters | 15-30 min |

### Cold-Start Session
1. Read `SPEC_STATE.md`
2. Report: active chapter, stage, decisions resolved, open questions, last session, next action, blockers
3. Ask user: "Ready to [recommended action]?"

### Grilling Session
1. Run Pre-Grilling Checklist
2. Identify next question(s) — prefer questions that unblock others or have cross-chapter impact
3. For each question: present options → user decides → fill decision sub-template → check cross-chapter impact
4. Update chapter session history (§7)
5. Update SPEC_STATE.md

### Review Session
1. Run Self-Review Checklist
2. Fix any failed items
3. Verify cross-chapter consistency
4. Mark chapter REVIEWED
5. Update SPEC_STATE.md

### Propagation Session
1. Find chapters with pending cross-chapter impacts in SPEC_STATE.md
2. For each: open affected chapter, read impact, update decisions if needed, mark propagated
3. Update SPEC_STATE.md

---

## 3. NEXT-ACTION DECISION TREE

Evaluate top-to-bottom. First match wins.

1. **Pending cross-chapter impacts?** → Propagation Session
2. **Chapter in COMPLETE (not REVIEWED)?** → Review Session
3. **Chapter IN PROGRESS?** → Continue grilling
4. **Chapter REVIEWED?** → Lock it, update state, re-evaluate
5. **Chapter CREATED with deps ≥ IN PROGRESS?** → Scope it
6. **Chapter SCOPED with deps ≥ IN PROGRESS?** → Start grilling
7. **All LOCKED?** → Spec book complete
8. **All blocked?** → Report blocker chain

---

## 4. END-OF-SESSION OBLIGATIONS

At the end of EVERY session, update SPEC_STATE.md:
- Last session date
- Last session summary (1-2 sentences)
- Chapter stage (if changed)
- Decisions resolved / total
- Open questions count
- Next recommended action (computed from decision tree)
- Blocked items
- Append entry to Recent Sessions log

**Rule:** If SPEC_STATE.md is not updated, the session didn't happen.
