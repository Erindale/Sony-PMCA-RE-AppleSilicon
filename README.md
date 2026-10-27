# Sony PMCA-RE for Apple Silicon Macs

> **This is a fork** of the excellent work done by [ma1co](https://github.com/ma1co/Sony-PMCA-RE/). I take no credit for their work. The goal of this fork is a single, self-contained macOS app for Apple Silicon (M1/M2/M3/M4) Macs that a non-technical person can download and open like any other app, with no Homebrew, Python or drivers to install. The coding work, including the build setup, was carried out with Claude Code. The macOS changes only take effect on macOS; Windows and Linux keep upstream's behaviour (see [Windows, Linux and older Macs](#windows-linux-and-older-macs)). These have not been tested on Windows or Linux hardware. You have been forewarned.
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

## If something goes wrong

- **"Operation timed out. Please run this command again when your camera has connected."** The camera switched into app-install mode but macOS took too long to notice. Wait until the camera's screen shows *Connecting via USB...*, then click **Install selected app** again. The second attempt picks up where the first left off.
- **The camera shows "Connecting via USB..."** That is the normal screen while an app is being installed. Leave it alone until the log in pmca-gui says *Task completed successfully*.
- **No devices found.** Check that the camera is switched on, that USB Connection is set to MTP, and that the cable carries data. Many cheap micro-USB cables only charge. Plug the camera straight into the Mac rather than through a hub if in doubt.
- **"Cannot access the camera"** Quit Photos, Image Capture, Dropbox and Google Drive, unplug the camera, plug it back in and try again.

## What works on Apple Silicon

| Feature | Status |
|---|---|
| Camera info (MTP mode) | Works |
| Install app (MTP mode) | Works |
| Tweaks via *updater mode* | Needs Sony's mass storage kernel driver, which cannot be loaded on Apple Silicon Macs without disabling security features. Not supported by this fork. |
| Update firmware | Same limitation as above. |
| Tweaks via *service mode* | Untested. |

Once *OpenMemories: Tweak* is installed on the camera, settings can be tweaked directly on the camera, which avoids the updater mode limitation.

## Windows, Linux and older Macs

The rest of this section is upstream's documentation, unchanged. This fork publishes macOS releases only; Windows and older macOS binaries are available from [upstream's releases](https://github.com/ma1co/Sony-PMCA-RE/releases/latest).

There are two binaries:

* **pmca-console**: The main command line application
* **pmca-gui**: A graphical user interface with a limited set of features

### Windows
The application should work fine on Windows using the operating system's mass storage and MTP USB drivers.

Download the [latest stable release](https://github.com/ma1co/Sony-PMCA-RE/releases/latest) or the newest [development build](https://ci.appveyor.com/project/ma1co/sony-pmca-re/build/artifacts).

### macOS
macOS binaries are also distributed, but less tested than the Windows equivalents. Getting the USB drivers to work may require some fiddling. To communicate with cameras in mass storage mode, Sony's [Camera Driver](https://support.d-imaging.sony.co.jp/mac/driver/11/ja/) has to be installed. Make sure to close all applications which could access USB drivers, including Photos, Dropbox and Google Drive.

The latest release binaries can be found in the [release section](https://github.com/ma1co/Sony-PMCA-RE/releases/latest).

### Linux
The application uses Python 3 and should work fine on Linux using libusb drivers.

Clone or download this repository, then run the following commands:
```bash
pip install -r requirements.txt  # to install the dependencies
./pmca-console.py  # for the command line application
./pmca-gui.py  # for the gui application
```

### Usage
There are three main modes of interfacing with a camera:

#### App Installer
If the camera supports *PlayMemories Camera Apps (PMCA)*, it is possible to install custom Android apps using this tool. A list of supported cameras can be found [here](https://openmemories.readthedocs.io/devices.html).

It is recommended to install the [*OpenMemories: Tweak*](https://github.com/ma1co/OpenMemories-Tweak) app. This app allows to tweak settings and to start *telnet* and *adb* servers to execute code on the system.

Other apps are available. A list can be found [here](https://github.com/ma1co/OpenMemories-AppList).

There are two ways to install apps:
* **pmca-gui**: In the *Install app* tab, select an app from the list and click *Install selected app*.
* **pmca-console**: Run `pmca-console install -i` to interactively select an app to install.

#### Firmware Updater
Sony cameras can boot from a secondary partition for firmware updates. Using a custom firmware file, we can execute code in this mode. Note that the camera firmware itself remains untouched. The firmware update process is only used to execute custom code.

This mode does not require any special drivers, the operating system's mass storage USB driver is enough.

A list of supported camera models can be found [here](https://openmemories.readthedocs.io/devices.html). Devices based on the CXD90045 and CXD90057 architectures are not compatible, since their firmware is cryptographically signed.

There are two ways to use this:
* **pmca-gui**: In the *Tweaks* tab, click *Start tweaking (updater mode)*. You can then use the checkboxes to configure your camera's settings.
* **pmca-console**: Run `pmca-console updatershell`. There are commands available to dump the firmware, execute Linux commands, and to tweak settings.

Note that this requires rebooting the camera to firmware update mode. You will be guided through this process.

#### Service Mode
Sony cameras have a USB mode called *senser mode*, which is used during servicing for calibration and other things. It can also be used to execute code on the running system.

Service mode has the best camera compatibility, but requires custom USB drivers.

It is currently only supported in the command line application:
* **pmca-console**: Run `pmca-console serviceshell`. There are commands available to dump the firmware and to execute Linux commands.

##### Windows Drivers
To use service mode on Windows, custom drivers have to be installed using [Zadig](http://zadig.akeo.ie/):
* Make sure the camera is connected in mass storage mode.
* In Zadig, check *Options -> List All Devices*, select the camera, select *libusb-win32* and click *Replace Driver*.
* Run `pmca-console serviceshell` to make the camera switch modes.
* Once the camera has switched, repeat the above step to install a driver for service mode.
* You should now be able to use `pmca-console serviceshell`.

To be able to use the camera normally again, the libusb drivers have to be uninstalled in device manager.

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

Windows and Linux are built by [a separate workflow](.github/workflows/build.yml) as a check; their binaries are uploaded as workflow artifacts, not released. Every macOS-specific change is behind `sys.platform == 'darwin'`. Three fixes apply on every platform, and each behaves like upstream wherever upstream worked: the apk manifest parser (`pmca/apk/axml.py`, replacing `axmlparserpy`, which fails on UTF-8 string pools), raw strings for regular expressions (Python 3.12+ warns about the old escapes), and a workaround for a `tlslite-ng` crash in frozen builds.

## Is it safe?
This is an experiment in a very early stage. All information has been found through reverse engineering. Even though everything worked fine for our developers, it could cause harm to your hardware. If you break your camera, you get to keep both pieces. **We won't take any responsibility.**

## What about custom apps?
It is possible to develop custom Android apps for supported cameras. Keep in mind that they should be compatible with Android 2.3.7. Debug and release certificates are accepted by the camera. See [*PMCADemo*](https://github.com/ma1co/PMCADemo) for a demo app.

There are a few special Sony APIs which allow you to take advantage of the features of your camera. They can be used through [*OpenMemories: Framework*](https://github.com/ma1co/OpenMemories-Framework).

## Special thanks
Without the work done by the people at [nex-hack](http://www.personal-view.com/faqs/sony-hack/hack-development), this wouldn't have been possible. Thanks a lot!
