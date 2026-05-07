
# Crack Subnautica 1 and Bellow Zero using dnSpy
## Patch DLL `com.rlabrecque.steamworks.net.dll`

Go to the Classe `InteropHelp` and clear `methodes`
- **TestIfAvailableClient**
- **TestIfAvailableGameServer**

## Patch DLL `Assembly-CSharp.dll`

Go to the Classe `PlatformUtils` and clear `methode`
- **PlatformInitAsync**

Go to the Classe `GameInput` and clear `methode` 
- **GetPrimaryDevice**

Go to the Classe `PlatformServicesSteam` and clear `methodes`
- **IsPresent**
- **InitializeAsync**
- **GetSupportVirtualKeyboard**
and `property`
- **IsBigPictureMode**

---

### `com.rlabrecque.steamworks.net.dll`

**TestIfAvailableClient :**
```C#
public static void TestIfAvailableClient()
{

}
```
Clears the Steamworks API client verification to prevent launch errors when Steam is missing.

**TestIfAvailableGameServer :**
```C#
public static void TestIfAvailableGameServer()
{

}
```
Clears the Steamworks API server verification to prevent the game from stopping without a detected Steam connection.

---

### `Assembly-CSharp.dll`

**PlatformInitAsync (Bellow Zero) :**
```C#
private IEnumerator PlatformInitAsync()
{
	Debug.log("Crack: Forcing Null Platform Services...");
	PlatformServicesNull nullServices = new PlatformServicesNull(PlatformServicesNUll.DefaultSavePath);
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
Forces the engine to use empty "Null" services, bypassing DRM checks and preventing auto-closure when Steam is not found.

**PlatformInitAsync () :**
```C#
private IEnumerator PlatformInitAsync()
{
	//No modification
}
```
Placeholder for the original initialization logic.

---
**GetPrimaryDevice (Bellow Zero) :**
```C#
public static GameInput.Device GetPrimaryDevice()
{
	return GameInput.lastDevice;
}
```
Returns a valid input device directly to prevent crashes from uninitialized input systems.

**GetPrimaryDevice (Subnautica 1) :**
```C#
public static GameInput.Device get_PrimaryDevice()  
{    
	if (GameInput.input == null)
	{
		return GameInput.Device.Keyboard;
	}
	return GameInput.input.PrimaryDevice;  
}
```
Prevents a NullReferenceException crash by defaulting to the keyboard if the Steam-dependent input object is missing.

---
**IsPresent :**
```C#
public static bool IsPresent()
{        
	RuntimePlatform platform = Application.platform;
	if (platform == RuntimePlatform.OSXPlayer)
	{
		return Directory.Exists(string.Format("{0}/Plugins/steam_api.bundle", Application.dataPath));
	}        
	if (platform == RuntimePlatform.WindowsPlayer)
	{
		return File.Exists(string.Format("{0}/Plugins/x86_64/steam_api64.dll", Application.dataPath));
	}
	Debug.LogWarningFormat("Unhandled platform {0} when checking for Steam library", new object[]
	{
		Application.platform
	});
	return false;
}
```
Tricks the game into thinking Steam is not installed, forcing it to use local fallback methods.

---
**InitializeAsync (Bellow Zero) :**
```C#
public IEnumerator InitializeAsync()
{
	yield break;
}
```
Skips the Steam initialization phase entirely to bypass license and DRM verification steps at startup.

**InitializeAsync (Subnautica 1) :**
```C#
public IEnumerator InitializeAsync()
{
	string savePath = Path.Combine(Directory.GetParent(Application.dataPath).FullName, "SNAppData/SavedGames");
	this.userStoragePC = new UserStoragePC(savePath);
	yield break;
}
```
Redirects save files to a local folder (SNAppData) since the Steam Cloud is unavailable.

---
**GetSupportVirtualKeyboard :**
```C#
public bool GetSupportsVirtualKeyboard()
{
	return false;
}
```
Disables Steam's virtual keyboard to avoid unnecessary system calls that could cause errors.

---
**IsBigPictureMode (Bellow Zero) :**
```C#
protected static bool get_IsBigPictureMode()  
{
	return false;  
}
```
Disables Big Picture mode detection to prevent the game from calling the missing Steam DLL.
