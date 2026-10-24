# 3mpq utilities — install & troubleshooting

Three small macOS menu-bar apps, all **signed with an Apple Developer ID and
notarized by Apple**, built for **Apple Silicon (arm64)**, macOS 12+.

| App | What it does | Menu-bar | Needs permission |
|-----|--------------|----------|------------------|
| **Pet in The System (PTS)** | A tiny pixel pet that roams across your windows, sleeps, falls when you grab a window, and gets thrown when you drag & drop it. | ● icon | Accessibility |
| **WGrid** | Tiles windows into grids (1×1…3×3), drag-to-rearrange. | grid icon | Accessibility |
| **Quirky** | Capture suite: OCR text from screen, color eyedropper, pixel measure, extract DOM/SVG from browsers. | ◆ icon | Screen Recording (+ Automation for browser modes) |

## Install

1. Open the `.dmg`, drag the app into **Applications**.
2. Launch it from **Applications** (or Spotlight). Each app lives in the **menu
   bar** (top-right), not the Dock — click its icon to use it.
3. On first launch macOS asks for a permission (see table). Grant it in
   **System Settings → Privacy & Security**, then relaunch the app.

That's it. Because the apps are notarized, they should open normally with no
"unidentified developer" warning.

---

## If something goes wrong

### "App is damaged / can't be opened / unidentified developer"
This happens if macOS marked the download as quarantined. Fix, per app:
- Right-click the app in **Applications → Open → Open** (only needed once), **or**
- Terminal: `xattr -dr com.apple.quarantine "/Applications/Quirky.app"`
  (swap in `PTS.app` / `WGrid.app`).

### The app opened but "does nothing"
It's a **menu-bar** app — look at the **top-right** of the screen for its icon,
not the Dock. If the icon is there but a feature doesn't work, it's almost
always a missing **permission** (see below) — grant it, then quit & relaunch
the app (permissions only take effect on relaunch).

### Granting permissions
- **Accessibility** (PTS, WGrid): System Settings → Privacy & Security →
  **Accessibility** → toggle the app on. Quick open:
  `open "x-apple.systempreferences:com.apple.preference.security?Privacy_Accessibility"`
- **Screen Recording** (Quirky): System Settings → Privacy & Security →
  **Screen Recording** → toggle Quirky on, then relaunch. Quick open:
  `open "x-apple.systempreferences:com.apple.preference.security?Privacy_ScreenCapture"`
- **Automation** (Quirky's DOM/SVG modes only): macOS will prompt to allow
  Quirky to control your browser — click **OK**.

---

## FOR YOUR AI ASSISTANT (Claude / any coding agent)

If the human says an app "won't open" or "doesn't work", run these checks and
act on the results. All three apps are Developer-ID-signed + Apple-notarized,
arm64, menu-bar (LSUIElement) apps.

```bash
APP="/Applications/Quirky.app"   # or PTS.app / WGrid.app

# 1. Right architecture? (these are arm64; Rosetta covers Intel for most, but
#    a pure-Intel Mac may struggle — expected: arm64)
uname -m
lipo -archs "$APP/Contents/MacOS/"* 2>/dev/null

# 2. macOS new enough? PTS needs 12+, WGrid & Quirky need 13+
sw_vers

# 3. Is Gatekeeper happy? Expected: "accepted, source=Notarized Developer ID"
spctl --assess --type execute --verbose=2 "$APP"
xcrun stapler validate "$APP"          # expected: "The validate action worked!"

# 4. Quarantined? If it prints a value, strip it:
xattr -p com.apple.quarantine "$APP" 2>/dev/null && xattr -dr com.apple.quarantine "$APP"

# 5. Is it actually running? (menu-bar apps show no window)
pgrep -x Quirky   # or PTS / WGrid
```

### Most common root cause → fix
- **spctl "rejected" / stapler fails** → the download got corrupted or altered.
  Re-download the `.dmg` from the link and reinstall.
- **quarantine attribute present** → `xattr -dr com.apple.quarantine "$APP"`.
- **Runs (pgrep shows a PID) but a feature is dead** → missing TCC permission.
  These are **user-granted only** (cannot be enabled from the CLI — TCC is
  protected). Open the right pane (commands above) and have the human toggle it,
  then `killall <App>` and relaunch. PTS/WGrid → Accessibility. Quirky → Screen
  Recording.
- **Nothing in the menu bar at all** → the process died on launch. Check Console:
  `log show --last 5m --predicate 'process == "Quirky"' --info` (swap the name),
  or launch from Terminal to see stderr:
  `"/Applications/Quirky.app/Contents/MacOS/Quirky"`.

### What these apps are NOT
No installer, no background daemon, no login item unless you add one, no network
telemetry. Each is a single self-contained `.app`. To fully remove one: quit it
from its menu-bar icon and drag it from Applications to Trash.
