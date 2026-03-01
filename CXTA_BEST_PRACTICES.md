# CXTA Best Practices & Jobfile Development Checklist

## ⚠️ CRITICAL: SSH Session Management

### Use Netmiko for Network Devices (Maintain Persistent Sessions)

**RULE**: For Cisco network devices, use Netmiko and maintain persistent SSH sessions across multiple commands.

```python
from netmiko import ConnectHandler

# ✅ GOOD - Persistent session for multiple commands
connection = ConnectHandler(
    device_type='cisco_nxos',
    host='YOUR_DEVICE_IP',
    username='YOUR_USERNAME',
    password='YOUR_PASSWORD',
    timeout=30
)

# Execute all commands on same session
output1 = connection.send_command("show version")
output2 = connection.send_command("show ip bgp summary")
output3 = connection.send_command("show mpls forwarding")

connection.disconnect()

# ❌ BAD - New session per command (inefficient, increases TACACS load)
for command in commands:
    conn = ConnectHandler(...)  # Don't reconnect for each command
    output = conn.send_command(command)
    conn.disconnect()
```

**Benefits of persistent sessions**:
- Faster execution (no repeated authentication)
- Reduced TACACS server load
- More reliable (avoids SSH rate limiting)
- Cleaner execution logs

### Use Paramiko for Linux Servers (NOT Netmiko)

```python
import paramiko

# ✅ GOOD - Paramiko for Linux servers
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('YOUR_SERVER_IP', username='YOUR_USERNAME', password='YOUR_PASSWORD')

stdin, stdout, stderr = ssh.exec_command('ls -la')
stdin, stdout, stderr = ssh.exec_command('df -h')

ssh.close()

# ❌ BAD - Don't use Netmiko for servers
```

---

## ⚠️ CRITICAL: Avoid Unix Pipe Commands on Network Devices

**NETWORK OS LIMITATION:** Unix pipe commands (tail, head, grep, awk, sed) do NOT work on Cisco network devices.

### ❌ NEVER Use Unix Commands:
```robot
# BAD - Unix commands not supported on NX-OS
run "show logging logfile | tail 50"
run "show interface | grep -i down"
run "show version | awk '{print $1}'"
```

### ✅ ALWAYS Use NX-OS Native Filters:
```robot
# GOOD - NX-OS native filtering
run "show logging logfile | last 50"
run "show logging logfile | include CRITICAL"
run "show interface | include down"
run "show version | exclude uptime"

# BETTER - Use existing CUSTOMER_NAME keywords
Check Logging NX-OS  ${DUT}
```

### NX-OS Native Pipe Commands:
- `| include <pattern>` - Show lines matching pattern
- `| exclude <pattern>` - Hide lines matching pattern
- `| begin <pattern>` - Show from first match onward
- `| last <lines>` - Show last N lines (NOT tail!)
- `| section <pattern>` - Show section containing pattern

**Why This Happens:**
Network device CLI is NOT a Unix shell. Cisco devices have their own filtering commands built into the CLI parser.

---

## ⚠️ CRITICAL: Avoid Unicode Characters (CXTA 25.15 Bug)

**CXTA Platform Bug:** Unicode characters in report strings trigger a platform bug in CXTA 25.15 that causes Digital Twin post-processing to fail with Python tuple syntax errors in bash wrapper scripts.

### ❌ NEVER Use Unicode Characters:
```robot
# BAD - Triggers CXTA bug
pass and add comment to report  ═══════════════════════════
pass and add comment to report  • Bullet point
pass and add comment to report  TEST RESULT: PASS ✓
```

### ✅ ALWAYS Use ASCII-Only Characters:
```robot
# GOOD - Safe ASCII characters
pass and add comment to report  ===========================================================
pass and add comment to report  * Bullet point
pass and add comment to report  TEST RESULT: PASS
pass and add comment to report  -----------------------------------------------------------
```

### Safe Formatting Characters:
- **Lines/Boxes**: Use `=`, `-`, `*`, `#`, `_`
- **Bullets**: Use `*`, `-`, `>`
- **Checkmarks**: Use `PASS`, `OK`, `SUCCESS` (not checkmark symbols)
- **Newlines**: Do NOT use `${\\n}` or `${\n}` - CXTA handles newlines internally (see below)

