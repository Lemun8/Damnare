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
| `CharacterCombat.cs` | Core character component managing all combat-related data and mechanics. Manages character stats (HP, Mind, AP, Agility, Luck, Evasion, Accuracy), Maintains body part system (limb-based combat), Handles equipment (weapons, armor, accessories per limb), Applies item effects (healing, buffs, mind restoration), Manages status effects (buffs, debuffs, DOTs), Implements hunger system integration (HP clamping), Stat resolution with equipment and effect modifiers. |
| `CombatManager.cs` | Orchestrates turn-based combat flow and manages all combat interactions. Combat state management (Setup, PlayerPlanning, Execution, Victory, Defeat), Turn order calculation based on Agility, Action execution (attacks, skills, items, healing, buffs, debuffs), Combat flow control and turn progression, Enemy AI decision-making, Victory/defeat detection and scene transitions. |
| `BodyPart.cs`  | Represents individual body parts in the limb-based combat system. Individual HP tracking per limb, Equipment slots, Limb-specific skills, Blackout state, Functional status tracking, Stat modifiers from equipment. |
| `PlayerActionPlanner.cs`  | Manages player action selection and planning during combat. Guides player through action selection workflow, Manages UI state transitions (body part → action type → skill/item → target), Queues player actions for execution,Interfaces with CombatUIManager for visual feedback |
| `MainMenuManager.cs`  | Responsible for managing the Main Menu, to allow the player to start the game and open settings menu in the game |
| `ModeSelection.cs`  | Responsible for Managing which Mode is currently selected in the UI so that the MainMenuManager can Start them |
| `InGameMenuManager.cs`  | Responsible for Managing the UI, such as Pause Menu, Winning and Losing screen, etc. of the SinglPlayer mode |
| `MultiplayerInGameMenu_Manager.cs`  | The same as InGameMenuManager.cs, but instead for Managing the UI of the Multiplayer mode |
| `InGameMenuManager.cs`  | Responsible for Managing the UI, such as Pause Menu, Winning and Losing screen, etc. of the SinglPlayer mode |
| `MultiplayerInGameMenu_Manager.cs`  | The same as InGameMenuManager.cs, but instead for Managing the UI of the Multiplayer mode |
<br>

<br> **The Player Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `PlayerAnimationManager.cs` | Responsible for Changing the current State of the player animator, from idling, attacking, enhanced, ulting, etc. by referencing  |
| `PlayerAttackManager.cs` | Responsible for perfroming the Attacks of the player, such as activating hitbox, doing combo, dealing damage, and spawning hit vfx |
| `PlayerAttributesManager.cs`  | Responsible for containing the stats of the player, and managing player's stats UI |
| `PlayerCombatManager .cs`  | Responsible for containing the stats (damage multiplier from ATK stats, attack timing, current skill charge and ultimate energy, current combo stage, etc.) of the player's attacks, calling the Attack in PlayerAttackManager.cs, and deciding the state of the player Animation in PlayerAnimationManager.cs |
| `PlayerEntity.cs`  | Responsible for storing the reference to all the player scripts so that it is easily accessible, and performing death process (calling animation, disabling input, and collider, etc.) |
| `PlayerMovement.cs`  | Responsible for Managing the movement of the player, such as dashing and walking, as well as setting the state of the player animation in PlayerAnimationManager.cs |
| `PlayerStateManager.cs`  | Responsible for storing the current player's state, and to be referenced by the PlayerAnimationManager.cs to decide which state the player is currently in |
| `PlayerAudio.cs`  | Responsible for storing the player's sound clips, and playing them by calling the AudioManager.cs |
| `PlayerProjectile.cs`  | Responsible for the projectiles created by player when in enhanced mode |
| `ChargeIndicator.cs and DashTrail.cs`  | ChargeIndicator.cs is responsible for the vfx in UI, and DashTrail.cs is responsible for the dash trail effect |
<br>

<br> **The Enemy Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `EnemyActions.cs` | Responsible for storing the all the functions for the enemy's actions, such as walk and attacks, as well as their parameters like damage, hitbox size, etc |
| `EnemActionsManager.cs` | Responsible for calling the EnemyActions, and changing the state of the Animator, from idling, to walking, to attacking |
| `EnemyAttributesManager.cs` | Responsible for containing the stats of the enemy, and managing enemy's stats UI |
<br>

<br> **The Multiplayer Script**:
|  Script       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `HazardsManager.cs` | Responsible for causing hazards to damage and hinder players during PVP gameplay |
| `BasePickUp.cs`, `ATKIncreasePickUp.cs`, `ChargePickUp.cs`, `HealPickUp.cs` | Responsible for pick ups to buff and heal the players during PVP gameplay |
| `PickUpContainer.cs`, `PickUpSpawnerManager.cs`, `PickUpContainer.cs` | Responsible for managing pick ups spawning location and timing |
| `PlayerPickUpAnimation.cs` | Responsible for simple vfx when picking up pick ups |
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
