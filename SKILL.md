---
name: pawn-skills
description: "Essential Pawn best practices, high-performance optimization techniques, and architectural guidelines for both SA-MP and open.mp. Use when writing new systems, refactoring code, optimizing server lag, or working with databases and streamer plugins. Triggers on: pawn, samp, open.mp, y_hooks, mysql, optimization, streamer."
version: 1.0.0
---

# Pawn Skills

A comprehensive guide to modern Pawn standards, performance optimization, and architectural best practices for open.mp.

## Before You Start

**This skill prevents legacy SA-MP bad practices and optimizes overall server performance.**

### Known Issues This Skill Prevents
1. Server lag from inefficient `MAX_PLAYERS` loops.
2. Memory leaks and stack overflows from oversized string allocations.
3. Main thread freezing from unthreaded MySQL queries.
4. Callback conflicts from manual hooking instead of using `y_hooks`.
5. Unnecessary processing overhead from misusing the Streamer plugin.

## 1. Naming & Consistency
Clear naming is the foundation of maintainable code.

*   **Variables:**
    *   **Local:** Use `camelCase` (e.g., `targetId`, `playerScore`).
    *   **Global/Static:** Use `gs_VariableName` or `VariableName`.
    *   **Strings:** Always account for the Null Terminator (+1).
*   **Functions/Stocks:**
    *   Use `Module_Action` naming (e.g., `Player_Spawn`, `Vehicle_LoadData`).
    *   Ensure consistency with `Get/Set` pairs (e.g., `GetPlayerPoints` / `SetPlayerPoints`).
*   **Macros & Constants:**
    *   Use `SCREAMING_SNAKE_CASE` (e.g., `MAX_VEHICLE_SLOTS`).
    *   **Safety:** Always wrap macro parameters in parentheses `((%0))` to prevent order-of-operation bugs.

