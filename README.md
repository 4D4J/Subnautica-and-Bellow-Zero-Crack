# Subnautica & Subnautica: Below Zero — Platform Integration Notes

> **Educational and interoperability research only.**
>
> This repository documents observations about platform-service integration, startup behavior, and fallback code paths in Subnautica and Subnautica: Below Zero. It is not an official patch, crack, or redistribution of modified game files.
>
> Do not use this documentation to bypass licensing, DRM, or access controls. Use legitimate copies of the games and follow the applicable software licenses and terms of service.

## Overview

Subnautica and Subnautica: Below Zero use platform services for parts of their startup flow, input handling, save storage, and optional Steam features. When those services are unavailable or incorrectly initialized, the game may fail to start or produce null-reference and input-related errors.

This project collects technical notes about those interactions, with a focus on:

- Steamworks availability checks
- Platform-service initialization
- Input-device fallback behavior
- Local save-storage behavior
- Virtual keyboard support
- Big Picture mode detection

## Games covered

- **Subnautica**
- **Subnautica: Below Zero**

Always verify the exact game build before comparing method names, signatures, or behavior. Game updates can change assemblies and invalidate earlier observations.

## Assemblies discussed

The notes in this repository refer to the following assemblies:

- `com.rlabrecque.steamworks.net.dll`
- `Assembly-CSharp.dll`

The relevant code is located in platform and input-related classes, including:

- `InteropHelp`
- `PlatformUtils`
- `GameInput`
- `PlatformServicesSteam`

## Technical summary

### Steamworks availability

The Steamworks wrapper contains client and game-server availability checks. These methods determine whether the expected platform API is available and can affect startup behavior when the runtime environment is incomplete.

Methods observed in the analysis include:

- `TestIfAvailableClient`
- `TestIfAvailableGameServer`

### Platform initialization

`PlatformInitAsync` is responsible for selecting and initializing the platform-service implementation used by the game. The initialization path is important because later systems may assume that a valid service object has already been created.

When investigating startup failures, check:

- Which platform implementation is selected
- Whether initialization completes successfully
- Whether the service object is null
- Whether quit callbacks and input services are registered

### Input-device selection

`GameInput.GetPrimaryDevice` and related properties can fail when the input subsystem has not finished initializing. A safe fallback device or a null check may be necessary during debugging, depending on the game version.

### Steam presence and optional features

`PlatformServicesSteam` contains checks and features related to Steam, including:

- `IsPresent`
- `InitializeAsync`
- `GetSupportsVirtualKeyboard`
- `IsBigPictureMode`

These methods should be treated as version-specific research targets rather than stable APIs. Their implementation and behavior may differ between Subnautica and Below Zero.

### Save storage

The game may use platform-backed storage or a local fallback path. When investigating save issues, check the effective save directory and confirm that the process has permission to read and write there.

Do not delete or overwrite save data while testing. Make a backup first.

## Recommended investigation workflow

1. Record the exact game version and platform.
2. Back up the original game files and save data.
3. Capture the relevant log files before making changes.
4. Compare the affected method with the matching assembly from the same game build.
5. Change one behavior at a time in an isolated test environment.
6. Verify startup, input, save loading, and quitting separately.
7. Keep a record of the original and modified behavior.

## Log locations

The game logs can usually be found in locations similar to:

```text
%USERPROFILE%\AppData\LocalLow\Unknown Worlds\Subnautica
%USERPROFILE%\AppData\LocalLow\Unknown Worlds\SubnauticaZero
```

The exact file names and paths may vary by version and installation. Check the latest `Player.log` or equivalent runtime log when diagnosing a crash.

## Troubleshooting

### The game does not start

- Confirm that the assembly matches the installed game version.
- Restore the original files and reproduce the issue without modifications.
- Check the runtime log for platform initialization errors.
- Verify that required dependencies and redistributables are installed.

### Input does not work

- Check whether the primary device is initialized before it is accessed.
- Test keyboard, controller, and mouse input independently.
- Look for null-reference errors in the input-related log entries.

### Saves are missing

- Check the effective local save directory.
- Confirm that the game has permission to access the directory.
- Restore from a backup instead of overwriting existing saves.

### Behavior changes after a game update

This is expected for reverse-engineering notes. Re-check method signatures and control flow against the new assembly rather than applying an older change blindly.

## Limitations

- These notes are not guaranteed to work with every game release.
- No compiled binaries or modified game files are provided here.
- The repository does not provide a license or permission to redistribute proprietary game code.
- Platform behavior may differ across operating systems and storefront versions.

## Contributing

When adding a new observation, include:

- Game name and exact version
- Platform and architecture
- Assembly name
- Class and method name
- Reproduction steps for the original issue
- Relevant log excerpt
- Expected and observed behavior

Avoid committing proprietary binaries, personal save files, credentials, or copyrighted game assets.

## Disclaimer

This is an independent technical documentation project and is not affiliated with Unknown Worlds Entertainment, Valve, Steam, or the developers of Subnautica. Use the information responsibly and only with software you are authorized to inspect or modify.
