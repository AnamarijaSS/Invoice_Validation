# Hydra

Hydra is an AI-assisted tool for validating provider invoices in claims operations. It reads each invoice, links it to the right claim, runs a fixed set of business checks and lays out the result for a handler. A named person makes every decision.

This repository documents the current Hydra prototype. The prototype runs on mock data only and is not connected to any live system.

## Principles

- **A person decides.** Hydra proposes values, matches and check results. A named handler confirms, corrects or escalates every invoice, one at a time. There is no bulk approval and no automatic approval lane.
- **Rules stay rules.** Matching on a known case reference and all business checks are deterministic: the same invoice always gets the same answer.
- **AI only where it earns its place.** In the proof of concept, AI is used for one step: reading the invoice and scoring how sure it is of each field.
- **Every step is evidence.** File arrival, AI readings, checks, decisions and exports are written to an audit trail that nobody can edit.
- **No money moves.** Hydra does not pay, approve payment or write back into core systems. It prepares an import file that a person loads into the CRM.

## Scope of the proof of concept

| In scope | Out of scope (later releases) |
|---|---|
| Network (contracted provider) invoices on the happy path | Country-specific variations |
| Invoices with a known case reference | Weak or missing references (AI-ranked matching) |
| One controlled intake folder | Production intake channels (provider portal, clearing system, email) |
| AI field extraction with confidence scores | AI recommendations for the next step |
| Eight fixed business checks | Exception queues: escalation, quarantine, handwritten |
| Import file for the CRM, loaded by a person | Direct connection to the CRM |

For network invoices, payment has already been settled through the weekly clearing run before validation starts. Validation confirms that the bill was right and keeps the proof.

The logic is designed once for all markets: one duplicate key, one intake route and one rule set. Only CRM field validation and local values differ per market.

## How an invoice moves through Hydra

| Stage | What happens | AI in the PoC |
|---|---|---|
| 1. Collect | The invoice is picked up from the intake folder and gets one ID | No |
| 2. Extract | AI checks the document is an invoice, reads every required field and scores its confidence | **Yes** |
| 3. Match | The case reference printed on the invoice links it to the claim, service line and request | No (AI ranking in R2) |
| 4. Validate | Eight fixed checks, each with its result and reason | No |
| 5. Resolve | The handler ticks the mandatory reminders and confirms, corrects or escalates | No (AI proposal in R2) |
| Export | Confirmed invoices collect in a batch per market; a person imports the file | No |
| Audit trail | Every step above is recorded on one timeline | No |

### Lifecycle states

`RECEIVED` → `SCREENED` → `EXTRACTED` → `IN_REVIEW` → `CONFIRMED` → `IMPORT_READY` → `IMPORTED`

Side states: `ESCALATED`, `QUARANTINED`, `DUPLICATE`, `NOT_PROCESSABLE`.

### Business checks

| Rule | Check |
|---|---|
| RULE-001 | Required invoice and CRM fields are present and readable |
| RULE-002 | Exactly one case and service line found for the reference |
| RULE-003 | Duplicate check on supplier, invoice number, invoice date and gross amount |
| RULE-006 | Invoice and service dates are present and plausible |
| RULE-007 | Amount against the approved limit (scored as review only while the tolerance is open) |
| RULE-008 | Extra review above a high-value threshold |
| RULE-010 | Currency and VAT context; gross-only VAT check |
| RULE-018 | Handler approves every invoice; reason-coded routing |

### Flags

Hydra raises flags that a handler sees on the queue and on the invoice:

- **Blocking:** unknown case ID, possible duplicate, system unavailable, parse failure
- **Warning:** low-quality scan, low confidence field, agent fee lines, network discount lines, stale reference export
- **Information:** handwritten (routed to manual), non-EUR currency, invoice that appears AI-generated

## Screens

| Screen | Purpose | Release |
|---|---|---|
| Work Queue | Handler's own market queue, oldest and most urgent first | R1 |
| Invoice Review | Document viewer beside extracted fields, match, checks and decision bar | R1 |
| Export & Import | Batches per market and target system; each import step confirmed by a person | R1 |
| Audit Trail | Search by invoice, case, handler, date or market; evidence pack as CSV or PDF | R1 |
| CRM Field Mapping | Hydra field to CRM target, with sign-off per row by product, claims and CRM owners | R1 |
| Model & Quality | Field accuracy and confidence monitoring | R2 |
| Control Tower | Intake health, queues and service levels | R3 |
| Escalations | Items raised to the team lead and product owner | R3 |
| Quarantine | Items that failed processing; retry or move to manual | R3 |
| Configuration | Thresholds, rate sources and mandatory reminders per market | R4 |
| Benefits & Readiness | Delivery readiness and benefit tracking | All |

Screens outside the selected release are shown locked.

### Roles

Handler, Team Lead, Product Owner, AI Governance / Compliance / DPO, Executive Sponsor, Admin / AI Builder. Each role sees only what it may act on. Built-in help on every screen explains the steps, and `?` opens it.

## Releases

| Release | What it adds | Gate |
|---|---|---|
| R0 | Readiness: intake source, test setup, field list, rules, output format, audit design | G0 |
| R1 | One complete happy-path invoice from approved source to handler review | G1 |
| R2 | AI-ranked claim matching and next-step recommendations, with reasons | G2 |
| R3 | Exception operations: queues, retries, escalation | Held until R2 is accepted |

Each release passes its gate on measured evidence before the next one starts. Production runs on the group's governed cloud platform.

## PoC success criteria (gate G1)

| # | Criterion | Target |
|---|---|---|
| 1 | Field accuracy | 85 % of fields read right; values below the CRM confidence threshold go to a handler |
| 2 | Correct case link | 100 % linked right, or routed when no case exists |
| 3 | Rule results | 100 % equal to the expected-answer file |
| 4 | Full trace | Every decided invoice reconstructable from file to CRM import |
| 5 | Handling time | Below today's measured time per invoice |
| 6 | Duplicate key | 0 missed duplicates, 0 false matches |

Results are scored from Hydra's audit trail against expected-answer files that claims leads prepare before the run.

## Status

- Prototype: clickable, all roles and screens, mock data only.
- AI extraction field test: 80.9 % of fields read right on a 12-invoice defect-finding sample; target 85 %.
- Next: CRM field sign-off, then the PoC run on two sample batches and gate G1.

## Not in this repository

Presentation material, customer data, real invoices and the interactive prototype itself are kept outside this repository.
