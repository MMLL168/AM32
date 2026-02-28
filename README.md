# AM32-MultiRotor-ESC-firmware
Firmware for ARM based speed controllers
<p align="left">
  <a href="/LICENSE"><img src="https://img.shields.io/badge/license-GPL--3.0-brightgreen" alt="GitHub license" /></a>
</p>

The AM32 firmware is designed for STM32 ARM processors to control a brushless motor (BLDC).
The firmware is intended to be safe and fast with smooth fast startups and linear throttle. It is meant for use with multiple vehicle types and a flight controller. The firmware can also be built with support for crawlers. For crawler usage please read this wiki page [Crawler Hardware](https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware/wiki/Crawler-Hardware-and-AM32)

## Features

AM32 has the following features:

- Firmware upgradable via betaflight passthrough, single wire serial or arduino
- Servo PWM, Dshot(300, 600) motor protocol support
- Bi-directional Dshot
- KISS standard ESC telemetry
- Variable PWM frequency
- Sinusoidal startup mode, which is designed to get larger motors up to speed

## Daily Development Quick Reference (Build / Task / IntelliSense)

Use this as a 1-minute onboarding checklist for new contributors.

### Development Docs

- Quick reference: [doc/development/Daily Development Quick Reference.md](doc/development/Daily%20Development%20Quick%20Reference.md)
- Docs index: [doc/development/README.md](doc/development/README.md)

### 1) Pick your target

- Common targets: `B_G431B_ESC`, `B_G071_ESC`, `B_F051_ESC`
- Keep build target and IntelliSense target aligned

### 2) Build quickly (daily loop)

Option A (VS Code task, recommended):

1. `Ctrl+Shift+P`
2. `Tasks: Run Task`
3. Run `AM32: build + hash (..., fast)` for your target

Option B (PowerShell):

```powershell
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

### 3) Run strict validation (before release/testing)

- Use task: `AM32: clean + build + hash (...)`
- Or run:

```powershell
& '.\tools\windows\make\bin\make.exe' clean
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

### 4) Verify output files

- Output directory: `obj`
- Expected artifact types: `*.elf`, `*.bin`, `*.hex`

Quick check:

```powershell
Get-ChildItem .\obj\AM32_* | Sort-Object LastWriteTime -Descending | Select-Object -First 10 Name, LastWriteTime
```

### 5) If IntelliSense is red but build is green

1. Select correct C/C++ configuration
2. `C/C++: Reset IntelliSense Database`
3. `Developer: Reload Window`
4. Re-open file and re-check squiggles

### 6) First things to check when command fails

- `python3` missing: run `python3 --version` (or use workspace `python3.bat` fallback)
- `make` missing: use `'.\tools\windows\make\bin\make.exe'` directly
- wrong path: ensure terminal is at repository root

## Build instructions
Download and install Keil community edition. Open the Keil project for the mcu you want in the "Keil projects" folder. Install any mcu packs if prompted. 
Select the build target from the drop down box and build project 

### Build on Windows (GNU Make)

This repository also supports command-line builds on Windows using GNU Make.

Prerequisites:

- GNU Arm toolchain and `make` available in your environment
- Python available (some post-build scripts call `python3`)
- Open a PowerShell terminal at the repository root

Example commands:

```powershell
# Clean and build G431 target
& '.\tools\windows\make\bin\make.exe' clean
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC

# Build only (faster incremental build)
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

Common targets used during development:

- `B_G431B_ESC`
- `B_G071_ESC`
- `B_F051_ESC`

Output artifacts are generated in the `obj` folder, typically:

- `*.elf`
- `*.bin`
- `*.hex`

### Verify build artifacts (optional but recommended)

After a build, you can verify file timestamps/sizes and SHA256 hashes:

```powershell
Get-Item .\obj\AM32_B_G431B_ESC_2.20.elf, .\obj\AM32_B_G431B_ESC_2.20.bin, .\obj\AM32_B_G431B_ESC_2.20.hex |
  Select-Object Name, Length, LastWriteTime | Format-Table -AutoSize

