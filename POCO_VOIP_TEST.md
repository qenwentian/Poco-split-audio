# POCO Audio Router — Discord/YouTube experiment

This is an **unverified experimental fork** of [SplitAudio](https://github.com/monarchsofchaos004-gif/splitaudio) (MIT license). It aims to route Discord voice output to the phone speaker while keeping YouTube on Bluetooth and retaining the phone microphone on HyperOS.

## What changed

- Custom mode has an opt-in **Route during Discord / VoIP calls (test)** switch, OFF by default. The toggle writes a shell-owned flag; the running audio policy detects it within the next polling cycle.
- When enabled, Custom mode no longer automatically pauses Bluetooth media or unregisters the per-app policy while Android reports `MODE_IN_COMMUNICATION`.
- `MODE_IN_CALL`, ringtone modes, and unknown non-normal modes still disable routing. Simple mode retains the upstream behavior.
- The free Shizuku build has a different app ID and name, with upstream update checking and report prompts suppressed.

## First phone test

1. Install/start [Shizuku](https://shizuku.rikka.app/guide/setup/) through Wireless debugging, and grant POCO Audio Router access.
2. Open **Custom** mode and set **YouTube** (`com.google.android.youtube`) to **Bluetooth**. Leave **Discord** (`com.discord`) on **Default** so its own speaker control remains in charge of the call. If that fails, try assigning Discord to **Phone speaker** as a second test.
3. Enable **Route during Discord / VoIP calls (test)** and acknowledge the warning.
4. Start YouTube media on the connected Bluetooth headphones; join a Discord voice channel and select the **Phone speaker** output in Discord.
5. Verify separately: (a) YouTube audio in Bluetooth, (b) Discord voices on the speaker, (c) other call participants hear your phone microphone, (d) leaving VC restores ordinary audio.

**If any sound goes silent or to the wrong device, disable the experiment and tap Stop split.** The switch cannot force the operating system to respect the route; Android/HyperOS may still override it during a voice call. Do not use the experiment during an emergency or important call. Speaker playback can leak into the built-in microphone, causing echo for other participants.

## Build

Requires JDK 17 and Android SDK 36. From the repo root:

```sh
sh route/build_dex.sh
./gradlew assembleFdroidDebug
```

The APK is produced at `app/build/outputs/apk/fdroid/debug/app-fdroid-debug.apk`. Build the new DEX before the APK; otherwise the APK may package the original helper and ignore the new call-mode behavior. Do **not** run the original SplitAudio app simultaneously: both use the same shell helper paths and local control port.

## Status

The Android DEX helper and signed **debug APK** built successfully with Android SDK 36 and JDK 17. The local test suite passed (184 passed, 1 skipped). A production release signature and physical-device validation are still pending. The source implements a real system-level routing experiment, but cannot guarantee Discord and YouTube stay separated until tested on the target POCO F6/HyperOS firmware.
