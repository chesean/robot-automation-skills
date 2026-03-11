# Consolidate Test Results as DOCX

Generate procedure-driven DOCX evidence report from test case artifacts.

## Usage
```
/consolidate-test-results-as-docx [test-case-directory]
```

## No AI Footprint (MANDATORY)
No "Claude", "AI", "generated", or tool attribution. Standard engineering report.

## Document Structure

```
TITLE: [TestID] - [DUT] [Test Description] (Automated)

TEST INFO NOTE (blue 9pt):
  Test Case, DUT, Platform, Test Port, Executed date

PRE-TEST - Baseline [CLI extracts, screenshots]
STEP 1 - [Action] [CLI output, screenshots, CSV tables]
STEP N - ...
POST-TEST - System Health
Test Summary [Courier 9pt block with result]
```

## Formatting Standards
- CLI output: Courier New 7pt, tight spacing
- Notes: 9pt blue (RGB 0,80,160)
- Screenshots: 5.5 inches wide
- CSV: Native docx tables (`Light Grid Accent 1` style), NOT text
- Source citations: 7pt italic gray

## Process

1. **Inventory** artifacts: *.txt, *.log, *.csv, *.png, *.robot, TEST_CASE_SUMMARY.md
2. **Read main report** - identify section markers (`==== STEP N ====`)
3. **Read jobfile.robot** for procedure reference
4. **Map evidence to steps** - which artifacts go where
5. **Generate `create_report.py`** using marker-based extraction (NOT line numbers):
   ```python
   def extract_section(lines, start_marker, end_marker=None, max_lines=80):
   def add_cli_block(doc, lines, font_size=7):
   def add_note(doc, text):  # blue 9pt
   def add_csv_evidence(doc, filepath, caption):  # native docx table
   def add_image_if_exists(doc, pattern, caption, width=Inches(5.5)):
   ```
6. **Run**: `python3 create_report.py`

## CSV as Tables (MANDATORY)
- Filter empty columns, use `Light Grid Accent 1` style
- Header: Courier 7pt bold, Data: Courier 7pt regular
- NEVER convert CSV to text key-value pairs

## Reference Implementations
- `21534-B1.02-1827185/create_report.py` - Multi-Auth with ISE screenshots
- `21534-C2.17-1900767/create_report.py` - MDA DACL with CSV tables

## Output Naming
`[TestID]-[DUT-Short]-[Test-Description]-automated.docx`
