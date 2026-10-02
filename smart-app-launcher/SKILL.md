---
name: smart-app-launcher
description: Launch any installed app with one command, zero guessing. Resolves the REAL package id from pm list in this session (never from memory), launches via monkey (never am start with a remembered activity name), checks disabled state, then verifies the app is truly in the foreground before claiming success. Works with app names in Arabic, French, English or raw package ids.
version: 1.0.0
author: AMINODa
category: apps
tags: [android, apps, launch, monkey, package, tiktok]
allowed-tools: shell_command, screen_read
---

# Smart App Launcher

Open any installed app with ONE command — honestly. Resolve first, launch with monkey, verify before you claim. A ✓ on the launch command alone is NOT success.

## Execution rules
- NEVER take a package id from memory. Always resolve it from `pm list packages` output seen in THIS session.
- NEVER use `am start -n <pkg>/<Activity>` with a remembered activity class name. Activity names change between app versions (TikTok's old `MusicallyMainActivity` no longer exists → "Error type 3 does not exist"). Use monkey — it always targets the real launcher activity.
- ✓ means the command executed — it does NOT mean the app opened. Success is only the verification step passing.
- If `shell_command` fails because the service is not connected: STOP and tell the user to connect the aMiNo service first (drawer → Adb). Never fabricate outputs.
- If a step fails, record FAIL with the real error text and continue only where meaningful.

## Steps (run in this order)
1. **Resolve the package** — user says "افتح تيك توك" / "open tiktok" / "lance tiktok":
   - `pm list packages | grep -i <keyword>`
   - Multiple hits → list them with labels (`cmd package list packages | grep -i <keyword>`) and ask the user to pick. Never pick silently.
   - Zero hits → try the alias table below (re-grep each candidate id). Still zero → verdict "NOT INSTALLED" is VALID (grep exit=1 is real evidence). Do not call grant_permissions, do not use screen tools toward a non-existent app. Stop honestly.
2. **Disabled check** — `pm list packages -d | grep -w <pkg>`
   - If listed here the app is DISABLED, not missing: tell the user to enable it in Settings → Apps and stop. A disabled app will not launch.
3. **Launch (monkey only)** — `monkey -p <pkg> -c android.intent.category.LAUNCHER 1`
   - If it prints an error, retry once with `monkey --pct-syskeys 0 -p <pkg> -c android.intent.category.LAUNCHER 1`, then report the real error text. Never fall back to am start with a guessed activity.
4. **Verify truth** — wait 3 seconds, then:
   `dumpsys activity activities | grep -m1 topResumedActivity`
   fallback if empty: `dumpsys window 2>/dev/null | grep -m1 mCurrentFocus`
   - Output contains the resolved package → SUCCESS.
   - Output shows a different package → the app exists but did not come to front. Report honestly and do ONE `screen_read` to see what is blocking (permission dialog, crash, lock screen).
5. **Screen evidence (optional)** — one `screen_read` call; report the focused app and one line about what is visible.

## Alias seed table (hints only — still verify each with pm list before launching)
```
tiktok        → com.zhiliaoapp.musically
tiktok lite   → com.zhiliaoapp.lite
douyin        → com.ss.android.ugc.aweme
whatsapp      → com.whatsapp
instagram     → com.instagram.android
youtube       → com.google.android.youtube
youtube music → com.google.android.apps.youtube.music
telegram      → org.telegram.messenger
facebook      → com.facebook.katana
snapchat      → com.snapchat.android
twitter/x     → com.twitter.android
settings      → com.android.settings
```

## Report format
```
[smart-app-launcher]
1. Resolve:  "tiktok" → com.zhiliaoapp.musically  (from pm list output)
2. Enabled:  PASS
3. Launch:   monkey ok
4. Verify:   topResumedActivity=…com.zhiliaoapp.musically/…  → OPENED
Overall: SUCCESS / NOT INSTALLED / DISABLED / FAILED — <real reason>
```

## Verify
Never claim an app opened unless `topResumedActivity` (or `mCurrentFocus`) in this session's output names its package. Never imply a NOT INSTALLED app became usable. Every line of the report must cite real output seen in this session.
