---
name: android-system-doctor
description: Full Android device health check through the aMiNo service shell — battery, storage, memory, CPU top consumers, network state, system identity, and an optional screen sanity check. Execute-first: every verdict line must cite real output seen in the session.
version: 1.0.0
author: AMINODa
category: diagnostics
tags: [android, diagnostics, system, battery, storage, memory, network]
allowed-tools: shell_command, screen_read, device_info
---

# Android System Doctor

Full device health check with REAL commands through the aMiNo service shell. Execute first, never guess: every verdict line must cite real output seen in this session.

## Execution rules
- Run each check with the `shell_command` tool (Android host shell). Do NOT use the Debian terminal for these — they are host commands.
- If `shell_command` fails because the service is not connected: STOP and tell the user to connect the aMiNo service first (drawer → Adb). Never fabricate outputs.
- If a single check fails, record FAIL with the real error text and continue with the remaining checks.
- This skill is strictly read-only: it never changes settings, never kills processes, never installs anything.

## Checks (run in this order)
1. **Battery** — `dumpsys battery | grep -E "level|status|temperature|health"`
   - level < 20 → WARN. temperature > 450 (that is 45.0°C) → WARN.
2. **Storage** — `df -h /data | tail -1` then `dumpsys diskstats | head -4`
   - /data usage ≥ 90% → FAIL. ≥ 80% → WARN.
3. **Memory** — `dumpsys meminfo | head -8` (read the Total RAM / Free RAM lines)
   - Free RAM < 15% of Total → WARN.
4. **CPU top consumers** — `top -n 1 | head -14`
   - If this toybox top prints nothing useful, retry with `top | head -14` or `ps -A | head -12` and report which variant worked.
5. **Network** — `ip route | head -4` then `dumpsys connectivity 2>/dev/null | grep -E "Active default network|NetworkAgentInfo" | head -4`
   - No default route → FAIL.
6. **System identity** — `getprop ro.product.model; getprop ro.build.version.release; getprop ro.build.version.security_patch; uptime`
7. **Screen sanity (optional)** — one `screen_read` call; report the focused app/activity. If screen tools are unavailable on this device, say so honestly and skip.

## Report format
```
[android-system-doctor]
1. Battery:  PASS — 92%, 28.4°C
2. Storage:  WARN — /data 83% used
3. Memory:   PASS — free 2.1G / 7.4G
...
Overall: HEALTHY / NEEDS ATTENTION — X pass, Y warn, Z fail
```

## Verify
A verdict without real output behind every line is INVALID — re-run the missing check instead of guessing. Never claim a check passed without having seen its command output in this session.
