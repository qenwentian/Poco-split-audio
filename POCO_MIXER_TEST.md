# POCO Audio Router: persistent App Volume Mixer

## Goal

Use a Windows-style volume mixer on HyperOS. A game saved at **0%** should remain
muted when launched later, while YouTube at **100%** plays through the normal
Android media volume. Output splitting and VoIP routing are separate and do not
have to be enabled for mixing.

## Setup

1. Start Shizuku and grant the **POCO Audio Router** app permission.
2. Open **App Volume Mixer** from the main screen.
3. Set the game to **0%**. Leave YouTube at **100%** (no adjustment).
4. Confirm the status reports **Mixer running**. Close the mixer screen.
5. Start YouTube, then open the game. YouTube should remain audible while
   the game is muted, subject to Android's audio-focus behavior.
6. Reopen the mixer to unmute the game or change its percentage.

Saved per-app settings survive closing the app and rebooting, but the shell
mixer must be restarted after a reboot (Shizuku also usually needs restarting).
This build checks the mixer on service startup, Shizuku reconnect, playback
callbacks, and every 60 seconds as a fallback. Once running, the helper polls
each second to find newly created game players and enforce their saved gains.

**100% means leave the app alone**, not boost it to full system volume. A game
at 0% may still be audible if its audio uses a player interface that the
Android shell cannot control on the installed HyperOS build; some OEM audio
frameworks reject the hidden PlayerProxy method. The helper now logs the first
such error for diagnosis. No root is required for this test build.

**Important:** Audio focus is separate from mixer gain. Muting a game does not
guarantee that launching it will let YouTube continue playing if the game
requests exclusive audio focus. Multi-app playback may require additional
settings and may not be supported consistently by HyperOS.

## Build

```sh
sh route/build_dex.sh
gradle testFdroidDebugUnitTest testPlayDebugUnitTest
gradle assembleFdroidDebug
```

Use the **fdroid debug** APK (Shizuku). The GitHub Actions release-unsigned
APK cannot be installed until properly signed. Debug builds on different
machines may also have incompatible certificates, requiring uninstall of the
previous test app (losing its saved settings).

## Validation

- Game 0% + YouTube default 100%, while both players are running
- Start game after setting 0%, with mixer screen closed
- Change game to 35% during playback, then return to 0%
- Set everything to 100%, confirm the helper stops and ordinary audio returns
- Stop/restart Shizuku and check that the saved mixer settings recover
- Reboot, restart Shizuku, and check for automatic recovery
- Inspect logs if a particular game still plays audio despite 0%
