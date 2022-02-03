---
layout: post
title: "AutoHotkey Always On Top"
date: 2022-02-03
categories:
  AutoHotkey
---

This script will toggle whether a window will always be on top when the Ctrl + Space key is pressed. It will also display a [`ToolTip`](https://www.autohotkey.com/docs/commands/ToolTip.htm).

The first part of the script uses [`WinSet`](https://www.autohotkey.com/docs/commands/WinSet.htm) with the `AlwaysOnTop` subcommand to toggle the always-on-top status.

```AutoHotkey
  ; Toggle AlwaysOnTop
  WinSet, Alwaysontop, , A
```

The last part displays a [`ToolTip`](https://www.autohotkey.com/docs/commands/ToolTip.htm) that displays whether the current window has the AlwaysOnTop flag set and the window title.

First, [`WinGetTitle`](https://www.autohotkey.com/docs/commands/WinGetTitle.htm) is used to get the window title. Then, [`WinGet`](https://www.autohotkey.com/docs/commands/WinGet.htm) is used to get a bitmask of the `ExStyle` --- bit 4 contains whether the `AlwaysOnTop` flag is set. Either way, [`ToolTip`](https://www.autohotkey.com/docs/commands/ToolTip.htm) will display a message for 500 ms. Calling `ToolTip` with no message will remove the popup.

```AutoHotkey
  ; Get the window title
  WinGetTitle, title, A

  ; Use ExStyle to figure out if AlwaysOnTop is set, and display a ToolTip
  WinGet, ExStyle, ExStyle, A

  if (ExStyle & 0x8) {
    ToolTip, AlwaysOnTop - "%title%"
  } else {
    ToolTip, Not AlwaysOnTop - "%title%"
  }

  ; Wait 0.5 seconds and disable the ToolTip
  Sleep 500
  ToolTip
```

Here's the full script.

```AutoHotkey
#SingleInstance force
^SPACE::
  ; Toggle AlwaysOnTop
  WinSet, Alwaysontop, , A

  ; Get the window title
  WinGetTitle, title, A

  ; Use ExStyle to figure out if AlwaysOnTop is set, and display a ToolTip
  WinGet, ExStyle, ExStyle, A

  if (ExStyle & 0x8) {
    ToolTip, AlwaysOnTop - "%title%"
  } else {
    ToolTip, Not AlwaysOnTop - "%title%"
  }

  ; Wait 0.5 seconds and disable the ToolTip
  Sleep 500
  ToolTip

  return
```