**Why This Happens:**
CXTA's post-processing wrapper incorrectly generates Python tuple syntax `('...',)` when handling Unicode characters, causing bash syntax errors: `open('('/tmp/testbed.yaml',)')` instead of `open('/tmp/testbed.yaml')`.

**Reference:** See `815235/CXTA_BUG_REPORT.md` for full technical details.

---

## ⚠️ CRITICAL: Avoid Colons in Plain Text Report Statements

**Robot Framework Parser Issue:** Colons (`:`) in plain text within `pass and add comment to report` statements can cause the parser to split the text into multiple arguments, resulting in "expected 1 argument, got 2" errors.

### ❌ NEVER Use Colons in Plain Text Labels:
```robot
# BAD - Parser splits at colon
pass and add comment to report  * Device Platform: N9K-C9364C-GX
pass and add comment to report  * Flow cache accessible: Direct access
pass and add comment to report  * Key finding: Some detail here
```

### ✅ ALWAYS Use Hyphens or Avoid Colons:
```robot
# GOOD - Use hyphen instead of colon
pass and add comment to report  * Device Platform - N9K-C9364C-GX
pass and add comment to report  * Flow cache accessible via direct access
pass and add comment to report  * Key finding - Some detail here
```

### ✅ Colons ARE Safe in These Contexts:
```robot
# SAFE - Colon in variable expansion (evaluated before parsing)
pass and add comment to report  ${DUT}: Configuration verified
pass and add comment to report  * Flow exporter: ${COLLECTOR_IP}:${COLLECTOR_PORT}
pass and add comment to report  * Export statistics: Incrementing normally

# SAFE - Colon at end of section headers
pass and add comment to report  KEY FINDINGS:
pass and add comment to report  SUMMARY OF RESULTS:
```

### Rule of Thumb:
- **Avoid**: `Plain text: More text` (splits into 2 arguments)
- **Safe**: `${Variable}: Text` (variable expands first)
- **Safe**: `Section header:` (at end of line)
- **Best**: Don't add unique descriptive lines - stick to working examples

**LESSON LEARNED:** Even with hyphens instead of colons, complex descriptive text can cause parser issues. The safest approach is to **follow working examples exactly** (E6.03, D6.03, F6.03) and avoid adding unique descriptive lines that aren't present in proven working jobfiles.

**Discovered:** January 12, 2026 - Test case G6.03 (CXTM 1828253)

---

## CRITICAL: Newline Variable Syntax in Report Statements

**`${\n}` (single backslash) WORKS. `${\\n}` (double backslash) FAILS.**

Using `${\\n}` causes: `Variable '${\}' was not closed properly.`

### NEVER Use Double Backslash:
```robot
# BAD - Double backslash fails
pass and add comment to report  ${DUT} - Check passed ${\\n}
fail and add comment to report  ${DUT} - Check failed!!! ${\\n}
```

### Single Backslash or No Newline - Both Work:
```robot
# GOOD - Single backslash (used in I4.07, confirmed working)
pass and add comment to report  ${DUT} - Check passed ${\n}

# ALSO GOOD - No newline at all (CXTA adds newlines internally)
pass and add comment to report  ${DUT} - Check passed
```

**Reference:** I4.07 (CXTM 1828313) uses `${\n}` throughout and passes. C2.13 used `${\\n}` and failed.

**Discovered:** February 12, 2026 - Test case C2.13 (CXTM 1893607)

---

## CRITICAL: `add report title` Embedded Argument Restrictions

**Robot Framework Embedded Argument Bug:** The CXTA keyword `add report title "${title}"` uses embedded argument matching. Strings that start with numbers followed by a period (`1.`, `2.`) or dashes (`-`) cause the embedded argument parser to fail, splitting the keyword name incorrectly.

### NEVER Start `add report title` Strings With These Patterns:
```robot
# BAD - Embedded argument parser fails on leading numbers/dashes
add report title "1. First item description"
add report title "2. Second item description"
add report title "- Bullet point item"
add report title "* Another bullet"
```

Error produced: `No keyword with name 'add report title "' found.`

### Use `pass and add comment to report` for Detail Lines:
```robot
# GOOD - Use add report title only for simple section headers
add report title "CoA Tests Performed"
pass and add comment to report  CoA Reauthenticate - Verify DATA client re-authenticates
pass and add comment to report  CoA Session Terminate - Verify DATA client removed

add report title "Key Validation"
pass and add comment to report  CoA operates at SESSION level on MDA ports
```

