# OffHandRelay

**Press buttons on one VR controller using the other one, in every SteamVR game.**

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

## Default setup

`offhandrelay.cfg` (in the driver folder) comes with:

| You press (right hand) | The game sees (left hand) |
|---|---|
| **X** (hold) | Left **bumper**, the grab button in most games |
| **Y** (hold) | Left **trigger** |

X and Y stop doing anything on the right hand, so leave them unbound in your
game bindings. Then bind the game's grab action to the **left bumper** and
its trigger/use action to the **left trigger**, as you normally would.

## Configuring

Edit `offhandrelay.cfg` and restart SteamVR. One rule per line:

```
<source hand> <source path> -> <target hand> <target path> [hold|toggle] [suppress|keep]
```

| Option | Meaning |
|---|---|
| `hold` | Target is pressed while the source is held (default) |
| `toggle` | Press once = target held down, press again = released |
| `suppress` | Source button no longer does anything on its own hand (default) |
| `keep` | Source button also keeps working normally on its own hand |

The target's `/click`, `/touch` and `/value` are all driven together, so games
that read analog trigger or grip values see a full press.

Examples:

```
right /input/x/click -> left /input/bumper/click hold suppress
right /input/y/click -> left /input/trigger/click hold suppress
left  /input/dpad_up/click -> right /input/a/click hold keep
```

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

Set `log_presses false` once everything works.

## Game notes

**Hot Dogs, Horseshoes & Hand Grenades:** bind the left Bumper to
`grip_button`. The [Accessibility Options](https://thunderstore.io/c/h3vr/p/Okkim/Accessibility_Options/)
mod pairs well with it.

**Half-Life: Alyx:** pick Valve's **"Dual Controllers (Movement on Weapon Hand)"**
binding, then on the **Interact** tab bind the left Bumper to grab. Movement is
on the right stick, and gravity gloves work by holding X and flicking the left wrist.

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

This is a hook into SteamVR internals, not an official API, so a future
SteamVR update could break it.

## Building

Requirements: CMake 3.16+ and Visual Studio 2019+ (or MinGW-w64).

```
cmake -S . -B build -A x64
cmake --build build --config Release
```

The DLL is copied to `driver/offhandrelay/bin/win64/`, so that folder is ready
to drop into SteamVR's `drivers` directory.

MinGW cross-compile from Linux:

```
x86_64-w64-mingw32-g++-posix -std=c++17 -O2 -shared -static -static-libgcc -static-libstdc++ \
  -Ithird_party/openvr src/driver.cpp -o driver/offhandrelay/bin/win64/driver_offhandrelay.dll
```

Pushing a tag such as `v1.0.0` makes GitHub Actions build the driver and attach
`OffHandRelay.zip` to the release.

## License

MIT, see [LICENSE](LICENSE). `third_party/openvr/openvr_driver.h` is
© Valve Corporation, BSD-3-Clause.
