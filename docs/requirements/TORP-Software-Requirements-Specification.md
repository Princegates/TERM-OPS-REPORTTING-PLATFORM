---
title: "Terminal Operations Reporting Platform (TORP)"
subtitle: "Software Requirements Specification"
date: "26 October 2026"
---

| | |
|---|---|
| **Document** | Software Requirements Specification (SRS), modelled on ISO/IEC/IEEE 29148 |
| **Version** | 0.1 – Draft for stakeholder review |
| **Date** | 26 October 2026 |
| **Status** | Draft. Open questions are in §13 and need answers before design sign-off. |
| **Basis** | Analysis of four live operational documents: T2 Day Duty Report (26-10-26), T2 Walkthrough (26-10-26), Terminal 3 Departure Duty Report, Terminal 3 Departures Walkthrough |

---

## Table of contents

1. Introduction
2. Evidence base: what the current reports tell us
3. Overall description
4. Business process: as-is and to-be
5. Domain and data requirements
6. Functional requirements
7. Report output requirements
8. Non-functional requirements
9. External interfaces and integrations
10. Proposed solution architecture
11. Data migration and onboarding
12. Delivery plan, acceptance and traceability
13. Assumptions, risks and open questions
14. Appendices (A: vocabulary mapping, B: location inventory, C: source-to-model worked example, D: glossary)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the requirements for the **Terminal Operations Reporting Platform (TORP)**. TORP replaces the Word-based shift duty reports and facility walkthrough reports used by the terminal duty officers at the airport. It captures, tracks, escalates and reports on terminal operational status and facility defects.

The intended readers are:

- Terminal Operations management, who own the requirements.
- Duty officers, who are the primary users.
- Facility-management and ICT counterparts, who receive the defects.
- The engineering team, which designs, builds and tests the system.
- QA, security and audit reviewers.

### 1.2 Problem statement

Each shift, duty officers inspect their terminal zones and type their findings into a Word table. The table has three columns: ITEM, DETAILS and REMARKS/ACTION TAKEN. A separate walkthrough document is compiled for each terminal. These documents are the only record of what was found, who was told and what happened next.

Analysis of the four documents supplied (§2) shows that the format causes five problems:

1. **No identity for a defect.** The same defect is retyped in every shift report and again in the walkthrough. Nothing says whether "leakage beside carousel 4" today is the same defect as yesterday's.
2. **Action is not linked to finding.** Findings and actions sit in separate columns and are matched only by line order. Most actions are the bare phrase "FMCC informed".
3. **No lifecycle.** Items are never closed in a verifiable way. The only status vocabulary is the words "Pending" and "FIXED". There is no age, no severity, no owner and no due date.
4. **Free-text vocabulary.** The same condition is written many ways ("diffused", "defused", "not serviceable", "unserviceable", "faulty"). Counts, status answers and ticket numbers are not validated.
5. **High effort, low insight.** The four documents supplied hold about 1,150 lines of typed text. Management cannot answer basic questions from them: how long has this been broken, who is slow to respond, or which washroom keeps failing?

### 1.3 Product vision

> Record each fact once, at the place and moment it is observed. Track it until it is fixed and verified. Generate the shift report, walkthrough, handover and management summaries from that single record in the layout stakeholders already know.

### 1.4 Scope

**In scope (Release 1 and 2)**

- Terminal 2 (Arrivals and Departures) and Terminal 3 (Departures) as observed in the source reports.
- Digital shift duty report, walkthrough inspection, shift handover and staff-strength capture.
- A persistent defect register covering the full lifecycle, with notifications to FMCC, ICT, SITA, Electromech and other parties.
- Output of reports in the current Word layout, plus PDF and spreadsheet exports.
- Dashboards, ageing and recurrence analytics.
- Import of historical Word reports.

**Out of scope for now (design must not preclude them)**

- Terminal 3 Arrivals and other terminals, which are added as configuration (§6.1).
- Replacement of the FMCC work-order system, which stays the system of record for repair work.
- Flight-schedule, passenger-flow or baggage-system telemetry. Automated feeds from BHS, FIDS or building-management systems are a future integration (§9).
- Payroll, HR rostering and airline billing.

### 1.5 Definitions

Terms and abbreviations are in Appendix D. Abbreviations are expanded from context. Entries marked *(inferred)* must be confirmed by the business (§13).

### 1.6 Requirement conventions

- Each requirement has a unique ID, `FR-<area>-<nn>` for functional and `NFR-<area>-<nn>` for non-functional.
- Priority uses MoSCoW: **M** Must, **S** Should, **C** Could, **W** Won't (this release).
- "The system shall" is binding. "Should" appears only in the Priority column.
- The Evidence column cites the finding in §2 that justifies the requirement (for example E3). Requirements without evidence come from good practice and are marked "–".

---

## 2. Evidence base: what the current reports tell us

This section is the factual basis for the requirements. All figures come from the four documents supplied and were counted programmatically.

### 2.1 The four documents

| # | Document | Type | Shape |
|---|---|---|---|
| D1 | T2 Day Duty Report 26-10-26 | Shift duty report | Two tables: *Terminal 2 Arrivals* (items 1–8) and *Terminal 2 Departures* (items 1–8). Columns: NO, ITEM, DETAILS, REMARKS/ACTION TAKEN. |
| D2 | T2 Walkthrough 26-10-26 | Facility walkthrough | One table by area: Information Office/Upper Concourse, Airside, Check-in, Lounge, Gates, Transit, Immigration, Baggage Hall, Customs Area/Airport Square. |
| D3 | Terminal 3 Departure Duty Report | Shift duty report | Items 1–9: Departures, Handover, Staff, Frontage, Pre-departure, Check-in (Aisles A, B and C, food court), Departure Lounge (gates C2–C8B), Baggage Make-up, OCC. |
| D4 | Terminal 3 Departures Walkthrough | Facility walkthrough | Boarding gates, departure check-in, aisles, washrooms by DR number, plus an "OTHERS" block. |

Combined, the four documents hold **1,147 text lines** in their detail columns and **686 entries** in their remarks columns (including a few headers and notes).

### 2.2 What the documents contain (information model)

**Shift context**

- Shift name and handover time: "CHARLIE shift took over operations at 0600HRS" in D1, "at 1800 HOURS" in D3.
- Staff strength by post, with presence. Posts seen: Arrival Hall, Announcement Room, Departures, Departure Lounge, OCC and PA.
- "Previous loggings", which is free text such as "Some issues remain unresolved".

**Checklist questions** (the ITEM column). Each question gets a status or a number:

- *Status-type:* serviceability of FIDS, PA, A/C, automated doors, escalators, elevators (by number), BHS, CUSS kiosks, i-Validate, scanners, TVs and Wi-Fi.
- *Presence-type:* Port Health staff, AVSEC officers.
- *Condition-type:* cleanliness, electricals/lighting.
- *Count-type:* trolleys, counter chairs, queue controls, weighing scales and counter telephones. Several carry a baseline in the question text, for example "NO. OF QUEUE CONTROLS AVAILABLE (NO. DEPLOYED 200)".
- *Comment-type:* "ANY OTHER COMMENTS".

**Defect observations.** Defects are written as sentences, nested under location headings:

- Terminal, then area (check-in hall, gate C4, customs area).
- Facility (washroom DR 479, "Wok Inn male washroom", "Aleeno café washroom").
- Sub-position ("3rd cubicle", "4th basin tap", "counters 19, 20, 21, 24 and 27").

**Action entries** (the REMARKS column). Entries fall into these classes:

| Class | Examples | Count (of 686) |
|---|---|---|
| Party notified, no further detail | "FMCC informed", "FMCC notified", "Reported to FMCC" | 535 (78%) |
| Ticket reference | "GA011715" | 98 (14%) |
| Pending | "Pending", "Pending issue" | 23 (3%) |
| Fixed | "FIXED" | 7 (1%) |
| Other party notified | "ICT informed", "SITA informed", "Electromech informed" | 8 (1%) |
| Other | Headers, planned works ("Area cordoned for renovation"), narrative | 15 (2%) |

### 2.3 Findings that drive the design

