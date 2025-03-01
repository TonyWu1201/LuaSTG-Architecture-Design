# Design Draft: Boss Scene System  

[中文版](../zh/Boss%20Scene%20System.md)  

## Background  

In the current game development framework, we use a global `boss` class as the logical object for bosses in the game, with modules like `boss_system` and `boss_ui` split out to handle the boss’s logic and UI display.  
However, due to the need to maintain compatibility with ancient code during splitting and modifications, the functionality is not fully refined and exhibits many strange quirks.  

This has led to several issues, such as:  

- When a boss needs to initiate dialogue at the end, we must either add an extra normal attack phase or disable the boss’s explosion effect via a flag.  
- After the boss’s explosion effect finishes, it actively deletes itself, making it inconvenient to implement post-explosion dialogue.  
- The UI logic and boss logic are coupled, making it difficult to reuse them as components.  
- The boss logic does not natively support multiple bosses well, requiring extensive additional logic to handle multi-boss scenarios.  
- Requirements like spell card practice or executing different battle phases based on different characters or logic demand significant extra logic to manage.  

## Proposal  

We propose a new boss scene system as a replacement to address the aforementioned issues.  

This boss scene system roughly consists of the following components:  

#### Data Structures  

1. `Boss Battle Stage`: Defines a boss’s battle phase, including the boss’s behavior, whether to enable score calculation, whether to trigger a spell card declaration, and other related logic.  
2. `Boss Style`: Defines a boss’s style, including its appearance, animations, sound effects, etc. It contains no logic, can be freely swapped, and does not affect the normal flow of execution.  
3. `Boss Scene`: Defines a boss scene, including how bosses appear within the scene, which battle stages are invoked, how transitions occur, and other related logic.  

#### Entity Objects  

1. `Boss Entity`: An instantiated object of `Boss Style`, responsible for rendering the boss’s appearance, handling attack detection, and executing animations, sound effects, and other logic in response to events.  
2. `Boss Scene Controller`: Manages the logic of the entire boss scene as a singleton, controlling the creation, destruction, and transitions of bosses, among other operations.  

## Detailed Explanation  

#### Boss Battle Stage  

This structure is similar to the current definitions of `spellcard`, `dialog`, or `move` phases, but with the following differences:  

1. Defined externally as a standalone entity, accessed via an ID or other indexing mechanism.  
2. Not bound to any specific object; available boss objects are passed as parameters, and different bosses are controlled via numeric indices. As long as the correct number of bosses is provided, the logic execution remains unaffected regardless of the boss styles used.  
3. Can define boss behavior, whether to enable score calculation, whether to trigger a spell card declaration, and other battle-phase-related logic.  
4. Can specify how health and time should be handled, such as whether health is synchronized, whether all bosses need to be defeated, or if defeating just one is sufficient.  

#### Boss Style / Boss Entity  

This structure is similar to the current `boss` class, but with the following differences:  

1. Contains no logic, only appearance, animations, sound effects, and similar information.  
2. Has a single event interface to receive events, execute corresponding animations and sound effects, and send hit detection data to the `Boss Scene Controller` for processing.  
3. Can be freely swapped without affecting the normal flow of execution.  

#### Boss Scene  

This structure is similar to the current list of battle phases defined for a boss, but with the following differences:  

1. Can execute some logic, such as creating bosses, destroying bosses, and switching battle phases.  
2. Requires logic to determine which battle phase to initiate and pass the required boss objects for that phase (while ensuring unused boss objects exit the scene correctly).  
3. Enters a waiting state after initiating a battle phase, resuming execution only after the phase concludes.  

#### Boss Scene Controller  

This structure is similar to the current `boss_system`, but with the following differences:  

1. Exists as a singleton, responsible for managing the logic of the entire boss scene.  
2. Handles all damage calculations, time tracking, and phase completion checks for boss objects.  
3. Provides a rich set of API interfaces for controlling boss creation, destruction, transitions, retrieving current score, spell card information, updating UI, and more.  
4. Supports interaction with external systems via an event mechanism, such as hit detection, boss events, and boss states.  

## Advantages  

1. Separation of logic and appearance allows boss styles to be swapped freely without affecting logic execution.  
2. Easily supports multiple bosses by simply passing the correct boss objects.  
3. Requirements like executing different battle phases based on characters or logic can be handled with conditional checks in the `Boss Scene`.  
4. Features like spell card practice can be implemented by defining standalone `Boss Scene` setups, offering greater customization flexibility.  

## Considerations  

1. The dialogue system’s logic will be detached from the boss system and needs to be implemented separately, as it is unrelated to boss logic and should be a standalone, directly callable system.  
2. Spell card practice may require significant manual definition logic, necessitating tools to simplify the process.  
3. The increased complexity of the boss scene system may introduce debugging challenges, and the added logical complexity could potentially lead to performance issues.  