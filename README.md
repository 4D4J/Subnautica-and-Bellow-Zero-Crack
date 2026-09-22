# Crack Subnautica 1 and Below Zero using dnSpy and x64dbg

> **Technical documentation and research notes.** This README describes the platform and startup behavior observed while analyzing Subnautica and Subnautica: Below Zero. Always use legitimate copies of the games and respect the applicable licenses and terms of service.

## Overview

This project documents the use of **x64dbg** and **dnSpy** to investigate startup and crash behavior in Subnautica and Subnautica: Below Zero.

The crash logs can be found in the following directories:

- `AppData\\LocalLow\\Unknown Worlds\\Subnautica`
- `AppData\\LocalLow\\Unknown Worlds\\SubnauticaZero`

The main log file is generally named `Player.log`, although the exact files may vary depending on the game version.

> **Important:** Make a backup of the original assemblies and save files before making any changes. Method names, signatures, and behavior may differ between game versions.

---

## Assemblies covered

The notes below refer to these assemblies:

- `com.rlabrecque.steamworks.net.dll`
- `Assembly-CSharp.dll`

The relevant classes include:

- `InteropHelp`
- `PlatformUtils`
- `GameInput`
- `PlatformServicesSteam`

---

## Patch summary

The documented changes concern platform initialization, Steam availability checks, input initialization, local save storage, virtual keyboard support, and Big Picture mode detection.

### `com.rlabrecque.steamworks.net.dll`

In the `InteropHelp` class, the following methods were reviewed:

- `TestIfAvailableClient`
- `TestIfAvailableGameServer`

### `Assembly-CSharp.dll`

In the `PlatformUtils` class:

- `PlatformInitAsync`

In the `GameInput` class:

- `GetPrimaryDevice`

In the `PlatformServicesSteam` class:

- `IsPresent`
- `InitializeAsync`
- `GetSupportVirtualKeyboard`
- `IsBigPictureMode`

---

## Detailed notes

### `com.rlabrecque.steamworks.net.dll`

#### `TestIfAvailableClient`

```csharp
public static void TestIfAvailableClient()
{

}
```

This removes the client-availability check that can cause startup errors when the expected Steam environment is unavailable.

#### `TestIfAvailableGameServer`

```csharp
public static void TestIfAvailableGameServer()
{

}
```

This removes the game-server availability check that can stop the game when no compatible Steam connection is detected.

---

### `Assembly-CSharp.dll`

#### `PlatformInitAsync` — Below Zero

```csharp
private IEnumerator PlatformInitAsync()
{
    Debug.Log("Crack: Forcing Null Platform Services...");
    PlatformServicesNull nullServices = new PlatformServicesNull(PlatformServicesNull.DefaultSavePath);
    yield return nullServices.InitializeAsync();
    this.services = nullServices;
    this._gamepadLightBar = new GamepadLightBar(this.services);
    foreach (IOnQuitBehaviour behaviour in this.deferredRegisterQuitBehaviours)
    {
        PlatformUtils.RegisterOnQuitBehaviour(behaviour);
    }
    this.deferredRegisterQuitBehaviours.Clear();
    yield break;
}
```

This forces the engine to use the internal null-platform service implementation instead of the Steam-backed implementation.

#### `PlatformInitAsync` — original placeholder

```csharp
private IEnumerator PlatformInitAsync()
{
    // No modification
}
```

This represents the original initialization area that was reviewed during the analysis.

---

#### `GetPrimaryDevice` — Below Zero

```csharp
public static GameInput.Device GetPrimaryDevice()
{
    return GameInput.lastDevice;
}
```

Returns the last known input device to avoid accessing an uninitialized input object.

#### `GetPrimaryDevice` — Subnautica

```csharp
public static GameInput.Device get_PrimaryDevice()
{
    if (GameInput.input == null)
    {
        return GameInput.Device.Keyboard;
    }
    return GameInput.input.PrimaryDevice;
}
```

Uses the keyboard as a fallback when the input object has not yet been initialized.

---

#### `IsPresent`

```csharp
public static bool IsPresent()
{
    RuntimePlatform platform = Application.platform;
    if (platform == RuntimePlatform.OSXPlayer)
    {
        return Directory.Exists(string.Format("{0}/Plugins/steam_api.bundle", Application.dataPath));
    }
    if (platform == RuntimePlatform.WindowsPlayer)
    {
        return File.Exists(string.Format("{0}/Plugins/x86_64/steam_api64.dll", Application.dataPath));
    }
    Debug.LogWarningFormat("Unhandled platform {0} when checking for Steam library", new object[]
    {
        Application.platform
    });
    return false;
}
```

Checks whether the expected Steam library exists for the current platform. This method is useful when investigating why the game selects or rejects a platform-service implementation.

---

#### `InitializeAsync` — Below Zero

```csharp
public IEnumerator InitializeAsync()
{
    yield break;
}
```

Skips the Steam initialization coroutine when the Steam service is not available.

#### `InitializeAsync` — Subnautica 1

```csharp
public IEnumerator InitializeAsync()
{
    string savePath = Path.Combine(Directory.GetParent(Application.dataPath).FullName, "SNAppData/SavedGames");
    this.userStoragePC = new UserStoragePC(savePath);
    yield break;
}
```

Creates a local user-storage object and redirects save handling to the `SNAppData/SavedGames` directory.

> Always back up existing saves before testing alternate storage behavior.

---

#### `GetSupportsVirtualKeyboard`

```csharp
public bool GetSupportsVirtualKeyboard()
{
    return false;
}
```

Reports that the Steam virtual keyboard is unavailable, avoiding calls to a service that may not exist in the current environment.

---

#### `IsBigPictureMode` — Below Zero

```csharp
protected static bool get_IsBigPictureMode()
{
    return false;
}
```

Disables Big Picture mode detection when the related Steam functionality is unavailable.

---

## Troubleshooting

### The game does not start

- Confirm that the assembly matches the installed game version.
- Restore the original files and reproduce the issue without modifications.
- Check the latest `Player.log` for platform initialization errors.
- Verify that required dependencies and redistributables are installed.

### Input crashes or does not work

- Check whether the primary device is initialized before it is accessed.
- Test keyboard and controller input separately.
- Look for null-reference errors in the input-related log entries.

### Saves are missing

- Check the effective local save directory.
- Confirm that the game has permission to read and write there.
- Restore from a backup instead of overwriting existing saves.

### A game update breaks the changes

This is expected when working with reverse-engineering notes. Re-check the method signatures and control flow against the new assembly rather than applying an older change blindly.

---

## Limitations

- These notes are version-specific and may not work with every release.
- No compiled binaries or modified game files are provided here.
- The repository does not grant permission to redistribute proprietary game code or assets.
- Platform behavior may differ between operating systems and storefront versions.

## Disclaimer

This is an independent technical documentation project and is not affiliated with Unknown Worlds Entertainment, Valve, Steam, or the developers of Subnautica. Use the information responsibly and only with software you are authorized to inspect or modify.
