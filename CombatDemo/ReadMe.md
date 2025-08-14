# Player Stats & Skills System Documentation

## Youtube Livestream
[![How to Make a Character in MapleStory Worlds](http://img.youtube.com/vi/3t_HcSKQ7M0/0.jpg)](https://www.youtube.com/live/3t_HcSKQ7M0?feature=shared "How to Make a Character in MapleStory Worlds")
[![How to Create a Character PT 2 w/ Ursus](http://img.youtube.com/vi/98-aFbh2pOg/0.jpg)](https://www.youtube.com/live/98-aFbh2pOg?feature=shared "How to Create a Character PT 2 w/ Ursus")

## Mod Package Installation Instructions
[Mod File Installation Guide](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects?tab=readme-ov-file#project-import-instructions)

## Project File
[Download Latest Project File](https://github.com/MapleStory-Worlds-Global/YoutubeTutorialProjects/raw/refs/heads/main/Zakums_Curse/Zakums_Curse_V1_1.mod)


## Table of Contents
- [Overview](#overview)
- [Setup & Installation](#setup-and-installation)
- [UI Components](#ui-components)
  - [UIMyInfoSimple](#uimyinfosimple)
  - [SkillUIComponent](#skilluicomponent)
  - [SkillWindowComponent](#skillwindowcomponent)
  - [StatWindowUIComponent](#statwindowuicomponent)
  - [StatsWindowUIItem](#statswindowuiitem)
- [Stats Systems](#stats-systems)
  - [PlayerStatComponent](#playerstatcomponent)
  - [CharacterStats](#characterstats)
  - [PlayerStats](#playerstats)
  - [MonsterStats](#monsterstats)
  - [PlayerStatsEnum](#playerstatsenum)
- [Skill Systems](#skill-systems)
  - [SkillsEnum](#skillsenum)
  - [PlayerSkillComponent](#playerskillcomponent)
  - [SkillFactory](#skillfactory)
  - [Skill (Base Class)](#skill-base-class)
  - [Skill Subclasses](#skill-subclasses)
- [Animation](#animation)
  - [PlayerAnimationComponent](#playeranimationcomponent)
- [Events](#events)
  - [SkillSelected](#skillselected)
  - [SkillUsedEvent](#skillusedevent)
  - [PlayerStatIncreased](#playerstatincreased)
  - [PlayerStatsUpdate](#playerstatsupdate)
- [Changelog & Downloads](#changelog)
---


## Overview
This documentation covers the Player Stats, Skills, Animation, and UI systems used in the game. Each section describes the core components, classes, and their interactions to assist developers in understanding and extending the functionality.

---
## Setup and Installation
* Update the uiPath variable under Sample>DebugLogic Logic script file with the correct path for your world by right clicking on the desired map under Heirarchy>maps>CombatDemo and selecting "Copy Entity Path" then pasting it to the uiPath variable. 
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
* The DebugLogic script file will spawn the necessary UI entities for the demo. It also contains debug input events that binds F1 to open the stat window, and F2 to trigger the player to level up. 
---

## UI Components

### UIMyInfoSimple
**Purpose**: Displays player nickname, level, and HP.

**Properties**
- `myName`: UI text element that shows the player’s name.
- `myHP`: Slider that represents current HP.
- `myLevel`: UI text element that shows the player’s level.

**Methods**
- `OnBeginPlay()`: Binds UI fields to logic.
- `OnUpdate(delta)`: Continuously updates player data like HP and level.

---

### SkillUIComponent
**Purpose**: Displays and manages active skill slots and allows player to trigger skills.

**Properties**
- `emptySlotRUID`: Resource ID for creating empty skill slots.
- `skillSlotParent`: Parent transform to hold the skill slot instances.

**Methods**
- `OnBeginPlay()`: Binds skill slot UI and button click handlers.
- `OnApplySkillSlot(index)`: Opens the skill selection window for a slot.
- `OnTriggerSkill(index)`: Triggers the equipped skill in the given slot.
- `ApplyIcon(...)`: Assigns icon states to each skill slot.
- `ResetIcon(index)`: Clears icon visuals.
- `EnterCooldown(index)`: Begins cooldown animation.

---

### SkillWindowComponent
**Purpose**: Displays a UI menu for players to choose which skills to equip.

**Properties**
- `skillMenu`: The root entity of the skill selection menu.
- `buttonConfirm`: Confirm selection button.
- `buttonCancel`: Cancel selection button.
- `selectedSkillId`: Stores the selected skill ID.
- `skills`: Table storing available skills.
- `skillButtonModel`: Model ID for instantiating skill buttons.
- `spawnedSkills`: Number of spawned skill buttons.

**Methods**
- `OnBeginPlay()`: Initializes confirm/cancel buttons and state.
- `Show(enabled)`: Enables/disables the skill window.
- `ApplySkill(skill)`: Instantiates and binds a skill to a UI button.
- `OnConfirm()`: Sends `SkillSelected` event and closes the menu.
- `OnCancel()`: Cancels selection and hides the menu.
- `SelectSkill(skillId)`: Sets selected skill ID internally and logs it.
---

### StatWindowUIComponent
**Purpose**: Displays player stats and allows them to be upgraded.

**Properties**
- `stats`: Table mapping stat indices to UI elements.
- `increaseValue`: Amount by which a stat is increased when upgraded.

**Methods**
- `OnBeginPlay()`: Connects UI close button.
- `Init(stats)`: Binds each stat UI item to its upgrade event.
- `Open()`, `Close()`: Opens/closes the stat window.
- `OnUpgradeStat(index)`: Sends a `PlayerStatIncreased` event with the specified index and value.
- `UpdateStat(index, value)`: Updates a stat's displayed value.

**Event Handlers**:
- `HandlePlayerStatsUpdate(PlayerStatsUpdate)`: Updates the UI based on the received index and new value.


---

### StatsWindowUIItem
**Purpose**: Represents a single stat in the stat window UI.

**Properties**
- `onClickCallback`: Function to call when this item is clicked.
- `button`: Button element for this stat item.
- `text`: Label displaying the stat value or name.

**Methods**
- `SetListener(callback)`: Assigns the click callback.
- `onClick()`: Executes assigned callback when button is pressed.
- `OnBeginPlay()`: Binds button to click event.
- `SetText(text)`: Updates the stat's UI text display.

---

## Stats Systems

### PlayerInfoComponent

**Purpose**:  
Manages player stats, level, HP/MP, and synchronizes stat changes between server and client, while updating the UI accordingly.

**Properties**  
- `stats`: Holds the `PlayerStats` instance representing the player's current stats.  
- `hp`: Player's current health points.  
- `mp`: Player's current mana points.  
- `level`: Player's level, synchronized between server and client.  
- `statsWin`: Reference to the `StatWindowUIComponent` responsible for displaying player stats in the UI.

**Methods**  
- `OnBeginPlay()`: Initializes player stats on the server, sets initial HP, and initializes the client UI.  
- `InitClient(stats)`: Sends player stats data to the UI component for initialization.  
- `GetAttackPower()`: Returns the player's calculated attack power by calling the `CalcDamage()` method on `PlayerStats`.  
- `SetHp(hp)`: Sets both current and max HP on the player component.  
- `LevelUp()`: Increases player level and stats, updates HP, triggers a visual effect, and logs the new level.  
- `IncreaseStats(index, value)`: Increases a specific stat by the given value and sends an update event to the client UI.  
- `SendStatUpdateEvt(index, finalValue)`: Sends a `PlayerStatsUpdate` event notifying the UI of stat changes.

**Event Handlers**  
- `HandlePlayerStatIncreased(event)`: Handles incoming stat increase events by updating the corresponding stat and notifying clients.

Ask ChatGPT

---

## Event Handlers

- `HandlePlayerStatIncreased(PlayerStatIncreased event)` (Self)  
  Handles incoming stat increase events by extracting the stat index and value, then calls `IncreaseStats` to update stats and notify clients.

---

### CharacterStats  
**Purpose**: Defines the fundamental stats and basic damage calculations common to all characters (players and monsters).

**Properties**  
- `hp`: Current health points.  
- `defense`: Defense stat.  
- `level`: Character level.  
- `power`: Power stat used for damage calculation.  
- `mp`: Mana points.  
- `finalDefense`: Calculated effective defense after modifiers.

**Methods**  
- `GetPower(defender)`: Returns the character’s power value. By default, returns `self.power`.  
- `GetDefense()`: Returns the final defense value.  
- `GetHP()`: Returns current HP.  
- `Init(context)`: Initialization logic (empty in base).  
- `GetDamage(level, powerValue)`: Returns damage value based on level and power. Default returns the input power value.

---

### PlayerStats  
**Purpose**: Manages player-specific stats, including base stats, added stats, percentages, mastery, and level progression.

**Properties**  
- `addedStats`: Additional stats gained from equipment or buffs.  
- `jobClassValue`: Combined value representing the player’s job/class stat strength.  
- `baseStats`: Base player stats before additions.  
- `mastery`: Mastery stat influencing damage range.  
- `percentageStats`: Percentage modifiers applied to stats.  
- `finalStats`: Final computed stats after all calculations.  
- `critChance`: Player critical hit chance.  
- `addedDefense`: Additional defense beyond base.

**Methods**  
- `IncreaseStat(type, value)`: Increases the base stat of a given type and recalculates dependent values.  
- `Init(context)`: Initializes player stats from player info component, sets base stats, and calculates derived values.  
- `SetToBaseStats()`: Sets default base stats, mastery, defense, and clears additions and percentages.  
- `GetUpperDamageRange(defender)`: Calculates the upper damage range using level advantage and job class value.  
- `GetLowerDamageRange(defender, upperRange)`: Calculates the lower damage range factoring mastery.  
- `CalculateClassValue()`: Calculates `jobClassValue` based on primary (STR) and secondary (DEX) stats.  
- `GetStat(type)`: Returns the final computed stat for a given type, factoring base, added, percentage, and final stats.  
- `GetLevelAdvantageMultiplier(level)`: Returns a multiplier based on level difference.  
- `SetLevelAdvantagePcts()`: Initializes an internal lookup table for level advantage multipliers.  
- `GetPower(defender)`: Returns a randomized power value between lower and upper damage ranges.  
- `GainHP()`: Returns HP gained on level up (random range).  
- `GainMP()`: Returns MP gained on level up (random range).  
- `LevelUp()`: Increments level, increases HP and MP.  
- `CalculateDefense()`: Calculates and logs `finalDefense` based on STR, DEX, LUK, and added defense.  
- `GetUpperDefenseRange(level, powerValue)`: Calculates upper defense-adjusted damage range.  
- `GetLowerDefenseRange(level, powerValue)`: Calculates lower defense-adjusted damage range.  
- `LevelDefenseAdvantageMultiplierA(level)`: Returns multiplier A based on level difference for defense calculation.  
- `LevelDefenseAdvantageMultiplierB(level)`: Returns multiplier B based on level difference for defense calculation.  
- `GetDamage(level, powerValue)`: Returns damage adjusted by defense ranges.  
- `GetStats()`: Returns a table of final computed stats for all stat types.
---

### MonsterStats  
**Purpose**: Manages monster-specific stats, including level-based HP calculation with scaling multipliers.

**Properties**  
- Inherits all properties from `CharacterStats`.

**Methods**  
- `Init(context)`: Initializes monster stats from the monster component and calculates HP.  
- `CalculateHP()`: Calculates and sets HP based on baseline and level multipliers.  
- `GetBaselineHP()`: Returns base HP depending on the monster’s level using predefined level ranges.  
- `GetLevelMultipliers()`: Returns a multiplier applied to HP based on level brackets.

---

### PlayerStatsEnum
**Purpose**: Defines the indexes used in the `Stats` table in `PlayerStats`.

**Values**
- `HP = 1`
- `STR = 2`
- `DEF = 3`
- `DEX = 4`
- `LUK = 5`
These indexes map directly to the internal stats table used for upgrades and display.

---

## Skill Systems

### SkillsEnum
**Purpose**: Enumerator for skill string identifiers.

**Values**
- `HERO_RAGINGBLOW = "hero_ragingblow"`
- `HERO_COMBOFORCE = "hero_comboforce"`
- `HERO_INCISING = "hero_incising"`
- `HERO_WARRIORLEAP = "hero_warriorleap"`

---

### PlayerSkillComponent
**Purpose**: Loads, equips, and triggers skills for a player.

**Properties**
- `skills`: Table of all available skill instances.
- `selectedSkills`: Table of equipped skills.
- `skillIndex`: Current index used when equipping new skills.
- `commonEntity`: Shared entity reference for accessing components.

**Methods**
- `LoadSkills()`: Loads skills using SkillFactory.
- `OnBeginPlay()`: Initializes and prepares the component.
- `EquipSkill(skillId)`: Equips a skill to a slot.
- `UseSkill(skillIndex)`: Executes the skill in the given index.
- `UpdateSkillWindow(skill)`: Notifies UI of equipped skill.
- `ClientUseSkill(skill)`: Triggers client-side effects.
- `OnSkillComplete(player)`: Called after skill execution ends.

**Events**
- `HandleSkillUsedEvent(SkillUsedEvent)`: Called when player uses skill icon.
- `HandleSkillSelected(SkillSelected)`: Called when skill is selected in UI.

---

### SkillFactory
**Purpose**: Creates skill instances based on provided data.

**Methods**
- `CreateSkillWithData(data)`: Returns the appropriate skill subclass.

---

### Skill (Base Class)
**Purpose**: Base logic for all skills.

**Methods**
- `Init(callback)`: Initializes the skill with a callback.
- `OnTrigger(player)`: Executes skill on server.
- `ClientTrigger(player)`: Executes skill visuals on client.

---

### Skill Subclasses
- `RagingBlow`
- `ComboForce`
- `Incising`
- `WarriorLeap`

Each subclass overrides `OnTrigger` and/or `ClientTrigger` for custom behavior.

---

#### ComboForce
**Purpose**: Pulls enemies in front of the player while hitting them multiple times.

- Plays activation effects and animation
- Calls AttackNormal() using sprite hitbox data
- Pulls hit enemies toward the player with tweening
- Re-enables controller after execution

---

#### RagingBlow
**Purpose**: A strong, multi-hit attack.
- Very similar to Incising
- Differentiated by animation, timing, and hit count


---

#### Incising
**Purpose**: Hits enemies multiple times in a wide arc.
- Plays animation and visual/audio effects
- Calculates hit area using sprite data
- Deals repeated damage to nearby enemies

---

#### WarriorLeap
**Purpose**: Allows a second jump when airborne.
- Overrides the `ClientTrigger()` to ensure the leap is peformed client side. 
- Checks if the player is off the ground
- If holding the up key, launches vertically with different animation
- Applies force using the RigidbodyComponent

---

## Animation

### PlayerAnimationComponent
**Purpose**: Manages player animation playback.

**Methods**
- `PlayImmediatly(actionName, playRate, startFrame, endFrame)`: Plays a specified animation clip immediately on the client by creating and sending an `ActionStateChangedEvent` to the player’s avatar body.
- `OnBeginPlay()`: Initializes a reusable instance of ActionStateChangedEvent and stores it in the internal _T.tempEvent table for repeated use.

---

## Events

### SkillSelected
Sent when a player picks a skill to apply in the UI.

**Fields**
- `skillId`: The ID of the selected skill.

---

### SkillUsedEvent
Triggered when a player uses/triggers a skill from UI.

**Fields**
- `skillId`: Which skill was used.

---

### PlayerStatIncreased
Triggered when a stat value is increased. 

**Fields**
- `increaseIndex`: Index of the stat to increase. 
- `increaseValue`: Amount to increase.

---

### PlayerStatsUpdate
Triggered when player stats are updated.

**Fields**
- `stats`: Updated stat values (HP, STR, etc.)

---

## Summary Notes
- All skill data and visual RUIDs must be defined correctly in advance.
- PlayerStatComponent and PlayerSkillComponent are the main synced components controlling backend logic.
- Stats are organized through a base class hierarchy for reuse across characters and monsters.
- UI and game logic communicate through clearly defined event types.
- Each skill follows a consistent base class format for easy expansion.
- Player animation integration is optional but highly recommended for polish.
---

## Changelog

| Version | Changelog | Download |
|---------|-----------|----------|
| V1.0 | Initial release of Player Stats & Skills System | Coming Soon |