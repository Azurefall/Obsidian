
# ✍️ How to use

### 🧩PhoneCall Class
---
```ad-note
collapse: close
title: Utility
~~~csharp
public static class StringExtention 
{
	public static string PadCenter(this string source, int totalWidth, char paddingChar)
	{
		if (source == null) throw new ArgumentNullException();
		if (source.Length >= totalWidth) return source;

		bool isOdd = source.Length % 2 > 0;
		int length = (totalWidth - source.Length) / 2;

		string leftPadding = string.Empty.PadLeft(length + (isOdd ? 1 : 0), paddingChar);
		string rightPadding = string.Empty.PadRight(length, paddingChar);

		return leftPadding + source + rightPadding;
	}
}
~~~
```

```ad-note
collapse: close
title: Enums
~~~csharp
enum Trigger
{
	CallDialed,
	CallConnected,
	LeftMessage,
	PlacedOnHold,
	TakenOffHold,
	PhoneHurledAgainstWall,
	MuteMicrophone,
	UnmuteMicrophone,
	SetVolume,
	DropPhone,
	PickupPhone
}

enum State
{
	OffHook,
	Ringing,
	Connected,
	OnHold,
	PhoneDestroyed
}
~~~
```

```ad-note
collapse: close
title: PhoneCall
~~~csharp
using Stateless;
using Stateless.Graph;

public class PhoneCall
{
	State _state;
	StateMachine<State, Trigger> _machine;
	StateMachine<State, Trigger>.TriggerWithParameters<int> _setVolumeTrigger;
	StateMachine<State, Trigger>.TriggerWithParameters<string> _setCalleeTrigger;

	string _caller;
	string _callee;

	public PhoneCall(string caller)
	{
		_caller = caller;
		_state = State.OffHook;

		_machine = new StateMachine<State, Trigger>(() => _state, s => _state = s);

		_setCalleeTrigger = _machine.SetTriggerParameters<string>(Trigger.CallDialed);
		_setVolumeTrigger = _machine.SetTriggerParameters<int>(Trigger.SetVolume);

		_machine
			.Configure(State.OffHook)
			.Permit(Trigger.CallDialed, State.Ringing)
			.OnEntryFrom(Trigger.DropPhone, () => Connected(), "Auto recalling")
			.Permit(Trigger.CallConnected, State.OnHold);

		_machine
			.Configure(State.Ringing)
			.OnEntryFrom(_setCalleeTrigger, callee => OnDialed(callee), "Caller number to call")
			.Permit(Trigger.CallConnected, State.Connected);

		_machine
			.Configure(State.Connected)
			.OnEntry(t => OnConnected())
			.OnExit(t => OnDisconnected())
			.InternalTransition(Trigger.MuteMicrophone, t => OnMute())
			.InternalTransition(Trigger.UnmuteMicrophone, t => OnUnmute())
			.InternalTransition(_setVolumeTrigger, (volume, t) => OnSetVolume(volume))
			.Permit(Trigger.LeftMessage, State.OffHook)
			.Permit(Trigger.PlacedOnHold, State.OnHold)
			.Permit(Trigger.PhoneHurledAgainstWall, State.PhoneDestroyed);

		_machine
			.Configure(State.OnHold)
			.SubstateOf(State.Connected)
			.Permit(Trigger.TakenOffHold, State.Connected)
			.PermitIf(Trigger.DropPhone, State.OffHook, () => { OnDestroy(false); return true; });

		_machine
			.Configure(State.PhoneDestroyed)
			.OnEntryFrom(Trigger.PhoneHurledAgainstWall, t => OnDestroy(true));

		_machine.OnTransitioned(t => Console.WriteLine($"▶OnTransitioned: {t.Source} -> {t.Destination} via {t.Trigger}({string.Join(", ", t.Parameters)})"));
	}

	#region Fire Triggers

	public void Dialed(string callee) { WriteLineFire(_setCalleeTrigger.Trigger); _machine.Fire(_setCalleeTrigger, callee); }
	public void Connected() { WriteLineFire(Trigger.CallConnected); _machine.Fire(Trigger.CallConnected); }
	public void SetVolume(int volume) { WriteLineFire(_setVolumeTrigger.Trigger); _machine.Fire(_setVolumeTrigger, volume); }
	public void Hold() { WriteLineFire(Trigger.PlacedOnHold); _machine.Fire(Trigger.PlacedOnHold); }
	public void Mute() { WriteLineFire(Trigger.MuteMicrophone); _machine.Fire(Trigger.MuteMicrophone); }
	public void Unmute() { WriteLineFire(Trigger.UnmuteMicrophone); _machine.Fire(Trigger.UnmuteMicrophone); }
	public void Resume() { WriteLineFire(Trigger.TakenOffHold); _machine.Fire(Trigger.TakenOffHold); }
	public void Destroy(bool onPurpose)
	{
		Trigger t = onPurpose ? Trigger.PhoneHurledAgainstWall : Trigger.DropPhone;

		WriteLineFire(t);
		_machine.Fire(t); 
	}

	#endregion

	#region State Events

	private void OnDialed(string callee) { _callee = callee; Console.WriteLine($"▷OnDialed      : placed for {_callee}"); }
	private void OnConnected() { Console.WriteLine($"▷OnConnected   : Call started at {DateTime.Now}"); }
	private void OnDisconnected() { Console.WriteLine($"▷OnDisconnected: Call ended at {DateTime.Now}"); }
	private void OnSetVolume(int volume) { Console.WriteLine($"▷OnSetVolume   : Volume set to {volume}!"); }
	private void OnMute() { Console.WriteLine("▷OnMute        : Microphone muted!"); }
	private void OnUnmute() { Console.WriteLine("▷OnUnmute      : Microphone unmuted!"); }
	private void OnDestroy(bool onPurpose) { Console.WriteLine(onPurpose ? "▷OnDestroy     : Fuck you!!" : "▷OnDestroy     : Oh my god!!"); }

	#endregion

	#region Util Methods

	private void WriteLineFire(Trigger trigger)
	{
		string triggerString = trigger.ToString().PadCenter(60, '=');
		Console.WriteLine($"{triggerString}");
	}
	public void Print() { Console.WriteLine($"[{_caller}] placed call and [Status] {_machine.State}"); }
	public string ToDotGraph() { return UmlDotGraph.Format(_machine.GetInfo()); }

	#endregion
}
~~~
```