| ID | Finding | Evidence | Design consequence |
|---|---|---|---|
| **E1** | **Finding and action are linked only by line position.** D2 has 193 detail lines against 158 remark lines. D4 has 274 against 225. D3 has 413 against 175. The detail column also contains headings and status answers, so the pairing is unreliable, and nothing says that a remark belongs to a particular defect. | Counts per document | Model an *Action* as a record attached to a specific *Defect* (§5, FR-ACT). |
| **E2** | **Actions carry no accountability.** 78% of actions are the words "FMCC informed". There is no time, no person, no channel and no reference. FMCC's response time cannot be measured. | §2.2 table | Every notification is a timestamped event with sender, recipient party, channel and optional ticket (FR-ACT-01–04). |
| **E3** | **The same defect is typed in two documents on the same day.** 5 ticket numbers appear in both D1 and D2 (GA011435, GA011440, GA011441, GA011663, GA011856). 7 appear in both D3 and D4 (GA00591, GA006310, GA007660, GA011409, GA011414, GA011456, GA011500). Washroom DR 479, 485, 335 and others are repeated sentence by sentence. | Cross-document ticket match | One *Defect* record, many *Observations*. The duty report and the walkthrough are *views* of the same data (§1.3, FR-DEF-02). |
| **E4** | **Ticket references are inconsistent.** 10 of 98 do not match `GA` plus six digits: `GA 001025`, `GA 007534`, `GA 0111448`, `GA1105018`, `GA0116622`, `GA00591`, `GA00598`, `GA00705`, `GA00615`. Some contain a space. Some have too few or too many digits. | Pattern check | Validate ticket format, with the pattern configurable (FR-ACT-06). Pending confirmation of the format (§13 Q2). |
| **E5** | **Spelling and vocabulary vary.** "diffused/defused" bulbs, "FMCC informed/Informed/notified/FCC informed/infomed/Reported to FMCC", "Isle" for aisle, "whole" for hole, "flashing knob" for flushing knob. Serviceability is written as Serviceable, Not Serviceable, Unserviceable, Out of service, Faulty, "Serviceable but not in use", "Serviceable at some areas". | Whole corpus | Controlled vocabularies and structured status fields (FR-TPL, Appendix A). |
| **E6** | **Counts are unvalidated and the answer unit varies.** D3 Aisle C reads "queue controls available (no. deployed 100)" with the answer "178 Available", which is more than deployed. D3 Aisle A reads deployed 200, answer "150". D1 reads deployed 100, answer "A total of 53". For counter telephones, D3 Aisle B answers "3 serviceable" while Aisles A and C answer only "Serviceable", so one question is answered with a count in one place and a status in another. | D1, D3 | Typed numeric fields, a baseline per location, computed availability %, and range warnings (FR-DUT-04, 05). |
| **E7** | **Question and answer are also matched by position.** A single ITEM cell holds from a handful to several dozen questions (D3 check-in is the largest). The DETAILS cell holds the answers in a separate stack. D1 Arrivals item 4 lists seven sub-questions but the answers are four words followed by free text. | D1, D3 | Questions and answers are stored as paired records, never as two parallel lists. |
| **E8** | **Defects recur and persist.** "Bins without covers and with faulty pedals", "hand dryer not serviceable", "soap dispenser not serviceable", "leakage" and "diffused bulb" appear in every document. Approximate keyword counts across the four documents are: `bin` 100, `hand dryer` 55, `leak` 63, `soap` 44, `diffused/defused` 61, `rusty` 26. D1 says "Most outstanding issues remain unresolved". Previous loggings are not enumerated. | Keyword count | Carry-forward of open defects into every shift report with age and recurrence (FR-CAR). |
| **E9** | **Severity is absent although some items are safety-relevant.** Examples: "exposed wires on the floor beside seats at Gate 4", "exposed hanging cables", "sharp and exposed carousel edge posing a potential safety risk to staff", "broken tiles", "rusty sockets", "leakage onto passenger seats", "wet carpet in tunnel", "missing window panel at gate 5 covered with a board", "podium glass smashed". All are in the same flat list as "slightly detached hand dryer". | D1–D4 | Severity classification with a safety-critical flag and immediate escalation (FR-DEF-05, 06). |
| **E10** | **Weather and works context are buried in prose.** "Leakage … whenever it rains" occurs repeatedly. Renovation cordons ("Area cordoned for renovation", "Works still ongoing") explain missing FIDS, counter shields and carousels. | D1, D2 | Structured *condition trigger* (rain) and *planned works / cordon* link, so that known planned works do not look like new faults (FR-DEF-08, 09). |
| **E11** | **Rooms are identified by code.** Washrooms carry DR numbers (DR 014–016, 335–337, 350–358, 382–386, 447–451, 467–474, 479–485, 510–520). Lifts, escalators and elevators are numbered (Lift 1, 2, 6, 11, 13, 14, 15). Gates (C2–C8B in T3; 3, 4, 5 in T2), carousels (1–4), counters (1–46) and aisles (A, B, C) are identified too. Today these appear inconsistently in sentences ("DR 355" under "Gate C5" in one place and "Gate C7" in another). | D3, D4 | A location registry with stable IDs (FR-LOC). |
| **E12** | **Document identity is weak.** None of the four documents carries a date in its body. The date of D1 and D2 survives only in the file name, and D3 and D4 have no date at all. Item numbers repeat (D1 Departures shows "5" twice). Stakeholder Activity (D1 item 8, both tables) is blank. | D1–D4 | Mandatory date, shift, terminal and author on every report. Structured stakeholder activity (FR-SHF, FR-STK). |
| **E13** | **Officers cover wide areas on foot.** In D1 one officer is listed for both the Arrival Hall and Departures. D3 lists five officers across Departures, Departure Lounge, OCC and PA. The reports themselves record weak signal ("Poor Wifi signal at Café Aleeno and Lift area"), and gates and tunnels are likely to be worse (assumption, to verify on site). | D1, D3 | Mobile-first, offline-capable capture with location pre-filled from a QR/NFC tag or a pick-list (NFR-AVL-02, FR-DUT-08). |

---

## 3. Overall description

### 3.1 Product perspective

TORP is a new, self-contained web application that integrates loosely with the FMCC work-order process, e-mail/messaging and the corporate identity provider. Today these are manual steps recorded as "FMCC informed". It is not a replacement for the facility-maintenance system. It is the airport operator's **operational record and escalation layer** that sits in front of it.

```
 Duty officers (mobile / desktop)               Management / auditors
          |                                             |
          v                                             v
  +------------------------------ TORP -------------------------------+
  |  Capture      Defect register     Escalation      Reports &       |
  |  (checklists, (single source      (notify FMCC,   dashboards      |
  |   walkthrough) of truth)           ICT, SITA ...) (DOCX/PDF/XLSX) |
  +--------------------+------------------------+--------------------+
                       |                        |
              FMCC work-order system      E-mail / SMS / chat
              (ticket numbers "GA...")    (notification channels)
```

### 3.2 User classes

| Role | Description | Key needs |
|---|---|---|
| **Duty Officer** (primary) | Patrols a zone each shift and records status and defects. Typically 1–5 per terminal per shift. | Fast entry on a phone or tablet, one entry per defect, no retyping, works offline. |
| **Shift Lead / Duty Manager** | Reviews and signs off the shift report and receives handover. | See what is critical and what is open. Approve, reject or amend. Hand over. |
| **Operations Manager / Management** | Consumes summaries. | Ageing, recurrence, response-time and trend dashboards. |
| **FMCC Coordinator** *(external)* | Receives defect notifications and works them. | A clear list of what is assigned, with location and photo. A way to acknowledge, update and mark fixed. |
| **Other service parties** *(external)* | ICT, SITA, Electromech Unit, concessionaires and others. | The same assigned-list capability, limited to their own items. |
| **Safety / Compliance Officer** | Reviews safety-critical items and audits. | Filter by safety flag and audit trail. |
| **System Administrator** | Maintains reference data, templates, users. | Configure without code changes (§6.1, §6.2). |
| **Auditor** (read-only) | Verifies records. | Immutable history, exports. |

### 3.3 Operating environment

- Primary: modern mobile browsers (Android and iOS) and desktop Chrome, Edge and Firefox, delivered as an installable Progressive Web App (PWA).
- Connectivity: Wi-Fi in some areas is poor or absent (the reports themselves log "Poor Wifi signal"). The application must work offline (§8.4).
- 24/7 operation across shifts.

### 3.4 Design constraints

1. The generated Word report must remain recognisable to current readers (§7).
2. FMCC remains the repair system of record. Integration is by e-mail/API/import, depending on what FMCC supports (§9, Q1).
3. Personal data of staff (names, posts) is processed under the applicable data-protection law (§8.5).
4. Photo capture in airside and security-sensitive areas may be restricted by airport policy. This must be confirmed (Q10).

### 3.5 Assumptions and dependencies

Listed in §13.1.

---

## 4. Business process: as-is and to-be

### 4.1 As-is

*Inferred from the documents. Steps not visible in the documents (paper notes, phone calls, e-mail circulation) must be confirmed in discovery (§13.1 A6).*

```
Shift starts -> take over from previous shift (Word file from shared drive)
  -> patrol zone, note faults on paper / phone
  -> phone or message FMCC / ICT (no record kept except the phrase "FMCC informed")
  -> type findings into duty-report Word table (retype yesterday's unresolved items)
  -> separately compile the walkthrough document (retype again)
  -> circulate files by e-mail
Shift ends -> hand over verbally; "Previous loggings: some issues remain unresolved"
```

### 4.2 To-be

