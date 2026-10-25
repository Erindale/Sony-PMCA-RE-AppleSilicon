# Sony PMCA-RE for Apple Silicon Macs

> **This is a fork** of the excellent work done by [ma1co](https://github.com/ma1co/Sony-PMCA-RE/). I take no credit for their work. The goal of this fork is a single, self-contained macOS app for Apple Silicon (M1/M2/M3/M4) Macs that a non-technical person can download and open like any other app, with no Homebrew, Python or drivers to install. The coding work, including the build setup, was carried out with Claude Code. I cannot confirm whether the Windows and Linux parts of this repo have been affected by these changes. You have been forewarned.
>
> **I cannot take any responsibility for any damages you may incur from using this software.** I just did this to install [Recipe Lab](https://github.com/voxivoid/recipe-lab-sony-pmca) on a Sony A6000.

This tool interfaces with Sony digital cameras through USB. It allows to tweak settings, dump firmware, and in some cases install custom Android apps. A list of supported cameras can be found [here](https://openmemories.readthedocs.io/devices.html).

## Installing the app on your Mac

1. Download the latest `pmca-gui-*-macos.dmg` from the [releases page](https://github.com/Erindale/Sony-PMCA-RE-AppleSilicon/releases/latest).
2. Open the downloaded `.dmg` and drag **pmca-gui** into the **Applications** folder.
3. Open **pmca-gui** from your Applications folder.

### The first time you open it

The app is not signed with a paid Apple developer certificate, so macOS will refuse to open it the first time and show a message like *"pmca-gui" cannot be opened because Apple cannot check it for malicious software* or *"pmca-gui" is damaged*. This is normal for free, open-source apps. To allow it once:

1. Click **Done** (or **Cancel**) on that message.
2. Open **System Settings → Privacy & Security**.
3. Scroll down to the **Security** section. You will see *"pmca-gui" was blocked to protect your Mac*. Click **Open Anyway**.
4. Confirm with your password or Touch ID, then click **Open**.

macOS only asks this once. After that the app opens normally.

<details>
<summary>Terminal alternative (optional)</summary>

If you are comfortable with Terminal, this removes the download quarantine flag and has the same effect:

```bash
xattr -dr com.apple.quarantine /Applications/pmca-gui.app
```
</details>

## Installing an app on your camera

1. On the camera, set the USB connection mode to **MTP**. On most Sony cameras this is under *Menu → Setup → USB Connection → MTP*. Leave *USB LUN Setting* on *Multi*.
2. Quit **Photos**, **Image Capture**, **Dropbox**, **Google Drive** and anything else that reacts to a camera being plugged in. The app will try to free the camera on its own if macOS has grabbed it, but it is more reliable if those apps are closed.
3. Connect the camera to the Mac with a USB cable and switch the camera on.
4. In **pmca-gui**, open the **Camera info** tab and click **Get camera info**. Your camera's name should appear in the log at the bottom. If it does not, unplug the camera, wait a few seconds, and plug it in again.
5. Open the **Install app** tab, pick an app from the list (or choose an `.apk` file you downloaded), and click **Install selected app**. The camera's screen will show the installation progress. Do not disconnect until the log says *Task completed successfully*.

It is recommended to install [*OpenMemories: Tweak*](https://github.com/ma1co/OpenMemories-Tweak) first. Other apps are listed [here](https://github.com/ma1co/OpenMemories-AppList).

## What works on Apple Silicon

| Feature | Status |
|---|---|
| Camera info (MTP mode) | Works |
| Install app (MTP mode) | Works |
| Tweaks via *updater mode* | Needs Sony's mass storage kernel driver, which cannot be loaded on Apple Silicon Macs without disabling security features. Not supported by this fork. |
| Update firmware | Same limitation as above. |
| Tweaks via *service mode* | Untested. |

Once *OpenMemories: Tweak* is installed on the camera, settings can be tweaked directly on the camera, which avoids the updater mode limitation.

## For developers

The app is plain Python. To run it from source or build the app yourself:

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

```bash
.venv/bin/python pmca-gui.py
```

```bash
.venv/bin/python -m PyInstaller pmca-gui.spec
```

The build produces `dist/pmca-gui-<version>-macos.app` and a matching `.dmg`. Use a Python from [python.org](https://www.python.org/downloads/macos/) rather than Homebrew, because the python.org build includes the Tk GUI toolkit that gets bundled into the app. `libusb` is bundled automatically through the `libusb-package` dependency.

Releases are built by [GitHub Actions](.github/workflows/build-macos.yml) on Apple Silicon runners. Pushing a tag such as `v1.0.0` creates a release with the `.dmg` and the command line tool attached. If repository secrets for an Apple Developer ID certificate are present, the workflow signs and notarizes the app, which removes the *Open Anyway* step for users.

The command line tool `pmca-console` is also built and attached to each release. Run `pmca-console --help` for its commands, including `install`, `updatershell` and `serviceshell`.

## Is it safe?
This is an experiment in a very early stage. All information has been found through reverse engineering. Even though everything worked fine for our developers, it could cause harm to your hardware. If you break your camera, you get to keep both pieces. **We won't take any responsibility.**

## What about custom apps?
It is possible to develop custom Android apps for supported cameras. Keep in mind that they should be compatible with Android 2.3.7. Debug and release certificates are accepted by the camera. See [*PMCADemo*](https://github.com/ma1co/PMCADemo) for a demo app.

There are a few special Sony APIs which allow you to take advantage of the features of your camera. They can be used through [*OpenMemories: Framework*](https://github.com/ma1co/OpenMemories-Framework).

## Special thanks
Without the work done by the people at [nex-hack](http://www.personal-view.com/faqs/sony-hack/hack-development), this wouldn't have been possible. Thanks a lot!
