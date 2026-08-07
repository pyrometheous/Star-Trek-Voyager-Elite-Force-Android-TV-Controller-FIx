
# Fix 8BitDo SN30 Pro Controller Support in Voyager Elite Force Android

These instructions fix full gamepad support for the **8BitDo SN30 Pro in X-Input mode** with **VoyagerSP / Star Trek: Voyager – Elite Force (`com.voyager.efsp`)** on Android 14.

The fix has two required parts:

1. Disable an Android compatibility restriction that breaks VoyagerSP's bundled SDL/HIDAPI controller initialization.
2. Tell VoyagerSP to use the SN30 Pro instead of the laptop keyboard, which SDL incorrectly enumerates as joystick 0.

---

## 1. Put the SN30 Pro in X-Input Mode

Turn the controller on using:

```text
Start + X
````

Connect it to Android over Bluetooth normally.

---

## 2. Connect to Android with Root ADB

From the terminal of your PC, I used Bazzite OS and konsole:

```bash
adb connect DEVICE_IP:PORT
```

Enable root ADB:

```bash
adb -s DEVICE_IP:PORT root
```

Verify root access:

```bash
adb -s DEVICE_IP:PORT shell id
```

You should see:

```text
uid=0(root)
```

---

## 3. Fix SDL/HIDAPI on Android 14

VoyagerSP targets a recent Android API but includes SDL/HIDAPI code that uses an older Android broadcast-receiver registration method.

Disable the Android compatibility rule for **VoyagerSP only**:

```bash
adb -s DEVICE_IP:PORT shell am compat disable 161145287 com.voyager.efsp
```

Expected result:

```text
Disabled change 161145287 for com.voyager.efsp.
```

This allows SDL's HIDAPI controller subsystem to initialize properly.

---

## 4. Force VoyagerSP to Use the SN30 Pro

On this system SDL enumerates the devices as:

```text
Joystick 0 = Translated Set 2 keyboard
Joystick 1 = 8BitDo SN30 Pro
```

VoyagerSP defaults to:

```cfg
seta in_joystickNo "0"
```

which causes it to open the laptop keyboard instead of the actual gamepad.

The fix is to force joystick 1.

The game directory is:

```text
/storage/emulated/0/Android/obb/com.voyager.efsp/baseEF
```

Back up the existing `autoexec.cfg`:

```bash
adb -s DEVICE_IP:PORT shell 'cp /storage/emulated/0/Android/obb/com.voyager.efsp/baseEF/autoexec.cfg /storage/emulated/0/Android/obb/com.voyager.efsp/baseEF/autoexec.cfg.backup'
```

Then append the controller override:

```bash
adb -s DEVICE_IP:PORT shell 'printf "\nseta in_joystickNo \"1\"\n" >> /storage/emulated/0/Android/obb/com.voyager.efsp/baseEF/autoexec.cfg'
```

The important final line is:

```cfg
seta in_joystickNo "1"
```

VoyagerSP executes `autoexec.cfg` after `efspconfig.cfg`, so this overrides the incorrect saved joystick selection every time the game starts.

---


## 5. Restart VoyagerSP

Fully stop and relaunch the game:

```bash
adb -s DEVICE_IP:PORT shell 'am force-stop com.voyager.efsp; monkey -p com.voyager.efsp -c android.intent.category.LAUNCHER 1 >/dev/null 2>&1'
```

Keep the SN30 Pro connected while the game launches.

---

# Result

After both fixes are applied:

* Left analog stick works for movement.
* Right analog stick works for looking.
* Triggers work.
* Shoulder buttons work.
* D-pad works.
* Face buttons work.
* Start/Select work.
* VoyagerSP recognizes the SN30 Pro through SDL GameController.
* The existing `PAD0_*` bindings in `efspconfig.cfg` work correctly.

---

# Important X-Input Button Note

The SN30 Pro has Nintendo-style physical button labels, while X-Input uses Xbox button positions.

Therefore in menus:

```text
Physical B → SDL/Xbox A → Confirm / Select
Physical A → SDL/Xbox B → Back / Escape
```

This is normal for the SN30 Pro in X-Input mode.

---

# Complete Fix Summary

Run:

```bash
adb connect DEVICE_IP:PORT

adb -s DEVICE_IP:PORT root

adb -s DEVICE_IP:PORT shell am compat disable 161145287 com.voyager.efsp

adb -s DEVICE_IP:PORT shell 'printf "\nseta in_joystickNo \"1\"\n" >> /storage/emulated/0/Android/obb/com.voyager.efsp/baseEF/autoexec.cfg'

adb -s DEVICE_IP:PORT shell 'am force-stop com.voyager.efsp; monkey -p com.voyager.efsp -c android.intent.category.LAUNCHER 1 >/dev/null 2>&1'
```

The two settings that actually fix the problem are:

```text
Android:
am compat disable 161145287 com.voyager.efsp
```

and:

```cfg
VoyagerSP:
seta in_joystickNo "1"
```

No custom Android `.kl` file and no controller rebinding are required.

---

## NOTE: This guide was written by an LLM