```
  Start shift                    Patrol / inspect                Escalate
 +----------------+        +-------------------------+     +----------------------+
 | Open shift,    |        | Walk the zone. Tap each |     | System groups new    |
 | confirm roster,|  --->  | checklist item. For a   | --> | defects by party and |
 | review carried |        | fault: pick location,   |     | sends one batched    |
 | -forward open  |        | category, severity,     |     | notification.        |
 | defects        |        | photo. Confirm or close |     | Ticket no. attached  |
 +----------------+        | carried defects.        |     | when FMCC replies.   |
                           +-------------------------+     +----------+-----------+
                                                                        |
  Close shift                                                           v
 +----------------+        +-------------------------+     +----------------------+
 | Lead reviews,  |  <---  | Report auto-generated:  | <-- | Defect lifecycle     |
 | signs off,     |        | duty report, walkthrough|     | updated by FMCC or   |
 | handover       |        | handover pack, in Word  |     | by officer; fixed    |
 | pack to next   |        | layout / PDF            |     | items verified       |
 | shift          |        +-------------------------+     +----------------------+
 +----------------+
```

### 4.3 Key scenarios

| ID | Scenario | Outcome |
|---|---|---|
| **S1** | Officer finds that the 3rd cubicle in the Customs female washroom has a broken lock | One *Defect* is created with the location "T2 > Arrivals > Customs > Female washroom > Cubicle 3". It appears in tonight's duty report, tomorrow's carry-forward, and the walkthrough, with no retyping. |
| **S2** | The same lock is still broken next shift | Officer confirms "still present". Age is shown (1 day), the recurrence count increments, the earlier ticket and notification history are retained. No new notification is sent unless the escalation rule fires. |
| **S3** | A roof leak appears at gate 4 each time it rains | The defect is flagged *weather-triggered (rain)*. Each recurrence is logged as an observation on the same defect, so frequency and rain correlation become visible. |
| **S4** | Exposed wires are found beside passenger seats | The officer marks the item *safety-critical*. The Duty Manager and FMCC are notified immediately with an acknowledgement deadline. The item leads the handover pack until closed. |
| **S5** | FMCC fixes the 4th tap | FMCC (or the officer) marks the defect "Fixed". The next patrol must **verify**. It is closed only after verification. A failed verification reopens it. |
| **S6** | Renovation cordons off carousels 1 and 2 | A *Planned Works / Cordon* record links the area. Related status answers show "Cordoned off". The defects inside are tagged "planned works", so they do not count as new faults in the metrics. |
| **S7** | Shift Lead reviews the report | The lead sees completeness (all mandatory checklist sections answered), counts of new, carried and closed defects, and any safety-critical items, then signs off. The report is locked. |

---

## 5. Domain and data requirements

### 5.1 Core concepts

| Entity | Purpose | Key attributes (illustrative) |
|---|---|---|
| **Site / Terminal / Zone** | Top levels of the location tree | name, code, type (Arrivals, Departures, Airside…) |
| **Location** | Any addressable place or asset position | id, parent, type (Facility, Gate, Carousel, Counter, Washroom, Cubicle, Lift…), code (e.g. `DR479`, `C4`), label, `qr_tag` |
| **Asset Category / Fault Type** | Controlled vocabulary | e.g. Lighting > Bulb diffused; Plumbing > Tap faulty; Structure > Ceiling open; Safety > Exposed wiring |
| **Party** | Organisation that receives work | FMCC, ICT, SITA, Electromech, Concessionaire, AVSEC, Port Health… |
| **User / Role** | Person and permissions | name, roles, post, party |
| **Shift** | One duty period for one terminal/zone | date, shift name, start, end, terminal, status |
| **Staff Assignment** | Who is posted where | shift, user, post (location), attendance |
| **Checklist Template** | Questions for a section | section, item, answer type, allowed values, baseline, mandatory |
| **Checklist Response** | An answer on a shift | shift, template item, location, value, time, author |
| **Duty Report** | The sign-off container for a shift | shift, status (draft, submitted, approved, locked), approver |
| **Walkthrough** | A dated inspection of an area | date, terminal, area list, inspector |
| **Observation** | One sighting of a defect in a report or walkthrough | defect, shift/walkthrough, observed-at, still-present flag, photo |
| **Defect** | The persistent fault record (the heart of the system) | id, location, category, fault type, quantity, description, severity, safety flag, status, first-seen, last-seen, age, recurrence count, owner party, links |
| **Action / Event** | Timeline entry on a defect | type (notified, ticketed, acknowledged, updated, fixed, verified, reopened, deferred), actor, party, channel, timestamp, note, ticket ref |
| **External Ticket** | Reference to an FMCC/ICT work order | system, number, status, linked defects |
| **Resource Count** | Count-type answer with baseline | resource (queue control, chair, scale…), location, deployed, available, serviceable |
| **Planned Works / Cordon** | Known works that explain closures | area, start, expected end, description, status |
| **Stakeholder Activity** | Structured entry for D1/D3 item 8 | stakeholder, activity, time, location, notes |
| **Attachment** | Photos and files | owner entity, file, captured-at, uploader |
| **Audit Log** | Immutable record of change | entity, field, old/new, actor, time |

### 5.2 Defect lifecycle

```
 NEW --notify--> REPORTED --ticket--> TICKETED --start--> IN_PROGRESS --fix--> FIXED --verify--> CLOSED
   \               \                     \                    \                    \
    \               \------------------- DEFERRED (planned works / awaiting parts / approved by manager)
     \                                                                               \
      +-------- DUPLICATE / WITHDRAWN (with reason; merged into another defect)      REOPENED (verification failed
                                                                                       or fault recurs) -> REPORTED
```

Rules:

- A defect is **never deleted**. It can be merged, withdrawn or closed with a reason.
- `FIXED` (claimed by FMCC or officer) is distinct from `CLOSED` (verified on patrol). In the sources these two are conflated as "FIXED".
- `DEFERRED` requires a reason and a review date. It covers "Pending" and "Area cordoned for renovation".
- Age is calculated from first-seen. Recurrence counts reopen events and weather-triggered re-sightings.

### 5.3 Data-quality rules (derived from E4–E7)

| Rule | Description |
|---|---|
| DQ-1 | Ticket reference matches a configurable pattern (default `^GA\d{6}$`). Spaces are stripped; a mismatch is a warning with an override reason. |
| DQ-2 | Count answers are non-negative integers. `available > deployed` raises a warning. |
| DQ-3 | Every defect has a location at "facility" depth or deeper, a category and a fault type. |
| DQ-4 | Every report has date, shift, terminal and author (E12). |
| DQ-5 | Free-text fields are spell-checked against the domain dictionary, and common errors are corrected on suggestion (diffused/defused, aisle/isle, hole/whole). |
| DQ-6 | A defect marked FIXED needs a source (party or officer) and timestamp. CLOSED needs a verifier. |
| DQ-7 | Locations use stable codes (E11). Free-typed room numbers are matched to the registry. |

---

## 6. Functional requirements

Each requirement is atomic and testable. "Priority" is the target for the **first production release (R1)**. Sections are numbered to match the traceability matrix (§12.3).

### 6.1 Location and asset registry (FR-LOC)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-LOC-01 | The system shall maintain a hierarchical location registry: Site > Terminal > Zone > Facility > Position (for example Cubicle 3, Basin tap 4). | M | E11 |
| FR-LOC-02 | Each location shall have a unique stable code and label, a type, and an optional QR/NFC tag value. | M | E11 |
| FR-LOC-03 | Administrators shall be able to add, rename, move, retire and merge locations without code changes. Retired locations keep their history. | M | – |
| FR-LOC-04 | The registry shall be pre-loaded with the inventory in Appendix B and validated by Operations. | M | E11 |
| FR-LOC-05 | A new terminal (for example T3 Arrivals) shall be added by configuration only. | S | §1.4 |
| FR-LOC-06 | The system shall support numbered position ordinals ("3rd cubicle", "counters 19–27") as structured values rather than text. | S | E11 |

### 6.2 Checklist templates (FR-TPL)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-TPL-01 | The system shall provide configurable checklist templates per terminal, zone and shift type, reproducing the sections in D1 and D3 (Take/Handover, Previous loggings, Staff strength, Frontage, Pre-departure, Check-in, Departure lounge, Gates, Baggage make-up/delivery, Arrivals, OCC, Stakeholder activity). | M | D1, D3 |
| FR-TPL-02 | Each checklist item shall have a type: **Status** (controlled list), **Presence** (Present/Absent), **Condition** (Good/Satisfactory/Poor), **Count** (integer with optional baseline), or **Comment**. | M | E5, E7 |
| FR-TPL-03 | Status items shall use the canonical set in Appendix A.1 (Operational, Degraded, Out of service, Not in use, Cordoned off, Not applicable) with the original phrase kept as a display alias. | M | E5 |
| FR-TPL-04 | Selecting a non-normal answer (Degraded, Out of service, Absent, Poor) shall open the defect-creation dialogue pre-filled with the item's location and category. | M | E1 |
| FR-TPL-05 | Administrators shall edit templates (add, remove, reorder items, change baselines such as "deployed 200") with version history. Reports keep the template version they were created with. | M | E6 |
| FR-TPL-06 | Mandatory items shall be marked, and a report cannot be submitted with unanswered mandatory items. | M | E12 |
| FR-TPL-07 | Item numbering shall be generated by the system, so duplicates (E12) cannot occur. | S | E12 |

