# Menus and file associations
- `.desktop` files - how to launch an application, which icon to use, where to put in the menu, etc
- `mimeapps.list` files - associates file types with applications.
- `.menu` files - Menu structure, categories

Reference: https://www.freedesktop.org/wiki/ 


## .desktop files
| Path                             | Usage                                           |
| -------------------------------- | ----------------------------------------------- |
| `~/.local/share/applications`    | user specific                                   |
| `/usr/share/applications/`       | system-wide (apps installed by package manager) |
| `/usr/local/share/applications/` | system-wide (apps installed manually)           |

Typical .desktop file
```
[Desktop Entry]
Type=Application
Version=1.0
Name=Htop
GenericName=Process Viewer
Comment=Show System Processes
Icon=htop
Exec=htop
Terminal=true
Categories=System;Monitor;ConsoleOnly;
Keywords=system;process;task
```
### Icon path
You can use a full path to the icon. If you don't use a full path, it will search recursively in default directories:

- `$HOME/.icons` (for backwards compatibility)
- `$XDG_DATA_DIRS/icons` i.e. `/usr/share/icons`
- `/usr/share/pixmaps`

**Note:** \
Typically, you have several "icon theme" subdirectories within `/usr/share/icons/`. Each theme has further subdirectories based on categories and icon sizes.
The current icon theme is searched first, if no icon found, the "fallback" icon theme "hicolor" is searched.

**Tips:** \
To browse the available icons, install the "yad" package.
- run `yad-icon-browser` to browse the icons in the current icon theme.
- To browse a different theme, add the theme's name, i.e. `yad-icon-browser hicolor`

To display all icons for a given application, do something like this:
```bash
cd /usr/share/icons
eom $(find . -name 'htop*')  # eom is Mate Desktop's image viewer.
```
### Update database of desktop entries
Usually, desktop entry changes are automatically picked up by desktop environments.
If this is not the case, first check the .desktop file
- Exec: filename should be in the $PATH, or use full path name
- Icon: the icon file must exist, may need full path name
- Category: use a default XDG category

These are the default XDG categories. Your menu may display something different. 
Example: "AudioVideo" displays as "Sound & Video"

| Main Category | Notes                                                                  |
| ------------- | ---------------------------------------------------------------------- |
| AudioVideo    |
| Audio         | Desktop entry must include AudioVideo as well                          |
| Video         | Desktop entry must include AudioVideo as well                          |
| Development   |
| Education     |
| Game          |
| Graphics      |
| Network       |
| Office        |
| Science       |
| Settings      | Entries may appear in a separate menu or as part of a "Control Center" |
| System        |
| Utility       |

you can manually update the desktop entries defined in ~/.local/share/applications:

``` bash
update-desktop-database ~/.local/share/applications
```
Add the -v (verbose) argument to show possible desktop entry errors.

## MIME
the MIME database provides file associations and default applications.

### MIME types
These directories contain the XML and text files that define mime types i.e. text/plain, and associated file extensions i.e. "*.txt"
| Path                 | Usage                    |
| -------------------- | ------------------------ |
| ~/.local/share/mime/ | user specific mime types |
| /usr/share/mime/     | system-wide mime types   |

- Applications describe what MIME types they can handle using **desktop entries**
- the desktop entries and mimetypes are scanned to produce a database file, `mimeinfo.cache`
- as usual, there's a system-wide file `/usr/share/applications/mimeinfo.cache` and a user-specific `~.local/share/applications/mimeinfo.cache`

Default applications for each MIME type are stored in `mimeapps.list` files, which can be stored in several locations. They are searched in the following order, with earlier associations taking precedence over later ones:
| Path                                        | Usage                          |
| ------------------------------------------- | ------------------------------ |
| ~/.config/mimeapps.list                     | user overrides                 |
| /etc/xdg/mimeapps.list                      | system-wide overrides          |
| ~/.local/share/applications/mimeapps.list   | (deprecated) user overrides    |
| /usr/local/share/applications/mimeapps.list | distribution-provided defaults |
| /usr/share/applications/mimeapps.list       | distribution-provided defaults |

Additionally, you may see desktop environment-specific default applications files. For example, /etc/xdg/xfce-mimeapps.list

Tip: Although deprecated, several applications still read/write to ~/.local/share/applications/mimeapps.list. To simplify maintenance, simply symlink it to ~/.config/mimeapps.list:

    $ ln -s ~/.config/mimeapps.list ~/.local/share/applications/mimeapps.list

Note: You might also find files in these locations named `defaults.list`. This file is similar to mimeapps.list except it only lists default applications (not added/removed associations). It is now deprecated and should be manually merged with mimeapps.list.

To discover all the files that are scanned it is possible to enable debug mode by setting the environment variable XDG_UTILS_DEBUG_LEVEL=2 for example: 
    XDG_UTILS_DEBUG_LEVEL=2 xdg-mime query default type text/
will print each configuration file it is searching for mime information.

## Menu files
- /etc/xdg/menus - system-wide
- /usr/share/desktop-directories - system-wide
- /usr/share/mate/desktop-directories - system-wide (specific to MATE desktop) Other desktops such as XFCE have similar directories
- ~/.local/share/desktop-directories - user-specific
These files define menu categories. They are customized based on the Desktop environment and the Linux distribution you are running.

Note: it's very confusing how all this works. 

## freedesktop base directories
Note: freedesktop.org was originally named "X Desktop Group" (XDG) where X means "cross"

- `$XDG_DATA_HOME` - user specific data files - default `$HOME/.local/share`
- `XDG_CONFIG_HOME` - user specific configuration files - default `$HOME/.config`
- `$XDG_DATA_DIRS` -  system data directories - default `/usr/local/share/:/usr/share/`
- `$XDG_CONFIG_DIRS` - system config directories - default `/etc/xdg`
- `$XDG_CACHE_HOME` - user specific non-essential data files - default `$HOME/.cache`