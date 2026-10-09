# SP Football Life 2026 on SteamOS ARM (Konkr Pocket Fit and similar)

Run a **pre-installed copy** of SP Football Life 2026 (SmokePatch, `FL_2026.exe`) on a SteamOS ARM handheld
without Lutris. Tested on the Konkr Pocket Fit Gen3 (Snapdragon SM8650, Adreno 750, Box64 0.4.5): menus, matches,
Sider mods and the built-in controller all work in Game Mode.

## The short version

1. Copy your working Windows install folder to the device, e.g. `/home/steamos/Games/SP Football Life 2026/`
   (the folder that contains `FL_2026.exe`, `SiderAddons/`, `Data/`).
2. Copy this whole folder (`install-spfl26.sh` + `artwork/`) to the device and, in Desktop Mode (Konsole), run:
   ```
   chmod +x install-spfl26.sh
   ./install-spfl26.sh "/home/steamos/Games/SP Football Life 2026"
   ```
   If your game is in the default `~/Games/SP Football Life 2026`, no argument is needed.
3. When it asks, let it close Steam (needed to add the Game Mode shortcuts). Switch to Game Mode and start
   **SP Football Life 2026** from *Library > Non-Steam*. Or double-click the Desktop icon.
   There is also a **SP Football Life 2026 - Settings** shortcut (resolution, V-Sync...) that can save and launch the game.

The script is safe to re-run. It needs internet once (about 450 MB for GE-Proton 9).

## Requirements

- SteamOS ARM image that already provides **Box64** with x86 binfmt (the Konkr image does; check with
  `box64 --version` and `cat /proc/sys/fs/binfmt_misc/box64`).
- `curl` and `python3` (standard).
- About 3 GB free (GE-Proton 9 ~1.5 GB extracted, Wine prefix ~0.7 GB).

## What the script does (for doing it by hand)

1. Downloads **GE-Proton9-27 (x86_64 build)** from the GloriousEggroll releases, checks its SHA-512, extracts to `~/x86ge/`.
2. In the game folder: copies `xinput1_3.dll` to `ddraw.dll` (this is how Sider hooks in, same as the Lutris script
   does) and writes `FL_2026.bat` (starts `SiderAddons\sider.exe`, waits for its log, then starts `FL_2026.exe`).
3. Writes `~/Games/launch-spfl26.sh`, which runs `proton run FL_2026.bat` with:
   - `WINEDLLOVERRIDES="ddraw=n,b;steam_api64=n,b;lsteamclient=d"`
   - `PROTON_NO_ESYNC=1 PROTON_NO_FSYNC=1 PROTON_NO_NTSYNC=1` (needed: Wine under Box64 deadlocks on ntsync)
   - `PROTON_USE_XALIA=0`, `ENABLE_GAMESCOPE_WSI=0`
   - prefix in `~/Games/SPFL26-prefix`
4. Creates the Wine prefix and installs a default `settings.dat` (XInput, Full Screen, V-Sync Enable 2, 1920x1080)
   into `…/pfx/drive_c/users/steamuser/Documents/KONAMI/eFootball PES 2021 SEASON UPDATE/`.
5. Installs the settings tool `~/Games/spfl26-settings.sh` (see "Settings and mods").
6. Creates two Desktop icons and two non-Steam shortcuts (Steam overlay off): **SP Football Life 2026** (the launcher) and
   **SP Football Life 2026 - Settings** (the settings tool). It installs the Steam library artwork from `artwork/` (game) and
   `artwork/settings/` (Settings) under each shortcut's ID. Existing images are never overwritten, so you can replace them
   later with the Decky SteamGridDB plugin.

## Why GE-Proton 9 + Box64 (important, don't "upgrade")

All of these were tried and **fail** for this game on the Konkr:

| Attempt | Result |
|---|---|
| Valve Proton 11 / Experimental (ARM64, FEX) | `FL_2026.exe` hits an invalid instruction (`0F 3E`) in its protector, FEX loops on the exception and the game never opens |
| Proton-CachyOS and GE-Proton 11 ARM64 (FEX) | same crash |
| GE-Proton 11 x86_64 via Box64 | spins forever at 100% on one thread, never opens a window |
| **GE-Proton 9-27 x86_64 via Box64** | **works** |

The SPFL community also reports that Wine/GE-Proton newer than Wine 10.16 / GE-Proton 10 break the Steam API
hooks this game relies on, so stay on 9.x.

## Settings and mods

- **Changing resolution / full screen / V-Sync / controller type:** double-click the **SP Football Life 2026 - Settings** Desktop icon
  (or the same-named entry in Steam, or run `~/Games/spfl26-settings.sh`). It is a small native replacement for the game's
  `Settings.exe`, which needs .NET/Mono and crashes under Box64. Close the game first, pick the values and press **OK**.
  It then asks **"Launch the game now?"**: choose **Launch game** to start the game with the new settings, or **Close**.
  It works from Game Mode too, so you can change settings and start playing without going back to Desktop Mode.
- The in-game menu you see with a controller button / Space is **Sider** (mod loader: stadiums, kits, camera…).
- Keep the Steam overlay **off** for this shortcut (the script does that).
- Gameplay "Switcher": the Windows `FL26 switcher.exe` needs .NET and wasn't tested on ARM. The repo
  https://github.com/eskay993/gamefiles/tree/main/sp-football-life-2026 has a bash version
  (`FL_2026_Switcher-Linux.zip`, needs `yad`) if you want to switch gameplay versions.

## Updating the game (SmokePatch updates)

Updates are `SPFL26_XXX.exe` installers. Easiest: install them on a Windows PC, then copy the changed files over the
game folder on the device (the script doesn't need re-running unless `xinput1_3.dll` changes; re-run it anyway, it's safe).

## Troubleshooting

- **Game never opens, one CPU thread at 100%:** wrong Proton (see table), or the ntsync variables are missing.
- **Black screen with sound in Game Mode:** make sure `ENABLE_GAMESCOPE_WSI=0` is in the launcher.
- **Controller buttons missing in matches in Desktop Mode:** use Game Mode (Steam Input); it works perfectly there.
- **Reset everything:** delete `~/Games/SPFL26-prefix` (this also deletes your saves in `Documents/KONAMI`, back them up first)
  and re-run the script.
- **Back up saves:** `~/Games/SPFL26-prefix/pfx/drive_c/users/steamuser/Documents/KONAMI/`
- Keep a copy of the game folder somewhere safe, the script does not install the game itself.

## License

The script and documentation are released under the [MIT License](LICENSE): use, copy, modify and share freely.
The images in `artwork/` are **not** covered by it (third-party artwork, see `artwork/README.md`).
SP Football Life 2026 itself is not included and is not part of this project; get it from SmokePatch.
This project is not affiliated with SmokePatch, Konami or Valve.

## Credits

- [SmokePatch](https://www.pessmokepatch.com/): SP Football Life (the game and mod).
- [GloriousEggroll](https://github.com/GloriousEggroll/proton-ge-custom): GE-Proton.
- [ptitSeb](https://github.com/ptitSeb/box64): Box64, which makes x86 programs run on ARM.
- [eskay993](https://github.com/eskay993/gamefiles/tree/main/sp-football-life-2026): the original Lutris/Wine script for
  SPFL 2026 on Linux. The `ddraw.dll` trick for Sider, the launcher `.bat`, the DLL overrides and the settings-file layout come from there.
- The SPFL community thread on [evoweb](https://evoweb.uk/threads/sp-fl-2026-on-linux.106593/), which pointed to Wine/GE-Proton 9.x
  as the working version.
- SteamGridDB and its community artists: the Steam artwork (see `artwork/README.md`).
- Konkr Pocket Fit SteamOS ARM image creators, for the Box64 / x86 binfmt setup this relies on.