### 6.3 Shifts, staffing and handover (FR-SHF)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-SHF-01 | The system shall support configurable shift patterns (name, start, end) and open a shift record per terminal/zone and date. Observed examples: shift CHARLIE at 06:00 (D1) and at 18:00 (D3). | M | D1, D3 |
| FR-SHF-02 | The officer shall record staff strength: each person, their post and presence. The total shall be computed ("STAFF STRENGTH (5)"). | M | D1, D3 |
| FR-SHF-03 | Handover shall be an explicit electronic step. The outgoing lead creates a handover pack (open defects, safety-critical items, pending actions, free-text notes). The incoming lead acknowledges it, and the acknowledgement time becomes the "took over at" time. | M | D1, D3 |
| FR-SHF-04 | The "Previous loggings" section shall be generated from the open defects in scope, not typed (see FR-CAR-01). | M | E8 |
| FR-SHF-05 | The system shall block closing a shift while a safety-critical defect has no recorded notification, unless the lead records an override. | S | E9 |
| FR-SHF-06 | Overlapping assignments shall be supported (one officer covering several posts). | S | E13 |

### 6.4 Duty-report capture (FR-DUT)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-DUT-01 | The system shall present the shift's checklist as a guided, section-by-section form with progress and completeness indicators. | M | D1, D3 |
| FR-DUT-02 | Entry shall need at most **two taps** for a normal answer. A "Mark section all-normal" action shall be offered, with exceptions entered afterwards. | M | E13 |
| FR-DUT-03 | Defect entry shall use location pick (tree, search or QR scan), category, fault type, optional quantity/ordinal, severity, optional photo(s), and optional note. It shall support **multi-select** ("3rd and 4th cubicle tissue holders faulty" creates one defect with positions 3 and 4). | M | E1, E5 |
| FR-DUT-04 | Count-type items shall show the baseline and compute availability, for example "150 of 200 (75%)". | M | E6 |
| FR-DUT-05 | The system shall warn if available exceeds deployed or a count falls below a configurable threshold. | S | E6 |
| FR-DUT-06 | Free-text comments shall remain available ("ANY OTHER COMMENTS") and may be promoted to a defect with one tap. | M | D1, D3 |
| FR-DUT-07 | The system shall allow a pasted block of legacy text (one line per issue) to be parsed into draft defects for review (§6.13). | C | E5 |
| FR-DUT-08 | All capture shall work offline and synchronise automatically (NFR-AVL-02). | M | E13 |
| FR-DUT-09 | The user shall be able to save drafts, and multiple officers shall be able to contribute to one shift report concurrently. | S | E13 |
| FR-DUT-10 | Each answer and defect shall record author and time automatically. | M | E12 |

### 6.5 Walkthrough capture (FR-WLK)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-WLK-01 | A walkthrough shall be a dated inspection of a terminal's configured areas (for example Check-in, Gates, Baggage hall, Customs area, Washrooms by DR number). | M | D2, D4 |
| FR-WLK-02 | During a walkthrough the officer shall see the existing open defects for the area and confirm each as **still present**, **fixed (needs verification)** or **changed**, so only new defects need to be typed. | M | E3, E8 |
| FR-WLK-03 | The walkthrough shall support an "OTHERS" free area for items outside the template, with the same defect form. | M | D4 |
| FR-WLK-04 | A defect found during a duty-report patrol shall appear in the same day's walkthrough automatically, and vice versa (same record, E3). | M | E3 |
| FR-WLK-05 | The walkthrough shall support a **route mode**: an ordered list of areas with progress, for example gate C2 > C3 > … > C8B. | S | E13 |
| FR-WLK-06 | The walkthrough shall be signed off separately by the inspector or lead and locked. | M | E12 |

### 6.6 Defect register and lifecycle (FR-DEF)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-DEF-01 | The system shall maintain one persistent **Defect** per distinct fault, with a unique reference (for example `TORP-000123`) independent of any external ticket. | M | E1, E3 |
| FR-DEF-02 | A new report entry that matches an open defect (same location, category and fault type) shall be offered as an **Observation of the existing defect** instead of creating a duplicate. The officer can override. | M | E3 |
| FR-DEF-03 | The system shall implement the lifecycle in §5.2 with the rules stated there, and enforce permitted transitions by role. | M | E2 |
| FR-DEF-04 | Each defect shall store: location, category, fault type, quantity/ordinal, description, severity, safety flag, owner party, first-seen, last-seen, age, recurrence count and status. | M | – |
| FR-DEF-05 | Severity shall be one of Critical (safety / operations stopped), High, Medium, Low, with default rules by category that the officer may raise. | M | E9 |
| FR-DEF-06 | A **safety-critical flag** shall trigger immediate escalation (FR-ACT-05). Candidate categories to configure: exposed live wiring, sharp edges, missing glass, trip hazards, leakage onto seats or into electrical areas. | M | E9 |
| FR-DEF-07 | A defect can have multiple photos, notes and linked observations. | S | – |
| FR-DEF-08 | A defect can be flagged **weather-triggered** (rain), and each sighting records the observed weather as Rain, Dry or Unknown. | S | E10 |
| FR-DEF-09 | A defect can be linked to a **Planned Works / Cordon** record and to a status of "Deferred – planned works" with a review date. | M | E10 |
| FR-DEF-10 | Defects can be merged or marked duplicate with a reason. History is retained. | S | E3 |
| FR-DEF-11 | Defects can be grouped by a **root-cause parent** (for example one roof leak causing several ceiling stains) to avoid ten "leak" lines for one cause. | C | E8 |
| FR-DEF-12 | **Bulk operations** shall be provided: notify FMCC about selected defects, update status for a set of defects, move a set of defects to Deferred. | M | E2 |
| FR-DEF-13 | A register view shall support filter and sort by terminal, area, category, severity, status, party, age, recurrence and safety flag, with saved filters. | M | – |

### 6.7 Actions, notifications and external tickets (FR-ACT)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-ACT-01 | Every escalation shall be recorded as an **Action** attached to one or more defects, with party, channel, actor, timestamp and optional note. This replaces the phrase "FMCC informed". | M | E1, E2 |
| FR-ACT-02 | The system shall send notifications to parties by e-mail at minimum. SMS/chat (for example WhatsApp or Teams) is a configurable extension. A notification shall be one **batched message** per party per run, grouped by location, with photos and defect references. | M | E2 |
| FR-ACT-03 | Parties can **acknowledge** a notification (link or reply). Acknowledgement time and the party's response time are recorded. | S | E2 |
| FR-ACT-04 | The system shall record external ticket numbers against one or many defects (many-to-many), including by quick-entry from the register. | M | E2, E3 |
| FR-ACT-05 | Safety-critical defects shall notify the Duty Manager and the owning party immediately, with a configurable acknowledgement deadline and automatic re-escalation if the deadline is missed. | M | E9 |
| FR-ACT-06 | Ticket references shall be validated against the configurable pattern (DQ-1). | M | E4 |
| FR-ACT-07 | A configurable **escalation policy** shall raise a defect's visibility when it exceeds an age threshold for its severity without progress, for example Critical 4 h, High 3 days, Medium 14 days, Low 30 days. Thresholds are for Operations to set. | S | E2, E8 |
| FR-ACT-08 | Party users (FMCC, ICT, SITA, Electromech) shall have a scoped view of their own assigned defects. They can acknowledge, add ticket numbers, update status to In progress or Fixed, and add notes. They cannot close. | S | E2 |
| FR-ACT-09 | Where FMCC exposes an API or a structured export, the system shall import ticket status updates automatically (§9). | C | Q1 |
| FR-ACT-10 | "Pending" shall become a structured state with a reason (awaiting FMCC, awaiting parts, awaiting approval, planned works) and an expected date. | M | E1, E10 |

### 6.8 Carry-forward and recurrence (FR-CAR)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-CAR-01 | At shift start the system shall list all open defects in the officer's scope, with age and last action. The officer confirms each as Still present, Fixed (to verify) or Changed. Confirmation is one tap. | M | E8, D1 |
| FR-CAR-02 | Unconfirmed carried defects shall block report submission, or be carried automatically with an "unverified" mark if the lead allows. | S | E8 |
| FR-CAR-03 | The system shall compute recurrence: the count of reopen events and the interval between sightings, per defect and per location. | S | E8 |
| FR-CAR-04 | The system shall identify **hotspots**: locations and categories with repeated defects in a rolling period (for example washroom DR 335 with five soap-dispenser failures in 30 days). | S | E8 |

### 6.9 Reports and distribution (FR-RPT)

