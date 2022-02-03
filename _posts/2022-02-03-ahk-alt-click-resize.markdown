---
layout: post
title: "AutoHotkey Alt-Click Window Resize"
date: 2022-02-03
Categories:
  AutoHotkey
---

Similar to the [AutoHotkey Alt-Click Window Drag]({% post_url 2022-02-03-ahk-alt-click-move %}) post, I was used to resizing a window by pressing Alt + Right mouse button to resize a window.

The changes to the window drag script are minimal ---

* Change the bind key from `!LButton` to `!RButton`
  * This includes all the press and releases of the mouse button
* Capture and store the initial mouse coordinates and window dimensions
  * Use these to resize the window accordingly

Here's a closer look at the main loop. [`WinMove`](https://www.autohotkey.com/docs/commands/WinMove.htm) is used to adjust the window dimensions by taking the initial window dimension (`win_w` and `win_h`), and subtracting off the difference between the initial mouse coordinates (`initial_mouse_x` and `initial_mouse_y`) with the current mouse coordinates (`mouse_x` and `mouse_y`).

The result is when the mouse moves by a certain number of pixels, the window dimension is also adjusted by the same number of pixels.

```AutoHotkey
      MouseGetPos, mouse_x, mouse_y
      WinMove ahk_id %id%, , , , win_w - (initial_mouse_x - mouse_x), win_h - (initial_mouse_y - mouse_y)
```

Here is the full script.

```AutoHotkey
; alt-click-resize.ahk
;
; When Alt + right mouse button is pressed, resize the window under the cursor,
; as long as the window is not maximized, or a vnc viewer.
;
; Cody Cziesler
#SingleInstance force
!RButton::

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

  ; Send Alt + right mouse button (if not ignoring)
  if (ignore) {
    Send {Alt down}{RButton down}
  }

  ; Get mouse and window position
  CoordMode, Mouse, Screen
  MouseGetPos, initial_mouse_x, initial_mouse_y
  WinGetPos, , , win_w, win_h, ahk_id %id%

  ; Reduce latency of movements
  SetWinDelay, 0

  ; Set the transparency
  if (!ignore) {
    WinActivate, ahk_id %id%
    WinSet Transparent, 150, ahk_id %id%
  }

  ; Loop until right button released
  loop {
    if !GetKeyState("RButton", "P") {
      break
    }
    if (!ignore) {
      MouseGetPos, mouse_x, mouse_y
      WinMove ahk_id %id%, , , , win_w - (initial_mouse_x - mouse_x), win_h - (initial_mouse_y - mouse_y)
    }
  }

  ; Reset the transparency
  WinSet Transparent, Off, ahk_id %id%

  ; Release Alt + right mouse button
  Send {Alt up}{RButton up}

  return
```
