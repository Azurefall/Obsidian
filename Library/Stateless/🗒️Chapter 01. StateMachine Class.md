
```toc
min_depth: 1
max_depth: 2
```
<br/>
<br/>

# ✍️Constructor
---
```ad-info
collapse: close
title: enum FiringMode
~~~csharp
/// <summary>
/// Enum for the different modes used when Fire-ing a trigger
/// </summary>
public enum FiringMode
{
	/// <summary>
	/// 트리거 이벤트 대기열이 필요하지 않을 때 사용되는 모드
	/// Run-to-Completion을 보장하지 않기 때문에 사용할 때는 주의
	/// </summary>
	Immediate,
	/// <summary> 
	/// Run-to-Completion이 필요한 경우 사용하는 모드
	/// 권장하는 모드 (Default)
	/// </summary>
	Queued
}
~~~
```
<br/>

## Use initial state [⭐]
* initialState : 초기화 상태값
```csharp
public StateMachine(TState initialState, FiringMode firingMode)
public StateMachine(TState initialState)
```
<br/>

## Use state Func and Action [⭐]
* stateAccessor : 현재의 상태 값을 읽어오기 위한 함수
* stateMutator : 변경되는 새로운 상태 값을 저장하기 위한 함수
```csharp
public StateMachine(Func<TState> stateAccessor, Action<TState> stateMutator, FiringMode firingMode)
public StateMachine(Func<TState> stateAccessor, Action<TState> stateMutator)
```
<br/>
<br/>

# ✍️Methods
---
## Activate
```csharp
void Activate()
// 비동기 지원
Task ActivateAsync()
```
현재 상태를 활성화합니다. 현재 상태 활성화와 관련된 작업이 호출됩니다. 활성화는 멱등적이며 동일한 현재 상태의 후속 활성화는 활성화 콜백의 재실행으로 이어지지 않습니다.
<br/>

## CanFire
```csharp
bool CanFire(TTrigger trigger)
bool CanFire(TTrigger trigger, out ICollection<string> unmetGuards)
```
현재 상태에서 트리거가 발현 가능할 경우 true 반환
	🚩PermitIf의 Guard 조건이 존재할 경우 CanFire 호출 시 해당 조건을 체크한다.
		즉, if (CanFire(T)) Fire(T) 하게 되면 조건 체크를 두 번 수행하게 된다.
<br/>

## Deactivate
```csharp
void Deactivate()
// 비동기 지원
Task DeactivateAsync()
```
현재 상태를 비활성화합니다. 현재 상태 비활성화와 관련된 작업이 호출됩니다. 비활성화는 멱등적이며 동일한 현재 상태의 후속 비활성화는 비활성화 콜백의 재실행으로 이어지지 않습니다.
<br/>

