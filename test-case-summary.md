# Test Case Summary

Analyze CXTM test execution results and generate formatted summary.

## Usage
```
/test-case-summary [test-case-directory]
```

## No AI Footprint (MANDATORY)
NEVER include "Claude", "AI", "generated", "LLM", or any attribution. Must appear as standard engineering test report.

## Data Verification Rules (MANDATORY)

1. **ONLY report what you can verify** - never claim "zero packet loss" or "test passed" without actual output
2. **Missing data = incomplete** - if Spirent verification output missing, status is "incomplete/unverifiable"
3. **Be conservative** - use "verification output not captured" instead of assuming results
4. **Red flags**: Missing Spirent output, truncated report, no RF summary, verification keyword with no output -> STOP, flag as incomplete

## Large File Handling
- NEVER Read log.html > 5MB (freezes terminal)
- Use `grep -E '"pass":[0-9]+.*"fail":[0-9]+' log.html` for stats
- Prefer main report TXT over log.html

## Process

1. Load template from `.claude/TEST_CASE_SUMMARY_TEMPLATE.md`
2. Read: main report, jobfile.robot, pre/post checks, log.html (if small)
3. **Verification checklist** before writing:
   - Found actual Spirent verification output with packet counts?
   - Found Robot Framework final test status?
   - Report complete (not truncated)?
   - If ANY is NO -> mark status incomplete
4. Generate summary (NO emojis), save to `TEST_CASE_SUMMARY.md`
5. **MANDATORY: Print full summary on screen** (never just "file created")

## Output Format

```
Devices Tested
  - [DEVICE] ([IP])
  - [N] interfaces tested: [DETAILS]

Test Operations
  - [N] iterations of [operation]
  - Each iteration: [steps]

This test case is [passed/failed/incomplete].

Key Findings:
1. [Primary validation result]
2. [Stability/convergence]
3. [Traffic impact - ONLY if verified]
4. [System health]
5. [Network-wide stability]

Cosmetic Warnings (Expected):
- [Warning with explanation]

Conclusion
[Summary]: recovery, routing, traffic, health, readiness.
```

## Key Findings Structure
- Point 1: Primary test objective
- Point 2: Protocol/routing stability
- Point 3: Traffic impact (zero loss ONLY if verified with actual numbers)
- Point 4: System health (cores, tracebacks, errors)
- Point 5: Network-wide stability (baseline devices, interface errors)
