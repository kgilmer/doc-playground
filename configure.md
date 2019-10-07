---
layout: page
title: Configure
---

Configuration files relating to the look and feel of Regolith are stored in `/etc/regolith/`.  For example, the i3 config file is at `/etc/regolith/i3/config`, and the root Xresources file is at `/etc/regolith/styles/root`.  These files are read-only for the user and are not intended to be modified.  If you find that you'd like to modify a config file, the first step is to copy the default into your user directory under the `~/.config` directory, or `~/.Xresources` for Xresources definitions.  Refer to the [table at the bottom of this page](#config-files) to determine where each user-defined config file should be placed in order for Regolith to pick it up automatically upon login.

### Simple Example

To change the default super key binding in i3 to `alt`~

1. Copy the default file into the user directory:

```
$ mkdir ~/.config/regolith/i3 
$ cp /etc/regolith/i3/config ~/.config/regolith/i3/
```

2. Edit your user copy to change the default mapping of the `$mod` variable.  Do this by editing `~/.config/regolith/i3/config` and changing `set $mod Mod4` to `set $mod Mod1`.

3. Restart i3.  This step is required only in the case that you wish i3 to load a new config file.  If you've already done the copy operation in step #1 in the past, simply reloading i3 interactively should suffice.

### Xresources Example

You may need to copy multiple files depending on what you want to change.  For example, Xresources is defined as a set of files that are loaded together.  In order to make a change to, say `typeface-ubuntu` you'll also need to copy the root Xresource file because it directly references `typeface-ubuntu`. 

1. Copy the Xresource files we want to change into the user directory:

```
$ cp /etc/regolith/styles/root ~/.Xresources-regolith # You can also use ~/.Xresources if you prefer
$ mkdir ~/.Xresources.d
$ cp /etc/regolith/styles/typeface-ubuntu ~/.Xresources.d/typeface-ubuntu
```

2. Modify the root file to refer to our user copy of `typeface-ubuntu` by editing `~/.Xresources-regolith` and updating the path of `typeface-ubuntu` to point to our user copy.  The edit should be to set to `#include ".Xresources.d/typeface-ubuntu"`.

3. Log out and back in to see changes take effect.  Alternatively, you can manually load the updated files with `xrdb` and then reload i3 interactively: `xrdb -merge ~/.Xresources-regolith`.

# Configuring Xresources

