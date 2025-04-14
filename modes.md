**Document 3: Functional Specification: Calculator, Programming & Modes**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document elaborates on the core functionalities of the SPNC, detailing the features available in the interactive Scientific Calculator mode, the capabilities and environment of the Programming Mode (using Micropython scripting), and the distinct operational Modes the user can navigate. It expands on the functional requirements outlined in the main specification (v0.3).

**2. Scientific Calculator Mode (CALC)**

This is the default mode on startup, providing standard scientific calculation capabilities.

**2.1. Input Method**
*   Expressions are entered using the 8x8 keyboard, leveraging Base, SHIFT, FN, and ALPHA layers for numbers, operators, functions, constants, and variables.
*   Input appears in the Primary Interaction Area (Zone 2) of the display, with a visible cursor.
*   The display should support multi-line input or horizontal scrolling for long expressions.
*   Key presses provide immediate visual feedback (character appears, cursor moves).
*   Editing:
    *   `[CMD:CE]` (Clear Entry/Backspace): Deletes the character/function before the cursor.
    *   `[CMD:AC]` (All Clear): Clears the current input expression, any pending operation, and potentially error messages. Does *not* clear variables or history.
    *   `[LEFT]`, `[RIGHT]`: Move the cursor within the input expression for inserting/deleting characters.

**2.2. Supported Operations & Functions**
*   **Arithmetic:** `+`, `-`, `*`, `/`, `%` (Modulo). Implicit multiplication (e.g., `2(3+1)`) might be supported but requires careful parser design; explicit `*` is safer initially.
*   **Operator Precedence:** Standard mathematical order of operations (PEMDAS/BODMAS) is enforced. Parentheses `(` `)` are used for grouping.
*   **Constants:** `pi` (π ≈ 3.14159...), `e` (e ≈ 2.71828...). Accessed via dedicated keys (likely FN/SHIFT layer).
*   **Trigonometry (respecting DEG/RAD/GRAD mode):**
    *   `sin(x)`, `cos(x)`, `tan(x)`
    *   `asin(x)`, `acos(x)`, `atan(x)` (Inverse/Arc functions)
    *   `sinh(x)`, `cosh(x)`, `tanh(x)` (Hyperbolic functions)
    *   `asinh(x)`, `acosh(x)`, `atanh(x)` (Inverse Hyperbolic)
*   **Logarithms & Exponentials:**
    *   `log(x)` (Common logarithm, base 10)
    *   `ln(x)` (Natural logarithm, base e)
    *   `exp(x)` (e raised to the power x, e^x)
    *   `pow(base, exp)` or `base ^ exp` (Power function)
    *   `sqrt(x)` (Square root)
    *   `10^x` (10 raised to the power x - often SHIFT+log)
*   **Number Functions:**
    *   `abs(x)` (Absolute value)
    *   `floor(x)`, `ceil(x)`
    *   `round(x, [ndigits])` (Rounding)
    *   `fact(x)` or `x!` (Factorial - for non-negative integers)
    *   `neg(x)` or unary `-` (Negation)
*   **Angle Mode Conversion:** Functions accessible via API/FN key to convert values (e.g., `todeg(rad_val)`, `torad(deg_val)`).

**2.3. Angle Modes (DEG/RAD/GRAD)**
*   The current mode (Degrees, Radians, Gradians) affects the input interpretation and output of all trigonometric functions.
*   The active mode is clearly indicated in the Status Bar.
*   The mode can be cycled using a dedicated key sequence (e.g., FN + MODE or specific DRG key) or set via the `calculator_api.set_mode()` function in scripts.