### 🧩Program
---
```ad-note
collapse: close
title: Program.cs
~~~csharp
using System;

class Program
{
	static void Main(string[] args)
	{
		var phoneCall = new PhoneCall("김대혁");            

		phoneCall.Print();
		phoneCall.Dialed("홍길동");
		phoneCall.Print();
		phoneCall.Connected();
		phoneCall.Print();
		phoneCall.SetVolume(2);
		phoneCall.Print();
		phoneCall.Hold();
		phoneCall.Print();
		phoneCall.Destroy(false);
		phoneCall.Print();
		phoneCall.Mute();
		phoneCall.Print();
		phoneCall.Unmute();
		phoneCall.Print();
		phoneCall.Resume();
		phoneCall.Print();
		phoneCall.SetVolume(11);
		phoneCall.Print();
		phoneCall.Destroy(true);
		phoneCall.Print();

		Console.WriteLine();
		Console.WriteLine(phoneCall.ToDotGraph());

		Console.WriteLine("Press any key...");
		Console.ReadKey(true);
	}        
}
~~~
```

### 🧩Result
---
```ad-success
collapse: close
title: Result
~~~powershell
[김대혁] placed call and [Status] OffHook
=========================CallDialed=========================
▶OnTransitioned: OffHook -> Ringing via CallDialed(홍길동)
▷OnDialed      : placed for 홍길동
[김대혁] placed call and [Status] Ringing
========================CallConnected=======================
▶OnTransitioned: Ringing -> Connected via CallConnected()
▷OnConnected   : Call started at 2023-02-03 오후 2:37:20
[김대혁] placed call and [Status] Connected
==========================SetVolume=========================
▷OnSetVolume   : Volume set to 2!
[김대혁] placed call and [Status] Connected
========================PlacedOnHold========================
▶OnTransitioned: Connected -> OnHold via PlacedOnHold()
[김대혁] placed call and [Status] OnHold
==========================DropPhone=========================
▷OnDestroy     : Oh my god!!
▷OnDisconnected: Call ended at 2023-02-03 오후 2:37:20
▶OnTransitioned: OnHold -> OffHook via DropPhone()
========================CallConnected=======================
▶OnTransitioned: OffHook -> OnHold via CallConnected()
▷OnConnected   : Call started at 2023-02-03 오후 2:37:20
[김대혁] placed call and [Status] OnHold
=======================MuteMicrophone=======================
▷OnMute        : Microphone muted!
[김대혁] placed call and [Status] OnHold
======================UnmuteMicrophone======================
▷OnUnmute      : Microphone unmuted!
[김대혁] placed call and [Status] OnHold
========================TakenOffHold========================
▶OnTransitioned: OnHold -> Connected via TakenOffHold()
[김대혁] placed call and [Status] Connected
==========================SetVolume=========================
▷OnSetVolume   : Volume set to 11!
[김대혁] placed call and [Status] Connected
===================PhoneHurledAgainstWall===================
▷OnDisconnected: Call ended at 2023-02-03 오후 2:37:20
▶OnTransitioned: Connected -> PhoneDestroyed via PhoneHurledAgainstWall()
▷OnDestroy     : Fuck you!!
[김대혁] placed call and [Status] PhoneDestroyed

digraph {
compound=true;
node [shape=Mrecord]
rankdir="LR"

subgraph "clusterConnected"
        {
        label = "Connected\n----------\nentry / Function\nexit / Function"
"OnHold" [label="OnHold"];
}
"OffHook" [label="OffHook"];
"Ringing" [label="Ringing"];
"PhoneDestroyed" [label="PhoneDestroyed"];

"OffHook" -> "Ringing" [style="solid", label="CallDialed / Caller number to call"];
"OffHook" -> "OnHold" [style="solid", label="CallConnected"];
"Ringing" -> "Connected" [style="solid", label="CallConnected"];
"Connected" -> "Connected" [style="solid", label="MuteMicrophone / Function [Function]"];
"Connected" -> "Connected" [style="solid", label="UnmuteMicrophone / Function [Function]"];
"Connected" -> "Connected" [style="solid", label="SetVolume / Function [Function]"];
"Connected" -> "OffHook" [style="solid", label="LeftMessage"];
"Connected" -> "OnHold" [style="solid", label="PlacedOnHold"];
"Connected" -> "PhoneDestroyed" [style="solid", label="PhoneHurledAgainstWall / Function"];
"OnHold" -> "Connected" [style="solid", label="TakenOffHold"];
"OnHold" -> "OffHook" [style="solid", label="DropPhone / Auto recalling [Function]"];
 init [label="", shape=point];
 init -> "OffHook"[style = "solid"]
}
Press any key...

~~~
```
