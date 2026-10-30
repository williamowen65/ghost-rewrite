# GHOST Software Requirements Specification

Status: Draft 0.1 — owner review required  
Date: 2026-10-30  
Planning issues: [Architecture baseline #1](https://github.com/williamowen65/ghost-rewrite/issues/1), [SRS review #2](https://github.com/williamowen65/ghost-rewrite/issues/2)  
Related: [Milestones](MILESTONES.md), [Architecture baseline](ARCHITECTURE-BASELINE.md)

## 1. Purpose and scope

Rebuild the Gig Harbor Open Studio Tour member application with a maintainable development workflow, reliable business operations and a repeatable deployment process. Keep the current application operational during development; replace it through a planned migration and cutover.

Candidate first-release scope: membership contracts, artist information/images, committee assignments, artist applications, scholarships, payments, notifications, member administration, timeline/settings and required Google exports. Final release scope is pending owner review. New features beyond replacement functionality must be explicitly prioritized.

This draft extracts business intent from a first-pass static review of legacy source at commit 2918b8a16c09d0c51735f43c542e790d64d94c5c and the owner's rewrite/development goals. It is not an exhaustive specification, production verification or statement that proposed requirements already work.

## 2. Evidence and requirement status

All requirements below are **Draft**, with implementation and verification **Not assessed**. “Legacy” means source evidence exists, not that the live behavior is correct or approved for preservation. “Proposed” means a rewrite improvement requiring review. “Owner” means an outcome requested in discussion. Mixed origins distinguish existing intent from improved behavior.

Source keys below resolve within [legacy-code](../legacy-code/):

| Key | Source paths and significance |
| --- | --- |
| E1 | ghost-committee-web-app_members-site/README.md — stated application workflows |
| E2 | ghost-committee-web-app_members-site/dist/firebase.js; dist/CRUD.js — authentication, Firestore and storage access |
| E3 | ghost-committee-web-app_members-site/app/pages/my-contract/index.js — contract data, roles, settings and scholarship/payment UI |
| E4 | ghost-committee-web-app_members-site/app/components/3rdParty/paypal/index.js; app/api/paypal/order.js; app/api/paypal/capture.js — payment workflow |
| E5 | ghost-committee-web-app_members-site/app/pages/new-application/index.js; app/components/application/ — artist applications |
| E6 | ghost-committee-web-app_members-site/app/pages/scholarship-application/; app/components/scholarship-application/ — scholarship UI; detailed rules still to extract |
| E7 | ghost-committee-web-app_members-site/app/pages/members/index.js; timelinePriviledge.js — dashboard, timeline, export calls and role-based UI |
| E8 | ghost-committee-web-app_members-site/app/outbound-emails/ — notification intent and recipients |
| E9 | ghost-committee-web-app_members-site/app/exports/google-sheets/processContractsToSheets.js — contract export schema |
| E10 | latest-ghost-clasp-deployment/Google-Server/router.js; controllers/; appsscript.json — Google integration |
| E11 | ghost-committee-web-app_members-site/app/firebase.rules/firestore.txt — checked-in access rules |
| O1 | Owner discussion, 2026-10-30 — full rewrite, local development/seed data, SRS, roadmap, GitHub documentation |

## 3. Actors and permissions

Candidate actors: public applicant, authenticated artist/member, artist application reviewer, scholarship reviewer, committee/board member and administrator. These are business roles, not a finalized authentication role schema. A person may hold multiple roles.

The permission matrix must define who may read/edit each record, review applications, manage roles/settings, export personal information and perform overrides. Existing UI privilege names are evidence to review, not the final authority model.

## 4. Functional requirements

Acceptance checks are draft verification targets. Business values and edge cases named as open decisions must be resolved before their implementation issues are accepted.

| ID | Requirement | Origin | Draft acceptance check |
| --- | --- | --- | --- |
| GHOST-AUTH-001 | Support account sign-in, sign-out and password recovery for the chosen identity system. | Legacy E2 | A seeded user signs in/out; recovery is tested with an isolated delivery mechanism. |
| GHOST-AUTH-002 | Enforce record ownership and privileged actions at trusted API/data boundaries. | Proposed; E11 | Direct unauthorized reads/writes/review/export requests fail even when UI checks are bypassed. |
| GHOST-PRO-001 | Store artist contact, studio, medium, statement and public-link details needed by contracts and exports. | Legacy E3/E9 | Saved details persist and map consistently to approved export fields. |
| GHOST-YEAR-001 | Associate annual participation, contracts and financial records with a defined tour year, separately from lasting identity/profile as agreed. | Proposed; E7 | Two seeded years can be viewed without overwriting one another; rollover rules are documented. |
| GHOST-CON-001 | Allow artists to save and resume their own contract information. | Legacy E1/E3 | Save, reload and edit preserve agreed fields and validation errors identify invalid input. |
| GHOST-CON-002 | Apply agreed contract deadlines, required fields, signature/consent rules and authorized override behavior. | Legacy intent E1/E3; Proposed enforcement | Before/after deadline and permitted/denied override cases are tested; exact policy is approved. |
| GHOST-IMG-001 | Support the agreed contract/application image categories, replacement and removal. | Legacy E2/E3/E5 | Upload/reload/replace/remove works; invalid files and cross-user changes are rejected. Limits remain TBD. |
| GHOST-ROLE-001 | Display committees/positions and allow eligible members to request or claim assignments under approved capacity rules. | Legacy E3/E4 | Concurrent claims produce the approved result without silently displacing another member. |
| GHOST-ROLE-002 | Allow authorized staff to maintain committees, positions, privileges and assignments. | Legacy E3 | Changes persist; ordinary members cannot make administrative changes. |
| GHOST-APP-001 | Accept new artist applications with required artist details and images. | Legacy E1/E5 | A valid submission is persisted and acknowledged; invalid submissions show actionable errors. |
| GHOST-APP-002 | Allow designated reviewers to view and record application decisions. | Legacy intent E1/E8 | Authorized reviewers can decide an application; applicants cannot edit its review decision. Decision states TBD. |
| GHOST-SCH-001 | Allow eligible artists to submit and edit scholarship applications under agreed rules. | Legacy intent E1/E6 | Submission and permitted edits persist; eligibility and deadline cases follow approved policy. |
| GHOST-SCH-002 | Allow designated scholarship reviewers to record decisions and any approved fee consequences. | Legacy intent E3/E8 | Decision access is restricted and the fee calculation uses the approved award policy. |
| GHOST-PAY-001 | Calculate membership charges and approved discounts using trusted application rules. | Legacy intent E1/E4; Proposed enforcement | Changing a browser-supplied amount cannot change the authoritative charge. |
| GHOST-PAY-002 | Verify payment outcome before recording paid membership and associate it with artist, contract and year. | Legacy E4; Proposed enforcement | Successful, declined, pending and duplicate completion cases preserve correct state. |
| GHOST-PAY-003 | Persist receipt details and recover incomplete post-payment processing independently of the browser. | Legacy receipt E4; Proposed recovery | Closing the browser after payment does not lose completion; retries do not duplicate payment records. |
| GHOST-MAIL-001 | Send agreed application, scholarship and membership notifications to approved recipients using maintained templates. | Legacy E8 | Each approved event produces the expected captured test message without exposing unrelated recipient data. |
| GHOST-MAIL-002 | Track notification outcomes and support retry/deduplication. | Proposed | A simulated provider failure can be retried without creating duplicate completed notification jobs. |
| GHOST-ADM-001 | Provide authorized staff access to member/contract lists and agreed administrative edits. | Legacy E3/E7 | Staff can complete approved administration; unauthorized access is denied. |
| GHOST-ADM-002 | Maintain active year, timeline, contract content and agreed annual configuration. | Legacy E3/E7 | Authorized updates appear in the relevant UI; year changes do not erase historical records. |
| GHOST-EXP-001 | Export approved contract fields to a configured Google spreadsheet for a selected year. | Legacy E9; Proposed year selection | Export matches the approved schema and repeat runs do not duplicate member/year rows. |
| GHOST-EXP-002 | Generate contract documents/PDFs and organize Drive output only if confirmed in release scope. | Legacy intent E1/E7/E10 | An agreed sample contract produces readable, complete output in the permitted destination. Source workflow is incomplete/mismatched. |
| GHOST-EXP-003 | Track export progress/failure and allow safe retries of required exports. | Proposed | Partial failure is visible and recoverable without uncontrolled duplication or unrelated resource changes. |

## 5. Development, quality and operational requirements

| ID | Requirement | Origin | Draft acceptance check |
| --- | --- | --- | --- |
| GHOST-DEV-001 | Provide documented fresh-checkout setup and one-command startup after prerequisites/configuration. | Owner O1 | A clean environment reaches the local sign-in screen using the guide. |
| GHOST-DEV-002 | Provide deterministic synthetic seed data and repeatable reset. | Owner O1 | Reset restores known users/roles, contracts, applications, scholarships, images and multiple years. |
| GHOST-DEV-003 | Isolate local development from production data, messages and charges by default. | Owner O1; Proposed isolation | Local configuration uses local adapters/stores and rejects unintended production integration targets. |
| GHOST-DEV-004 | Provide local substitutes for Google, email and payment integrations plus isolated real integration tests. | Proposed | Ordinary workflows run without external credentials; failure/retry scenarios can be selected. |
| GHOST-DEV-005 | If Firebase is retained, configure selected Firebase emulators and seed/reset support; otherwise provide equivalent local services. | Owner O1 | Authentication/data/image workflows run locally with documented, verified tooling prerequisites. |
| GHOST-OPS-001 | Build/check changes in CI and deploy a minimal working application to isolated staging early. | Proposed | A representative change passes required checks and reaches staging without manual source edits. |
| GHOST-OPS-002 | Provide useful failure/job diagnostics while protecting secrets and personal information. | Proposed | Operators can trace a failed payment/export/email by safe identifiers without logging credentials or full private payloads. |
| GHOST-OPS-003 | Maintain separate environment configuration and documented credential/account ownership. | Proposed | Local/staging/production destinations and recovery owners are documented; secrets are outside tracked source. |
| GHOST-UX-001 | Support usable responsive forms, keyboard operation, labels and understandable validation/status messages. | Proposed | Agreed critical workflows pass keyboard and narrow-screen review. Accessibility target remains TBD. |
| GHOST-TEST-001 | Trace approved requirements to issues and meaningful automated/manual verification evidence. | Owner O1; Proposed process | Every release requirement has implementation and verification links; no inferred coverage from file counts. |
| GHOST-MIG-001 | Rehearse migration with record/media mapping, counts, financial reconciliation and permissions checks. | Proposed | A trial migration produces reviewed reconciliation results and an exceptions list. |
| GHOST-MIG-002 | Define final-write handling, backup, cutover, rollback and post-launch validation before production switch. | Proposed | A rehearsal verifies the agreed plan, including handling writes received during cutover/rollback. |

Performance targets, upload limits, retention, recovery objectives and precise accessibility criteria require agreement. No numeric target is invented in this draft.

## 6. Data and interface decisions

Candidate concepts: artist identity/profile, tour year/participation, contract/consent, image asset, committee/position/assignment, artist application/review, scholarship/review/award, charge/payment/receipt, notification job, export job and configuration.

A proposed data model must define stable identifiers, historical snapshots, ownership, audit needs and migration mappings. Separating profile and annual participation is a proposal, not an approved schema.

Define one authoritative store per business fact. Current evidence shows Firestore writes and explicit Sheets exports; bidirectional synchronization has not been established. Google spreadsheet edits must not implicitly become authoritative unless that workflow is explicitly required.

Google interfaces should expose named application operations such as ExportContractsForYear and SendMembershipReceipt. The backend-owned integration direction is provisional; see the architecture baseline for retained Apps Script and direct API options. Framework, language, persistence provider and hosting remain undecided.

## 7. Open decisions

| ID | Decision needed |
| --- | --- |
| D01 | Confirm live Apps Script source/version, active wrappers and any unpushed production edits. |
| D02 | Confirm Google account type, owner, administrator availability, sender mailbox and consent/credential approach. |
| D03 | Choose Apps Script or direct Google API integration after a non-production proof. |
| D04 | Choose frontend/backend, identity, persistence, hosting and local tooling based on requirements. |
| D05 | Confirm permission matrix, application/review states and public submission protections. |
| D06 | Confirm annual rollover, profile snapshots and historical-data visibility. |
| D07 | Confirm contract fields, signatures, deadlines, overrides and committee capacity/conflict rules. |
| D08 | Confirm fees, discounts, scholarship consequences, payment/refund/reconciliation policies. |
| D09 | Confirm required Sheets/PDF/image exports and whether staff edit spreadsheets as input. |
| D10 | Agree first-release priorities, new feature exclusions and measurable quality targets. |
| D11 | Inventory production records/media, retention, backup ownership and migration constraints. |

## 8. Traceability and change control

Keep requirement IDs stable. Mark each approved, deferred or rejected after owner review, retaining rationale for changes. Each implementation issue references requirement IDs and acceptance cases; PRs and verification results link back. Track implementation separately from approval. Draft 0.1 has no verified implementation coverage.

Milestone mapping is in MILESTONES.md. Detailed implementation issues will be created after review; current immediate work is #1, #2 and #3.