## Fire [⭐]
```csharp
void Fire(TTrigger trigger)
void Fire(TriggerWithParameters trigger, params object[] args)
void Fire<TArg0>(TriggerWithParameters<TArg0> trigger, TArg0 arg0)
void Fire<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, TArg0 arg0, TArg1 arg1)
void Fire<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, TArg0 arg0, TArg1 arg1, TArg2 arg2)
// 비동기 지원
Task FireAsync(TTrigger trigger)
Task FireAsync<TArg0>(TriggerWithParameters<TArg0> trigger, TArg0 arg0)
Task FireAsync<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, TArg0 arg0, TArg1 arg1)
Task FireAsync<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, TArg0 arg0, TArg1 arg1, TArg2 arg2)
```
지정된 트리거를 통해 현재 상태에서 전환합니다. 대상 상태는 현재 상태의 구성에 따라 결정됩니다.
현재 상태를 떠나고 새 상태로 들어가는 것과 관련된 작업이 호출됩니다.
🧩TriggerWithParameters 는 [SetTriggerParameters](#SetTriggerParameters[⭐]) 통해 설정
<br/>

## GetPermittedTriggers
```csharp
IEnumerable<TTrigger> GetPermittedTriggers(params object[] args)
```
현재 허용된 트리거 목록을 제공합니다.
<br/>

## GetDetailedPermittedTriggers
```csharp
IEnumerable<TriggerDetails<TState, TTrigger>> GetDetailedPermittedTriggers(params object[] args)
```
현재 허용된 트리거들의 설정 항목을 포함한 목록을 제공합니다.
<br/>

## GetInfo [⭐]
```csharp
StateMachineInfo GetInfo()
```
머신의 상태, 전환 및 작업을 노출하는 정보 개체를 제공합니다.
<br/>

## IsInState [⭐]
```csharp
 bool IsInState(TState state)
```
StateMachine 이 제공된 상태인지 확인합니다.
<br/>
<br/>

# ✍️Methods for Configuration
---
## Configure [⭐]
```csharp
StateConfiguration Configure(TState state)
```
StateMachine이 특정 상태에 있을 때 시작/종료 작업 및 허용된 전환의 구성을 시작합니다.
<br/>

## OnTransitioned [⭐]
```csharp
void OnTransitioned(Action<Transition> onTransitionAction)
// 비동기 지원
void OnTransitionedAsync(Func<Transition, Task> onTransitionAction)
```
StateMachine 이 한 상태에서 다른 상태로 전환될 때마다 호출되는 콜백을 등록합니다.
<br/>

## OnTransitionCompleted
```csharp
void OnTransitionCompleted(Action<Transition> onTransitionAction)
// 비동기 지원
void OnTransitionCompletedAsync(Func<Transition, Task> onTransitionAction)
```
StateMachine 이 한 상태에서 다른 상태로 전환되고 모든 OnEntryFrom 등의 메서드가 호출될 때마다 호출되는 콜백을 등록합니다.
<br/>

## OnUnhandledTrigger
```csharp
void OnUnhandledTrigger(Action<TState, TTrigger> unhandledTriggerAction)
void OnUnhandledTrigger(Action<TState, TTrigger, ICollection<string>> unhandledTriggerAction)
// 비동기 지원
void OnUnhandledTriggerAsync(Func<TState, TTrigger, Task> unhandledTriggerAction)
void OnUnhandledTriggerAsync(Func<TState, TTrigger, ICollection<string>, Task> unhandledTriggerAction)
```
처리되지 않은 트리거가 실행될 때 예외를 throw하는 기본 동작을 재정의합니다.
<br/>

## SetTriggerParameters [⭐]
```csharp
TriggerWithParameters SetTriggerParameters(TTrigger trigger, params Type[] argumentTypes)
TriggerWithParameters<TArg0> SetTriggerParameters<TArg0>(TTrigger trigger)
TriggerWithParameters<TArg0, TArg1> SetTriggerParameters<TArg0, TArg1>(TTrigger trigger)
TriggerWithParameters<TArg0, TArg1, TArg2> SetTriggerParameters<TArg0, TArg1, TArg2>(TTrigger trigger)
```
특정 트리거가 실행될 때 제공되어야 하는 인수를 지정합니다.
```ad-info
collapse: close
title: class TriggerWithParameters
~~~csharp
/// <summary>
/// Associates configured parameters with an underlying trigger value.
/// </summary>
public class TriggerWithParameters
{
	readonly TTrigger _underlyingTrigger;
	readonly Type[] _argumentTypes;

	/// <summary>
	/// Create a configured trigger.
	/// </summary>
	/// <param name="underlyingTrigger">Trigger represented by this trigger configuration.</param>
	/// <param name="argumentTypes">The argument types expected by the trigger.</param>
	public TriggerWithParameters(TTrigger underlyingTrigger, params Type[] argumentTypes)
	{
		_underlyingTrigger = underlyingTrigger;
		_argumentTypes = argumentTypes ?? throw new ArgumentNullException(nameof(argumentTypes));
	}

	/// <summary>
	/// Gets the arguments types expected by this trigger.
	/// </summary>
	public IEnumerable<Type> ArgumentTypes => _argumentTypes;

	/// <summary>
	/// Gets the underlying trigger value that has been configured.
	/// </summary>
	public TTrigger Trigger { get { return _underlyingTrigger; } }

	/// <summary>
	/// Ensure that the supplied arguments are compatible with those configured for this
	/// trigger.
	/// </summary>
	/// <param name="args"></param>
	public void ValidateParameters(object[] args)
	{
		if (args == null) throw new ArgumentNullException(nameof(args));

		ParameterConversion.Validate(args, _argumentTypes);
	}
}

public class TriggerWithParameters<TArg0> : TriggerWithParameter
public class TriggerWithParameters<TArg0, TArg1> : TriggerWithParameter
public class TriggerWithParameters<TArg0, TArg1, TArg2> : TriggerWithParameter
~~~
```
<br/>
<br/>

---
