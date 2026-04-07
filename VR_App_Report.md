# VR App Development Report

## Introduction

This VR application was developed as a mobile-first fantasy adventure experience built in Unity 6 for Google Cardboard. The final app places the player inside a stylized world where they move through the environment, complete crystal-based trials, unlock a castle gate, survive a boss encounter, and finally reach an ending sequence with performance feedback.

The project was designed to balance three goals:

1. create a complete VR gameplay loop instead of a simple tech demo,
2. keep the interaction model simple enough for mobile VR hardware,
3. make the experience readable and comfortable for the player.

For that reason, I focused on a clear progression structure, lightweight but meaningful interaction mechanics, and systems that support immersion without overcomplicating the controls.

## What I Developed

The app is built around one main playable scene, `Assets/Scenes/Latest Scene.unity`, which acts as the full game world. Inside that scene, the player progresses through a sequence of connected gameplay systems:

- VR entry and mobile setup using Google Cardboard.
- First-person player movement with head-directed navigation.
- Object pickup, carrying, dropping, and throwing.
- Teleport-based transitions between challenge areas.
- Timed crystal trials.
- Crystal collection as the main progress mechanic.
- A castle gate that opens only after the collection requirement is met.
- A boss fight built around damaging a central weak point with thrown objects.
- Dynamic HUD, audio, and end-of-game results feedback.

In practice, the app functions like a compact VR adventure level rather than a disconnected set of experiments. The player is always working toward a visible goal: collect crystals, clear trials, open the path forward, defeat the boss, and escape.

## How I Built It

### 1. VR platform and project setup

I used Unity 6 (`6000.3.8f1`) and configured the project to support mobile VR through the Google Cardboard XR plugin. This choice made it possible to target smartphone-based VR while still using Unity's XR pipeline. The project also includes the Input System, UI, terrain tools, post-processing, URP/HDRP packages, and Android-focused support.

The `CardboardStartup` script handles the startup sequence by:

- preventing the phone from sleeping,
- setting brightness high for better headset visibility,
- reloading Cardboard device parameters,
- starting XR automatically.

This was important because mobile VR needs a reliable startup flow. If VR does not initialize cleanly, the player immediately loses immersion or may not even be able to begin the experience correctly.

### 2. Player movement and input

Player locomotion is managed through `PlayerMovement.cs`, using a `CharacterController`. I combined standard first-person movement with VR-specific orientation logic:

- movement direction is based on the headset/camera forward vector,
- vertical camera pitch is clamped to avoid unnatural over-rotation,
- gravity and jumping are applied through the controller,
- sprint and camera stick look are available when a controller is connected,
- a simple touch fallback is used for Android/Cardboard play.

I also created a shared input layer in `GameInput.cs`, which reads input from gamepad, joystick, keyboard, or mobile touch paths depending on what is available.

I chose this approach because VR controls must feel simple and predictable. Instead of building a complicated gesture-heavy interaction model, I used a hybrid control design:

- controller support gives better testing and faster navigation during development,
- Cardboard trigger and touch support keeps the app usable on mobile VR hardware,
- head-relative movement helps the player move in the direction they are looking, which is more intuitive in VR.

### 3. Core interaction: pickup and throw system

One of the central mechanics is object interaction, implemented in `PickUpScript.cs`. The player can:

- raycast forward to detect pickup objects,
- grab valid rigidbodies tagged as pickup items,
- hold them in front of the player,
- drop them,
- throw them forward to activate targets or damage enemies.

The script temporarily changes the object's gravity, damping, collision handling, and velocity so it behaves correctly while being carried. It also ignores collisions with the player while the item is held to prevent jitter and unwanted physics issues.

I developed the app this way because throwing objects is one of the easiest and most satisfying interaction patterns in VR. It gives the player a direct sense of agency without requiring complex hand-tracking or advanced controller hardware. It also lets one mechanic serve multiple purposes:

- solving environmental interactions,
- triggering trial targets,
- damaging the boss,
- making the player feel physically present in the world.

### 4. Progression structure and trials

To make the game feel purposeful, I built progression around trials and crystal collection.

The timing system is handled through `TrialTimerSystem.cs` and `GameManager.cs`. When the player enters certain teleports, the `Teleport` script starts a trial using a `trialId` such as `Trial_1` or `Trial_2`. The trial timer updates the HUD in real time and stores the final times when objectives are completed.

`CrystalCollector.cs` tracks how many crystals the player has collected, updates the UI, and can complete a trial once the required crystal threshold is reached.

This structure was used because it creates short-term and long-term goals at the same time:

- short-term goal: finish the immediate challenge quickly,
- long-term goal: collect all required crystals and progress to the castle.

That combination gives the player something to focus on moment by moment while still feeling that every area contributes to a bigger journey.

### 5. Environment logic and fail-safe design

