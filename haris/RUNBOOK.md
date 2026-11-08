# Runbook: CANDY Android TV DVB

The debloat and launcher setup for one CANDY Android TV DVB (Haier, MediaTek m7332, Android TV 14), done
over ADB from a Mac in November 2026. Every change on this list can be undone.

Not in this file on purpose: serial numbers, IMEI, `adb devices` names (they contain the serial), Wi-Fi
names, account details and the `backup/` folder. Replace `<tv ip>` with the TV's address on your home network.

## Results

- The Anoki ACR tracker is off, along with Haier's auto-installer and content apps and Google's
  recommendation and ad rows.
- With Haier's app manager (`com.cvte.haier.tv.appmanager`) disabled, nothing comes back on at boot.
- The home screen is the CleanYourTV launcher with the `haris` theme, and the TV boots straight to Live TV.
- The Prayer Times add-on is set to Rawalpindi with 12-hour times.
- A phone can be used as a remote through any Android TV remote app.
- Free RAM (MemAvailable) went from about 450 MB to about 485 MB, out of 1.4 GB.

## Device

| Property | Value |
| --- | --- |
| Model | CANDY Android TV DVB, made by Haier on a CVTE board |
| Chip | MediaTek m7332 |
| Android | Android TV 14, firmware UKNV.260408.001 |
| RAM / storage | 1.4 GB / 4.2 GB `/data` |
| Stock launcher | `com.google.android.tvlauncher` (now disabled) |
| Live TV app | `com.mediatek.wwtv.tvcenter` |
| Closest repo list | `examples/haier-android-tv-dvb2k` (Android 11, a different board) |

## Mac setup

| Tool | How |
| --- | --- |
| adb 37+ | `brew install android-platform-tools` |
| Java 17 | `brew install openjdk@17`, then `export JAVA_HOME=/opt/homebrew/opt/openjdk@17` in each Terminal window (Homebrew keeps it off the PATH) |
| Android SDK | `brew install --cask android-commandlinetools`, `sdkmanager "platforms;android-34" "build-tools;34.0.0"`, `yes \| sdkmanager --licenses` |
| Repo helper | `python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt` |

The repo's tools are run with a small zsh function (zsh doesn't split variables into words):

    export TV_HOST=<tv ip>
    tv() { tools/venv/bin/python -I tools/tv.py "$@"; }

macOS also blocks adb from reaching devices on the home network until the terminal is allowed:
System Settings → Privacy & Security → **Local Network** → turn on the terminal app.

## Connecting

There are two routes in:

- **Wireless debugging (encrypted).** Pair with `adb pair <ip>:<pairing port> <code>` while the TV's pairing
  dialog is open, then connect on the main port shown on the TV. `adb mdns services` lists the live ports.
- **Port 5555 (unencrypted).** Open while USB debugging is on. Run `adb connect <tv ip>:5555` and
  accept the prompt on the TV. The repo's `tools/tv.py` only uses this route.

| Symptom | Fix |
| --- | --- |
| `protocol fault` on `adb pair` | Often it paired anyway, so check `adb devices` before retrying. Also check the Local Network permission. |
| `No route to host` while ping works | `adb kill-server`, then connect again |
| `Network is unreachable` | Check the Mac is on the same home network, then `adb kill-server` |
| The same TV listed twice, one `offline` | `adb disconnect` the IP entry |
| `failed to connect` after a reboot | The TV forgot the pairing. Pair again, or use port 5555. |
| `unauthorized` on 5555 | Accept the "Allow USB debugging?" prompt on the TV |
| `adb: unknown command pm` | The device variable was empty; reconnect |
| `tools/tv.py --push` hangs on an APK | Use `adb -s <tv ip>:5555 install -r <apk>` |

## Package changes

`pm disable-user --user 0` is undone with `pm enable`. `pm uninstall --user 0` is undone with
`cmd package install-existing`. The full list is in `haris/candy-debloat.txt`; the upstream-format list is in
`examples/candy-android-tv-dvb/debloat.txt`.

