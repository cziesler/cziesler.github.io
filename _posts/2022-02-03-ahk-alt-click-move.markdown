---
layout: post
title: "AutoHotkey Alt-Click Window Drag"
date: 2022-02-03
Categories:
  AutoHotkey
---

[AutoHotkey](https://www.autohotkey.com/) is an automation scripting language for Windows. It can be used to create macros and hotkeys, create autocorrect replacements, and even GUIs.

The following script mimics how Linux can move windows around by Alt+Clicking anywhere on the window. It also ignores maximized windows and VNC viewers.

The first part ensures that the script only [runs once](https://www.autohotkey.com/docs/commands/_SingleInstance.htm). It then assigns a hotkey to this script, specifically the Alt button (`!`)and the left mouse button (`LButton`).

```AutoHotkey
#SingleInstance force
!LButton::
```

Next, [`MouseGetPos`](https://www.autohotkey.com/docs/commands/MouseGetPos.htm) is used to get a window ID for the window under the cursor.

```AutoHotkey
  ; Get window under cursor
  MouseGetPos, , , id
```

Then, some abort logic occurs --- if the window is maximized or the current app is a VNC viewer, do not move the window. Maximized windows cannot be moved, since they are maximized. VNC viewers should also be ignored, since typically I'm running Linux and want to use their window managing system.

To determine if a window is maximized, I use [`WinGet`](https://www.autohotkey.com/docs/commands/WinGet.htm) with the `MinMax` subcommand to get the minimized/maximized state.

To figure out if an application is a VNC viewer, I use [`WinGetClass`](https://www.autohotkey.com/docs/commands/WinGetClass.htm) to retrieve the window's class. I then use [`InStr`](https://www.autohotkey.com/docs/commands/InStr.htm) to see if "vnc" is within the class name.

If either of the above is true, the `ignore` flag is set to true. Rather than aborting the script, I chose to pass the Alt + left mouse button through. When those buttons are released, I then release the buttons to the OS (later in the script).

```AutoHotkey
  ; Abort if window is maximized
  ignore := flase
  WinGet, window_minmax, MinMax, ahk_id %id%
  if (window_minmax <> 0) {
    ignore := true
  }

  ; Abort if app is a vnc viewer
  WinGetClass, class, A
  if (InStr(class, "vnc")) {
    ignore := true
  }

  ; Send Alt + left mouse button (if not ignoring)
  if (ignore) {
    Send {Alt down}{LButton down}
  }
```

After that, I do some initial setup to get the mouse position relative to the screen ([`CoordMode`](https://www.autohotkey.com/docs/commands/CoordMode.htm) and [`MouseGetPos`](https://www.autohotkey.com/docs/commands/MouseGetPos.htm)). [`WinGetPos`](https://www.autohotkey.com/docs/commands/WinGetPos.htm) is then used to get the X and Y coordinates of the window's upper left corner relative to the screen. I then adjust the window coordinates by the mouse coordinates to get the initial window position relative to the mouse cursor.

The [`SetWinDelay`](https://www.autohotkey.com/docs/commands/SetWinDelay.htm) command causes the movement to occur immediately.

I then activate the window with [`WinActivate`](https://www.autohotkey.com/docs/commands/WinActivate.htm) and set the transparency with [`WinSet`](https://www.autohotkey.com/docs/commands/WinSet.htm) to show that the window is currently being moved.

```AutoHotkey
  ; Get mouse and window position
  CoordMode, Mouse, Screen
  MouseGetPos, mouse_x, mouse_y
  WinGetPos, win_x, win_y, , , ahk_id %id%

  ; Change win_x/win_y based on initial mouse position
  win_x := mouse_x - win_x
  win_y := mouse_y - win_y

  ; Reduce latency of movements
  SetWinDelay, 0

  ; Set the transparency
  if (!ignore) {
    WinActivate, ahk_id %id%
    WinSet Transparent, 150, ahk_id %id%
  }
```

The main loop will continuously run until the left mouse button was released ([`GetKeyState`](https://www.autohotkey.com/docs/commands/GetKeyState.htm)). If the mouse wasn't released and as long as we are not ignoring the hotkey, the window is moved by taking the mouse position ([`MouseGetPos`](https://www.autohotkey.com/docs/commands/MouseGetPos.htm)) and subtracting the window position from it -- that coordinate is then passed into [`WinMove`](https://www.autohotkey.com/docs/commands/WinMove.htm) to perform the movement.

```AutoHotkey
  ; Loop until left button released
  loop {
    if !GetKeyState("LButton", "P") {
      break
    }
    if (!ignore) {
      MouseGetPos, mouse_x, mouse_y
      WinMove ahk_id %id%, , (mouse_x - win_x), (mouse_y - win_y)
    }
  }
```

The final part disables the transparency with [`WinSet`](https://www.autohotkey.com/docs/commands/WinSet.htm), then releases the Alt + left mouse button with [`Send`](https://www.autohotkey.com/docs/commands/Send.htm). The [`return`](https://www.autohotkey.com/docs/commands/Return.htm) exits the script.

```AutoHotkey
  ; Reset the transparency
  WinSet Transparent, Off, ahk_id %id%

  ; Release Alt + left mouse button
  Send {Alt up}{LButton up}

  return
```

Here is the full script.

```AutoHotkey
; alt-click-move.ahk
;
; When Alt + left mouse button is pressed, move the window under the cursor,
; as long as the window is not maximized, or a vnc viewer.
;
; Cody Cziesler
#SingleInstance force
!LButton::

  ; Get window under cursor
  MouseGetPos, , , id

  ; Abort if window is maximized
  ignore := flase
  WinGet, window_minmax, MinMax, ahk_id %id%
  if (window_minmax <> 0) {
    ignore := true
  }

  ; Abort if app is a vnc viewer
  WinGetClass, class, A
  if (InStr(class, "vnc")) {
    ignore := true
  }

  ; Send Alt + left mouse button (if not ignoring)
  if (ignore) {
    Send {Alt down}{LButton down}
  }

  ; Get mouse and window position
  CoordMode, Mouse, Screen
  MouseGetPos, mouse_x, mouse_y
  WinGetPos, win_x, win_y, , , ahk_id %id%

  ; Change win_x/win_y based on initial mouse position
  win_x := mouse_x - win_x
  win_y := mouse_y - win_y

  ; Reduce latency of movements
  SetWinDelay, 0

  ; Set the transparency
  if (!ignore) {
    WinActivate, ahk_id %id%
    WinSet Transparent, 150, ahk_id %id%
  }

  ; Loop until left button released
  loop {
    if !GetKeyState("LButton", "P") {
      break
    }
    if (!ignore) {
      MouseGetPos, mouse_x, mouse_y
      WinMove ahk_id %id%, , (mouse_x - win_x), (mouse_y - win_y)
    }
  }

  ; Reset the transparency
  WinSet Transparent, Off, ahk_id %id%

  ; Release Alt + left mouse button
  Send {Alt up}{LButton up}

  return
```

## References

* [Alt+LButton window dragging](https://www.autohotkey.com/board/topic/25106-altlbutton-window-dragging/)
