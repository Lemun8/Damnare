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

##  System Design
Other than individual scripts, the game relies on several interconnected systems to handle several mechanics. Below is an overview of how the core mechanics are engineered.
#### 1. Animation-Driven Hit Detection
Instead of decoupling logic from visuals, the combat uses **Animation Events** to trigger hitbox calculations exactly when the weapon swings visually.
*   **How it works:** We set an Animation Event in each Attacking Animation Clips, which when triggered will call the `Attack()` function in the `PlayerCombatManager.cs` and start the attacking logic in the `PlayerAttackManager.cs` to activate a `Physics.BoxCast`.
*   **Combat Assist:** To improve game feel, the system detects if an enemy is slightly out of range during the wind-up and applies a forward "Drift" force as well as rotating them towards the enemy, sliding the player into effective range automatically.

#### 2. Animator Controller as AI State Machine
The Enemy AI logic in `EnemyActionsManager.cs` offloads state management to the **Unity Animator Controller**, effectively using it as a Finite State Machine (FSM).

*   **Logic Flow:** The script calculates parameters like distance and timing, to decide the states in the Animator.
*   **Decision Making:** The Animator transitions between **Idle**, **Walk**, and **Attack** states based on those paramaters.
*   **Randomization:** When entering the "Attack" state, the AI randomly select different attack patterns, keeping the combat unpredictable.
