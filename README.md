# GameLaunch
Simplify launching games with wrappers and environment variables

The culmination of my own scripts and helpers for launching games; simplifying launching games with environment variables set, multiple wrapper commands, and hot-swappable MangoHud overlay detail levels.

#	Contents
- [Installation](#installation)
-	[Usage](#usage)
	-	[Steam Launch options usage example](#steam-launch-options-usage-example)
	- [Options](#options)
		-	[Option notes](#option-notes)
	- [Feature flags](#feature-flags)
		- [Feature flag notes](#feature-flag-notes)
	- [Default options and features](#default-options-and-features)
	- [MangoHud overlay detail levels](#mangohud-overlay-detail-levels)
		- [Changing detail levels](#changing-detail-levels)
- [Notes on execution](#notes-on-execution)
	- [Arguments](#arguments)
	- [Variables and wrappers](#variables-and-wrappers)
	- [Debug logging](#debug-logging)
	- [MangoHud cleanup](#mangohud-cleanup)
- [Customization](#customization)
	- [Basename](#basename)
	- [Temp directory](#temp-directory)
	- [Debug log](#debug-log)
	- [LibStrangle FPS](#libstrangle-fps)
	- [MangoHud](#mangohud)
		- [Adding detail levels](#adding-detail-levels)
	- [Default features](#default-features)
- [Special cases / specific games](#special-cases-specific-games)
	- [SDL video driver](#sdl-video-driver)
	- [TModLoader](#tmodloader)
	- [FNA titles](#fna-titles)
	- [Back 4 Blood](#back-4-blood)
- [Testing notes](#testing-notes)
- [To-do / Possible future improvements](#to-do-possible-future-improvements)

# Installation
GameLaunch is meant to be tolerant to various setup issues and should, in the worst case scenario, do no setup and run the command it is meant to be wrapping. As such, it is acceptable to simply download the script and use it, though doing so will result in some lost functionality.

For best experience and full functionality, do the following:
- Ensure GameMode, LibStrangle, and MangoHud are installed on the system
- Copy each of the `level*` files in `mangohud-config` to `XDG_CONFIG_HOME/MangoHud/` (Defaults to `~/.config/MangoHud`, but will obey `XDG_CONFIG_HOME` if it is set)
- Copy the `gamelaunch` script to a location within your `PATH` variable, such as `~/.local/bin`

There are few hard requirements in this script. Really the only requirements are Bash, `basename`, and `cat`. `basename` and `cat` are both GNU Coreutils. `basename` can be removed if desired (see [Customization/Basename](#basename)). `cat` can be replaced if needed, it's used in the functions `_help` and `_helpmore`. Bash may not be required, but I have not tested GameLaunch in any other shell, so unexpected behavior is very possible; especially when it comes to variable substitutions and the `read` builtin.

Only tested in Bash. Various parts, such as the `read` builtin and variable substitution, may behave differently under different shells

# Usage
`gamelaunch [-option] [+feature flags] <command>`

## Steam Launch options usage example
`gamelaunch %command%`

## Options
- `--help`, `-h`, `-?` : Prints help message
- `--helpmore`, `-hh`, `-??` : Prints extended help message with notes about tested games and special cases
- `-f <x>` : Sets `x` as the framerate limit given to LibStrangle. Must be a positive integer
- `-s` : Exports `SDL_VIDEODRIVER` variable to a default value of `x11,windows`
- `-S <val>` : Exports `SDL_VIDEODRIVER` variable to the value of `val`. If `val` is given as `-1`, `SDL_VIDEODRIVER` will be unset
- `-v` : Enable verbose messages to STDOUT and debug log file at `/tmp/GameLaunch_debug.log`

### Option notes
These options currently do not support multiple single letter flags as a single argument.

Example: to use verbose logging and default SDL driver flag as options, you must supply `-v -s`; `-vs` would not be recognized as a valid option

## Feature flags
- None: Defaults enabled
- `c` : Enable forced MangoHud temp config file cleanup
- `d` : Set `DXVK_ASYNC` environment variable to 1
- `e` : Set `PROTON_NO_ESYNC` environment variable to 1
- `f` : Set `PROTON_NO_FSYNC` environment variable to 1
- `g` : Enable GameMode wrapper command
- `i` : Enable automatic MangoHud config cleanup
- `k` : Enable LibStrangle Vulkan-only flag
- `l` : Set `PROTON_LOG` environment variable to 1
- `m` : Enable MangoHud wrapper, with custom config file path to allow dynamically changing overlay layout
- `n` : Enable dry run. Just prints the setup which would've been launched otherwise
- `s` : Enable LibStrangle wrapper command
- `y` : Enable MangoHud DLSYM flag
- `0`-`4` : Start MangoHud with specified detail level
- `C` : Disable forced MangoHud temp config file cleanup
- `D` : `DXVK_ASYNC` environment variable will not be set
- `E` : `PROTON_NO_ESYNC` environment variable will not be set
- `F` : `PROTON_NO_FSYNC` environment variable will not be set
- `G` : Disable GameMode wrapper command
- `I` : Disable MangoHud automatic config cleanup
- `K` : Disable LibStrangle Vulkan-only flag
- `L` : `PROTON_LOG` environment variable will not be set
- `M` : Disable MangoHud wrapper command
- `S` : Disable LibStrangle wrapper command
- `Y` : Disable MangoHud DLSYM flag

### Feature flag notes
- Flags for disabling features are well used alone, as they allow for easily enabling all defaults except certain features. Flags for enabling features will result in **only** requested features being enabled, with the following exceptions still enabling the defaults: Vulkan-only for LibStrangle (`k`), DLSYM for MangoHud (`y`), dry-run (`n`), MangoHud config levels (`0`-`4`), MangoHud automatic cleanup (`i`), MangoHud forced cleanup (`c`)
- Flags for enabling/disabling a feature and MangoHud config levels will override their counterpart, and the last of each pair/set to appear will be in effect. Example: `+dD20` would result in `DXVK_ASYNC` not being exported and MangoHud config level being set to `0`

## Default options and features:
- DXVK_ASYNC = 1
- PROTON_NO_ESYNC = 1
- PROTON_NO_FSYNC = 1
- PROTON_LOG = 0
- GameMode: Enabled
- LibStrangle: Enabled
- LibStrangle FPS limit: 144
- LibStrangle Vulkan-only: Disabled
- MangoHud: Enabled
- MangoHud DLSYM: Disabled
- MangoHud config level: 0
- MangoHud config auto cleanup: Enabled
- MangoHud config forced cleanup: Disabled

## MangoHud overlay detail levels
This script includes 5 detail levels for MangoHud, mimicking the detail levels provided by SteamOS.

Detail levels are as follows:

0. No visible overlay
1. FPS only
2. Horizontal bar with GPU usage and temp, CPU usage and temp, and FPS
3. Fairly detailed, with GPU, CPU, IO RW, VRAM, RAM, and Vulkan/OpenGL stats
4. Most detailed. Full stats for GPU, CPU (including individual cores), IO, VRAM, RAM, Vulkan/OpenGL, and more info about the running processes/wrappers

Level 2 and higher do exclude some info that is provided by the SteamOS detail levels, namely battery percentage, as they did not apply to my desktop, which this was designed for.

As these detail levels are meant to be easily changed and not effect a MangoHud instance being launch outside of this script, the config file for the active detail level will be copied to `/tmp/mango_conf` and that copy will be deleted before exiting if it is deemed applicable (see [Notes on execution/MangoHud cleanup](#mangohud-cleanup) and [Customization/MangoHud](#mangohud))

### Changing detail levels
There are two ways to change detail levels when using this script, setting the initial detail level when launching an application and changing the detail level of an already running instance.

To change the initial detail level, use the 0-4 options in feature flags for the desired level. Example: `gamelaunch +1 <command>`.

To change the detail level while of an already running instance, copy the config file for the desired level to the live config location (defaults to `/tmp/mango_conf`). Example: To set detail level 2, copy `~/.config/MangoHud/level2` to `/tmp/mango_conf`.

**Note:** In my testing, MangoHud was only consistently recognizing the change of detail level when overwriting its config file. Initial testing was writing a desired config level into the live config file (such as `echo "{CONFIG} > /tmp/mango_conf`), but MH would only recognize the changed level roughly half the time. Overwriting the live config has resulted in no issues with MH recognizing changes.

# Notes on execution

## Arguments
Due to this script being a wrapper, it will assume any item on the command line which it does not understand is the beginning of the command it is wrapping. Upon seeing the first argument which does not match any of the '-' flags and does not start with '+', the script will immediately stop attempting to parse flags meant for it and assume everything remaining on the command line is intended to be directly passed to the enabled wrapper commands.

Example: If the command line is `-q +m vkcube`, the script will see that it does not understand `-q` and will attempt to execute `-q +m vkcube` within the default enabled wrappers

## Variables and wrappers
Exporting variables is essentially guaranteed. Any enabled variable exports will be exported.

Enabling wrappers is not as simple as enabling them though. GameLaunch will check for each wrappers executable (via calling `which <executable>`) and, for any which could not be found, will block them from being enabled. This is in order to allow the script to still work if executables get renamed or you don't have them installed in the first place.

If you notice that any of the wrappers are not enabling when they should be, you should enable verbose messages (-v) and check for any messages about "Cannot find <wrapper exe\> in path, <wrapper name\> unavailable". This message means that `which` returned an error when attempting to lookup the path to the wrapper program. This either means the program is missing within your `PATH`, or has changed name. In the first case, check that the program is installed and that your `PATH` is set correctly. In the second case, the names of the wrappers are in `_gamemode_cmd`, `_strangle_cmd`, and `_mango_cmd`. You can rename them to the program's new name; or even notify me in the issues, I may already know about it and just haven't pushed the change yet.

If MangoHud's overlay shows the wrong layout (probably the default one when no config is found), check the verbose messages for any lines saying either "MangoHud config directory doesn't exist" or "Missing MangoHud config level #". These are the script alerting about failing to find the config directory or preset files. These messages profile the path that was checked and failed. If these messages show up, it means that the script has deemed the config files for MH unavailable and will not attempt to copy them and will not export `MANGOHUD_CONFIGFILE`. This will result in MangoHud using its default config file if it exists.

## Debug logging
Throughout the script, many strings are sent to `vmsg` or `dbglog`; these are the functions that handle debug logging. Anything sent to either of these functions will show up in the debug log, but only lines sent to `vmsg` will show up on the terminal. These messages will show everything from the feature flags being parsed, to what wrappers are available, to the final full command that was executed.

If you're having issues with the script, enable verbosity (-v) and see what the log file at `/tmp/GameLaunch_debug.log` says.

If you'd like to report an issue, enable verbosity and dry run with the command that's causing the issue and upload the debug log with your issue; maybe a non-dry run log as well.

## MangoHud cleanup
MangoHud config files are set to be automatically cleaned by default, but a few things influence doing so. In the following cases, GameLaunch will not clean up the live MH config file:

1. GameLaunch did not handle setting up the config file, either because `MANGOHUD_CONFIGFILE` was already set in the environment GameLaunch was called from or because the config files were deemed unavailable (See last paragraph of [Variables and wrappers](#variables-and-wrappers))
2. There is another instance of GameLaunch running, in which case it assumes that the other instance is also running MH and would get cobbled if it deleted the config file
3. Automatic config cleanup is disabled by feature flag

Cleanup can be forced via a feature flag (+c). If forced cleanup is enabled, all other checks related to cleanup are skipped. GameLaunch won't care if it didn't handle the config file at all. If the live config file exists, it'll be deleted.

# Customization
Many of the default values and presets are based on the values desired for my personal setup, you may want yours to differ. Being a Bash script using a lot of variables, simple edits are quite easy to make, and this section explains some of them.

## Basename
GameLaunch utilizes its file name and a PascalCase version of that name in various places. This behavior of changing the casing of the name is handled in the function `_name`. That function has a basename it expects (seen in variable `_expected_bn`) and a 'fancy' version it can change the name into if desired (seen in `_fancy_bn`). When the file name matches the expected name and the function isn't being called with an argument, `_name` will return the 'fancy' name, otherwise it will return just the file name as returned by the `basename $0` command.

If you would like to rename the script an retain this behavior, simply change the values in `_expected_bn` and `_fancy_bn` as desired.

If you wish to remove currently one of the only hard requirements (being the `basename` command), just change the assignment of `_bn` in `_name` to either a static name or a different command.

## Temp directory
The temporary directory for this script is set to `/tmp`. This directory is used in the paths for the MangoHud live config file and the debugging log. If you wish to change this location, change the variable `_tmp_dir` to your desired location.

Keep in mind that (assuming default flags) this script will copy a MangoHud config file to, and later delete that file from this directory every time it runs. Enabling verbosity and debugging will write the debug log file to this directory as well. A TMPFS mount point is recommended to use.

## Debug log
The debug logging file defaults to the path `/tmp/GameLaunch_debug.log`, though debug logging is not enabled by default. This location is set in the variable `_debug_file`, which will be influenced by both the [Temp directory](#temp-directory) and the [Basename](#basename).

If you change this path, either directly or indirectly, the help message will always show you the debug file's location on the line which enables debugging and verbosity (option `-v`).

## LibStrangle FPS
The default FPS that LibStrangle will limit to is set to 144. If you wish to change this, find the variable `_strangle_fps_arg` and change the 144 to your desired framerate limit.

## MangoHud
As GameLaunch does not directly interact with the configuration options given to MangoHud (only copying the preset config files), these config levels can be edited to your liking to add/remove any info you wish.

The location of MH's live config file is defaulted to `/tmp/mango_conf`. This location is influenced by [Temp directory](#temp-directory). The location and file name can be changed as desired by changing the assignment of `_mango_live`.

### Adding detail levels
Adding new detail levels for MH's overlay is a bit more involved than other customization options, but is also not required if you won't be telling GameLaunch to start with one of your additional levels, as MH will be reading any config file that is placed in the location of its live config location.

To begin, place your new config is the same location as the provided preset files. The file name should be `levelX`, where `X` is the level number which will be assigned to your config level.

Two code edits are now needed to allow GameLaunch to start with your config level. The first is in the function `_readflags`; find the case in the switch statement which is followed by the comment "MangoHud level flags", and add your new level to the matching statement so that it is `0|1|2|3|4|X)`. The second code edit required is in the function `_mango_cp`; the second if statement there determines whether the selected config level is valid or not, and you will need to change the second condition of that if statement to match your new config's number, such as `[[ $_mango_level -le X]]`.

If you wish to allow GameLaunch to check that your config file exists the same way it checks the provided files, you'll need to add another `elif` case in the chain of MangoHud checks within the function `_check_available`. Duplicating the lines for level 4 and changing all the fours with your `X` will be enough.

## Default features
Modifying which features are enabled by default is quite easy, as that is set up in a single function, you just need to know which variables to set. All feature enabling variables are set to 0 (disabled) at the beginning of the script. To enable a feature here, its variable will need to be set to 1. The feature variables are as follows:

- `_dxvk_state` for exporting DXVK_ASYNC
- `_esync_state` for exporting PROTON_NO_ESYNC
- `_fsync_state` for exporting PROTON_NO_FSYNC
- `_log_state` for exporting PROTON_LOG
- `_sdl_state` for exporting SDL_VIDEODRIVER (Make sure you also set `_sdl_driver` to your desired value, or it will be blank by default)
- `_gamemode_state` for GameMode wrapper
- `_strangle_state` for LibStrangle wrapper
- `_strangle_vulkan` for LibStrangle Vulkan-only argument
- `_mango_state` for MangoHud wrapper
- `_mango_dlsym` for MangoHud DLSYM argument
- `_mango_level` for MangoHud initial config level (Set to `0`-`4` to match the preset config levels)
- `_mango_cleanup` for MangoHud automatic config cleanup
- `_verbose` for verbose messages and debug logging

To change the default feature setup, go to the function `_try_enable_all`. Leave the line that sets `_any=1`; this is to ensure that this setup only runs once, in order to prevent cobbling any features that may have been intentionally disabled via feature flags. The assignments below `_any` are the defaults this function will set up. Add/remove/change these to your liking

# Special cases / specific games
These notes are about specific games or engines that have been figured out while testing. These notes also exist in GameLaunch's extended help (-??).

## SDL video driver
Setting `SDL_VIDEODRIVER` in your environment can cause issues for some games depending on what it is set to. Many of the issues come about when it is set to `wayland`.

Some games will run happily with this set to `wayland,x11`, and others won't. See [FNA titles](#fna-titles) and [Back 4 Blood](#back-4-blood) as two examples of games that are not happy with this setting.

To avoid as many of these issues as I can, the simple SDL driver flag for GameLaunch (-s) will set `SDL_VIDEODRIVER="x11,windows"`, as this setting has the best track record in my testing. This default can be changed in the script by changing the assignment of `_sdl_driver` in the `-s` flag section of the main section near the end of the script (line 688 as of writing this section). The default assignment can also be overridden on a per-game basis by instead using the `-S <val>` argument, where `val` is the assignment for `SDL_VIDEODRIVER`.

## TModLoader
The way TML launches has caused issues for GameLaunch during testing. Upon launch, TML will exit itself before launching Terraria and itself again. From GameLaunch's perspective, the process it was told to run in all the wrappers had exited when TML closed itself at first, and therefor GameLaunch proceeded to delete the live MangoHud config file. This issue led to adding the feature flag to enable/disable MangoHud automatic cleanup. TML (and probably any other mod loader/launcher/game that exits before properly starting) requires automatic cleanup to be disabled (+I) in order for MangoHud to work as expected.

## FNA titles
FNA games have been a sore spot in testing. My main example of such is Celeste, which needs two things to run properly (at least when your system is Wayland).

1. `SDL_VIDEODRIVER` needs to be set to `x11` or the game will immediately crash and the error log will have "No supported FNA3D driver found!". Strangely, the simple `-s` flag doesn't work. `-S x11` needs to be set.
2. `-gldevice:Vulkan` needs to be appended to the end of Celeste's launch command to prevent a completely garbled Steam overlay. This isn't a GameLaunch option, and I have not yet implemented one to do this.

The Steam launch option which has been consistently successful for FNA titles: `gamelaunch -S x11 %command% -gldevice:Vulkan`

## Back 4 Blood
B4B has always crashed in my testing unless `SDL_VIDEODRIVER` contains `windows` as one of the available drivers. In my testing, I've always been given an Easy AntiCheat window saying "Failed to initialize dependencies". This led to the addition of and is fixed by the simple SDL driver option (-s). Other titles using EAC may benefit from the same fix.

# Testing notes
- My personal testing has been on KDE Plasma, almost entirely under Wayland, and with `SDL_VIDEODRIVER` explicitly set to `wayland`. 
- If a game is starting on the wrong screen or seeming to be full screen despite not taking up to whole screen, this has usually been fixed by setting `SDL_VIDEODRIVER` with either `-s` or `-S x11`, depending on the game.
- Games I have found which need `-S x11`:
	- Celeste
	- Crypt of the NecroDancer
	- The Dishwasher: Vampire Smile
	- Terraria (also fine with `-s`)

# To-do / Possible future improvements
- Config file for this script to pull default values from