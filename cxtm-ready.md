# CXTM Ready Skill

Load all CXTM Robot Framework automation context.

## Usage
```
/cxtm-ready
```
Run at start of every CXTM work session.

## Steps (execute in order)

### 1. Read Project Context Pack
```
Read: repomix/cxtm-project-context.md
```
Contains: Goldman_Keywords.robot, CXTA_BEST_PRACTICES.md, HBN_Network_Config_Template.md, design_doc.md, .svstest/

### 2. Cache Lab Config
```
Read: lab.yaml
Read: Variables.yaml
```
Cache all 35 device hostnames, IPs, credentials. User should never need to ask.

### 3. Read CXTA Library
```
Read: repomix/cxta-library-code.md
```
Contains: core framework, community resource files (nxos.robot, spirent.robot), keyword implementations.

**Large modules (load on demand):**
- DeviceCli.py (105K), Parsing.py (71K), Config.py (57K), Compare.py (34K)
- Spirent.py (145K), SpirentRest.py (192K)

### 4. Confirm Ready

```
CXTM AUTOMATION CONTEXT LOADED
  Project: 21534 - Goldman Sachs HBN MAB Phase 1
  Devices: 35 cached | Standards: ASCII-only, jobfile.robot, absolute paths
  Context: Project pack + CXTA library loaded
READY FOR CXTM TEST AUTOMATION
```
