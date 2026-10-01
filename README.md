# OffHandRelay

A SteamVR driver that lets buttons on one controller press buttons on the
other controller. Because it works inside SteamVR itself, every SteamVR game
sees a normal left-hand (or right-hand) press, with no per-game setup.

Example: press the right bumper and SteamVR reports "left grip pressed".

## Install

1. Quit SteamVR.
2. Copy the `offhandrelay` folder into
   `C:\Program Files (x86)\Steam\steamapps\common\SteamVR\drivers\`
   so you end up with `...\SteamVR\drivers\offhandrelay\driver.vrdrivermanifest`.
3. Start SteamVR, then turn your controllers on (or power-cycle them) so the
   driver sees their buttons being created.
4. In SteamVR **Settings > Startup / Shutdown > Manage Add-ons**, check that
   **offhandrelay** is On. SteamVR turns off add-ons that crashed, so look here
   first if nothing happens.

To uninstall, quit SteamVR and delete the folder.

## First run: find your button names

The included `offhandrelay.cfg` has `log_presses true`. With it on, the driver
writes to SteamVR's log:

- every button it sees when a controller connects
  (`[OffHandRelay] Registered button /input/grip/click ...`)
- every press (`[OffHandRelay] right /input/grip/click PRESSED`)
- every relay (`[OffHandRelay] Relay -> left /input/grip ON`)

The log is in `C:\Program Files (x86)\Steam\logs\vrserver.txt`, or live in
**SteamVR Settings > Developer > Web Console**. Press the buttons you want to
use, copy their paths into the rules, and restart SteamVR.

## Rules

```
<source hand> <source path> -> <target hand> <target path> [hold|toggle] [suppress|keep]
```

- `hold`: target pressed while the source is held.
- `toggle`: press once = target held, press again = released. Good for grabbing.
- `suppress`: the source button stops doing anything on its own hand.
- `keep`: the source button also still works on its own hand.

The target's `/click`, `/touch` and `/value` are driven together, so games that
read analog trigger or grip values see a full press.

## Then fix your bindings

Remove any chords or cross-hand hacks you made in the binding editor and go
back to the game's normal bindings: the left grip/trigger will now "just work".

## Notes and limits

- If a controller was already on when SteamVR started the driver, its buttons
  may not be registered. Power-cycle the controller.
- SteamVR updates can change internals. If it stops working after an update,
  check the log for `Hooked IVRDriverInput_00x: OK`.
- Built from `src/driver.cpp` (C++17). Rebuild on Windows with CMake +
  Visual Studio, or MinGW:
  `x86_64-w64-mingw32-g++ -std=c++17 -O2 -shared -static -Ithird_party/openvr src/driver.cpp -o driver_offhandrelay.dll`
- OpenVR header (`third_party/openvr`) is (c) Valve, BSD-3 licensed.
