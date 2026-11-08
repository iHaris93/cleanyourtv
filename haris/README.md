# Haris's TV setup (branch `haris-tv`)

My personal build of CleanYourTV for a CANDY Android TV DVB (Haier, MediaTek m7332, Android TV 14). This branch
holds my local changes so a `git pull` from upstream never wipes them. `main` stays in sync with
Gathin23/cleanyourtv; the TV list I contributed lives on the `candy-android-tv-dvb` branch (upstream PR #3559).

The full setup, fixes and undo steps are in [RUNBOOK.md](RUNBOOK.md).

## What's changed from upstream

| File | Change |
| --- | --- |
| `themes/haris/config.json` | My theme: dock-glass with cinematic shelf; dock Live TV, YouTube, Aryzap, Tamasha, Tapmad, Imou; factory apps hidden; app library and boot-to-Live-TV on; stats and Wi-Fi colour off |
| `launcher/app/src/main/res/values/config.xml` | Built-in dock and hidden apps (the theme overrides these) |
| `launcher/app/src/main/java/com/gathin/tvhome/MainActivity.java` | Hides the cinematic shelf's "Last opened" line (one line: `long when = 0;`) |
| `addons/prayer-times/app/src/main/res/values/config.xml` | Rawalpindi, method 1 (Karachi), Hanafi Asr |
| `addons/prayer-times/app/src/main/java/.../MainActivity.java`, `PrayerAlertActivity.java` | 12-hour times (`to12()` helper) |
| `haris/candy-debloat.txt` | Everything turned off on my set, for `tools/check.py` after a firmware update |
| `examples/candy-android-tv-dvb/` | The list submitted upstream |

## Common commands (from the repo folder, USB debugging on)

    TV_HOST=<tv ip> tools/theme.sh themes/haris            # push theme changes
    TV_HOST=<tv ip> launcher/build-and-install.sh          # rebuild and update the launcher
    TV_HOST=<tv ip> tools/venv/bin/python -I tools/check.py haris/candy-debloat.txt com.gathin.tvhome
    tools/venv/bin/python -I tools/adb-closed.py <tv ip>   # after closing ADB: must say closed

## Getting upstream updates

    git fetch upstream && git merge upstream/main     # on haris-tv; resolve the few patched lines if they conflict