I added environmental systems to make traversal more reliable and less frustrating:

- `MovingPlatform.cs` carries the player correctly while standing on the platform.
- `TrialArenaSafetyInstaller.cs` creates runtime safety colliders and reset zones.
- a fall reset is used for Trial 1 so the player can be returned to the correct entry point instead of getting stuck.
- Trial 2 receives generated arena boundary walls to keep the encounter contained.

I made these choices because VR discomfort often increases when players become disoriented, trapped, or forced to restart manually. Safety systems protect the flow of the experience and reduce frustration. In a VR app, comfort and recovery design are just as important as the main mechanic.

### 6. Gate unlock and world progression

`CastleGateController.cs` listens for the "all crystals collected" event and then animates the gate open with sound and optional object activation. The gate only opens once the player has completed the required progression.

This was done to make progress visible in the world itself, not just in UI text. A large animated gate is a strong environmental reward because it turns the player's achievement into a physical change in the environment. In VR, world feedback is usually stronger than abstract menu feedback.

### 7. Boss encounter

The boss sequence begins through `BossFightTrigger.cs`, which activates the boss state and enables or disables scene objects when the player enters the encounter zone.

The actual damage logic is handled by either `BossHealth.cs` or `GemHealth.cs`, depending on the setup. In the current scene, `GemHealth` appears to be the main active boss-health system. The player damages the boss by hitting the weak point with objects tagged `canPickUp`. The boss can also attack back using `GemShooter.cs`, which procedurally fires glowing projectiles toward the player and uses `ProjectileImpact.cs` for collision effects and damage delivery.

I designed the boss this way for several reasons:

- it reuses the pickup-and-throw mechanic instead of introducing a completely new combat system,
- it makes the weak point clear and readable,
- it creates a progression from exploration to action,
- it adds tension at the end of the experience without needing advanced AI.

This was a deliberate design choice: instead of trying to build a very complex enemy, I built a focused encounter where the challenge comes from timing, positioning, and using an already learned mechanic under pressure.

### 8. HUD, audio, and feedback systems

The `HUDManager` controls:

- crystal count,
- trial timer,
- boss health display.

The `EndScreenUIController` presents:

- crystals collected,
- total game time,
- total combined trial time,
- individual trial results,
- basic completion feedback.

Audio is managed globally with `AudioDirector.cs` and `AudioLibrary.cs`. Music changes between ambient exploration, trial music, and boss music depending on the current gameplay state. Additional sound effects support jumping, throwing, footsteps, gate movement, and boss death.

I included these systems because feedback is essential in VR. Since the player is surrounded by the world, they need constant confirmation that their actions matter. The HUD provides objective progress, while audio supports emotional pacing and environmental presence. Together, they make the app feel more complete and responsive.

## Why I Developed the App This Way

The overall design approach was based on simplicity, clarity, and immersion.

First, I chose a fantasy exploration structure because it fits VR well. Castles, crystals, gates, and boss arenas are visually clear and naturally support environmental storytelling. The player immediately understands that crystals are important, the castle is a destination, and the boss is a final challenge.

Second, I reused one main interaction mechanic across the whole experience: pickup and throw. This was intentional. In VR, every extra mechanic adds learning cost and can confuse the player. By reusing the same interaction for puzzles, traversal support, and combat, the app becomes easier to learn and more cohesive.

Third, I designed around the limits of mobile VR. Google Cardboard does not provide the same interaction depth as high-end VR hardware, so the app needed controls that still worked with simple input. That is why the experience relies on:

- gaze/head-directed movement,
- simple trigger-based interaction,
- teleport transitions,
- readable objective design,
- controlled encounter spaces.

Fourth, I paid attention to comfort and reliability. Features like bounded arenas, reset triggers, stable object-holding physics, and limited interaction complexity help prevent the player from getting lost, stuck, or overwhelmed. That is especially important in VR, where confusion affects both usability and comfort.

Finally, I wanted the app to feel like a full mini-game rather than a collection of unrelated prototypes. The timing system, crystal counter, gate unlock, boss fight, and end screen all support that goal. They give the player a beginning, middle, and end, which makes the project stronger as an assignment and more satisfying as an experience.

## Conclusion

This VR app was developed as a complete, mobile-friendly fantasy adventure built around accessible VR interaction. The design combines exploration, timed trials, crystal collection, environmental progression, and a final boss encounter into one connected gameplay loop.

I developed it this way because the project needed to be practical for Cardboard hardware while still demonstrating core VR design principles:

- presence through first-person immersion,
- interaction through object manipulation,
- progression through visible world changes,
- feedback through UI, sound, and results,
- comfort through controlled movement and safety systems.

Overall, the final result reflects a deliberate design process: keep the controls simple, make the objectives clear, reuse mechanics intelligently, and build an experience that feels complete from start to finish.