### Rule:
- **`add report title`** - Use ONLY for simple section headers (no leading numbers, dashes, or special patterns)
- **`pass and add comment to report`** - Use for all detail lines, bullet points, and numbered items

**Discovered:** February 12, 2026 - Test case C2.13 (CXTM 1893607)

---

## 📋 MANDATORY Requirements for All Jobfiles

### 0. File Naming Convention (CRITICAL)
```
# ✅ CORRECT - Standard jobfile name
jobfile.robot

# ❌ WRONG - Do not use other names
Robot.txt
test.robot
automation.robot
```

**RULE**: All Robot Framework automation files MUST be named `jobfile.robot`

### 1. File Paths (CRITICAL - Must Use Absolute Paths)
```robot
# ✅ CORRECT - Absolute Linux paths for CXTA server
Resource    /home/cisco/cxta/GS/N9K_HBN_Certification/Common/Customer_Keywords.robot
Variables   /home/cisco/cxta/GS/N9K_HBN_Certification/Common/Variables.yaml
load testbed "/home/cisco/cxta/GS/N9K_HBN_Certification/Common/Topology.yaml"

# ❌ WRONG - Never use relative paths
Resource    ../Common/Customer_Keywords.robot
Variables   ./Variables.yaml
```

### 2. Connection Management (CRITICAL - Must Cleanup)
```robot
# ✅ MANDATORY - Always include Suite Setup and Teardown
Suite Setup     Run Keywords
...             load testbed "/home/cisco/cxta/GS/N9K_HBN_Certification/Common/Topology.yaml"

Suite Teardown  Disconnect from all devices  # ✅ ALWAYS disconnect!
```

**Why:** Prevents stale SSH sessions and ensures proper resource cleanup even if test fails.

---

## 🎯 Test Case Structure & Reporting Rules

### RULE: Every Test Case Must Follow This Pattern

#### Step 1: Initialize Test Variables and Create Report
```robot
Initialize Test Variables
    [Documentation]  Initialize variables and reporting

    ${DUT}=  Set Variable  DEVICE-NAME
    ${main_report}=  Set Variable  TestCase_ID_Description_Report.txt
    ${output_file}=  Set Variable  ${DUT}_Output.txt

    Set Suite Variable  ${DUT}
    Set Suite Variable  ${main_report}
    Set Suite Variable  ${output_file}

    # Create and activate report
    ${current_date}=  Get Current Date
    initialize logging to "${main_report}"
    activate report "${main_report}"
    add report title "Test Case Title - Test Report"
    add report title "Device: ${DUT}"
    add report title "Date: ${current_date}"

    add report line break
```

#### Step 2: Add Report Title BEFORE Each Step
```robot
PRE-TEST Baseline Verification
    [Documentation]  Verify system baseline before test

    # ✅ MANDATORY - Add title BEFORE executing step
    add report title "PRE-TEST: Baseline Verification"

    connect to device "${DUT}"
    run "terminal length 0"

    Verify Software NX-OS  ${DUT}
    Check Memory NX-OS  ${DUT}
    Check CPU and System Resources NX-OS  ${DUT}

    # ✅ MANDATORY - Report Pass/Fail for this step
    pass and add comment to report  ${DUT}: Baseline verification completed successfully

    # ✅ MANDATORY - Add line break after step completes
    add report line break
```

#### Step 3: Report Pass/Fail for Each Major Step
```robot
STEP 1 Execute Configuration Changes
    [Documentation]  Apply configuration changes

    # ✅ Title BEFORE execution
    add report title "STEP 1: Execute Configuration Changes"

    select device "${DUT}"

    # Execute operations
    ${result}=  run "configure terminal ; interface Eth1/1 ; no shutdown"

    # ✅ MANDATORY - Report result (Pass or Fail)
    Run Keyword And Continue On Failure
    ...  pass and add comment to report  ${DUT}: STEP 1 PASSED - Configuration applied successfully

    # ✅ MANDATORY - Line break after step
    add report line break


STEP 2 Verify Configuration
    [Documentation]  Verify changes took effect

    # ✅ Title BEFORE execution
    add report title "STEP 2: Verify Configuration"

    select device "${DUT}"

    ${output}=  run "show interface Eth1/1 | inc is"

    # Check if interface is up
    ${status}=  Run Keyword And Return Status
    ...  Should Contain  ${output}  is up

    # ✅ MANDATORY - Report Pass OR Fail explicitly
    Run Keyword If  ${status}
    ...  pass and add comment to report  ${DUT}: STEP 2 PASSED - Interface is up
    ...  ELSE
    ...  fail and add comment to report  ${DUT}: STEP 2 FAILED - Interface is not up

    # ✅ MANDATORY - Line break after step
    add report line break
```

