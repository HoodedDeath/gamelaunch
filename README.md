# GameLaunch
 A wrapper script for running GameMode, LibStrangle, and MangoHud all in one
 
## Features
- Wrapper commands: GameMode, LibStrangle, and MangoHud
- Exporting variables: `DXVK_ASYNC`, `PROTON_NO_ESYNC`, `PROTON_NO_FSYNC`, `PROTON_LOG`, and `MANGOHUD_CONFIGFILE`
- Customized MangoHud launch for a setup that mimics how the SteamDeck uses MangoHud

## Installation
### For Script:
Place the `gamelaunch` script somewhere, and ensure it is executable.
Place `gamelaunch` somewhere in your PATH to invoke it with just its name, instead of a full path

### Mangohud Config
Place all the files in the `mangohud-config` directory in your `${XDG_CONFIG_HOME}/MangoHud/`
Note: `XDG_CONFIG_HOME` should usually default to `${HOME}/.config`, according to [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html).
Note: The `gamelaunch` script will use `${XDG_CONFIG_HOME}/MangoHud` if `XDG_CONFIG_HOME` is set, but will use `${HOME}/.config/MangoHud` if `XDG_CONFIG_HOME` is not set

## Usage
`gamelaunch [--help] [-f <num>] [+defgklmnsy01234DEFGKLMSY] <command>`

`--help, -h, -? : Prints help mesage`

`-f <x> : Sets x as the framerate limit given to LibStrangle`

### `+` Options
Letter options are in enable/disable pairs, number options change MangoHud's starting detail level.
Any of these options can be declared alongside any other options, but a given enable/disable pair will have the behavior set by the last of that pair when reading the string from left to right. Same concept applies to MangoHud detail levels.
Example: `+1g4G` will result in GameMode being disabled and MangoHud detail level set to 4

- `d/D` : Enables/disables exporting `DXVK_ASYNC=1`
- `e/E` : Enables/disables exporting `PROTON_NO_ESYNC=1`
- `f/F` : Enables/disables exporting `PROTON_NO_FSYNC=1`
- `l/L` : Enables/disables exporting `PROTON_LOG=1`
- `g/G` : Enables/disables GameMode wrapper
- `s/S` : Enables/disables LibStrangle wrapper
- `k/K` : Enables/disables Vulkan-only (`-f`) flag for LibStrangle
- `m/M` : Enables/disables MangoHud wrapper
- `y/Y` : Enables/disables DLSYM hijacking (`--dlsym`) flag for MangoHud
- `0-4` : Sets the starting detail level for MangoHud overlay
- `n` : Enables dry run mode

### Default behavior
When no flags are given to gamelaunch, the default setup will be as follows:

- For each of `DXVK_ASYNC`, `PROTON_NO_ESYNC`, and `PROTON_NO_FSYNC`, export it as equal to 1 if it is not already set in the current environment.
- Export `MANGOHUD_CONFIGFILE=/tmp/mango_conf` if MangoHud is available and variable is not already set in current environment.
- Set the wrapper command to `gamemoderun strangle -f 144 mangohud`, if each of them is available.

## MangoHud Notes
This script attempts to change MangoHud config file to enable SteamDeck-like overlay behavior, with multiple detail levels that can be dynamically changed. In an intended installation, `${XDG_CONFIG_HOME}/MangoHud/` contains 5 config files for varrying detail levels, `level0` through `level4`.

GameLaunch checks various things to ensure it's fine to override the MangoHud config file.
1. Check that the MangoHud directory exists in `XDG_CONFIG_HOME` (`${HOME}/.config` if `XDG_CONFIG_HOME` is not set), and that the MangoHud directory contains all 5 of the detail level files. If either of those checks fails, MangoHud may still be enabled, but the custom `MANGOHUD_CONFIGFILE` variable will not be exported, so MangoHud will continue to use the config it would've already used.
2. Second GameLaunch will check that `MANGOHUD_CONFIGFILE` has is not already set in your environment. If that variable is already set, GameLaunch will not overwrite it, and MangoHud will continue to use the config file set by `MANGOHUD_CONFIGFILE`

To change MangoHud's detail level while running (assuming GameLaunch set `MANGOHUD_CONFIGFILE` to its custom path), simply copy the desired config file from `${XDG_CONFIG_HOME}/MangoHud/` to the file `/tmp/mango_conf`. This can be set up through any form of shortcut buttons which can run a command.

## Unavailability
GameLaunch checks each of it's wrapper commands is available and can be used as the script expects. Depending on your setup, you may see various messages saying a feature was unavailable. Here's an explanation of each message:

- GameMode : `gamemoderun` could not be found in `PATH`, likely because it is not installed
- LibStrangle : `strangle` could not be found in `PATH`, likely because it is not installed
- MangoHud : `mangohud` could not be found in `PATH`, likely because it is not installed
- MangoHud custom config : The `MangoHud` directory or one of the files in there couldn't be found in `${XDG_CONFIG_HOME}/` (`${HOME}/.config` if undefined)

Executables (`gamemoderun`, `strangle`, and `mangohud`) are checked for by `which`. For any that are unavailable, that one will not be added to the final wrapper command, regardless of if it was enabled by flags or not. MangoHud's custom config being unavailable does not prevent MangoHud from being used, it just prevents mimicking the SteamDeck's dynamic detail levels
