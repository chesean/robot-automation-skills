# Consolidate Test Results as DOCX

Generate a procedure-driven DOCX evidence report from test case execution artifacts.

## What This Skill Does

Creates a professional test evidence document that follows the exact test procedure, embedding CLI output, CSV data, screenshots/diagrams at each corresponding step. The report tells the story of the test from start to finish with explanatory notes.

## CRITICAL RULES

### No AI Footprint
- NEVER include "Claude", "AI", "generated", "automated by AI", "LLM", or any similar attribution
- NEVER include "Co-Authored-By" or "Generated with" or any tool attribution
- The document must appear as a standard engineering test report
- No meta-commentary about how the report was created

### Procedure-Driven Format (Not Dump Format)
- Each section matches a step in the test procedure (PRE-TEST, STEP 1, STEP 2, ... POST-TEST)
- CLI output is extracted from the main report for specific sections using markers
- Screenshots/images are embedded at the relevant step they belong to
- CSV files are embedded at the relevant step they provide evidence for
- Blue note paragraphs explain what is happening at each step
- The document reads as a narrative of the test execution

### Formatting Standards
- Title: Heading level 1
- Test info block: Blue note (9pt, RGB 0,80,160)
- Step headings: Heading level 2
- Sub-sections: Heading level 3
- CLI output: Courier New 7pt black, tight spacing
- Notes/explanations: 9pt blue (RGB 0,80,160)
- Summary block: Courier New 9pt
- Screenshots: 5.5 inches wide
- CSV data: Rendered as native docx tables (NOT converted to text)
- File source citations: 7pt italic gray

## Usage

```
/consolidate-test-results-as-docx [test-case-directory]
```

## When To Use

- After test execution completes and all artifacts are in the test case directory
- After `/test-case-summary` has been created
- Before uploading results to CXTM
- When consolidating manual screenshots with automated logs

## Implementation

### Step 1: Inventory All Artifacts

Scan the test case directory for:
```
*.txt          - Main report file(s), device logs
*.log          - Device session logs
*.csv          - ISE RADIUS records (Livelog, Live Sessions, Accounting)
*.png / *.jpg  - Screenshots (ISE, Spirent, topology diagrams)
*.robot        - Jobfile (for procedure reference)
TEST_CASE_SUMMARY.md - Summary (if already created)
```

List all found artifacts and map them to test steps.

### Step 2: Read the Main Report

The main report file (typically `*_Main_Report.txt`) contains the complete test execution log with section markers like:
```
==================== PRE-TEST - ... ====================
==================== STEP 1 - ... ====================
==================== STEP 2 - ... ====================
[Comment] PASS: ...
[Comment] FAIL: ...
[Run] DEVICE - command
```

Read the full report and identify all section markers.

### Step 3: Read the Jobfile

Read `jobfile.robot` to understand:
- Test procedure and step definitions
- Variable references (device names, IPs, MACs)
- Expected behavior at each step

### Step 4: Map Evidence to Steps

For each test step, identify which artifacts provide evidence:

| Step | Evidence Type | Source |
|------|--------------|--------|
| PRE-TEST | CLI output | Main report sections |
| STEP N (ISE action) | Screenshot | *.png matching step |
| STEP N (RADIUS auth) | CSV data | RADIUS Livelog*.csv, Live Sessions*.csv |
| STEP N (verification) | CLI output | Main report sections |
| POST-TEST | CLI output | Main report sections |

### Step 5: Generate create_report.py

Create a Python script `create_report.py` in the test case directory that:

1. Uses `python-docx` library
2. Reads the main report file
3. Extracts sections using marker-based extraction (NOT hardcoded line numbers)
4. Embeds evidence at corresponding steps
5. Adds explanatory blue notes for each step
6. Generates the final .docx

**CRITICAL: Use marker-based extraction, not line numbers:**
```python
def extract_section(lines, start_marker, end_marker=None, max_lines=80):
    """Extract lines between markers from report."""
    result = []
    capturing = False
    for line in lines:
        if start_marker in line:
            capturing = True
            continue
        if capturing:
            if end_marker and end_marker in line:
                break
            result.append(line.rstrip())
            if len(result) >= max_lines:
                break
    return result
```

### Step 6: Run and Verify

```bash
cd [test-case-directory]
python3 create_report.py
```

Confirm output file is created and report file size.

## Document Structure Template

