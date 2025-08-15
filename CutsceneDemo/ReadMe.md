# Cutscene Demo Documentation

## Youtube Livestream
[![How to Create a Cutscene Using the Dialog-Package](http://img.youtube.com/vi/uBnIJ5Am7Ds/0.jpg)](https://www.youtube.com/live/uBnIJ5Am7Ds?feature=shared "How to Create a Cutscene Using the Dialog-Package")

## Mod Package Installation Instructions
[Mod File Installation Guide](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects?tab=readme-ov-file#project-import-instructions)

## Project File
[Download Latest Project File](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CutsceneDemo/CutsceneDemo_V1.modpackage)

## Table of Contents
- [Overview](#overview)
- [Setup & Installation](#setup-and-installation)
- [Cutscene Sequence](#cutscenebehaviors)
- [Cutscene Sequence](#cutscenesequence)
- [Cutscene Sequence Item](#cutscenesequenceitem)
- [Cutscene System](#cutscenesystem)
- [Cutscene Trigger](#cutscenetrigger)
- [UI Interaction](#uiinteraction)
- [Custom Movement Component](#custommovementcomponent)
- [Emote Enum](#emoteenum)
- [Player Emote](#playeremote)
- [Changelog & Downloads](#changelog)
---

## Overview
 This documentation provides descriptions for the purpose of each script as well as a brief description of the properties and methods contained within the demo package. 

---

## Setup and Installation
* Download and install the dialogue package from the MapleStoryWorlds Packages page: [Dialogue Package](https://github.com/MSW-Git/MSWPackages?tab=readme-ov-file#download)
    * Reference the Mod Package installation instructions for package installation instructions : [Mod File Installation Guide](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects?tab=readme-ov-file#project-import-instructions)
    * Please ensure that you follow the read-me file's instructions located inside the dialogue package after import to ensure that the package functions as expected. 
* Update the MyDesk>Dialog>Core>UI>DialogueLogic script file to include callback logic. 
    * Add callback property 
        ```lua
        property any callback = nil
        ```
    * Add StartDialogWithCallback method
        ```lua
        	method void StartDialogWithCallback(string dialogId, any onComplete)
                self.callback = onComplete

                self:StartDialog(dialogId)
            end
        ```
    * Update the End Method to call the callback method once the dialogue is over. 
        ```lua
            @ExecSpace("ClientOnly")
            method void End()
                self.DialogPanel:Close()
                if self.callback then
                    self.callback()
                    self.callback = nil
                end

                self:ServerEnd()
            end
        ```
    * Add the ServerEnd() method
        ```lua
        	@ExecSpace("Server")
            method void ServerEnd()
                if self.callback then
                    self.callback()
                    self.callback = nil
                end
            end
        ```
* Update MyDesk > Dialog > Sample > SampleDialogDataSet DataSet with the following data: 
    | DialogId  | ScriptId | PortraitRUID                          | CharacterName   | CharacterScript                                                         | SelectionScripts                  | SelectionScriptIds |
    |-----------|----------|--------------------------------------|----------------|-------------------------------------------------------------------------|----------------------------------|------------------|
    | Tutorial  | T_1      | 20dd266f4d1c4ee1a2fbc1a25503cf9f     | Unknown Spirit | Don't touch that *@Nick*. That's an ancient power not intended for man to wield.. it could destroy you. | Umm...                           | T_2              |
    | Tutorial  | T_2      |                                      | @Nick          | ..Who are you..? And how do you know my name?                           |                                  | T_3              |
    | Tutorial  | T_3      | 20dd266f4d1c4ee1a2fbc1a25503cf9f     | Unknown Spirit | Never mind that. Just don't touch the sword..                           | Leave the sword alone. | *Poke* | T_4|T_5         |
    | Tutorial  | T_4      | 20dd266f4d1c4ee1a2fbc1a25503cf9f     | Unknown Spirit | Its for your own good.                                                  | Sure                             |                  |
    | Tutorial  | T_5      | 20dd266f4d1c4ee1a2fbc1a25503cf9f     | Unknown Spirit | *...*                                                                    |                                  |                  |

* Attach the PlayerEmote component to the Workspace>DefaultPlayer entity. 
* Attach the CustomMovementComponent component to the Workspace>DefaultPlayer entity. 
---

## CutsceneBehaviors

**Purpose**  
Provides a set of cutscene-related utility methods for controlling player movement, showing dialogue, triggering effects, and manipulating the camera during scripted events in MapleStory Worlds.

---

### Method Summary

| Method            | Execution Scope | Purpose |
|-------------------|-----------------|---------|
| `ShowBubble`      | Client          | Shows a temporary chat balloon above an entity. |
| `ShowEmote`       | Client/Server   | Plays an emote animation for an entity. |
| `DisablePlayer`   | Client          | Disables player controls for the triggering player. |
| `EnablePlayer`    | Client          | Re-enables player controls for the triggering player. |
| `MoveToPosition`  | Client          | Moves an entity to a given position. |
| `Focus`           | Client          | Focuses the camera on an entity for 5 seconds. |
| `ShowDialogue`    | Client          | Displays a dialogue window by ID. |
| `Shake`           | Client          | Shakes the camera for 2 seconds. |
| `Appear`          | Client          | Spawns an entity at a given position, optionally with an effect. |
| `Disappear`       | Client          | Removes an entity from the scene, optionally with an effect. |

### Methods

<details>
<summary>ShowBubble</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `ChatBalloonComponent`, `_TimerService`

**Purpose**  
Displays a temporary chat balloon message above an entity for 5 seconds, restoring the original chat settings afterward.

**Parameters**  
- `item` *(CutsceneSequenceItem)* – The cutscene item containing the entity and message.  
- `onComplete` *(any)* – Optional completion callback.

**Behavior**  
1. Spawns or retrieves the target entity.  
2. Configures its chat balloon with the new message.  
3. Restores original balloon settings after 5 seconds.

</details>

<details>
<summary>ShowEmote</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `CutsceneSequenceItem`, `_EmoteEnum`, `PlayerEmote`, `_TimerService`

**Purpose**  
Plays an emote animation for an entity during a cutscene.

**Parameters**  
- `item.params.emote` *(string)* – Name of the emote to play.  
- `item.params.duration` *(number)* – How long to play the emote.  
- `item.params.isBubble` *(boolean)* – Whether the emote should display as a bubble.

</details>

<details>
<summary>DisablePlayer</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `PlayerControllerComponent`

**Purpose**  
Disables player control for the triggering player.

</details>

<details>
<summary>EnablePlayer</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `PlayerControllerComponent`

**Purpose**  
Re-enables player control for the triggering player.

</details>

<details>
<summary>MoveToPosition</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `CustomMovementComponent`, `_TweenLogic`, `_TimerService`, `EaseType`

**Purpose**  
Moves an entity to a specified position using either:
- `CustomMovementComponent` if available.
- A tweened movement fallback if not.

**Parameters**  
- `item.params.position` *(Vector2)* – Target position.

**Behavior**  
- Handles flipping entity direction.  
- Plays movement tween or uses built-in movement.  
- Completes after movement finishes.

</details>

<details>
<summary>Focus</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `_CameraService`, `_TimerService`

**Purpose**  
Switches the camera focus to the target entity for 5 seconds.

</details>

<details>
<summary>ShowDialogue</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `_DialogLogic`

**Purpose**  
Displays a dialogue window with a specified dialog ID, then marks the cutscene step complete after the dialogue finishes.

**Parameters**  
- `item.params.dialogID` *(number)* – The ID of the dialogue to display.

</details>

<details>
<summary>Shake</summary>

**Execution Scope**: `Client`  
**Dependencies**: `_CameraService`, `_TimerService`

**Purpose**  
Shakes the current camera for 2 seconds.

**Behavior**  
- Intensity: `1.0`  
- Duration: `2` seconds.

</details>

<details>
<summary>Appear</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `_EffectService`, `_TimerService`

**Purpose**  
Places an entity at a given position and optionally plays a spawn effect.

**Parameters**  
- `item.params.position` *(Vector2/Vector3)* – Target spawn position.  
- `item.params.effectRUID` *(string, optional)* – Effect to play.

</details>

<details>
<summary>Disappear</summary>

**Execution Scope**: `Client`  
**Dependencies**: `CutsceneSequenceItem`, `_EffectService`, `_TimerService`

**Purpose**  
Removes an entity from the scene, optionally playing an effect. Cannot remove the local player.

**Behavior**  
- If target is `"LocalPlayer"`, logs a warning instead.  
- Plays an optional effect before destroying the entity.

</details>

---

### Related Types & Services
- **CutsceneSequenceItem** – Provides `GetOrSpawnEntity()`, `GetTriggeringPlayerEntity()`, `params`, and `isComplete`.  
- **_TimerService** – Schedules delayed functions.  
- **_CameraService** – Controls active camera and effects.  
- **_EffectService** – Plays visual effects.  
- **_DialogLogic** – Handles in-game dialogue.  
- **_TweenLogic** – Animates values over time.  
- **PlayerControllerComponent** – Enables/disables player input.  
- **CustomMovementComponent** – Moves entities to positions.  
- **ChatBalloonComponent** – Displays messages above entities.  
- **PlayerEmote** – Shows character animations and emotes.

## CutsceneSequence

**Purpose**  
Manages a sequence of cutscene items in MapleStory Worlds, allowing creators to add items and retrieve the total count of items in the sequence.

---

### Method Summary

| Method            | Execution Scope | Purpose |
|-------------------|----------------|---------|
| `AddSequence`     | Client/Server  | Adds a cutscene item to the sequence. |
| `Count`           | Client/Server  | Returns the number of items currently in the sequence. |

---

### Methods

<details>
<summary>AddSequence</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `CutsceneSequenceItem`

**Purpose**  
Adds a new `CutsceneSequenceItem` to the sequence's item list.

**Parameters**  
- `item` *(CutsceneSequenceItem)* – The cutscene item to add to the sequence.

**Behavior**  
1. Inserts the provided `CutsceneSequenceItem` into the `items` table.  
2. Updates the sequence count automatically.

</details>

<details>
<summary>Count</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: None

**Purpose**  
Returns the total number of cutscene items in the sequence.

**Returns**  
- *(number)* – The current count of items in the `items` table.

**Behavior**  
- Simply returns the length of the `items` table using `#self.items`.

</details>

---

### Related Types & Services
- **CutsceneSequenceItem** – Represents a single step or action in a cutscene sequence.

## CutsceneSequenceItem

**Purpose**  
Represents a single cutscene step in MapleStory Worlds, encapsulating a command, target, parameters, message, and the triggering player. Provides utility methods for entity management and command handling.

---

### Properties

| Property            | Type       | Purpose |
|--------------------|------------|---------|
| `command`           | string     | The command to execute for this cutscene item. |
| `target`            | string     | The target entity or model ID associated with the command. |
| `params`            | table      | Optional parameters for the command. |
| `message`           | string     | Optional message associated with the cutscene item. |
| `triggeredPlayer`   | string     | User ID of the player who triggered this cutscene item. |
| `isComplete`        | boolean    | Whether the cutscene item has finished executing. |
| `withNext`          | boolean    | Indicates if this cutscene item should automatically proceed to the next. |

---

### Method Summary

| Method                 | Execution Scope | Purpose |
|------------------------|----------------|---------|
| `Init`                 | Client/Server  | Initializes the cutscene item from a data row and sets the triggering player. |
| `ConvertToTable`       | Client/Server  | Converts a string of parameters into a Lua table. |
| `GetTriggeringPlayerEntity` | Client/Server | Returns the entity corresponding to the triggering player. |
| `GetOrSpawnEntity`     | Client/Server  | Retrieves or spawns the target entity for this cutscene item. |
| `DestroyEntity`        | Client/Server  | Destroys the target entity, except for the local player. |
| `InitWithCommand`      | Client/Server  | Initializes logic associated with the command. |
| `CleanupWithCommand`   | Client/Server  | Cleans up after command execution. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `UserDataRow`

**Purpose**  
Initializes the cutscene item from a data row and assigns the triggering player ID. Sets default values for `isComplete` and `withNext`.

**Parameters**  
- `row` *(any / UserDataRow)* – The data row containing cutscene item information.  
- `triggeredPlayer` *(string)* – The user ID of the player triggering this cutscene item.

**Behavior**  
1. Reads properties from the row: `Command`, `Params`, `Target`, `Message`, `WithNext`.  
2. Converts `Params` string to a table.  
3. Sets `triggeredPlayer` and `isComplete` flags.

</details>

<details>
<summary>ConvertToTable</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `_UtilLogic`

**Purpose**  
Converts a string of parameters into a Lua table for easier access during execution.

**Parameters**  
- `params` *(string)* – A string of parameters separated by `|`.

**Returns**  
- *(table)* – Parsed Lua table of parameters, or `nil` if conversion fails.

**Behavior**  
1. Replaces `|` with tab characters and `\n` with actual newlines.  
2. Uses `_UtilLogic:StringToTable` to convert the string to a table.  
3. Returns the table.

</details>

<details>
<summary>GetTriggeringPlayerEntity</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `_UserService`

**Purpose**  
Retrieves the player entity that triggered this cutscene item.

**Returns**  
- *(Entity)* – The entity corresponding to `triggeredPlayer`.

</details>

<details>
<summary>GetOrSpawnEntity</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `_UserService`, `_EntityService`, `_SpawnService`, `_UtilLogic`

**Purpose**  
Retrieves the target entity, or spawns it if it does not exist.

**Returns**  
- *(Entity)* – The target entity, or `nil` if invalid.

**Behavior**  
1. Returns nothing if `target` is nil or empty.  
2. If `target` is `"LocalPlayer"`, returns the triggering player entity.  
3. Otherwise, checks if an entity exists for the target model ID; spawns one if none exists.  
4. Returns the entity.

</details>

<details>
<summary>DestroyEntity</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `GetOrSpawnEntity`

**Purpose**  
Destroys the target entity unless it is the local player.

**Behavior**  
1. Logs a warning and returns if the target is `"LocalPlayer"`.  
2. Retrieves the target entity via `GetOrSpawnEntity`.  
3. Destroys the entity if valid.

</details>

<details>
<summary>InitWithCommand</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Prepares the cutscene item to execute its command logic.

**Behavior**  
- Placeholder for command-specific initialization.  
- Currently handles the `"Focus"` command (additional commands can be added).

</details>

<details>
<summary>CleanupWithCommand</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Cleans up any state or effects after executing a command.

**Behavior**  
- Placeholder for command-specific cleanup logic.

</details>

---

### Related Types & Services
- **_UtilLogic** – Provides helper functions for string and table conversions.  
- **_UserService** – Provides access to player entities by user ID.  
- **_EntityService** – Retrieves entities spawned in the world.  
- **_SpawnService** – Handles spawning of entities.  
- **UserDataRow** – Represents a row of cutscene data with helper `GetItem` method.  
- **Entity** – Represents a spawned entity in the world.
---

## CutsceneSystem

**Purpose**  
Manages the execution of cutscenes in MapleStory Worlds, handling cutscene sequences, controlling player visibility, showing chat bubbles, and broadcasting cutscene states to clients.

---

### Properties

| Property              | Type      | Purpose |
|----------------------|-----------|---------|
| `cutsceneBubble`      | string    | Effect ID used for the chat bubble during cutscenes. |
| `cutsceneIDs`         | table     | Table storing available cutscene identifiers. |
| `sequenceInProgress`  | boolean   | Indicates if a cutscene sequence is currently running. |

---

### Method Summary

| Method                     | Execution Scope | Purpose |
|----------------------------|----------------|---------|
| `OnBeginPlay`              | ClientOnly     | Initializes internal state for active bubbles. |
| `LoadData`                 | Client/Server  | Loads a cutscene sequence from data and creates `CutsceneSequenceItem`s. |
| `BeginCutscene`            | Client         | Starts executing a cutscene sequence for a specific player. |
| `BroadcastShowClient`      | Server         | Sends visibility updates to all clients except the triggering player. |
| `ShowOtherClients`         | Client         | Toggles the visibility of other players during a cutscene. |
| `ShowBubble`               | Client         | Shows or hides a cutscene chat bubble above a player. |
| `HandleKeyDown`            | Event          | Handles input events to trigger cutscenes. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes the internal `_T.activeBubbles` table to track active chat bubbles.

**Behavior**  
- Sets `_T.activeBubbles` to an empty table when the client begins play.

</details>

<details>
<summary>LoadData</summary>

**Execution Scope**: `Client/Server`  
**Dependencies**: `_DataService`, `CutsceneSequence`, `CutsceneSequenceItem`

**Purpose**  
Loads a cutscene sequence from a data source and initializes all items with the triggering player.

**Parameters**  
- `data` *(string)* – Name of the cutscene data table.  
- `triggeredPlayer` *(string)* – User ID of the player triggering the cutscene.

**Returns**  
- *(CutsceneSequence)* – A populated cutscene sequence, or `nil` if data is invalid.

**Behavior**  
1. Retrieves data table using `_DataService:GetTable`.  
2. Iterates through rows to create and initialize `CutsceneSequenceItem`s.  
3. Adds each item to a `CutsceneSequence` object.  
4. Returns the populated sequence.

</details>

<details>
<summary>BeginCutscene</summary>

**Execution Scope**: `Client`  
**Dependencies**: `_TimerService`, `_CutsceneBehaviors`, `ShowOtherClients`, `BroadcastShowClient`

**Purpose**  
Executes a cutscene sequence for a player, handling sequence timing, progression, and completion events.

**Parameters**  
- `cutsceneName` *(string)* – Name of the cutscene to play.  
- `triggeredPlayer` *(string)* – User ID of the player triggering the cutscene.

**Behavior**  
1. Loads sequence data using `LoadData`.  
2. Prevents starting if a sequence is already in progress.  
3. Iterates through each `CutsceneSequenceItem`, executing its command.  
4. Uses `_TimerService` to poll for sequence completion or auto-proceed (`withNext`).  
5. Manages player visibility and bubble effects via `ShowOtherClients` and `BroadcastShowClient`.  
6. Marks `sequenceInProgress` as `false` when completed.

</details>

<details>
<summary>BroadcastShowClient</summary>

**Execution Scope**: `Server`  
**Dependencies**: `_UserService`, `ShowBubble`

**Purpose**  
Updates other clients’ visibility of a triggering player during a cutscene.

**Parameters**  
- `triggeredPlayer` *(string)* – User ID of the triggering player.  
- `isShow` *(boolean)* – Whether to show or hide the cutscene bubble for others.

**Behavior**  
1. Retrieves the triggering player entity.  
2. Iterates through all players on the same map except the trigger player.  
3. Calls `ShowBubble` to show/hide bubbles for each player.

</details>

<details>
<summary>ShowOtherClients</summary>

**Execution Scope**: `Client`  
**Dependencies**: `_UserService`

**Purpose**  
Enables or disables visibility of other players’ avatars and nametags during a cutscene.

**Parameters**  
- `triggeredPlayer` *(string)* – User ID of the triggering player.  
- `activeState` *(boolean)* – Whether other players should be visible.

**Behavior**  
- Iterates through all players on the same map except the triggering player.  
- Toggles `AvatarRendererComponent.Enable` and `NameTagComponent.Enable` according to `activeState`.

</details>

<details>
<summary>ShowBubble</summary>

**Execution Scope**: `Client`  
**Dependencies**: `_UserService`, `_EffectService`

**Purpose**  
Shows or hides a chat bubble effect above the triggering player.

**Parameters**  
- `triggeredPlayer` *(string)* – User ID of the player.  
- `isShow` *(boolean)* – Whether to show or remove the bubble.

**Behavior**  
1. Retrieves the triggering player entity.  
2. If `isShow` is true, plays the effect and stores it in `_T.activeBubbles`.  
3. If `isShow` is false, removes the effect from `_EffectService` if active.

</details>

<details>
<summary>HandleKeyDown</summary>

**Execution Scope**: `Event`  
**Dependencies**: `BeginCutscene`, `_UserService`

**Purpose**  
Listens for key input and triggers a cutscene when a specific key is pressed.

**Parameters**  
- `event` *(KeyDownEvent)* – Input event containing the pressed key.

**Behavior**  
- If key `F` is pressed, calls `BeginCutscene` with `"DialogueSequence_1"` for the local player.

</details>

---

### Related Types & Services
- **_DataService** – Provides access to cutscene data tables.  
- **_TimerService** – Schedules and manages timed events.  
- **_UserService** – Provides player entity lookup.  
- **_EffectService** – Plays and removes visual effects.  
- **_CutsceneBehaviors** – Maps cutscene commands to behavior functions.  
- **CutsceneSequence** – Contains multiple `CutsceneSequenceItem`s.  
- **CutsceneSequenceItem** – Represents a single cutscene step.  
- **Entity** – Represents a player or spawned entity in the world.  
- **KeyDownEvent** – Represents an input key event from the client.


## CutsceneTrigger

**Purpose**  
Handles triggering cutscenes when a player enters a specified area, manages UI prompts, and responds to input events.

---

### Properties

| Property        | Type      | Purpose |
|-----------------|-----------|---------|
| `canTrigger`     | boolean   | Indicates whether the cutscene can currently be triggered. |
| `cutsceneName`   | string    | Name of the cutscene to trigger. Defaults to `"DialogueSequence_1"`. |

---

### Method Summary

| Method              | Execution Scope | Purpose |
|--------------------|----------------|---------|
| `HandleTriggeEnter` | Client         | Handles logic when a player enters the trigger area. |
| `HandleTriggerExit` | Client         | Handles logic when a player exits the trigger area. |
| `TriggerCutscene`   | Client         | Starts the specified cutscene for the local player. |
| `OnTriggerEnter`    | Client/Server  | Event handler for trigger enter events. |
| `OnTriggerExit`     | Client/Server  | Event handler for trigger exit events. |
| `OnKeyDown`         | Client/Server  | Event handler for key input to trigger cutscene. |

---


### Methods

<details>
<summary>HandleTriggeEnter</summary>

**Execution Scope**: `Client`  

**Purpose**  
Handles logic when a player enters the trigger area, displays a UI prompt, and calculates sprite offset.

**Behavior**  
1. Sets `canTrigger` to `true`.  
2. Retrieves the entity’s `SpriteRendererComponent`.  
3. Uses `_SpriteUtils:GetSpriteAnimSize` to calculate offset.  
4. Displays the UI interaction prompt at the computed position via `_UIInteraction:ShowAtPosition`.

</details>

<details>
<summary>HandleTriggerExit</summary>

**Execution Scope**: `Client`  

**Purpose**  
Handles logic when a player exits the trigger area, hiding the UI prompt.

**Behavior**  
- Hides the UI prompt via `_UIInteraction:Hide`.  
- Sets `canTrigger` to `false`.

</details>

<details>
<summary>TriggerCutscene</summary>

**Execution Scope**: `Client`  

**Purpose**  
Triggers the cutscene specified by `cutsceneName` for the local player.

**Behavior**  
1. Retrieves the local player’s user ID.  
2. Calls `_CutsceneSystem:BeginCutscene` with `cutsceneName` and local player ID.

</details>

<details>
<summary>OnTriggerEnter</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Event handler that executes when a trigger area is entered.

**Behavior**  
- Calls `HandleTriggeEnter`.

</details>

<details>
<summary>OnTriggerExit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Event handler that executes when a trigger area is exited.

**Behavior**  
- Calls `HandleTriggerExit`.

</details>

<details>
<summary>OnKeyDown</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Event handler for key input to trigger a cutscene if the player can trigger it.

**Parameters**  
- `event` *(KeyDownEvent)* – Input event containing the pressed key.

**Behavior**  
- If `KeyboardKey.F` is pressed and `canTrigger` is true, calls `TriggerCutscene`.

</details>

---

### Related Types & Services
- **_SpriteUtils** – Provides sprite animation sizing utilities.  
- **_UIInteraction** – Handles UI prompts on the client.  
- **_UserService** – Provides access to the local player and user entities.  
- **_CutsceneSystem** – Executes cutscene sequences.  
- **SpriteRendererComponent** – Component holding sprite information.  
- **TransformComponent** – Provides entity position information.  
- **TriggerEnterEvent / TriggerLeaveEvent** – Trigger area events.  
- **KeyDownEvent** – Input event for key presses.  
- **KeyboardKey** – Enumeration of keyboard keys.


## UIInteraction

**Purpose**  
Manages a UI element that prompts the player to interact with objects or triggers in the world.

---

### Properties

| Property             | Type                     | Purpose |
|----------------------|-------------------------|---------|
| `interactionUI`       | UITransformComponent     | The UI element to show or hide for interaction prompts. Defaults to `"0b55ce67-01cb-41e9-829d-eca916f7eb20"`. |

---

### Method Summary

| Method            | Execution Scope | Purpose |
|------------------|----------------|---------|
| `ShowAtPosition`  | Client/Server  | Shows the interaction UI at the specified world position. |
| `Hide`            | Client/Server  | Hides the interaction UI. |

---

### Methods

<details>
<summary>ShowAtPosition</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Displays the interaction UI at a specific world position.

**Parameters**  
- `targetPos` *(Vector3)* – The position where the UI should appear.

**Behavior**  
1. Sets `interactionUI.WorldPosition` to the target position.  
2. Enables the UI entity so it becomes visible.

</details>

<details>
<summary>Hide</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Hides the interaction UI.

**Behavior**  
- Sets `interactionUI.Entity` to disabled, making it invisible.

</details>

---

### Related Types & Services
- **UITransformComponent** – Represents a UI element’s position and enable/disable state in the world.


## CustomMovementComponent

**Purpose**  
Extends `MovementComponent` to provide automatic movement functionality, including moving to a target position along a straight path.

---

### Properties

| Property       | Type      | Purpose |
|----------------|-----------|---------|
| `isAutoMove`    | boolean  | Indicates whether the entity should automatically move. Synchronized across clients and server. |

---

### Method Summary

| Method            | Execution Scope | Purpose |
|------------------|----------------|---------|
| `OnBeginPlay`     | ClientOnly     | Initializes the custom movement component and adds the `AUTOWALK` state to the entity. |
| `MoveToPosition`  | ClientOnly     | Moves the entity to a target X-position using the `AUTOWALK` state. |
| `OnUpdate`        | ClientOnly     | Called each frame; can be used to update movement logic. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes the component and sets up the `AUTOWALK` state for the entity’s state machine.

**Behavior**  
1. Retrieves the entity’s `StateComponent`.  
2. Adds the `"AUTOWALK"` state using the `Autowalk` class.  
3. Stores the state component reference in `_T.stateComp`.

</details>

<details>
<summary>MoveToPosition</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Moves the entity to a specified X-position along a straight path. Automatically changes the entity state to `"AUTOWALK"` during movement and back to `"IDLE"` when the destination is reached.

**Parameters**  
- `targetPos` *(Vector2)* – The X/Y position to move the entity toward. Only X-axis movement is considered.  
- `onComplete` *(any)* – Callback executed once the movement finishes.

**Behavior**  
1. Computes the direction to the target.  
2. Changes the entity’s state to `"AUTOWALK"`.  
3. Sets a repeating timer (`1/30` sec) to move toward the target.  
4. Stops movement when the entity is within 0.5 units of the target, clears the timer, changes state to `"IDLE"`, and calls `onComplete`.

</details>

<details>
<summary>OnUpdate</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Frame update method for custom movement logic.

**Behavior**  
- Currently a placeholder; can be used to update entity movement each frame.

</details>

---

### Related Types & Services
- **MovementComponent** – Base component for entity movement.  
- **StateComponent** – Manages entity states such as `"AUTOWALK"` and `"IDLE"`.  
- **_TimerService** – Handles repeated timers.  
- **_UtilLogic** – Provides server time utilities like `ServerElapsedSeconds`.  
- **TransformComponent** – Provides the entity’s world position.  
- **PlayerControllerComponent** – Provides control over the entity’s look direction.  
- **Autowalk** – State used for automatic walking animation.


## EmoteEnum

**Purpose**  
Provides a set of predefined emote constants for use in cutscenes and gameplay, along with utility methods for validation and string-to-index conversion.

---

### Properties

| Property   | Type    | Purpose |
|------------|---------|---------|
| `HAPPY`    | integer | Represents the "Happy" emote. Value: `1`. |
| `QUESTION` | integer | Represents the "Question" emote. Value: `2`. |
| `ANGRY`    | integer | Represents the "Angry" emote. Value: `3`. |
| `LOVE`     | integer | Represents the "Love" emote. Value: `4`. |
| `CRY`      | integer | Represents the "Cry" emote. Value: `5`. |
| `SHOCK`    | integer | Represents the "Shock" emote. Value: `6`. |
| `MAX`      | integer | The maximum emote value + 1. Value: `7`. |

---

### Method Summary

| Method        | Execution Scope | Purpose |
|---------------|----------------|---------|
| `OnBeginPlay` | Client/Server  | Initializes an internal string-to-index table for quick lookups. |
| `Validate`    | Client/Server  | Checks if a given emote index is valid. |
| `FromString`  | Client/Server  | Converts a string representation of an emote to its corresponding index. Returns `-1` if not found. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes a mapping of emote names (strings) to their numeric values for runtime use.

**Behavior**  
1. Creates a local `stringTable` mapping names to their property values.  
2. Stores the table in `_T.stringTable` for later lookup.

</details>

<details>
<summary>Validate</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Validates whether a given integer index is a valid emote.

**Parameters**  
- `index` *(integer)* – The emote index to validate.

**Returns**  
- `boolean` – `true` if `index > 0` and `index < MAX`; otherwise `false`.

</details>

<details>
<summary>FromString</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Converts a string emote name into its corresponding numeric index.

**Parameters**  
- `value` *(string)* – The name of the emote (case-insensitive).

**Returns**  
- `integer` – The index of the emote. Returns `-1` if the string does not match any emote.

**Behavior**  
1. Looks up the uppercase version of the string in `_T.stringTable`.  
2. Returns the corresponding index if found, otherwise returns `-1`.

</details>

---

### Related Types & Services
- **Logic** – Base class for scripts providing runtime logic.  
- **_T** – Temporary storage table used internally by the script.


## PlayerEmote

**Purpose**  
Provides functionality to display emotes for a player entity, either as floating bubble effects or facial animations, using predefined emotes from `_EmoteEnum`.

---

### Properties

| Property        | Type    | Purpose |
|-----------------|---------|---------|
| `_T.bubbleEmotes` | table  | Maps emote indices to bubble effect RUIDs. |
| `_T.faceEmotes`   | table  | Maps emote indices to facial emotional types. |

---

### Method Summary

| Method        | Execution Scope | Purpose |
|---------------|----------------|---------|
| `OnInitialize` | Client/Server  | Initializes internal tables mapping emote indices to bubble effects and face emotions. |
| `ShowEmote`    | Client/Server  | Displays an emote either as a bubble or facial animation based on parameters. |
| `BubbleEmote`  | Client/Server  | Plays a bubble emote attached to the entity for a specified duration. |
| `FaceEmote`    | Client/Server  | Plays a facial animation on the entity for a specified duration. |

---

### Methods

<details>
<summary>OnInitialize</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes internal mappings for bubble and facial emotes.

**Behavior**  
1. Defines `bubbleEmotes`, mapping `_EmoteEnum` indices to effect RUIDs.  
2. Defines `faceEmotes`, mapping `_EmoteEnum` indices to `EmotionalType` values.  
3. Stores both tables in `_T.bubbleEmotes` and `_T.faceEmotes`.

</details>

<details>
<summary>ShowEmote</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Shows an emote for the entity, either as a floating bubble or a facial animation.

**Parameters**  
- `emote` *(integer)* – Emote index from `_EmoteEnum`.  
- `duration` *(number)* – Duration in seconds to show the emote.  
- `isBubble` *(boolean)* – Whether to show as a bubble (`true`) or facial animation (`false`).

**Behavior**  
- Validates the emote index using `_EmoteEnum:Validate()`.  
- Calls `BubbleEmote` if `isBubble` is true, otherwise calls `FaceEmote`.

</details>

<details>
<summary>BubbleEmote</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Displays a floating bubble emote above the entity for a specified duration.

**Parameters**  
- `emote` *(integer)* – Emote index from `_EmoteEnum`.  
- `duration` *(number)* – Duration to display the bubble.

**Behavior**  
1. Retrieves the bubble effect RUID from `_T.bubbleEmotes`.  
2. Plays the effect attached to the entity.  
3. Schedules a timer to remove the effect after `duration` seconds.

</details>

<details>
<summary>FaceEmote</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Plays a facial emotion on the entity’s avatar renderer for a given duration.

**Parameters**  
- `emote` *(integer)* – Emote index from `_EmoteEnum`.  
- `duration` *(number)* – Duration to display the facial emotion.

**Behavior**  
1. Retrieves the entity’s `AvatarRendererComponent`.  
2. Looks up the corresponding `EmotionalType` in `_T.faceEmotes`.  
3. Calls `PlayEmotion` on the avatar renderer for the specified duration.

</details>

---

### Related Types & Services
- **Component** – Base class for Unity-style entity components.  
- **_T** – Temporary table for storing runtime state.  
- **_EffectService** – Plays and removes visual effects.  
- **_TimerService** – Schedules delayed and repeated callbacks.  
- **_EmoteEnum** – Provides emote indices and validation.  
- **AvatarRendererComponent** – Handles playing facial emotions on the entity.