Get-FileHash .\obj\AM32_B_G431B_ESC_2.20.elf -Algorithm SHA256
Get-FileHash .\obj\AM32_B_G431B_ESC_2.20.bin -Algorithm SHA256
Get-FileHash .\obj\AM32_B_G431B_ESC_2.20.hex -Algorithm SHA256
```

### VS Code Tasks (one-click workflow)

This workspace includes predefined tasks in `.vscode/tasks.json`:

- `AM32: clean + build + hash (G431)`
- `AM32: clean + build + hash (G071)`
- `AM32: clean + build + hash (F051)`
- `AM32: build + hash (G431, fast)`
- `AM32: build + hash (G071, fast)`
- `AM32: build + hash (F051, fast)`

Usage in VS Code:

1. Open Command Palette (`Ctrl+Shift+P`)
2. Run `Tasks: Run Task`
3. Select the target task

Recommended usage:

- Daily iteration: use `build + hash (..., fast)`
- Validation before release/testing: use `clean + build + hash (...)`

### Troubleshooting (python3 / make / paths)

#### 1) `python3` not found (`Error 9009` or similar)

Symptoms:

- Build completes compile/link steps but fails in post-build script stage
- Error mentions `python3` is not recognized

Cause:

- Windows often provides `python`, but not `python3` command alias

Fix options:

- Preferred: add a `python3` command to your `PATH`
- Workspace fallback: create `python3.bat` at repository root with:

```bat
python %*
```

Quick check:

```powershell
python --version
python3 --version
```

If `python` works but `python3` fails, use one of the fixes above.

#### 2) `make` command not found

Symptoms:

- `make` is not recognized in terminal
- VS Code task fails before compile starts

Cause:

- `make` executable is not installed or not in `PATH`

Fix options:

- Use project-local make binary directly:

```powershell
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

- Or install make globally and restart terminal/VS Code so `PATH` is refreshed

Quick check:

```powershell
Get-Command make -ErrorAction SilentlyContinue
Test-Path '.\tools\windows\make\bin\make.exe'
```

#### 3) Wrong working directory / relative path errors

Symptoms:

- File not found for scripts, linker files, or output artifacts
- Commands that worked before suddenly fail in another terminal

Cause:

- Terminal current directory is not repository root

Fix:

```powershell
Set-Location 'D:\Work_202506\VS_ESC\AM32'
Get-Location
```

Then re-run the same build task/command.

#### 4) New install works in one terminal but not another

Symptoms:

- One terminal can run tool, another says command not found

Cause:

- Environment variables were updated after terminal was opened

Fix:

- Close and reopen terminal (or restart VS Code)
- Re-run version checks (`python --version`, `make --version` if available)

#### 5) Artifacts not generated or name mismatch

Symptoms:

- `Get-Item`/`Get-FileHash` fails because expected file name is missing

Cause:

- Built a different target, or version string changed

Fix:

```powershell
Get-ChildItem .\obj\AM32_* | Sort-Object LastWriteTime -Descending | Select-Object -First 20 Name, LastWriteTime
```

Use the actual generated filenames for hash checks.

### IntelliSense common false positives (quick triage)

If source code compiles successfully but VS Code still shows red squiggles, use this checklist in order.

#### 1) Confirm active C/C++ configuration matches your target

Typical symptom:

- `main.c` shows many undefined macros/types for a different MCU target

Action:

- Open Command Palette (`Ctrl+Shift+P`)
- Run `C/C++: Select a Configuration...`
- Select the correct target profile from `.vscode/c_cpp_properties.json`

#### 2) Verify compiler path and parser mode

Typical symptom:

- Standard integer types (`uint8_t`, `uint32_t`) or CMSIS/HAL types appear unresolved

Action:

- Check `.vscode/c_cpp_properties.json` has a valid `compilerPath`
- Use ARM GCC-compatible IntelliSense mode (`gcc-arm`)
- Keep C standard aligned with project settings (for this repo: `gnu11`)

#### 3) Reload IntelliSense database after config changes

Typical symptom:

