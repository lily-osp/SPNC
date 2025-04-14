**Document 2: Keymap Definition Document**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1 (Initial Draft - Requires Iteration)
**Date:** 2023-10-27

**1. Introduction**

This document defines the logical mapping of functions, characters, and symbols to the physical keys of the 8x8 (64 keys) matrix keyboard used by the SPNC. This mapping is essential for usability and will be implemented by the `keyboard_manager.py` module, referencing the configuration loaded by `config_manager.py`.

**2. Modifier Keys**

The following keys are designated as modifiers. Their active state changes the output of subsequent key presses. The UI (status bar) must clearly indicate the active modifier state.

*   **`SHIFT`:** Accesses secondary functions, uppercase letters, and specific symbols. Behavior can be momentary (hold) or latching (press once to activate, press again to deactivate) - *Decision: Initial implementation: Latching preferred for complex entry.*
*   **`FN` (Function):** Accesses tertiary functions, often programming keywords or less common scientific functions. *Decision: Latching.*
*   **`ALPHA`:** Accesses primarily lowercase/uppercase letters for variable names, string input, and scripting. *Decision: Latching.*

**3. Keymap Layers**

The keymap is defined in multiple layers:

*   **Base Layer:** Output when no modifier keys are active. Contains numbers, basic operators, navigation, core functions.
*   **Shift Layer:** Output when `SHIFT` is active. Contains secondary operators, common symbols, uppercase letters (if ALPHA not primary), shifted functions.
*   **FN Layer:** Output when `FN` is active. Contains programming commands, system functions, advanced scientific functions.
*   **Alpha Layer:** Output when `ALPHA` is active. Contains letters (primarily lowercase unless SHIFT is also active?). Needs careful design.

**4. Keymap Table (Example - **THIS IS A DRAFT and requires significant refinement and user testing)**

This table represents the physical 8x8 grid. Each cell shows `Base / Shift / FN / ALPHA` layer outputs. `[CMD]` indicates a command code rather than a character, which `main.py` interprets. `[MOD]` indicates a modifier key.

**(Row 0 - Top Row)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `7` / `[` / `STO A` / `G` | `8` / `]` / `STO B` / `H` | `9` / `{` / `STO C` / `I` | `/` / `}` / `PI` / ` ` | `[UP]` / `[PGUP]` / `[HOME]` / ` ` | `sin` / `asin` / `ASINH` / ` ` | `cos` / `acos` / `ACOSH` / ` ` | `[CMD:AC]` / `[CMD:OFF]` / `[CMD:RESET]` / ` ` |

**(Row 1)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `4` / `(` / `STO D` / `D` | `5` / `)` / `STO E` / `E` | `6` / `,` / `STO F` / `F` | `*` / `^` / `SQRT` / ` ` | `[LEFT]` / ` ` / ` ` / ` ` | `tan` / `atan` / `ATANH` / ` ` | `log` / `10^x` / `LOG1P` / ` ` | `[CMD:CE]` / `[CMD:DEL]` / `[CMD:UNDO]` / ` ` |

**(Row 2)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `1` / `!` / `STO X` / `A` | `2` / `;` / `STO Y` / `B` | `3` / `:` / `STO Z` / `C` | `-` / `_` / `NEG` / ` ` | `[RIGHT]` / ` ` / ` ` / ` ` | `ln` / `e^x` / `EXP` / ` ` | `abs` / `round` / `IPART` / ` ` | `[MOD:SHIFT]` / `[MOD:SHIFT]` / `[MOD:SHIFT]` / `[MOD:SHIFT]` |

**(Row 3)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `0` / ` ` / `RCL X` / ` ` | `.` / ` ` / `RCL Y` / ` ` | `E` / `e` / `RCL Z` / ` ` | `+` / ` ` / `ANS` / ` ` | `[DOWN]` / `[PGDN]` / `[END]` / ` ` | `[CMD:RUN]` / `[CMD:STOP]` / `[CMD:STEP]` / ` ` | `[CMD:OK]` / `[CMD:ENTER]` / ` ` / ` ` | `[MOD:ALPHA]` / `[MOD:ALPHA]` / `[MOD:ALPHA]` / `[MOD:ALPHA]` |

