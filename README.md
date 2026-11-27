# MC55A Apps

The app catalog for **[App Store](https://github.com/oseart55/AppStore)**, which
installs and updates apps on Zebra/Motorola MC55A handhelds and other Windows
Mobile / Windows CE devices.

This repository holds just one file, [`catalog.json`](catalog.json). App Store
downloads it from here every time you open it or tap **Refresh**. Each app
listed points to its own GitHub repository, whose latest release is what gets
installed.

## Apps

| App | What it does | Repository |
|---|---|---|
| **Book Scout** | Scan books and see instantly whether Ziffit will buy them. Tracks your haul and profit. | [oseart55/BookScanner](https://github.com/oseart55/BookScanner) |
| **ClockSync** | Sets the clock from internet time servers at every boot and whenever the network connects. | [oseart55/ClockSync](https://github.com/oseart55/ClockSync) |
| **App Store** | The store itself. Install it from the list to keep a permanent copy in `\Application` that updates itself. | [oseart55/AppStore](https://github.com/oseart55/AppStore) |
| **CeTls** | Modern TLS for the whole device: replaces the TLS library IE and other programs use, adding TLS 1.2 and SNI. Open it to install, test or remove it; a clean boot also puts the original back. | [oseart55/CeTls](https://github.com/oseart55/CeTls) |
| **TlsProbe** | Diagnostic for the CeTls project (modern TLS for the whole device): records how TLS is set up and writes `TlsProbe.txt` next to itself. Read-only. | [oseart55/CeTls](https://github.com/oseart55/CeTls) |
| **CeTls Spy** | Diagnostic for CeTls: logs how programs use secure sockets, passing everything through unchanged. Open it to switch it on or off. | [oseart55/CeTls](https://github.com/oseart55/CeTls) |
| **SchShadow** | Diagnostic for CeTls: briefly puts a logging stand-in for the TLS library in place to see what IE's HTTPS code asks of it, then restores the original. | [oseart55/CeTls](https://github.com/oseart55/CeTls) |
| **RootTest** | Diagnostic for CeTls: adds ISRG Root X1 (Let's Encrypt) to the root store and tests whether the device then trusts modern certificates. Uninstall removes the root. | [oseart55/CeTls](https://github.com/oseart55/CeTls) |

All of them use [CeHttps](https://github.com/oseart55/CeHttps) for modern HTTPS
(TLS 1.3 / 1.2). That's how they reach GitHub and other servers on devices
whose built-in TLS stops at 1.0.

## Getting started on a device

- **Devices with .NET Compact Framework 3.5** (e.g. Windows Embedded Handheld
  6.5): copy `AppStore.exe` to the device and run it.
- **Devices with .NET CF 2.0** (many Windows Mobile 5 / 6.0 / 6.1 devices): run
  `AppStoreSetup.exe`. It installs ClockSync, CF 3.5 and App Store in one go.

Then pick an app and tap **Install**. The
[App Store README](https://github.com/oseart55/AppStore#installing-on-a-device)
has the full steps and troubleshooting.

## Adding an app

1. **Publish the app as a GitHub release** in its own repository. Use a version
   tag such as `v1.0.0` and attach the app's files: an exe, loose files or one
   `.zip`. (GitHub can't make a release in an empty repository, so add a first
   commit before the release.)
   ```
   gh release create v1.0.0 MyApp.exe --repo you/MyApp --title v1.0.0 --notes "First release"
   ```
2. **Add an entry to `catalog.json`.** Only `name` and `repo` are required:
   ```json
   {
     "id": "myapp",
     "name": "My App",
     "repo": "you/MyApp",
     "description": "One line about what it does.",
     "folder": "MyApp",
     "exe": "MyApp.exe"
   }
   ```
3. **Commit and push.** Devices see the new app on their next Refresh. Nothing
   needs reinstalling.

### Fields

| Field | Meaning |
|---|---|
| `name` | Shown in the list. **Required.** |
| `repo` | `owner/name` of the repository whose releases hold the app. **Required.** |
| `id` | Stable id that links installed copies to this entry. Defaults to the repo. **Don't change it once devices have the app installed**, or they'll lose track of it. |
| `description` | Shown below the list. |
| `folder` | Install folder under `\Application`. Defaults to the name without spaces. |
| `exe` | What Open and the Start menu shortcut start, and where the version is read. Defaults to the first `.exe`. |
| `shortcut` | Start menu shortcut name, or `false` for none. Defaults to the name. |
| `assets` | Which release files to install, e.g. `["MyApp.exe"]` (wildcards work). By default App Store takes the release's only `.zip`, or else every attached file. |
| `install` / `uninstall` | Arguments to run the exe with after installing or updating, and before uninstalling. Used for apps with their own setup step, like ClockSync's `/install`. |
| `keep` | Files never overwritten once they exist, e.g. settings the user edits. |
| `logs` | Log files the app writes in its folder (wildcards work). App Store's **Send logs** uploads them to your private logs repo. |

Folder, exe and shortcut names must be plain names: no `\`, `/` or `..`. App
Store skips entries that break this. The
[App Store README](https://github.com/oseart55/AppStore#the-catalog) explains
how updates, `keep` files and uninstalls work.

### Before you push

- **Check that the file is valid JSON**, for example with
  `python -m json.tool catalog.json`. A syntax error stops every device from
  loading the catalog until it's fixed. JSON has no comments and no trailing
  commas.
- **Write backslashes twice inside strings** (`"\\Application"`).
- **Releasing an update** only needs a new release in the app's repository, with
  a higher version tag and a matching version number in the exe. The catalog
  doesn't change.

## Using a different catalog

App Store reads this catalog by default. To point a device at your own, put a
`store.cfg` file next to `AppStore.exe` containing:

```
catalog=you/your-catalog-repo
```

Or give a full `https://` URL to any `catalog.json`.