<sub>_See [quick steps](#quick-steps) below for just making changes quickly, or read this to get an understanding of how configuration works in Regolith._</sub>

Regolith Linux utilizes the [Xresource facility](https://en.wikipedia.org/wiki/X_resources#Location_and_use) to define colors, fonts, and themes.  These resources live can be customized in your home directory, and are designed to be easy to understand, modify, and reset.  The default Regolith Xresources file is at `/etc/regolith/styles/root` and to customize, copy it to `~/.Xresources-regolith`.  Here's what that file looks like:
```
! This is the Regolith root-level Xresources file.
!
! -- Styles - Colors
!
! Uncomment one and only one of the following color definitions: 
!#include "/etc/regolith/styles/color-solarized-dark"
!#include "/etc/regolith/styles/color-solarized-light"
!#include "/etc/regolith/styles/color-gruvbox"
#include "/etc/regolith/styles/color-nord"

! -- Styles - Fonts
! NOTE: Font packages may need to be installed when enabling typefaces.
! Uncomment one and only one of the following font definitions:
#include "/etc/regolith/styles/typeface-sourcecodepro"
!#include "/etc/regolith/styles/typeface-ubuntu"

! -- Styles - Theme
! NOTE: GTK theme and icon packages may need to be installed when enabling themes.
! Uncomment one and only one of the following theme definitions:
!
! The following theme requires solarc-theme and arc-icon-theme to be installed.
!#include "/etc/regolith/styles/theme-regolith"
!
! The following theme requires gnome-themes-extra-data and adwaita-icon-theme to be installed.
!#include "/etc/regolith/styles/theme-ubuntu-dark"
!
! The following theme requires nordic and paper-icon-theme to be installed
#include "/etc/regolith/styles/theme-nordic"

! -- Applications
! These files map values defined above into specific app settings.
#include "/etc/regolith/styles/st-term"
#include "/etc/regolith/styles/i3-wm"
#include "/etc/regolith/styles/i3xrocks"
#include "/etc/regolith/styles/rofi"
#include "/etc/regolith/styles/gnome"
```

This file is loaded into the Xresource database upon login and values are referenced by various programs, such as i3 and scripts, such as blocklets to load values rather than hardcode them directly.  This allows us to change a value in one place, say the primary font, and have that value propogate to all programs.  

However, you'll note that we don't have any actual typeface or color values in `~/.Xresources-regolith`.  There is a level of indirection that provides us the ability to swap out definition files without having to modify each value in place.  Specifically, we use the `#define` keyword to specify a literal value, and then map it to a specific application property in application-specific Xresource files such as `~/.Xresources.d/st-term`.  For example, the default color configuration, `.Xresources.d/color-solarized-dark` defines a key `color_red` and sets it to a specified RGB value.  Now if we look at `/etc/regolith/styles/color-gruvbox`, we can see the same key, but set to a different color value.  Then, if we look at `/etc/regolith/styles/i3-wm`, we can see that key mapped to specific UI elements, such as `i3-wm.client.urgent.color.background`.  In essense `color-solarized-dark` and `color-gruvbox` are two complimentary color definitions.  We can change all of them at once by toggling which resource file to load; by commenting and uncommenting files with the `!` character as needed.

As an example, if I want to set my colors to Solarized Light, I would make the folowing change to the file above:
```
! Uncomment one and only one of the following color definitions 
!#include ".Xresources.d/color-solarized-dark"
#include ".Xresources.d/color-solarized-light"
!#include ".Xresources.d/color-gruvbox"
!#include ".Xresources.d/color-nord"
```

I've simply commented the line that specifies Solarized Dark, meaning the resources will not be loaded, and replaced them with Solarized Light.  Because both files define the same keys with different values, this is the only change I need to make to change the color selection.

This same pattern applies to fonts (Xresources that are prefixed with `typeface-`) and GTK themes (Xresources that are prefixed with `theme-`).

In a similar matter, it is straightforward to add new settings without modifying existing definitions.  For example, to create a new color scheme, simply copy an existing definition:
```
$ mkdir ~/.Xresources.d
$ cp /etc/regolith/styles/color-solarized-dark ~/.Xresources.d/color-mything
```

Now edit `~/.Xresources.d/color-mything` and change the color definitions however you like.  To add them to the Xresources, change the root file to include your new file:
```
! Uncomment one and only one of the following color definitions 
!#include "/etc/regolith/styles/color-solarized-dark"
!#include "/etc/regolith/styles/color-solarized-light"
!#include "/etc/regolith/styles/color-gruvbox"
!#include "/etc/regolith/styles/color-nord"
#include ".Xresources.d/color-mything"
```

### Refreshing the UI

Currently, you must log out and back in to see changes to your Xresources, or you can perform these steps:
```
$ xrdb -merge ~/.Xresources-regolith
```
Now reload i3 and your updates should be visible.

# [Quick Steps](#quick-steps)

Before making changes, be sure to copy the default Regolith Xresources file to your user directory:
```
$ cp /etc/regolith/styles/root ~/.Xresources-regolith
```

## Change Color Theme

1. Edit `~/.Xresources-regolith`, add a `!` to the line `#include ".Xresources.d/color-solarized-dark"` and uncomment one of the other lines in the same section, say gruvbox by removing the `!` at the beginning of the line: `#include ".Xresources.d/color-gruvbox"`.
2. Run `xrdb -merge ~/.Xresources-regolith`.
3. Reload i3.

## Change Gnome Theme

1. Edit `~/.Xresources-regolith`, add a `!` to the line `#include ".Xresources.d/theme-regolith"` and uncomment one of the other lines in the same section, say Ubuntu Dark by removing the `!` at the beginning of the line: `#include ".Xresources.d/theme-ubuntu-dark"`.
2. Log out and back in.

<sub>Note: any Icon or GTK theme you specify in Xresources must be installed via `apt`.  For example the Ubuntu package `arc-icon-theme` must be installed if you wish to set `icon_theme` to Arc.</sub>

## Add a New Color Theme

1. Copy the new theme file into your `~/.Xresources.d` directory.  For example, if you have a color theme `color-sunday`, `cp color-sunday ~/.Xresources.d/`.
2. Reference your new color file in the Regolith Xresources file:
```
! -- Styles - Colors
!
! Uncomment one and only one of the following color definitions 
!#include ".Xresources.d/color-solarized-dark"
!#include ".Xresources.d/color-solarized-light"
!#include ".Xresources.d/color-gruvbox"
!#include ".Xresources.d/color-nord"
#include ".Xresources.d/color-sunday"
```
3. Run `xrdb -merge ~/.Xresources-regolith`.
4. Reload i3.

# [Config Files](#config-files)

Prior to August 2019, Regolith did not have a consistent way of updating theme information.  To update colors or typeface, several files with different syntax and rules had to be modified directly.  If you have done this, to use the Xresource system, you'll need to move your changes into Xresource files as described above.  

|---------|----------|------------------------|---|----|
| **Config** | **R1.0** | **R1.1** | **R1.2** | **User File**
| **i3** [(source)](https://github.com/regolith-linux/regolith-i3-gaps-config/blob/master/config) | `~/.config/i3-regolith/config` | `~/.config/i3-regolith/config-<version>` | `/etc/regolith/i3/config` | `~/.config/regolith/i3/config` 
| **Xresources** [(source)](https://github.com/regolith-linux/regolith-styles/blob/master/Xresources/root) | | | `/etc/regolith/styles/root` | `~/.Xresources-regolith`
| **Rofi** [(source)](https://github.com/regolith-linux/regolith-rofi-config/blob/master/regolith-theme.rasi) | hard-coded in i3 config | Defined in Xresources | `/etc/regolith/rofi/regolith-theme.rasi` | Defined in `/etc/regolith/styles/rofi` |
| **Conky** [(source)](https://github.com/regolith-linux/regolith-conky-config/blob/master/conky.config) | `/etc/xdg/conky/config` | `/etc/xdg/conky/config` | `/etc/regolith/conky/config` |
| **i3blocks** | `~/.config/i3-regolith/i3blocks.conf` | Deprecated | |
| **i3xrocks** [(source)](https://github.com/regolith-linux/regolith-i3xrocks-config/blob/master/i3xrocks.conf) | Not Used | `~/.config/i3-regolith/i3xrocks.conf` | `/etc/regolith/i3xrocks/config` | User defined, in i3 config.
| **compton** [(source)](https://github.com/regolith-linux/regolith-compton-config/blob/master/config) | | | `/etc/regolith/compton/config` | User defined, in i3 config.