**(Row 4 - Dedicated Programming/System?)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `[CMD:MODE]` / ` ` / ` ` / ` ` | `[CMD:PROG]` / `[CMD:SAVE]` / `[CMD:LOAD]` / ` ` | `def` / `class` / `IMPORT` / `J` | `if` / `else` / `ELIF` / `K` | `for` / `in` / `RANGE` / `L` | `while` / `break` / `CONTINUE` / `M` | `return` / `yield` / `PASS` / `N` | `[MOD:FN]` / `[MOD:FN]` / `[MOD:FN]` / `[MOD:FN]` |

**(Row 5 - More Alpha/Symbols?)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `'` / `"` / `` ` `` / `O` | `<` / `<=` / ` ` / `P` | `>` / `>=` / ` ` / `Q` | `=` / `==` / `!=` / `R` | `[` / ` ` / `STO G` / `S` | `]` / ` ` / `STO H` / `T` | `{` / ` ` / `STO I` / `U` | `}` / ` ` / `STO J` / `V` |

**(Row 6 - System/API Keys?)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `[CMD:WIFI]` / `[CMD:IP]` / `[CMD:PING]` / `W` | `[CMD:API WIKI]` / ` ` / ` ` / `X` | `[CMD:API LLM]` / ` ` / ` ` / `Y` | `[CMD:I2C SCAN]` / ` ` / ` ` / `Z` | `True` / ` ` / `RCL G` / ` ` | `False` / ` ` / `RCL H` / ` ` | `None` / ` ` / `RCL I` / ` ` | `[CMD:ESC]` / `[CMD:BACK]` / ` ` / ` ` |

**(Row 7 - Bottom Row - Maybe less used?)**

| Col 0         | Col 1        | Col 2        | Col 3        | Col 4      | Col 5      | Col 6        | Col 7        |
| :------------ | :----------- | :----------- | :----------- | :--------- | :--------- | :----------- | :----------- |
| `&` / ` ` / `RCL J` / ` ` | `|` / ` ` / ` ` / ` ` | `~` / ` ` / ` ` / ` ` | `%` / ` ` / `MOD` / ` ` | `[CMD:HIST]` / ` ` / ` ` / ` ` | `[CMD:LOG]` / ` ` / ` ` / ` ` | `[CMD:CONFIG]` / ` ` / ` ` / ` ` | `[CMD:HELP]` / ` ` / ` ` / ` ` |

**5. Interpretation of Key Outputs**

*   **Characters (`'1'`, `'a'`, `'+'`, `'_'`):** Appended directly to the current input buffer (calculator input, script editor, API prompt).
*   **Function Names (`'sin'`, `'log'`):** Appended to the input buffer, typically followed by an opening parenthesis `(`.
*   **Keywords (`'def'`, `'if'`):** Appended to the input buffer in script editing mode, potentially with a space.
*   **Commands (`[CMD:AC]`, `[CMD:RUN]`, `[CMD:MODE]`):** These are not characters. `keyboard_manager` returns a special code or string identifying the command. `main.py` interprets these codes to perform actions (clear screen, run script, change mode, etc.).
*   **Modifiers (`[MOD:SHIFT]`, `[MOD:FN]`, `[MOD:ALPHA]`):** These keys toggle the internal modifier state within `keyboard_manager`. They don't produce direct output but change the interpretation of subsequent keys.
*   **Navigation (`[UP]`, `[LEFT]`, `[PGDN]`, `[HOME]`):** Special codes interpreted by `main.py` or the active UI component (editor, browser) to move the cursor or scroll the view.

**6. Configuration**

The actual mapping (which character/command corresponds to which row/col/layer) will be defined in `/sd/config.json` to allow user customization without reflashing firmware. The `keyboard_manager` will load this mapping.

**7. Open Questions / Areas for Refinement**

*   How to handle Python indentation easily?
*   Optimal placement of frequently used symbols (`=`, `()`, `"`).
*   Clarity of `ALPHA` vs `SHIFT` for letters (e.g., does `ALPHA` give lowercase, `SHIFT`+`ALPHA` give uppercase?).
*   Are dedicated `STO`/`RCL` keys needed, or use `FN`+Variable Key? (Example uses `FN` layer).
*   Is latching the best behavior for all modifiers?
*   Need user testing to validate layout ergonomics.

---

**Document 7: Configuration File Specification (`/sd/config.json`)**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document specifies the structure and content of the main configuration file, `/sd/config.json`, used by the SPNC. This file allows customization of hardware pins, network settings, API configurations, and other parameters without modifying the firmware. The `config_manager.py` module is responsible for parsing this file. If the file is missing or invalid, the system should attempt to use hardcoded default values where possible, but some features (WiFi, APIs) may be unavailable.

**2. File Format**

The file MUST be a valid JSON object. Comments are not allowed in standard JSON.

**3. Root Object Structure**

```json
{
  "system": { ... },
  "hardware": { ... },
  "keymap": { ... },
  "wifi": { ... },
  "apis": { ... },
  "ui": { ... },
  "i2c_map": { ... } // Optional, could be separate /sd/i2c_map.json
}
```

**4. Section Details**

**4.1. `system` Section**

*   **Description:** General system settings.
*   **Keys:**
    *   `log_level`: (String) Logging threshold ("DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"). Default: "INFO".
    *   `log_file_path`: (String) Path for system log. Default: "/sd/calculator.log".
    *   `log_max_size_kb`: (Integer) Maximum size of the log file in KB before rotation (if rotation implemented). Default: 1024.
    *   `history_file_path`: (String) Path for calculation history. Default: "/sd/history.jsonl".
    *   `history_max_entries`: (Integer) Maximum number of history entries to keep. Default: 500.
    *   `default_angle_mode`: (String) Initial angle mode ("DEG", "RAD", "GRAD"). Default: "DEG".

**Example `system`:**
```json
  "system": {
    "log_level": "INFO",
    "log_file_path": "/sd/calculator.log",
    "log_max_size_kb": 512,
    "history_file_path": "/sd/history.jsonl",
    "history_max_entries": 200,
    "default_angle_mode": "DEG"
  }
```

**4.2. `hardware` Section**

*   **Description:** Defines GPIO pin assignments and hardware parameters. **Pin numbers must correspond to Micropython `machine.Pin` numbers.**
*   **Keys:**
    *   `oled_interface`: (String) "SPI" or "I2C".
    *   `oled_spi_pins` (Object, if `oled_interface` is "SPI"):
        *   `sck`: (Integer) Clock pin.
        *   `mosi`: (Integer) MOSI pin.
        *   `cs`: (Integer) Chip Select pin.
        *   `dc`: (Integer) Data/Command pin.
        *   `rst`: (Integer | null) Reset pin (null if not used).
    *   `oled_i2c_pins` (Object, if `oled_interface` is "I2C"):
        *   `sda`: (Integer) SDA pin.
        *   `scl`: (Integer) SCL pin.
        *   `address`: (Integer) I2C address (e.g., 0x3C -> 60).
    *   `oled_width`: (Integer) Display width in pixels. Default: 256.
    *   `oled_height`: (Integer) Display height in pixels. Default: 128.
    *   `sd_spi_pins`: (Object)
        *   `sck`: (Integer) Clock pin.
        *   `mosi`: (Integer) MOSI pin.
        *   `miso`: (Integer) MISO pin.
        *   `cs`: (Integer) Chip Select pin.
    *   `keyboard_rows`: (List[Integer]) List of 8 GPIO pin numbers for keyboard rows (Outputs).
    *   `keyboard_cols`: (List[Integer]) List of 8 GPIO pin numbers for keyboard columns (Inputs, Pull-up enabled).
    *   `ext_i2c_pins`: (Object)
        *   `sda`: (Integer) SDA pin for external bus.
        *   `scl`: (Integer) SCL pin for external bus.
    *   `ext_i2c_freq`: (Integer) Frequency for external I2C bus (e.g., 100000, 400000). Default: 100000.

**Example `hardware` (SPI Display):**
```json
  "hardware": {
    "oled_interface": "SPI",
    "oled_spi_pins": { "sck": 18, "mosi": 23, "cs": 5, "dc": 17, "rst": 16 },
    "oled_width": 256,
    "oled_height": 128,
    "sd_spi_pins": { "sck": 18, "mosi": 23, "miso": 19, "cs": 4 },
    "keyboard_rows": [32, 33, 27, 14, 12, 13, 15, 2],
    "keyboard_cols": [39, 34, 35, 36, 21, 22, 25, 26],
    "ext_i2c_pins": { "sda": 25, "scl": 26 },
    "ext_i2c_freq": 100000
  }
```

**4.3. `keymap` Section**

*   **Description:** Defines the logical key mapping for the 8x8 matrix.
*   **Keys:**
    *   `layout_name`: (String) User-defined name for this layout (e.g., "Standard_QwertyAlpha").
    *   `modifier_keys`: (Object) Defines which physical keys act as modifiers. Key is `"[RxCy]"` (e.g., "[R2C7]"), Value is modifier name ("SHIFT", "FN", "ALPHA").
    *   `layers`: (Object) Contains definitions for each layer ("base", "shift", "fn", "alpha").
        *   Each layer is an Object where the Key is `"[RxCy]"` (Row/Col coordinate string) and the Value is the output string (character, function name, keyword, or `[CMD:...]` / `[MOD:...]` code).

**Example `keymap` (Partial - simplified from Doc 6):**
```json
  "keymap": {
    "layout_name": "SPNC_Default_v0.1",
    "modifier_keys": {
      "[R2C7]": "SHIFT",
      "[R3C7]": "ALPHA",
      "[R4C7]": "FN"
    },
    "layers": {
      "base": {
        "[R0C0]": "7", "[R0C1]": "8", "[R0C2]": "9", "[R0C3]": "/", /* ... */ "[R0C7]": "[CMD:AC]",
        "[R1C0]": "4", "[R1C1]": "5", "[R1C2]": "6", "[R1C3]": "*", /* ... */ "[R1C7]": "[CMD:CE]",
        /* ... more rows ... */
        "[R7C7]": "[CMD:HELP]"
      },
      "shift": {
        "[R0C0]": "[", "[R0C1]": "]", "[R0C2]": "{", "[R0C3]": "}", /* ... */ "[R0C7]": "[CMD:OFF]",
        "[R1C0]": "(", "[R1C1]": ")", "[R1C2]": ",", "[R1C3]": "^", /* ... */ "[R1C7]": "[CMD:DEL]",
        /* ... more rows ... */
        "[R7C7]": " " // Example: Shift+Help does nothing
      },
      "fn": {
         "[R0C0]": "[CMD:STO A]", "[R0C1]": "[CMD:STO B]", /* ... */ "[R0C7]": "[CMD:RESET]",
         /* ... */
         "[R4C0]": "[CMD:MODE]", "[R4C1]": "[CMD:PROG]", /* ... */
         /* ... more rows ... */
         "[R7C7]": " "
      },
      "alpha": {
         "[R0C0]": "g", "[R0C1]": "h", "[R0C2]": "i", /* ... */ // Assuming lowercase alpha, SHIFT+ALPHA for uppercase handled by keyboard_manager
         /* ... */
         "[R4C0]": " ", "[R4C1]": " ", "[R4C2]": "def", "[R4C3]": "if", /* ... */
         /* ... more rows ... */
         "[R7C7]": " "
      }
    }
  }
```

**4.4. `wifi` Section**

*   **Description:** WiFi connection details. Store credentials here **only if security is not a major concern for the target environment**. Otherwise, prompt user or use another mechanism.
*   **Keys:**
    *   `auto_connect`: (Boolean) Attempt to connect automatically on boot. Default: `true`.
    *   `ssid`: (String) WiFi network name. Default: "".
    *   `password`: (String) WiFi password. Default: "". **Handle with care!**

**Example `wifi`:**
```json
  "wifi": {
    "auto_connect": true,
    "ssid": "MyHomeNetwork",
    "password": "MySecretPassword123" // Security risk!
  }
```

**4.5. `apis` Section**

*   **Description:** Configuration for external web APIs. **API Keys MUST NOT be stored here if the SD card could be compromised.** Consider prompting the user or using device-specific secure storage if available. For prototyping, they might be here.
*   **Keys:**
    *   `wikipedia`: (Object)
        *   `endpoint`: (String) Wikipedia API endpoint URL (e.g., "https://en.wikipedia.org/w/api.php").
        *   `default_params`: (Object) Default query parameters (e.g., `{"format": "json", "action": "query", "prop": "extracts", "exintro": true, "explaintext": true}` ).
    *   `llm`: (Object)
        *   `endpoint`: (String) URL for the LLM API.
        *   `api_key`: (String) API key for the LLM. **Major Security Risk!**
        *   `model`: (String) Optional: Specify LLM model name if required by API.
        *   `default_params`: (Object) Optional: Default parameters for LLM request body.

**Example `apis`:**
```json
  "apis": {
    "wikipedia": {
      "endpoint": "https://en.wikipedia.org/w/api.php",
      "default_params": {"format": "json", "action": "query", "prop": "extracts", "exintro": true, "explaintext": true, "redirects": 1 }
    },
    "llm": {
      "endpoint": "https://api.example-llm.com/v1/completions",
      "api_key": "YOUR_API_KEY_HERE", // SECURITY RISK - Load differently in production!
      "model": "text-davinci-003"
    }
  }
```

**4.6. `ui` Section**

*   **Description:** User interface appearance and behavior settings.
*   **Keys:**
    *   `default_font`: (String) Path to a default font file (e.g., "/flash/lib/fonts/font_8x8.py") or name of built-in font.
    *   `large_font`: (String | null) Path/name for a larger font used for results (optional).
    *   `scroll_speed_ms`: (Integer) Delay between steps for automatic text scrolling (if implemented). Default: 150.
    *   `modifier_latching`: (Boolean) True if modifier keys latch, False if momentary. Default: `true`.
    *   `show_variable_pane`: (Boolean) Whether to attempt showing a variable watch area in CALC mode. Default: `true`.

**Example `ui`:**
```json
  "ui": {
    "default_font": "builtin_8x8",
    "large_font": null,
    "scroll_speed_ms": 200,
    "modifier_latching": true,
    "show_variable_pane": true
  }
```

**4.7. `i2c_map` Section** (Alternatively, `/sd/i2c_map.json`)

*   **Description:** Optional mapping of symbolic names to I2C device/register addresses. See structure in Requirements Doc v0.3, Section 9.
*   **Keys:** Object where each key is a device name (String), and the value is an object containing `addr` (String/Int) and `registers` (Object mapping register names to addresses/numbers).

**Example `i2c_map`:**
```json
  "i2c_map": {
    "TEMP_SENSOR": {
      "addr": "0x48",
      "registers": { "READ_TEMP": "0x00", "CONFIG": "0x01" }
    },
    "RTC": {
      "addr": "0x68",
      "registers": { "SECONDS": 0, "MINUTES": 1, "HOURS": 2 }
    }
  }
```

---

**Document 8: Project Plan / Roadmap (Initial)**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document outlines the planned phases and major milestones for the development of the SPNC project. Timelines are estimates and subject to change based on complexity and resources.

**2. Development Philosophy**

*   **Incremental Development:** Build core features first, then add complexity in layers.
*   **Modular Design:** Adhere to the defined software architecture for maintainability.
*   **Test-Driven (Aspirational):** Write unit/integration tests for critical components (sandbox, API, core math) where feasible. Manual system testing is essential throughout.
*   **Iterative UI/UX:** The initial UI and keymap are starting points; expect refinement based on testing and feedback.

**3. Phases & Milestones**

**Phase 1: Foundation & Core Calculator (Estimate: 2-4 Weeks)**

*   **M1.1: Hardware Bring-up:** Assemble prototype hardware. Verify basic ESP32 operation, display initialization (showing basic text), SD card mounting/reading/writing.
*   **M1.2: Basic I/O:** Implement `keyboard_manager` (basic scan, no modifiers yet) and `display_manager` (basic text drawing). Implement `config_manager` to load pin settings. `main.py` loop reads keys and echoes to display.
*   **M1.3: Core Math Engine:** Implement `math_engine` for basic arithmetic (+,-,*,/) using sandboxed `eval` or simple parser. Handle number input. Implement basic CALC mode UI layout.
*   **M1.4: SD Card Basics:** Implement `sd_manager` for basic file listing and reading/writing plain text. Implement history logging (simple append).
*   **M1.5: Configuration Loading:** Fully implement `config_manager` to load all hardware sections from `/sd/config.json`.
*   **Outcome:** A basic calculator capable of arithmetic, displaying input/output, reading the keyboard, and interacting minimally with the SD card.

**Phase 2: Scientific & UI Enhancement (Estimate: 3-5 Weeks)**

*   **M2.1: Scientific Functions:** Integrate `math` module functions into `math_engine`. Implement DEG/RAD/GRAD mode switching and display indicator.
*   **M2.2: Advanced Input/Display:** Handle parentheses, operator precedence correctly in `math_engine`. Implement multi-line display/scrolling in `display_manager`. Handle scientific notation input/output.
*   **M2.3: Variable Memory:** Implement STO/RCL logic and `variables` dictionary in `math_engine`.
*   **M2.4: Keymap Implementation:** Fully implement `keyboard_manager` including modifier keys (SHIFT/FN/ALPHA) based on `config.json` keymap. Refine keymap layout.
*   **M2.5: UI Layout Refinement:** Implement the multi-zone UI layout for CALC mode, potentially including variable watch pane. Improve status bar.
*   **Outcome:** A functional scientific calculator with variable memory and a more refined UI leveraging the 256x128 display.

**Phase 3: Scripting Engine & Sandbox (Estimate: 4-6 Weeks)**

*   **M3.1: Script Editor UI:** Implement `script_manager` UI for basic line editing (displaying script, editing current line).
*   **M3.2: Script File I/O:** Implement saving/loading `.py` files to `/sd/scripts/` via `script_manager` and `sd_manager`. Implement File Browser UI.
*   **M3.3: Sandbox Implementation:** **Critical.** Develop `sandbox_exec.py` with restricted builtins/globals. Implement safe `exec()`. Test rigorously.
*   **M3.4: Basic API:** Implement core `calculator_api.py` functions (variable access, display output, basic input, delay).
*   **M3.5: Script Execution:** Integrate sandbox execution into `script_manager` (RUN command). Implement basic error reporting (exceptions, line numbers) to the display.
*   **Outcome:** Users can write, save, load, and execute simple Micropython scripts that can manipulate variables and display output. Sandbox provides basic security.

**Phase 4: API & Networking (Estimate: 3-5 Weeks)**

*   **M4.1: WiFi Connection:** Implement `network_manager` for connecting to WiFi based on config. Display status icon.
*   **M4.2: HTTP API:** Implement `calculator_api.http_get/http_post` using `urequests`.
*   **M4.3: Wikipedia API:** Implement `calculator_api.wiki_search` including request formatting and JSON response parsing. Add API-WIKI mode UI.
*   **M4.4: LLM API:** Implement `calculator_api.llm_query` including request formatting, *secure* API key handling (loading from config, but warn user), JSON parsing. Add API-LLM mode UI.
*   **M4.5: Network Error Handling:** Improve robustness of network operations (timeouts, error reporting).
*   **Outcome:** Calculator can connect to WiFi and user scripts can interact with basic web APIs (HTTP, Wikipedia, LLM).

**Phase 5: I2C Integration (Estimate: 2-4 Weeks)**

*   **M5.1: I2C Hardware/Manager:** Implement `i2c_manager` to initialize and control the external I2C bus based on config.
*   **M5.2: I2C API:** Implement `calculator_api.i2c_scan/read/write` functions.
*   **M5.3: I2C Device Map:** Implement loading and usage of the optional I2C map for `calculator_api.i2c_read/write_reg`.
*   **M5.4: Testing:** Test thoroughly with known external I2C devices using scripts. Add I2C Monitor mode (optional).
*   **Outcome:** User scripts can communicate with external hardware via the I2C bus.

**Phase 6: Refinement & Documentation (Ongoing)**

*   **M6.1: Bug Fixing:** Address issues identified during testing.
*   **M6.2: Performance Optimization:** Profile code, optimize bottlenecks (display updates, script execution).
*   **M6.3: UI/UX Polish:** Refine layouts, fonts, key mappings based on usability testing.
*   **M6.4: Feature Completion:** Implement remaining 'nice-to-have' API functions or UI features from requirements.
*   **M6.5: Documentation:** Finalize User Manual, Developer Guide, and update all specification documents.
*   **M6.6: Release Preparation:** Create distributable code/firmware package, write release notes.
*   **Outcome:** A stable, well-documented version 1.0 release candidate.

**4. Dependencies & Risks**

*   **Hardware Availability:** Sourcing ESP32 w/PSRAM and specific 256x128 OLED.
*   **Micropython Driver Availability/Stability:** Ensuring stable drivers exist for chosen OLED and potentially PSRAM.
*   **Sandbox Security:** Implementing the execution sandbox correctly and securely is paramount and complex.
*   **UI/UX Complexity:** Designing an intuitive interface for coding and complex functions on a small device with a matrix keyboard is challenging.
*   **Memory Management:** Even with PSRAM, careful memory management in Micropython is required, especially with networking and large data structures.
*   **API Key Security:** Handling external API keys securely is a significant challenge.

**5. Tools**

*   IDE: Thonny, VS Code with Micropython extensions.
*   Version Control: Git (e.g., GitHub, GitLab).
*   Serial Monitor: For debugging output.
*   File Transfer: Thonny, rshell, ampy.
*   Hardware Debugger: JTAG debugger (optional, advanced).
*   I2C Analyzer: Logic analyzer or dedicated I2C tool for debugging external bus.
