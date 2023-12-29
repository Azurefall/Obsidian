
```toc
min_depth: 1
max_depth: 2
```
<br/>
<br/>

# ✍️Constructor
---
## Use Configure method in StateMachine class
```csharp
internal StateConfiguration(StateMachine<TState, TTrigger> machine, StateRepresentation representation, Func<TState, StateRepresentation> lookup)
```
🧩일반적으로 StateMachine.Configure ([[🗒️Chapter 01. StateMachine Class#Configure [⭐]]]) 메소드를 통해 설정하며 최초 호출 시 아래와 같이 생성된다.
```csharp
new StateConfiguration(this, GetRepresentation(state), GetRepresentation);
```
<br/>
<br/>

# ✍️Methods
---
## Ignore
```csharp
StateConfiguration Ignore(TTrigger trigger)
```
구성된 상태(TState)일 때 지정된 trigger를 무시합니다.
<br/>

## IgnoreIf
```csharp
StateConfiguration IgnoreIf(TTrigger trigger, Func<bool> guard, string guardDescription = null)
StateConfiguration IgnoreIf(TTrigger trigger, params Tuple<Func<bool>, string>[] guards)
StateConfiguration IgnoreIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, bool> guard, string guardDescription = null)
StateConfiguration IgnoreIf<TArg0>(TriggerWithParameters<TArg0> trigger, params Tuple<Func<TArg0, bool>, string>[] guards)
StateConfiguration IgnoreIf<TArg0, TArgo1>(TriggerWithParameters<TArg0, TArgo1> trigger, Func<TArg0, TArgo1, bool> guard, string guardDescription = null)
StateConfiguration IgnoreIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, params Tuple<Func<TArg0, TArg1, bool>, string>[] guards)
StateConfiguration IgnoreIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, bool> guard, string guardDescription = null)
StateConfiguration IgnoreIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, params Tuple<Func<TArg0, TArg1, TArg2, bool>, string>[] guards)
```
Func guard가 true를 반환하는 경우 구성된 상태(TState)에 있을 때 지정된 trigger를 무시합니다.
<br/>

## InitialTransition
```csharp
StateConfiguration InitialTransition(TState targetState)
```
targetState 일 때의 내부 전환을 추가합니다. 현재 상태에 들어갈 때 StateMachine은 초기 전환(Initial Transition)을 찾고 대상 상태에 들어갑니다.
<br/>

## InternalTransition [⭐]
```csharp
StateConfiguration InternalTransition(TTrigger trigger, Action<Transition> entryAction)
StateConfiguration InternalTransition(TTrigger trigger, Action internalAction)
StateConfiguration InternalTransition<TArg0>(TTrigger trigger, Action<Transition> internalAction)
StateConfiguration InternalTransition<TArg0>(TriggerWithParameters<TArg0> trigger, Action<TArg0, Transition> internalAction)
StateConfiguration InternalTransition<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Action<TArg0, TArg1, Transition> internalAction)
StateConfiguration InternalTransition<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Action<TArg0, TArg1, TArg2, Transition> internalAction)
// 비동기 지원
StateConfiguration InternalTransitionAsync(TTrigger trigger, Func<Transition, Task> entryAction)
StateConfiguration InternalTransitionAsync(TTrigger trigger, Func<Task> internalAction)
StateConfiguration InternalTransitionAsync<TArg0>(TTrigger trigger, Func<Transition, Task> internalAction)
StateConfiguration InternalTransitionAsync<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, Transition, Task> internalAction)
StateConfiguration InternalTransitionAsync<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, Transition, Task> internalAction)
StateConfiguration InternalTransitionAsync<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, Transition, Task> internalAction)
```
StateMachine 에 내부 전환을 추가합니다. 내부 작업은 Exit 및 Entry 작업을 트리거하지 않으며 StateMachine 의 상태를 변경하지 않습니다.
🧩InternalTransition 은 호출 시 내부적으로 아래의 InternalTransitionIf를 호출한다. (비동기도 마찬가지...)
<br/>

## InternalTransitionIf [⭐]
```csharp
StateConfiguration InternalTransitionIf(TTrigger trigger, Func<object[], bool> guard, Action<Transition> entryAction, string guardDescription = null)
StateConfiguration InternalTransitionIf(TTrigger trigger, Func<object[], bool> guard, Action internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0>(TTrigger trigger, Func<object[], bool> guard, Action<Transition> internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, bool> guard, Action<TArg0, Transition> internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<object[], bool> guard, Action<TArg0, TArg1, Transition> internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, bool> guard, Action<TArg0, TArg1, Transition> internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<object[], bool> guard, Action<TArg0, TArg1, TArg2, Transition> internalAction, string guardDescription = null)
StateConfiguration InternalTransitionIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, bool> guard, Action<TArg0, TArg1, TArg2, Transition> internalAction, string guardDescription = null)
// 비동기 지원
StateConfiguration InternalTransitionAsyncIf(TTrigger trigger, Func<bool> guard, Func<Transition, Task> entryAction)
StateConfiguration InternalTransitionAsyncIf(TTrigger trigger, Func<bool> guard, Func<Task> internalAction)
StateConfiguration InternalTransitionAsyncIf<TArg0>(TTrigger trigger, Func<bool> guard, Func<Transition, Task> internalAction)
StateConfiguration InternalTransitionAsyncIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<bool> guard, Func<TArg0, Transition, Task> internalAction)
StateConfiguration InternalTransitionAsyncIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<bool> guard, Func<TArg0, TArg1, Transition, Task> internalAction)
StateConfiguration InternalTransitionAsyncIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<bool> guard, Func<TArg0, TArg1, TArg2, Transition, Task> internalAction)
```
StateMachine 에 내부 전환을 추가합니다. 내부 작업은 Exit 및 Entry 작업을 트리거하지 않으며 StateMachine 의 상태를 변경하지 않습니다.
단, Func guard가 true를 반환하는 경우에만 동작합니다.
<br/>

## OnActivate (🚩구버전에서만 가능)
```csharp
StateConfiguration OnActivate(Action activateAction, string activateActionDescription = null)
// 비동기 지원
StateConfiguration OnActivateAsync(Func<Task> activateAction, string activateActionDescription = null)
```
구성된 상태(TState)가 활성화될 때 실행할 조치를 지정합니다.
<br/>

## OnDeactivate (🚩구버전에서만 가능)
```csharp
StateConfiguration OnDeactivate(Action deactivateAction, string deactivateActionDescription = null)
// 비동기 지원
StateConfiguration OnDeactivateAsync(Func<Task> deactivateAction, string deactivateActionDescription = null)
```
구성된 상태(TState)가 비활성화될 때 실행할 조치를 지정합니다.
<br/>

## OnEntry [⭐]
```csharp
StateConfiguration OnEntry(Action entryAction, string entryActionDescription = null)
StateConfiguration OnEntry(Action<Transition> entryAction, string entryActionDescription = null)
// 비동기 지원
StateConfiguration OnEntryAsync(Func<Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryAsync(Func<Transition, Task> entryAction, string entryActionDescription = null)
```
구성된 상태(TState)로 진입할 때 실행할 작업을 지정합니다.
<br/>

## OnEntryFrom [⭐]
```csharp
StateConfiguration OnEntryFrom(TTrigger trigger, Action entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom(TTrigger trigger, Action<Transition> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom(TriggerWithParameters trigger, Action<Transition> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0>(TriggerWithParameters<TArg0> trigger, Action<TArg0> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0>(TriggerWithParameters<TArg0> trigger, Action<TArg0, Transition> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Action<TArg0, TArg1> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Action<TArg0, TArg1, Transition> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Action<TArg0, TArg1, TArg2> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFrom<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Action<TArg0, TArg1, TArg2, Transition> entryAction, string entryActionDescription = null)
// 비동기 지원
StateConfiguration OnEntryFromAsync(TTrigger trigger, Func<Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync(TTrigger trigger, Func<Transition, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, Transition, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, Transition, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, Task> entryAction, string entryActionDescription = null)
StateConfiguration OnEntryFromAsync<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, Transition, Task> entryAction, string entryActionDescription = null)
```
지정된 trigger를 통해 구성된 상태(TState)로 진입할 때 실행할 작업을 지정합니다.
<br/>

## OnExit [⭐]
```csharp
StateConfiguration OnExit(Action exitAction, string exitActionDescription = null)
StateConfiguration OnExit(Action<Transition> exitAction, string exitActionDescription = null)
// 비동기 지원
StateConfiguration OnExitAsync(Func<Task> exitAction, string exitActionDescription = null)
StateConfiguration OnExitAsync(Func<Transition, Task> exitAction, string exitActionDescription = null)
```
구성된 상태(TState)에서 진출할 때 실행할 작업을 지정합니다.
<br/>

## Permit [⭐]
```csharp
StateConfiguration Permit(TTrigger trigger, TState destinationState)
```
지정된 trigger를 수락하고 destinationState로 전환합니다.
<br/>

## PermitDynamic
```csharp
StateConfiguration PermitDynamic(TTrigger trigger, Func<TState> destinationStateSelector, string destinationStateSelectorDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamic<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector, string destinationStateSelectorDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamic<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, TState> destinationStateSelector, string destinationStateSelectorDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamic<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, TState> destinationStateSelector, string destinationStateSelectorDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
```
지정된 trigger를 수락하고 Func destinationStateSelector에 의해 동적으로 계산된 대상 상태로 전환합니다.
<br/>

## PermitIf [⭐]
```csharp
StateConfiguration PermitIf(TTrigger trigger, TState destinationState, Func<bool> guard, string guardDescription = null)
StateConfiguration PermitIf(TTrigger trigger, TState destinationState, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitIf<TArg0>(TriggerWithParameters<TArg0> trigger, TState destinationState, Func<TArg0, bool> guard, string guardDescription = null)
StateConfiguration PermitIf<TArg0>(TriggerWithParameters<TArg0> trigger, TState destinationState, params Tuple<Func<TArg0, bool>, string>[] guards)
StateConfiguration PermitIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, TState destinationState, Func<TArg0, TArg1, bool> guard, string guardDescription = null)
StateConfiguration PermitIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, TState destinationState, params Tuple<Func<TArg0, TArg1, bool>, string>[] guards)
StateConfiguration PermitIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, TState destinationState, Func<TArg0, TArg1, TArg2, bool> guard, string guardDescription = null)
StateConfiguration PermitIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, TState destinationState, params Tuple<Func<TArg0, TArg1, TArg2, bool>, string>[] guards)
```
Func guard가 true를 반환하는 경우 지정된 trigger를 수락하고 destinationState로 전환합니다.
	🚩PermitIf의 Guard 조건이 존재할 때때 CanFire 호출 시 해당 조건을 체크한다.
		즉, if (CanFire(T)) Fire(T) 하게 되면 조건 체크를 두 번 수행하게 된다.
<br/>

## PermitDynamicIf
```csharp
StateConfiguration PermitDynamicIf(TTrigger trigger, Func<TState> destinationStateSelector, Func<bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf(TTrigger trigger, Func<TState> destinationStateSelector, string destinationStateSelectorDescription, Func<bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf(TTrigger trigger, Func<TState> destinationStateSelector, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitDynamicIf(TTrigger trigger, Func<TState> destinationStateSelector, string destinationStateSelectorDescription, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitDynamicIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector, Func<bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector)
StateConfiguration PermitDynamicIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitDynamicIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, TState> destinationStateSelector, Func<bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, TState> destinationStateSelector, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitDynamicIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, TState> destinationStateSelector, Func<bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, TState> destinationStateSelector, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitDynamicIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector, Func<TArg0, bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, TState> destinationStateSelector, Reflection.DynamicStateInfos possibleDestinationStates = null, params Tuple<Func<TArg0, bool>, string>[] guards)
StateConfiguration PermitDynamicIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, TState> destinationStateSelector, Func<TArg0, TArg1, bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, TState> destinationStateSelector, Tuple<Func<TArg0, TArg1, bool>, string>[] guards, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, TState> destinationStateSelector, Func<TArg0, TArg1, TArg2, bool> guard, string guardDescription = null, Reflection.DynamicStateInfos possibleDestinationStates = null)
StateConfiguration PermitDynamicIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, TState> destinationStateSelector, Tuple<Func<TArg0, TArg1, TArg2, bool>, string>[] guards, Reflection.DynamicStateInfos possibleDestinationStates = null)
```
Func guard가 true를 반환하는 경우 지정된 trigger를 수락하고 Func destinationStateSelector에 의해 동적으로 계산된 대상 상태로 전환합니다.
<br/>

## PermitReentry [⭐]
```csharp
StateConfiguration PermitReentry(TTrigger trigger)
```
지정된 trigger를 수락하여 종료 작업을 실행하고 진입 작업을 다시 실행합니다. 재진입은 구성된 상태가 동일한 형제 상태로 전환되는 것처럼 동작합니다.
현재 상태에만 적용됩니다. 상위 상태 작업을 다시 실행하거나 상위 상태와 하위 상태 간 전환을 실행하도록 조치를 유발하지 않습니다.
<br/>

## PermitReentryIf [⭐]
```csharp
StateConfiguration PermitReentryIf(TTrigger trigger, Func<bool> guard, string guardDescription = null)
StateConfiguration PermitReentryIf(TTrigger trigger, params Tuple<Func<bool>, string>[] guards)
StateConfiguration PermitReentryIf<TArg0>(TriggerWithParameters<TArg0> trigger, Func<TArg0, bool> guard, string guardDescription = null)
StateConfiguration PermitReentryIf<TArg0>(TriggerWithParameters<TArg0> trigger, params Tuple<Func<TArg0, bool>, string>[] guards)
StateConfiguration PermitReentryIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, Func<TArg0, TArg1, bool> guard, string guardDescription = null)
StateConfiguration PermitReentryIf<TArg0, TArg1>(TriggerWithParameters<TArg0, TArg1> trigger, params Tuple<Func<TArg0, TArg1, bool>, string>[] guards)
StateConfiguration PermitReentryIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, Func<TArg0, TArg1, TArg2, bool> guard, string guardDescription = null)
StateConfiguration PermitReentryIf<TArg0, TArg1, TArg2>(TriggerWithParameters<TArg0, TArg1, TArg2> trigger, params Tuple<Func<TArg0, TArg1, TArg2, bool>, string>[] guards)
```
Func guard가 true를 반환하는 경우 지정된 trigger를 수락하여 종료 작업을 실행하고 진입 작업을 다시 실행합니다. 재진입은 구성된 상태가 동일한 형제 상태로 전환되는 것처럼 동작합니다.
현재 상태에만 적용됩니다. 상위 상태 작업을 다시 실행하거나 상위 상태와 하위 상태 간 전환을 실행하도록 조치를 유발하지 않습니다.
<br/>

## SubstateOf [⭐]
```csharp
StateConfiguration SubstateOf(TState superstate)
```
구성된 상태(TState)가 하위 상태가 되는 superstate(상위 상태)를 설정합니다. 즉, 상위 상태의 구성된 상태를 트리거할 수 있습니다.
하위 상태는 상위 상태의 허용된 전환을 상속합니다.
상위 상태 외부에서 하위 상태로 직접 들어가면 상위 상태에 대한 진입 작업이 실행됩니다.
마찬가지로 하위 상태에서 상위 상태 외부로 나갈 때 상위 상태에 대한 종료 작업이 실행됩니다.

🧩별도로 여러 구성 상태들을 하나의 상위 상태로 그룹화하여 해당 그룹 상태에 대한 조건 체크를 할 경우 사용할 수 있다.
	상태 Busy, A, B, C 가 있을 때 Configure(A).SubstateOf(Busy), Configure(B).SubstateOf(Busy), Configure(C).SubstateOf(Busy) 로 설정 후 IsInState(Busy) - [[🗒️Chapter 01. StateMachine Class#IsInState [⭐]]] 로 체크
<br/>
<br/>

---