### Pawn Macro Mastery (#define)
Treat macros purely as **"Text Replacement"**.
*   **Safety:** Always wrap parameters in parentheses `(%0)` to avoid logic errors during replacement.
*   **Structure:** Do **not** use trailing semicolons in `#define`. Use backslashes (`\`) for multi-line macros.
*   **Comments:** Use macros for readability, but remember that commented-out code (`//`) is removed during preprocessing and consumes **zero** RAM.

## 2. Advanced Optimization (Anti-Lag)
Pawn is an interpreted language; every opcode counts.

### A. Looping Patterns
The standard `MAX_PLAYERS` loop is the most common cause of lag ($O(N)$ where $N$ is maximum capacity).
*   **Best:** Use `foreach(new i : Player)` (requires the foreach library).
*   **Backward Iteration:** The fastest standard loop is backward iteration using pre-calculated bounds.
    ```pawn
    for(new i = GetPlayerPoolSize(); i != -1; i--) { ... }
    ```

### B. Array & Memory Management
*   **Standalone Variables:** Fetching coordinates into separate variables (x, y, z) is faster than using an array due to lower indexing overhead in AMX.
*   **Fast Copying:** Use `memcpy` for bulk array transfers instead of manual loops.
*   **Stack Conservation:** Declare complex arrays *exactly* when needed, after early return checks.
*   **Multi-Variable Assignment:** Save bytecode size with `x = y = z = 0;`.
*   **2D Array Optimization:** Dereferencing a 2D array inside a loop `arr[id][i]` is expensive. Pass the sub-array to a function `stock Work(sub[])` to treat it as 1D, which is significantly faster.
*   **Memset for Arrays:** Use `memset` instead of `for` loops to zero out large arrays.

### C. High-Performance Math
*   **Avoid Square Roots:** Calculating `floatsqrt` is expensive. Compare squared distances instead.
*   **Explicit Floats:** Never mix integers in float math (e.g., use `5.0 * 2.0`, not `5.0 * 2`) to avoid runtime conversion.

### Memory & Variable Guard
*   **Variable Sizing:** Avoid "Lazy Allocation". Do not use `512` for a short 32-character message.
*   **Null Terminator:** Always account for the `+1` cell required for the null terminator in strings.
*   **Comma-Separated:** Declare related variables in a single statement (e.g., `new a, b, c;`) to reduce opcode overhead.
*   **On-Demand:** Declare variables only at the moment they are needed to conserve AMX stack space.


## 3. Syntactic Sugar (Modern Pawn)
Pawn supports unique syntax that makes code cleaner and faster.

*   **Zero/False Checks:** Use logical NOT `!`.
    *   ✅ `if (!GetFactionOnline(type))`
*   **Range Checking:** Use chained relational operators.
    *   ✅ `if (0 < health < 100)`
    *   ✅ `if (!(0 < health < 100))` (for "Outside" range)
*   **Inline Assignments:** Fetch and check in one line.
    *   ✅ `if ((id = GetTarget(playerid)) != INVALID_PLAYER_ID) { ... }`

## 4. Architectural Standards
*   **Stock vs Public:** Use `stock` for internal logic. Use `public` only for Timers, RPCs, or Engine Callbacks. `stock` allows the compiler to remove unused code.
*   **Identity Management:** Never use hardcoded IDs (Magic Numbers). Use `enum` for Dialog IDs and data slots to prevent collisions.
*   **Variadic Support:** If using a framework with variadic messaging, do not manually `format()` strings.
    *   ✅ `SendClientMessage(playerid, -1, "ID: %d", id);`
*   **Manual Hooking:** For precision, avoid auto-hooking systems. Call system-specific procedures manually in main callbacks (e.g., `main\loader\hooks.inc`) to control execution order exactly.
*   **YSI Hooking (Automated):** If using the YSI library, utilize `y_hooks` for modularity. It manages execution order via library-internal priorities.
    *   ✅ `hook OnPlayerConnect(playerid) { ... }`
*   **Capture Flow:** Use boolean signals to stop callback chains early if an event is "captured".
    ```pawn
    if (Dialog_SystemA(playerid, ...)) return 1; // Handled, stop here
    if (Dialog_SystemB(playerid, ...)) return 1; // Not handled by A, check B
    ```
*   **Functional Pawn (y_inline):** Use `y_inline` to create closures for MySQL or Dialog callbacks, keeping related logic in one place.
*   **Compile-time Guards:** Avoid `CallLocalFunction` if the name is known. Use `#if defined FunctionName` to call it directly. It avoids expensive internal string comparisons.

## 5. Comparison Examples: Common vs. Best Practice

### Ex 1: Distance Comparison (Performance)
*   ❌ **Bad (Slow):**
    ```pawn
    if (floatsqrt((x*x) + (y*y)) > 5.0) return 1;
    ```
*   ✅ **Best (Fast):**
    ```pawn
    if (((x*x) + (y*y)) > 25.0) return 1; // 5 squared is 25
    ```

### Ex 2: Player Looping
*   ❌ **Bad (Legacy/Slow):**
    ```pawn
    for(new i = 0; i < MAX_PLAYERS; i++) {
        if (!IsPlayerConnected(i)) continue;
        // logic
    }
    ```
*   ✅ **Best (Modern):**
    ```pawn
    foreach(new i : Player) {
        // logic (Foreach handles connection check)
    }
    ```

### Ex 3: Range Checking
*   ❌ **Bad (Verbose):**
    ```pawn
    if (playerScore >= 100 && playerScore <= 500) { ... }
    ```
*   ✅ **Best (Clean):**
    ```pawn
    if (100 <= playerScore <= 500) { ... }
    ```

### Ex 4: Memory Usage (Large Strings)
*   ❌ **Bad (Static Bloat):**
    ```pawn
    new msg[1024]; // Declared at the top, stays on stack for the whole function
    if (!valid) return;
    format(msg, ...);
    ```
*   ✅ **Best (Deferred/Ondemand):**
    ```pawn
    if (!valid) return;
    new msg[1024]; // Only declares if logic reaches this point
    format(msg, ...);
    ```

### Ex 5: Hooking Methods
*   🔧 **Manual (Total Control):**
    ```pawn
    // main\loader\hooks.inc
    public OnPlayerConnect(playerid) {
        Auth_OnConnect(playerid);
        Inv_OnConnect(playerid); // Order is explicit
        return 1;
    }
    ```
*   🚀 **YSI (Automated Modularity):**
    ```pawn
    #include <YSI_Coding\y_hooks>
    hook OnPlayerConnect(playerid) {
        // Just write logic, YSI handles the link
        return 1;
    }
    ```

## 6. Pro Tips: Color Management
Use a dual-constant system for colors to keep code readable.
*   `COLOR_RED` (Hex): `0xFF0000FF` (for function arguments)
*   `CL_RED` (String): `"{FF0000}"` (for embedding in messages)

**Standard Usage:**
```pawn
SendClientMessage(playerid, COLOR_WHITE, "This is "CL_RED"Red "CL_WHITE"and this is White");
```

## 7. open.mp & Database Standards
*   **OPENMP_COMPAT:** Define `#define OPENMP_COMPAT` for legacy SA-MP native support in open.mp environments.
*   **SQL Injection:** **Mandatory** use of `mysql_format`. Never pass raw string values directly into queries.
*   **Threaded Queries:** Use `mysql_pquery` to prevent the main thread from freezing during heavy DB tasks.
*   **Native Superiority:** Always prefer Native functions (`strcat`, `memcpy`, `strfind`) over manual Pawn loops. Natives run on the actual CPU, bypassing the virtual machine overhead.
*   **Memory Footprint:** Save RAM by redefining core constants to your actual server capacity:
    ```pawn
    #undef MAX_PLAYERS
    #define MAX_PLAYERS 100 // Instead of 500 or 1000
    ```

## 8. Streamer Plugin Strategy
Do not use the Streamer plugin unless absolutely necessary. Using it for small numbers of objects (e.g., < 1000) adds unnecessary overhead.

### A. When to Avoid
*   **SAMP Limits:** SAMP natively supports up to 1000 objects. If your map is small, use `CreateObject`.
*   **Performance Cost:** Streamer is a database of objects that updates based on player distance. This process runs on a separate tick and has a cost.

### B. The Smart Switch Trick
If you aren't sure if you'll need a streamer later, write your code using a dynamic-ready style, but point it to native functions:
```pawn
// At the top of your script (if not using streamer yet)
#define CreateDynamicObject CreateObject
#define DestroyDynamicObject DestroyObject
```

### C. Hybrid Placement
*   **Static Zones (High Traffic):** Use `CreateObject` for objects at Spawn points or main cities. Since players are always there, the streamer would create them anyway—using native saves the "check" cost.
*   **Remote Zones:** Use `CreateDynamicObject` for distant areas that players rarely visit to save on the 1000-object limit.

## 9. Anti-Patterns to Avoid
1.  **Thread Blocking:** Never run heavy SQL queries without `mysql_pquery` (threaded).
2.  **Hardcoded Integers:** Always use `enum` to manage Dialogs, Menus, and Data Slots.
3.  **OnPlayerUpdate Bloat:** Never place complex logic or database saving inside `OnPlayerUpdate`. Use timers.
4.  **Returning Functions in Callbacks:** Never `return SendClientMessage(...)`. Call it first, then return `1` or `0` explicitly to ensure predictable callback behavior.
5.  **Mixed Encoding:** Always ensure `.pwn` files are saved in `Windows-874` when using Thai characters for SAMP.
6.  **Unnecessary Streamer Dependency:** Don't include the streamer plugin if you are managing less than 1000 total objects.
