# Test Case Summary

Analyze CXTM test case execution results and generate formatted summary for test case notes.

## What This Skill Does

Comprehensive test result analysis:
- Loads CXTM test case summary template format
- Analyzes test execution artifacts (main report, jobfile, pre/post checks)
- Extracts device information, test operations, and results
- Identifies key findings and system behavior
- Detects cosmetic warnings vs actual failures
- Generates summary in standardized CXTM format
- Saves to TEST_CASE_SUMMARY.md in test case directory

**Saves you from manual result interpretation!**

## CRITICAL: Data Verification Rules (MANDATORY)

**RULE 1: ONLY Report What You Can Actually Verify**
- Never claim "zero packet loss" without seeing actual Spirent/Ixia output
- Never claim "test passed" without seeing final verification results
- Never claim "no errors" without seeing complete system health checks
- If verification output is missing, explicitly state it's missing

**RULE 2: Missing Data = Incomplete Test Status**
- If "Verify Spirent Zero Packet Loss" output is missing → Test status is "incomplete/unverifiable"
- If Robot Framework logs (log.html, output.xml) are missing → Cannot confirm pass/fail
- If post-test verification is truncated → Flag as incomplete

**RULE 3: Be Conservative and Factual**
- State only what the logs actually show
- Use phrases like "verification output not captured" instead of assuming results
- Mark test status as "incomplete" or "requires verification" if data is missing
- Never infer or assume test outcomes

**RULE 4: Immediate Red Flags**
- Missing Spirent verification output → STOP, do not claim zero packet loss
- Report ends abruptly → STOP, flag as incomplete
- No Robot Framework summary → STOP, cannot determine pass/fail
- Verification keyword called but no output → STOP, data missing

**RULE 5: What to Do When Data is Missing**
```
WRONG:
"This test case is passed."
"Zero traffic loss: No packet loss"

CORRECT:
"This test case status is incomplete - packet loss verification result not captured in available logs."
"Traffic validation incomplete: Post-test traffic ran but Spirent verification output not captured in report."
"IMPORTANT: Review CXTA execution logs or Spirent reports to confirm results before marking test as PASSED."
```

**Example of Missing Data Handling:**
```
Key Findings:
3. Traffic validation incomplete: Post-test traffic ran for 300s but Spirent verification output not captured in report

Additional Notes:
- "Verify Spirent Zero Packet Loss" keyword output not present in report
- No Robot Framework log files available for review
- Actual packet loss result requires verification from CXTA execution logs

Conclusion:
IMPORTANT: Packet loss verification result not captured in available logs. Review CXTA execution logs or Spirent traffic reports to confirm zero packet loss before marking test as PASSED.
```

**Lesson Learned (Feb 6, 2026):**
- An O4.03 test summary incorrectly claimed "zero packet loss" and "test passed" without verifying the actual Spirent output
- The "Verify Spirent Zero Packet Loss" keyword output was completely missing from the report
- This error was caught by the user, highlighting the critical importance of data-driven reporting
- **Never put stuff in report if you don't know - all result summaries MUST be based on real analysis of data**

## Usage

```
/test-case-summary [test-case-directory]
```

The user will provide the test case directory path after test execution completes.

**Example pattern**: User says "the test results are in [directory], please create test case summary"

**For format and structure reference**: See `.claude/TEST_CASE_SUMMARY_TEMPLATE.md` which contains:
- Complete format specification
- Working example (N4.08-1828897 Core interface shut/no-shut test)
- All sections explained with real data

## When To Use

**After test execution completes:**
- When test results downloaded from CXTA
- Before uploading results to CXTM
- To document test findings
- To prepare notes for test case review

## What Happens

**Analysis Process:**

1. **Load Template Format**
   - Read `.claude/TEST_CASE_SUMMARY_TEMPLATE.md`
   - Understand required structure and formatting rules

2. **Read Test Artifacts**
   - Main report (test execution log)
   - Robot Framework jobfile (test case definition)
   - Pre-check report (baseline before test)
   - Post-check report (baseline after test)
   - log.html / report.html (Robot Framework results)
   - Device logs (if present)

3. **Extract Information**
   - Device names and IP addresses
   - Test iterations and operations performed
   - Timing and convergence periods
   - Verification steps executed
   - System health checks performed

4. **Analyze Results (VERIFY EACH CLAIM)**
   - **MANDATORY CHECK**: Search for actual "Verify Spirent Zero Packet Loss" output
   - **MANDATORY CHECK**: Confirm Robot Framework test completion (look for final status)
   - **MANDATORY CHECK**: If verification missing, mark as "incomplete/unverifiable"
   - Overall test status (PASSED/FAILED/INCOMPLETE) - only if verified
   - Key findings (5 points: resilience, stability, traffic, health, network-wide)
   - Identify cosmetic warnings (authentication timeouts, expected messages)
   - Determine system behavior and stability

   **Before Writing Summary - Verification Checklist:**
   - [ ] Found actual Spirent verification output with packet counts?
   - [ ] Found Robot Framework final test status?
   - [ ] Report appears complete (not truncated)?
   - [ ] All verification keywords show output?
   - If ANY checkbox is NO → Mark test status as incomplete and state what's missing

