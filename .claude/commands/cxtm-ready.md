# CXTM Ready Skill

Load all CXTM Robot Framework automation context using Repomix code packs.

## What This Skill Does

**One command loads everything needed for CXTM test automation.**

Replaces reading multiple individual files with focused Repomix packs that provide complete context for writing Robot Framework tests.

## Usage

```
/cxtm-ready
```

## When To Use

- Start of every CXTM work session
- Before automating test cases
- Before creating new jobfiles
- Before reviewing/modifying existing Robot scripts

---

## Implementation (4 Steps)

Execute these steps in order:

### Step 1: Read Project Context Pack (Standards + Custom Keywords)

```
Read: repomix/cxtm-project-context.md
```

This single file (90K tokens, 21 files) contains:
- **Customer_Keywords.robot** (1,963 lines) - All custom keywords for this project
- **CXTA_BEST_PRACTICES.md** - Mandatory coding standards (ASCII-only, step reporting, naming)
- **SVS_TEMPLATE_BEST_PRACTICES.md** - General automation patterns
- **HBN_Network_Config_Template.md** - Network design reference (NX-OS configs, SR-MPLS, EVPN)
- **design_doc.md** - Original design document with topology
- **.svstest/** - Project setup framework, schemas, scripts, linting checklist

After reading, you know HOW to write Robot tests for CUSTOMER_NAME HBN.

### Step 2: Cache Lab Configuration (Device IPs + Credentials)

```
Read: lab.yaml
Read: Variables.yaml
```

Cache in memory:
- All 35 device hostnames, IPs, and credentials
- Test parameters and variables
- User should NEVER need to ask for device info

### Step 3: Read CXTA Library Code Pack (Framework Keywords)

```
Read: repomix/cxta-library-code.md
```

This file (107K tokens, 98 files) contains:
- **core/** - Framework internals (connectors, logging, reporting, secrets)
- **robot/community/** - Platform resource files (nxos.robot, spirent.robot, iosxe.robot)
- **robot/*.py** - Keyword implementations (Shared, Util, Results, Testbed, ReportLogging, etc.)
- **robot/platforms/nexus/** - NX-OS platform-specific code

After reading, you know WHAT CXTA keywords are available and how they work.

**Large modules NOT in pack (load on demand when needed):**
- `CXTA/src/CXTA/robot/DeviceCli.py` (105K) - CLI interaction keywords
- `CXTA/src/CXTA/robot/Parsing.py` (71K) - Output parsing keywords
- `CXTA/src/CXTA/robot/Config.py` (57K) - Config management keywords
- `CXTA/src/CXTA/robot/Compare.py` (34K) - Comparison keywords
- `CXTA/src/CXTA/robot/Spirent.py` (145K) - Spirent traffic keywords
- `CXTA/src/CXTA/robot/SpirentRest.py` (192K) - Spirent REST keywords

### Step 4: Confirm Ready

Display summary and confirm ready for work.

---

## Summary Display

After all steps, present:

```
CXTM AUTOMATION CONTEXT LOADED - READY FOR WORK

PROJECT
  ID: 21534 - CUSTOMER_NAME HBN MAB Phase 1 Certification
  Platform: CXTA [version from repo]
  Devices: 35 total (cached from lab.yaml)

CONTEXT LOADED
  Project standards: 90K tokens (21 files - keywords, best practices, design)
  CXTA library: 107K tokens (98 files - framework code, keyword implementations)
  Lab config: All device IPs and credentials cached

MANDATORY STANDARDS
  - File naming: jobfile.robot
  - Step reporting: Every action reported
  - ASCII only: NO Unicode (CXTA bug)
  - Absolute paths: /home/cisco/cxta/GS/N9K_HBN_Certification/...
  - Suite teardown: Disconnect from all
  - Connection handling: Continue On Failure
  - Timeouts: 120s for large outputs

NETWORK TOPOLOGY
  ToR: TOR0-6, TOR-SC1-2 (9 devices)
  Aggregation: AGG0-4, AGG-SC1-2 (7 devices)
  Core: CORE1-4 (4 devices)
  MBA: MBA1-4 (4 devices)
  WAN: WAN1-2, R1-2 (4 devices, IOS-XE)
  Console: FANOUT1-6 (6 devices)
  Servers: 4 (Linux/ISE)

PLATFORM NOTES
  - NO Unix pipes (grep, tail) on network devices
  - Use NX-OS native: | include, | last, | section
  - Netmiko for network devices, Paramiko for Linux servers
  - NX-OS 10.6.2 target (except AGG-SC1/SC2, CDC, CORE2)

READY FOR CXTM TEST AUTOMATION
```

---

## On-Demand Loading

When writing specific test types, load the relevant large module:

| Test Type | Load File |
|-----------|-----------|
| CLI interaction | `CXTA/src/CXTA/robot/DeviceCli.py` |
| Output parsing | `CXTA/src/CXTA/robot/Parsing.py` |
| Config management | `CXTA/src/CXTA/robot/Config.py` |
| Comparisons | `CXTA/src/CXTA/robot/Compare.py` |
| Spirent traffic | `CXTA/src/CXTA/robot/Spirent.py` + `SpirentRest.py` |

---

## Key Knowledge (Always Available After Loading)

- **CUSTOMER_NAME standards**: ASCII-only, step reporting, jobfile.robot naming, absolute paths, suite teardown
- **Custom keywords**: Customer_Keywords.robot (1,963 lines of project-specific automation)
- **Device access**: All 35 devices cached (hostnames, IPs, credentials, platforms)
- **Network design**: SR-MPLS, BGP-LU, EVPN, NX-OS 10.6.2 target
- **CXTA framework**: Core keywords, community resource files, utility functions
- **SSH rules**: Netmiko for network devices (persistent sessions), Paramiko for Linux servers

## Regenerating the Packs

When CXTA is updated or project files change:

```
See: repomix/README.md for regeneration instructions
```

---

## Related Skills

After `/cxtm-ready`, you can also use:
- `/test-case-summary` - Analyze test results and generate formatted summary
- Standard CXTM operations (list test cases, get details, upload results)

---

**Use `/cxtm-ready` at the start of every CXTM work session!**
