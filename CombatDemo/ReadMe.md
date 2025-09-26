# Player Stats & Skills System Documentation

## Youtube Livestream
[![How to Make a Character in MapleStory Worlds](http://img.youtube.com/vi/3t_HcSKQ7M0/0.jpg)](https://www.youtube.com/live/3t_HcSKQ7M0?feature=Client/Server "How to Make a Character in MapleStory Worlds")
[![How to Create a Character PT 2 w/ Ursus](http://img.youtube.com/vi/98-aFbh2pOg/0.jpg)](https://www.youtube.com/live/98-aFbh2pOg?feature=Client/Server "How to Create a Character PT 2 w/ Ursus")

## Mod Package Installation Instructions
[Mod File Installation Guide](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects?tab=readme-ov-file#project-import-instructions)

## Project File
[Download Latest Project File](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CombatDemo/CombatDemo.modpackage)


## Table of Contents
- [Overview](#overview)
- [Setup & Installation](#setup-and-installation)
- [Healthbar Component](#healthbarcomponent)
- [Healthbar Update](#healthbarupdate)
- [Monster](#monster)
- [Monster Attack](#monsterattack)
- [OnSkillEquippedEvent](#onskillequippedevent)
- [PlayerSkillComponent](#playerskillcomponent)
- [Skill](#skill)
- [SkillBuilder](#skillbuilder)
- [SkillInventoryUdatedEvent](#skillinventoryupdatedevent)
- [CharacterStats](#characterstats)
- [PlayerStatsEnum](#playerstatsenum)
- [PlayerInfoComponent](#playerinfocomponent)
- [PlayerStatsUpdate](#playerstatsupdate)
- [PlayerAnimationComponent](#playeranimationcomponent)
- [SkillInfoBuilder](#skillinfobuilder)
- [SkillFactory](#skillfactory)
- [SkillsEnum](#skillsenum)
- [MonsterStats](#monsterstats)
- [PlayerStats](#playerstats)
- [PlayerAttack](#playerattack)
- [PlayerHit](#playerhit)
- [SkillUIComponent](#skilluicomponent)
- [SkillSelected](#skillselected)
- [SkillUsedEvent](#skillusedevent)
- [SkillWindowComponent](#skillwindowcomponent)
- [StatWindowUIComponent](#statwindowuicomponent)
- [PlayerStatIncreased](#playerstatincreased)
- [StatsWindowUIItem](#statswindowuiitem)
- [UIMyInfoSimple](#uimyinfosimple)
- [MonsterSpawn](#monsterspawn)
- [DebugLogic](#debuglogic)
- [Changelog & Downloads](#changelog)
---


## Overview
This documentation covers the Player Stats, Skills, Animation, and UI systems used in the game. Each section describes the core components, classes, and their interactions to assist developers in understanding and extending the functionality.

---
## Setup and Installation
* Update the uiPath variable under Sample>DebugLogic Logic script file with the correct path for your world by right clicking on the desired map under Heirarchy>UI>["YourUIGroupName"] and selecting "Copy Entity Path" then pasting it to the uiPath variable. 
* Attach the following components to the Default Player Object found under the Workspace root folder. 
  * PlayerSkillComponent
  * PlayerInfoComponent
  * PlayerAnimationComponent
  * PlayerAttack
  * PlayerHit
    * Set the "CollisionGroup" property to the "Player" group. 
  * ColliderVisualizer (Optional)
    * If using ColliderVisualizer, enable the "VisializeAttack" property under the PlayerAttack component. 
* Place the "Sample>Monster>Model>Model_MonsterSpawner" model entity into the Heirarchy by right clicking on the model and clicking "PlaceToHeirarchy". This will periodically spawn in monsters into the world. 
* Locate monster model "Combat>Sample>Monster>Models>Model_Monster-35" and check if the collision group under the hitcomponent shows an error. If it does, this is because the "Monster" collision layer does not yet exist.
    * Create the "Monster" collision group by selecting the arrow next to the Collision Group and creating a new "Monster" collision group. 
    * Assign the monsters collision group to the newly created collision group. 
* The DebugLogic script file will spawn the necessary UI entities for the demo. It also contains debug input events that binds F1 to open the stat window, and F2 to trigger the player to level up. 
---

## HealthbarComponent

**Purpose**  
Displays and manages a health bar above an entity, synchronizing its visual state with the entity’s health both on the client and server.

---

### Properties

| Property   | Type    | Default Value                                                   | Purpose |
|------------|---------|-----------------------------------------------------------------|---------|
| `healthBar`| string  | `"model://e1b5c661-4d26-4f88-9002-fa1fb0c8c662"`                  | Resource ID for the health bar model. |
| `offset`   | Vector2 | `(0.5, 0.7)`                                                    | Position offset for placing the health bar relative to the entity. |

---

### Method Summary

| Method              | Execution Scope | Purpose |
|---------------------|-----------------|---------|
| `Init(maxHealth)`   | Client          | Spawns and positions the health bar, initializes visual references, and triggers server setup. |
| `ServerInit()`      | Server          | Connects the server-side health update event to the update handler. |
| `Set(health)`       | Client          | Updates the fill amount of the health bar based on the current health percentage. |
| `OnUpdate(delta)`   | ClientOnly      | Maintains correct scaling of the health bar to match the entity's scale. |
| `OnUpdateHealthBar(evt)` | Server    | Receives server-side health update events and updates the health bar display. |

---

### Methods

<details>
<summary>Init(maxHealth)</summary>

**Execution Scope**: `Client`  

**Purpose**  
- Spawns a health bar above the entity, adjusting position based on the entity’s sprite size if available.  
- Stores references to the health bar object, the "Fill" component, and the maximum health value.  
- Invokes `ServerInit()` to prepare server-side event handling.

**Behavior**  
1. Checks if the entity has a `SpriteRendererComponent`.  
2. Calculates position offset based on sprite size or the provided `offset` property.  
3. Spawns the health bar model and attaches it to the entity.  
4. Stores references for later updates.  
5. Calls `ServerInit()`.

</details>

<details>
<summary>ServerInit()</summary>

**Execution Scope**: `Server`  

**Purpose**  
Binds the `HealthBarUpdate` event to the `OnUpdateHealthBar` method so that health changes from the server are reflected on clients.

</details>

<details>
<summary>Set(health)</summary>

**Execution Scope**: `Client`  

**Purpose**  
Adjusts the fill amount of the health bar based on the current health percentage.

**Parameters**  
- `health` *(number)*: Current health value to display.

**Behavior**  
- Calculates `health / maxHealth` to determine fill percentage.  
- Updates the `Fill` sprite’s `FillAmount`.

</details>

<details>
<summary>OnUpdate(delta)</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Ensures the health bar’s scale matches the entity’s scale dynamically.

**Behavior**  
- Skips update if the health bar is not yet initialized.  
- Matches the health bar’s X-scale to the entity’s X-scale.

</details>

<details>
<summary>OnUpdateHealthBar(evt)</summary>

**Execution Scope**: `Server`  

**Purpose**  
Handles `HealthBarUpdate` events sent from the server, triggering a visual update on the client.

**Parameters**  
- `evt.health` *(number)*: Updated health value.

**Behavior**  
- Calls `Set(evt.health)` to refresh the health bar.

</details>

---

### Related Types & Services
- **Component** – Base class for all attachable game components.
- **_SpawnService** – Handles spawning of game objects by model ID.
- **SpriteGUIRendererComponent** – Renders and adjusts GUI sprites such as health bar fill elements.
- **HealthBarUpdate** – Event carrying updated health values from the server.

---

## HealthBarUpdate

**Purpose**  
Represents an event that carries updated health values from the server to clients for display purposes (e.g., updating a health bar UI).

---

### Properties

| Property | Type   | Default Value | Purpose |
|----------|--------|---------------|---------|
| `health` | number | `0`           | The current health value to be displayed on the health bar. |

---

### Usage  
This event is typically fired by the server when an entity's health changes, so that connected clients can update their visual health bar display accordingly.  

**Example Flow**  
1. Server detects an entity’s health has changed.  
2. A `HealthBarUpdate` event is dispatched with the new `health` value.  
3. Client receives the event and uses it to adjust the health bar fill percentage.

---

### Related Scripts
- **HealthbarComponent** – Listens for `HealthBarUpdate` events to visually update the health bar.

## Monster

**Purpose**  
Represents a monster entity in MapleStory Worlds, managing health, death state, combat interactions, and health bar updates. Handles receiving damage, triggering death logic, and synchronizing health with the client.

---

### Properties

| Property      | Type    | Purpose |
|---------------|---------|---------|
| `MaxHp`       | number  | Maximum health value of the monster. |
| `Hp`          | number  | Current health value of the monster. |
| `IsDead`      | boolean | Indicates whether the monster is dead. |
| `DestroyDelay`| number  | Time in seconds before the monster is hidden/disabled after death. |
| `Power`       | number  | Attack power of the monster. |
| `dropModelID` | string  | Model ID for the item dropped upon monster defeat. |
| `level`       | integer | Monster's level. |
| `stats`       | any     | Reference to the monster's stats object, initialized at runtime. |

---

### Method Summary

| Method               | Execution Scope | Purpose |
|----------------------|----------------|---------|
| `OnBeginPlay()`      | Server/Client  | Initializes monster stats, health, and the health bar when the entity spawns. |
| `Dead()`             | ServerOnly     | Handles monster death, changes state, and schedules entity hide/disable after `DestroyDelay`. |
| `Respawn(position)`  | Server/Client  | Resets health and repositions the monster. |
| `UpdateHealth()`     | Client         | Sends a `HealthBarUpdate` event to synchronize the health bar display on the client. |
| `OnInitialize()`     | ServerOnly     | Reserved for server-side initialization logic. |
| `HandleHitEvent()`   | ServerOnly     | Processes incoming hit events, applies damage, plays effects/sounds, updates health bar, and triggers death if health reaches zero. |

---

### Events

- **Receives**:  
  - `HitEvent` (via `HandleHitEvent`) — Contains damage, hit effects, and sound information when the monster is attacked.  

- **Sends**:  
  - `HealthBarUpdate` — Dispatched to update the monster’s health bar on clients.

---

### Methods

<details>
<summary>OnBeginPlay()</summary>

**Execution Scope**: Server/Client  

**Purpose**  
Initializes monster stats, sets `MaxHp` and `Hp`, and initializes the health bar component if present.

**Behavior**  
1. Creates and initializes a `MonsterStats` object.  
2. Sets `MaxHp` and `Hp` based on stats.  
3. If `HealthbarComponent` exists, stores a reference and calls `Init(MaxHp)` on it.

</details>

<details>
<summary>Dead()</summary>

**Execution Scope**: ServerOnly  

**Purpose**  
Handles death state logic, changes the entity state to `"DEAD"`, and schedules the entity to be hidden and disabled after `DestroyDelay`.

**Behavior**  
1. Sets `IsDead = true`.  
2. Changes the entity's state to `"DEAD"` if a `StateComponent` exists.  
3. Schedules a timer to call `SetVisible(false)` and `SetEnable(false)` after `DestroyDelay`.

</details>

<details>
<summary>Respawn(position)</summary>

**Execution Scope**: Server/Client  

**Purpose**  
Resets monster health and moves the monster to the given position.

**Parameters**  
- `position` *(Vector3)* – The world coordinates to respawn the monster.

**Behavior**  
1. Sets `Hp` to `MaxHp`.  
2. Calls `UpdateHealth()` to update the health bar.  
3. Moves the entity to `position`.

</details>

<details>
<summary>UpdateHealth()</summary>

**Execution Scope**: Client  

**Purpose**  
Synchronizes the monster’s current health with the client-side health bar.

**Behavior**  
1. If the health bar reference exists, sends a `HealthBarUpdate` event with the current `Hp`.

</details>

<details>
<summary>OnInitialize()</summary>

**Execution Scope**: ServerOnly  

**Purpose**  
Reserved for server-side initialization logic. Currently empty.

</details>

<details>
<summary>HandleHitEvent(event)</summary>

**Execution Scope**: ServerOnly  

**Purpose**  
Processes incoming hit events, applies damage, plays effects and sounds, updates the health bar, and triggers death if `Hp` reaches zero.

**Parameters**  
- `event` *(HitEvent)* – Contains attack details including damage, effects, sounds, and extra metadata.

**Behavior**  
1. Subtracts `TotalDamage` from `Hp`.  
2. If `Extra` is valid, decodes it and plays any hit effects or sounds.  
3. Updates the health bar by sending a `HealthBarUpdate` event.  
4. If `Hp <= 0` and the monster was alive, calls `Dead()`.

</details>

---

### Related Components & Services
- **HealthbarComponent** — Manages the visual health bar above the monster.  
- **HealthBarUpdate** — Event used to update the client-side health bar.  
- **MonsterStats** — Utility for initializing monster attributes.  
- **StateComponent** — Manages monster states, such as `"DEAD"`.  
- **_EffectService** — Plays visual effects.  
- **_SoundService** — Plays sound effects at a specific position.  
- **_TimerService** — Schedules delayed actions for death and other timed behaviors.

## MonsterAttack

**Purpose**  
Handles a monster's attack logic in MapleStory Worlds, including attack timing, area detection, target validation, and damage calculation. This component periodically checks for nearby players and applies damage using the monster’s power.

---

### Properties

| Property        | Type      | Purpose |
|-----------------|-----------|---------|
| `AttackInterval`| number    | Time interval in seconds between attack checks. |
| `Shape`         | any       | Collision shape used to detect targets in the attack area. |
| `SpriteSize`    | Vector2   | Size of the monster sprite for calculating attack area. |
| `PositionOffset`| Vector2   | Offset of the attack area relative to the monster's sprite. |

---

### Method Summary

| Method                 | Execution Scope | Purpose |
|------------------------|----------------|---------|
| `OnBeginPlay()`        | ServerOnly     | Initializes attack area, sprite size, position offset, and starts a repeating attack timer. |
| `AttackNear()`         | Server/Client  | Updates the attack shape position based on monster transform and triggers attack on nearby players. |
| `IsAttackTarget()`     | Server/Client  | Determines if a given entity is a valid attack target. |
| `CalcDamage()`         | Server/Client  | Calculates the damage dealt to a target entity based on monster power and target stats. |

---

### Methods

<details>
<summary>OnBeginPlay()</summary>

**Execution Scope**: ServerOnly  

**Purpose**  
Initializes attack parameters and sets up a repeating timer for performing nearby attacks.

**Behavior**  
1. Retrieves the monster component from the entity.  
2. Initializes `Shape` as a `BoxShape` and calculates `SpriteSize` and `PositionOffset` using `_SpriteUtils`.  
3. Sets a repeating timer via `_TimerService` that calls `AttackNear()` at intervals defined by `AttackInterval`, only if the monster is alive.

</details>

<details>
<summary>AttackNear()</summary>

**Execution Scope**: Server/Client  

**Purpose**  
Updates the attack area based on the monster's transform and performs an attack against players within range.

**Behavior**  
1. Retrieves the monster's world position and scale from `TransformComponent`.  
2. Calculates the attack area size and offset, applying rotation.  
3. Updates the `Shape` size, position, and angle.  
4. Calls `AttackFast()` with the updated shape to apply damage to entities in `CollisionGroups.Player`.

</details>

<details>
<summary>IsAttackTarget(defender, attackInfo)</summary>

**Execution Scope**: Server/Client  

**Purpose**  
Determines whether a specific entity can be attacked by the monster.

**Parameters**  
- `defender` *(Entity)* – Potential target entity.  
- `attackInfo` *(string)* – Optional identifier for the attack.

**Behavior**  
1. Verifies the defender has a valid `PlayerComponent`.  
2. Calls the base class `IsAttackTarget()` method to perform additional checks.

**Returns**  
- `true` if the entity is a valid target; otherwise `false`.

</details>

<details>
<summary>CalcDamage(attacker, defender, attackInfo)</summary>

**Execution Scope**: Server/Client  

**Purpose**  
Calculates the damage dealt from the monster to a target entity.

**Parameters**  
- `attacker` *(Entity)* – The monster performing the attack.  
- `defender` *(Entity)* – The entity being attacked.  
- `attackInfo` *(string)* – Optional attack identifier.

**Behavior**  
1. Retrieves monster power from `Monster.Power`.  
2. Calculates the final damage using the target's `PlayerInfoComponent.stats:GetDamage()` method.  
3. Returns the calculated damage if positive; otherwise returns `-1`.

**Returns**  
- *(integer)* – Damage value applied to the defender.

</details>

---

### Related Components & Services
- **AttackComponent** — Base class providing attack utilities such as `AttackFast()` and target validation.  
- **Monster** — Provides stats, power, and state information used during attacks.  
- **_SpriteUtils** — Used to calculate sprite size and offset for attack positioning.  
- **_TimerService** — Handles repeating attack logic.  
- **CollisionGroups.Player** — Specifies the target group for monster attacks.

## OnSkillEquippedEvent

**Purpose**  
Event triggered when a skill is equipped or switched by a player. Contains information about the currently active skill, the previously inactive skill, and the skill highlighted in the UI.

---

### Properties

| Property               | Type    | Purpose |
|------------------------|---------|---------|
| `activeSkillRUID`      | string  | Resource UID of the skill that is currently active. |
| `inactiveSkillRUID`    | string  | Resource UID of the skill that was previously inactive. |
| `highlightSkillRUID`   | string  | Resource UID of the skill currently highlighted in the UI. |
| `index`                | integer | Index position of the skill in the skill bar or skill list. |

---

### Related Types & Services
- **EventType** — Base class for all custom events in MapleStory Worlds.

## PlayerSkillComponent

**Purpose**  
Manages a player’s skill inventory, equipped skills, and skill usage. Handles loading skills from data, equipping skills, triggering skill actions, and synchronizing skill UI events with clients.

---

### Properties

| Property          | Type    | Purpose |
|------------------|---------|---------|
| `skills`          | table   | Dictionary of all skills available to the player, keyed by skill ID. |
| `selectedSkills`  | table   | Dictionary of currently equipped skills, keyed by their skill slot index. |
| `skillIndex`      | integer | Tracks the next available skill slot index for equipping skills. |

---

### Method Summary

| Method                        | Execution Scope | Purpose |
|--------------------------------|----------------|---------|
| `LoadSkills()`                 | Server         | Loads all skill data from `_DataService` and creates skill instances, sending initial skill inventory updates to clients. |
| `OnBeginPlay()`                | ServerOnly     | Initializes the component on entity spawn and calls `LoadSkills()`. |
| `EquipSkill(skillId)`          | Server         | Equips a skill into the next available slot, initializes it, and sends a skill equip event to the client. |
| `UseSkill(skillIndex)`         | Server         | Triggers a skill at the given slot index. |
| `OnSkillComplete(player)`      | Server/Client  | Callback executed when a skill finishes; intended for post-skill logic. |
| `SendSkillEquipEvent(skill, index)` | Client   | Sends an `OnSkillEquippedEvent` to the client with active, inactive, and highlighted skill icons. |
| `SendSkillInventoryUpdateEvent(skill)` | Client | Sends a `SkillInventoryUpdatedEvent` to update the client UI with new skills. |
| `ClientUseSkill(skill)`        | Client         | Initializes and triggers a skill on the client side. |
| `HandleSkillUsedEvent(event)`  | Server/Self    | Handles `SkillUsedEvent` to trigger the corresponding skill. |
| `HandleSkillSelected(event)`   | Server/Self    | Handles `SkillSelected` events to equip the selected skill. |

---

### Methods

<details>
<summary>LoadSkills()</summary>

**Execution Scope**: `Server`  

**Purpose**  
Loads all skill data from `_DataService`, creates `Skill` objects, stores them in the `skills` dictionary, and sends initial inventory update events to clients.

**Behavior**  
1. Fetches the `SkillData` table from `_DataService`.  
2. Iterates through each row, creating a `Skill` instance via `_SkillFactory`.  
3. Stores each skill in the `skills` dictionary keyed by `skillId`.  
4. Sends a `SkillInventoryUpdatedEvent` for each skill.

</details>

<details>
<summary>EquipSkill(skillId)</summary>

**Execution Scope**: `Server`  

**Purpose**  
Equips a skill into the next available slot and synchronizes it with the client.

**Behavior**  
1. Retrieves the `Skill` instance from `skills`.  
2. Sets `skillIndex` and initializes the skill with a completion callback.  
3. Stores the skill in `selectedSkills` at the current `skillIndex`.  
4. Sends an `OnSkillEquippedEvent` to the client.  
5. Increments `skillIndex` for the next equip.

</details>

<details>
<summary>UseSkill(skillIndex)</summary>

**Execution Scope**: `Server`  

**Purpose**  
Triggers the skill equipped in the specified slot index.

**Behavior**  
1. Retrieves the skill from `selectedSkills`.  
2. Calls `OnTrigger` on the skill to execute its effect.

</details>

<details>
<summary>SendSkillEquipEvent(skill, index)</summary>

**Execution Scope**: `Client`  

**Purpose**  
Notifies the client UI that a skill has been equipped, providing references for active, inactive, and highlighted icons.

**Behavior**  
- Creates an `OnSkillEquippedEvent` and fills its properties with the skill’s icon RUIDs and index.  
- Sends the event to the entity.

</details>

<details>
<summary>SendSkillInventoryUpdateEvent(skill)</summary>

**Execution Scope**: `Client`  

**Purpose**  
Updates the client UI with a new skill in the inventory.

**Behavior**  
- Creates a `SkillInventoryUpdatedEvent` containing the skill’s active icon and skillId.  
- Sends the event to the entity.

</details>

<details>
<summary>ClientUseSkill(skill)</summary>

**Execution Scope**: `Client`  

**Purpose**  
Initializes and triggers a skill locally for client-side effects.

**Behavior**  
- Calls `Init` on the skill with a completion callback.  
- Executes `ClientTrigger` for the skill.

</details>

<details>
<summary>HandleSkillUsedEvent(event)</summary>

**Execution Scope**: `Server/Self`  

**Purpose**  
Handles incoming `SkillUsedEvent` by executing the corresponding skill.

**Behavior**  
- Retrieves `skillId` from the event and calls `UseSkill`.

</details>

<details>
<summary>HandleSkillSelected(event)</summary>

**Execution Scope**: `Server/Self`  

**Purpose**  
Equips a skill based on user selection from the UI.

**Behavior**  
- Extracts `skillId` from the event.  
- Calls `EquipSkill(skillId)`.

</details>

---

### Related Components & Events
- **Skill** — Represents individual skills and their logic.  
- **OnSkillEquippedEvent** — Event sent when a skill is equipped to update the client UI.  
- **SkillInventoryUpdatedEvent** — Event sent to update the client skill inventory UI.  
- **_SkillFactory** — Service responsible for creating skill instances from data.  
- **_DataService** — Provides access to skill data tables.  
- **_TimerService** — Used to schedule repeating or delayed actions for skills.  

## Skill

**Purpose**  
Represents an individual skill with all its properties, effects, animations, and behavior logic. Handles skill initialization, activation, target selection, and completion callbacks.

---

### Properties

| Property                  | Type    | Purpose |
|---------------------------|---------|---------|
| `skillIconActiveRUID`     | string  | Resource ID for the skill's active icon. |
| `skillIconInactiveRUID`   | string  | Resource ID for the skill's inactive icon. |
| `skillIconHighlightRUID`  | string  | Resource ID for the skill's highlighted icon. |
| `activationEffectRUIDs`   | table   | List of effect IDs triggered when the skill is activated. |
| `targetEffectRUIDs`       | table   | List of effect IDs applied to targets of the skill. |
| `cooldown`                | number  | Time in seconds before the skill can be used again. |
| `damage`                  | number  | Base damage value of the skill. |
| `hitCount`                | number  | Number of hits the skill deals per activation. |
| `animation`               | string  | Animation ID associated with the skill. |
| `skillIndex`              | integer | Slot index of the skill when equipped. |
| `skillId`                 | string  | Unique identifier for the skill. |
| `activationSFX`           | string  | Sound effect played when the skill is activated. |
| `hitSFX`                  | string  | Sound effect played when the skill hits a target. |
| `maxTargetCount`          | integer | Maximum number of targets the skill can affect. |
| `level`                   | integer | Current skill level, used for scaling damage or effects. |

---

### Method Summary

| Method                | Execution Scope | Purpose |
|-----------------------|----------------|---------|
| `Build(builder)`      | Client/Server         | Populates the skill properties using a `SkillBuilder` instance and returns the skill. |
| `Init(onSkillComplete)` | Client/Server       | Initializes the skill with a callback function to be called when the skill completes. |
| `OnTrigger(context)`  | Client/Server         | Activates the skill for the provided entity context. |
| `AttackEnemy(context, target)` | Client/Server | Executes the attack logic on a target entity. |
| `CanTargetEnemy()`    | Client/Server         | Determines if the skill can target an enemy. Defaults to `true`. |
| `OnSkillComplete()`   | Client/Server         | Calls the skill completion callback set in `Init`. |
| `ClientTrigger(context)` | ClientOnly   | Triggers the skill logic on the client side for visual or audio effects. |

---

### Methods

<details>
<summary>Build(builder)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes all skill properties using a `SkillBuilder` object.

**Behavior**  
- Copies all relevant fields from `builder` to the skill instance.  
- Returns the populated `Skill` object.

</details>

<details>
<summary>Init(onSkillComplete)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets up a callback function that will be executed when the skill finishes.

**Parameters**  
- `onSkillComplete` *(function)*: Callback invoked after skill execution.

**Behavior**  
- Stores the callback internally for later use by `OnSkillComplete`.

</details>

<details>
<summary>OnTrigger(context)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Activates the skill for a given entity.

**Parameters**  
- `context` *(Entity)*: The entity using the skill.

**Behavior**  
- Stores the entity context internally for skill execution.

</details>

<details>
<summary>AttackEnemy(context, target)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Handles logic to apply the skill's effects to a target entity.

**Parameters**  
- `context` *(Entity)*: Entity using the skill.  
- `target` *(Entity)*: Target entity affected by the skill.

</details>

<details>
<summary>CanTargetEnemy()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Checks whether the skill is allowed to target an enemy.

**Returns**  
- *(boolean)* – `true` if targeting is allowed; defaults to `true`.

</details>

<details>
<summary>OnSkillComplete()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Invokes the callback function set during initialization to notify that the skill has finished.

</details>

<details>
<summary>ClientTrigger(context)</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Activates client-side effects such as visuals or audio for the skill.

**Parameters**  
- `context` *(Entity)*: Entity using the skill.

</details>

---

### Related Types & Services
- **SkillBuilder** — Constructs and provides initial skill data.  
- **Entity** — Represents the context in which the skill executes.  
- **_EffectService** — Handles visual effects for skills.  
- **_SoundService** — Plays audio effects associated with skills.  

## SkillBuilder

**Purpose**  
A builder struct used to construct `Skill` instances with fluent method chaining. Encapsulates all configurable properties of a skill, including icons, effects, cooldowns, damage, animations, and targeting rules.

---

### Properties

| Property                  | Type    | Purpose |
|---------------------------|---------|---------|
| `skillIconActiveRUID`     | string  | Resource ID for the skill's active icon. |
| `skillIconInactiveRUID`   | string  | Resource ID for the skill's inactive icon. |
| `skillIconHighlightRUID`  | string  | Resource ID for the skill's highlighted icon. |
| `activationEffectRUIDs`   | table   | List of effect IDs triggered when the skill is activated. |
| `targetEffectRUIDs`       | table   | List of effect IDs applied to the skill's targets. |
| `cooldown`                | number  | Time in seconds before the skill can be reused. |
| `damage`                  | number  | Base damage value of the skill. |
| `hitCount`                | integer | Number of hits per skill activation. |
| `hitInterval`             | number  | Interval in seconds between consecutive hits. |
| `animation`               | string  | Animation ID associated with the skill. |
| `targetType`              | string  | Specifies the type of target the skill can affect. |
| `skillId`                 | string  | Unique identifier for the skill. |
| `hitSFX`                  | string  | Sound effect ID played when the skill hits a target. |
| `activationSFX`           | string  | Sound effect ID played when the skill is activated. |
| `maxTargetCount`          | integer | Maximum number of targets the skill can hit. |

---

### Method Summary

| Method                         | Execution Scope | Purpose |
|--------------------------------|----------------|---------|
| `WithIcon(active, inactive, highlight)` | Client/Server | Sets the skill icons for active, inactive, and highlight states. |
| `withActivationEffectRUIDs(ids)`       | Client/Server | Sets activation effects by parsing a string of IDs separated by underscores. |
| `withTargetEffectRUIDs(ids)`           | Client/Server | Sets target effects by parsing a string of IDs separated by underscores. |
| `withCooldown(cooldown)`               | Client/Server | Sets the cooldown time of the skill. |
| `withDamage(damage)`                   | Client/Server | Sets the skill's damage value. |
| `withHitCount(hitCount, hitInterval)`  | Client/Server | Sets the number of hits and interval between hits. |
| `withAnimation(animation)`             | Client/Server | Sets the animation associated with the skill. |
| `withTargetingType(type)`              | Client/Server | Sets the targeting type of the skill. |
| `WithId(skillId)`                      | Client/Server | Sets the unique skill ID. |
| `WithActivationSFX(activationSFX)`    | Client/Server | Sets the activation sound effect ID. |
| `WithHitSFX(hitSFX)`                   | Client/Server | Sets the hit sound effect ID. |
| `WithMaxTargetCount(maxTargetCount)`   | Client/Server | Sets the maximum number of targets the skill can affect. |
| `Build()`                              | Client/Server | Creates a `Skill` instance using the current builder configuration. |

---

### Methods

<details>
<summary>WithIcon(active, inactive, highlight)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the skill's active, inactive, and highlight icon IDs.

**Parameters**  
- `active` *(string)* – Active icon RUID.  
- `inactive` *(string)* – Inactive icon RUID.  
- `highlight` *(string)* – Highlight icon RUID.

**Returns**  
- `SkillBuilder` – Enables fluent method chaining.

</details>

<details>
<summary>withActivationEffectRUIDs(ids)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns activation effect IDs from a string of IDs separated by underscores.

**Parameters**  
- `ids` *(string)* – Underscore-delimited effect IDs.

**Returns**  
- `SkillBuilder` – Enables chaining.

</details>

<details>
<summary>withTargetEffectRUIDs(ids)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns target effect IDs from a string of IDs separated by underscores.

**Parameters**  
- `ids` *(string)* – Underscore-delimited effect IDs.

**Returns**  
- `SkillBuilder` – Enables chaining.

</details>

<details>
<summary>withCooldown(cooldown)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the skill's cooldown duration.

**Parameters**  
- `cooldown` *(number)* – Time in seconds.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>withDamage(damage)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the base damage value of the skill.

**Parameters**  
- `damage` *(number)* – Damage amount.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>withHitCount(hitCount, hitInterval)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Configures the number of hits and interval between hits.

**Parameters**  
- `hitCount` *(integer)* – Number of hits per activation.  
- `hitInterval` *(number)* – Time between hits in seconds.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>withAnimation(animation)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns an animation ID for the skill.

**Parameters**  
- `animation` *(string)* – Animation resource ID.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>withTargetingType(type)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the type of targets the skill can affect.

**Parameters**  
- `type` *(string)* – Target type identifier.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>WithId(skillId)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the unique skill identifier.

**Parameters**  
- `skillId` *(string)* – Unique skill ID.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>WithActivationSFX(activationSFX)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns a sound effect to play when the skill is activated.

**Parameters**  
- `activationSFX` *(string)* – Activation sound ID.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>WithHitSFX(hitSFX)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns a sound effect to play when the skill hits a target.

**Parameters**  
- `hitSFX` *(string)* – Hit sound ID.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>WithMaxTargetCount(maxTargetCount)</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Limits the number of targets the skill can affect.

**Parameters**  
- `maxTargetCount` *(integer)* – Maximum targets.

**Returns**  
- `SkillBuilder`.

</details>

<details>
<summary>Build()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Creates a `Skill` instance using the current configuration of the builder.

**Returns**  
- `Skill` – Fully built skill object.

</details>

---

### Related Types & Services
- **Skill** — Represents the final skill with all properties and behavior.  
- **_UtilLogic** — Provides utility functions like string splitting used for effect IDs.  

## SkillInventoryUpdatedEvent

**Purpose**  
Event dispatched when a skill is added to or updated in the player's skill inventory UI. Carries information about the skill's icon and identifier for client-side display.

---

### Properties

| Property     | Type   | Purpose |
|-------------|--------|---------|
| `iconRUID`  | string | Resource ID of the skill's icon to display in the UI. |
| `skillId`   | string | Unique identifier of the skill that was updated. |

---

### Related Types & Services
- **EventType** — Base class for all event structures.
- **PlayerSkillComponent** — Component responsible for managing the player's skills and sending this event.

## CharacterStats

**Purpose**  
Represents the core combat statistics for a character or entity, including health, power, defense, and level. Provides utility methods for retrieving stats and calculating damage.

---

### Properties

| Property       | Type    | Purpose |
|----------------|---------|---------|
| `hp`           | number  | Maximum health points of the character. |
| `defense`      | number  | Base defense value. |
| `level`        | number  | Character level, affecting damage calculations. |
| `power`        | number  | Attack power of the character. |
| `mp`           | number  | Magic or mana points (unused in basic calculations). |
| `finalDefense` | number  | Effective defense after adjustments or buffs. |

---

### Method Summary

| Method                | Execution Scope | Purpose |
|-----------------------|----------------|---------|
| `GetPower(defender)`  | Client/Server  | Returns the character's attack power; can be customized based on defender stats. |
| `GetDefense()`        | Client/Server  | Returns the effective defense value. |
| `GetHP()`             | Client/Server  | Returns the character's current or maximum health. |
| `Init(context)`       | Client/Server  | Initializes stats based on the provided entity context. |
| `GetDamage(level, powerValue)` | Client/Server | Calculates the damage output based on attacker level and power. |

---

### Methods

<details>
<summary>GetPower(defender)</summary>

**Execution Scope**: Client/Server  

**Purpose**  
Returns the attack power of the character, potentially factoring in the target’s stats.

**Parameters**  
- `defender` *(CharacterStats)*: Stats of the target being attacked.

**Returns**  
- *(number)*: Attack power value.

</details>

<details>
<summary>GetDefense()</summary>

**Execution Scope**: Client/Server  

**Purpose**  
Returns the final effective defense of the character.

**Returns**  
- *(number)*: Defense value.

</details>

<details>
<summary>GetHP()</summary>

**Execution Scope**: Client/Server  

**Purpose**  
Retrieves the character’s current health points.

**Returns**  
- *(number)*: Current HP value.

</details>

<details>
<summary>Init(context)</summary>

**Execution Scope**: Client/Server  

**Purpose**  
Initializes character stats using the provided entity context.

**Parameters**  
- `context` *(Entity)*: The entity whose stats are being initialized.

</details>

<details>
<summary>GetDamage(level, powerValue)</summary>

**Execution Scope**: Client/Server  

**Purpose**  
Calculates the raw damage output based on attacker level and power value.

**Parameters**  
- `level` *(integer)*: Level of the attacker.  
- `powerValue` *(number)*: Base attack power to use for calculation.

**Returns**  
- *(number)*: Calculated damage value.

</details>

---

### Related Types & Services
- **Entity** — Provides context for initializing stats.  
- **CharacterStats** — Can be used for both attacker and defender calculations.

## PlayerStatsEnum

**Purpose**  
Defines constant identifiers for primary player stats, allowing consistent references in code for attributes like Strength, Dexterity, Intelligence, and Luck.

---

### Properties

| Property | Type   | Purpose |
|----------|--------|---------|
| `STR`   | number | Identifier for the Strength stat. |
| `DEX`   | number | Identifier for the Dexterity stat. |
| `INT`   | number | Identifier for the Intelligence stat. |
| `LUK`   | number | Identifier for the Luck stat. |
| `MAX`   | number | Represents the total count of primary stats. |

---

### Method Summary

- This script contains no methods. It serves solely as a constant enum for referencing player stats.

---

### Related Types & Services
- **PlayerStatsEnum** — Can be used in conjunction with character stat management systems, skill calculations, or UI displays.

## PlayerInfoComponent

**Purpose**  
Manages player-specific information such as stats, HP, MP, and level. Handles stat updates, level-ups, and synchronization between server and client.

---

### Properties

| Property | Type  | Sync | Default Value | Purpose |
|----------|-------|------|---------------|---------|
| `stats` | any   | No   | `nil`         | Reference to the player's stats object. |
| `hp`    | number| No   | `50`          | Current HP of the player. |
| `mp`    | number| No   | `50`          | Current MP of the player. |
| `level` | number| Yes  | `20`          | Current player level, synchronized across server and client. |

---

### Method Summary

| Method                  | Execution Scope | Purpose |
|-------------------------|----------------|---------|
| `OnBeginPlay()`         | Client/Server  | Reserved for initialization logic when the entity spawns. |
| `GetAttackPower()`      | Client/Server  | Returns the player's attack power based on current stats. |
| `SetHp(hp)`             | Server         | Updates the player’s MaxHp and current Hp on the server. |
| `LevelUp()`             | Server         | Increases player level, updates stats, sets HP, and plays level-up effect. |
| `IncreaseStats(index, value)` | Server   | Increases a specific stat and triggers a client update event. |
| `SendStatUpdateEvt(index, finalValue)` | Client | Sends a `PlayerStatsUpdate` event to update the UI. |
| `GetStats()`            | Client/Server  | Returns the `PlayerStats` object. |
| `OnInitialize()`        | Client/Server  | Initializes the player stats object and sets initial HP on the server. |
| `HandlePlayerStatIncreased(event)` | Server/Client | Handles stat increase events, applying changes via `IncreaseStats`. |

---

### Methods

<details>
<summary>OnBeginPlay()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Reserved for initialization logic when the entity spawns.

</details>

<details>
<summary>GetAttackPower()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns the player's calculated attack power using the `stats` object.

</details>

<details>
<summary>SetHp(hp)</summary>

**Execution Scope**: `Server`  

**Purpose**  
Updates the player's MaxHp and current Hp on the server.  

**Parameters**  
- `hp` *(number)*: New HP value to set.

**Behavior**  
- Sets `PlayerComponent.MaxHp` and `PlayerComponent.Hp` to the specified value.

</details>

<details>
<summary>LevelUp()</summary>

**Execution Scope**: `Server`  

**Purpose**  
Handles player leveling logic.

**Behavior**  
1. Calls `stats:LevelUp()`.  
2. Updates `level` and HP values.  
3. Plays a visual effect indicating the level-up.

</details>

<details>
<summary>IncreaseStats(index, value)</summary>

**Execution Scope**: `Server`  

**Purpose**  
Increases a specific player stat and updates clients.

**Parameters**  
- `index` *(integer)*: Stat index to increase.  
- `value` *(number)*: Amount to increase the stat by.

**Behavior**  
- Calls `stats:IncreaseStat(index, value)`.  
- Sends a `PlayerStatsUpdate` event to clients.

</details>

<details>
<summary>SendStatUpdateEvt(index, finalValue)</summary>

**Execution Scope**: `Client`  

**Purpose**  
Sends an event to the client UI to reflect updated stat values.

**Parameters**  
- `index` *(integer)*: Stat index that changed.  
- `finalValue` *(number)*: Updated value of the stat.

</details>

<details>
<summary>GetStats()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns the player's `PlayerStats` object for external use.

</details>

<details>
<summary>OnInitialize()</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes the player's stats object and sets starting HP for server-side entities.

**Behavior**  
- Creates a new `PlayerStats` object and initializes it.  
- Sets the server-side HP if running on the server.

</details>

<details>
<summary>HandlePlayerStatIncreased(event)</summary>

**Execution Scope**: `Server/Client`  

**Purpose**  
Handles incoming stat increase events and applies the changes.

**Parameters**  
- `event.increaseIndex` *(integer)*: Stat index to increase.  
- `event.increaseValue` *(number)*: Amount to increase.

**Behavior**  
- Calls `IncreaseStats` with the event values.

</details>

---

### Related Components & Scripts
- **PlayerStats** — Contains the player’s stat values and logic for calculation.  
- **PlayerStatsUpdate** — Event sent to update client UI when a stat changes.  
- **PlayerComponent** — Represents core player data like HP and MP.  
- **_EffectService** — Handles visual effects such as level-up animations.

---

## PlayerStatsUpdate

**Purpose**  
Event sent to clients to update the UI or other systems when a player’s stat changes.

---

### Properties

| Property     | Type    | Purpose |
|-------------|---------|---------|
| `index`     | integer | Index of the stat that has been updated. |
| `finalValue`| number  | New value of the updated stat. |

---

## PlayerAnimationComponent

**Purpose**  
Handles player avatar animations, allowing immediate playback of actions with configurable frame ranges and playback speed.

---

### Method Summary

| Method             | Execution Scope | Purpose |
|--------------------|----------------|---------|
| `PlayImmediatly`   | Client         | Plays a specified animation action immediately on the player avatar. |
| `OnBeginPlay`      | ClientOnly     | Initializes a temporary event object for animation playback. |

### Methods

<details>
<summary>PlayImmediatly</summary>

**Execution Scope**: `Client`  
**Dependencies**: `ActionStateChangedEvent`, `SpriteAnimClipPlayType`, `AvatarRendererComponent`

**Purpose**  
Immediately plays the given animation action on the player avatar.

**Parameters**  
- `actionName` *(string)* – Name of the animation action.  
- `playRate` *(number)* – Playback speed multiplier.  
- `startFrame` *(integer)* – Starting frame index.  
- `endFrame` *(integer)* – Ending frame index.

**Behavior**  
1. Retrieves `_T.tempEvent`, a temporary `ActionStateChangedEvent`.  
2. Sets the core and parts action names, play rate, play type, and frame range.  
3. Sends the event to the body entity via `AvatarRendererComponent:GetBodyEntity()`.

</details>

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Prepares a temporary `ActionStateChangedEvent` instance for later use in `PlayImmediatly`.

**Behavior**  
- Initializes `_T.tempEvent` as a new `ActionStateChangedEvent`.

</details>

---

### Related Types & Services
- **ActionStateChangedEvent** – Event used to trigger animation changes on an entity.  
- **SpriteAnimClipPlayType** – Enum specifying animation playback type (e.g., Onetime).  
- **AvatarRendererComponent** – Provides access to the player’s body entity for sending animation events.  

---

## SkillInfoBuilder

**Purpose**  
Provides a fluent interface to construct skill execution information for entities, including animations, effects, attack data, and execution flags in MapleStory Worlds.

---

### Properties

| Property       | Type    | Purpose |
|----------------|---------|---------|
| `infos`        | table   | Stores skill execution flags and data keyed by `_SkillExecutionEnum`. |
| `context`      | Entity  | The entity that will execute the skill. |

---

### Method Summary

| Method                     | Execution Scope   | Purpose |
|-----------------------------|-----------------|---------|
| `Init`                      | Client/Server   | Initializes the builder with a context entity and default execution flags. |
| `withAttackInfo`            | Client/Server   | Sets attack information for the skill. |
| `withActivationEffect`      | Client/Server   | Assigns effects that trigger when the skill activates. |
| `withActivationSFX`         | Client/Server   | Assigns a sound effect to play upon activation. |
| `withAnimation`             | Client/Server   | Sets the animation for the skill execution. |
| `WithClientExecution`       | Client/Server   | Marks the skill for client-side execution. |
| `WithDisableController`     | Client/Server   | Disables the player's controller during skill execution. |
| `WithOnTargetHit`           | Client/Server   | Marks the skill to trigger logic when hitting a target. |
| `WithAttackFrameInfo`       | Client/Server   | Defines the attack frame shape, activation, and length. |
| `WithAttack`                | Client/Server   | Marks the skill as an attacking skill. |
| `Build`                     | Client/Server   | Returns the constructed skill info table. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes the builder with a context entity and default skill execution flags.

**Parameters**  
- `context` *(Entity)* – The entity that will execute the skill.

**Behavior**  
1. Sets `self.context` to the provided entity.  
2. Initializes default flags (`ATTACKING`, `ONTARGETHIT`, `DISABLECONTROLLER`, `CLIENTEXECUTION`) to `false`.  
3. Returns the builder instance.

</details>

<details>
<summary>withAttackInfo</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets attack-related information for the skill.

**Parameters**  
- `attackInfo` *(table)* – Contains attack behavior or parameters.

**Behavior**  
- Stores the data in `infos[_SkillExecutionEnum.ATTACKINFO]`.  
- Returns the builder instance.

</details>

<details>
<summary>withActivationEffect</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Assigns effects that trigger when the skill is activated.

**Parameters**  
- `activationEffect` *(table)* – List of effect identifiers.

**Behavior**  
- Stores the table in `infos[_SkillExecutionEnum.ACTIVATIONEFFECT]`.  
- Returns the builder instance.

</details>

<details>
<summary>withActivationSFX</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Specifies a sound effect to play upon skill activation.

**Parameters**  
- `sfx` *(string)* – Sound effect identifier.

**Behavior**  
- Stores the SFX in `infos[_SkillExecutionEnum.ACTIVATIONSFX]`.  
- Returns the builder instance.

</details>

<details>
<summary>withAnimation</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the animation for the skill execution.

**Parameters**  
- `animation` *(string)* – Animation identifier.

**Behavior**  
- Stores the animation in `infos[_SkillExecutionEnum.ANIMATION]`.  
- Returns the builder instance.

</details>

<details>
<summary>WithClientExecution</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Marks the skill for client-side execution.

**Behavior**  
- Sets `infos[_SkillExecutionEnum.CLIENTEXECUTION] = true`.  
- Returns the builder instance.

</details>

<details>
<summary>WithDisableController</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Disables the player's controller during skill execution.

**Behavior**  
- Sets `infos[_SkillExecutionEnum.DISABLECONTROLLER] = true`.  
- Returns the builder instance.

</details>

<details>
<summary>WithOnTargetHit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Marks the skill to trigger logic when hitting a target.

**Behavior**  
- Sets `infos[_SkillExecutionEnum.ONTARGETHIT] = true`.  
- Returns the builder instance.

</details>

<details>
<summary>WithAttackFrameInfo</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Defines the timing and shape of the skill's attack frames.

**Parameters**  
- `frameShape` *(integer)* – Shape or index of the attack.  
- `frameActivation` *(integer)* – Frame when the attack activates.  
- `frameLength` *(integer)* – Total frame duration.

**Behavior**  
- Stores a table with `Shape`, `Activation`, and `Length` in `infos[_SkillExecutionEnum.ATTACKFRAMEINFO]`.  
- Returns the builder instance.

</details>

<details>
<summary>WithAttack</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Marks the skill as an attacking skill.

**Behavior**  
- Sets `infos[_SkillExecutionEnum.ATTACKING] = true`.  
- Returns the builder instance.

</details>

<details>
<summary>Build</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Finalizes and returns the constructed skill info table.

**Returns**  
- *(table)* – Complete skill execution information.

</details>

---

### Related Types & Services
- **_SkillExecutionEnum** – Enum containing keys and flags for skill execution (e.g., ATTACKING, ONTARGETHIT, DISABLECONTROLLER, CLIENTEXECUTION).  
- **Entity** – Context entity executing the skill.

---

## SkillFactory

**Purpose**  
Provides methods to construct skill instances with data-driven configuration, including specialized skills like `ComboForce`, `Incising`, and `RagingBlow`, using a fluent `SkillBuilder`.

---

### Properties

| Property       | Type    | Purpose |
|----------------|---------|---------|
| `NewValue1`    | integer | Example placeholder property, usage context may vary. |

---

### Method Summary

| Method                  | Execution Scope | Purpose |
|-------------------------|----------------|---------|
| `CreateSkillWithData`   | Client/Server  | Creates a skill instance based on data from a `UserDataRow`, configuring it via `SkillBuilder`. |

---

### Methods

<details>
<summary>CreateSkillWithData</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Constructs a skill instance using provided data and returns the fully built skill.

**Parameters**  
- `data` *(any)* – Expected to be a `UserDataRow` containing skill configuration.

**Behavior**  
1. Retrieves the skill ID from `data`.  
2. Instantiates a specific skill class depending on the ID (`ComboForce`, `Incising`, `RagingBlow`, `WarriorLeap`) or defaults to `Skill`.  
3. Uses `SkillBuilder` to configure the skill with:
   - Icons (active, inactive, highlight)  
   - Activation and target effects  
   - Cooldown, damage, hit count, and hit interval  
   - Skill ID  
   - Hit and activation sound effects  
   - Maximum target count  
   - Animation  
4. Calls `skill:Build(builder)` to finalize the skill configuration.  
5. Returns the configured skill instance.

**Returns**  
- *(Skill)* – Fully built skill instance ready for use.

</details>

---

### Related Types & Services
- **SkillBuilder** – Fluent builder for configuring skill properties.  
- **UserDataRow** – Data structure containing skill parameters.  
- **_SkillsEnum** – Enum defining skill IDs.  
- **Skill** – Base skill class.  
- **ComboForce**, **Incising**, **RagingBlow**, **WarriorLeap** – Specialized skill subclasses.  

---

## SkillsEnum

**Purpose**  
Provides a centralized set of string constants representing unique skill IDs for hero skills, used for identification and logic branching in skill creation and execution.

---

### Properties

| Property               | Type   | Purpose |
|------------------------|--------|---------|
| `HERO_RAGINGBLOW`      | string | Skill ID for the "Raging Blow" hero skill. |
| `HERO_COMBOFORCE`      | string | Skill ID for the "Combo Force" hero skill. |
| `HERO_INCISING`        | string | Skill ID for the "Incising" hero skill. |
| `HERO_WARRIORLEAP`     | string | Skill ID for the "Warrior Leap" hero skill. |

---

### Related Types & Services
- **Logic** – Base class for logic scripts in MapleStory Worlds.  
- **SkillsEnum** – Provides a central reference for hero skill identifiers.

---

## MonsterStats

**Purpose**  
Extends `CharacterStats` to provide specialized stats and HP calculations for monsters in MapleStory Worlds, including level-based scaling.

---

### Methods

| Method                  | Execution Scope | Purpose |
|-------------------------|----------------|---------|
| `Init`                  | Client/Server  | Initializes monster stats from an entity context, setting level and calculating HP. |
| `CalculateHP`           | Client/Server  | Calculates the monster's total HP using baseline HP and level multipliers. |
| `GetBaselineHP`         | Client/Server  | Returns the base HP for the monster based on its level. |
| `GetLevelMultipliers`   | Client/Server  | Returns a multiplier based on level for scaling HP. |

---

### Methods

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes the monster stats from the given entity context.

**Parameters**  
- `context` *(Entity)* – The monster entity used to retrieve level and other info.

**Behavior**  
1. Retrieves the monster object from the entity.  
2. Sets `self.level` to the monster's level.  
3. Calls `CalculateHP()` to compute total HP.

</details>

<details>
<summary>CalculateHP</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates the monster's total HP based on baseline HP and level multipliers.

**Behavior**  
- Sets `self.hp` to `GetBaselineHP() * GetLevelMultipliers()`.

</details>

<details>
<summary>GetBaselineHP</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Computes the base HP of a monster based on its level using piecewise linear formulas.

**Returns**  
- *(integer)* – The baseline HP value.

**Behavior**  
- Uses a series of conditional checks for different level ranges to calculate base HP.

</details>

<details>
<summary>GetLevelMultipliers</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Provides a multiplier for HP scaling based on the monster’s level.

**Returns**  
- *(number)* – The level multiplier.

**Behavior**  
- Multiplier increases for higher level ranges to adjust monster HP appropriately.

</details>

---

### Related Types & Services
- **CharacterStats** – Base stats class for both players and monsters.  
- **Entity** – Represents the monster in the game world.  
- **Monster** – Contains level information for the entity.  

---

## PlayerStats

**Purpose**  
Extends `CharacterStats` to manage player-specific stats, including HP, MP, primary/secondary attributes, mastery, damage calculation, defense, and level scaling. Supports dynamic computation for combat and leveling systems.

---

### Properties

| Property              | Type    | Purpose |
|-----------------------|---------|---------|
| `addedStats`          | table   | Additional stats applied to base stats (e.g., from equipment or buffs). |
| `jobClassValue`       | number  | Combined value of primary and secondary stats used for damage calculation. |
| `baseStats`           | table   | The base stats for each stat type. |
| `mastery`             | number  | Skill mastery percentage affecting damage. |
| `percentageStats`     | table   | Percentage-based stat modifiers. |
| `finalStats`          | table   | Additional stat values applied after base and percentage calculations. |
| `critChance`          | number  | Chance of critical hit (percentage). |
| `addedDefense`        | number  | Additional defense applied on top of computed defense. |

---

### Method Summary

| Method                          | Execution Scope | Purpose |
|---------------------------------|----------------|---------|
| `IncreaseStat`                  | Client/Server  | Increases a specific stat and recalculates class value and defense. |
| `Init`                           | Client/Server  | Initializes player stats from entity context and sets up base stats. |
| `SetToBaseStats`                 | Client/Server  | Sets default base stats for the player. |
| `GetUpperDamageRange`            | Client/Server  | Computes the maximum damage value against a defender. |
| `GetLowerDamageRange`            | Client/Server  | Computes the minimum damage value against a defender. |
| `CalculateClassValue`            | Client/Server  | Computes the job class value based on primary and secondary stats. |
| `GetStat`                        | Client/Server  | Returns the effective stat value for a given type. |
| `GetLevelAdvantageMultiplier`    | Client/Server  | Returns a multiplier based on level difference for damage calculation. |
| `SetLevelAdvantagePcts`          | Client/Server  | Initializes level advantage percentages table. |
| `GetPower`                        | Client/Server  | Calculates random power (damage) range against a defender. |
| `GainHP`                         | Client/Server  | Returns HP gain on leveling. |
| `GainMP`                         | Client/Server  | Returns MP gain on leveling. |
| `LevelUp`                        | Client/Server  | Levels up the player and increases HP/MP. |
| `CalculateDefense`               | Client/Server  | Calculates final defense based on stats and added defense. |
| `GetUpperDefenseRange`           | Client/Server  | Computes maximum damage reduction against an attack. |
| `GetLowerDefenseRange`           | Client/Server  | Computes minimum damage reduction against an attack. |
| `LevelDefenseAdvantageMultiplierA` | Client/Server | Returns A multiplier for defense scaling based on level difference. |
| `LevelDefenseAdvantageMultiplierB` | Client/Server | Returns B multiplier for defense scaling based on level difference. |
| `GetDamage`                       | Client/Server | Calculates actual damage taken after defense. |
| `GetStats`                        | Client/Server | Returns a table of all computed stat values. |

---

### Methods

<details>
<summary>IncreaseStat</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Increases a given stat and recalculates dependent values.

**Parameters**  
- `type` *(integer)* – Stat type identifier.  
- `value` *(integer)* – Value to add to the stat.

**Behavior**  
1. Adds `value` to `baseStats[type]`.  
2. Recalculates class value and defense.  
3. Placeholder to save new class value to database in the future.

</details>

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes player stats using the entity context.

**Parameters**  
- `context` *(Entity)* – Player entity context containing level and info.

**Behavior**  
1. Calls base class init.  
2. Sets `self.level` from player info.  
3. Sets base stats, calculates class value and defense, and sets level advantage percentages.

</details>

<details>
<summary>SetToBaseStats</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes base stats, added stats, percentage stats, and final stats.

**Behavior**  
- HP = 50, MP = 50, Mastery = 20, Defense = 5.  
- Loops through all stat types and assigns random values for base stats.  
- Initializes other stat tables to 0.

</details>

<details>
<summary>CalculateClassValue</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Computes `jobClassValue` using primary and secondary stats.

**Behavior**  
- Primary = STR stat, Secondary = DEX stat.  
- `jobClassValue = primary * 4 + secondary`.

</details>

<details>
<summary>CalculateDefense</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Computes `finalDefense` using STR, DEX, LUK, and added defense.

**Behavior**  
- `finalDefense = (1.5 * STR + 0.4 * (DEX + LUK) + addedDefense) * (1 + defense/100)`

</details>

<details>
<summary>GetUpperDamageRange</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates maximum damage output against a defender based on level advantage and class value.

</details>

<details>
<summary>GetLowerDamageRange</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates minimum damage output based on upper damage range and mastery.

</details>

<details>
<summary>GainHP / GainMP</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Provides random HP/MP gains on level up.

</details>

<details>
<summary>LevelUp</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Levels up the player and increases HP/MP.

</details>

<details>
<summary>GetLevelAdvantageMultiplier / SetLevelAdvantagePcts</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Provides multipliers for damage based on level difference.

</details>

<details>
<summary>GetUpperDefenseRange / GetLowerDefenseRange</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates maximum and minimum damage reduction based on monster attack and player defense.

</details>

<details>
<summary>LevelDefenseAdvantageMultiplierA / LevelDefenseAdvantageMultiplierB</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Return scaling multipliers for defense based on level difference.

</details>

<details>
<summary>GetDamage</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates actual damage taken after applying defense ranges.

</details>

<details>
<summary>GetStats</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns a table of all computed stats for the player.

</details>

---

### Related Types & Services
- **CharacterStats** – Base class for player and monster stats.  
- **Entity** – Player entity context.  
- **_PlayerStatsEnum** – Enum for stat type identifiers.  
- **_UtilLogic** – Provides utility functions like `RandomIntegerRange`.  
- **_T** – Temporary table for runtime values such as level advantage percentages.  

---

## PlayerAttack

**Purpose**  
Extends `AttackComponent` to handle player attack logic, including normal attacks, damage calculation, critical hits, attack visualization, and skill completion events.

---

### Properties

| Property             | Type      | Purpose |
|----------------------|-----------|---------|
| `power`              | number    | Base power of the attack. |
| `attackVisualizer`   | Entity    | Visual entity used for debugging or showing attack areas. |
| `visualizeAttack`    | boolean   | Whether attack visualization is enabled. |
| `targets`            | number    | Tracks the number of targets hit by the current attack. |

---

### Method Summary

| Method                    | Execution Scope   | Purpose |
|----------------------------|-----------------|---------|
| `AttackNormal`             | Server          | Performs a normal attack with optional shape, offset, and attack info. |
| `CalcDamage`               | Client/Server   | Calculates damage dealt to a defender, applying attack info modifiers. |
| `CalcCritical`             | Client/Server   | Determines whether the attack is a critical hit. |
| `GetCriticalDamageRate`    | Client/Server   | Returns critical damage multiplier. |
| `OnAttack`                 | Client/Server   | Called when an attack hits a target; triggers events and counts targets. |
| `IsAttackTarget`           | Client/Server   | Checks whether an entity can be attacked and applies MaxTargets limits. |
| `OnSkillComplete`          | Server          | Event triggered when a skill is completed. |
| `VisualizeAttack`          | Client          | Shows a temporary visualization of the attack area. |
| `OnBeginPlay`              | ClientOnly      | Initializes attack visualizer entity if visualization is enabled. |
| `GetDisplayHitCount`       | Client/Server   | Returns the number of hits to display for an attack. |
| `HandlePlayerActionEvent`  | ServerOnly      | Handles player input events to trigger attacks. |

---

### Methods

<details>
<summary>AttackNormal</summary>

**Execution Scope**: `Server`  

**Purpose**  
Executes a standard attack from the player with optional shape, offset, attack info, and callback.

**Parameters**  
- `power` *(number)* – Attack power.  
- `customShape` *(Vector2)* – Optional size of the attack hitbox.  
- `offset` *(Vector2)* – Offset from the player position for the attack.  
- `msg` *(table)* – Optional attack info to encode and send.  
- `onAttack` *(any)* – Optional callback executed for each attack hit.

**Behavior**  
1. Calculates world position with offset and player direction.  
2. Visualizes the attack if `visualizeAttack` is true.  
3. Creates a `BoxShape` for collision detection.  
4. Encodes `msg` as JSON for attack info.  
5. Resets and stores `onAttack` callback.  
6. Resets `targets` count.  
7. Calls `AttackFast` with the hitbox and attack info.

</details>

<details>
<summary>CalcDamage</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Calculates the actual damage to a defender based on attacker stats and attack info.

**Parameters**  
- `attacker` *(Entity)* – Player entity performing the attack.  
- `defender` *(Entity)* – Target entity.  
- `attackInfo` *(string)* – JSON string containing attack modifiers.

**Behavior**  
1. Computes base damage from attacker’s stats.  
2. Applies `Power` modifier from `attackInfo` if present.  
3. Returns final damage or -1 if invalid.

</details>

<details>
<summary>CalcCritical</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Determines whether an attack is critical based on attacker’s crit chance.

**Behavior**  
- Generates a random double and compares to a percentage from crit chance.

</details>

<details>
<summary>GetCriticalDamageRate</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Returns multiplier for critical damage.

**Behavior**  
- Default multiplier = 2x.

</details>

<details>
<summary>OnAttack</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Called when an attack hits a target; executes stored callback and increments hit count.

</details>

<details>
<summary>IsAttackTarget</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Checks if an entity is a valid attack target and respects `MaxTargets` from attack info.

**Behavior**  
- Plays miss text if damage is zero.  
- Returns false if target limit reached.

</details>

<details>
<summary>VisualizeAttack</summary>

**Execution Scope**: `Client`  

**Purpose**  
Displays a temporary visual representation of the attack area.

**Parameters**  
- `size` *(Vector2)* – Size of the attack box.  
- `position` *(Vector2)* – World position of the attack.

**Behavior**  
- Enables `attackVisualizer`, sets size and position, waits 1 second, then disables.

</details>

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Spawns the attack visualizer entity for debugging purposes if enabled.

</details>

<details>
<summary>HandlePlayerActionEvent</summary>

**Execution Scope**: `ServerOnly`  

**Purpose**  
Handles player input events (e.g., "Attack") and triggers corresponding attack methods.

</details>

---

### Related Types & Services
- **AttackComponent** – Base attack logic.  
- **PlayerInfoComponent** – Provides player stats.  
- **DamageSkinService** – Plays visual damage feedback.  
- **TriggerComponent / ColliderVisualizer** – Used for visualizing attack areas.  
- **_HttpService** – JSON encode/decode for attack info.  
- **_UtilLogic** – Provides random number utilities.  
- **_SpawnService** – Spawns entities for visualization.
- **CollisionGroups.Monster** – Layer mask for hitting monsters.

---

## PlayerHit

**Purpose**  
Extends `HitComponent` to handle player hit logic, including immunity cooldowns, applying forces on hit, and displaying damage feedback.

---

### Properties

| Property          | Type      | Purpose |
|------------------|-----------|---------|
| `ImmuneCooldown` | number    | Minimum time (in seconds) between consecutive hits. |
| `LastHitTime`    | number    | Timestamp of the last hit received (hidden in inspector). |
| `hitForce`       | number    | Horizontal force applied to the player when hit. |
| `upwardForce`    | number    | Vertical force applied to the player when hit. |

---

### Method Summary

| Method          | Execution Scope | Purpose |
|-----------------|----------------|---------|
| `IsHitTarget`   | Client/Server  | Determines whether the entity can currently be hit, respecting immunity cooldown. |
| `OnHit`         | Client/Server  | Applies damage, knockback forces, and triggers damage feedback effects. |

---

### Methods

<details>
<summary>IsHitTarget</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Checks if the entity is eligible to receive a hit based on the immunity cooldown.

**Behavior**  
1. Retrieves current elapsed time from `_UtilLogic`.  
2. Compares with `LastHitTime + ImmuneCooldown`.  
3. Returns `true` if cooldown has passed and updates `LastHitTime`.  
4. Returns `false` otherwise.

**Parameters**  
- `attackInfo` *(string)* – Optional attack information.

</details>

<details>
<summary>OnHit</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Handles the entity being hit: applies knockback forces and triggers visual damage feedback.

**Parameters**  
- `attacker` *(Entity)* – Entity performing the attack.  
- `damage` *(integer)* – Amount of damage to apply.  
- `isCritical` *(boolean)* – Whether the hit was a critical strike.  
- `attackInfo` *(string)* – Optional attack information.  
- `hitCount` *(int32)* – Hit count in a multi-hit attack.

**Behavior**  
1. If damage ≤ 0, displays a "Miss" damage text and returns.  
2. Calculates push direction away from the attacker.  
3. Applies a normalized force combining horizontal and upward components.  
4. Calls base `OnHit` method to apply damage and other effects.

</details>

---

### Related Types & Services
- **HitComponent** – Base hit logic for entities.  
- **RigidbodyComponent** – Physics component for applying forces.  
- **TransformComponent** – Provides world position for calculations.  
- **_DamageSkinService** – Displays visual damage feedback (numbers, miss text).  
- **_UtilLogic** – Provides elapsed time and utility functions.
---

## SkillUIComponent

**Purpose**  
Manages the player's skill UI, including skill slots, icon states, and skill usage interactions in MapleStory Worlds.

---

### Properties

| Property             | Type    | Purpose |
|----------------------|---------|---------|
| `emptySlotRUID`      | string  | RUID of the empty skill slot icon. |
| `skillSlotParent`    | Entity  | Parent entity containing all skill slot UI elements. |

---

### Method Summary

| Method                  | Execution Scope | Purpose |
|-------------------------|----------------|---------|
| `OnBeginPlay`           | ClientOnly      | Initializes skill slots and connects button events. |
| `OnApplySkillSlot`      | Client          | Opens the skill window for the selected slot. |
| `OnTriggerSkill`        | Server/Client   | Sends a skill usage event for the selected slot. |
| `ApplyIcon`             | Client          | Updates a skill slot icon with active, highlight, and inactive images. |
| `ResetIcon`             | Client          | Resets a skill slot to inactive state. |
| `EnterCooldown`         | Client          | Activates cooldown visuals on the inactive icon. |
| `GetSkillSlot`          | Client          | Returns the skill slot entity at a given index. |
| `ActivateActiveIcon`    | Client          | Enables or disables the active icon for a skill slot. |
| `ActivateInactiveIcon`  | Client          | Enables or disables the inactive icon for a skill slot. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes skill slots and sets up button click events for activating or triggering skills.

**Behavior**  
1. Creates `_T.skillSlots` table.  
2. Iterates over all children of `skillSlotParent` and stores them in `_T.skillSlots`.  
3. Connects `ButtonClickEvent` on each slot to `OnApplySkillSlot(index)`.  
4. Connects `ButtonClickEvent` on each slot's `"Active"` child to `OnTriggerSkill(index)`.

</details>

<details>
<summary>OnApplySkillSlot</summary>

**Execution Scope**: `Client`  

**Purpose**  
Opens the skill window UI for the selected skill slot.

**Parameters**  
- `index` *(integer)* – Index of the skill slot.

**Behavior**  
- Calls `SkillWindowComponent:Show(true)` on the player entity.

</details>

<details>
<summary>OnTriggerSkill</summary>

**Execution Scope**: `Client`  

**Purpose**  
Triggers a skill usage event for the given skill slot.

**Parameters**  
- `index` *(integer)* – Index of the skill slot.

**Behavior**  
1. Creates a `SkillUsedEvent`.  
2. Sets `evt.skillId` to the slot index.  
3. Sends the event from `LocalPlayer`.

</details>

<details>
<summary>ApplyIcon</summary>

**Execution Scope**: `Client`  

**Purpose**  
Sets the active, highlight, and inactive icons for a skill slot.

**Parameters**  
- `active` *(string)* – RUID of the active icon.  
- `highlight` *(string)* – RUID of the highlight icon.  
- `inactive` *(string)* – RUID of the inactive icon.  
- `index` *(integer)* – Index of the skill slot.

**Behavior**  
1. Retrieves the skill slot entity.  
2. Activates the active icon.  
3. Updates `SpriteGUIRendererComponent` of active and inactive icons.  
4. Configures button highlight image.

</details>

<details>
<summary>ResetIcon</summary>

**Execution Scope**: `Client`  

**Purpose**  
Resets both active and inactive icons of a skill slot to disabled state.

**Parameters**  
- `index` *(integer)* – Index of the skill slot.

</details>

<details>
<summary>EnterCooldown</summary>

**Execution Scope**: `Client`  

**Purpose**  
Activates the inactive icon to indicate the skill is on cooldown.

**Parameters**  
- `skillIndex` *(integer)* – Index of the skill slot.

</details>

<details>
<summary>GetSkillSlot</summary>

**Execution Scope**: `Client`  

**Purpose**  
Returns the skill slot entity at the given index.

**Parameters**  
- `index` *(number)* – Skill slot index.

**Returns**: Entity – The skill slot entity.

</details>

<details>
<summary>ActivateActiveIcon</summary>

**Execution Scope**: `Client`  

**Purpose**  
Enables or disables the active icon of a skill slot.

**Parameters**  
- `index` *(integer)* – Skill slot index.  
- `isActive` *(boolean)* – Whether to enable the active icon.

</details>

<details>
<summary>ActivateInactiveIcon</summary>

**Execution Scope**: `Client`  

**Purpose**  
Enables or disables the inactive icon of a skill slot.

**Parameters**  
- `index` *(integer)* – Skill slot index.  
- `isActive` *(boolean)* – Whether to enable the inactive icon.

</details>

<details>
<summary>OnSkillEquippedEvent</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Handles skill equipped events and updates the UI icons accordingly.

**Parameters**  
- `event` *(OnSkillEquippedEvent)* – Event containing the skill RUIDs and slot index.

**Behavior**  
- Calls `ApplyIcon` with event-provided RUIDs and index.

</details>

---

### Related Types & Services
- **Component** – Base class for UI components.  
- **ButtonClickEvent** – Event fired when a button is clicked.  
- **SkillWindowComponent** – Handles opening/closing the skill window UI.  
- **_UserService** – Provides access to the local player.  
- **SkillUsedEvent** – Event for signaling skill usage.  
- **SpriteGUIRendererComponent** – Renders UI images for icons.  
- **ButtonComponent** – UI button component handling click states.  
- **_T** – Temporary table for runtime state storage.  
---

## SkillSelected

**Purpose**  
Represents an event that is fired when a player selects a skill in the UI or gameplay.

---

### Properties

| Property    | Type   | Purpose |
|-------------|--------|---------|
| `skillId`   | string | The unique identifier of the selected skill. |

---

### Usage

This event can be sent by a skill UI component or input handler to notify other systems that a specific skill has been chosen. Systems listening for `SkillSelected` can respond by updating UI, preparing skill execution, or triggering cooldowns.

---

### Related Types & Services
- **EventType** – Base class for defining custom events in MapleStory Worlds.  
- **SkillUsedEvent** – Can be used in conjunction to signal that a skill is actively being executed.  
- **_UserService** – Provides access to the local player for sending or handling events.  
---

## SkillUsedEvent

**Purpose**  
Represents an event that is fired when a player actively uses a skill in gameplay.

---

### Properties

| Property    | Type    | Purpose |
|-------------|---------|---------|
| `skillId`   | integer | The index or identifier of the skill that was used. |

---

### Usage

This event is typically sent by the player's skill system or UI component when a skill is executed. Other systems, such as cooldown management, visual effects, or combat calculations, can subscribe to this event to react accordingly.

---

### Related Types & Services
- **EventType** – Base class for defining custom events in MapleStory Worlds.  
- **SkillSelected** – Event that signals a skill has been chosen, which may precede `SkillUsedEvent`.  
- **_UserService** – Provides access to the local player for sending or receiving events.  
---

## SkillWindowComponent

**Purpose**  
Manages the player’s skill selection window UI, allowing the player to view available skills, select a skill, and confirm or cancel their choice.

---

### Properties

| Property             | Type    | Purpose |
|----------------------|---------|---------|
| `skillMenu`          | Entity  | Root UI entity for the skill menu. |
| `buttonConfirm`      | Entity  | Confirm button for selecting a skill. |
| `buttonCancel`       | Entity  | Cancel button to close the menu without selection. |
| `selectedSkillId`    | string  | Currently selected skill ID. |
| `skills`             | table   | Stores the skill buttons/UI elements. |
| `skillButtonModel`   | string  | Model ID used to spawn skill buttons. |
| `spawnedSkills`      | integer | Counter for instantiated skill buttons. |

---

### Method Summary

| Method                  | Execution Scope | Purpose |
|-------------------------|----------------|---------|
| `OnBeginPlay`           | ClientOnly      | Initializes the skill window and connects UI buttons. |
| `Show`                  | Client/Server   | Shows or hides the skill window. |
| `ApplySkill`            | Client/Server   | Spawns a skill button with an icon and connects its click event. |
| `OnConfirm`             | Client/Server   | Confirms the selected skill and sends a `SkillSelected` event. |
| `OnCancel`              | Client/Server   | Cancels the selection and hides the skill menu. |
| `SelectSkill`           | Client/Server   | Sets the currently selected skill ID. |
| `SkillInventoryUpdatedEvent` | ClientOnly / LocalPlayer | Updates the skill window when new skills are added to inventory. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes the skill window and connects the confirm and cancel buttons.

**Behavior**  
1. Resets `selectedSkillId` to an empty string.  
2. Connects `buttonConfirm` click event to `OnConfirm`.  
3. Connects `buttonCancel` click event to `OnCancel`.

</details>

<details>
<summary>Show</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Shows or hides the skill menu UI.

**Parameters**  
- `enabled` *(boolean)* – Whether to show (`true`) or hide (`false`) the menu.

**Behavior**  
- Resets `selectedSkillId` to empty and sets `skillMenu` enabled state.

</details>

<details>
<summary>ApplySkill</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Adds a skill button to the UI with the specified icon and connects its click behavior.

**Parameters**  
- `iconRUID` *(string)* – Icon RUID for the skill button.  
- `skillId` *(string)* – Skill identifier.

**Behavior**  
1. Increments `spawnedSkills` counter.  
2. Spawns a skill button under the `SkillScroll` container.  
3. Sets the icon image of the button.  
4. Connects the button click to `SelectSkill(skillId)`.

</details>

<details>
<summary>OnConfirm</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Confirms the currently selected skill and sends a `SkillSelected` event to the local player.

**Behavior**  
1. Shows a message if no skill is selected.  
2. Sends `SkillSelected` event with `selectedSkillId`.  
3. Hides the skill menu.

</details>

<details>
<summary>OnCancel</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Cancels skill selection and hides the skill menu.

</details>

<details>
<summary>SelectSkill</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sets the currently selected skill ID.

**Parameters**  
- `skillId` *(string)* – Skill to select.

</details>

<details>
<summary>SkillInventoryUpdatedEvent</summary>

**Execution Scope**: `ClientOnly`  
**Event Sender**: `LocalPlayer`  

**Purpose**  
Handles updates to the player’s skill inventory by adding new skills to the skill window.

**Parameters**  
- `event.iconRUID` *(string)* – Icon for the new skill.  
- `event.skillId` *(string)* – ID of the new skill.

**Behavior**  
- Calls `ApplySkill` to spawn and display the new skill button.

</details>

---

### Related Types & Services
- **Component** – Base class for entity components.  
- **_SpawnService** – Used to spawn UI skill buttons.  
- **_UIToast** – Displays popup messages to the player.  
- **SkillSelected** – Event sent when a skill is chosen.  
- **ButtonClickEvent** – UI button click event type.  
- **_UserService** – Provides access to the local player entity.  
- **Entity** – Represents UI elements in the scene.
---

## StatWindowUIComponent

**Purpose**  
Manages the player’s stat window UI, allowing the display, update, and upgrading of player stats.

---

### Properties

| Property           | Type    | Purpose |
|-------------------|---------|---------|
| `stats`            | table   | Stores UI elements for each stat. |
| `increaseValue`    | integer | Amount to increase a stat when upgrading. |

---

### Method Summary

| Method                   | Execution Scope | Purpose |
|--------------------------|----------------|---------|
| `OnBeginPlay`            | ClientOnly      | Initializes the stat window and connects the close button. |
| `Init`                   | Client/Server   | Populates the stat window with current player stats and sets click listeners. |
| `Open`                   | Client/Server   | Opens the stat window. |
| `Close`                  | Client/Server   | Closes the stat window. |
| `OnUpgradeStat`          | Client/Server   | Sends a `PlayerStatIncreased` event when the player upgrades a stat. |
| `UpdateStat`             | Client          | Updates the displayed value of a specific stat. |
| `HandlePlayerStatsUpdate`| Client / LocalPlayer | Updates the UI when the player's stats are updated. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes the stat window and connects the close button.

**Behavior**  
1. Finds the `CloseButton` child and connects its click event to `Close`.  
2. Retrieves the local player stats and calls `Init` to populate the UI.

</details>

<details>
<summary>Init</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Populates the stat window UI with the current player stats and attaches click listeners to each stat.

**Parameters**  
- `stats` *(table)* – Table of player stat values indexed by `_PlayerStatsEnum`.

**Behavior**  
1. Finds the `StatWindow` and `PlayerStat` UI containers.  
2. Iterates through all stats from 1 to `_PlayerStatsEnum.MAX-1`.  
3. Sets the text of each stat UI element to the corresponding value.  
4. Adds a click listener to each stat UI element that calls `OnUpgradeStat(index)`.  
5. Stores the UI elements in `self.stats`.

</details>

<details>
<summary>Open</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Enables the stat window UI.

</details>

<details>
<summary>Close</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Disables the stat window UI.

</details>

<details>
<summary>OnUpgradeStat</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Sends a `PlayerStatIncreased` event to the local player when a stat is upgraded.

**Parameters**  
- `index` *(integer)* – Index of the stat being upgraded.

**Behavior**  
1. Creates a new `PlayerStatIncreased` event.  
2. Sets the event's `increaseIndex` and `increaseValue`.  
3. Sends the event from the local player.

</details>

<details>
<summary>UpdateStat</summary>

**Execution Scope**: `Client`  

**Purpose**  
Updates the displayed value of a specific stat in the UI.

**Parameters**  
- `index` *(integer)* – Index of the stat to update.  
- `value` *(number)* – New value to display.

**Behavior**  
- Calls `SetText` on the corresponding stat UI element with the new value.

</details>

<details>
<summary>HandlePlayerStatsUpdate</summary>

**Execution Scope**: `Client / LocalPlayer`  
**Event Sender**: `LocalPlayer`  

**Purpose**  
Handles updates from the server or local player and updates the UI with the latest stat values.

**Parameters**  
- `event.index` *(integer)* – Index of the stat updated.  
- `event.finalValue` *(number)* – Updated value of the stat.

**Behavior**  
- Calls `UpdateStat(event.index, event.finalValue)` to refresh the UI.

</details>

---

### Related Types & Services
- **Component** – Base class for entity components.  
- **_UserService** – Provides access to the local player entity.  
- **PlayerStatsUpdate** – Event sent when a player stat value changes.  
- **PlayerStatIncreased** – Event sent when the player upgrades a stat.  
- **ButtonClickEvent** – UI button click event type.  
- **StatsWindowUIItem** – UI element that displays a single stat value.  
- **Entity** – Represents UI elements in the scene.
---

## PlayerStatIncreased

**Purpose**  
Represents an event triggered when a player increases a stat, communicating which stat was increased and by how much.

---

### Properties

| Property           | Type    | Purpose |
|-------------------|---------|---------|
| `increaseIndex`    | number  | Index of the stat that was increased (corresponds to `_PlayerStatsEnum`). |
| `increaseValue`    | number  | The amount by which the stat was increased. |

---

### Related Types & Services
- **EventType** – Base class for all event definitions.  
- **_PlayerStatsEnum** – Enum providing indices for all player stats.
---

## StatsWindowUIItem

**Purpose**  
Represents a single UI element for displaying and interacting with a player stat in the stats window. Allows setting the displayed text and attaching a click callback.

---

### Properties

| Property           | Type    | Purpose |
|-------------------|---------|---------|
| `onClickCallback`  | any     | Stores the callback function to invoke when the item is clicked. |
| `button`           | Entity  | Reference to the button entity that triggers the click event. |
| `text`             | TextComponent | Reference to the text component displaying the stat value. |

---

### Method Summary

| Method        | Execution Scope | Purpose |
|---------------|----------------|---------|
| `SetListener` | Client/Server  | Sets a callback function to be invoked when the item is clicked. |
| `onClick`     | Client/Server  | Invokes the registered click callback if valid. |
| `OnBeginPlay` | ClientOnly     | Connects the `onClick` method to the button's click event. |
| `SetText`     | Client/Server  | Updates the text displayed in the UI item. |

---

### Methods

<details>
<summary>SetListener</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Registers a callback to be invoked when the stat UI item is clicked.

**Parameters**  
- `callback` *(any)* – Function to call on click.

**Behavior**  
Stores the callback in `self.onClickCallback`.

</details>

<details>
<summary>onClick</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Handles the button click event by invoking the stored callback if it exists.

**Behavior**  
- Checks if `self.onClickCallback` is valid.  
- Calls the callback function if valid.

</details>

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Connects the `onClick` method to the UI button's click event.

**Behavior**  
- Calls `ConnectEvent` on the `button` entity linking `ButtonClickEvent` to `self.onClick`.

</details>

<details>
<summary>SetText</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Updates the text displayed on the UI item.

**Parameters**  
- `text` *(string)* – New text to display.

**Behavior**  
- Sets `self.text.Text` to the provided string.

</details>

---

### Related Types & Services
- **Component** – Base class for Unity-style entity components.  
- **Entity** – Represents an in-game entity with components.  
- **TextComponent** – Handles text rendering in the UI.  
- **ButtonClickEvent** – Event triggered when a button is clicked.
---

## UIMyInfoSimple

**Purpose**  
Displays the local player's basic information in the UI, including name, HP bar, HP text, and level.

---

### Properties

| Property     | Type            | Purpose |
|-------------|-----------------|---------|
| `myName`    | TextComponent   | Displays the player's nickname. |
| `myHPText`  | TextComponent   | Displays the player's current HP in text form. |
| `myHP`      | SliderComponent | Visual slider representing the player's current HP. |
| `myLevel`   | TextComponent   | Displays the player's current level. |

---

### Method Summary

| Method       | Execution Scope | Purpose |
|--------------|----------------|---------|
| `OnBeginPlay`| ClientOnly      | Initializes references to the UI elements and local player components. |
| `OnUpdate`   | ClientOnly      | Updates the UI elements with the player's current HP, level, and name every frame. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Initializes references to the UI elements and local player components.

**Behavior**  
1. Retrieves the entity path of the component.  
2. Uses `_EntityService:GetEntityByPath` to get references to `myName`, `myHP`, `myHPText`, and `myLevel` components.  
3. Stores references to the local player's `PlayerComponent` and `PlayerInfoComponent` in `_T.pc` and `_T.pic`.

</details>

<details>
<summary>OnUpdate</summary>

**Execution Scope**: `ClientOnly`  

**Purpose**  
Updates the player's name, HP, HP text, and level in the UI every frame.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last update frame.

**Behavior**  
1. Returns early if `_UserService.LocalPlayer` is nil.  
2. Retrieves the local player's `PlayerComponent` and `PlayerInfoComponent`.  
3. Updates `myName.Text` with the player's nickname.  
4. Updates `myHP.Value` with the normalized HP (current HP / max HP).  
5. Updates `myHPText.Text` with the formatted HP string (`current/max`).  
6. Updates `myLevel.Text` with the player's current level prefixed by "LV.".

</details>

---

### Related Types & Services
- **Component** – Base class for Unity-style entity components.  
- **TextComponent** – Handles text rendering in the UI.  
- **SliderComponent** – Handles slider UI elements.  
- **_EntityService** – Provides access to in-game entities by path or ID.  
- **_UserService** – Provides access to the local player and user-related services.
---

## MonsterSpawn

**Purpose**  
Manages spawning of monsters within a defined range, maintaining a maximum number of active monsters and periodically respawning them at random positions.

---

### Properties

| Property         | Type    | Purpose |
|-----------------|---------|---------|
| `monsterID`      | string  | Model ID of the monster to spawn. |
| `spawnRange`     | Vector2 | X-coordinate range for random spawn positions. |
| `MaxSpawnCount`  | number  | Maximum number of monsters allowed simultaneously. |
| `spawnInterval`  | number  | Time interval (in seconds) between spawn attempts. |
| `Time`           | number  | Accumulated time for tracking spawn intervals. |
| `MonsterArray`   | table   | Stores references to spawned monster entities. |

---

### Method Summary

| Method             | Execution Scope | Purpose |
|-------------------|----------------|---------|
| `OnBeginPlay`      | ServerOnly     | Initializes the monster array at the start of the game. |
| `SpawnMonster`     | Client/Server  | Spawns a specified number of monsters at random positions within the spawn range. |
| `GetRandomPosition`| Client/Server  | Generates a list of random Vector3 positions within the defined spawn range. |
| `GetCurMonsterCount` | Client/Server | Returns the count of currently active monsters. |
| `OnUpdate`         | ServerOnly     | Updates the timer and triggers monster spawns when the interval is reached. |

---

### Methods

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `ServerOnly`  

**Purpose**  
Initializes the `MonsterArray` to an empty table at the start of the server simulation.

**Behavior**  
- Sets `self.MonsterArray = {}`.

</details>

<details>
<summary>SpawnMonster</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Spawns monsters at random positions and ensures existing monsters are respawned if disabled.

**Parameters**  
- `spawnCount` *(integer)* – Number of monsters to spawn.

**Behavior**  
1. Retrieves the parent entity for the current map.  
2. Generates `spawnCount` random positions using `GetRandomPosition`.  
3. Iterates through each position:  
   - If the corresponding monster is invalid, spawns a new monster using `_SpawnService:SpawnByModelId`.  
   - If the monster exists but is disabled, calls its `Respawn` method.  
4. Plays a spawn effect at each position using `_EffectService:PlayEffect`.

</details>

<details>
<summary>GetRandomPosition</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Generates a list of random spawn positions within the defined `spawnRange`.

**Parameters**  
- `spawnCount` *(integer)* – Number of positions to generate.

**Returns**  
- `positions` *(table)* – List of `Vector3` positions.

**Behavior**  
1. Determines left and right boundaries from `spawnRange`.  
2. For each spawn, generates a random X-coordinate within the range.  
3. Sets Y-coordinate to 0 and Z-coordinate to 0.  
4. Returns the list of generated positions.

</details>

<details>
<summary>GetCurMonsterCount</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Counts the number of currently active monsters in `MonsterArray`.

**Returns**  
- `count` *(integer)* – Number of active monsters.

**Behavior**  
1. Iterates through `MonsterArray`.  
2. Increments `count` for each valid and enabled monster.  
3. Returns the total count.

</details>

<details>
<summary>OnUpdate</summary>

**Execution Scope**: `ServerOnly`  

**Purpose**  
Handles timed spawning of monsters based on `spawnInterval`.

**Parameters**  
- `delta` *(number)* – Time elapsed since the last update.

**Behavior**  
1. Increments `self.Time` by `delta`.  
2. If `self.Time >= spawnInterval`:  
   - Resets `self.Time = 0`.  
   - Calculates `spawnCount` as `MaxSpawnCount - GetCurMonsterCount()`.  
   - Calls `SpawnMonster(spawnCount)` if `spawnCount > 0`.

</details>

---

### Related Types & Services
- **Component** – Base class for Unity-style entity components.  
- **Vector2 / Vector3** – Represent 2D and 3D positions.  
- **_EntityService** – Access to entities by path.  
- **_SpawnService** – Handles spawning entities in the world.  
- **_EffectService** – Plays visual effects at positions.  
- **_UtilLogic** – Provides utility functions, e.g., random number generation.
---

## DebugLogic

**Purpose**  
Provides debug and utility functions for testing player stats, skill UI, and level-up functionality, with input-based triggers.

---

### Properties

| Property   | Type    | Purpose |
|------------|---------|---------|
| `uiPath`   | string  | Path to the UI parent entity where debug windows will be spawned. |

---

### Method Summary

| Method               | Execution Scope | Purpose |
|---------------------|----------------|---------|
| `OpenStatWindow`     | Client/Server  | Opens the player stat window UI. |
| `OnBeginPlay`        | Client/Server  | Initializes and spawns debug UI windows on the client. |
| `LevelUpPlayer`      | Server         | Levels up a player by their user ID. |
| `HandleKeyDownEvent` | Server         | Handles key press events (F1/F2) to trigger debug actions. |

---

### Methods

<details>
<summary>OpenStatWindow</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Opens the player's stat window UI.

**Behavior**  
- Calls `Open()` on the `StatWindowUIComponent` stored in `_T.playerStatWindow`.

</details>

<details>
<summary>OnBeginPlay</summary>

**Execution Scope**: `Client/Server`  

**Purpose**  
Initializes debug UI components when the game begins on the client.

**Behavior**  
1. Checks if the logic is running on a client.  
2. Retrieves the UI parent entity using `_EntityService:GetEntityByPath(uiPath)`.  
3. Spawns the Player Stat Window model and stores its `StatWindowUIComponent` in `_T.playerStatWindow`.  
4. Spawns the Player Skill Window model and stores its `SkillWindowComponent` in `_T.playerSkillWindow`.

</details>

<details>
<summary>LevelUpPlayer</summary>

**Execution Scope**: `Server`  

**Purpose**  
Levels up a specific player using their user ID.

**Parameters**  
- `id` *(string)* – User ID of the player to level up.

**Behavior**  
- Retrieves the player entity using `_UserService:GetUserEntityByUserId(id)`.  
- Calls `LevelUp()` on the player's `PlayerInfoComponent`.

</details>

<details>
<summary>HandleKeyDownEvent</summary>

**Execution Scope**: `Server`  

**Purpose**  
Handles key input events to trigger debug actions such as opening the stat window or leveling up the player.

**Parameters**  
- `event` *(KeyDownEvent)* – Event data containing the key pressed.

**Behavior**  
1. Checks the `event.key` value.  
2. If `F1` is pressed, calls `OpenStatWindow()`.  
3. If `F2` is pressed, calls `LevelUpPlayer()` for the local player's user ID.

**Event Metadata**  
- **Event Sender**: `InputService`  
- **Sender Space**: Client

</details>

---

### Related Types & Services
- **Logic** – Base class for game logic scripts.  
- **_EntityService** – Provides access to entities by path.  
- **_SpawnService** – Spawns entities into the world.  
- **_UserService** – Provides access to player entities and user IDs.  
- **_T** – Temporary table for storing runtime references.  
- **KeyboardKey** – Enum of keyboard keys.  
- **PlayerInfoComponent** – Handles player level and stats.  
- **StatWindowUIComponent / SkillWindowComponent** – UI components for stats and skills.  
---


## Changelog

| Version | Changelog | Download |
|---------|-----------|----------|
| V1.0 | Initial release of Player Stats & Skills System | [Download](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CombatDemo/CombatDemo_V1.modpackage) |
| V1.1 | UI Asset Fixes. Monster Spawner Foothold Tracking Addition. | [Download](https://github.com/MapleStory-Worlds-Global/Livestream-Projects/raw/refs/heads/main/CombatDemo/CombatDemo.modpackage) |