- **Disabled:** Anoki ACR, retail demo, Haier auto-install, Haier app manager, Haier Apps, Evo, smart-device
  service, TV Housekeeper, content feed, partner setup and the home customizer, Google feedback, Play Games,
  the bundled browser, plus the owner's choices (Prime Video, YouTube Music, Google TTS, Miracast) and the stock
  launcher.
- **Removed for the user** (they came back at boot while the app manager was on): FAST channels,
  Recommendations, calendar sync, the Haier content launcher, Netflix.
- **Enabled again:** `com.example.bt_pair_tips` (part of Bluetooth remote pairing, not a tips popup).

**Never touch:**

| Group | Packages |
| --- | --- |
| Live TV | `com.mediatek.wwtv.tvcenter`, `com.mediatek.tvinput`, `com.mediatek.tv.service`, `com.android.providers.tv` |
| Remote | `com.mediatek.autopair`, `com.example.bt_pair_tips`, `com.haier.tv.hmagicboard` (hot keys), `com.cvte.tv.hkcfac`, `com.google.android.tv.remote.service` |
| Factory | `com.cvte.tv.fac.*`, `com.cvte.tv.at`, `com.at.factory.*`, `mediatek.factorymenu.ui` |
| Core | Play services, Play Store, WebView, the setup wizard, system UI, settings, the keyboard, Dolby |

**Never open a factory app**, not even to look. On this board family, that's reported to switch Play
Services, Netflix and Chromecast off at the next boot.

## Launcher and theme

The launcher was installed with `TV_HOST=<tv ip> launcher/build-and-install.sh`. The script started the
new launcher before it disabled the stock one, then set it as Home. A reboot was checked afterwards.

The theme is `themes/haris`, based on dock-glass:

- It has its own dock and hidden apps. Every factory app is hidden.
- `cinematic_home`, `app_library` and `boot_to_live_tv` are on.
- `show_stats` and `wifi_status_color` are off.

Push the theme with `TV_HOST=<tv ip> tools/theme.sh themes/haris`. Any stock theme replaces the dock, so
make changes in `themes/haris` only.

There are two local code patches:

- `launcher/.../MainActivity.java`: the cinematic shelf's "Last opened" line is hidden (`long when = 0;`).
- `addons/prayer-times`: Rawalpindi, the Karachi method and Hanafi Asr are set, and times show in 12-hour
  format through a `to12()` helper.

## Undo

| Problem | Fix |
| --- | --- |
| Home screen broken | `adb -s <tv ip>:5555 shell pm enable com.google.android.tvlauncher`, then pick it as Home |
| Back to the stock setup | `TV_HOST=<tv ip> tools/restore.sh examples/candy-android-tv-dvb/debloat.txt com.google.android.tvlauncher` |
| A remote button stopped working | `pm enable com.haier.tv.hmagicboard` and `pm enable com.cvte.tv.hkcfac` |
| A removed app is needed | `cmd package install-existing <package>` |
| Theme problems | `tools/theme.sh --reset`, then push `themes/haris` again |

## After a firmware update

    TV_HOST=<tv ip> tools/venv/bin/python -I tools/check.py haris/candy-debloat.txt com.gathin.tvhome

Add `--fix` after `check.py` to apply the list again and put the launcher back on Home.

## Closing ADB (every session)

1. Run `adb disconnect`.
2. On the TV, turn off Wireless debugging and USB debugging.
3. Restart the TV.
4. Run `tools/venv/bin/python -I tools/adb-closed.py <tv ip>`. It must say **closed**.

**Don't:** factory reset (it brings everything back), open factory menus, push stock themes, or `git pull`
without merging onto this branch.

## Lessons

- A debloat only sticks once you find what turns apps back on. Here, that was Haier's app manager.
- Check a sibling model's tested list before disabling anything with a vague name. A name like `bt_pair_tips`
  doesn't say what the package actually does.
- On a 1.4 GB TV, debloating frees little RAM. The real gains are privacy, no ads and a cleaner home screen.