The generated documents and their layout are specified in §7.

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-RPT-01 | The system shall generate the **Shift Duty Report** per terminal/zone/shift in the layout of D1/D3 as a Word document and as a PDF. | M | D1, D3 |
| FR-RPT-02 | The system shall generate the **Walkthrough Report** per terminal and date in the layout of D2/D4 as Word and PDF. | M | D2, D4 |
| FR-RPT-03 | The system shall generate a **Handover Pack**, a **Consolidated Daily Report** across terminals, and an **Open Defects Register** (CSV/XLSX/PDF). | M | E8 |
| FR-RPT-04 | Reports shall be generated from data on demand, in under 30 seconds (NFR-PERF-03), and be reproducible: regenerating a signed report gives identical content. | M | – |
| FR-RPT-05 | Submitted reports shall be locked. Corrections are made by an **addendum** with reason and author, never by silently editing. | M | NFR-AUD |
| FR-RPT-06 | The system shall support distribution lists by terminal and report type, with scheduled and manual send by e-mail. | S | – |
| FR-RPT-07 | Management summaries (weekly/monthly) shall show defects raised, closed, ageing, recurrence, response times and counts per party and per area. | S | E2, E8 |
| FR-RPT-08 | Report templates (headings, logo, ordering) shall be configurable by administrators. | S | E12 |
| FR-RPT-09 | All reports shall carry date, shift, terminal, author, approver, template version and generation time in the header or footer. | M | E12 |

### 6.10 Dashboards and analytics (FR-DSH)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-DSH-01 | A **live terminal status board** shall show for each terminal the count of open defects by severity, safety-critical items, out-of-service assets (FIDS, lifts, escalators, A/C, carousels) and shift completion status. | S | – |
| FR-DSH-02 | An **ageing view** shall show open defects by age bucket (0–1 d, 2–7 d, 8–30 d, 31 d+) and party. | S | E2 |
| FR-DSH-03 | A **party performance view** shall show time from notification to acknowledgement, to ticket, to fix and to closure, by party. | S | E2 |
| FR-DSH-04 | A **category and hotspot view** shall show top fault types and top locations (for example bins, hand dryers, leaks, soap dispensers, diffused bulbs). | S | E8 |
| FR-DSH-05 | A **resource availability view** shall trend deployed vs available queue controls, chairs and scales. | C | E6 |
| FR-DSH-06 | A **weather correlation view** shall show leak sightings against rain flags. | C | E10 |
| FR-DSH-07 | All charts shall be exportable and use accessible colour palettes and data tables. | S | – |

### 6.11 Stakeholder activity (FR-STK)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-STK-01 | The system shall capture structured stakeholder activity per shift: stakeholder (airline, ground handler, AVSEC, Immigration, Customs, Port Health, concessionaire, contractor), activity type, time, location and notes. | M | E12 |
| FR-STK-02 | Stakeholder activity shall be selectable from a configurable list. | S | E12 |
| FR-STK-03 | A stakeholder activity may optionally link to a defect or planned-works record (for example contractors working on renovation). | C | E10 |

### 6.12 Search, history and audit (FR-AUD)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-AUD-01 | Full-text and structured search shall cover defects, reports, actions and notes. | M | – |
| FR-AUD-02 | Each entity shall show its complete history: who changed which field, when, from what to what. | M | NFR-AUD |
| FR-AUD-03 | Audit records shall be append-only and not editable by any role. | M | NFR-AUD |
| FR-AUD-04 | Users shall be able to open any historical report exactly as it was signed. | M | FR-RPT-05 |

### 6.13 Legacy import and assisted entry (FR-MIG)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-MIG-01 | The system shall import historical Word duty reports and walkthroughs of the D1–D4 layouts into draft observations and defects for human review, with confidence flags. | S | §11 |
| FR-MIG-02 | The import shall not auto-confirm matches between old and new defects. A reviewer accepts or rejects each. | M | E3 |
| FR-MIG-03 | The vocabulary mapping in Appendix A shall be used to normalise imported text, and unmapped text shall be preserved verbatim. | M | E5 |
| FR-MIG-04 | An assisted entry mode (C) shall convert pasted free-text lines into structured draft defects, using the location registry and controlled vocabularies. Output is always a draft for confirmation. | C | E5 |

### 6.14 Users, access and administration (FR-ADM)

| ID | Requirement | Pri | Evidence |
|---|---|---|---|
| FR-ADM-01 | Authentication shall use the organisation's identity provider (OIDC/SAML SSO) where available, with local accounts and MFA as a fallback. | M | NFR-SEC |
| FR-ADM-02 | Role-based access control shall implement the roles in §3.2, scoped by terminal and by party. | M | NFR-SEC |
| FR-ADM-03 | Administrators manage users, roles, parties, vocabularies, severity rules, escalation thresholds, shift patterns, distribution lists and templates through the UI. | M | – |
| FR-ADM-04 | Configuration changes shall be versioned and audited. | S | NFR-AUD |
| FR-ADM-05 | External party users can be invited with limited, expiring access. | S | FR-ACT-08 |

### 6.15 Acceptance criteria for the most critical requirements

**FR-DEF-02: duplicate prevention**

> *Given* an open defect "Female washroom DR 485, cubicle 3, door lock faulty".
> *When* an officer records "3rd cubicle door lock faulty" at the same location during the next shift.
> *Then* the system proposes the existing defect, and on confirmation adds an observation, raises the age and last-seen, and creates **no new defect**.

**FR-ACT-01 / FR-ACT-02: structured escalation**

> *Given* three new defects at gate C4 are owned by FMCC.
> *When* the officer chooses "Notify FMCC".
> *Then* one message is sent listing the three defects with location, photo and reference. Each defect records an Action with timestamp, officer, party and channel. The defect status becomes REPORTED, and the daily report shows "Notified FMCC at 09:42" rather than "FMCC informed".

**FR-DEF-06 / FR-ACT-05: safety-critical**

> *Given* an officer marks "Exposed wires on the floor beside seats at Gate 4" as safety-critical.
> *When* the defect is saved.
> *Then* the Duty Manager and FMCC are notified within one minute (when online) with an acknowledgement deadline. If no acknowledgement arrives by the deadline, the item is re-escalated. The item heads the handover pack and the dashboard until it is closed.

**FR-RPT-01: report parity**

> *Given* a signed shift for Terminal 3 Departures.
> *When* a user generates the Word report.
> *Then* the document contains the same sections, in the same order, as the current T3 Departure Duty Report, with every checklist answer paired with its question and every defect paired with its action and ticket, and no line is orphaned from its remark.

**FR-CAR-01: carry-forward**

> *Given* 40 open defects in the officer's zone.
> *When* the shift opens.
> *Then* all 40 are listed with age and last action, the officer can confirm all as "Still present" in one bulk action, and the "Previous loggings" section of the report shows the counts and list automatically.

---

## 7. Report output requirements

The system **stores structured data** and **renders documents**. The two source report families are reproduced as rendered views.

### 7.1 Shift Duty Report: layout

| Section | Content | Data source |
|---|---|---|
| Header | Terminal, date, shift name, hours | Shift |
| 1 Hand/Take over | "Shift X took over operations at HH:MM" | Handover acknowledgement (FR-SHF-03) |
| 2 Previous loggings | Counts and list of open carried defects | Defects (FR-CAR-01) |
| 3 Staff strength | Total, with name, post and presence | Staff Assignment |
| 4 Frontage / curbside | Trolleys, queue controls, cleanliness, lifts, lighting, comments | Checklist |
| 5 Pre-departure screening | Doors, A/C, thermal scanner, Port Health, lighting, comments | Checklist |
| 6 Check-in | PA, FIDS, A/C, CUSS, lifts, per aisle: chute, BHS, scales, FIDS, chairs, phones, queue controls; washrooms by DR number; food court | Checklist, Resource Count, Defects |
| 7 Departure lounge | Immigration, central screening, i-Validate, FIDS, seats, TVs, Wi-Fi, lifts/escalators, per gate: boarding system, FIDS, scales, chairs, phones, queue controls, door, tunnel carpet; washrooms by gate area | Checklist, Defects |
| 8 Baggage make-up | BHS, chute, X-ray, lighting, washrooms | Checklist |
| 9 OCC (Level 5) | Staircase, corridor, lighting, comments | Checklist |
| Arrivals sections (T2) | Customs area, arrival hall, baggage hall, baggage delivery | Checklist, Defects |
| Stakeholder activity | Structured list | Stakeholder Activity |
| Remarks / action taken | **For each defect line**: status, "Notified FMCC 09:42", ticket `GA011715`, pending reason | Actions (never a free-floating list) |
| Sign-off | Author, approver, times | Duty Report |

### 7.2 Walkthrough Report: layout

Areas follow the current documents. Examples are Information Office/Upper Concourse, Airside, Check-in, Lounge, Gates, Transit, Immigration, Baggage Hall, Customs Area/Airport Square (T2) and Boarding Gates/Departure Check-in with gates C2–C8B, aisles A–C, washrooms by DR and Others (T3). Each defect line shows its location, text, severity, age, action(s) and ticket.

### 7.3 Output rules

1. Every defect line carries its action and ticket **in the same row** (fixes E1).
2. A new defect is flagged **NEW**; a carried defect shows **Day n**. Age is visible.
3. Count items show "available / deployed".
4. Status answers print the canonical value, with the original wording available as a style option for continuity.
5. Formats: DOCX (editable layout parity), PDF (archival), XLSX/CSV (register and analytics).
6. Page header and footer carry report identity (FR-RPT-09).

---

## 8. Non-functional requirements

### 8.1 Performance and capacity (NFR-PERF)