#### Step 4: Always Include POST-TEST and Summary
```robot
POST-TEST System Health Checks
    [Documentation]  Final verification

    # ✅ Title BEFORE execution
    add report title "POST-TEST: System Health Checks"

    select device "${DUT}"

    Check Memory NX-OS  ${DUT}
    Check CPU and System Resources NX-OS  ${DUT}
    Check Logging NX-OS  ${DUT}

    # ✅ Report Pass/Fail
    pass and add comment to report  ${DUT}: POST-TEST PASSED - System health verified

    # ✅ Line break
    add report line break


Test Summary
    [Documentation]  Test execution summary

    # ✅ Title for summary section
    add report title "TEST EXECUTION SUMMARY"

    pass and add comment to report  Test Case: [Test Case ID and Name]
    pass and add comment to report  Device: ${DUT}
    pass and add comment to report  SUMMARY:
    pass and add comment to report  Step 1: PASSED - Configuration applied
    pass and add comment to report  Step 2: PASSED - Verification successful
    pass and add comment to report  POST-TEST: PASSED - System healthy
    pass and add comment to report  OVERALL TEST RESULT: PASS

    # ✅ MANDATORY - Close report at the end
    add report line break
    disable report logging
```

---

## 📝 Reporting Rules Summary

### MANDATORY for Every Step:
1. ✅ **Add report title** → Execute code → Report Pass/Fail → Add line break
2. ✅ **Pass/Fail visibility**: Every major step MUST report its result explicitly
3. ✅ **Line breaks**: Add after each completed step for readability
4. ✅ **Close report**: Always call `disable report logging` at the end

### Template for Each Step:
```robot
STEP X Step Description
    [Documentation]  What this step does

    add report title "STEP X: Step Description"  # ← BEFORE execution

    # Execute your code here
    # ...

    # Report result
    pass and add comment to report  ${DUT}: STEP X PASSED - Success message
    # OR
    fail and add comment to report  ${DUT}: STEP X FAILED - Failure message

    add report line break  # ← AFTER completion
```

---

## 🔧 NX-OS Version Information

### Target Code Version
**NX-OS 10.6.2** (upgraded from 10.3.4a)

**Devices on 10.6.2 (majority)**:
- All ToR devices (TOR0-6, TOR-SC1/SC2)
- AGG0, AGG1, AGG2, AGG3, AGG4
- CORE1, CORE3, CORE4
- All MBA devices (MBA1-4)

**Devices on old code (compatibility testing)**:
- AGG-SC1-C9364C-GX
- AGG-SC2-C9364C-GX
- CDC-C93180YC-FX
- CORE2-C9504

**Note**: Tests should target 10.6.2 features unless specifically testing backward compatibility.

---

## 🔧 Platform-Specific Keywords

### Use Correct Platform Suffix
```robot
# ✅ For NX-OS (Nexus switches)
Check Memory NX-OS  ${DUT}
Verify BGP Peering NX-OS  ${DUT}
verify list of interfaces are up NX-OS  ${DUT}  @{Interface_List}

# ✅ For IOS-XE (ASR/Catalyst routers)
Check Memory IOS-XE  ${ROUTER}
Verify BGP Peering IOS-XE  ${ROUTER}
verify list of interfaces are up IOS-XE  ${ROUTER}  @{Interface_List}
```

---

## 📚 Use Existing Keywords (Don't Reinvent the Wheel)

### Always Check Customer_Keywords.robot First!
```robot
# ✅ GOOD - Use existing keywords
Verify Software NX-OS  ${DUT}
Check Memory NX-OS  ${DUT}
Check CPU and System Resources NX-OS  ${DUT}
Verify BGP Peering NX-OS  ${DUT}
Verify BGP Route Summary NX-OS  ${DUT}

# ❌ BAD - Don't create new keywords if they exist
Custom Check Memory  ${DUT}  # Reinventing the wheel!
```

