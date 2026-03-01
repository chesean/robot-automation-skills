# HBN Baseline Check Skill

SSH into all 22 HBN Baseline_DUT_List devices and verify BGP route scale and ECMP groups against Variables.yaml expected values.

## What This Skill Does

Connects to all 22 NX-OS devices in parallel via Netmiko and checks:
1. BGP route count in default VRF (`show ip route bgp summary`)
2. ECMP group utilization (`show hardware internal forwarding table utilization module 1`)
3. Compares against expected values defined in Variables.yaml
4. Uses same pass/fail logic as Customer_Keywords.robot: FAIL if found < expected OR found > expected + 50

## Usage

```
/hbn-baseline-check
```

## When To Use

- Before running any 4-6 hour CXTM test execution (I4.xx, J4.xx, K4.xx, etc.)
- After network changes (config, upgrades, route-map modifications)
- After Spirent traffic profile is started and routes have converged (~18 min)
- When troubleshooting test failures related to BGP route counts or ECMP

## Pre-requisites

- Spirent traffic profile must be running with emulated devices started
- TOR-SC1/SC2 must be fully converged (~18 min after Spirent device start)
- All MBA devices must have `ip routing download-on-convergence` enabled
- SSH access to all devices via gssvs-cxtm credentials

## Implementation

### Step 1: Read Variables.yaml for Expected Baselines

Read Variables.yaml and extract:
- `{DEVICE}_BGP_Routes: default:` value for each of the 22 Baseline_DUT_List devices
- `{DEVICE}_ECMP_groups_used:` value for each device

The 22 devices in Baseline_DUT_List are:
```
TOR-SC1-C93180YC-FX, TOR-SC2-C93180YC-FX,
TOR1-C9348GC-FXP, TOR2-C9348GC-FXP, TOR3-C9348GC-FXP, TOR4-C9348GC-FXP,
TOR5-C9348GC-FX3PH, TOR6-C9348GC-FX3PH,
AGG-SC1-C9364C-GX, AGG-SC2-C9364C-GX,
AGG1-C93180YC-FX3, AGG2-C93180YC-FX3, AGG3-C93180YC-FX, AGG4-C93180YC-FX,
CORE1-C9508, CORE2-C9504, CORE3-C9364C-GX, CORE4-C9364C-GX,
MBA1-C93240YC-FX2, MBA2-C93180YC-FX, MBA3-C93180YC-FX, MBA4-C93240YC-FX2
```

### Step 2: Run the Check Script

Execute `check_hbn_route_scale.py` from the Automation directory. This script:
- SSHes into all 22 devices in parallel (10 threads) using Netmiko
- Collects BGP route summary for default VRF
- Collects ECMP group utilization from forwarding table
- Also samples GSINET and VPN-2 VRF route counts on devices that have them
- Compares all values against Variables.yaml baselines
- Tolerance: PASS if found >= expected AND found <= expected + 50

```bash
cd "/Users/sche/Documents/AS-SVS-Projects/GS/HBN MAB Ph1/Automation"
python3 check_hbn_route_scale.py
```

**IMPORTANT**: Before running, verify the expected values in check_hbn_route_scale.py match Variables.yaml. If Variables.yaml baselines have been updated, the script's hardcoded values must also be updated. Alternatively, read the values directly from Variables.yaml at runtime.

### Step 3: Review Results and Report

After the script completes, display a clear summary to the user:

```
HBN BASELINE CHECK RESULTS - [date/time]

BGP Default VRF: [X]/22 PASS, [Y]/22 FAIL
ECMP Groups:     [X]/22 PASS, [Y]/22 FAIL
VRF Samples:     [X] PASS, [Y] FAIL

[If all pass]
ALL DEVICES MEET BASELINE - Ready for testing

[If any fail]
DEVICES NOT MEETING BASELINE:

BGP Failures:
  [device]: expected [X], found [Y] (delta [Z])

ECMP Failures:
  [device]: expected [X], found [Y] (delta [Z])

RECOMMENDED ACTIONS:
  - [specific action per failure type]
```

### Step 4: Save Results

Save the output to `hbn-route-scale-analysis.md` in the Automation directory (overwrite previous).
This serves as the pre-test baseline verification record.

Also save a timestamped copy to `HBN-baseline/` for historical tracking:
```
HBN-baseline/YYYYMMDD_route_scale_baseline.md
```

## Pass/Fail Logic (same as Customer_Keywords.robot)

```
BGP:  FAIL if found < expected  OR  found > expected + 50
ECMP: FAIL if found < expected  OR  found > expected + 50
```

This means the acceptable range for each device is: [expected, expected + 50]

## Common Failure Patterns

| Pattern | Likely Cause | Action |
|---------|-------------|--------|
| All MBA ECMP at ~5523 | Missing `ip routing download-on-convergence` | Enable on MBA1-4 |
| TOR1-6 ECMP ~1100 short | TOR-SC1/SC2 not converged | Wait 18 min after Spirent device start |
| Single device -1 BGP route | Structural topology asymmetry | Update Variables.yaml baseline |
| Multiple devices BGP short | Route redistribution issue | Check BGP peering and route-maps |
| Connection errors | Device unreachable | Check management IP and SSH access |

## Related Files

- `Variables.yaml` - Source of expected BGP and ECMP baselines
- `check_hbn_route_scale.py` - The check script
- `hbn-route-scale-analysis.md` - Latest check results
- `HBN-baseline/` - Historical baseline snapshots
- `Customer_Keywords.robot` - Keywords that use these baselines during testing
