# Character States and Animations Demo Documentation

## Youtube Livestream
[![How to work with States and Animation in MapleStory Worlds](http://img.youtube.com/vi/wQXuV91kcR0/0.jpg)](https://www.youtube.com/live/wQXuV91kcR0?feature=Client/Server "Creator Livestream: How to work with States and Animation in MapleStory Worlds!")

## Mod Package Installation Instructions
[Mod File Installation Guide](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects?tab=readme-ov-file#project-import-instructions)

## Project File
[Download Latest Project File](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects/raw/refs/heads/main/Zakums_Curse/Zakums_Curse_V1_1.mod)


## Table of Contents
- [Overview](#overview)
- [Setup & Installation](#setup-and-installation)
- [StateMachine](#statemachine)
- [TriggerType](#triggertype)
- [TriggerAutoOff](#triggerautooff)
- [CustomStateController](#customstatecontroller)
- [Comparison](#comparison)
- [Comparison Enum](#comparisonenum)
- [Monster State Controller](#monsterstatecontroller)
- [Monster](#monster)
- [Monster Attack](#monsterattack)
- [Monster Hit](#monsterhit)
- [Custom Hit Event](#customhitevent)
- [Action Follow](#actionfollow)
- [Action Move Random](#actionmoverandom)
- [Action Stop](#actionstop)
- [Action Wait](#actionwait)
- [Attack Node](#attacknode)
- [Attack1 Node](#attack1node)
- [Attack2 Node](#attack2node)
- [DecoHasNoTarget](#decohasnotarget)
- [DecoHasTarget](#decohastarget)
- [DecoIsNotAttackRadius](#decoisnotattackradius)
- [Die Node](#dienode)
- [Hit Node](#hitnode)
- [Root Node](#rootnode)
- [Changelog & Downloads](#changelog)
---


## Overview
This documentation covers custom states and animations for monsters, as well as a basic implementation of the AI behavior tree. 
---
## Setup and Installation
* Attach Player Components to the DefaultPlayer Entity located under Workspace>DefaultPlayer
  * Player Attack Component
  * Player Hit Component
      * Ensure the "Collision Group" Property is set to "Player"
* Place the Model_monster-14 Model entity to the Heirarchy's world by right clicking on the model and selecting "Place to Heirarchy". 
* Test the world!!
---

## StateMachine

**Purpose**  
Manages entity state transitions in MapleStory Worlds by handling triggers, animation timing, and state-specific execution logic. This state type runs code when entering, exiting, and checking conditions for switching states.

---

### Properties

| Property             | Type      | Purpose |
|----------------------|-----------|---------|
| `triggers`           | table     | Collection of trigger objects for the current state. |
| `animationComplete`  | boolean   | Flag indicating if the state's animation has finished. |
| `animTimer`          | integer   | Timer ID for tracking when the animation completes. |
| `execTimer`          | integer   | Timer ID for running the state's execution logic at a certain animation point. |

---

### Method Summary

| Method               | Execution Scope | Purpose |
|----------------------|----------------|---------|
| `OnEnter`            | Client/Server  | Initializes triggers, schedules animation and execution timers when entering a state. |
| `OnExit`             | Client/Server  | Cleans up triggers, resets state flags, and clears timers. |
| `OnConditionCheck`   | Client/Server  | Validates if the next state transition should occur based on triggers or animation completion. |

---

### Methods

<details>
<summary>OnEnter</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `CustomStateController`, `SpriteRendererComponent`, `StateAnimationComponent`, `_SpriteUtils`, `_TimerService`

**Purpose**  
Sets up the state's triggers, calculates animation length, and schedules execution and animation completion timers.

**Behavior**  
1. Retrieves the current state's name and associated triggers from `CustomStateController`.  
2. Gets the corresponding animation from `StateAnimationComponent` and calculates its duration using `_SpriteUtils`.  
3. If the state has an `onExecute` function, schedules it to run at a percentage of the animation's duration (`executePct`).  
4. Schedules a timer to mark `animationComplete = true` when the animation ends.

</details>

<details>
<summary>OnExit</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `TriggerType`, `_TimerService`

**Purpose**  
Resets the state when exiting, clearing all timers and notifying triggers.

**Behavior**  
1. Iterates through all state triggers, calling their `OnExit` method.  
2. Resets `animationComplete` to `false`.  
3. Clears both `animTimer` and `execTimer` using `_TimerService`.

</details>

<details>
<summary>OnConditionCheck</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `TriggerType`

**Purpose**  
Determines whether the next state transition is allowed.

**Parameters**  
- `nextStateName` *(string)* – Name of the target state.

**Returns**  
- *(boolean)* – `true` if conditions are met, otherwise `false`.

**Behavior**  
1. If triggers are present, checks each one for compatibility with the target state using `HasComparison` and `Check`.  
2. Stops checking on the first failed trigger condition.  
3. If no triggers are assigned, returns `true` only if `animationComplete` is set.

</details>

---

### Related Types & Services
- **CustomStateController** – Retrieves and manages state and trigger data.  
- **SpriteRendererComponent** – Handles sprite rendering for the entity.  
- **StateAnimationComponent** – Provides animations and maps states to animation keys.  
- **_SpriteUtils** – Utility for calculating animation lengths.  
- **_TimerService** – Schedules and clears timers.  
- **TriggerType** – Represents conditions for transitioning between states.  
---

## StateInfo

**Purpose**  
Stores metadata and execution parameters for a specific state, including its name, execution timing, and an optional action to run during the state's lifecycle.

---

### Properties

| Property     | Type   | Purpose |
|--------------|--------|---------|
| `stateName`  | string | The name of the state. |
| `executePct` | number | The fraction (0–1) of the animation’s duration at which the `onExecute` action should be triggered. Default is `0.5`. |
| `onExecute`  | any    | The function or action to execute at the specified execution point. |

---

### Method Summary

| Method            | Execution Scope | Purpose |
|-------------------|-----------------|---------|
| `Init`            | Client/Server          | Initializes the state with the specified name. |
| `WithExecution`   | Client/Server          | Configures the execution timing and the action to run. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Initializes a `StateInfo` instance with the given state name.

**Parameters**  
- `stateName` *(string)* – The name of the state.

**Returns**  
- *(StateInfo)* – The updated instance for method chaining.

**Behavior**  
1. Sets `self.stateName` to the provided value.  
2. Returns the updated instance.

</details>

<details>
<summary>WithExecution</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Sets when and what execution logic should run for the state.

**Parameters**  
- `executePct` *(number)* – Percentage (0–1) of animation duration at which to execute the action.  
- `onExecute` *(any)* – The function or action to execute.

**Returns**  
- *(StateInfo)* – The updated instance for method chaining.

**Behavior**  
1. Updates `self.executePct` with the provided value.  
2. Sets `self.onExecute` to the given action.  
3. Returns the updated instance.

</details>

---

## TriggerType

**Purpose**  
Represents a state transition trigger in MapleStory Worlds, storing its value, associated connections, and comparison logic for validating state changes.

---

### Properties

| Property       | Type   | Purpose |
|----------------|--------|---------|
| `triggerID`    | string | Unique identifier for the trigger. |
| `value`        | any    | Current value of the trigger used for comparison checks. |
| `comparisons`  | table  | Mapping of target state names to their associated `Comparison` objects. |
| `connections`  | table  | List of state names that this trigger is linked to. |

---

### Method Summary

| Method             | Execution Scope | Purpose |
|--------------------|-----------------|---------|
| `Init`             | Client/Server          | Initializes the trigger with a value and unique ID. |
| `GetValue`         | Client/Server          | Retrieves the trigger’s current value. |
| `SetValue`         | Client/Server          | Updates the trigger’s value if the type matches the original. |
| `AddConnection`    | Client/Server          | Links the trigger to a state. |
| `HasConnection`    | Client/Server          | Checks if the trigger is linked to a specific state. |
| `HasComparison`    | Client/Server          | Checks if a comparison exists for a target state. |
| `AddComparison`    | Client/Server          | Assigns a comparison rule to a target state. |
| `OnExit`           | Client/Server          | Placeholder method called when the trigger exits a state. |
| `Check`            | Client/Server          | Evaluates the trigger’s value against a comparison for a target state. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Initializes a `TriggerType` with a starting value and unique trigger ID.

**Parameters**  
- `value` *(any)* – Initial trigger value.  
- `triggerID` *(string)* – Unique identifier for the trigger.

**Returns**  
- *(TriggerType)* – The updated instance for method chaining.

**Behavior**  
1. Assigns `value` and `triggerID` to the instance.  
2. Returns the instance for method chaining.

</details>

<details>
<summary>GetValue</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Retrieves the current trigger value.

**Returns**  
- *(any)* – The stored trigger value.

</details>

<details>
<summary>SetValue</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Updates the trigger’s value, ensuring the type matches the original value.

**Parameters**  
- `value` *(any)* – New value to assign.

**Behavior**  
1. Compares the type of the existing value to the new value.  
2. If types match, updates the value.  
3. If types differ, logs a warning and ignores the update.

</details>

<details>
<summary>AddConnection</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Links the trigger to a target state.

**Parameters**  
- `stateName` *(string)* – Name of the state to connect.

**Behavior**  
Adds the state name to the `connections` list.

</details>

<details>
<summary>HasConnection</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Checks if the trigger is linked to a specific state.

**Parameters**  
- `stateName` *(string)* – Target state name.

**Returns**  
- *(boolean)* – `true` if linked, otherwise `false`.

</details>

<details>
<summary>HasComparison</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Checks if a comparison rule exists for the given target state.

**Parameters**  
- `nextStateName` *(string)* – Name of the next state.

**Returns**  
- *(boolean)* – `true` if a comparison exists, otherwise `false`.

</details>

<details>
<summary>AddComparison</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Associates a comparison rule with a target state.

**Parameters**  
- `nextStateName` *(string)* – Name of the target state.  
- `comparison` *(Comparison)* – Comparison object defining evaluation rules.

</details>

<details>
<summary>OnExit</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Placeholder method for cleanup or trigger-specific exit behavior.  
(Currently empty implementation.)

</details>

<details>
<summary>Check</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `_ComparisonEnum`

**Purpose**  
Evaluates the trigger’s value against the comparison rules for the given target state.

**Parameters**  
- `nextStateName` *(string)* – Name of the target state.

**Returns**  
- *(boolean)* – Result of the comparison.

**Behavior**  
1. Retrieves the comparison for the given `nextStateName`.  
2. If found, evaluates based on `_ComparisonEnum` type:  
   - `E` – Equal to comparison value.  
   - `G` – Greater than comparison value.  
   - `GE` – Greater than or equal to comparison value.  
   - `L` – Less than comparison value.  
   - `LE` – Less than or equal to comparison value.  
3. Returns the evaluation result.

</details>

---

### Related Types & Enums
- **Comparison** – Defines a comparison type and value for evaluating triggers.  
- **_ComparisonEnum** – Enumeration of comparison operations (`E`, `G`, `GE`, `L`, `LE`).


## TriggerAutoOff

**Purpose**  
A specialized `TriggerType` that automatically resets its value to `false` whenever it exits a state.

---

### Inheritance
- **Extends**: `TriggerType`

---

### Method Summary

| Method   | Execution Scope | Purpose |
|----------|-----------------|---------|
| `OnExit` | Client/Server          | Resets the trigger value to `false` upon state exit. |

---

### Methods

<details>
<summary>OnExit</summary>

**Execution Scope**: `Client/Server`  
**Overrides**: `TriggerType.OnExit`

**Purpose**  
Ensures the trigger is disabled after leaving a state.

**Behavior**  
1. Sets `self.value` to `false`.

</details>


## CustomStateController

**Purpose**  
An extended `StateComponent` for managing state machines with associated triggers and state-specific execution data.  
It allows adding states with execution info, binding triggers to states, and managing conditional transitions.

---

### Inheritance
- **Extends**: `StateComponent`

---

### Properties

| Property         | Type  | Default | Purpose |
|------------------|-------|---------|---------|
| `triggers`       | table | `{}`    | Holds all registered `TriggerType` instances, keyed by `triggerID`. |
| `stateFunctions` | table | `{}`    | Stores `StateInfo` objects keyed by state name for execution logic. |

---

### Method Summary

| Method                           | Execution Scope | Purpose |
|----------------------------------|-----------------|---------|
| `AddStateWithInfo`               | Client/Server          | Adds a state to the controller with an associated `StateInfo`. |
| `AddStateFunction`               | Client/Server          | Registers a `StateInfo` without adding a state to the state machine. |
| `GetStateInfoByState`            | Client/Server          | Retrieves the `StateInfo` for a given state. |
| `GetTriggersByState`             | Client/Server          | Returns all triggers connected to a specific state. |
| `GetValue`                       | Client/Server          | Gets the value of a specific trigger by ID. |
| `SetValue`                       | Client/Server          | Sets the value of a specific trigger by ID. |
| `AddTrigger`                     | Client/Server          | Registers a new trigger. |
| `AddConditionWithTrigger`        | Client/Server          | Adds a transition condition to a state based on a trigger and comparison. |

---

### Methods

<details>
<summary>AddStateWithInfo</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Adds a new state to the state machine and associates it with the given `StateInfo`.

**Parameters**  
- `stateInfo` (`StateInfo`): The state information containing the state name and optional execution logic.

**Behavior**  
1. Adds the state to the controller with the `StateMachine` implementation.
2. Stores the `StateInfo` in `stateFunctions` keyed by state name.

</details>

---

<details>
<summary>AddStateFunction</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Registers a `StateInfo` object for a state without creating the state in the machine.

**Parameters**  
- `stateInfo` (`StateInfo`): The state information to register.

</details>

---

<details>
<summary>GetStateInfoByState</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Retrieves the `StateInfo` associated with a given state name.

**Parameters**  
- `stateName` (`string`): Name of the state.

**Returns**  
- `StateInfo` or `nil`

</details>

---

<details>
<summary>GetTriggersByState</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Finds all triggers that have a connection to the given state.

**Parameters**  
- `stateName` (`string`): The target state name.

**Returns**  
- `table` of `TriggerType` instances, or `nil` if no matches found.

</details>

---

<details>
<summary>GetValue</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Gets the current value of a trigger.

**Parameters**  
- `triggerID` (`string`): The trigger identifier.

**Returns**  
- `any`: The stored value.

</details>

---

<details>
<summary>SetValue</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Sets the value of a trigger, ensuring it exists.

**Parameters**  
- `triggerID` (`string`): The trigger identifier.  
- `value` (`any`): The value to set.

</details>

---

<details>
<summary>AddTrigger</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Registers a trigger into the controller.

**Parameters**  
- `triggerType` (`TriggerType`): The trigger instance to add.

</details>

---

<details>
<summary>AddConditionWithTrigger</summary>

**Execution Scope**: `Client/Server`  
**Purpose**  
Links a trigger to a state and defines a transition condition based on a comparison.

**Parameters**  
- `stateName` (`string`): Origin state.  
- `nextStateName` (`string`): Destination state.  
- `triggerID` (`string`): Trigger identifier.  
- `comparison` (`Comparison`): The comparison object.  
- `reverseResult` (`boolean`): If true, reverses the condition's evaluation result.

**Behavior**  
1. Finds the trigger by ID.  
2. Adds a connection and comparison for the transition.  
3. Registers the condition with the base `StateComponent`.  
4. Logs a warning if the trigger is not found.

</details>

## Comparison

**Purpose**  
A simple data structure for representing a comparison operation type and its associated value.  
Used primarily in state machine triggers to determine whether a condition is met.

---

### Properties

| Property | Type   | Default | Purpose |
|----------|--------|---------|---------|
| `type`   | string | `""`    | Defines the type of comparison (e.g., `"equal"`, `"greater"`, `"less"`). |
| `value`  | any    | `nil`   | The value to compare against during condition evaluation. |

---

### Method Summary

| Method  | Execution Scope | Purpose |
|---------|-----------------|---------|
| `Init`  | Client/Server   | Initializes the comparison with a type and value. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: Client/Server  
**Purpose**  
Initializes the `Comparison` instance with a specified type and comparison value.

**Parameters**  
- `type` (`string`): The comparison type (e.g., `"equal"`, `"greater"`).  
- `value` (`any`): The value to be used in the comparison.

**Returns**  
- `Comparison`: The initialized instance.

</details>

## ComparisonEnum

**Purpose**  
Defines string constants representing different comparison operators.  
Used by other scripts (e.g., `Comparison`, `TriggerType`) to standardize comparison types in conditional logic.

---

### Properties

| Property | Type   | Value | Purpose |
|----------|--------|-------|---------|
| `G`      | string | `">"` | Greater than comparison operator. |
| `GE`     | string | `">="` | Greater than or equal to comparison operator. |
| `L`      | string | `"<"` | Less than comparison operator. |
| `LE`     | string | `"<="` | Less than or equal to comparison operator. |
| `E`      | string | `"=="` | Equal to comparison operator. |

---

### Methods

This script does not define any methods.

## MonsterStateController

**Purpose**  
Controls a monster entity’s animation states, AI behavior, and state transitions in MapleStory Worlds.  
Integrates with `CustomStateController` to define triggers, states, and transitions, while also managing AI logic through a behavior tree.

---

### Properties

| Property             | Type    | Purpose |
|----------------------|---------|---------|
| `currentStateName`   | string  | Tracks the monster's current animation/logic state. |

---

### Method Summary

| Method                  | Execution Scope | Purpose |
|-------------------------|-----------------|---------|
| `OnBeginPlay`           | ServerOnly      | Initializes component references and sets up animation and AI states. |
| `SetupAnimationStates`  | Client/Server   | Defines animation states, execution points, and transitions. |
| `SetupAIStates`         | Server          | Builds the monster’s behavior tree logic. |
| `StandState`            | Client/Server   | Stops movement animations. |
| `MoveState`             | Client/Server   | Starts movement animations. |
| `AttackState`           | Client/Server   | Sets attack triggers and randomizes attack animations. |
| `Attack`                | Client/Server   | Executes a monster attack with given damage. |
| `GetCurrentState`       | Client/Server   | Returns the current state name from the state controller. |
| `OnHit`                 | Client/Server   | Marks the monster as hit. |
| `HitState`              | Client/Server   | Switches to hit animation if not dead. |
| `OnHitEnd`              | Client/Server   | Clears the hit status. |
| `DieState`              | Client/Server   | Triggers death animation and state flag. |
| `Die`                   | Client/Server   | Executes monster death behavior. |
| `Test`                  | Server          | Debug helper to trigger attacks. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ServerOnly`  
**Dependencies**: `CustomStateController`, `MovementComponent`, `Monster`, `MonsterAttack`

**Purpose**  
Initializes all required components for controlling the monster's state and sets up animation and AI logic.

**Behavior**  
1. Retrieves component references.  
2. Calls `SetupAnimationStates()` to configure animations.  
3. Calls `SetupAIStates()` to configure AI if running on the server.

</details>

<details>
<summary>SetupAnimationStates</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `CustomStateController`, `StateInfo`, `TriggerBool`, `TriggerNumber`

**Purpose**  
Defines all available animation states and their transitions.

**Behavior**  
1. Registers states such as `STAND`, `MOVE`, `ATTACK`, `ATTACK2`, `HIT`, `DIE`.  
2. Configures execution timing with `WithExecution()` for attack states.  
3. Sets up triggers: `attack`, `attackIndex`, `hit`, `isDead`, `isMove`.  
4. Defines transitions based on trigger conditions.  

</details>

<details>
<summary>SetupAIStates</summary>

**Execution Scope**: `Server`  
**Dependencies**: `AIComponent`, `AISelectorNode`, `AISequenceNode`, `DieNode`, `HitNode`, `AttackNode`, `FollowNode`, `PatrolNode`, `WaitNode`, `MoveRandomNode`

**Purpose**  
Builds a hierarchical AI behavior tree for the monster.

**Behavior**  
1. Creates root `AISelectorNode`.  
2. Adds `DieNode` to handle death immediately when flagged.  
3. Creates combat sequence:
   - If hit, run `HitNode`.  
   - If target in range, run `AttackNode`.  
   - Otherwise, follow the target using `FollowNode`.  
4. Adds patrol sequence for when no target is present.  

</details>

<details>
<summary>StandState</summary>

**Execution Scope**: `Client/Server`  
Sets `"isMove"` trigger to `false`.

</details>

<details>
<summary>MoveState</summary>

**Execution Scope**: `Client/Server`  
Sets `"isMove"` trigger to `true`.

</details>

<details>
<summary>AttackState</summary>

**Execution Scope**: `Client/Server`  
Sets `"attack"` to `true` and randomly selects `"attackIndex"` from 1 or 2.

</details>

<details>
<summary>Attack</summary>

**Execution Scope**: `Client/Server`  
**Parameters**: `damage` *(number)* — Amount of damage to inflict.  
Calls the `MonsterAttack` component to execute an attack.

</details>

<details>
<summary>GetCurrentState</summary>

**Execution Scope**: `Client/Server`  
**Returns**: *(string)* — The current state's name from the `CustomStateController`.

</details>

<details>
<summary>OnHit</summary>

**Execution Scope**: `Client/Server`  
Marks the monster as hit by setting `isHit = true`.

</details>

<details>
<summary>HitState</summary>

**Execution Scope**: `Client/Server`  
If not dead, transitions the monster to `"HIT"` state.

</details>

<details>
<summary>OnHitEnd</summary>

**Execution Scope**: `Client/Server`  
Resets `isHit = false`.

</details>

<details>
<summary>DieState</summary>

**Execution Scope**: `Client/Server`  
Switches to `"DIE"` state and sets `"isDead"` trigger to `true`.

</details>

<details>
<summary>Die</summary>

**Execution Scope**: `Client/Server`  
Calls the monster's death logic.

</details>

<details>
<summary>Test</summary>

**Execution Scope**: `Server`  
Triggers debug attack behavior.

</details>

---

### Event Handlers

| Event                | Sender           | Execution Scope | Purpose |
|----------------------|------------------|-----------------|---------|
| `KeyDownEvent`       | `InputService`   | Client          | If key `F` is pressed, calls `Test()`. |
| `CustomHitEvent`     | Self             | Client/Server   | Transitions to `"HIT"` state when hit. |

---

### Related Types & Services
- **CustomStateController** – Manages states and triggers.  
- **MovementComponent** – Controls monster movement.  
- **Monster** – Monster entity logic.  
- **MonsterAttack** – Handles attack execution.  
- **AIComponent** – Runs the behavior tree.  
- **TriggerBool / TriggerNumber** – Defines triggers for state changes.  
- **SelectorNode / SequenceNode** – Behavior tree node types.  

## Monster

**Purpose**  
Represents a monster entity in MapleStory Worlds, handling health, respawn, movement, and interactions with players and attacks.

---

### Properties

| Property            | Type     | Default | Purpose |
|---------------------|---------|---------|---------|
| `MaxHp`             | number  | 100     | Maximum health points of the monster. |
| `Hp`                | number  | 0       | Current health points. |
| `RespawnOn`         | boolean | false   | Determines if the monster should respawn after death. |
| `IsDead`            | boolean | false   | Tracks if the monster is dead. Hidden from inspector. |
| `RespawnDelay`      | number  | 5       | Delay in seconds before the monster respawns. |
| `DestroyDelay`      | number  | 0.6     | Delay in seconds before the entity is hidden/destroyed after death. |
| `detectionDistance` | number  | 4.0     | Maximum distance to detect players. |
| `isHit`             | boolean | false   | Flag indicating if the monster has been hit. |

---

### Method Summary

| Method                | Execution Scope | Purpose |
|-----------------------|----------------|---------|
| `OnBeginPlay`         | Client/Server  | Initializes the monster’s HP. |
| `Dead`                | ServerOnly     | Handles death logic, hides entity, and optionally destroys it. |
| `Respawn`             | ServerOnly     | Resets the monster to alive state and restores HP. |
| `IsInAttackRange`     | Client/Server  | Checks if a given position is within attack range. |
| `Flip`                | Client/Server  | Flips the monster sprite horizontally. |
| `GetNearestFoothold`  | Client/Server  | Returns the closest foothold within a given distance. |
| `GetNearestPlayer`    | Client/Server  | Finds the nearest player within detection distance. |
| `lookAtTarget`        | Client/Server  | Flips the monster to face a target entity. |
| `Push`                | Client/Server  | Applies a push force away from a point (e.g., attack center). |
| `HandleHitEvent`      | ServerOnly     | Handles incoming hit events, applies damage, and triggers death/respawn if needed. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Initializes the monster’s HP when the entity is spawned.

**Behavior**  
- Sets `Hp` to `MaxHp`.

</details>

<details>
<summary>Dead</summary>

**Execution Scope**: `ServerOnly`

**Purpose**  
Handles death of the monster, disables visibility and interaction, and optionally destroys the entity.

**Behavior**  
1. Sets `IsDead` to `true`.  
2. Schedules a timer (`DestroyDelay`) to hide and disable the entity.  
3. If `RespawnOn` is false, destroys the entity.

</details>

<details>
<summary>Respawn</summary>

**Execution Scope**: `ServerOnly`

**Purpose**  
Restores the monster to a live state, resets HP, and changes its state to idle.

**Behavior**  
1. Sets `IsDead` to `false`.  
2. Makes the entity visible and enabled.  
3. Resets `Hp` to `MaxHp`.  
4. Changes the state to `"IDLE"` if a state component exists.

</details>

<details>
<summary>IsInAttackRange</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Determines whether a given position is within attack range of the monster.

**Parameters**  
- `position` *(Vector3)* – Target position to check.

**Returns**  
- *(boolean)* – `true` if the position is within half of the monster's attack radius.

</details>

<details>
<summary>Flip</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Flips the monster sprite horizontally.

**Parameters**  
- `flipX` *(boolean)* – Whether to flip the sprite along the X-axis.

</details>

<details>
<summary>GetNearestFoothold</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Finds the nearest foothold within a specified distance for positioning logic.

**Parameters**  
- `distance` *(number)* – Maximum distance to search for footholds.

**Returns**  
- *(Foothold/any)* – The closest foothold or `nil` if none is found.

</details>

<details>
<summary>GetNearestPlayer</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Finds the nearest player entity within the monster's detection range.

**Returns**  
- *(Entity)* – The closest player entity or `nil` if none are within range.

</details>

<details>
<summary>lookAtTarget</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Flips the monster sprite to face a target entity.

**Parameters**  
- `target` *(Entity)* – The entity to face.

</details>

<details>
<summary>Push</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Applies a push force to the monster away from a point, temporarily disabling movement.

**Parameters**  
- `attackCenter` *(Vector2)* – The point from which to push the monster.

**Behavior**  
1. Disables the monster's movement.  
2. Applies a force in the direction away from `attackCenter`.  
3. Re-enables movement after 1 second using `_TimerService`.

</details>

<details>
<summary>HandleHitEvent</summary>

**Execution Scope**: `ServerOnly`  
**Event Source**: `Self`

**Purpose**  
Handles when the monster is hit by an attack, applying damage, push, and triggering death/respawn if HP reaches zero.

**Parameters**  
- `event` *(CustomHitEvent)* – Contains attacker info, damage, and other hit data.

**Behavior**  
1. Reduces `Hp` by the total damage.  
2. Applies push force if the monster is still alive.  
3. If HP reaches 0 and `RespawnOn` is true, schedules respawn after `RespawnDelay`.  
4. If HP reaches 0 and `RespawnOn` is false, calls `Dead()` to hide or destroy the entity.

</details>

---

### Related Types & Services
- **_TimerService** – Used for scheduling delayed actions.  
- **MonsterStateController** – Controls the monster's state machine and animations.  
- **MonsterAttack** – Provides attack range and attack logic.  
- **MovementComponent** – Handles entity movement.  
- **TransformComponent** – Provides position data.  
- **SpriteRendererComponent** – Handles sprite rendering and flipping.  
- **_UserService** – Accesses players in the current map.  
- **RigidbodyComponent** – Applies forces for physics interactions.  
- **CustomHitEvent** – Event data for hits on the monster.  

## MonsterAttack

**Purpose**  
Handles a monster's attack logic in MapleStory Worlds, including attack area calculation, damage application, and detection of valid targets.

---

### Properties

| Property          | Type      | Default      | Purpose |
|------------------|----------|-------------|---------|
| `AttackInterval`  | number   | 0.03        | Interval in seconds between repeated attacks (currently unused in this script). |
| `Shape`           | any      | nil         | Represents the attack hitbox (BoxShape). |
| `SpriteSize`      | Vector2  | (0,0)       | Size of the attack hitbox, derived from the monster's sprite. |
| `PositionOffset`  | Vector2  | (0,0)       | Offset of the attack hitbox relative to the monster's position. |

---

### Method Summary

| Method                | Execution Scope | Purpose |
|-----------------------|----------------|---------|
| `OnBeginPlay`         | ServerOnly     | Initializes the attack hitbox size and position using the monster's sprite. |
| `AttackNear`          | Client/Server  | Performs an attack in the area defined by the hitbox and applies damage. |
| `CalcDamage`          | Client/Server  | Calculates damage for an attack based on serialized attack info. |
| `IsAttackTarget`      | Client/Server  | Determines if a given entity is a valid attack target. |
| `GetAttackRadius`     | Client/Server  | Returns the effective horizontal range of the monster's attack. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ServerOnly`  

**Purpose**  
Initializes the monster’s attack hitbox and calculates sprite-based dimensions for attack detection.

**Behavior**  
1. Retrieves the monster entity attached to this component.  
2. Creates a default `BoxShape` for the attack hitbox.  
3. Preloads the monster's sprite resource.  
4. Calculates the `SpriteSize` in world units and `PositionOffset` based on sprite pivot.  
5. (Optional commented logic) A repeating timer could attack nearby players periodically.

</details>

<details>
<summary>AttackNear</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Performs a melee attack in the area defined by the hitbox and applies damage to valid targets.

**Parameters**  
- `damage` *(number)* – Amount of damage to apply.

**Behavior**  
1. Retrieves the monster’s world transform.  
2. Updates the attack `Shape` position, size, and rotation based on the sprite and entity transform.  
3. Creates an `attackInfo` table with the damage value.  
4. Calls `AttackFast` to apply the attack to entities in the `Player` collision group.

</details>

<details>
<summary>CalcDamage</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates the damage dealt to a target based on serialized attack info.

**Parameters**  
- `attacker` *(Entity)* – The entity performing the attack.  
- `defender` *(Entity)* – The entity being attacked.  
- `attackInfo` *(string)* – Serialized table containing attack parameters.

**Returns**  
- *(integer)* – Damage value extracted from `attackInfo`.

</details>

<details>
<summary>IsAttackTarget</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Checks whether a given entity is a valid attack target for this monster.

**Parameters**  
- `defender` *(Entity)* – Candidate target entity.  
- `attackInfo` *(string)* – Serialized attack info (currently not used in filtering).  

**Returns**  
- *(boolean)* – `true` if the entity is a valid player target, `false` otherwise.

**Behavior**  
- Ensures the defender has a `PlayerComponent`.  
- Calls base class logic (`__base:IsAttackTarget`) for additional validation.

</details>

<details>
<summary>GetAttackRadius</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns the horizontal range of the monster’s attack based on its sprite size and entity scale.

**Returns**  
- *(number)* – Effective attack radius along the X-axis.

**Behavior**  
1. Retrieves the entity’s transform component.  
2. Multiplies `SpriteSize.x` by the absolute scale to get world-space attack range.

</details>

---

### Related Types & Services
- **BoxShape** – Represents the attack hitbox.  
- **Monster** – Provides entity reference and sprite information.  
- **_ResourceService** – Used to preload sprite resources and retrieve sprite dimensions.  
- **_TimerService** – Optionally used for repeated attacks.  
- **AttackFast** – Applies attack logic to entities within a collision group.  
- **CollisionGroups.Player** – Target group for player entities.  

## MonsterHit

**Purpose**  
Handles applying damage effects and notifying the entity when it has been hit by an attack in MapleStory Worlds.

---

### Method Summary

| Method       | Execution Scope | Purpose |
|--------------|----------------|---------|
| `OnHit`      | Client/Server  | Applies damage visuals and sends a `CustomHitEvent` to notify the entity of the hit. |

---

### Methods

<details>
<summary>OnHit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Triggers damage feedback (damage skin) and notifies the monster entity that it has been hit.

**Parameters**  
- `attacker` *(Entity)* – The entity that caused the hit.  
- `damage` *(integer)* – Amount of damage dealt.  
- `isCritical` *(boolean)* – Indicates whether the hit was a critical strike.  
- `attackInfo` *(string)* – Additional serialized attack information.  
- `hitCount` *(int32)* – Number of hits in a multi-hit attack (currently unused).

**Behavior**  
1. Plays the damage skin effect using `_DamageSkinService` to display damage numbers.  
2. Creates a `CustomHitEvent` and sets properties including attacker, damage, critical status, extra info, total damage, and attack center.  
3. Sends the `CustomHitEvent` to the monster entity to trigger further reactions such as knockback or state changes.

</details>

---

### Related Types & Services
- **CustomHitEvent** – Event type used to notify entities of being hit.  
- **_DamageSkinService** – Handles displaying damage numbers on screen.  
- **DamageSkinTweenType** – Enum for animation type of damage skins.  

## CustomHitEvent

**Purpose**  
Represents a custom hit event that is sent to an entity when it is damaged. Used to carry information about the attack and trigger appropriate reactions in the entity.

---

### Properties

| Property           | Type      | Purpose |
|-------------------|-----------|---------|
| `attackerEntity`   | Entity    | The entity that performed the attack. |
| `damages`          | table     | Table of individual damage values applied. |
| `isCritical`       | boolean   | Indicates if the hit was a critical strike. |
| `extra`            | string    | Additional serialized attack information. |
| `totalDamage`      | number    | Total damage inflicted by this hit. |
| `attackCenter`     | Vector2   | World position where the attack originated or hit center. |

---

### Related Types & Services
- **EventType** – Base type for all event structures in MapleStory Worlds.  
- **MonsterHit** – Component that listens for this event to trigger hit reactions.  



## ActionFollow

**Purpose**  
A behavior tree node that moves a monster toward its nearest player target in MapleStory Worlds. This action keeps the monster in a "follow" state until the player is reached or no target is found.

---

### Properties

| Property        | Type                     | Purpose |
|-----------------|--------------------------|---------|
| `monsterState`  | `MonsterStateController` | Controls the monster's AI states, including movement. |
| `monster`       | `Monster`                | Reference to the monster entity that will follow a target. |

---

### Method Summary

| Method       | Execution Scope | Purpose |
|--------------|----------------|---------|
| `OnInit`     | Client/Server  | Initializes references to the monster and state controller, and sets the monster to the movement state. |
| `OnBehave`   | Client/Server  | Executes follow behavior by moving the monster toward the nearest player. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `Monster`, `MonsterStateController`

**Purpose**  
Prepares the monster for follow behavior by linking required components and enabling movement state.

**Behavior**  
1. Retrieves the monster instance from `ParentAI.Entity`.  
2. Retrieves the monster's state controller from `ParentAI.Entity`.  
3. Calls `MoveState()` to switch the monster into its movement-ready state.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `Monster`, `MovementComponent`, `TransformComponent`, `BehaviourTreeStatus`

**Purpose**  
Determines follow behavior toward the nearest player.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last behavior tick.

**Returns**  
- `BehaviourTreeStatus.Running` – Monster is moving toward the target.  
- `BehaviourTreeStatus.Success` – Monster has reached the target.  
- `BehaviourTreeStatus.Failure` – No target is available.

**Behavior**  
1. Finds the nearest player using `monster:GetNearestPlayer()`.  
2. If no target exists, returns `Failure`.  
3. Calculates the 2D directional vector and distance to the target.  
4. If distance is greater than 0:  
   - Normalizes direction.  
   - Moves the monster toward the target using `MovementComponent:MoveToDirection()`.  
   - Flips the monster sprite based on horizontal direction.  
   - Returns `Running`.  
5. If already at target location, returns `Success`.

</details>

---

### Related Types & Services
- **Monster** – Entity type representing an enemy character.  
- **MonsterStateController** – Handles AI state transitions for the monster.  
- **MovementComponent** – Manages directional movement logic for entities.  
- **TransformComponent** – Provides position and orientation data.  
- **BehaviourTreeStatus** – Enum representing behavior execution status (`Running`, `Success`, `Failure`).  

## ActionMoveRandom

**Purpose**  
A behavior tree node that causes a monster to move in a random direction for a set amount of time within horizontal foothold bounds in MapleStory Worlds.

---

### Properties

| Property        | Type                     | Purpose |
|-----------------|--------------------------|---------|
| `monsterState`  | `MonsterStateController` | Controls the monster's AI states, including movement. |
| `monster`       | `Monster`                | Reference to the monster entity executing this behavior. |
| `MoveDirection` | `Vector2`                | Direction vector in which the monster will move. |
| `MoveTime`      | `number`                 | Duration (in seconds) the monster will continue moving in the chosen direction. |
| `ElapsedTime`   | `number`                 | Time elapsed since movement began. |
| `horizBounds`   | `Vector2`                | Minimum and maximum horizontal X boundaries within which the monster can move. |

---

### Method Summary

| Method       | Execution Scope | Purpose |
|--------------|----------------|---------|
| `OnInit`     | Client/Server  | Initializes monster movement state, selects a random movement direction, calculates horizontal bounds from foothold data, and starts movement. |
| `OnBehave`   | Client/Server  | Continues movement until the time limit is reached or boundary conditions require a direction flip. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `Monster`, `MonsterStateController`, `MovementComponent`, `FootholdComponent`, `_UtilLogic`

**Purpose**  
Sets up random movement direction, determines horizontal movement limits from foothold layout, and initiates movement.

**Behavior**  
1. Retrieves monster and its state controller from `ParentAI.Entity`.  
2. Switches monster to movement-ready state using `MoveState()`.  
3. Generates a random direction vector (`randomX`, `randomY`) between -0.5 and 0.5.  
4. Retrieves the nearest foothold and calculates leftmost and rightmost footholds in the chain.  
5. Determines horizontal bounds (`horizBounds`) from foothold positions with a small margin.  
6. Normalizes the random direction vector and sets movement duration between 2–4 seconds.  
7. Resets `ElapsedTime` to 0.  
8. Initiates movement in the chosen direction via `MovementComponent:MoveToDirection()`.  
9. Flips monster sprite based on movement direction.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `MovementComponent`, `TransformComponent`, `BehaviourTreeStatus`

**Purpose**  
Executes ongoing random movement, checks time limit, and adjusts direction if horizontal bounds are exceeded.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last behavior tick.

**Returns**  
- `BehaviourTreeStatus.Success` – Movement duration has elapsed.  
- `BehaviourTreeStatus.Running` – Monster is still moving within time limit.

**Behavior**  
1. Increments `ElapsedTime` by `delta`.  
2. If `ElapsedTime` exceeds `MoveTime`, stops movement and returns `Success`.  
3. Checks if the monster’s current X position has passed left or right bounds:  
   - If exceeded, reverses movement direction horizontally.  
   - Updates monster sprite flip accordingly.  
4. Continues returning `Running` while movement is ongoing.

</details>

---

### Related Types & Services
- **Monster** – Entity type representing an enemy character.  
- **MonsterStateController** – Handles AI state transitions for the monster.  
- **MovementComponent** – Controls entity movement in a specified direction.  
- **TransformComponent** – Provides position and orientation data.  
- **FootholdComponent** – Manages foothold-based navigation and positioning.  
- **_UtilLogic** – Utility class providing helper functions such as random number generation.  
- **BehaviourTreeStatus** – Enum representing behavior execution status (`Running`, `Success`, `Failure`).  

## ActionStop

**Purpose**  
A behavior tree node that immediately stops the monster’s movement in MapleStory Worlds.

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnBehave` | Client/Server  | Stops the monster's movement and returns a success status. |

---

### Methods

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `MovementComponent`, `BehaviourTreeStatus`

**Purpose**  
Halts all movement for the entity executing this behavior.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last behavior tick (not used in this method).

**Returns**  
- `BehaviourTreeStatus.Success` – Indicates the stop action was executed successfully.

**Behavior**  
1. Retrieves the entity’s `MovementComponent` from `ParentAI.Entity`.  
2. If `MovementComponent` is present, calls its `Stop()` method to halt all movement.  
3. Returns `Success` to signal the behavior completed.

</details>

---

### Related Types & Services
- **MovementComponent** – Controls and stops entity movement.  
- **BehaviourTreeStatus** – Enum representing behavior execution status (`Running`, `Success`, `Failure`).  

## ActionWait

**Purpose**  
A behavior tree node that pauses execution for a set duration before returning success in MapleStory Worlds.

---

### Property Summary

| Property       | Type    | Default | Purpose |
|----------------|--------|---------|---------|
| `ElapsedTime`  | number | 0       | Tracks the total elapsed time since the wait began. |
| `Time`         | number | 2       | The duration (in seconds) the node should wait before succeeding. |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Resets the elapsed time to zero at the start of the wait. |
| `OnBehave` | Client/Server  | Increments elapsed time each tick and determines whether to continue waiting or succeed. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`

**Purpose**  
Initializes the node’s state by resetting the elapsed time counter.

**Behavior**  
- Sets `ElapsedTime` to `0`.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `BehaviourTreeStatus`

**Purpose**  
Controls the wait behavior, tracking time until the set duration is reached.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last behavior tick.

**Returns**  
- `BehaviourTreeStatus.Success` – When total elapsed time reaches or exceeds `Time`.  
- `BehaviourTreeStatus.Running` – If still within the wait duration.

**Behavior**  
1. Add the `delta` time to `ElapsedTime`.  
2. If `ElapsedTime >= Time`, return `Success`.  
3. Otherwise, return `Running`.

</details>

---

### Related Types & Services
- **BehaviourTreeStatus** – Enum representing behavior execution status (`Running`, `Success`, `Failure`).  

## AttackNode

**Purpose**  
Behavior tree node that manages monster attack behavior. Handles setting the target, initiating attack animations, and checking whether the monster can continue attacking.

---

### Properties

| Property           | Type                        | Purpose |
|-------------------|-----------------------------|---------|
| `monsterState`     | MonsterStateController      | Reference to the monster's state controller for changing and querying attack states. |
| `monster`          | Monster                     | Reference to the monster entity. |
| `attackTarget`     | Entity                      | The current target the monster is attacking. |

---

### Method Summary

| Method             | Execution Scope | Purpose |
|-------------------|----------------|---------|
| `SetAttackTarget`  | Client/Server  | Sets the entity that the monster should attack. |
| `OnInit`           | Client/Server  | Initializes references, sets attack state, and orients the monster toward the target. |
| `OnBehave`         | Client/Server  | Determines the behavior tree status based on current state and hit condition. |

---

### Methods

<details>
<summary>SetAttackTarget</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns a specific entity as the monster's attack target.

**Parameters**  
- `target` *(Entity)* – The entity to attack.

**Behavior**  
1. Stores the provided target in `attackTarget`.

</details>

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Prepares the node for execution by initializing references, starting the attack state, and orienting the monster toward the nearest player.

**Behavior**  
1. Retrieves references to `Monster` and `MonsterStateController`.  
2. Calls `AttackState()` on `monsterState` to trigger the attack animation.  
3. Finds the nearest player and makes the monster face the target using `lookAtTarget`.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns the behavior tree status based on whether the monster is currently attacking or has been hit.

**Returns**  
- `BehaviourTreeStatus.Running` if attack animation is ongoing.  
- `BehaviourTreeStatus.Success` if attack is complete.  
- `BehaviourTreeStatus.Failure` if the monster is hit.

**Behavior**  
1. If `monster.isHit` is `true`, returns `Failure`.  
2. If `monsterState` is currently in `"ATTACK"` state, returns `Running`.  
3. Otherwise, returns `Success`.

</details>

---

### Related Types & Services
- **MonsterStateController** – Manages the monster’s state machine and transitions.  
- **Monster** – Represents the monster entity.  
- **BehaviourTreeStatus** – Enum for behavior tree node execution results (`Success`, `Failure`, `Running`).  

## Attack1Node

**Purpose**  
Specialized behavior tree node for the monster’s first attack type. Extends `AttackNode` and triggers the attack animation for the first attack variant.

---

### Inheritance

- **Base Class**: `AttackNode`

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes the node and triggers the first attack animation. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Prepares the node by triggering the monster’s first attack state.

**Behavior**  
1. Calls `AttackState(1)` on the `monsterState` to start the first attack animation.  

</details>

---

### Related Types & Services
- **AttackNode** – Base behavior tree node handling generic attack logic.  
- **MonsterStateController** – Manages monster state transitions and animations.  

## Attack2Node

**Purpose**  
Specialized behavior tree node for the monster’s second attack type. Extends `AttackNode` and triggers the attack animation for the second attack variant.

---

### Inheritance

- **Base Class**: `AttackNode`

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes the node and triggers the second attack animation. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Prepares the node by triggering the monster’s second attack state.

**Behavior**  
1. Calls the base class `OnInit()` to execute common attack initialization logic.  
2. Calls `AttackState(1)` on `monsterState` to start the second attack animation.

</details>

---

### Related Types & Services
- **AttackNode** – Base behavior tree node handling generic attack logic.  
- **MonsterStateController** – Manages monster state transitions and animations.  

## DecoHasNoTarget

**Purpose**  
Behavior tree decorator node that executes its child node only if the monster currently has no target in range. Used to control AI flow when the monster is idle or patrolling.

---

### Properties

| Property              | Type                       | Purpose |
|-----------------------|----------------------------|---------|
| `Child`               | `any`                      | The child node to execute if no target is found. |
| `monsterState`        | `MonsterStateController`   | Reference to the monster’s state controller for managing state transitions. |
| `monster`             | `Monster`                  | Reference to the monster entity itself. |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes references to the monster and its state controller. |
| `OnBehave` | Client/Server  | Checks for nearby player targets and executes the child node if none are found. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up references to the monster entity and its state controller.

**Behavior**  
1. Assigns `self.monster` to the parent AI’s monster entity.  
2. Assigns `self.monsterState` to the parent AI’s monster state controller.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Determines whether to execute the child node based on the presence of nearby players.

**Behavior**  
1. Calls `GetNearestPlayer()` on the monster to detect nearby targets.  
2. If no target is found, executes `self.Child:Behave(delta)` and returns its status.  
3. If a target is present, returns `BehaviourTreeStatus.Failure` to halt child execution.

</details>

---

### Related Types & Services
- **Monster** – Provides player detection and movement logic.  
- **MonsterStateController** – Controls the monster’s current animation and state.  

## DecoHasTarget

**Purpose**  
Behavior tree decorator node that executes its child node only if the monster currently has a target in range. Used to control AI actions when the monster detects a nearby player.

---

### Properties

| Property              | Type                       | Purpose |
|-----------------------|----------------------------|---------|
| `Child`               | `any`                      | The child node to execute if a target is found. |
| `monsterState`        | `MonsterStateController`   | Reference to the monster’s state controller for managing state transitions. |
| `monster`             | `Monster`                  | Reference to the monster entity itself. |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes references to the monster and its state controller. |
| `OnBehave` | Client/Server  | Checks for nearby player targets and executes the child node if a target is found. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up references to the monster entity and its state controller.

**Behavior**  
1. Assigns `self.monster` to the parent AI’s monster entity.  
2. Assigns `self.monsterState` to the parent AI’s monster state controller.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Determines whether to execute the child node based on the presence of nearby players.

**Behavior**  
1. Calls `GetNearestPlayer()` on the monster to detect nearby targets.  
2. If a target is found, executes `self.Child:Behave(delta)` and returns its status.  
3. If no target is present, returns `BehaviourTreeStatus.Failure` to halt child execution.

</details>

---

### Related Types & Services
- **Monster** – Provides player detection and movement logic.  
- **MonsterStateController** – Controls the monster’s current animation and state.  

## DecoIsNotAttackRadius

**Purpose**  
Behavior tree decorator node that executes its child node only if a detected player target is outside the monster’s attack range. This is used to control AI behavior when the monster needs to move or reposition before attacking.

---

### Properties

| Property              | Type                       | Purpose |
|-----------------------|----------------------------|---------|
| `Child`               | `any`                      | The child node to execute if a target is outside attack range. |
| `monster`             | `Monster`                  | Reference to the monster entity for detecting players and attack range. |
| `monsterState`        | `MonsterStateController`   | Reference to the monster’s state controller for managing state transitions. |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes references to the monster and its state controller. |
| `OnBehave` | Client/Server  | Checks if a player is outside the attack range and executes the child node if so. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up references to the monster entity and its state controller.

**Behavior**  
1. Assigns `self.monster` to the parent AI’s monster entity.  
2. Assigns `self.monsterState` to the parent AI’s monster state controller.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Determines whether to execute the child node based on the target’s distance relative to the monster’s attack range.

**Behavior**  
1. Calls `GetNearestPlayer()` on the monster to detect a nearby target.  
2. If no target is found, returns `BehaviourTreeStatus.Failure`.  
3. If the target is within the monster’s attack range (`IsInAttackRange` returns `true`), returns `BehaviourTreeStatus.Failure`.  
4. If the target is outside the attack range, executes `self.Child:Behave(delta)` and returns its status.

</details>

---

### Related Types & Services
- **Monster** – Provides player detection, attack range, and movement logic.  
- **MonsterStateController** – Controls the monster’s current animation and state.


## DieNode

**Purpose**  
Behavior tree node that handles the monster’s death logic. It ensures the monster enters the "DIE" state and prevents further AI actions when the monster is dead.

---

### Properties

| Property           | Type                       | Purpose |
|-------------------|----------------------------|---------|
| `monster`          | `Monster`                  | Reference to the monster entity for checking death state. |
| `monsterState`     | `MonsterStateController`   | Reference to the monster’s state controller for triggering the death state. |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes references to the monster and its state controller. |
| `OnBehave` | Client/Server  | Checks if the monster is dead and triggers the "DIE" state if so. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up references to the monster entity and its state controller.

**Behavior**  
1. Assigns `self.monster` to the parent AI’s monster entity.  
2. Assigns `self.monsterState` to the parent AI’s monster state controller.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Executes death logic if the monster is dead.

**Behavior**  
1. Checks if `self.monster.IsDead` is `false`. If so, returns `BehaviourTreeStatus.Failure`.  
2. If the monster is dead, calls `self.monsterState:DieState()` to change the state to "DIE".  
3. Returns `BehaviourTreeStatus.Running` to indicate the death behavior is in progress.

</details>

---

### Related Types & Services
- **Monster** – Provides health, death status, and entity visibility/respawn logic.  
- **MonsterStateController** – Manages animation and state transitions for the monster.


## HitNode

**Purpose**  
Behavior tree node that monitors if a monster is in the "hit" state. Prevents other AI actions from executing while the monster is being hit.

---

### Properties

| Property           | Type                       | Purpose |
|-------------------|----------------------------|---------|
| `monster`          | `Monster`                  | Reference to the monster entity to check the hit status. |
| `monsterState`     | `MonsterStateController`   | Reference to the monster’s state controller (not actively used in behavior but initialized for potential extensions). |

---

### Method Summary

| Method     | Execution Scope | Purpose |
|------------|----------------|---------|
| `OnInit`   | Client/Server  | Initializes references to the monster and its state controller. |
| `OnBehave` | Client/Server  | Returns running status if the monster is currently hit, otherwise indicates failure. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up references to the monster entity and its state controller.

**Behavior**  
1. Assigns `self.monster` to the parent AI’s monster entity.  
2. Assigns `self.monsterState` to the parent AI’s monster state controller.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Checks if the monster is in the "hit" state.

**Behavior**  
1. If `self.monster.isHit` is `false`, returns `BehaviourTreeStatus.Failure`.  
2. If `self.monster.isHit` is `true`, returns `BehaviourTreeStatus.Running` to indicate the hit state is active.

</details>

---

### Related Types & Services
- **Monster** – Provides hit state tracking and entity behavior.  
- **MonsterStateController** – Handles state transitions for animations and gameplay effects.

## RootNode

**Purpose**  
Top-level behavior tree node that serves as the entry point for AI logic. Continuously runs to allow child nodes or sequences to execute.

---

### Properties

| Property | Type | Purpose |
|----------|------|---------|
| *(none)* | -    | Root node does not have intrinsic properties; it delegates behavior to child nodes or sequences. |

---

### Method Summary

| Method       | Execution Scope | Purpose |
|--------------|----------------|---------|
| `OnInit`     | Client/Server  | Initializes the root node and sets up references if needed. |
| `OnBehave`   | Client/Server  | Continuously runs, acting as the main driver for AI behavior execution. |

---

### Methods

<details>
<summary>OnInit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Prepares the root node for execution. Currently, it does not perform any specific initialization.

</details>

<details>
<summary>OnBehave</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Keeps the behavior tree active by continuously returning a running status.

**Behavior**  
- Always returns `BehaviourTreeStatus.Running` to ensure the AI logic remains active and child nodes can execute.

</details>

---

### Related Types & Services
- **BTNode** – Base class for behavior tree nodes providing lifecycle methods such as `OnInit` and `OnBehave`.


## Changelog
| Version | Changelog | Download |
|---------|-------------|-------------|
| V1_1 | Character Animations Demo Project Files Released! | [Download](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects/raw/refs/heads/main/Zakums_Curse/Zakums_Curse_V1_1.mod) |
