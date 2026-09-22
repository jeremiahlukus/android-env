# android-env

A single-file Bash CLI that sets up a complete Android development environment
without installing Android Studio. It installs the Android SDK command-line
tools, platform tools, a system image and an emulator, creates a virtual device
with a realistic phone frame, and adds a desktop launcher so you can start the
emulator like any other app.

Works on macOS, Linux and Windows (via Git Bash).


## Requirements

`setup` checks for all of these before downloading anything, and tells you
which are missing:

| Tool | Notes |
| --- | --- |
| `curl` | Usually pre-installed on macOS and Linux |
| `unzip` | Usually pre-installed on macOS and Linux |
| `git` | Used to download the device frames |
| Java | JDK 17 or newer, required by `avdmanager` |


## Quick start

1. Make the script executable:

       chmod +x android-env

2. Run the setup:

       ./android-env setup

3. Choose a device frame when prompted: Pixel 6 Pro, Pixel 6 or Pixel 5.

4. Enter a name for your emulator, for example `MyPixelPhone`.

5. Optional, but recommended. Put the command on your PATH so it works from
   any directory:

       ./android-env install

   After this you can drop the `./` and run `android-env` from anywhere.

That is all. The first run downloads roughly 1.7 GB and takes a few minutes.

On Windows, run the same commands from Git Bash. Standard CMD and PowerShell
will not work.


## Starting your emulator

There are two ways.

As a desktop app. Setup creates a launcher named after your emulator, so search
for it the way you would any other application:

- macOS: Spotlight (Cmd+Space) or Launchpad
- Windows: Start Menu
- Linux: your application launcher

From the terminal:

    android-env start

If you have not run `android-env install`, use `./android-env start` from the
directory holding the script.


## Commands

| Command | What it does |
| --- | --- |
| `setup` | Installs the SDK, creates an emulator, adds a desktop launcher |
| `start [name]` | Starts an emulator. Uses the first one if no name is given |
| `list` | Lists your emulators |
| `delete [name]` | Deletes an emulator and its desktop launcher |
| `doctor` | Checks Java, the SDK, and every tool the script needs |
| `tools` | Shows your command-line tools version and whether a newer one exists |
| `stop [name]` | Stops a running emulator, or all of them if none is named |
| `ensure [name]` | Guarantees a booted emulator, provisioning first if needed |
| `install [dir]` | Puts the command on your PATH so it runs from anywhere |
| `uninstall` | Removes it from your PATH |
| `version` | Prints the version |

### Deleting an emulator

    ./android-env delete                       # choose from a list
    ./android-env delete MyPixelPhone_API_34   # or name it directly

You are shown what will be removed and its size, and you must retype the
emulator name to confirm. Add `-y` to skip the prompt. Deleting is refused
while that emulator is running, and never touches the SDK, system images or
your other emulators.


## Running it from anywhere

By default you have to be in the directory holding the script and type
`./android-env`. To use it like any other command:

    ./android-env install

This picks the first directory that is both on your PATH and writable without
`sudo`, trying `~/.local/bin`, then `~/bin`, then `/usr/local/bin`. It creates
a symlink, so any later edit to the script takes effect immediately. On systems
where symlinks are restricted it installs a copy instead and tells you so.

To choose the directory yourself:

    ./android-env install ~/bin

If that directory is not on your PATH, the command prints the exact line to add
to your shell profile.

To remove it again:

    android-env uninstall

Uninstalling only removes the command from your PATH. Your SDK, emulators and
desktop launchers are left alone.


## Unattended use

`ensure` is the one call to make from a script, a CI job or an MCP server. It
guarantees a booted, ready emulator: it provisions the SDK and creates an
emulator if either is missing, starts it if it is not running, waits for the
boot to finish, and prints the serial.

    android-env ensure --json

    {"success":true,"avd":"Pixel6Pro_API_34","serial":"emulator-5554",
     "api_level":"34","abi":"arm64-v8a","already_running":false,
     "provisioned":false}

It is safe to call repeatedly. If the emulator is already booted it returns in
about a second with `already_running: true`. Failures come back as
`{"success": false, "error": "..."}` rather than a non-zero exit with no
detail, so a caller can report the reason.

To boot a specific emulator and wait without the provisioning step:

    android-env start MyPixelPhone_API_34 --wait --timeout=300

To shut down:

    android-env stop                      # every running emulator
    android-env stop MyPixelPhone_API_34  # just this one

These variables drive the tool where nobody can answer a prompt:

| Variable | Effect |
| --- | --- |
| `ANDROID_ENV_YES=1` | Never prompt, take the documented default |
| `ANDROID_ENV_AVD_NAME` | Emulator name for `setup`, instead of asking |
| `ANDROID_ENV_PROFILE` | Device profile for `setup`: 1, 2 or 3 |
| `ANDROID_ENV_BOOT_TIMEOUT` | Seconds to wait for boot, default 300 |

Prompts are also skipped automatically when stdin is not a terminal, so piping
into the tool will not hang it.


## Configuration

Nothing needs configuring. These environment variables exist for when you need
to pin a version, force an upgrade, or work offline.

| Variable | Effect |
| --- | --- |
| `ANDROID_HOME` | Install into this SDK directory instead of the default |
| `ANDROID_CMDLINE_TOOLS_BUILD` | Install a specific command-line tools build |
| `ANDROID_CMDLINE_TOOLS_URL` | Install an exact archive URL |
| `ANDROID_CMDLINE_TOOLS_PIN=1` | Use the built-in fallback build, no network lookup |
| `ANDROID_CMDLINE_TOOLS_UPGRADE=1` | Reinstall the command-line tools even if present |
| `ANDROID_ENV_USE_SDKMANAGER=1` | Use the legacy `sdkmanager` instead of the `android` CLI |

Run `./android-env tools` to see exactly what would be downloaded before
committing to it.

To upgrade an existing install to the newest command-line tools:

    ANDROID_CMDLINE_TOOLS_UPGRADE=1 ./android-env setup

During an upgrade the existing install is moved aside, and restored
automatically if the new one fails to unpack.

### Default SDK locations

| OS | Path |
| --- | --- |
| macOS | `~/Library/Android/sdk` |
| Linux | `~/Android/Sdk` |
| Windows | `~/AppData/Local/Android/Sdk` |


## How it works

No Android Studio. Only the command-line tools are installed.

Current tooling. The newest stable command-line tools build is looked up in
Google's own SDK index at install time rather than hardcoded, so it does not go
stale. This includes the architecture-specific macOS archives Google publishes
from revision 22.0 onward.

Modern installer. SDK packages are installed with the `android` CLI that ships
with command-line tools revision 23 and later, passing `--no-metrics` to opt
out of usage-data collection. Older tools fall back to `sdkmanager`.

Licenses. Required Google SDK licenses are accepted without prompting.

Verified installs. After installing, the script confirms on disk that `adb`,
the emulator, the platform and the system image all arrived. This matters
because both `android sdk install` and `sdkmanager` exit with status `0` even
when a package is not found, so exit codes alone cannot be trusted.

No interference. `sdkmanager`, `avdmanager`, `emulator` and `adb` are called by
absolute path, so another Android SDK earlier in your `PATH`, from Homebrew or
Android Studio, cannot intercept the install. `doctor` warns you when it finds
one.

Architecture detection. Picks `arm64-v8a` on Apple Silicon and ARM Linux,
`x86_64` elsewhere.

Device frames. Downloads Pixel 6 Pro, Pixel 6 and Pixel 5 frames and configures
your emulator with the one you choose, so it looks like a real phone. You can
preview them in your image viewer first.

Safe to re-run. Setup detects existing emulators and offers to skip.


## Troubleshooting

Start with `./android-env doctor`. It reports every tool it needs and the exact
path it looked in.

Check your tooling with `./android-env tools`. It shows whether your
command-line tools are current and how to upgrade them.

Install logs are kept at `$ANDROID_HOME/.oneclick-sdk-install.log`.

"SDK XML versions up to 3 but ... version 4 was encountered" means your
command-line tools are older than the rest of your SDK. Upgrade them:

    ANDROID_CMDLINE_TOOLS_UPGRADE=1 ./android-env setup

"The SDK Manager CLI tool (sdkmanager) is deprecated" is expected on
command-line tools revision 23 and later. This script calls the `android` CLI
directly, so you will only see this if you set `ANDROID_ENV_USE_SDKMANAGER=1`.

Reclaiming disk space. Emulators are large, often several GB each. Use
`./android-env delete` to remove one along with its launcher. To also remove
the downloaded system image:

    android --no-metrics --sdk="$ANDROID_HOME" sdk remove \
      system-images/android-34/google_apis/arm64-v8a platforms/android-34


## Acknowledgements

Device frames are downloaded from
[larskristianhaga/Android-emulator-skins](https://github.com/larskristianhaga/Android-emulator-skins).