**2.4. Variables & Memory**
*   **`Ans` Variable:** The result of the last successful calculation triggered by `=` is automatically stored in `Ans`. It can be used in subsequent calculations by pressing a dedicated `ANS` key (likely FN layer).
*   **Named Variables (A-Z):** 26 memory locations accessible by single letters.
    *   **Storing:** `[CMD:STO A]` (or similar key sequence) followed by evaluating an expression or using the current `Ans`. Example sequence: `5 * 3 =` (Ans=15), `[CMD:STO A]` stores 15 into A. Or `[CMD:STO A] 2*pi` directly stores the result.
    *   **Recalling:** Use the variable letter directly in expressions (requires ALPHA key access) or via `[CMD:RCL A]` (or similar) which inserts the value of A into the input expression.
*   Variable values are persistent within a session but are *not* automatically saved to SD card on shutdown (unless explicitly done by a script or future auto-save feature). Scripts can save/load variables using `calculator_api.save/load_data`.

**2.5. Calculation & Output**
*   Pressing `=` (or ENTER/OK) triggers the evaluation of the expression in the input buffer by the `math_engine`.
*   The result is displayed in the Main Content Area (Zone 3). Sufficient precision should be shown. Scientific notation is used for very large or small numbers.
*   The result also updates the `Ans` variable.
*   Calculation history (input expression, result) is logged to `/sd/history.jsonl`. A portion of recent history might be viewable on the display (Zone 4 or dedicated HISTORY mode).

**2.6. Error Handling**
*   **Syntax Errors:** Detected upon pressing `=`. Display message like "SYNTAX ERR". Input buffer remains for correction.
*   **Math Errors:** Detected during evaluation (e.g., division by zero, log of negative number, sqrt of negative). Display message like "MATH ERR: DOMAIN" or "MATH ERR: DIV/0".
*   Error messages are displayed clearly, potentially overwriting the result area until the next key press or AC.

**3. Programming Mode (SCRIPT)**

This mode allows users to write, edit, save, load, and run custom scripts using Micropython syntax within a sandboxed environment.

**3.1. Environment**
*   **Language:** Micropython (superset of Python 3 syntax relevant to embedded systems).
*   **Execution:** Scripts are executed via `exec()` within the sandbox (`sandbox_exec.py`).
*   **Sandbox Restrictions:**
    *   Limited builtins (essential calculation, type, control flow - no file I/O, no `eval`/`exec`, no `__import__`).
    *   Limited allowed imports (`math`, `random`, `ujson`, `time`, `calculator_api`).
    *   No direct hardware access (GPIO, SPI, internal peripherals).
    *   File system access limited to `/sd/data/` via `calculator_api` functions.
*   **API Access:** All interaction with the calculator system (display, variables, SD data, network, I2C) MUST go through the functions provided by the implicitly available `calculator_api` module (See Document 7: API Specification).

**3.2. Script Editor (SCRIPT-EDIT Mode)**
*   **Interface:** Entered via `[CMD:PROG]` key or MODE cycle.
*   **Display:** Shows script content with line numbers in the scrollable Main Content Area (Zone 3). The current line being edited is shown in the Primary Interaction Area (Zone 2) with a cursor.
*   **Navigation:** `[UP]`/`[DOWN]` scroll through lines and select the line to edit. `[LEFT]`/`[RIGHT]` move the cursor within the current editor line (Zone 2).
*   **Editing:** Use keyboard (ALPHA, SHIFT, FN layers) to type Micropython code into Zone 2. `[CMD:CE]` backspaces. `[CMD:ENTER]` confirms the edit for the current line and typically moves to the next line for editing.
*   **Indentation:** Handling Python's significant whitespace is a challenge. Potential solutions:
    *   Dedicated `[INDENT]`, `[DEDENT]` keys (using FN layer?).
    *   Automatic indentation after lines ending in `:`.
    *   Requires careful implementation and clear user feedback.
*   **File Operations:** Dedicated key sequences (e.g., FN+S for Save, FN+L for Load) or on-screen menus trigger saving the current buffer to a `.py` file or loading a different `.py` file (entering FILE-BROWSE mode).

