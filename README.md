Ubuntu 24.04 gtk4 tweaks
========================
[Ubuntu 24.04.2 LTS (noble)](https://discourse.ubuntu.com/t/ubuntu-24-04-lts-noble-numbat-release-notes/39890)
uses ([GNOME 46](https://release.gnome.org/46/)) and GNOME 4 apps run on top of the [gtk4](https://docs.gtk.org/gtk4/) toolkit.

[GNOME](https://en.wikipedia.org/wiki/GNOME) supports different types of devices and screen resolutions.
In this scenario the target system is a PC with a single GNOME Workspace on a
100dpi screen. The screen provides enough horizontal space, to work with a
single taskbar which includes the active window list.

In the past the system used the GNOME Flashback Desktop Environment [https://en.wikipedia.org/wiki/GNOME#GNOME_Flashback]
on GNOME 3, which provides a similar look & feel to the preceeding GNOME 2 version.
GNOME Flashback consists of the [Metacity Window Manager](https://wiki.gnome.org/Projects(2f)Metacity.html)
and the [GNOME Panel](https://wiki.gnome.org/Projects(2f)GnomePanel.html) taskbar.

GNOME Panel left                 | GNOME Panel right
---------------------------------|----------------------------------
![](assets/gnome_panel_left.png) | ![](assets/gnome_panel_right.png)

Metacity is a gtk3 X11 window manager and will not support Wayland.
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

- GNOME Panel has been replaced since GNOME 3 with [GNOME Shell](https://en.wikipedia.org/wiki/GNOME_Shell),
  which only works with the [Mutter window manager](https://en.wikipedia.org/wiki/Mutter_(software))

- Mutter is not compatible with old GNOME Panel addons, but it is highly
  configurable via [GNOME Extensions](https://extensions.gnome.org/).
  ```
  $ sudo apt install gnome-shell-extensions
  ```
  The extensions are installed and managed via the [Extension Manager](https://github.com/mjakeman/extension-manager).
  ```
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
  replaces the old [GNOME Panel](https://gitlab.gnome.org/GNOME/gnome-applets) gweather applet.
  ```
  $ sudo apt install gnome-weather
  ```

GNOME Dash-to-Panel+ArcMenu left              | GNOME Dash-to-Panel+ArcMenu right
----------------------------------------------|-----------------------------------------------
![](assets/gnome_dash_panel_arcmenu_left.png) | ![](assets/gnome_dash_panel_arcmenu_right.png)

