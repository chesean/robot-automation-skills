# HBN Baseline Check Skill

SSH into all 22 Baseline_DUT_List devices and verify BGP route scale + ECMP groups.

## Usage
```
/hbn-baseline-check
```

## When To Use
- Before 4-6 hour CXTM test executions (I4.xx, J4.xx, K4.xx, etc.)
- After network changes or Spirent traffic profile start (~18 min convergence)

## Pre-requisites
- Spirent traffic profile running with emulated devices started
- TOR-SC1/SC2 converged (~18 min)
- MBA devices have `ip routing download-on-convergence` enabled

## Process

1. Read Variables.yaml for expected BGP routes and ECMP groups per device
2. Run `python3 check_hbn_route_scale.py` (SSHes 22 devices in parallel via Netmiko)
3. Pass/fail: FAIL if found < expected OR found > expected + 50
4. Report results, save to `hbn-route-scale-analysis.md`

## 22 Devices
TOR-SC1/SC2, TOR1-6, AGG-SC1/SC2, AGG1-4, CORE1-4, MBA1-4

## Common Failures

| Pattern | Cause | Fix |
|---------|-------|-----|
| All MBA ECMP ~5523 | Missing download-on-convergence | Enable on MBA1-4 |
| TOR ECMP ~1100 short | SC not converged | Wait 18 min after Spirent start |
| Single device -1 BGP | Topology asymmetry | Update Variables.yaml |