| ID | Requirement | Pri |
|---|---|---|
| NFR-PERF-01 | 95% of interactive actions (navigation, save, filter) complete in under 1 s at normal load, on a mid-range phone on a 4G connection. | M |
| NFR-PERF-02 | The system supports at least 100 concurrent users and 50 shifts open at once. | M |
| NFR-PERF-03 | A full shift report (about 500 lines) generates in under 30 s. A daily consolidated report generates in under 90 s. | M |
| NFR-PERF-04 | Sizing basis: the four supplied documents hold roughly 1,150 detail lines (§2.1). Assuming about 6 shifts-and-walkthroughs per day, that is on the order of 7,000 lines per day. Design for 10× that and 10 years of retention without redesign. Confirm the real daily volume in discovery. | M |

### 8.2 Usability and accessibility (NFR-USE)

| ID | Requirement | Pri |
|---|---|---|
| NFR-USE-01 | A new officer shall complete a representative shift report after one hour of training. | S |
| NFR-USE-02 | The mobile UI shall be usable one-handed, with touch targets of at least 44 × 44 px. | M |
| NFR-USE-03 | Wording shall follow plain, verb-first labels and the terms used in the current reports (for example "Notify FMCC"). | S |
| NFR-USE-04 | The UI shall conform to WCAG 2.2 AA, including colour contrast, keyboard operation and screen-reader labels. | M |
| NFR-USE-05 | Light and dark themes, readable in bright terminal areas and at night. | S |
| NFR-USE-06 | Language is English. All text is externalised for later translation. | M |
| NFR-USE-07 | Error messages shall state what happened and how to recover, and never blame the user. | S |

### 8.3 Reliability and recovery (NFR-REL)

| ID | Requirement | Pri |
|---|---|---|
| NFR-REL-01 | Availability of the online service: 99.5% per month, measured outside agreed maintenance windows. | M |
| NFR-REL-02 | Recovery point objective (RPO): 15 minutes. Recovery time objective (RTO): 4 hours. | M |
| NFR-REL-03 | Backups are automated, encrypted, and restore-tested at least quarterly. | M |
| NFR-REL-04 | No data loss on device failure: unsynced offline data is retained until acknowledged by the server. | M |

### 8.4 Offline and synchronisation (NFR-AVL)

| ID | Requirement | Pri |
|---|---|---|
| NFR-AVL-01 | The installed PWA shall open and show the user's shift, templates, locations and open defects with no network. | M |
| NFR-AVL-02 | All capture operations (answers, defects, observations, photos, notes) shall queue offline and sync when connectivity returns, in order, with visible sync status. | M |
| NFR-AVL-03 | Conflicts (two officers edit the same defect offline) shall be resolved at field level with last-writer-wins for status events kept in the timeline, and a visible merge note. No silent data loss. | M |
| NFR-AVL-04 | Photos are compressed on the device and uploaded in the background. | S |

### 8.5 Security and privacy (NFR-SEC)

| ID | Requirement | Pri |
|---|---|---|
| NFR-SEC-01 | TLS 1.2+ in transit; AES-256 encryption at rest for data and backups. | M |
| NFR-SEC-02 | Authentication via SSO with MFA for managers, administrators and external parties. | M |
| NFR-SEC-03 | Least-privilege RBAC scoped by terminal and party (FR-ADM-02). | M |
| NFR-SEC-04 | The application meets OWASP ASVS Level 2 and has penetration testing before go-live. | M |
| NFR-SEC-05 | Photos and notes are stored in a private bucket; access is by signed, expiring URL. | M |
| NFR-SEC-06 | Photos of security-sensitive airside areas follow the airport's policy (Q10). Photos can be disabled per area. | M |
| NFR-SEC-07 | Staff personal data (names, posts) are processed lawfully. The applicable data-protection law is confirmed in Q9 (for example the Ghana Data Protection Act 2012, if the deployment is in Ghana). | M |
| NFR-SEC-08 | External party users see only the defects assigned to their party. | M |
| NFR-SEC-09 | Session timeout, device-loss remote sign-out, and no secrets in client code. | S |

### 8.6 Auditability and retention (NFR-AUD)

| ID | Requirement | Pri |
|---|---|---|
| NFR-AUD-01 | Every create, update and status change is audited (who, when, old, new). | M |
| NFR-AUD-02 | Signed reports are immutable (FR-RPT-05). | M |
| NFR-AUD-03 | Retention is configurable per record type. The default is 7 years, subject to the airport's records policy (Q8). | M |

### 8.7 Maintainability and operability (NFR-OPS)

| ID | Requirement | Pri |
|---|---|---|
| NFR-OPS-01 | Reference data, templates, vocabularies and thresholds are configuration, not code (FR-ADM-03). | M |
| NFR-OPS-02 | Infrastructure as code, automated CI/CD, automated tests (unit, integration, end-to-end) with at least 80% coverage of domain logic. | M |
| NFR-OPS-03 | Structured logs, metrics and alerts for sync failures, notification failures and report-generation failures. | M |
| NFR-OPS-04 | Zero-downtime deployments. A rollback within 15 minutes. | S |
| NFR-OPS-05 | All dependencies are pinned and scanned for vulnerabilities. | M |

### 8.8 Compatibility (NFR-CMP)

| ID | Requirement | Pri |
|---|---|---|
| NFR-CMP-01 | Supported browsers: the latest two versions of Chrome, Edge, Safari and Firefox. Android 10+ and iOS 15+ as PWA. | M |
| NFR-CMP-02 | Generated DOCX opens without repair prompts in Microsoft Word 2016+ and LibreOffice. | M |
| NFR-CMP-03 | The system time zone is a site setting. All timestamps are stored in UTC. | M |

---

## 9. External interfaces and integrations

| ID | Interface | Direction | Mechanism | Priority |
|---|---|---|---|---|
| IF-01 | Identity provider | In | OIDC / SAML | M |
| IF-02 | E-mail service | Out/In | SMTP/API; reply-to parsing for acknowledgement | M |
| IF-03 | FMCC work-order system (ticket numbers `GA…`) | Out/In | **Unknown** (Q1). Order of preference: API; scheduled structured export/import; e-mail with a parsed reply; manual entry as the fallback. | M (manual), C (API) |
| IF-04 | SMS / chat gateway | Out | Provider API, behind a notification abstraction | C |
| IF-05 | ICT / SITA / Electromech | Out/In | Same as IF-02 via party users | S |
| IF-06 | Document conversion (DOCX to PDF) | Internal | Headless converter service | M |
| IF-07 | Weather source (rain flag) | In | Optional weather API or manual flag | C |
| IF-08 | Future: BHS/FIDS/BMS health feeds | In | Event API | W |

Integrations are implemented behind adapters so a change in FMCC's tooling does not alter the core model.

---

## 10. Proposed solution architecture

This section is a recommendation. Final selection follows an architecture decision record (ADR) once Q1, Q6 and Q7 are answered.

### 10.1 Logical architecture

```
 +--------------------+     HTTPS      +----------------------------------------+
 |  PWA client        | <------------> |  API gateway / backend (modular        |
 |  - offline store   |                |  monolith)                             |
 |  - sync queue      |                |   Auth | Locations | Templates         |
 |  - camera / QR     |                |   Shifts | Reports | Defects | Actions |
 +--------------------+                |   Notifications | Reporting | Admin    |
                                       +--------+---------------+---------------+
                                                |               |
                         +----------------------+----+    +-----+------------------+
                         | PostgreSQL (OLTP, JSONB   |    | Object storage         |
                         | for template payloads)    |    | (photos, generated     |
                         +----------------------+----+    |  reports)              |
                                                |         +------------------------+
                                  +-------------+-------------+
                                  | Background workers (queue) |
                                  |  - report render (DOCX/PDF)|
                                  |  - notifications           |
                                  |  - escalation timers       |
                                  |  - import / analytics ETL  |
                                  +----------------------------+
```

### 10.2 Technology recommendations

| Concern | Recommendation | Rationale |
|---|---|---|
| Client | TypeScript + React as an installable PWA, with IndexedDB and a service worker for offline | Meets NFR-AVL with one codebase for phone and desktop |
| API | REST (OpenAPI) on a TypeScript or Python framework; modular monolith | Small team and moderate load; avoids microservice overhead |
| Database | PostgreSQL | Relational integrity for defects and actions, JSONB for template payloads, full-text search |
| Files | S3-compatible object storage | Photos and generated reports |
| Queue/workers | Redis- or database-backed queue | Reports, notifications, timers |
| Report rendering | Template-driven DOCX generation, then PDF through headless LibreOffice | Layout parity with the present Word reports (FR-RPT-01) |
| Identity | OIDC with the organisation's IdP | NFR-SEC-02 |
| Hosting | Cloud region or on-premise, per Q7 | – |
| Observability | Structured logging, metrics and alerting | NFR-OPS-03 |

### 10.3 Key design decisions

