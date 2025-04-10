# Clone Installs

## About
This tutorial describes the process of creating clone installs of Baldur's Gate II.

## What are clone installs?
A 'clone install' refers to an installation of Baldur's Gate II that copies only the minimum files to the install directory. With some editing, the clone can be configured to look for the rest of the files in a different location (e.g., the main install location instead of the clone install location).

## Why use clone installs?
Using clone installs allows for substantially smaller installations of Baldur's Gate II. This enables several dozen clone installs instead of a single main install, facilitating more isolated testing, development, and gameplay.

## How to make clone installs

1. Perform a full install of Baldur's Gate II. This tutorial assumes a main installation path of `C:\games\bg2\base`.
2. Perform a full install of Baldur's Gate II: Throne of Bhaal.
3. Install the latest official patch (v26498).
4. Create a new folder where you want the clone install to be located. This tutorial assumes a clone installation path of `C:\games\bg2\cloneA`.
5. Copy the following files from the main install to the clone install:

   ```
   baldur.ini
   bgmain.exe
   chitin.key
   dialog.tlk
   dialogf.tlk
   keymap.ini
   ```

6. Create the following folders in the clone install:

   - `override`
   - `script compiler` (some programs decompile scripts to this folder)

   Optionally, you may also create folders for:

   - `music`
   - `sounds`
   - `portraits`
   - `scripts`

7. Run `ikc` and enter the relative path from `bgmain.exe` to the BIFF files (e.g., `..\main\data`).
8. Edit the `.exe` file to configure the following:

   - `dialog.tlk` - for text
   - `dialogf.tlk` - for female text
   - `override` - shared override folder
   - Sacrifice characters to override - to simplify mod installations
   - Cache (optional)

The resulting clone install should be approximately 16.5 MB.

## Comprehensive Multi-Install Guide

1. **Reinstall the entire game**: Install the game from CDs to a different directory. This is equivalent to copying the BG2 directory and altering `baldur.ini`.
2. **Clone install**: Copy the main files and patch the `.exe` to look in the local directory.
3. **Tiny clone**: Copy only the essential files and reuse a secondary override folder.

### Notes

- IDS files in the path (override or BIFF) are required for `weidu` to compile scripts/dialogs.
- Use `REQUIRE_FILE` for BIFFs.
- `InfExp` BIFFs are necessary.
- `DLTCEP` only checks the local `\data` folder.

### File Paths

- `hd0:\dialog.tlk`
- `hd0:\dialogf.tlk`
- `hd0:\movies`

### Folder Structure

- `./arenas`
- `hd0:\cache`
- `./characters`
- `./mpsave`
- `hd0:\music`
- `./override`
- `./portraits`
- `./save`
- `./scripts` (optional)
- `./sounds` (optional)
- `./temp`