```
TITLE: [TestID] - [DUT] [Test Description] (Automated)

TEST INFO NOTE (blue):
  Test Case: [ID] - [Description]
  DUT: [Device] ([IP]) | [Platform/Version]
  [Additional devices if applicable]
  Test Port: [Interface] ([mode/config])
  [Key endpoints/MACs/IPs]
  Executed: [Date] via CXTA automated jobfile

PRE-TEST - Baseline
  [Sub-sections with CLI extracts showing initial state]
  [Screenshots of ISE baseline if applicable]
  [Note confirming baseline state]

STEP 1 - [First Action]
  [Note explaining what this step does]
  [CLI output from main report]
  [Screenshots if ISE action]
  [CSV data if RADIUS event]
  [Note confirming step result]

STEP 2 - [Second Action]
  ...

STEP N - [Final Action / Cleanup]
  ...

POST-TEST - System Health
  [System resources, cores, tracebacks]
  [Final connectivity verification]
  [Note confirming system health]

Test Summary
  [Courier New 9pt block with:]
  - Test case info
  - Step-by-step procedure summary
  - Key data points (counters, match hits)
  - Key validation points
  - RESULT: PASSED/FAILED
```

## Helper Functions (Standard Pattern)

Every create_report.py should include these standard functions:

```python
def read_report():
    """Read the main report file."""

def extract_section(lines, start_marker, end_marker=None, max_lines=80):
    """Extract lines between markers from report."""

def add_cli_block(doc, lines, font_size=7):
    """Add CLI output as Courier New formatted block."""

def add_note(doc, text):
    """Add blue explanatory note (9pt, RGB 0,80,160)."""

def find_file(pattern):
    """Find file matching glob pattern in test directory."""

def add_csv_evidence(doc, filepath, caption):
    """Add CSV as native docx table (NOT text). Filter empty columns, use 'Light Grid Accent 1' style, Courier New 7pt."""

def add_image_if_exists(doc, pattern, caption, width=Inches(5.5)):
    """Add screenshot if found in directory."""
```

## CSV Data: Render as Native DOCX Tables (MANDATORY)

**CRITICAL: CSV files MUST be rendered as proper docx tables, NOT converted to text or key-value pairs.**

### Implementation Pattern:

```python
def add_csv_evidence(doc, filepath, caption):
    """Add CSV as native docx table. Filter empty columns."""
    with open(filepath, "r", errors="replace") as f:
        reader = csv.reader(f)
        rows = list(reader)
    if len(rows) < 2:
        return
    headers = rows[0]
    # Filter out columns where all data cells are empty
    non_empty_cols = []
    for col_idx in range(len(headers)):
        header_val = headers[col_idx].strip()
        has_data = any(
            col_idx < len(row) and row[col_idx].strip()
            for row in rows[1:]
        )
        if header_val and has_data:
            non_empty_cols.append(col_idx)
    # Create table with style
    table = doc.add_table(rows=1 + len(data_rows), cols=len(filtered_headers))
    table.style = "Light Grid Accent 1"
    # Header: Courier New 7pt bold
    # Data: Courier New 7pt regular
    # Source citation: 7pt italic gray below table
```

### Rules:
- Always filter out empty columns (ISE CSVs often have many empty fields)
- Use `Light Grid Accent 1` table style for consistent formatting
- Header row: Courier New 7pt **bold**
- Data rows: Courier New 7pt regular
- Add source filename citation below table (7pt italic gray)
- NEVER convert CSV to text key-value pairs - always use table format

## Reference Implementations

These existing reports demonstrate the correct format:

- `21534-B1.02-1827185/create_report.py` - Multi-Auth dACL with CoA and ISE screenshots
- `21534-B2.15-1900100/create_report.py` - MDA DACL Voice VLAN with ISE screenshots
- `21534-B2.17-1900766/create_report.py` - MDA DACL DenyAll with CSV evidence
- `21534-C2.17-1900767/create_report.py` - MDA DACL DenyAll on TOR5 with CSV as tables, ISE pre-condition screenshots

Study these before creating new reports to maintain consistency.

## Output Naming Convention

```
[TestID]-[DUT-Short]-[Test-Description]-automated.docx
```

Examples:
- `B1.02-TOR1-Multi-Auth-with-dACL-CoA-revoke-access-automated.docx`
- `B2.15-TOR1-DACL-MDA-Port-Voice-automated.docx`
- `B2.17-TOR1-MDA-DACL-DenyAll-PC-Revoke-automated.docx`

## What NOT to Include

- No AI attribution or footprint of any kind
- No emojis
- No raw log dumps (extract relevant sections only)
- No redundant CLI output (show same data once, not repeatedly)
- No marketing language or superlatives
- No timestamps in notes (timestamps are in the CLI output already)
- No "this was generated by" statements

## Integration with Other Skills

1. Run `/cxtm-ready` first (loads project context)
2. Run `/test-case-summary` (creates TEST_CASE_SUMMARY.md)
3. Run `/consolidate-test-results-as-docx` (creates evidence DOCX)
4. Upload both to CXTM

---

**Use this after every test execution to create professional evidence documents!**
