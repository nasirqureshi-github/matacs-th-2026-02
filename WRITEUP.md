# AI Proposal Triage Design

## Design summary

The application is a configurable, evidence-based decision-support workflow. Admins create category criteria in the admin screens; the application stores a versioned snapshot of those criteria for every evaluation. Reviewers upload proposals, choose a category, inspect extracted evidence, and receive a ranked list. The authorized human makes the final award decision.

### Process

1. **Configure criteria (admin):** Admin creates a category, mandatory pass/fail checks, weighted criteria totaling 100%, scoring rubrics, thresholds, and evidence requirements. The system validates the configuration and publishes an immutable version.
2. **Upload and identify:** Reviewer uploads PDF files, selects the category, and confirms bidder/document identity. The system checks file type, size, malware, duplicate files, and whether text extraction is possible.
3. **Extract (AI):** A constrained AI prompt extracts criterion-relevant facts into a strict JSON schema. Every fact includes a short evidence quote and page/section locator where available. Unknown or ambiguous facts are returned as `unknown`, not guessed.
4. **Validate (code):** Schema validation checks types, required fields, source locations, and evidence references. The reviewer can correct extracted fields; corrections are recorded.
5. **Apply mandatory gates (rules):** Deterministic code evaluates each mandatory requirement. `pass` requires explicit evidence; `fail` means explicit contrary evidence or a stated requirement is not met; `unknown` means evidence is missing/ambiguous. A fail is `ineligible`; an unknown is `flagged` and held out of ranking until resolved.
6. **Score and rank (rules):** Only proposals that pass every mandatory gate and have sufficient evidence for ranking are scored using the category's versioned rubric. The system calculates weighted totals, sorts descending, and uses a stable tie-break rule.
7. **Review and decide (human):** Reviewer sees the rank, score breakdown, evidence, missing information, and flags. They can request clarification, correct extraction, or record an override with a reason. An authorized procurement decision-maker records the final award separately from the AI-assisted ranking.

## Scoring model used for this sample

The brief supplies weights but not detailed numeric conversion rules. To make the sample ranking reproducible, I define the following illustrative rubric. In production, admins configure and approve these rules per category; they are not hard-coded.

- **Relevant experience (30%):** `min(qualifying installs in last 5 years / 10, 1) × 100`. Only commercial installs of at least 100 kW count.
- **Price (25%):** If within the $260,000 ceiling, `(260,000 − quoted total) / 260,000 × 100`. Above ceiling fails the configured budget gate or is handled according to the admin's explicit policy. The quote must be an all-in comparable total; exclusions trigger clarification.
- **Timeline (15%):** For a confirmed schedule of 16 weeks or less, `(16 − weeks) / 16 × 100`. More than 16 weeks is non-compliant with the stated project constraint; an unspecified schedule is `unknown` and flagged.
- **Warranty (15%):** Average of panel warranty divided by 25 years and workmanship warranty divided by 12 years, each capped at 100%.
- **Team and approach (15%):** 25 points each for an explicitly named project lead, stated crew, safety plan, and monitoring plan. Evidence of a roof assessment is useful supporting information but is not a substitute for one of these four defined elements.

Weighted total = sum of each criterion score × its weight. Scores are rounded to two decimals only for display; sorting uses the unrounded total. These benchmarks are transparent sample assumptions, not claims that the brief mandated those exact formulas.

## Sample outcome

- **A — ranked first (64.62):** Passes both mandatory checks and has strong documented experience, a 12-week schedule, complete team/approach evidence, and a competitive in-budget price.
- **E — ranked second (58.50):** Passes mandatory checks; its 12-year workmanship warranty and complete approach score well. Its price and experience score are slightly behind A.
- **C — ranked third (51.66):** Passes mandatory checks and has strong experience and warranties, but its 14-week schedule and less complete team/approach evidence reduce its score. The instruction-like text in its appendix is ignored as untrusted input.
- **D — flagged:** Mandatory checks pass, but no completion duration is provided. Since the project must be completed within 16 weeks and timeline carries 15%, the system requests clarification and does not invent a score or rank.
- **B — ineligible:** The bidder's own Ontario electrical contractor licence is not stated; the proposal says a licensed subcontractor will be confirmed after award. This does not satisfy the explicit bidder-held mandatory requirement. Its low price and short schedule cannot compensate for a failed gate.

## 1. AI vs. rules

AI is used for reading unstructured proposal documents and extracting relevant facts with evidence. It is useful because proposals can describe the same information in different wording and layouts. Plain code handles file validation, schema checks, mandatory pass/fail gates, arithmetic, sorting, tie-breaks, and audit logging. Those tasks need predictable, testable results; an LLM should not calculate the official score or decide eligibility.

## 2. Consistency and explainability

The same document, criteria version, extraction model/prompt version, and validated extracted facts should produce the same official score because scoring and ranking are deterministic. AI extraction can vary, so the system stores the original extraction, model and prompt versions, evidence, reviewer corrections, and the final validated facts used for scoring. The reviewer sees criterion-level scores, weights, formulas, evidence quotes and page locations, plus reasons for flags or failed gates. Stable sorting uses total score, then higher mandatory/criterion completeness, then a fixed bidder-name order; tie-break rules are published in advance.

## 3. Human in the loop

A reviewer confirms bidder identity, checks low-confidence or missing evidence, and can correct extracted facts before scoring. A procurement approver reviews the ranked output and makes the award decision; the tool never awards automatically. The audit record includes uploader, timestamps, file hash, category and criteria version, prompt/model version, raw extraction, evidence locations, validation results, score calculation, ranking, reviewer edits, clarification responses, overrides with reasons, and final decision/decision-maker.

## 4. Changing criteria

Criteria are versioned and immutable once used in an evaluation. An admin edit creates a new version with validation (including weights totaling 100%) and an effective date. Existing evaluations retain their original version and results. If the organization wants to compare proposals under the new rules, an authorized user starts a new evaluation or explicitly reruns all proposals in that batch against the new version; the system preserves both results and clearly labels the version used.

## 5. Risks and safeguards

- **Wrong extraction or missing pages:** Validate PDF parsing, preserve page-level evidence, show confidence/unknown states, and require human confirmation for uncertain or mandatory facts.
- **Hallucinated facts:** Require verbatim evidence and source locations; reject unsupported values and never let the model fill gaps from general knowledge.
- **Prompt injection in a proposal:** Treat document text as untrusted data, instruct the model not to follow document instructions, use a fixed output schema, and keep scoring/eligibility outside the model. Proposal C's embedded ranking instruction is ignored.
- **Bias or inconsistent criteria:** Use admin-approved job-related criteria, version them, test outcomes, and provide the same rubric to all bidders. Avoid inferring protected or irrelevant personal characteristics.
- **Manipulated or malicious files:** Apply file-size/type limits, malware scanning, isolated parsing, access controls, and immutable file hashes.
- **Cost, latency, and privacy:** Extract only necessary content, limit page/token usage, cache by file hash and version, monitor per-document cost, restrict access, encrypt data, and define retention/deletion policies.
- **False confidence in the rank:** Display evidence and uncertainty, flag incomplete proposals, retain human authority, and make clear that a score is decision support rather than a procurement decision.

## Operational note

The system should be tested with synthetic edge cases: missing licence, expired or unstated clearance, price above budget, timeline over 16 weeks, conflicting values across pages, unreadable scans, duplicate uploads, ties, and malicious instructions embedded in documents.
