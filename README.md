# Crack Subnautica 1 and Bellow Zero using dnSpy
## Patch DLL `com.rlabrecque.steamworks.net.dll`

Go to the Classe `InteropHelp` and clear methodes
- #TestIfAvailableClient
- #TestIfAvailableGameServer

## Patch DLL `Assembly-CSharp.dll`

Go to the Classe `PlatformUtils` and clear methode
- #PlatformInitAsync

Go to the Classe `GameInput` and clear methode 
- #GetPrimaryDevice

Go to the Classe `PlatformServicesSteam` and clear methodes
- #IsPresent
- #InitializeAsync
- #GetSupportVirtualKeyboard
and property
- #IsBigPictureMode


### `com.rlabrecque.steamworks.net.dll`

#TestIfAvailableClient :
```C#
public static void TestIfAvailableClient()
{
}
```
---
#TestIfAvailableGameServer :
```C#
public static void TestIfAvailableGameServer()
{
}
```


### `Assembly-CSharp.dll`

#PlatformInitAsync (Bellow Zero):
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
#PlatformInitAsync ():
```C#
private IEnumerator PlatformInitAsync()
{
	//No modification
}
```
---
#GetPrimaryDevice (Bellow Zero):
```C#
public static GameInput.Device GetPrimaryDevice()
{
	return GameInput.lastDevice;
}
```
#GetPrimaryDevice (Subnautica 1) :
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

---
#IsPresent :
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
---
#InitializeAsync (Bellow Zero):
```C#
public IEnumerator InitializeAsync()
{
	yield break;
}
```
#InitializeAsync (Subnautica 1) :
```C#
public IEnumerator InitializeAsync()
{
	string savePath = Path.Combine(Directory.GetParent(Application.dataPath).FullName, "SNAppData/SavedGames");
	this.userStoragePC = new UserStoragePC(savePath);
	yield break;
}
```
---
#GetSupportVirtualKeyboard :
```C#
public bool GetSupportsVirtualKeyboard()
{
	return false;
}
```
---
#IsBigPictureMode (Bellow Zero) :
```C#
protected static bool get_IsBigPictureMode()  
{
	return false;  
}
```
