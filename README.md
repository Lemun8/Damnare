# Project-Impact-Portfolio
The Code Design Documentation for the Project Impact

<img src="https://github.com/user-attachments/assets/1dac9800-41d3-4dfb-b0f1-e7abcf1a6b75"/>
<!-- About the game -->
<td width="70%" valign="top" style="padding:15px;">
  <h2>About </h2>
  <p style="max-width:700px;">
    Damnare is a turn based RPG with limb mechanics referenced from the game Fear and Hunger. The game takes in other RPG elements and mish mash them into 1 game. This game was solo developed by myself so all of the assets used are not mine. The game is still in demo/prototype. (Development duration 3 months+)
  </p>
  <a href="https://lemun8.itch.io/damnare-demo">
    <img src="https://img.shields.io/badge/Itch.io-FA5C5C?style=for-the-badge&logo=itch.io&logoColor=white" />
  </a>
</td>


##  Scripts and Features
Here are some of the main script that is used to manage the game.
<br>
**Core Combat System**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `CharacterCombat.cs` | Core character component managing all combat-related data and mechanics. Manages character stats (HP, Mind, AP, Agility, Luck, Evasion, Accuracy), Maintains body part system (limb-based combat), Handles equipment (weapons, armor, accessories per limb), etc. |
| `CombatManager.cs` | Orchestrates turn-based combat flow and manages all combat interactions. Combat state management (Setup, PlayerPlanning, Execution, Victory, Defeat), Turn order calculation based on Agility, Action execution (attacks, skills, items, healing, buffs, debuffs), etc. |
| `BodyPart.cs`  | Represents individual body parts in the limb-based combat system. Individual HP tracking per limb, Equipment slots (weapon, armor, accessory), Limb-specific skills, etc. |
| `PlayerActionPlanner.cs`  | Manages player action selection and planning during combat. Guides player through action selection workflow, Manages UI state transitions (body part → action type → skill/item → target), Queues player actions for execution, etc. |
<br>

<br> **Player Systems Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `PlayerMind.cs` | Manages player's Mind resource in the overworld. Mind decay over time (configurable rate per minute), Persistence via PlayerGameState, Dynamic max Mind based on character class (Mage: 150, others: 100), etc. |
| `PlayerHunger.cs` | Implements hunger mechanics affecting player survival. Hunger decay over time, Hunger stage management (Normal, Hunger, Greater Hunger), HP penalty application based on hunger level, etc. |
| `PlayerGameState.cs`  | Static runtime state manager for player data persistence. Save/load player Mind and Hunger to PlayerPrefs, Character state initialization per class, Scene-to-scene data persistence, etc. |
<br>

<br> **Overworld & Scene Management**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `OverworldSceneManager.cs` | Manages overworld scene setup and player spawning. Player character instantiation based on selected class, Spawn point management, Player state application (Mind, Hunger, HP), etc. |
| `OverworldSkillExecutor.cs` | Executes skills outside of combat (healing). Skill execution in overworld context, Mind cost deduction, Healing skill application to all body parts, etc |
| `EnemyOverworldAI.cs` | Controls enemy behavior in the overworld. Patrol and chase AI states, Player detection, Combat initiation, etc. |
<br>

<br> **UI Systems Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `CombatUIManager.cs` | Renders and manages combat UI using OnGUI. Display action selection menus (body parts, skills, items, targets), Show player AP and Mind in real-time, Display status effects on player and enemies, etc. |
| `OverworldMenuManager.cs` | Full-featured pause menu for overworld gameplay. Tab-based navigation (Stats, Items, Equipment, Skills, Settings, Exit), Real-time stat display (HP, Mind, Hunger), Inventory management (use items), etc. |
| `DamagePopupManager.cs` | Displays floating damage/healing numbers and status text. Render damage numbers (red), Render healing numbers (green with + prefix), Show miss/evade messages, etc. |
| `NotificationManager.cs` | Displays temporary on-screen notifications. Queue-based notification system, Auto-dismiss after timeout, Visual feedback for game events (hunger stage changes, item usage, etc). |
<br>

<br> **Data Systems Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `SkillBase.cs` | Defines all skill data as ScriptableObjects. Skill name, description, AP and Mind costs, Damage range (min/max), etc. |
| `ItemData.cs` | Defines consumable items. Item name, description, AP cost to use, Heal amount, etc. |
| `EquipmentData.cs` | Defines weapons, armor, and accessories. Equipment name, type (Weapon, Armor, Accessory), Stat modifiers (ATK, MATK, DEF, MDEF), Limb slot assignment, etc. |
| `CharacterPrefabLibrary.cs` | Maps character classes to their prefabs. Centralized character prefab registry, Used by scene managers to instantiate correct character, Supports character selection flow. |
<br>

<br> **Inventory System Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `Inventory.cs` | Manages player inventory with items and equipment. Add/remove items with quantity tracking, Stacking system, Slot-based storage, Query inventory for specific items, Support for both ItemData and EquipmentData. |
<br>

<br> **Supporting Systems Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `StatusEffect.cs` | Represents active buffs, debuffs, and DOTs. Effect type (buff/debuff), Stat modifiers (ATK, MATK, DEF, MDEF, AGI), DOT type (Bleed, Burn, Poison), etc. |
| `EquipmentSlot.cs` | Container for equipped items on body parts. Hold reference to equipped item, Slot type identification, Equip/unequip operations, etc. |
| `TurnAction.cs` | Represents a queued combat action. Actor (who performs the action), Skill or Item to use, Target character and body part, etc. |
<br>

##  System Design
Other than individual scripts, the game relies on several interconnected systems to handle several mechanics. Below is an overview of how the core mechanics are engineered.
#### 1. Limb-Based Combat Architecture
Instead of treating characters as single HP entities, the combat system implements a granular body part system where each limb is an independent combat unit.
*   **How it works:** Each CharacterCombat contains a list of BodyPart components. Every body part maintains its own HP, equipment slots (weapon/armor/accessory), and skill list. During combat, players select which limb to use for actions and which enemy limb to target.
*   **Blackout Mechanic:** When a body part's HP reaches zero, it enters a "blackout" state—unable to perform actions or equip items until healed. This creates tactical depth: targeting enemy weapon arms disables their attacks, while protecting your own limbs becomes critical.
*   **Equipment Integration:** Each BodyPart has three EquipmentSlot instances. Stat calculations in CharacterCombat.ResolveStat() aggregate base stats + equipment bonuses + active status effects, allowing per-limb stat modifiers to affect overall combat effectiveness.
  
#### 2. Animator Controller as AI State Machine
The Enemy AI logic in `EnemyActionsManager.cs` offloads state management to the **Unity Animator Controller**, effectively using it as a Finite State Machine (FSM).

*   **Logic Flow:** The script calculates parameters like distance and timing, to decide the states in the Animator.
*   **Decision Making:** The Animator transitions between **Idle**, **Walk**, and **Attack** states based on those paramaters.
*   **Randomization:** When entering the "Attack" state, the AI randomly select different attack patterns, keeping the combat unpredictable.
