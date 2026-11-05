# Fix-Flickering-MPO

Fix screen flickering on Windows 11 and 10 by turning off **Multiplane Overlay (MPO)**. It's one registry value and one restart, and it's fully reversible.

I made this after **AdGuard for Windows** started making my screen flicker. This change fixed it, and AdGuard keeps running as normal.

**Download:** [Fix-Flickering-MPO.reg](https://github.com/LeerBox/Fix-Flickering-MPO/releases/latest/download/Fix-Flickering-MPO.reg) · [Undo-Fix-Flickering-MPO.reg](https://github.com/LeerBox/Fix-Flickering-MPO/releases/latest/download/Undo-Fix-Flickering-MPO.reg)

---

## The problem

With AdGuard for Windows running, the screen flickers.

Other apps can cause the same kind of flicker. It usually looks like one of these:

- flickering or black flashes on the desktop or inside an app window
- flicker or stutter while moving or resizing windows
- flicker in videos, browsers, or streaming and remote desktop windows
- problems that only show up with more than one monitor, especially at different refresh rates

If one of these matches what you see, MPO is the usual cause.

## Why it happens

Normally the **Desktop Window Manager (DWM)**, the part of Windows that draws the desktop, combines all your windows into one image and sends it to the monitor.

With **Multiplane Overlay (MPO)**, Windows hands part of that job to the graphics card's display hardware instead. The card gets separate layers, called planes, such as a video and the rest of the desktop, and combines them as the picture goes out to the monitor. That saves power and lowers latency. But some combinations of graphics card, driver, monitor and app don't handle it well, and the result is flicker.

It's a known problem. NVIDIA ships this same registry value (as `mpo_disable.reg`) as its official workaround: [NVIDIA support article 5157](https://nvidia.custhelp.com/app/answers/detail/a_id/5157).

In my case, AdGuard is what set it off. The fix doesn't change anything in AdGuard. It turns MPO off, so you don't need to uninstall or reconfigure anything.

## What the fix does

[`Fix-Flickering-MPO.reg`](Fix-Flickering-MPO.reg) is four lines:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=dword:00000005
```

- `Windows Registry Editor Version 5.00` is the header that tells Windows this is a registry file.
- `[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]` is the Desktop Window Manager's settings key. It's machine-wide, so it applies to every user and needs administrator rights.
- `"OverlayTestMode"=dword:00000005` creates a 32-bit value named `OverlayTestMode`, set to `5`, which tells DWM not to use Multiplane Overlay.

That's the whole change. Nothing is deleted or installed, and nothing runs in the background.

## Apply the fix

1. Download [Fix-Flickering-MPO.reg](https://github.com/LeerBox/Fix-Flickering-MPO/releases/latest/download/Fix-Flickering-MPO.reg).
2. Double-click it. Click **Yes** in the User Account Control prompt, **Yes** to confirm, then **OK**.
3. **Restart your PC.** The change takes effect after a restart.

Your browser may warn that `.reg` files can harm your device. That's a generic warning for registry files. You can open this one in Notepad and check that it contains only the four lines above.

Or, from a terminal opened as administrator:

```bat
reg add "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayTestMode /t REG_DWORD /d 5 /f
```

## Undo the fix

1. Download [Undo-Fix-Flickering-MPO.reg](https://github.com/LeerBox/Fix-Flickering-MPO/releases/latest/download/Undo-Fix-Flickering-MPO.reg).
2. Double-click it, then **Yes**, **Yes**, **OK**.
3. Restart your PC.

It deletes the `OverlayTestMode` value, which puts Windows back to its default with MPO on. From an administrator terminal:

```bat
reg delete "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayTestMode /f
```

## Check if the fix is on

```bat
reg query "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayTestMode
```

- `OverlayTestMode    REG_DWORD    0x5` means the fix is on.
- `ERROR: The system was unable to find the specified registry key or value.` means the fix is off (the Windows default).

Don't go by `dxdiag`. Its `MPO MaxPlanes` line can still show a number above 1 after the fix. On the PC below it still shows 3, and the flicker is gone.

## Side effects

- **Slightly more GPU work during video playback**, because Windows draws the desktop the regular way again. On a desktop PC you won't notice it. On a laptop, battery life during video can drop a little.
- **Windowed and borderless games** can get a little more input lag while an overlay, such as Discord, Steam or a notification, is drawn on top of them.

AdGuard and your other apps keep working as before.

## Tested on

| | |
|---|---|
| Windows | Windows 11 Pro 26H2, build 26300.9818 |
| Graphics card | NVIDIA GeForce RTX 5090, driver 621.14 |
| Monitors | 2 (main monitor at about 280 Hz) |
| App that caused the flicker | AdGuard for Windows 8.0.5595 |

## Still flickering?

- Make sure you restarted after applying the fix.
- Run the check above and confirm it shows `0x5`.
- Some people report that `OverlayTestMode` has no effect on certain Windows 11 24H2 and 25H2 builds. It works on the build listed above.
- If nothing changes, your flicker probably has a different cause. Run the undo file to put Windows back to default, and try updating your graphics driver.

## FAQ

**Is it safe?**
It changes one registry value, it's the same workaround NVIDIA provides, and the undo file removes it completely.

**Do I need an NVIDIA graphics card?**
No. MPO is a Windows feature and Windows itself reads this value, so it applies to NVIDIA, AMD and Intel graphics alike.

**Do I have to uninstall AdGuard?**
No. AdGuard keeps running. Only MPO is turned off.

**Will it survive updates?**
It's a normal registry value, so it should. If the flicker comes back, for example after a major Windows update, run the check above and apply the fix again if needed.

## Credits

- [LeerBox](https://github.com/LeerBox) found that this fixes the AdGuard flicker, tested it, and made this repo.
- NVIDIA published the `OverlayTestMode` value as an official workaround in [support article 5157](https://nvidia.custhelp.com/app/answers/detail/a_id/5157).

## License

[MIT](LICENSE)
