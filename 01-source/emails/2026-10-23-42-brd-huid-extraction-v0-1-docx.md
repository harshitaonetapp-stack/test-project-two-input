---
source_item_id: 42
message_id: "doc:TESTTWO:01-source/documents/BRD - HUID Extraction-v0.1.docx"
thread_key: "doc:TESTTWO:01-source/documents/BRD - HUID Extraction-v0.1.docx"
project: "TESTTWO"
from: ""
date: "2026-10-23T06:00:51.639651+00:00"
subject: "BRD - HUID Extraction-v0.1.docx"
---

P. N. GADGIL JEWELLERS LIMITED
IT
Business Requirements Document
HUID Extraction & Validation at Goods Receipt (GRN)
Automated reading and controlled entry of the BIS Hallmark Unique Identification number into Acme ERP
Field | Detail
Document title | BRD - HUID Extraction & Validation at GRN
Prepared by | GM - IT, PNG Jewellers
Intended audience | Prospective IT implementation partners (RFP / scoping)
Version | 0.1 - Draft for circulation
Status | Draft - subject to confirmation of open items (Section 12)
Classification | Confidential - for shortlisted vendors under NDA
Contents
1.  Purpose & Audience	
This document sets out the business requirements for a solution that reads the BIS Hallmark Unique Identification (HUID) number etched on hallmarked gold jewellery from a magnified image, and enables the operator to validate and enter it into Acme ERP at the point of stock inward (GRN).
It is issued to prospective IT implementation partners to support proposal preparation. It states what the solution must achieve and the controls it must enforce; it does not prescribe a specific technology stack. Vendors are expected to propose their approach against these requirements. A set of non-binding reference approaches is provided in Appendix A for context only.
2.  Business Context & Background
PNG Jewellers is a listed omnichannel jewellery retailer operating a large multi-store network on the Acme ERP platform. Under BIS regulations, every hallmarked gold item carries a HUID: a six-character alphanumeric code, laser-etched on the piece alongside the BIS triangle mark, the purity/fineness mark (e.g. 22K916 / 916) and the maker's mark (PNGJ).
The HUID is frequently etched in print too small to be read reliably by the unaided eye, and sits on a reflective, often curved gold surface. A digital magnification device has been evaluated (demo completed) that magnifies the etched region and displays it on the operator's workstation monitor. The operator is required to capture the HUID during GRN and record it in Acme.
Acme does not currently hold HUID values for stock. HUID must therefore be captured at the point of inward. The organisation's own HUID records exist in the BIS system, but the BIS portal is currently not accessible; consequently, validation against a BIS reference list is not available in the first phase and is deferred (see Sections 3 and 12).
3.  Problem Statement
Manual reading and keying of the HUID at inward is slow and error-prone for reasons intrinsic to the medium, not the operator:
The code is micro-etched on reflective gold, so image quality varies significantly by piece geometry and lighting; the same piece can read differently between two captures.
Several character pairs are visually ambiguous in the etched font - notably I/1, O/Q/0, S/5, B/8 and Z/2.
The HUID has no check digit, so a single misread character produces a valid-looking but incorrect code with nothing to catch it.
Because Acme has no prior HUID, the captured value becomes master data at the point of creation, with no downstream reconciliation in the interim. A wrong HUID is therefore effectively permanent until corrected.
With no reference list available in Phase 1, the accuracy bar is higher, not lower: the solution cannot lean on a known-valid set to resolve ambiguous reads and must instead build independent checks into the capture-and-entry workflow.
4.  Objectives & Success Criteria
O1. Accurately extract the six-character HUID from the magnified image at GRN and present it to the operator.
O2. Provide independent validation so that misreads are surfaced and corrected before any value is posted to Acme - minimising silent errors in master data.
O3. Post the validated HUID against the correct item / GRN line in Acme.
O4. Deliver an architecture that is ready to add BIS reference-list matching and weight cross-check once BIS access is restored, without redesign.
O5. Flag all interim-captured HUIDs so they can be batch-reconciled against BIS data when it becomes available.
Success is measured by field-level accuracy on a labelled sample set, the absence of silent (unchecked) posts to Acme, completeness of the audit trail, and demonstrable readiness of the Phase 2 hooks.
5.  Scope
5.1  In scope - Phase 1 (interim, to be delivered now)
Ingestion of captured image frames from the magnification device at the operator workstation.
Automated isolation of the HUID token by stripping the known fixed marks (BIS triangle, purity mark, maker's mark).
Automated extraction of the candidate HUID with multi-frame and/or multi-engine consensus and per-character confidence.
Operator validation workflow enforcing double-blind entry (defined in Section 8).
Format validation gate (exactly six valid alphanumeric characters).
Posting to Acme with an 'unverified' flag and a batch identifier.
Exception handling queue and full audit logging.
5.2  In scope - Phase 2 (future; architecture to be ready in Phase 1)
Ingestion of PNG's BIS HUID reference list (HUID with weight, purity and article type).
Closed-set matching of the extracted HUID against the reference list, with weight cross-check.
Batch reconciliation of Phase 1 'unverified' records against BIS data, with an exception report.
5.3  Out of scope
Procurement of the magnification hardware (already evaluated separately).
Remediation of BIS portal access.
Any change to the hallmarking / assaying process itself.
Changes to Acme core beyond the interface required to record the HUID and its flags.
6.  Solution Overview
The conceptual flow for Phase 1 is a capture-extract-validate-post pipeline in which the machine reading and the operator reading are kept independent, and no value reaches Acme without passing an explicit control:
Stage | What happens
1 - Capture | The device auto-captures multiple frames of the etched region to the workstation.
2 - Isolate | The solution locates and strips the purity and maker marks, isolating the 6-character HUID token.
3 - Extract | The token is read; results are reconciled across frames (and/or two engines) with per-character confidence, and ambiguous characters are flagged rather than silently committed.
4 - Present | The magnified image and the machine candidate are shown, with low-confidence characters highlighted.
5 - Validate | Double-blind: the operator keys what they read; the system then reveals its candidate and forces a re-look on any mismatch. A format gate blocks anything that is not 6 valid characters.
6 - Post | The validated HUID is written to the correct Acme item/GRN line with unverified=true and a batch_id. Exceptions route to a supervised queue.
Phase 2 inserts a reference-match and weight cross-check between stages 3 and 5, and adds a reconciliation job over the Phase 1 records. The Phase 1 data model and token-isolation logic are designed so this is an addition, not a rebuild.
7.  Functional Requirements
ID | Requirement | Priority
FR-01 | The solution shall ingest image frames of the etched HUID region from the magnification device connected to the operator workstation. | Must
FR-02 | The solution shall automatically capture multiple frames per piece; frame count for consensus shall not depend on operator discretion. | Must
FR-03 | The solution shall detect and strip the known fixed marks (BIS triangle, purity mark e.g. 22K916/916, maker's mark e.g. PNGJ/PNGJ7) and isolate the six-character HUID token. | Must
FR-04 | The solution shall extract a candidate HUID from the isolated token. | Must
FR-05 | The solution shall reconcile readings across frames and/or two independent engines, voting per character position, and expose per-character confidence. | Must
FR-06 | The solution shall treat known confusable character classes (I/1, O/Q/0, S/5, B/8, Z/2) as low-confidence and declare uncertain characters rather than silently resolving them. | Must
FR-07 | The solution shall display the magnified image alongside the machine candidate, with low-confidence characters visibly highlighted. | Must
FR-08 | The solution shall NOT pre-fill the extracted value into the entry field. The operator shall independently key the HUID; the system shall then reveal the machine candidate and flag mismatches for re-examination (double-blind entry). | Must
FR-09 | The solution shall reject any value that is not exactly six valid alphanumeric characters before it can be posted (format gate). | Must
FR-10 | The solution shall route no-consensus, mismatch and format-failure cases to a supervised exception queue for adjudication. | Must
FR-11 | The solution shall post the validated HUID to the correct item / GRN line in Acme via an agreed interface. | Must
FR-12 | Every HUID posted in Phase 1 shall carry an 'unverified' flag and a batch identifier to enable later reconciliation. | Must
FR-13 | The solution shall maintain an audit trail per piece: image reference, machine candidate(s) and confidence, operator entry, final value, operator ID, timestamp, and exception path taken. | Must
FR-14 | The solution shall be able to ingest PNG's BIS HUID reference list (HUID, weight, purity, article type). [Phase 2] | Should
FR-15 | The solution shall match the extracted HUID against the reference set under ambiguity-class equivalence and snap to a unique valid HUID above a confidence threshold. [Phase 2] | Should
FR-16 | The solution shall cross-check the GRN scale weight against the BIS-recorded weight and flag mismatches. [Phase 2] | Should
FR-17 | The solution shall batch-reconcile Phase 1 'unverified' records against BIS data and produce an exception report. [Phase 2] | Should
8.  Mandatory Data-Integrity Controls
These controls are non-negotiable because they are the only thing standing between a misread character and corrupted master data. A proposal that omits or weakens any of them will not be accepted.
Control | Why it is mandatory
Double-blind entry | If the machine candidate is pre-filled, the operator anchors to it and rubber-stamps exactly the hard reads where errors occur. Independent keying, then reveal-and-compare, is what makes the human a genuine check.
Multi-frame / multi-engine consensus | One reader cannot check itself on ambiguous glyphs. Voting across frames or engines localises doubt to specific characters instead of committing a confident wrong answer.
Ambiguity-class handling | I/1, O/Q/0, S/5, B/8 and Z/2 are the dominant error mode on etched gold and must be surfaced, not smoothed over.
Format gate | A HUID is exactly six alphanumeric characters; anything else is rejected before it can post.
Unverified flag + batch reconciliation | With no reference list in Phase 1, an interim error is otherwise permanent. Flagging every record converts 'permanently wrong' into 'recoverable' once BIS data is available.
BIS-ready architecture | The BIS access issue is temporary. Building Phase 1 so the reference-match and weight-check layer slots in later avoids shipping a throwaway keyer that must be rebuilt.
9.  Non-Functional Requirements
ID | Requirement | Priority
NFR-01 | The vendor shall state the achievable field-level accuracy on a representative PNG sample set; acceptance is measured against a labelled test set (Section 13). | Must
NFR-02 | The solution shall process a piece within an agreed cycle time compatible with GRN throughput (target to be set once volume is confirmed - Section 12). | Must
NFR-03 | Where any cloud/AI service is used, the solution shall provide a defined behaviour for service unavailability (e.g. graceful degradation to the exception queue). | Must
NFR-04 | The solution shall support the confirmed deployment topology - a central inward point and/or distribution across the store network - and scale accordingly. | Must
NFR-05 | HUID image crops contain no personal data; nonetheless the solution shall comply with PNG information-security and data-governance standards, and shall not capture or store any PII. | Must
NFR-06 | All processing shall be auditable and reproducible from the stored artefacts (image reference, candidates, entries). | Must
NFR-07 | The operator interface shall be usable with minimal training and shall not slow inward beyond the agreed cycle time. | Should
NFR-08 | The extraction model shall be improvable over time using the labelled crops the workflow generates. | Should
10.  Integration Requirements
INT-1. Capture device → workstation: the solution shall receive image frames from the magnification device already evaluated by PNG.
INT-2. Solution → Acme ERP: the solution shall write the validated HUID, the 'unverified' flag and the batch identifier to the correct item / GRN line through an interface to be agreed with PNG (method and field availability to be confirmed - Section 12).
INT-3. BIS reference data → solution [Phase 2]: the solution shall ingest PNG's BIS HUID reference list for matching and reconciliation once available.
11.  Assumptions
The evaluated magnification device provides image clarity sufficient for extraction on the majority of pieces.
Acme exposes, or can be configured to expose, an interface to record the HUID and its flags against an item / GRN line.
HUIDs conform to the BIS six-character alphanumeric format.
PNG's BIS reference list will become available for Phase 2; timing is presently uncertain.
12.  Open Items - To Be Confirmed
The following are not yet settled and materially affect design, cost and timeline. Vendors should either price against the stated assumption or present options; PNG will confirm before contract.
Item | Why it matters
Deployment topology - central inward point vs distributed across the store network | Central allows one controlled rig and an on-premise model; distributed pushes toward cloud services and makes connectivity, uptime and per-site cost real constraints. Vendors should price the confirmed topology, or both.
Daily inward volume (pieces/day) | Drives cycle-time targets, sizing, and any per-call AI cost.
Acme integration method and HUID field availability | Determines the interface build for FR-11 / INT-2.
BIS reference-list export format and availability | Gates the whole of Phase 2 (FR-14 to FR-17).
Cloud vs on-premise extraction | Follows from topology and connectivity. Note: HUID crops carry no PII, so data-protection rules are not a blocker; the driver is cost, latency and offline resilience.
Confirmed permitted HUID character set | To be verified with the hallmarking centre - whether visually ambiguous characters are excluded changes the ambiguity-handling logic (FR-06).
13.  Acceptance Criteria
Field-level extraction accuracy on an agreed labelled test set meets or exceeds the target set with PNG at contract.
No silent posts: every HUID written to Acme has either passed double-blind agreement or been cleared through the exception queue.
Every Phase 1 record carries the 'unverified' flag and a batch identifier.
The audit trail is complete and reproducible for a sampled set of pieces.
Phase 2 hooks (reference ingestion, matching, weight cross-check, reconciliation) are demonstrably present and testable, even if BIS data is not yet live.
14.  What Vendors Must Submit
Proposed architecture for Phase 1, with explicit evidence of Phase 2 readiness.
Extraction approach and engine(s), with accuracy evidence on the PNG sample images provided during scoping.
Implementation detail for the mandatory controls in Section 8 - in particular how double-blind entry and consensus are enforced.
Deployment options (central and/or distributed) with the cost of each.
Acme integration approach and dependencies.
Reconciliation design for Phase 2.
Commercials, delivery timeline, and support / SLA terms.
A data-handling statement covering where images and any derived data are processed and stored.
Appendix A - Reference Approaches (Non-Binding)
The following are provided to convey intent only and do not constrain the vendor's proposal. Requirements are stated in Sections 7 to 10.
Isolating the six-character token by first removing the fixed purity and maker marks tends to improve reliability, as it removes ambiguity about which text is the HUID.
Combining two independent reads - for example an on-premise OCR engine and a vision-language model accessed via API under a locked output contract - allows per-character voting. Any LLM used should be integrated behind an API with a constrained prompt (fixed six-character uppercase output; uncertain characters declared explicitly), not exposed to operators as a free-form chat interface, which removes the output controls and invites confidently wrong readings.
Reading multiple frames of the same piece and voting per character exploits natural variation between captures to localise uncertainty.
Over time, a small extraction model fine-tuned on PNG's own labelled HUID crops can reduce cost and dependency on external services while improving accuracy.
Appendix B - Glossary
Term | Meaning
HUID | Hallmark Unique Identification - a six-character alphanumeric code laser-etched on each hallmarked gold item under BIS regulations.
BIS | Bureau of Indian Standards - the body governing hallmarking and the HUID scheme.
GRN | Goods Receipt Note - the stock-inward point at which the HUID is to be captured.
Acme | PNG Jewellers' ERP platform, the system of record for stock.
OCR | Optical Character Recognition - automated reading of text from images.
VLM | Vision-Language Model - an AI model that reads and reasons over images and text.
Double-blind entry | Operator keys the value independently before the machine candidate is revealed, so the two reads act as checks on each other.
Token isolation | Separating the six-character HUID from the surrounding fixed marks before extraction.
Consensus | Voting across multiple frames or engines to agree each character and flag disagreement.
