# OffHandRelay

**Press buttons on one VR controller using the other one, in every SteamVR game.**
Includes a small app for choosing which button does what.

OffHandRelay is a small SteamVR driver for players who can move and track
both hands but can only press buttons with one of them. For example, with a
controller strapped to a hand that can't press buttons, you can hold **X** on the
right controller and the game sees **left grab**.

Because it works inside SteamVR itself, games just see a normal left-hand (or
right-hand) press. That means no mods and no per-game setup beyond normal
bindings. It's tested on the Steam Frame controllers and should work with any
SteamVR controller.

## Install

1. Download `OffHandRelay.zip` from the [Releases](../../releases) page.
2. Quit SteamVR.
3. Unzip the `offhandrelay` folder into
   `C:\Program Files (x86)\Steam\steamapps\common\SteamVR\drivers\`,
   so that this file exists:
   `...\SteamVR\drivers\offhandrelay\driver.vrdrivermanifest`
4. Start SteamVR and turn on your controllers.
5. In **SteamVR Settings > Startup / Shutdown > Manage Add-ons**, make sure
   **offhandrelay** is **On**. SteamVR turns off add-ons that crashed, so check
   here first if nothing happens.

To uninstall, quit SteamVR and delete the `offhandrelay` folder.

## Changing the buttons: OffHandRelay Config

Open **`OffHandRelay Config.exe`**, which is in the same `offhandrelay` folder.
Right-click it and choose *Send to > Desktop (create shortcut)* for easy access.

![OffHandRelay Config](docs/config-app.png)

- **The header** shows whether SteamVR is running and the driver loaded. If
  SteamVR is running but the driver isn't, it tells you where to turn it on.
- **The controller drawing** shows each rule as a colored arc from the button
  you press to the button it acts as. With *Show live presses* on, buttons
  light up as you press them in VR. Click a button to edit its rule, or to
  add one.
- **Buttons** lists your rules, one sentence each, such as "Right X button acts
  as Left Bumper". Use **Add**, **Edit** (or double-click) and **Remove**.
  Untick a rule to turn it off without deleting it.
- **Sticks and D-pad** has one line for the thumbstick relay and one for the
  D-pad layer, each with an on/off checkbox and an **Edit** button.
- **Open SteamVR bindings** opens SteamVR's controller bindings screen in your
  browser (SteamVR must be running), for changing what each game does with
  each button.

Editing opens a small window with the details. For a button rule, that's the
button you press and the button it acts as (pick from the list, type a path
such as `/input/b/click`, or click **Detect a press** and press it on your
controller), *Hold* or *Toggle*, and whether the original button's own job is
turned off. For the thumbstick relay, it's which stick acts as which and when.
For the D-pad layer, it's which button you hold.

The defaults: the right stick works normally (turning, in most games), and
while you hold the right bumper it acts as the left stick (moving). Holding the right bumper makes Y, X, B, A
press the left D-pad up, left, right, down, and the bumper itself keeps working.

Every change is saved immediately, and the driver applies it within a second.
You don't need to restart SteamVR.

## Default setup

`offhandrelay.cfg` (in the driver folder) comes with:

| You press (right hand) | The game sees (left hand) |
|---|---|
| **Right stick** | The right stick, as normal (turning, in most games) |
| **X** (hold) | Left **bumper** (the grab button in most games) |
| **Y** (hold) | Left **trigger** |
| **Bumper** (hold) + **right stick** | The **left stick** (moving). The bumper still does its own job too. |
| **Bumper** (hold) + **Y / X / B / A** | Left **D-pad** up / left / right / down (instead of their usual jobs) |

This works with each game's **default bindings**, so there's no need to move
movement or turning between sticks. Bind the game's left-hand grab to the
**left bumper** and its trigger/use to the **left trigger** if they aren't
already. X and Y stop doing anything on the right hand, so leave them unbound.
The bumper still reaches the game while you hold it, so in games where it
grabs, your right hand closes while you move. Pick a different button for the
thumbstick relay in the app if that gets in the way.

## Editing the config by hand

The app writes `offhandrelay.cfg`, a plain text file you can also edit
yourself. Changes apply within a second of saving. One rule per line:

```
<source hand> <source path> -> <target hand> <target path> [hold|toggle] [suppress|keep]
```

| Option | Meaning |
|---|---|
| `hold` | Target is pressed while the source is held (default) |
| `toggle` | Press once = target held down, press again = released |
| `suppress` | Source button no longer does anything on its own hand (default) |
| `keep` | Source button also keeps working normally on its own hand |
| `while <hand> <button>` | A **layer**: the rule only applies to presses made while that button is held, and takes over the source button from rules without `while` for those presses. The held button keeps working normally. |
| `off` | Rule stays in the file but does nothing (the app's checkbox) |

The target's `/click`, `/touch` and `/value` are all driven together, so games
that read analog trigger or grip values see a full press.

Examples:

```
right /input/x/click -> left /input/bumper/click hold suppress
right /input/y/click -> left /input/trigger/click hold suppress
left  /input/dpad_up/click -> right /input/a/click hold keep
right /input/x/click -> left /input/dpad_left/click hold suppress while right /input/bumper/click
```

### Stick rules

```
stick <hand> <stick path> -> <hand> <stick path> [while|unless <hand> <button path>] [suppress|keep]
```

```
stick right /input/thumbstick -> left /input/thumbstick while right /input/bumper/click suppress keep_button
```

The source stick's `x` and `y` are copied to the target stick, and the target's
`touch` reports as touched while the stick is pushed. With `while`, the relay
only runs while that button is held. With `unless`, it runs all the time
*except* while that button is held. Either way the button itself is hidden from
games, unless you add `keep_button`. Without either, the relay is always on. `suppress` (default) makes the source stick
read as centered while relaying. `keep` lets it keep working on its own hand too.

### Finding button names

With `log_presses true`, the driver logs every button it sees and every press
to SteamVR's log, in `C:\Program Files (x86)\Steam\logs\vrserver.txt` or live
in **SteamVR Settings > Developer > Web Console**:

```
[OffHandRelay] Registered button /input/bumper/click (device 4294967298)
[OffHandRelay] right /input/x/click PRESSED
[OffHandRelay] Relay -> left /input/bumper ON
```

Steam Frame controller buttons (`/input/<name>/click`):

- **Right:** `a` `b` `x` `y` `menu` `system` `thumbstick` `grip` `bumper` `trigger`
- **Left:** `dpad_up` `dpad_down` `dpad_left` `dpad_right` `view` `system` `thumbstick` `grip` `bumper` `trigger`

`log_presses` is the app's *Show live presses* setting. The live drawing and
Detect need it on. Turning it off makes the log smaller, nothing else.

## Game notes

**Hot Dogs, Horseshoes & Hand Grenades:** bind the left Bumper to
`grip_button`. The [Accessibility Options](https://thunderstore.io/c/h3vr/p/Okkim/Accessibility_Options/)
mod pairs well with it.

**Half-Life: Alyx:** use the **default** binding (not "Movement on Weapon Hand"),
and on the **Interact** tab bind the left Bumper to grab. The right stick turns
you, and holding the right bumper makes it move you. Gravity gloves work by holding X and flicking the
left wrist.

**The Walking Dead: Saints & Sinners – Chapter 2:** use the **default** binding.
Movement only seems to respond to the left controller, which the stick relay
takes care of. Set *Grab style* to **Hold** in the game's options.

## Troubleshooting

- **Nothing happens.** Check Manage Add-ons (step 5), then look for
  `[OffHandRelay] Hooked IVRDriverInput_00x: OK` in the log.
- **Presses are logged, but the game doesn't react.** The driver is working, so
  the cause is the game binding. Check that the action is on the left bumper/trigger
  and that you saved a *personal* binding.
- **It stopped after a SteamVR update.** Look for `Relay ->` lines in the log
  and open an issue with the `[OffHandRelay]` lines.

## How it works

SteamVR controller drivers report every button through the `IVRDriverInput`
interface. OffHandRelay loads into the same process (`vrserver`), patches the
first four entries of that interface's vtable (create/update boolean and
scalar components), and keeps a map of every input component per hand. When a
configured source button changes, it updates the target hand's components
through the original functions. It adds no devices of its own.

Once a second (from `RunFrame`), the driver checks the config file's
modification time. When the file changes, it releases every active relay,
loads the new rules, and syncs them to the buttons' current state. The
config app saves through a temp file plus an atomic rename, so the driver
never reads a half-written file.

This is a hook into SteamVR internals, not an official API, so a future
SteamVR update could break it.

## Building

Requirements: CMake 3.16+ and Visual Studio 2019+ (or MinGW-w64).

```
cmake -S . -B build -A x64
cmake --build build --config Release
```

The DLL is copied to `driver/offhandrelay/bin/win64/` and the config app to
`driver/offhandrelay/`, so that folder is ready to drop into SteamVR's
`drivers` directory. Both are plain C++ (Win32 for the app) and need no runtime.

### Tests

On Linux or macOS, the same CMake build also produces `fake_vrserver`. It loads
the real driver into a fake SteamVR with fake controllers and checks relaying,
suppression, reconnects, toggle mode and live reload:

```
cmake -S . -B build && cmake --build build && ctest --test-dir build --output-on-failure
```

To cross-compile from Linux with MinGW, use a toolchain file that sets
`CMAKE_SYSTEM_NAME Windows`, `CMAKE_CXX_COMPILER x86_64-w64-mingw32-g++-posix`
and `CMAKE_RC_COMPILER x86_64-w64-mingw32-windres`.

Pushing a tag such as `v1.0.0` makes GitHub Actions build the driver and attach
`OffHandRelay.zip` to the release.

## License

MIT, see [LICENSE](LICENSE). `third_party/openvr/openvr_driver.h` is
© Valve Corporation, BSD-3-Clause.