- Errors remain unchanged even after editing include paths/defines

Action:

- Run `C/C++: Reset IntelliSense Database`
- Run `Developer: Reload Window`

#### 4) Check include paths and forced include file

Typical symptom:

- Target symbols/macros missing only in editor, not in actual build

Action:

- Verify include paths cover `Inc` and target-specific MCU headers
- Verify forced include file (if used) path is valid
- Re-open the problematic file after changes

#### 5) Distinguish real compile errors vs editor-only noise

Rule of thumb:

- Build result is the source of truth
- If build is green but IntelliSense is red, treat as config/indexing issue first

Quick compile check:

```powershell
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

#### 6) Fast recovery sequence (when in doubt)

1. Select correct C/C++ configuration
2. Reset IntelliSense database
3. Reload VS Code window
4. Run one real build command
5. Re-open file and re-check squiggles

## Firmware Release & Configuration Tool

The latest release of the firmware can be found [here](https://am32.ca/downloads).

The primary configurator is the [AM32 Configurator](https://am32.ca)
which supports web browser based configuration and firmware update.

You can also use a desktop configurator which you can download from here:

[WINDOWS](https://drive.google.com/file/d/16kaPek9umz7fQFunzBeW4pp2LgT6_E5o/view?usp=drive_link)
[LINUX](https://drive.google.com/file/d/1QtSKwp3RT6sncPADsPkmdasGqNIk68HH/view?usp=sharing)

Alternately you can use the [Online-ESC Configurator](https://esc-configurator.com/) to flash or change settings with any web browser that supports web serial.



## Hardware
AM32 currently has support for STSPIN32F0, STM32F051, STM32G071, GD32E230, AT32F415 and AT32F421.
The CKS32F051 is not recommended due to too many random issues.
Target compatibility List can be found [here](https://github.com/am32-firmware/AM32/blob/main/Inc/targets.h)


## Installation & Bootloader

To use AM32 firmware on a blank ESC, a bootloader must first be installed using an ST-LINK, GD-LINK , CMIS-DAP or AT-LINK.  THe bootloader will be dependant on the MCU used ont he esc . Choose the bootloader that matches the MCU type and signal input pin of the ESC.
The compatibility chart has the bootloader pinouts listed.
Current bootloaders can be found [here](https://github.com/am32-firmware/AM32-bootloader).

After the bootloader has been installed the main firmware from can be installed either with the configuration tools and a Betaflight flight controller or a direct connection with a usb serial adapter modified for one wire.

To update an existing AM32 bootloader an update tool can be found [here](https://github.com/am32-firmware/AM32-unlocker).

## Support and Developers Channel
There are two ways you can get support or participate in improving am32.
We have a discord server here:

https://discord.gg/h7ddYMmEVV

Etiquette: Please wait around long enough for a reply - sometimes people are out flying, asleep or at work and can't answer immediately. 

If you wish to support the project please join the Patreon.

https://www.patreon.com/user?u=44228479


## Sponsors
The AM32 project would not have made this far without help from the following sponsors:

Holmes Hobbies - https://holmeshobbies.com/ - The project would not be where it is today without the support of HH. Check out the Crawlmaster V2 for the best am32 experience!

Repeat Robotics - https://repeat-robotics.com/ - Bringing Am32 esc's to the fighting robot community!

Quaternium - https://www.quaternium.com/ - Firmware development support and hardware donations

Airbot - Many hardware donations

NeutronRC - For hardware, am32 promotion and schematics 

Aikon - Hardware donations and schematics\
Skystars  - For hardware and taking a chance on the first commercial am32 esc's\
Diatone - Hardware donations\
T-motor - Motor and Hardware donations\
HLGRC  - Hardaware donations


## Contributors
A big thanks to all those who contributed time, advice and code to the AM32 project.\
Un!t\
Hugo Chiang (Dusking)\
Micheal Keller (Mikeller)\
ColinNiu\
Jacob Walser

And for feedback from pilots and drivers:\
Jye Smith\
Markus Gritsch\
Voodoobrew

(and many more)

