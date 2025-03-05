Ubuntu 24.04 gtk4 tweaks
========================
[Ubuntu 24.04.2 LTS (noble)](https://discourse.ubuntu.com/t/ubuntu-24-04-lts-noble-numbat-release-notes/39890)
uses [GNOME 46](https://release.gnome.org/46/) and GNOME 4 apps run on top of the [gtk4](https://docs.gtk.org/gtk4/) toolkit.

[GNOME](https://en.wikipedia.org/wiki/GNOME) supports different types of devices and screen resolutions.
In this scenario the target system is a PC with a single [GNOME Workspace](https://help.gnome.org/users/gnome-help/stable/shell-workspaces.html.en) on a
100dpi screen. The screen provides enough horizontal space to work with a
single taskbar which includes the active window list.

In the past the system used the [GNOME Flashback](https://en.wikipedia.org/wiki/GNOME#GNOME_Flashback) desktop environment
on GNOME 3, which provides a similar look & feel to the preceeding GNOME 2 version.
GNOME Flashback consists of the [Metacity Window Manager](https://wiki.gnome.org/Projects(2f)Metacity.html)
and the [GNOME Panel](https://wiki.gnome.org/Projects(2f)GnomePanel.html) taskbar.

GNOME Panel left                 | GNOME Panel right
---------------------------------|----------------------------------
![](assets/gnome_panel_left.png) | ![](assets/gnome_panel_right.png)

Metacity is a [gtk3](https://docs.gtk.org/gtk3/) X11 window manager and will not support [Wayland](https://en.wikipedia.org/wiki/Wayland_(protocol)).
The Metacity Window Manager is still available for Ubuntu 24.04. But
to support modern apps and features the system was changed to use a
gtk4 environment with a similar user experience to GNOME Flashback.

Switch from GNOME Flashback to GNOME 4 Classic
----------------------------------------------
Different Ubuntu Desktop Environments can be selected from the login menu
after a logout from the current session.

The best choice on Ubuntu 24.04 is GNOME Classic on Xorg. Even if Wayland
is supposed to replace X11 in Linux, the GNOME Classic on Wayland session
has still too many issues.

- GNOME Panel has been replaced with [GNOME Shell](https://en.wikipedia.org/wiki/GNOME_Shell) in GNOME 4,
  which only works with the [Mutter window manager](https://en.wikipedia.org/wiki/Mutter_(software))

- Mutter is not compatible with old GNOME Panel addons, but it is highly
  configurable via [GNOME Extensions](https://extensions.gnome.org/).
  The extensions are installed and managed via the [Extension Manager](https://github.com/mjakeman/extension-manager).
  ```
  $ sudo apt install gnome-shell-extensions
  $ sudo apt install gnome-shell-extension-manager
  ```

The best solution for a single taskbar on a GNOME 4 desktop are the extensions:
- [Dash-to-Panel](https://extensions.gnome.org/extension/1160/dash-to-panel/)
- [ArcMenu](https://extensions.gnome.org/extension/3628/arcmenu/)
  ```
  -> Menu->Menu Layout: Traditional Gnome Menu
  -> Menu->Tweaks->Category Activation: Mouse Hover
  -> General->Position: Left
  ```
- the menu entries can be managed via the [Alacarte](https://en.wikipedia.org/wiki/Alacarte) menu editor.
  ```
  $ sudo apt install alacarte
  ```
- for CPU temperatures [Freon](https://extensions.gnome.org/extension/841/freon/)
  replaces the old [sensor-applet](https://help.ubuntu.com/community/SensorInstallHowto) for the GNOME Panel.
  ```
  $ sudo apt install lm-sensors
  ```
- to display the current weather next to the clock [Weather O'Clock](https://extensions.gnome.org/extension/5470/weather-oclock/)
  replaces the old GNOME Panel [gweather applet](https://wiki.gnome.org/Projects/GnomeApplets).
  ```
  $ sudo apt install gnome-weather
  ```

GNOME Dash-to-Panel+ArcMenu left              | GNOME Dash-to-Panel+ArcMenu right
----------------------------------------------|-----------------------------------------------
![](assets/gnome_dash_panel_arcmenu_left.png) | ![](assets/gnome_dash_panel_arcmenu_right.png)

Allow window positioning for gtk4 startup applications
------------------------------------------------------
The gtk4 runtime does not support to set the window position anymore.
- gtk4 is modelled on Wayland, which (unlike X11) does not expose toplevel positions to clients.
- there is no longer a way to do this in gtk4, all window management is done through the window manager.
  the window manager positions top level windows by its own strategy (to avoid some security risks).
- the gtk_window_move() API from gtk3 does not work under Wayland, and it has been removed in gtk4.
  - [gtk_window_move](https://docs.gtk.org/gtk3/method.Window.move.html)
  - [Adapt to GdkWindow API changes](https://docs.gtk.org/gtk4/migrating-3to4.html#adapt-to-gdkwindow-api-changes)
  - [Adapt to GtkWindow API changes](https://docs.gtk.org/gtk4/migrating-3to4.html#adapt-to-gtkwindow-api-changes)
- this type of operation is only possible through manipulation via the GNOME Shell.
- the geometry option like `--geometry=100x50+10+10` of older X11 applications will ignore the X/Y-position.

The [Windows Calls](https://github.com/ickyicky/window-calls) extension provides access to the
Windows.Move API of the GNOME Shell, and can be called via [D-Bus](https://en.wikipedia.org/wiki/D-Bus) messages.
```
$ gdbus call --session --dest org.gnome.Shell --object-path /org/gnome/Shell/Extensions/Windows --method org.gnome.Shell.Extensions.Windows.List | python3 -c 'import sys; a=eval(sys.stdin.read()); print (a[0])' | jq '.[] | select(.wm_class_instance=="ghostty")'
  {
    "in_current_workspace": true,
    "wm_class": "com.mitchellh.ghostty",
    "wm_class_instance": "ghostty",
    "title": "~",
    "pid": 3873,
    "id": 3149279314,
    "frame_type": 0,
    "window_type": 0,
    "focus": false
  }
$ gdbus call --session --dest org.gnome.Shell --object-path /org/gnome/Shell/Extensions/Windows --method org.gnome.Shell.Extensions.Windows.Move 3149279314 10 10
```
It is very useful to have an option to set the position of startup windows.
Here is a simple [start_win_at](scripts/start_win_at) shell script, that takes the
window position, the application name and parameters as argument, and can be called
from the autostart folder.
```
$ start_win_at 10 10 ghostty --window-width=100 --window-height=50 --theme="Adwaita Dark"
```
The script works for any application opening a single window and has to
1) wait for the Window Calls extension to be available on startup,
2) start the application,
3) find the window id as soon as the window is displayed, and
4) move the window to the requested position.

Note: There are extensions like [Another Window Session Manager](https://extensions.gnome.org/extension/4709/another-window-session-manager/)
that can restore window size and positions, but this is not the same like a clean startup configuration.

To run an application when a GNOME session starts, a script like [terminal1.desktop](autostart/terminal1.desktop)
has to be placed in the `.config/autostart` folder of the home directory. The command itself must be avaible in the shell `PATH` (add the scripts folder to the PATH).

Note: It is also possible to manage startup-applications with the [gnome-tweaks](https://gitlab.gnome.org/GNOME/gnome-tweaks) tool.
But on GNOME 4 gnome-tweaks has some issues and sometimes overrides old entries.

Workaround for broken gtk3 window resize
----------------------------------------
Old GNOME 3 gtk3 apps are still supported in the GNOME 4 environment.
But in the Ubuntu 24.04 Xorg session the gtk3 windows are not resizable after
startup. This is a known issue and can be resolved by the command:
```
$ pkill -HUP mutter-x11-fram
```
see: [Mutter on X11 with Gnome 46, some windows no shadows, not resizable](https://gitlab.gnome.org/GNOME/mutter/-/issues/3417)

The script [fix_gtk3_win_resize](scripts/fix_gtk3_win_resize) waits for Mutter to be
available before the `SIGHUP` is send. To run this script when a GNOME session starts,
a autostart script like [fix_gtk3_win_resize.desktop](autostart/fix_gtk3_win_resize.desktop)
has to be placed in the `.config/autostart` folder of the home directory.

Fix appearance of gtk3 applications
-----------------------------------
GNOME 4 tries to provide the Look & Feel of gtk4 apps to old gtk3 apps and
[snap](https://en.wikipedia.org/wiki/Snap_(software)) applications like Firefox & Thunderbird.
One difference is that just the upper corners of the windows are rounded.

To add rounded corners to all windows the [Rounded Window Corners Reborn](https://extensions.gnome.org/extension/7048/rounded-window-corners-reborn/)
extension can be installed (see: https://github.com/flexagoon/rounded-window-corners).

Improve font rendering in terminal emulator
-------------------------------------------
GNOME Classic uses the `DejaVu Sans Mono` font as standard `Monospace` console font.
```
$ fc-match Monospace
  DejaVuSansMono.ttf: "DejaVu Sans Mono" "Book"
```
The font-size and antialiasing can be changed in the gnone-tweaks font dialog. For a
100dpi IPS LCD screen a 13pt font-size and grayscale antialiasing seems to be the best choice
for the GNOME `Adwaita Dark` theme with light text on a dark background. The
subpixel-antialiasing shows slight vertical color-shadows on some chars.

grayscale antialiasing | subpixel antialiasing | no antialiasing
-----------------------|-----------------------|-----------------
![](assets/terminal_dejavu_sans_mono_regular_13pt(17px)_spc10x21_aa=grayscale.png) | ![](assets/terminal_dejavu_sans_mono_regular_13pt(17px)_spc10x21_aa=lcd.png) | ![](assets/terminal_dejavu_sans_mono_regular_13pt(17px)_spc10x21_aa=none.png)

Another aspect which affects readability is the vertical space between text lines.
In gnome-terminal the 13pt Monospace font uses a default cell-size of 10x21 pixel
for each character. In the gnome-terminal Preferences dialog it is only possible to
increase the cell-spacing of a selected custom font.

Since the 13pt Monospace font uses a quite large vertical spacing, a reduction by
1 pixel is reasonable. This can be accomplished by a different terminal emulator
with more configuration options like [ghostty](https://ghostty.org/docs).
```
$ ghostty --theme="Adwaita Dark" --font-family="DejaVu Sans Mono" --font-size=13 --adjust-cell-height=0
```
DejaVu Sans Mono 13pt (10x20 cell) | DejaVu Sans Mono 12pt (10x19 cell)
-----------------------------------|------------------------------------
![](assets/ghostty_dejavu_sans_mono_regular_13pt(17px)_spc10x20.png) | ![](assets/ghostty_dejavu_sans_mono_regular_12pt(16px)_spc10x19.png)

Using a terminal emulator with its own font rendering has also the advantage, that the
global system settings do not affect the console font and can keep using subpixel-antialiasing
(which looks better for proportional fonts). ghostty uses grayscale-antialiasing as a default.

For comparision the `JetBrains Mono` font which is [configured as the ghostty default font](https://ghostty.org/docs/config) is also a good console font. For the 12pt font ghostty uses
a cell-size of 10x22 with a quite large vertical spacing. But this can easily be configured
to use a cell-size of 10x20.
```
$ ghostty --theme="Adwaita Dark" --adjust-cell-height=-2
```
JetBrains Mono 12pt (10x22 cell) | JetBrains Mono 12pt (10x20 cell)
-----------------------------------|--------------------------------
![](assets/ghostty_jetbrains_mono_12pt(16px)_spc10x22.png) | ![](assets/ghostty_jetbrains_mono_12pt(16px)_spc10x20.png)

### Comparision with GNOME Flashback Ubuntu Mono fonts

The old GNOME Flashback session used the `Ubuntu Mono` console font. This font is still
available on Ubuntu 24.04, but it looks slightly different.

[Ubuntu 24.04 updated the Ubuntu font family](https://discourse.ubuntu.com/t/ubuntu-24-04-lts-noble-numbat-release-notes/39890#p-99950-updated-ubuntu-font)
to a slimmer version. The new terminal font has the same 'Ubuntu Mono' name as the old 2011
font from Ubuntu 22.04 (see: [Ubuntu font](https://design.ubuntu.com/font) & [Ubuntu Sans Mono Font Family](https://github.com/canonical/ubuntu-sans-mono-fonts)).

It is possible to switch back and forth between the current and the old console font
from Ubuntu 22.04 (note that sometimes gnome-terminal shows just 'boxes' after the
switch and a reboot is required).
(see: [How to Restore the Old Fonts in Ubuntu 24.04 LTS](https://ubuntuhandbook.org/index.php/2023/04/restore-old-fonts-ubuntu-2304/))
```
$ sudo apt install fonts-ubuntu-classic
$ sudo apt install fonts-ubuntu
```
The classic Ubuntu Mono font is smaller for the same font-size, and also uses a smaller
cell-spacing. So terminal dimensions will differ when switching between the fonts. The
classic Ubuntu Mono font looks good in 12pt and 13pt sizes, but looks blurry with 14pt.

size | classic Ubuntu Mono | 24.04 Ubuntu Mono
-----|---------------------|-------------------
12pt | ![](assets/ghostty_ubuntu2204_mono_classic_12pt(16px)_spc8x16.png) | ![](ghostty_ubuntu2404_mono_12pt(16px)_wght400(reg)_spc9x18.png)
13pt | ![](assets/ghostty_ubuntu2204_mono_classic_13pt(17px)_spc9x18.png) | ![](assets/ghostty_ubuntu2404_mono_13pt(17px)_wght400(reg)_spc10x20.png)

