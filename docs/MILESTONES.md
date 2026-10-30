# GHOST rewrite milestones

Status: Proposed roadmap — scope and architecture require review  
Date: 2026-10-30  
Related: [SRS](SRS.md), [architecture baseline](ARCHITECTURE-BASELINE.md), [issue #1](https://github.com/williamowen65/ghost-rewrite/issues/1)

## Delivery strategy

Full rewrite delivered as small demonstrable workflows while the legacy application continues serving members. Extract business intent, review it with the owner, then implement agreed behavior with traceability. Keep legacy-code as a reference; build a separately runnable replacement.

Deploy a minimal application to staging in M3. Demonstrate and verify every later slice there. Investigate Google authentication and migration early; production cutover remains last. Do not make simultaneous replacement of Firebase, Heroku, frontend and Apps Script an assumption.

These are documentation milestones, not native GitHub milestone objects. None is complete, scheduled or time-estimated. Final release scope must be reviewed before estimates.

## Milestones and completion criteria

| ID | Milestone | Work and evidence | Exit criteria | Dependencies |
| --- | --- | --- | --- | --- |
| M1 | Establish requirements | Extract source workflows; review roles, rules, requested changes, data ownership and release exclusions. | SRS requirements have reviewed status, acceptance criteria and an agreed first-release scope; unresolved decisions are recorded. | None |
| M2 | Choose architecture | Compare stack/persistence choices, Google options, account ownership, hosting and annual data model; prove highest-risk integration. | Architecture decision records explain choices/tradeoffs; isolated integration proof succeeds; canonical legacy Google source identified. | M1 sufficient to assess options |
| M3 | Build development foundation | Local runtime, repeatable seeds/reset, integration substitutes, CI and staging. | Fresh checkout runs by documented steps; local isolation is verified; minimal working app automatically deploys to staging. | M2 |
| M4 | First complete workflow | Identity, trusted permissions, basic profile and contract save/resume. | Seeded artist signs in, saves, reloads and edits own contract; unauthorized record access fails; slice demonstrated in staging. | M3 |
| M5 | Applications and reviews | Artist/scholarship submission, decision workflows and notification foundation. | Approved applicant/reviewer cases pass, role boundaries hold and failure/retry behavior is demonstrated. | M4; M1 review rules |
| M6 | Membership operations | Annual participation, contract deadlines/consent, committee assignments, fees/awards, payment and receipts. | Sandbox payment reliably completes membership independently of browser; duplicate/failure cases and assignment conflicts are verified. | M4; scholarship fee integration from M5 |
| M7 | Administration and exports | Timeline/settings, member management, approved Sheets/docs/PDF/image workflows and job operations. | Staff complete agreed tasks in staging; exports reconcile and partial failures can be retried safely. | M4; integration decision M2; required business data M5/M6 |
| M8 | Migrate and launch | Trial migrations, identity/media mapping, reconciliation, acceptance, cutover rehearsal and rollback. | Owner accepts release scope; migration results reconcile; final-write and rollback handling are rehearsed; production switch and smoke checks complete. | Required M1–M7 scope verified |

M5–M7 can be reordered where business dependencies permit. Avoid building every screen before completing a working end-to-end slice.

## Requirement mapping

| Milestone | Primary SRS groups |
| --- | --- |
| M1 | All groups; D01–D11 decisions |
| M2 | Data/interface decisions; AUTH-002; YEAR-001; OPS-003; DEV-005 |
| M3 | DEV-001–005; OPS-001–003; TEST-001 foundation |
| M4 | AUTH-001–002; PRO-001; CON-001; initial IMG-001; UX-001 |
| M5 | APP-001–002; SCH-001–002; MAIL-001–002 |
| M6 | YEAR-001; CON-002; ROLE-001–002; PAY-001–003; receipt notifications |
| M7 | ADM-001–002; EXP-001–003; remaining required image workflows |
| M8 | MIG-001–002; complete release traceability and operational acceptance |

All shortened IDs above have prefix GHOST-. Quality, authorization and isolation apply throughout.

## Initial issue register

- [#1 Architecture walkthrough and Google integration options](https://github.com/williamowen65/ghost-rewrite/issues/1): source baseline and architecture discussion, M1/M2.
- [#2 Review draft SRS and agree release scope](https://github.com/williamowen65/ghost-rewrite/issues/2): M1.
- [#3 Local development, seed data and staging](https://github.com/williamowen65/ghost-rewrite/issues/3): M3, dependent on M2.

Later implementation tasks remain candidates until scope review. Create focused issues for the architecture proof, permission matrix, payment completion, durable notifications/exports, document contract mismatches, annual model, deployment and migration. Include SRS IDs, acceptance cases, dependencies and verification expectations.

## Development environment deliverable

Provide synthetic artists with different permissions, contracts in different states, artist/scholarship review examples, committee capacity/conflicts, payment outcomes and at least two tour years. Seeds must reset predictably and contain no copied production personal information.

If Firebase stays, use the selected Firebase local emulators with explicitly configured local endpoints and verified CLI/runtime instructions. Otherwise supply equivalent local persistence/identity/storage. Apps Script needs its own substitute/test setup; do not treat it as part of Firebase emulation.

Default local Google/email/payment adapters capture intent and simulate outcomes. Exercise real Google and PayPal integrations against isolated test resources/sandbox after local cases work. Include failure/retry and duplicate-processing examples.

## Migration work starts before M8

During M1/M2, inventory live data, identity ownership, media links and Google deployments. Design migration mapping as the new schema is selected. Rehearse imports during working slices, document discrepancies and decide whether writes will pause or be synchronized at cutover. Do not introduce a second active writer without a deliberate consistency design.

## Progress policy

An approved requirement is not an implemented requirement. A completed milestone needs demonstrated behavior and linked verification evidence, not just merged files. Track first-release exclusions explicitly. Record new requests and changes in GitHub and update SRS/roadmap when their scope changes.
