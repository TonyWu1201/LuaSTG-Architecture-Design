# Design Draft: Unified Destruction Callback Mechanism

[中文版](../zh/统一的销毁回调机制.md)

## Background

In the current game development framework, object destruction is managed by using the `lstg.Kill` and `lstg.Del` functions to clean up objects. These functions trigger the `kill` and `del` callbacks of the object type respectively to provide post-processing logic.
Additionally, in the script layer, there are `RawKill` and `RawDel` functions that are used to destroy objects while skipping the `kill` and `del` callbacks.

This design leads to a lot of inconsistent handling and is inconvenient to manage, especially when objects need to execute certain fixed logic.
For example, when needing to perform linked cleanup of certain associated objects, using cleanup functions that skip callbacks will result in incorrect outcomes.

Moreover, within the existing framework, there is a frequently used follower binding system. This system’s chained cleanup of bound objects is implemented by overriding and wrapping the `lstg.Kill` and `lstg.Del` functions in the script layer.
However, this approach causes issues when objects are cleaned up off-screen (determined by the underlying system), as the underlying system directly triggers the object’s `del` callback, bypassing the chained cleanup logic, which leads to additional problems.

## Problem Statement

- **Inconsistent Logic:** The separation of `kill` and `del` callbacks complicates destruction handling, especially when similar but slightly different logic is needed in different scenarios.
- **Raw Function Limitations:** Using `RawKill` and `RawDel` to skip callbacks may inadvertently skip necessary processing logic.
- **Off-Screen Cleanup Issue:** Off-screen destruction only directly triggers the object’s `del` callback, which may cause the cleanup logic for bound followers to be unexpectedly skipped.

## Proposed Solution

We propose using a unified destruction function as the core cleanup function.
In this proposal, we use `Destroy(target: lstg.GameObject, reason: string, ...)` as the abstraction for this destruction function.
We need to replace the underlying `lstg.Kill` and `lstg.Del` functions with wrappers around this function.
Objects will no longer have separate `kill` and `del` callbacks; instead, they will be modified to pass a `reason` parameter to the `del` callback, enabling centralized and flexible handling of all destruction scenarios.

We will provide several built-in `reason` parameter values and allow users to define custom `reason` parameters for other needs.Here, we assume the built-in `reason` parameter values are as follows:

- `"kill"`: Corresponds to the processing logic when using the `lstg.Kill` function
- `"del"`: Corresponds to the processing logic when using the `lstg.Del` function
- `"out-bound"`: Corresponds to the processing logic for object off-screen cleanup
- As for the `RawKill` and `RawDel` functions, they will be custom-provided in the script layer.

### Key Changes

1. **Unified Function:** Introduce `Destroy(target: lstg.GameObject, reason: string, ...)` as the core destruction function.
2. **Wrappers:** Redefine existing functions as wrappers:

   - `Kill(target: lstg.GameObject)` → `Destroy(target: lstg.GameObject, "kill")`
   - `Del(target: lstg.GameObject)` → `Destroy(target: lstg.GameObject, "del")`
   - Off-screen destruction → `Destroy(target: lstg.GameObject, "out-bound")`
   - `RawKill(target: lstg.GameObject)` / `RawDel(target: lstg.GameObject)` → Converted to use custom `reason` in the script layer
3. **Parameterized Callback:** Update the `del` callback to accept a `reason` parameter for handling specific scenario logic.

### Example Usage

```lua
---@param reason string @The reason for destruction
function class:del(reason, ...)
    if reason == "out-bound" then
        -- Handle off-screen destruction
    elseif reason == "kill" then
        -- Handle kill-specific logic
    elseif reason == "del" then
        -- Handle del-specific logic
    end
    -- Common cleanup logic
end
```

## Benefits

- **Consistency:** Centralizes destruction handling, reducing errors caused by scattered callbacks.
- **Flexibility:** The `reason` parameter supports customized responses to different destruction triggers.
- **Simplification:** Eliminates separate `kill` and `del` callbacks, simplifying the framework design.

## Implementation Considerations

- **Breaking Change:** Code using existing `kill` and `del` callbacks will need to be updated.
- **Migration Support:** Provide detailed documentation and examples to assist in transitioning to the new system.
- **Custom Reasons:** Allow users to define custom `reason` values to meet special cleanup requirements.

## Conclusion

This unified destruction callback mechanism addresses the inconsistencies in the current system, providing a more robust and adaptable solution for object management. Although it is a breaking change, the improvements in usability and maintainability justify the transition.
