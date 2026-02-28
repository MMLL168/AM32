## Daily Development Quick Reference (Build / Task / IntelliSense)

This page is a shareable 1-minute onboarding flow for AM32 daily development on Windows.

## 1) Pick your target

Use the same target in both build and IntelliSense configuration.

Common targets:

- `B_G431B_ESC`
- `B_G071_ESC`
- `B_F051_ESC`

## 2) Build quickly (daily loop)

### Option A: VS Code task (recommended)

1. Press `Ctrl+Shift+P`
2. Run `Tasks: Run Task`
3. Select `AM32: build + hash (..., fast)` for your target

### Option B: PowerShell

```powershell
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

## 3) Run strict validation (before release/testing)

Use `clean + build + hash` task, or run:

```powershell
& '.\tools\windows\make\bin\make.exe' clean
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

## 4) Check artifacts quickly

Output is generated in `obj` (typically `*.elf`, `*.bin`, `*.hex`).

```powershell
Get-ChildItem .\obj\AM32_* | Sort-Object LastWriteTime -Descending | Select-Object -First 10 Name, LastWriteTime
```

## 5) If IntelliSense is red but build is green

1. Select the correct C/C++ configuration
2. Run `C/C++: Reset IntelliSense Database`
3. Run `Developer: Reload Window`
4. Re-open the file and re-check squiggles

## 6) Fast failure triage (python3 / make / path)

### `python3` not found

```powershell
python --version
python3 --version
```

If only `python` works, add `python3` alias or keep repository-root `python3.bat` fallback.

### `make` not found

Use project-local binary directly:

```powershell
& '.\tools\windows\make\bin\make.exe' B_G431B_ESC
```

### Wrong current directory

```powershell
Set-Location 'D:\Work_202506\VS_ESC\AM32'
Get-Location
```

## Related documents

- [README](../../README.md)
- [Building in WSL](Building%20in%20WSL.md)
