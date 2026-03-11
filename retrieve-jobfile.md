# Retrieve Jobfile from CXTM

Download a jobfile from CXTM project 21534 by testcase ID → save as `jobfile-old.txt`.

## Usage
```
/retrieve-jobfile <testcase_id>
```

---

## Critical Facts

| Item | Value |
|------|-------|
| Auth header | `X-TM2-API-KEY: YOUR_API_KEY_HERE` |
| Wrong headers (→ 401) | `Authorization: Token ...` or `Authorization: Bearer ...` |
| Robot content field | `data.data.text` (NOT `data.content`, `data.robot_text`) |
| Testcase ID ≠ Jobfile ID | Folder name has testcase ID; jobfile ID must be looked up |
| Jobfile list tool | **CXTM MCP** `list_project_jobfiles` (Playwright fetch fails for list endpoint) |
| Content download tool | **Playwright MCP** `browser_evaluate` (CXTM MCP `get_jobfile_robot_content` gets aborted) |
| Save filename | `jobfile-old.txt` (NOT `jobfile.robot`) |

---

## Steps

**1. Find local folder**
```
Glob pattern: 21534-*-<testcase_id>
Base path: /Users/sche/Documents/AS-SVS-Projects/GS/HBN MAB Ph1/Automation/
```

**2. Find jobfile ID** via CXTM MCP:
```
mcp__hub__cxtm__list_project_jobfiles(project_id=21534, cxtm_api_token="YOUR_API_KEY_HERE")
```
Parse result (large JSON — use Python/grep on saved file) to match jobfile name to testcase title.
Get testcase title first if needed: navigate to CXTM, then `browser_evaluate` → `fetch('/api/v1/testcases/<testcase_id>/', { headers: h })`.

**3. Download jobfile content** via Playwright `browser_evaluate` (navigate to `https://cxtm.cisco.com` first):
```javascript
async () => {
  const h = { 'X-TM2-API-KEY': 'YOUR_API_KEY_HERE' };
  const r = await fetch('/api/v1/jobfiles/<JOBFILE_ID>/', { headers: h });
  const d = await r.json();
  return d.data.text;
}
```

**4. Save** using Write tool → `<local_folder>/jobfile-old.txt`

---

## Known ID Mappings

| Testcase ID | Jobfile ID | Description |
|-------------|------------|-------------|
| 1829256 | 800649 | Link UP AGG1-C93180YC-FX3 ↔ CORE1-C9508 |
| 1829257 | 800650 | Link DOWN AGG1-C93180YC-FX3 ↔ CORE2-C9504 |
| 1829258 | 800651 | Link UP AGG1-C93180YC-FX3 ↔ CORE2-C9504 |
| 1829259 | 800652 | Link DOWN AGG1-C93180YC-FX3 ↔ CORE3-C9364C-GX |