### Common Keyword Categories in Customer_Keywords.robot:
- **Connection Management**: `connect to device`, `select device`, `run`
- **Interface Verification**: `verify list of interfaces are up NX-OS/IOS-XE`
- **BGP Verification**: `Verify BGP Peering`, `Verify BGP Route Summary`
- **System Health**: `Check Memory`, `Check CPU and System Resources`, `Check Logging`, `Check Core`
- **Spirent Traffic**: `spirent load`, `spirent start traffic`, `Verify Spirent Zero Packet Loss`
- **Reporting**: `pass and add comment to report`, `fail and add comment to report`, `warn and add comment to report`

---

## 🎯 Required CXTA Libraries

```robot
*** Settings ***

# ✅ MANDATORY - SVS CXTA libraries
Library     CXTA
Resource    community/generic.robot
Resource    cxta.robot

# ✅ MANDATORY - Standard Robot Framework libraries
Library     String
Library     BuiltIn
Library     DateTime
Library     Collections
Library     OperatingSystem

# ✅ MANDATORY - CUSTOMER_NAME resources (ABSOLUTE PATHS!)
Resource    /home/cisco/cxta/GS/N9K_HBN_Certification/Common/Customer_Keywords.robot
Variables   /home/cisco/cxta/GS/N9K_HBN_Certification/Common/Variables.yaml
```

---

## 🔍 Device and Variable References

### From topology.yaml
```robot
# ✅ Use exact device names from topology.yaml
${DUT}=  Set Variable  TOR1-C9348GC-FXP
${DUT}=  Set Variable  AGG1-C93180YC-FX3
${DUT}=  Set Variable  CORE1-C9508
```

### From Variables.yaml
```robot
# ✅ Dictionary variables
${TOR1-C9348GC-FXP_Links}           # Device links
${TOR1-C9348GC-FXP_BGP_Neighbors}   # BGP neighbors
${TOR1-C9348GC-FXP_Interfaces}      # Interface list

# ✅ List variables
@{Interface_List}=  Get Dictionary Keys  ${${DUT}_Links}

# ✅ Scalar variables
${SPIRENTCONFIG}
${precheck_report}
```

---

## ⚙️ Documentation and Tags

```robot
# ✅ MANDATORY - Suite-level documentation
Documentation   Test case description
...             What is being tested
...             Pass/fail criteria

# ✅ MANDATORY - Test case documentation
Test Case Name
    [Documentation]  Clear description of what this test validates
    [Tags]  feature-name  platform-type  test-phase
```

---

## ⏱️ Timeouts and Waits

```robot
# ✅ Set appropriate timeouts for long operations
set unicon execute timeout to "600" seconds

# ✅ Add stabilization waits when needed
Sleep  10s  # Allow system to stabilize after changes
Sleep  5s   # Wait for BGP convergence
```

---

## 📤 Output File Management

```robot
# ✅ Create output files for command captures
Create File  ${output_file}  Header: Test started at ${current_date}\n

# ✅ Append command outputs
Append To File  ${output_file}  \n========================================\n
Append To File  ${output_file}  Command: ${cmd}\n
Append To File  ${output_file}  ========================================\n
Append To File  ${output_file}  ${output}\n\n
```

---

## 🛡️ Error Handling Best Practices

### Device Connection Pattern (MANDATORY)

```robot
# ✅ MANDATORY - Always use Continue On Failure for device connections
# This handles transient SSH authentication issues while allowing Unicon auto-reconnect
FOR  ${DUT}  IN  @{DUT_List}
    add report comment "DUT : ${DUT}"
    Run Keyword And Continue On Failure  connect to device "${DUT}"
    run "terminal length 0"
END
```

**Why This Pattern:**
- Handles transient SSH authentication failures (common with TACACS)
- Allows Unicon's automatic reconnection to complete in background
- Test continues and uses reconnected session for subsequent commands
- Prevents false negatives from temporary connection issues