| # | Decision | Reason |
|---|---|---|
| ADR-1 | **Defect as a first-class, long-lived entity; reports as views.** | E1, E3, E8 |
| ADR-2 | **Action events are append-only** and attached to defects. | E2, NFR-AUD |
| ADR-3 | **Templates are data** (versioned), not code. | E6, FR-TPL-05 |
| ADR-4 | **Offline-first with an operation log** (commands synced in order) instead of state overwrite. | E13, NFR-AVL-03 |
| ADR-5 | **FMCC integration behind an adapter** with manual fallback. | Q1 |
| ADR-6 | **Locations as an addressable tree with stable codes.** | E11 |
| ADR-7 | **Assisted entry (FR-MIG-04) only ever produces drafts.** | Safety of record |

---

## 11. Data migration and onboarding

| Step | Description |
|---|---|
| 1 | Build the location registry from Appendix B with Operations. Resolve inconsistencies such as DR 355, which appears under both Gate C5 and Gate C7 areas. |
| 2 | Load vocabularies (Appendix A), categories, fault types, parties and shift patterns. |
| 3 | Configure the templates reproducing D1 and D3 sections, with baselines (queue controls, chairs and so on). |
| 4 | Import the most recent 30–90 days of Word reports as drafts (FR-MIG-01). Reviewers accept matches. Only the **open** defects at cut-over are mandatory to migrate. |
| 5 | Pilot: one terminal, one shift pattern, 2–4 weeks, in parallel with Word reports. Compare generated reports with the Word reports. |
| 6 | Train officers (target one hour, NFR-USE-01), shift leads and FMCC contacts. |
| 7 | Cut over terminal by terminal. Word reports are retired once report parity is accepted (FR-RPT-01). |

---

## 12. Delivery plan, acceptance and traceability

### 12.1 Suggested releases

| Release | Scope | Outcome |
|---|---|---|
| **R0: Foundations** (about 4 weeks) | Identity, location registry, vocabularies, templates engine, audit log, CI/CD | Platform ready |
| **R1: Core capture and register** (about 8 weeks) | Shift, checklist capture (offline), defect register and lifecycle, carry-forward, structured actions with manual ticket entry, Shift Duty Report and Walkthrough Report in DOCX/PDF, handover | Retires the Word documents for one terminal |
| **R2: Escalation and visibility** (about 6 weeks) | Batched e-mail notification with acknowledgement, safety escalation, party portal, dashboards (status board, ageing, party performance), distribution lists, stakeholder activity | Management insight, accountability |
| **R3: Integration and intelligence** (about 6 weeks) | FMCC integration (if available), hotspot and weather analytics, assisted entry, legacy import, further terminals | Scale-out |

Estimates are indicative and depend on Q1, Q6 and team size.

### 12.2 Acceptance approach

- Each requirement is verified by test (unit, integration, end-to-end), demonstration, inspection or analysis. Acceptance criteria in §6.15 are mandatory scenarios.
- **Parallel-run acceptance** for R1: for two weeks, generated reports for one terminal are compared with the manually produced Word reports. Differences are classified as system defects or as manual-report errors (for example E1, E6).
- **Success measures** (targets to confirm with management):

| Measure | Baseline (today) | Target |
|---|---|---|
| Time to produce shift duty report | Not measured (the four supplied documents hold about 1,150 typed lines) | −50% |
| Defect lines typed twice (duty vs walkthrough) | 12+ ticket references shared across documents (E3) | 0 |
| Actions with timestamp, actor and channel | 0% | 100% |
| Actions with a ticket or reason for no ticket | 14% (98 of 686) | ≥ 95% |
| Malformed ticket references | 10 of 98 (10%) | 0 |
| Open defects with known age and owner | 0% | 100% |
| Safety-critical items acknowledged within deadline | Not measured | ≥ 95% |

### 12.3 Traceability (evidence to requirement)

| Finding | Requirements |
|---|---|
| E1 Action not linked to finding | FR-TPL-04, FR-DEF-01, FR-ACT-01, FR-ACT-04, FR-RPT output rule 1 |
| E2 No accountability | FR-ACT-01–05, FR-ACT-07, FR-ACT-08, FR-DSH-03, FR-DEF-12 |
| E3 Duplicate typing | FR-DEF-01, FR-DEF-02, FR-DEF-10, FR-WLK-02, FR-WLK-04, FR-MIG-02 |
| E4 Ticket format | DQ-1, FR-ACT-06 |
| E5 Vocabulary | FR-TPL-02, FR-TPL-03, DQ-5, Appendix A, FR-MIG-03 |
| E6 Counts | DQ-2, FR-DUT-04, FR-DUT-05, FR-TPL-05 |
| E7 Position-based pairing | FR-TPL-02, §7.3 rule 1 |
| E8 Recurrence | FR-CAR-01–04, FR-DSH-04, FR-DEF-11 |
| E9 Severity and safety | FR-DEF-05, FR-DEF-06, FR-ACT-05, FR-SHF-05 |
| E10 Weather and works | FR-DEF-08, FR-DEF-09, FR-ACT-10, FR-STK-03 |
| E11 Location identity | FR-LOC-01–06, DQ-7 |
| E12 Document identity | FR-TPL-06, FR-TPL-07, DQ-4, FR-RPT-09, FR-STK-01 |
| E13 Mobile and offline | FR-DUT-02, FR-DUT-08, NFR-AVL, NFR-USE-02 |

---

## 13. Assumptions, risks and open questions

### 13.1 Assumptions