**3.3. Script Execution (SCRIPT-RUN Mode)**
*   **Trigger:** `[CMD:RUN]` key pressed while in SCRIPT-EDIT mode (runs current buffer) or FILE-BROWSE mode (runs selected `.py` file).
*   **Process:** `script_manager` passes the script code to `sandbox_exec.execute()`.
*   **Output:** Script output via `calculator_api.display()` appears in the Main Content Area (Zone 3).
*   **Input:** `calculator_api.prompt_number/string()` pauses execution and activates keyboard input.
*   **Status:** Status bar indicates "RUNNING". Execution pauses for input or `delay_ms`. Display shows "DONE" on completion or "ERROR" on failure.
*   **Interruption:** A dedicated key (e.g., ESC or AC, possibly long-press) should allow the user to attempt to interrupt a running script (requires cooperative checks within long loops or specific interrupt handling in the execution environment). A confirmation prompt ("Stop Script? Y/N") is recommended.
*   **Error Handling:** Python exceptions caught by the sandbox are displayed with the script name and line number. Execution halts.

**3.4. File Browser (FILE-BROWSE Mode)**
*   **Context:** Used for loading/saving scripts (`/sd/scripts/`) and potentially user data files (`/sd/data/`).
*   **Display:** Lists files/directories in the current path. Highlights the selected item. May show file size/date.
*   **Navigation:** `[UP]`/`[DOWN]` navigate the list. `[ENTER]` selects a file (for Load/Run) or enters a directory. `[CMD:SAVE]` might use the highlighted name or prompt for a new name. `[ESC]` goes up one directory level or exits the browser. Keys for Delete, Rename might be added.

**4. Operational Modes**

The calculator operates in several distinct modes, managed by `main.py`. The `[CMD:MODE]` key typically cycles through the primary modes. Specific actions (like Load/Save) might temporarily enter sub-modes (like FILE-BROWSE).

*   **`CALC` (Calculator Mode):** Default mode for interactive scientific calculations.
*   **`SCRIPT-EDIT` (Script Editor):** Mode for writing and editing `.py` scripts.
*   **`SCRIPT-RUN` (Script Running):** Transient mode while a script is executing. UI is primarily controlled by script output/prompts.
*   **`FILE-BROWSE` (File Browser):** Used for selecting files from the SD card (scripts, data). Entered from SCRIPT-EDIT (Load/Save) or potentially a dedicated File Manager option.
*   **`API-WIKI` (Wikipedia Query):** Mode for entering a search term and viewing Wikipedia results.
*   **`API-LLM` (LLM Query):** Mode for entering a prompt and viewing LLM results.
*   **`I2C-MONITOR` (I2C Monitor - Optional):** A diagnostic mode for direct interaction with the external I2C bus (scan, read raw, write raw). Useful for hardware debugging.
*   **`CONFIG` (Configuration - Optional):** A potential mode to view/edit certain settings directly on the device (e.g., WiFi credentials, display brightness) instead of editing `config.json` manually.
*   **`HISTORY` (History View - Optional):** Mode to scroll through and potentially recall items from the calculation history log.

**5. Mode Transitions**

*   `[CMD:MODE]` key cycles through a predefined sequence (e.g., CALC -> SCRIPT-EDIT -> FILE-BROWSE -> API-WIKI -> API-LLM -> CALC).
*   Specific actions trigger transitions:
    *   In SCRIPT-EDIT, pressing LOAD/SAVE enters FILE-BROWSE.
    *   In SCRIPT-EDIT or FILE-BROWSE, pressing RUN enters SCRIPT-RUN.
    *   Completing an API query might return to CALC or SCRIPT mode.
*   `[CMD:ESC]` / `[CMD:BACK]` typically cancels the current action within a mode or reverts to the previous major mode (e.g., from FILE-BROWSE back to SCRIPT-EDIT).
*   `[CMD:AC]` usually resets the state *within* the current mode (clears input, stops script prompt) but doesn't necessarily change the mode itself, unless perhaps held long in certain modes.