**CRITICAL:** Some devices (like TOR2 in CUSTOMER_NAME HBN lab) experience SSH authentication retries due to key-based auth attempts before password auth. Using `Run Keyword And Continue On Failure` allows the test to proceed while Unicon handles reconnection automatically.

### General Error Handling

```robot
# ✅ Continue on failure (don't stop entire test)
Run Keyword And Continue On Failure
...  pass and add comment to report  Step passed

# ✅ Capture status for conditional reporting
${status}=  Run Keyword And Return Status
...  Should Contain  ${output}  expected_text

Run Keyword If  ${status}
...  pass and add comment to report  Check passed
...  ELSE
...  fail and add comment to report  Check failed

# ✅ Use try-except pattern for critical operations
Run Keyword And Ignore Error
...  Some Non-Critical Operation
```

---

## 📋 Pre-Flight Checklist (Before Uploading to CXTA)

### File Structure
- [ ] File named `jobfile.robot` (standard naming convention)
- [ ] Absolute paths for all resources (`/home/cisco/cxta/...`)
- [ ] Suite Setup includes `load testbed`
- [ ] Suite Teardown includes `Disconnect from all devices`
- [ ] All required libraries imported
- [ ] **Device connections use `Run Keyword And Continue On Failure`** (MANDATORY)

### Test Case Structure
- [ ] Initialize Test Variables (create report)
- [ ] PRE-TEST baseline verification
- [ ] Each STEP has: Title → Execute → Pass/Fail → Line break
- [ ] POST-TEST health verification
- [ ] Test Summary with overall result
- [ ] `disable report logging` at the end

### Reporting Quality
- [ ] Report title added BEFORE each step
- [ ] Pass/Fail explicitly stated for each step
- [ ] Line breaks after each step
- [ ] Summary shows all step results
- [ ] Overall PASS/FAIL in summary
- [ ] **NO Unicode characters** (═ • ✓) - Use ASCII only (= * PASS)

### Keywords and References
- [ ] Using existing keywords from Customer_Keywords.robot
- [ ] Correct platform suffix (NX-OS vs IOS-XE)
- [ ] Device names from topology.yaml
- [ ] Variables from Variables.yaml

### Documentation
- [ ] Suite Documentation present
- [ ] Test case [Documentation] present
- [ ] [Tags] applied appropriately

---

## ✅ Final Verification Commands

Before marking jobfile as ready, verify:
```bash
# Check file locally for absolute paths
cat /path/to/jobfile.robot | grep -E "(Resource|Variables|load testbed)"
# Should show absolute paths: /home/cisco/cxta/...

# Verify all steps have proper structure
cat /path/to/jobfile.robot | grep "add report title"
cat /path/to/jobfile.robot | grep "add report line break"
cat /path/to/jobfile.robot | grep "pass and add comment to report"

# CRITICAL: Check for Unicode characters (triggers CXTA bug)
grep -P "[\u2500-\u257F\u2022\u2713\u2717]" /path/to/jobfile.robot
# Should return NOTHING - if Unicode found, replace with ASCII!
```

---

## 🎯 Quality Standards

**Every jobfile must:**
1. ✅ Named `jobfile.robot` (standard filename)
2. ✅ Use absolute CXTA paths
3. ✅ Include Suite Teardown for cleanup
4. ✅ **Device connections with `Run Keyword And Continue On Failure`** (handles SSH auth retries)
5. ✅ Report title before EVERY step
6. ✅ Pass/Fail after EVERY step
7. ✅ Line break after EVERY step
8. ✅ Test Summary with all results
9. ✅ Close report with `disable report logging`

**Zero tolerance for:**
- ❌ Wrong filename (must be `jobfile.robot`)
- ❌ Relative paths
- ❌ Missing disconnection
- ❌ **Device connections without `Run Keyword And Continue On Failure`** (causes false negatives)
- ❌ Steps without Pass/Fail reporting
- ❌ Missing report titles
- ❌ No test summary
- ❌ **Unicode characters** (═ • ✓ etc.) - **CRITICAL: Triggers CXTA bug!**

---

## 📞 Questions or Issues?

Refer to:
- `Customer_Keywords.robot` - Available keywords
- `Variables.yaml` - Device variables and parameters
- `topology.yaml` - Device names and connection details
- `CLAUDE.md` - Project-specific guidelines

**Remember**: Quality and consistency are critical for CUSTOMER_NAME validation testing!
