# MATACS AI Proposal Triage Design

**Candidate:** Nasir Hussain  
**Role:** Associate Software Developer  
**Reference:** MATACS-TH-2026-02  
**Time spent:** [Enter your actual time spent before submission.]

## Repository contents

- [Process flow diagram](diagram/process-flow.png)
- [Design write-up](WRITEUP.md)
- [AI extraction prompt](PROMPTS.md)
- [Expected proposal ranking](ranking.json)

## How I used AI tools

I used ChatGPT to help structure the design, check the scoring approach, draft the extraction prompt, and prepare the documentation. I reviewed the recommendations rather than accepting them blindly: an initial price-and-speed-focused interpretation could have favored Proposal B, but I corrected that approach because the bidder-held Ontario electrical licence is a mandatory pass/fail requirement, and a subcontractor to be confirmed later does not satisfy the stated criterion. I also treated the instruction embedded in Proposal C as untrusted document content, not as a command to the AI.

## Assumptions

- The supplied summaries are the only evidence available; no external licence, WSIB, insurance, or reference checks are performed.
- A mandatory criterion passes only when the required fact is explicitly stated for the bidder.
- A proposal with a missing fact needed to establish compliance or score eligibility is flagged for human clarification rather than silently treated as a pass.
- Scores use the explicit illustrative rubric documented in `WRITEUP.md`; an administrator would configure and version the rubric in the real product.
- The ranking is advisory. The authorized human reviewer makes the award decision.

## Submission checklist

1. Replace the time-spent placeholder with the actual time you spent.
2. Read the files and make sure you can explain the design in a follow-up call.
3. Create a **public** GitHub repository named `matacs-th-2026-02`.
4. Upload this folder's contents while preserving the exact names and layout.
5. Commit your work as you go, then submit only the repository URL through the Google Form in the assessment brief.
6. Keep the repository public and unchanged for the requested period.

No proposal text from the assessment brief is reproduced in this repository.