5. **Generate Summary**
   - Format in CXTM standard structure
   - NO emojis (CXTM doesn't support them)
   - Devices Tested section
   - Test Operations section
   - Key Findings (exactly 5 points)
   - Cosmetic Warnings (if any)
   - Conclusion section

6. **Save and Display Output**
   - Write to `TEST_CASE_SUMMARY.md` in test case directory
   - Confirm file created successfully
   - **MANDATORY: Print the full summary as plain text on screen** so the user can review and copy it immediately without opening the file
   - This is NOT optional - the user expects to see the complete summary printed in the conversation output every time
   - Do NOT just say "file created" - you MUST output the entire summary text

## CRITICAL: Large File Handling (MANDATORY)

**NEVER use Read tool on large HTML files - causes terminal freeze!**

**File Size Check Protocol:**

```bash
# ALWAYS check file size first
ls -lh [test-directory]/*.html

# If log.html > 5MB, DO NOT use Read tool
# Extract statistics using bash instead:
grep -E '"pass":[0-9]+.*"fail":[0-9]+' log.html | head -5
```

**Known Issue (Feb 6, 2026):**
- Reading 11MB log.html file froze Claude Code terminal
- Ctrl+C cannot interrupt - user must close terminal
- Robot Framework log.html contains massive JavaScript data
- Read tool output buffer overflow cannot be stopped

**Safe Extraction Methods:**

1. **Test Statistics (from log.html):**
   ```bash
   grep -E '"pass":[0-9]+.*"fail":[0-9]+' log.html | head -5
   # Output: window.output["stats"] = [[{"pass":5,"fail":0,...}]]
   ```

2. **Test Duration:**
   ```bash
   grep -o '"elapsed":"[^"]*"' log.html | head -1
   ```

3. **Test Names:**
   ```bash
   grep -o '"name":"[^"]*"' log.html | head -10
   ```

4. **Spirent Verification (if in log.html):**
   ```bash
   grep -A 10 -B 5 "Verify Spirent Zero Packet Loss" log.html | head -30
   ```

**Decision Tree:**

```
Check log.html size
  ├─ < 5MB → Safe to use Read tool
  └─ > 5MB → MANDATORY: Use bash grep extraction
              ├─ Extract statistics with grep
              ├─ Extract test status with grep
              └─ NEVER attempt to Read full file
```

**Alternative Sources:**
- Always prefer main report TXT file over log.html
- Use output.xml if available (smaller, structured)
- Use report.html if available (smaller summary)
- Only extract from log.html when absolutely necessary

## Output Format

**Standard CXTM Format (NO emojis):**

```
Devices Tested

  - [DEVICE-NAME] ([IP ADDRESS])
  - [N] interfaces/iterations tested: [DETAILS]

  Test Operations

  -  [Number] iterations of [operation description]
  -  Each iteration: [step-by-step process]
  -  [Convergence/wait periods]
  -  [Verification steps]
  -  [Baseline checks]


This test case is [passed/failed].

  Key Findings:
  1. [Primary validation result]
  2. [Stability/convergence result]
  3. [Traffic impact result]
  4. [System health result]
  5. [Network-wide impact result]

  Cosmetic Warnings (Expected):
  - [Warning description with explanation]
  - [Root cause and why it's acceptable]

  ---
  Conclusion

  [Summary sentence describing system behavior]:
  -  [Key recovery/stability point]
  -  [Routing/protocol convergence]
  -  [Traffic impact statement]
  -  [System-wide stability statement]
  -  [Production readiness statement]

  [Validation statement] - [System capability demonstrated].
```

## Analysis Guidelines

**CRITICAL - What to Look For:**

1. **Test Status Determination (VERIFY BEFORE CLAIMING)**
   - **MANDATORY**: Check Robot Framework report: total tests, passed, failed
   - **MANDATORY**: Verify packet loss results - find actual "Verify Spirent Zero Packet Loss" output
   - **MANDATORY**: If verification output is missing, mark status as "incomplete/unverifiable"
   - Look for core dumps, tracebacks, critical errors
   - Check BGP convergence and routing stability
   - Verify baseline comparison (pre vs post)

   **Red Flags for Incomplete Test:**
   - Report ends during baseline verification (suggests truncation)
   - "Verify Spirent Zero Packet Loss" called but output not present
   - No Robot Framework log files (log.html, output.xml, report.html)
   - Missing final test summary or result statement

2. **Devices Tested Section**
   - Extract device names from jobfile or main report
   - Get IP addresses from lab.yaml or test output
   - Count interfaces/iterations from test operations
   - Include network scale if relevant (route counts, VRF counts)

3. **Test Operations Section**
   - Identify iteration count from Variables.yaml or jobfile
   - List operations in chronological order
   - Include timing (convergence waits, verification periods)
   - Mention traffic validation steps
   - Include baseline verification on multiple devices

4. **Key Findings (Exactly 5 Points)**
   - Point 1: Primary test objective validation (what was tested)
   - Point 2: Protocol/routing stability (BGP, ISIS, OSPF, etc.)
   - Point 3: Traffic impact (zero packet loss or loss amount)
   - Point 4: System health (cores, tracebacks, errors, responsiveness)
   - Point 5: Network-wide stability (baseline devices, interface errors)

5. **Cosmetic Warnings**
   - RADIUS/TACACS authentication timeouts (expected)
   - 100G QSFP28 negotiation delays (normal)
   - Minor OutDiscards during interface flapping (expected)
   - Always explain why these are acceptable

6. **Conclusion Section**
   - Start with summary of system behavior
   - 4-5 bullet points covering:
     * Device/interface recovery
     * Routing convergence
     * Traffic impact
     * Hardware resource status
     * Production readiness
   - End with validation statement

## What NOT to Include

**CRITICAL - Avoid These:**
- NO emojis (checkmarks, warning symbols, etc.)
- NO excessive technical details (save for full report)
- NO command outputs in summary
- NO screenshots or diagrams
- NO verbose explanations
- NO marketing language or superlatives
- NO time estimates

## Common Test Scenarios

**Interface Shut/No-Shut Tests:**
- Count total operations (iterations × devices)
- Note convergence timing (BGP/BFD wait periods)
- Check for interface negotiation warnings (100G QSFP28)

**Node SID Add/Remove Tests:**
- Count SR-MPLS label operation iterations
- Verify BGP-LU convergence
- Check route scale during test
- Validate hardware forwarding table stability

**Configuration Add/Remove Tests:**
- Count configuration change cycles
- Verify protocol reconvergence
- Check for configuration errors
- Validate rollback if applicable

**Traffic Loss Tests (CRITICAL - VERIFY DATA):**
- **MANDATORY**: Find actual "Verify Spirent Zero Packet Loss" output in report
- **MANDATORY**: Look for packet counts (Tx frames, Rx frames, Loss)
- **NEVER claim "zero packet loss" without seeing actual numbers**
- If Spirent output missing → State "Traffic validation incomplete"
- Only state "Zero packet loss" if you see actual verification output showing 0 loss
- Include actual packet counts if available (e.g., "Tx: 12,345,678 Rx: 12,345,678 Loss: 0")
- Include traffic duration and validation period

## Integration with /cxtm-ready

After running `/cxtm-ready`, you already know:
- Device IPs and credentials (from lab.yaml)
- CUSTOMER_NAME standards (from CXTA_BEST_PRACTICES.md)
- Network design context (from HBN_Network_Config_Template.md)
- Test case summary format (from TEST_CASE_SUMMARY_TEMPLATE.md)

This skill focuses on **analysis and formatting** only.

## Example Usage

```
User: /test-case-summary [test-case-directory]

Claude:
1. Loads template from .claude/TEST_CASE_SUMMARY_TEMPLATE.md
2. Reads test artifacts in provided directory:
   - Main report file (test execution log)
   - jobfile.robot (test case definition)
   - Precheck_Report.txt (baseline before)
   - Postcheck_Report.txt (baseline after)
   - log.html / report.html (Robot Framework results)
3. Analyzes:
   - Devices tested and IP addresses
   - Test operations and iterations performed
   - Test results (passed/failed, packet loss)
   - System health (cores, tracebacks, errors)
   - Network-wide stability
4. Generates formatted summary (no emojis)
5. Saves to TEST_CASE_SUMMARY.md in test case directory
6. **Prints the FULL summary as plain text on screen** (MANDATORY - never skip this)

Result: Formatted summary ready for CXTM upload AND visible on screen for immediate review
```

## Benefits

**Accuracy**: Comprehensive analysis of all test artifacts
**Consistency**: Always follows CXTM standard format
**Efficiency**: Automated analysis vs manual interpretation
**Completeness**: Never miss key findings or warnings
**Quality**: Professional, factual, technical documentation

---

## Related Skills

- `/cxtm-ready` - Load CXTM context first (run once per session)
- `/test-case-summary` - Analyze test results (run per test case)

---

**Use this after EVERY test execution to create professional test summaries!**
