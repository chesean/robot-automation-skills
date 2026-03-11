# Robot Coding Standards Skill

Load Robot Framework coding standards and syntax rules before writing or editing jobfile.robot files.

## What This Skill Does

**Loads CXTA best practices directly from source** (always up-to-date, unlike repomix snapshots).

Lightweight alternative to `/cxtm-ready` when you only need coding standards, not the full CXTA library and project context.

## Usage

```
/robot-coding
```

## When To Use

- Before writing a new jobfile.robot
- Before editing an existing jobfile.robot
- After a Robot Framework syntax error to check the rules
- When reviewing jobfile code for compliance

## When To Use `/cxtm-ready` Instead

- Start of a full CXTM work session (need keywords, variables, CXTA library)
- When you need Goldman_Keywords.robot reference
- When you need CXTA keyword implementations
- When you need device IPs and lab configuration

---

## Implementation (2 Steps)

### Step 1: Read CXTA Best Practices (Coding Standards)

```
Read: CXTA_BEST_PRACTICES.md
```

This file contains all mandatory Robot Framework coding rules:
- File naming convention (jobfile.robot)
- Absolute path requirements
- Connection management and Suite Teardown
- Report structure (PRE-TEST, STEPs, POST-TEST, Summary)
- ASCII-only rule (no Unicode - CXTA bug)
- No newline variables in report statements (no ${\\n})
- `add report title` embedded argument restrictions
- Colon restrictions in plain text
- NX-OS CLI limitations (no Unix pipes)
- Error handling patterns
- Pre-flight checklist

### Step 2: Confirm Ready

Display a brief confirmation:

```
ROBOT CODING STANDARDS LOADED

Key rules to follow:
  - File naming: jobfile.robot
  - ASCII only: NO Unicode characters
  - No ${\\n}: CXTA handles newlines internally
  - add report title: Simple headers only (no leading 1. 2. or -)
  - Report details: Use pass/fail and add comment to report
  - Absolute paths: /home/cisco/cxta/GS/N9K_HBN_Certification/...
  - Suite teardown: Disconnect from all devices
  - Connections: Use Run Keyword And Continue On Failure

Ready to write Robot Framework code.
```

---

## Quick Reference (Common Pitfalls)

These are the most frequently hit issues - all documented in CXTA_BEST_PRACTICES.md:

| Issue | Wrong | Right |
|-------|-------|-------|
| Newlines | `...text ${\\n}` (double backslash) | `...text ${\n}` (single) or `...text` (none) |
| Report titles | `add report title "1. First item"` | `pass and add comment to report  1. First item` |
| Report titles | `add report title "- Bullet point"` | `pass and add comment to report  Bullet point` |
| Unicode | `pass and add comment to report  PASS ✓` | `pass and add comment to report  PASS` |
| Colons | `pass and add comment to report  Platform: N9K` | `pass and add comment to report  Platform - N9K` |
| Paths | `Resource  ../Common/Goldman_Keywords.robot` | `Resource  /home/cisco/cxta/GS/.../Goldman_Keywords.robot` |

---

## Related Skills

- `/cxtm-ready` - Full context load (standards + keywords + CXTA library + lab config)
- `/test-case-summary` - Analyze test results and generate formatted summary
