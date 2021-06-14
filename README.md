# Signals
Lightweight type-safe messaging system.

## How to Install
### Git Installation (Best way to get latest version)

If you have Git on your computer, you can open Package Manager indside Unity, select "Add package from Git url...", and paste link ```https://github.com/IntoTheDev/Signals.git```

or

Open the manifest.json file of your Unity project.
Add ```"com.intothedev.signals": "https://github.com/IntoTheDev/Signals.git"```

### Manual Installation (Version can be outdated)
Download latest package from the Release section.
Import Signals.unitypackage to your Unity Project

## Usage

### How to create a signal

Just create a plain C# struct and implement `ISignal` interface.

```csharp
public readonly struct SignalPlayerCreated : ISignal
{
	public readonly string Name;
	public readonly GameObject StartWeapon;

	public SignalPlayerCreated(string name, GameObject startWeapon)
	{
		Name = name;
		StartWeapon = startWeapon;
	}
}
```

### How to send a signal

You need to call `Hub<S>.Dispatch(new S())`. Instead of `S` you need to pass in type of your signal.

```csharp
public class Player : MonoBehaviour
{
	[SerializeField] private string _name = "";
	[SerializeField] private GameObject _weapon = null;

	private void Start()
	{
		Hub.Dispatch(new SignalPlayerCreated(_name, _weapon));
	}
}
```

### How to receive a signal

Your class/struct must implement `IReceiver<S>` interface. Instead of `S` you need to pass in type of your signal.

```csharp
public class PlayerUI : MonoBehaviour, IReceiver<SignalPlayerCreated>
{
	[SerializeField] private TMP_Text _playerName = null;
	[SerializeField] private Image _weaponIcon = null;

	private void Awake()
    	{
		// You can get last dispatched signal like this
		// This is useful if for example UI was created later then the player
        	Receive(Hub<SignalPlayerCreated>.Last);
    	}

	private void OnEnable()
	{
		// Subscribing to signal
		Hub<SignalPlayerCreated>.Add(this);
	}

	private void OnDisable()
	{
		// Unsubscribing from signal
		Hub<SignalPlayerCreated>.Remove(this);
	}

	// Receiving the signal
	public void Receive(in SignalPlayerCreated value)
	{
		_playerName.text = value.Name;
		_weaponIcon.sprite = value.StartWeapon.GetComponent<Weapon>().Icon;
	}
}
```

### UnityEvent

Create a class that inherits from `Listener<SignalType>` and attach it to any game object you want. Now you can invoke UnityEvent to corresponding Signal.

```csharp
public class PlayerCreatedListener : Listener<SignalPlayerCreated> { }
```
