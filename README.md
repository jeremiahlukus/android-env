# 🚀 OneClick Android Emulator Setup

![macOS](https://img.shields.io/badge/macOS-000000?style=for-the-badge&logo=apple&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)

**OneClick Emulator Setup** is a lightning-fast, automated CLI tool designed to completely set up an Android Development environment—including the Android SDK, Command-Line Tools, Platform Tools, and an Android Virtual Device (AVD)—without the need to install the heavy 1GB+ Android Studio IDE.

---

## ✨ Features

- **No Android Studio Required:** Skips the massive IDE download and installs only the necessary command-line tools.
- **Always-Current SDK Tools:** Looks up the newest stable command-line tools build in Google's own SDK index at install time, instead of hardcoding one that goes stale. Fully overridable — see the **Configuration** section below.
- **Uses the modern Android CLI:** Installs SDK packages with the `android` CLI that ships with command-line tools rev 23+, passing `--no-metrics` to opt out of usage-data collection. Falls back to `sdkmanager` automatically on older tools.
- **Automated Licensing:** Accepts all required Google SDK licenses without prompting.
- **Verified Installs:** After installing, checks on disk that `adb`, the emulator, the platform and the system image all actually landed. Both `android sdk install` and `sdkmanager` exit `0` even when a package is not found, so exit codes alone are not trusted.
- **Smart OS Detection:** Automatically detects macOS, Windows (Git Bash), or Linux and downloads the optimal architecture tools and system images (e.g. `arm64-v8a` for Apple Silicon Macs, including the architecture-split macOS tools archives Google publishes from rev 22.0 onward).
- **Custom Naming & App Shortcuts (NEW):** Let's you give a custom name to your emulator, and generates a native Desktop App Shortcut!
  - **macOS:** Creates a native `.app` in your Applications folder, searchable via **Spotlight** & Launchpad!
  - **Windows:** Creates a `.bat` shortcut directly in your **Start Menu**!
  - **Linux:** Creates a `.desktop` file searchable in your App Launcher!
- **Modern Device Skins:** Automatically downloads and configures your emulator with modern device frames (Pixel 6 Pro, Pixel 6, Pixel 5) so it looks like a real phone.
- **Interactive Selection & Preview:** Allows developers to choose their preferred phone frame, and preview the frames in your default image viewer before installing!
- **Smart Re-installation Check:** Detects if you already have emulators installed and offers to safely skip the setup process.

---

## 🚀 Quick Start

Choose your Operating System below:

### 🍎 macOS & 🐧 Linux

From the directory containing `android-env`, run:

```bash
# 1. Make the script executable
chmod +x android-env

# 2. Run the setup
./android-env setup
```
*(Follow the on-screen prompts to select your modern Pixel device frame!)*

### 🪟 Windows (Git Bash)

Since this tool leverages bash scripts and command-line SDKs, Windows users should run this natively using **Git Bash** (which comes installed with Git for Windows).

1. Open **Git Bash**.
2. Run the same commands as macOS/Linux:
   ```bash
   chmod +x android-env
   ./android-env setup
   ```
*(Note: Do not run this in standard CMD or PowerShell. It requires Git Bash.)*

---

## 🎮 Usage Guide

Once you have run the setup command, you can launch your emulator in two ways:

### 1. The Magic Way (No Commands Needed! ✨)
Since the tool automatically creates a native Desktop App for you during setup, you can launch your emulator exactly like a normal app! Just search for the name you gave it:
- **macOS:** Open **Spotlight Search** (Cmd+Space) or Launchpad.
- **Windows:** Open your **Start Menu**.
- **Linux:** Open your **App Launcher**.

### 2. Using the CLI Tool
You can also use the script to manage your environment from the terminal:

- **Start the Emulator:** (Boots up the phone)
  ```bash
  ./android-env start
  ```

- **Check Environment Health:** (Verifies Java, the SDK, `sdkmanager`, `avdmanager`, the emulator and ADB)
  ```bash
  ./android-env doctor
  ```

- **List Installed Emulators:** (Shows all virtual devices you've created)
  ```bash
  ./android-env list
  ```

- **Inspect SDK Tooling:** (Shows the installed command-line tools revision, the newest available build, and every override)
  ```bash
  ./android-env tools
  ```

- **Show Version:**
  ```bash
  ./android-env version
  ```

---

## ⚙️ Configuration

Everything works with no configuration. These environment variables are there for
when you need to pin, upgrade, or work offline.

| Variable | Effect |
| --- | --- |
| `ANDROID_HOME` / `ANDROID_SDK_ROOT` | Install into this SDK directory instead of the per-OS default. |
| `ANDROID_CMDLINE_TOOLS_BUILD=<build>` | Install a specific command-line tools build (e.g. `13114758`). |
| `ANDROID_CMDLINE_TOOLS_URL=<url>` | Install an exact archive URL, bypassing lookup entirely. |
| `ANDROID_CMDLINE_TOOLS_PIN=1` | Use the built-in fallback build and make no network lookup. |
| `ANDROID_CMDLINE_TOOLS_UPGRADE=1` | Reinstall/upgrade the command-line tools even if already present. |
| `ANDROID_ENV_USE_SDKMANAGER=1` | Use the legacy `sdkmanager` instead of the `android` CLI. |

Run `./android-env tools` to see what would be downloaded before committing to it.

Upgrade an existing install to the newest command-line tools:

```bash
ANDROID_CMDLINE_TOOLS_UPGRADE=1 ./android-env setup
```

The existing install is moved aside during an upgrade and restored automatically if
the new one fails to unpack.

**Default SDK locations**

| OS | Path |
| --- | --- |
| macOS | `~/Library/Android/sdk` |
| Linux | `~/Android/Sdk` |
| Windows | `~/AppData/Local/Android/Sdk` |

If you already have an Android SDK from Android Studio or Homebrew, note that this
tool only ever touches the directory above (or your `ANDROID_HOME`). It addresses
`sdkmanager`, `avdmanager`, `emulator` and `adb` by absolute path, so a different
SDK earlier in your `PATH` cannot intercept the install. `./android-env doctor`
warns you when it detects one.

---

## 🛠 Prerequisites

`setup` checks for all of these before downloading anything, and tells you which are missing:
- `curl` and `unzip` (usually pre-installed on Mac/Linux)
- `git` (used to fetch the device skins)
- Java — JDK 17+ (required by `sdkmanager` and `avdmanager`)

---

## 🔍 Troubleshooting

- **`./android-env doctor`** is the first stop — it reports every tool it needs, with the exact path it looked in.
- **`./android-env tools`** shows whether your command-line tools are current and how to upgrade them.
- **SDK install logs** are kept at `$ANDROID_HOME/.oneclick-sdk-install.log`.
- **`sdkmanager` warns that it is deprecated.** Expected on command-line tools rev 23+; it delegates to the `android` CLI. This tool calls the `android` CLI directly, so you will only see this with `ANDROID_ENV_USE_SDKMANAGER=1`.
- **`SDK XML versions up to 3 but ... version 4 was encountered`** means your command-line tools are older than the rest of your SDK. Upgrade with `ANDROID_CMDLINE_TOOLS_UPGRADE=1 ./android-env setup`.

---

## 🙏 Acknowledgements

Device frames are downloaded from
[larskristianhaga/Android-emulator-skins](https://github.com/larskristianhaga/Android-emulator-skins).
