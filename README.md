# Continue On Workaround for the Googlebook

Google's Continue On feature lets a Googlebook pick up apps from a phone, but only from phones that run Android
17 or newer. This repository documents the workaround I use to bring my older Android tablet and phone onto the
Googlebook's screen anyway. It uses Scrcpy for Android, an open-source scrcpy client that runs on GooglebookOS
itself. This is documentation only: there is no code to install, just the exact steps, settings and lessons from
my setup.

## What it does

This workaround brings my Android devices onto the Googlebook's screen when Google's Continue On feature is not
available for them. It uses [Scrcpy for Android](https://github.com/Miuzarte/ScrcpyForAndroid), an unofficial,
open-source scrcpy client by Miuzarte, installed on GooglebookOS itself. It streams a device's screen and sound
into a window on the Googlebook, and the Googlebook's trackpad, mouse and keyboard control the device directly.

Nothing is installed on the phone or tablet as an app, and no root is needed. Like the desktop version of
scrcpy, the app copies a small server file into the device's temporary folder over Wireless debugging and runs it
only while it streams.

I use it with my Lenovo Tab P12 (Android 15) and my Samsung Galaxy S23 Ultra (Android 16).

## The problem

Continue On is part of Google's Better Together features. It lets a Googlebook pick up an app you are using on
your phone. According to Google's help page
[Use cross-device services on your Googlebook](https://support.google.com/googlebook/answer/16945063), the
cross-device features need:

- a Googlebook running the latest GooglebookOS version,
- a compatible Android phone running Android 17 or newer,
- Bluetooth and Wi-Fi turned on, on both devices,
- both devices on the same Wi-Fi network or personal hotspot,
- both devices signed in to the same Google Account.

The feature also launched for Pixel phones only. Google said support for Samsung phones would follow in the
following weeks, as reported by
[9to5Google](https://9to5google.com/2026/10/06/googlebook-android-sync-features-samsung-delay/) and
[Digital Trends](https://www.digitaltrends.com/computing/some-googlebook-features-are-off-limits-if-your-phone-is-stuck-below-android-17/)
in October 2026. My Lenovo Tab P12 runs Android 15 and my Galaxy S23 Ultra runs Android 16, so both devices are
left out. The requirement depends on the phone's own system, so nothing on the Googlebook can switch it on.

Setting up a replacement had its own obstacles:

1. **Android 17's new local network permission.** GooglebookOS is based on Android 17, which adds a runtime
   permission for reaching devices on the local network, `android.permission.ACCESS_LOCAL_NETWORK`. After
   installation it was not granted to the app (`granted=false`), and the app needs it to reach the devices over
   Wi-Fi.
2. **Background and battery limits.** The app's own FAQ warns that the adb connection drops when the app goes to
   the background unless background use is unrestricted.
3. **Discovery failed.** The app looks for devices with mDNS, the way desktop adb does. On my network it only ever
   found the Googlebook itself and never the phone or the tablet, so I had to add each device by hand, by the IP
   address and port shown on that device's Wireless debugging screen.
4. **A misleading entry in the device list.** The app also lists USB devices, and it showed an entry called
   "Google FPMCU". That is the Googlebook's own fingerprint sensor, not an Android device. Trying to connect to
   it only produced handshake timeouts.
5. **The tablet refused the first connection.** It rejected the app during the secure handshake
   (`javax.net.ssl.SSLHandshakeException: connection closed`), because it did not trust the app's key, and it had
   to be paired again.
6. **The first stream lagged.** Every setting was at its default, which means no resolution limit, so the tablet
   was encoding its full 2944 by 1840 resolution and sending it over Wi-Fi. The Wi-Fi round trip between the
   Googlebook and the tablet also averaged about 32 milliseconds, with spikes above 100 milliseconds.

## How I solved it

1. **I picked an app that runs on GooglebookOS.** Scrcpy for Android is an Android app with an x86_64 build, so it
   runs directly on the Googlebook. It is licensed under the Apache License 2.0. Its author states that it is
   based on scrcpy but is not an official version and is not affiliated with scrcpy's authors. I tested version
   0.7.0, released on October 5, 2026.
2. **I checked the download before installing it.** I compared the APK's SHA-256 checksum with the digest that
   GitHub publishes for the release file, and they matched. I also verified the APK's signature with `apksigner`.
   The signer is `CN=Miuzarte`.
3. **I installed it for my Android user with adb** from the Linux Terminal.
4. **I granted the permissions it needs.** I granted `ACCESS_LOCAL_NETWORK` and `POST_NOTIFICATIONS`, allowed it to
   run in the background, and added it to the battery optimization whitelist, so the connection does not drop
   when the app is not in front.
5. **I paired each device once over Wireless debugging** with a pairing code, and added each device by the IP
   address and port shown on its Wireless debugging screen. The Galaxy S23 Ultra connected straight away. The Tab
   P12 needed to be paired again after the handshake error, and then it connected too.
6. **I removed the lag.** In the app's scrcpy parameters, I set the maximum resolution to 1920 pixels and the
   maximum frame rate to 60 frames per second. The app's log then showed the stream at 1920 by 1200 pixels and 60
   frames per second, with the Googlebook decoding the video in hardware. The stream became much smoother.

## Requirements

- A Lenovo Googlebook 15. I tested on GooglebookOS build `CL3B.260622.271.R1` (Android 17).
- Scrcpy for Android. I tested version 0.7.0, the x86_64 build (`ScrcpyForAndroid-v0.7.0-app-x86_64-release.apk`),
  package name `io.github.miuzarte.scrcpyforandroid`.
- The Linux Terminal with `adb` connected to the Googlebook over Wireless debugging, to install the app and grant
  its permissions. The shared
  [Getting Started guide](https://github.com/iamflying29-sketch/googlebook-projects/blob/main/GETTING-STARTED.md)
  explains how to set this up and how to find your Android user ID.
- `curl`, `sha256sum` and `python3`, which the Linux Terminal already has, and optionally `apksigner`
  (`sudo apt install apksigner`) to check the signature.
- Android phones or tablets with Developer options and Wireless debugging (Android 11 or newer), on the same
  Wi-Fi network as the Googlebook. The app's audio forwarding also needs Android 11 or newer. I used a Lenovo Tab
  P12 with Android 15 and a Samsung Galaxy S23 Ultra with Android 16.

## Installation

1. **Connect adb to the Googlebook** from the Linux Terminal, as described in the
   [Getting Started guide](https://github.com/iamflying29-sketch/googlebook-projects/blob/main/GETTING-STARTED.md).

2. **Download the x86_64 build** from the project's
   [releases page](https://github.com/Miuzarte/ScrcpyForAndroid/releases). For the version I tested:

   ```
   mkdir -p ~/scrcpy-for-android
   cd ~/scrcpy-for-android
   curl -LO https://github.com/Miuzarte/ScrcpyForAndroid/releases/download/v0.7.0/ScrcpyForAndroid-v0.7.0-app-x86_64-release.apk
   ```

   Use the `x86_64` file. The Googlebook has an Intel processor.

3. **Check the SHA-256 checksum against the digest that GitHub publishes for the release.**

   ```
   sha256sum ScrcpyForAndroid-v0.7.0-app-x86_64-release.apk
   curl -s https://api.github.com/repos/Miuzarte/ScrcpyForAndroid/releases/tags/v0.7.0 \
     | python3 -I -c 'import json,sys; [print(a.get("digest"), a["name"]) for a in json.load(sys.stdin)["assets"]]'
   ```

   The checksum of your file must match the `sha256:` digest listed for the x86_64 file. For version 0.7.0 the
   checksum was `ade2261dcb7da302be050d4ec3652df875b93d033e04b40872b0472a9636f7bf`. If they do not match, do not
   install the file.

4. **Check the signature (recommended).**

   ```
   apksigner verify --print-certs -v ScrcpyForAndroid-v0.7.0-app-x86_64-release.apk
   ```

   The output should say `Verifies` and show `Signer #1 certificate DN: CN=Miuzarte`. For version 0.7.0 the
   signer's certificate SHA-256 digest was
   `01aed2b0e870d3b5b1b5584dc3548fb2f4fa6812a04a7cb8ff8e0999dd6a0717`. Future versions from the same author
   should show the same certificate.

5. **Install the app for your Android user.**

   ```
   adb install --user current ScrcpyForAndroid-v0.7.0-app-x86_64-release.apk
   adb shell pm list packages --user current | grep scrcpy
   ```

   The second command should print `package:io.github.miuzarte.scrcpyforandroid`.

6. **Grant the permissions and allow background use.**

   ```
   U=$(adb shell am get-current-user | tr -d '\r')
   P=io.github.miuzarte.scrcpyforandroid
   adb shell pm grant --user "$U" $P android.permission.ACCESS_LOCAL_NETWORK
   adb shell pm grant --user "$U" $P android.permission.POST_NOTIFICATIONS
   adb shell cmd appops set --user "$U" $P RUN_ANY_IN_BACKGROUND allow
   adb shell cmd appops set --user "$U" $P RUN_IN_BACKGROUND allow
   adb shell dumpsys deviceidle whitelist +$P
   adb shell dumpsys package $P | grep -E 'ACCESS_LOCAL_NETWORK: granted|POST_NOTIFICATIONS: granted'
   ```

   The last command should show `granted=true` for both permissions.

7. **Turn on Wireless debugging on each phone or tablet.** Turn on Developer options by tapping **Build number**
   seven times. On my Galaxy S23 Ultra it is under Settings > About phone > Software information, and on my Tab P12
   under Settings > About tablet. Then open Settings > Developer options > **Wireless debugging** and turn it on.
   The device must be on the same Wi-Fi network as the Googlebook.

8. **Pair each device once.** In Scrcpy for Android on the Googlebook, open the **Devices** tab and, under
   **Wireless pairing**, tap **Pair device using pairing code**. On the phone or tablet, in the Wireless debugging
   screen, tap **Pair device with pairing code**. Enter the IP address, the pairing port and the six-digit code
   exactly as the device shows them. The pairing port is different from the connection port in the next step.

9. **Add each device by its address.** On the device's main Wireless debugging screen, read the address under
   **IP address & Port**. In the app, type that address into the **Connect / Add device** field, for example
   `192.0.2.10:37000`, and tap **Add device**. Then tap **Connect** on the device's row. Do not scan your network
   for ports. The correct port is always shown on the device's Wireless debugging screen.

   > **Warning: never tap Connect on the "Google FPMCU" entry.** It is the Googlebook's own fingerprint sensor,
   > which the app lists as a USB device. It is not an Android device, and there is no reason for the app to talk
   > to it.

10. **Limit the resolution and frame rate.** In the app, open **Devices** > **More parameters**
    (All scrcpy parameters). Tap the title of **Max resolution**, type `1920` and tap **OK**. Tap the title of
    **Max framerate**, type `60` and tap **OK**. Tapping a slider's title is how the app lets you type an exact
    value. These settings are in the **Global** configuration, so they apply to every device.

11. **Start the stream.** With a device connected, start scrcpy with the app's **Start** button. The device's
    screen and sound appear in the app's window, and you control the device with the Googlebook's trackpad, mouse
    and keyboard. A device's settings, which open when you long-press its row, also include the option
    **Start scrcpy immediately after connecting**.

## Using it

- **Each time,** make sure Wireless debugging is on on the device, open Scrcpy for Android, tap **Connect** on the
  device's row and start the stream.
- **One device at a time.** The app connects to one device at a time in a single window. To switch, tap
  **Disconnect** on the connected device and then **Connect** on the other one. Both stay in the saved list.
- **After a device restarts,** its Wireless debugging port changes, and Wireless debugging may be off. Turn it on
  again, read the new address on the device's Wireless debugging screen, and update the saved entry: long-press
  the row to edit it, or delete it there and add the device again. You do not need to pair again unless the device
  has forgotten the pairing.
- **When you are done,** stop the stream and disconnect in the app. Turning off Wireless debugging on the device is
  a good habit.

## Updating, and what happens after app or GooglebookOS updates

- **New versions of the app.** The app was installed with adb, not from the Play Store, so it does not update
  itself. To update, download the new x86_64 file from the releases page, repeat the checksum and signature checks
  (the signer certificate should stay the same), and install it over the old one:

  ```
  adb install -r --user current ScrcpyForAndroid-<VERSION>-app-x86_64-release.apk
  ```

  Then run the last command of installation step 6 again to confirm that both permissions are still granted.
  I have only tested version 0.7.0.
- **GooglebookOS updates.** Check the permissions in the same way after a GooglebookOS update. The local network
  permission is new in Android 17, so its behavior could change.
- **Phone and tablet updates.** When a device gets Android 17 and Google supports it, the official Continue On
  should work with it, and this workaround is no longer needed for that device.

## Uninstalling

On the Googlebook, remove the battery whitelist entry and uninstall the app (you can also uninstall it like any
other app):

```
U=$(adb shell am get-current-user | tr -d '\r')
adb shell dumpsys deviceidle whitelist -io.github.miuzarte.scrcpyforandroid
adb shell pm uninstall --user "$U" io.github.miuzarte.scrcpyforandroid
```

On each phone or tablet, open Settings > Developer options > Wireless debugging, remove the Googlebook's entry
under **Paired devices**, and turn Wireless debugging off.

## Troubleshooting

- **The app does not find your devices by itself.** That is what happened on my network. Add each device by the IP
  address and port shown on its Wireless debugging screen (installation step 9). Do not use network scanning or
  port scanning to look for the port.
- **The app cannot reach any device.** Check that `ACCESS_LOCAL_NETWORK` is granted (installation step 6). Without
  it the app cannot talk to devices on your Wi-Fi network.
- **The connection fails with "ADB connection failed" and an SSL handshake error.** The device does not trust the
  app's key. Pair it again with a pairing code (installation step 8) and connect again.
- **A saved device stopped connecting.** The device probably restarted, so its port changed or Wireless debugging
  is off. See [Using it](#using-it).
- **The connection drops when the app is in the background.** Repeat the background and battery commands in
  installation step 6.
- **The stream lags.** Make sure Max resolution and Max framerate are set (installation step 10). You can try a
  lower resolution such as 1600 or 1280, which reduces delay but softens the picture, and turn off audio
  forwarding if you do not need sound. Short delay spikes come from the Wi-Fi network and cannot be removed
  completely.
- **Some apps show a black screen.** Those apps block screen capture. This is expected and cannot be changed
  without root on the device.
- **The device list shows "Google FPMCU".** Leave it alone. It is the Googlebook's fingerprint sensor, and trying
  to connect to it only ends in a handshake timeout.

## Safety and privacy notes

- Wireless debugging gives a paired computer or app full adb control of the phone or tablet. Pair devices only on
  a network you trust, keep the pairing to your own Googlebook, and turn Wireless debugging off when you do not
  use it.
- Scrcpy for Android receives the local network and notification permissions, background use, and an exemption
  from battery optimization on the Googlebook. It needs no root on either side.
- The app is open source under the Apache License 2.0, but it is a third-party app installed from outside the
  Play Store. Check every download against the published digest and the signer certificate.
- I did not change anything on the phone or tablet except turning on Developer options and Wireless debugging.

## Limitations and known issues

- The app mirrors one device at a time, in a single window.
- It streams the device. It does not hand an app over to the Googlebook the way Continue On does, and the app keeps
  running on the phone or tablet.
- After a device restarts, its Wireless debugging port changes, so the saved entry needs the new port.
- Apps that block screen capture show a black screen.
- Automatic discovery did not work on my network, so devices have to be added by hand.
- The app also offers clipboard sync and file transfer. I did not test them. I also only used Wi-Fi and did not
  test a USB connection.
- I tested only version 0.7.0 of the app, on one GooglebookOS build, with two devices.

## Files in this repository

| File | Purpose |
|---|---|
| `README.md` | This document: the full workaround, step by step. |
| `LICENSE` | The MIT License for this documentation. Scrcpy for Android has its own license. |
| `.gitignore` | Keeps downloaded APKs, keys and logs out of the repository. |

## Related projects

- [Googlebook Projects](https://github.com/iamflying29-sketch/googlebook-projects): the overview of all my
  Googlebook projects and the shared Getting Started guide.
- [Waydroid on the Googlebook](https://github.com/iamflying29-sketch/googlebook-waydroid): another way I run
  Android apps on the Googlebook, inside the Linux Terminal.
- [Scrcpy for Android](https://github.com/Miuzarte/ScrcpyForAndroid) by Miuzarte and
  [scrcpy](https://github.com/Genymobile/scrcpy) by Genymobile: the projects this workaround relies on. I am not
  involved with either.

## Disclaimer and license

This is an independent, unofficial project. It is not affiliated with, endorsed by, or sponsored by Google,
Lenovo or any other company named here. All product names belong to their owners. Released under the MIT
License. Copyright (c) 2026 Thomas Liviakis.