| # | Assumption |
|---|---|
| A1 | "GA…" numbers are work-order tickets issued by FMCC (or the operator's work-order process) and are `GA` followed by six digits. |
| A2 | FMCC is the facility-management contractor or unit, and the main recipient of defects. |
| A3 | Shifts have names (for example CHARLIE) and fixed start times (06:00 and 18:00 are seen). |
| A4 | Duty officers have smartphones or tablets (company or personal) for capture, and that is acceptable under security policy. |
| A5 | The four documents are representative of the two report families used for all terminals. |
| A6 | Reports are circulated by e-mail today to a known distribution. |

### 13.2 Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Officers resist changing from a familiar Word table | Medium | High | Layout parity in output; two-tap entry; pilot with the real users; carry-forward saves time |
| R2 | FMCC will not use the system or provide a data feed | High | Medium | Make the tool valuable without FMCC (batched e-mail, manual ticket entry); the party portal is optional |
| R3 | Poor connectivity airside | High | High | Offline-first (NFR-AVL); test in tunnels and gates |
| R4 | Over-notification (alert fatigue) | Medium | Medium | Batching, severity-based rules, clear escalation policy |
| R5 | Location registry incomplete or inconsistent | Medium | Medium | Operations review (§11 step 1); allow "propose new location" with approval |
| R6 | Low-quality legacy data in migration | High | Low | Migrate open defects only; human review (FR-MIG-02) |
| R7 | Security restrictions on photos/devices | Medium | Medium | Q10; photos optional per area |
| R8 | Scope creep into a full CAFM/maintenance system | Medium | High | Hold the boundary: operational record and escalation only (§3.1) |
| R9 | Report parity disagreements | Medium | Medium | Parallel-run acceptance (§12.2); template configuration (FR-RPT-08) |

### 13.3 Open questions

| # | Question | Needed for | Owner |
|---|---|---|---|
| Q1 | What is the FMCC work-order system, and does it offer an API, export or e-mail interface? Who issues the `GA…` numbers? | IF-03, FR-ACT-09 | Ops / FMCC |
| Q2 | What is the official ticket format? Are the 10 malformed references typing errors or valid variants? | DQ-1 | FMCC |
| Q3 | Which shift names and times exist (CHARLIE seen at 06:00 and 18:00)? Are shifts per terminal or per airport? | FR-SHF-01 | Ops |
| Q4 | Are Terminal 3 Arrivals and any other terminals or areas to be covered, and when? | FR-LOC-05, plan | Ops |
| Q5 | Who reads the reports today and who must receive them? Is there a fixed deadline? | FR-RPT-06 | Ops |
| Q6 | Expected users and devices (number of officers, company devices or personal)? | NFR-PERF, sizing | Ops / ICT |
| Q7 | Hosting: cloud region, on-premise or hybrid? Any data-residency rule? | §10 | ICT |
| Q8 | Retention period for operational records? | NFR-AUD-03 | Compliance |
| Q9 | Which data-protection law and internal policy apply to staff data? | NFR-SEC-07 | Compliance |
| Q10 | May officers take photos in all areas (airside, security areas)? | NFR-SEC-06 | Security / AVSEC |
| Q11 | What severity policy and response targets (SLA) does FMCC work to? | FR-ACT-07 | Ops / FMCC |
| Q12 | Is there an existing safety-management or incident-reporting system to link safety-critical defects to? | FR-DEF-06 | Safety |
| Q13 | Who approves reports, and is a digital signature required? | FR-RPT-05 | Ops |
| Q14 | Is the Stakeholder Activity section meant to be filled (it is blank in the supplied reports)? What should it capture? | FR-STK | Ops |
| Q15 | What do **PWRM**, **GACL** and **OCC** stand for in your organisation? (§Appendix D lists inferred meanings.) | Glossary, labels | Ops |

---

## Appendix A: Vocabulary mapping (from the source reports)

### A.1 Status answers

| Canonical status | Source phrases observed |
|---|---|
| **Operational** | Serviceable, Functioning, In order, Audible, Stable, Available and serviceable |
| **Degraded** | Serviceable at some areas, Weak sensors, A/C lowered due to maintenance, Slow printer, "Blows very hot air", "Shaky/wobbly" |
| **Out of service** | Not serviceable, Unserviceable, Unserviceable (FIDS), Out of service, Faulty, Not functioning, Non-functional |
| **Not in use** | Serviceable but not in use (thermal scanner), "No more stationed there" |
| **Cordoned off** | Cordoned off, Area undergoing renovation, Cordoned for renovation |
| **Absent / Not available** | Absent, Not Available, Removed, Missing |
| **Present** | Present, Available |
| **Not applicable** | blank, "N/A" |

### A.2 Condition answers

| Canonical | Source phrases |
|---|---|
| **Good** | Good, Bright, Clean |
| **Satisfactory** | Satisfactory |
| **Poor** | Dim, Dirty, Worn out, Stained, "Not well lit" |

### A.3 Action vocabulary

| Source phrase | Canonical action | Notes |
|---|---|---|
| "FMCC informed", "FMCC Informed", "FMCC notified", "Reported to FMCC", "FMCC-Informed", "FCC informed", "FMCC infomed" | **Notified**: party = FMCC | No time, actor or channel in the source |
| "ICT informed", "SITA informed", "Electromech informed" | **Notified**: party = ICT / SITA / Electromech | |
| "GA011715" and similar | **Ticketed**: ticket number | Validate against the pattern (DQ-1) |
| "GA011715 – off" | **Ticketed** + free note | Mixed content: needs review |
| "Pending", "Pending issue" | **Deferred / Pending** | Reason required |
| "FIXED" | **Fixed (claimed)** | Verification required before Closed |
| "Area cordoned for renovation", "Works still ongoing", "Area undergoing renovation" | **Deferred: planned works** | |
| "Replacement is required to restore legibility. FMCC informed" | **Note** + **Notified** | Two facts in one cell |
| "A trolley facilitator was on standby for manual bags offload" | **Operational note** | Not a defect action |

### A.4 Spelling normalisation

| Seen | Normalise to |
|---|---|
| diffused / defused (bulb) | diffused (kept as the term of art for blown/non-lit bulbs; synonym "blown") |
| Isle | Aisle |
| whole (beside Valuejet office) | hole |
| flashing knob | flushing knob |
| out of guage | out of gauge |
| SSTAFF | Staff |
| peddles | pedals |
| carrousel | carousel |

### A.5 Defect category starter set (to confirm with FMCC)

| Category | Typical faults seen |
|---|---|
| Lighting | Bulb diffused, flickering, dim, sign light off, flood lights off |
| Ceiling and structure | Open ceiling, hole in ceiling, stained, peeling, leaking, dent, mold, broken tiles, broken railing/balustrade, missing window panel |
| Water and plumbing | Leakage, tap faulty/removed, urinal faulty, wobbly water closet, cistern cover missing, clogged sink, burst adaptor |
| Washroom fittings | Soap dispenser, tissue holder, hand dryer, bin cover/pedal, door lock/handle, flush knob, cubicle door, mirror rusty/stained, toilet brush base |
| Electrical | Socket faulty/rusty/exposed/disconnected, exposed wires/cables, trucking broken, charging station not working |
| HVAC | A/C not working, tunnel too warm, air curtain faulty, vents open/rusty/stained |
| Vertical transport and doors | Lift indicator/lights/buttons, escalator opening, automatic/sliding door fault, trolley exit door, swing door |
| Information systems | FIDS, BIDS, PA, TV, printer, Wi-Fi, telephone, CCTV, DSTV |
| Furniture and equipment | Counter chairs, counter shields, podium glass, weighing scale missing, signage faded |
| Cleanliness and odour | Pungent smell, stained carpet, congested with old items |
| Safety | Exposed wiring, sharp edge, trip hazard, falling railing, leakage onto seats |

---

## Appendix B: Location inventory (from the source reports)

**Terminal 2**

- *Arrivals:* Arrival hall and tunnel, Customs area / Airport Square (male and female washrooms, Wok Inn male and female washrooms), Baggage hall (Carousels 1–4, travellators, BIDS), Baggage delivery, Immigration area.
- *Departures:* Frontage/curbside, Pre-departure screening, Check-in hall (counters, escalators and staircase, Mezzanine, PWRM washroom, Aleeno café washrooms, Upper Concourse including Passion Air office washrooms), Information office / PA, Baggage make-up (airside, with washrooms), Departure lounge (Central screening, Gates 3, 4, 5, with Muttina cafeteria and washrooms), Transit, Lifts (Lift 2 and others).

**Terminal 3 (Departures)**

- *Frontage* and *Pre-departure screening*.
- *Check-in:* Aisles A, B, C; counters 1–46; Giant FIDS and clocks by Aisle B; CUSS kiosks; Out-of-gauge elevator (no. 2); Food-court elevator (no. 15).
- *Food court.*
- *Departure lounge:* Immigration area; Central screening; Gates C2, C3, C4, C5, C6, C7, C8A, C8B.
- *Baggage make-up.*
- *OCC (Level 5).*
- *Lifts and elevators:* 1, 2, 6, 8, 9, 11, 13, 14, 15.

**Washroom / facility codes seen**

| Area | Codes |
|---|---|
| Gate C2 | DR 479 (male), DR 480 (disabled), DR 485 (female) |
| Gate C4 | DR 467 (male), DR 469 (female), DR 470 (shower), DR 471 (shower), DR 472 (shower), DR 473 (disabled), DR 474 (baby care) |
| Gate C5 | DR 350 (baby care), DR 351 (disabled/PWRM), DR 353 (male), DR 355 (female), DR 358 (prayer room, female) |
| Gate C7 | DR 335 (female), DR 336 (disabled), DR 337 (male) |
| Gate C8 | DR 014 (female), DR 015 (PWRM), DR 016 (male) |
| Aisle A/B | DR 382 (disabled/PWRM), DR 384 (female), DR 386 (male) |
| Aisle C | DR 447 (male), DR 449 (female), DR 451 (disabled) |
| Food court | DR 510 (disabled), DR 511 (male), DR 512 (female), DR 518 (disabled), DR 519 (male), DR 520 (female) |

*The D3 duty report places DR 355 under "Gate C5" and the same room is described as "C7 DR 355" in the "Any other comments" row. The registry must assign it one location (E11, §11 step 1).*

---

## Appendix C: Worked example, source line to data model

**Source (D2 Airside, plus its remark):**

> "Female washroom 3rd door lock and handle broken" → remark "FMCC informed"

**Target records:**

| Record | Content |
|---|---|
| Location | `T2 > Departures > Baggage Make-up (Airside) > Female washroom > Cubicle 3` |
| Defect `TORP-000412` | category Washroom fittings; fault types Door lock broken, Door handle broken; quantity 2 components; severity Medium; safety flag no; status REPORTED; owner party FMCC; first-seen 2026-10-26 |
| Observation | defect `TORP-000412`; walkthrough 2026-10-26; author *officer*; still present = yes |
| Action | defect `TORP-000412`; type Notified; party FMCC; channel e-mail; actor *officer*; time 2026-10-26 09:42 |
| Next shift | The defect is listed under carry-forward as "Day 2 – still present?" The report line reads: *Female washroom 3rd cubicle door lock and handle broken – Day 2 – Notified FMCC 26-10 09:42 – Ticket: pending*. |

**Source (D3 Aisle C, count item):**

> "NO OF QUEUE CONTROLS AVAILABLE (NO DEPLOYED 100)" → "178 Available"

**Target:** Resource Count `queue control` at Aisle C; deployed 100 (from template baseline); available 178; **validation warning** "available exceeds deployed". The officer corrects the count or updates the baseline (with audit trail).

---

## Appendix D: Glossary

| Term | Meaning |
|---|---|
| A/C | Air conditioning |
| AVSEC | Aviation security |
| BHS | Baggage handling system |
| BIDS | Baggage information display system |
| CCTV | Closed-circuit television |
| CUSS | Common-use self-service (kiosks) |
| DR *nnn* | Facility/room code used for washrooms and similar rooms (for example DR 479) |
| Duty Officer | Terminal operations officer who patrols a zone and writes the report |
| Electromech | Electromechanical unit (lifts, doors and so on) |
| FIDS | Flight information display system |
| FMCC | Facility-management contractor/unit that receives defects (*inferred*) |
| GACL | The airport operator named in the reports (*inferred*: the company whose staff strength is recorded) |
| GA… number | Work-order/ticket reference, `GA` + six digits (*inferred*) |
| ICT | Information and communications technology unit |
| MoSCoW | Must / Should / Could / Won't prioritisation |
| OCC | Operations Control Centre (Level 5) |
| PA | Public address |
| PWA | Progressive web app |
| PWRM | Passengers with reduced mobility (*inferred*) |
| Port Health | Port health authority staff stationed at arrivals/pre-departure |
| SITA | Airline-industry IT provider (for printers/kiosks, as the reports show) |
| Walkthrough | A dated, area-by-area facility inspection |

---

*End of document.*
