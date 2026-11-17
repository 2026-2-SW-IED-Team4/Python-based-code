# EV3 Remote: pick a room on the map, joystick and speed from a phone

A Python program for a tracked EV3 robot. The robot runs a small web server, and the phone opens the control page in an ordinary browser. Nothing has to be installed on the phone.

The page has three tabs:
- **Rooms.** Your drawn map shows where the robot is, with the rooms marked. Tap a room on the map or its number button, and the robot finds the way along the corridor from wherever it stands.
- **Joystick.** Drive the robot with your finger. Its position on the map keeps updating.
- **Settings.** Map upload, calibration and behaviour.

The **Speed** slider and the red **STOP** button are always visible.

The page language (**EN / 한국어 / RU**) is switched at the top right. The phone remembers the choice.

This guide in other languages: [한국어](README.ko.md), [Русский](README.ru.md).

## Wiring (same as the EV3 Classroom project)

| Port | Device | What it does |
|---|---|---|
| B, D | track motors | driving |
| 1 | ultrasonic sensor | closer than 15 cm, the robot waits |
| 2 | touch sensor | stop and turn 180° (can be set to "stop only") |
| 3 | colour sensor, facing the floor | a yellow sticker corrects the robot's position on the map |
| 4 | gyro sensor | which way the robot faces (turns are inaccurate without it) |

## What you need

1. **A microSD card with ev3dev.** The standard LEGO firmware can't run the Python server. Download the *ev3dev-stretch* image from ev3dev.org, write it to the card with balenaEtcher and insert the card into the EV3. Without the card, the EV3 works with EV3 Classroom as before.
   A "LEGO EV3 MicroPython" card also works: it is ev3dev and includes `python3` and `ev3dev2`.
2. **A connection to the phone**, one of these:
   - **Bluetooth** (nothing to buy). Turn on the phone's hotspot (on Android, "Bluetooth tethering"). On the EV3: *Wireless and Networks → Bluetooth*, pair with the phone, then *Network Connection → Connect*.
   - **A USB Wi-Fi adapter for the EV3** (more reliable). It must be on the ev3dev compatibility list. Turn on the phone's hotspot and join it from *Wireless and Networks → Wi-Fi* on the EV3.

## Installation

1. Copy the `ev3robot` folder (`robot_server.py` and the `static` folder) to the EV3, for example from a Mac or PC on the same network:
   `scp -r ev3robot robot@<EV3 address>:~/` (password `maker`).
   Visual Studio Code with the *ev3dev-browser* extension also works.
2. Once, allow the file to run: `ssh robot@<EV3 address>`, then `chmod +x ~/ev3robot/robot_server.py`.
3. On the EV3, open *File Browser → ev3robot → robot_server.py* and press the centre button.
4. **Don't touch the robot** for the first 2 seconds while the gyro calibrates.
5. The EV3 screen shows a line like `Phone: http://172.20.10.6:8000`. Open that address in the phone's browser.

To try it without the robot, run `python3 robot_server.py --sim` on a computer and open `http://localhost:8000`. The robot is simulated in this mode.

## First run, in order

1. **Calibration** (*Settings*). Tap "Drive 3 rotations", measure how far the robot went, enter it and save. Without this, distances on the map will be wrong.
2. **Direction** (*Joystick*). Push up: the robot should drive forward, and the arrow on the map should turn the same way as the robot.
   - Drives backwards: see "Troubleshooting".
   - The arrow on the map turns the wrong way: *Settings → Gyro → Reversed*.
3. **Map** (*Settings*). Photograph your drawn map straight from above, upload it, enter its **real width in centimetres** and tap "Save map".
4. **Rooms and corridors** (*Rooms* tab → "Edit map"). You only do this once:
   - **Corridor.** Place points along the middle of the corridors. Each new point joins the previous one with a line. To branch into another corridor, tap an existing point and continue from it. "Start a new line" begins a separate line. The robot drives **only along these lines**, so they must not cross walls.
   - **Room.** Tap a room door and type its number (301, 302, 303…). Tapping an existing room renames it.
   - **Sticker.** Mark every yellow sticker on the floor. When the robot drives over one, it corrects its position, so errors don't build up.
   - **Delete.** Tap a room, corridor point or sticker to delete it.
   - Tap "Done".
5. **Robot position.** Tap "Set robot position", touch where the robot stands on the map and, without lifting your finger, drag toward where it faces. Do this each time you switch on, unless the robot is where it stopped last time.
6. **Go.** Tap a room on the map and confirm "Go", or tap the big number button. The robot picks the shortest way along the corridor.
   You can pick another room while it is driving: it turns and heads there from where it is.

## How it works

- **Position on the map** comes from the motor rotations (how far it drove) and the gyro (how far it turned). This is called odometry. Small errors build up, which is what the yellow stickers correct.
- **The way to a room.** The robot joins the nearest corridor line, follows the lines along the shortest path (like a navigation app) and turns off to the door. Between points it turns on the spot with both tracks, then drives straight while the gyro holds its heading.
- **After the joystick**, the robot still knows where it is, so you can pick a room straight away.
- **Obstacle closer than 15 cm.** On a route, the robot waits until the way is clear and carries on from the same place. With the joystick it won't drive forward, but backing up and turning still work.
- **The joystick is safe.** If the phone loses the connection for more than 0.6 seconds, the robot stops by itself.
- **Settings** are saved on the robot in `settings.json`; the map, rooms, corridors and stickers in `map.json`.
- **Messages** on the robot are in English. The phone page shows them in the language you chose.

## Troubleshooting

| Problem | What to do |
|---|---|
| The robot drives backwards | In `settings.json`, flip `"invert_motors"` to the opposite value and restart the program. |
| Left and right are swapped (joystick right turns left) | In `settings.json`, swap `"left_port": "D"` and `"right_port": "B"` and restart. |
| The arrow on the map turns the wrong way | *Settings → Gyro → Reversed*. |
| It drives further or shorter than it should | Repeat the calibration. |
| "Corridor points are not connected by lines" | In "Edit map → Corridor", join the lines into one network: tap a point on one line, then a point on the other. |
| The robot cuts a corner through a wall | Add a corridor point in the corner so the line follows the corridor. |
| Turns are inaccurate | Check the gyro is in port 4 and the robot stood still at start-up. |
| The page doesn't open | The phone and the EV3 must be on the same network. Check the address on the EV3 screen. |
| A yellow sticker is not noticed | The colour sensor should sit low over the floor (about 1 cm), and the sticker should be bright yellow and matte. |

`settings.json` appears after the settings are saved the first time. You can open and edit it in VS Code directly on the EV3.
