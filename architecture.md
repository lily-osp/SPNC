**Document 1: System Architecture Document**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document details the software and hardware architecture for the SPNC project, complementing the main Requirements Specification (v0.3). It describes the interaction between major components, data flow, and key design decisions.

**2. High-Level Architecture**

The SPNC employs a modular architecture running on a single ESP32 with PSRAM. The system revolves around a central event loop (`main.py`) that dispatches tasks based on user input and system state. Key subsystems include User Interface (Keyboard/Display), Calculation Engine, Scripting Engine (with Sandbox), Storage Management (SD Card), Networking, and I2C Communication. User scripts interact with the system exclusively through a sandboxed API (`calculator_api`).

**(Conceptual Diagram - Textual Representation)**

```mermaid
graph TD
    subgraph ESP32 Hardware (with PSRAM)
        A[User Input: 8x8 Matrix Keyboard] --> B(Keyboard Manager);
        B --> C{main.py Event Loop / Mode Dispatcher};
        C --> D[Display Manager];
        D --> E[Output: 256x128 OLED];

        C --> F[Math Engine: Interactive Calc];
        F --> G[Shared State: Variables, Mode];
        G --> F;

        C --> H[Script Manager: Editor UI, File Ops];
        H --> I[SD Manager: Read/Write .py];
        I --> J[(SD Card)];
        H --> K[Sandbox Executor];
        K --> L[calculator_api Module];
        L --> G;  // Access state via API
        L --> D;  // Update display via API
        L --> F;  // Use math functions? maybe via API
        L --> I;  // Access SD data via API
        L --> M[Network Manager];
        L --> N[I2C Manager];
        K --> G; // Sandbox updates state based on script exec via API

        C --> M; // Mode-specific network ops? (e.g. API direct access)
        M --> O[(WiFi)];
        M --> J; // Load config? (via Config Manager)

        C --> N; // Mode-specific I2C ops? (e.g. I2C Monitor)
        N --> P[(External I2C Devices)];
        N --> J; // Load I2C Map? (via Config Manager)

        C --> I; // History/Log Saving

        Q[Config Manager] --> J; // Load config.json
        Q --> C; // Provide config to main
        Q --> M; // Provide WiFi/API config
        Q --> N; // Provide I2C config
        Q --> B; // Provide Keymap
    end

    style J fill:#f9f,stroke:#333,stroke-width:2px
    style E fill:#ccf,stroke:#333,stroke-width:2px
    style A fill:#ccf,stroke:#333,stroke-width:2px
    style O fill:#cfc,stroke:#333,stroke-width:2px
    style P fill:#cfc,stroke:#333,stroke-width:2px
```

**3. Module Breakdown**

*   **`main.py`:** Core application entry point. Initializes all managers and hardware interfaces. Runs the main event loop, polling the keyboard and dispatching actions based on the current operational mode (CALC, SCRIPT-EDIT, etc.) and the detected key press. Manages transitions between modes.
*   **`config_manager.py`:** Responsible for loading configuration settings (Pins, WiFi, API keys/endpoints, I2C settings, Keymap, I2C Device Map) from `/sd/config.json` on startup. Provides access to configuration data for other modules. Handles defaults if the config file is missing or invalid.
*   **`keyboard_manager.py`:** Scans the 8x8 hardware matrix. Debounces key presses. Translates row/column signals into key codes based on the current modifier state (SHIFT, FN, ALPHA) and the loaded keymap. Provides a simple interface for `main.py` to get the latest valid key press.
*   **`display_manager.py`:** Abstracts the specific OLED hardware driver (for 256x128 resolution). Provides high-level functions for drawing UI elements (status bar, input lines, scrollable text areas, menus). Manages fonts and screen updates. Takes commands from `main.py` (for direct UI) and `calculator_api.py` (for script output).
*   **`math_engine.py`:** Handles evaluation of mathematical expressions entered in interactive CALC mode (likely using a sandboxed `eval` or a custom parser). Manages the shared calculator state (variables A-Z, Ans) and the current angle mode (DEG/RAD/GRAD). Provides core math functions.
*   **`script_manager.py`:** Manages the UI for the script editor and file browser (`/sd/scripts/`). Handles loading script content into an editor buffer and saving it back to the SD card via `sd_manager`. Initiates script execution by passing the script content and the `calculator_api` instance to `sandbox_exec.py`.
*   **`sandbox_exec.py`:** **Critical Security Component.** Takes script code (string) and executes it using `exec()`. Crucially, it defines a highly restricted global and builtin environment for the `exec()` call, preventing access to sensitive functions (`open`, `eval`, `__import__`, etc.) and modules. It injects the `calculator_api` module instance into the script's global scope. Catches exceptions during script execution and reports them with line numbers.
*   **`calculator_api.py`:** **Facade/Interface for User Scripts.** Defines the *only* way user scripts can interact with the calculator's system resources (display, variables, SD card data, networking, I2C). Functions in this module call the appropriate underlying managers (`display_manager`, `sd_manager`, `network_manager`, `i2c_manager`, `math_engine`) to perform actions safely.
*   **`network_manager.py`:** Manages the WiFi connection state (connecting, disconnecting, status checks) based on configuration. Provides abstracted functions (`http_get`, `http_post`) using `urequests` for making network calls, used by `calculator_api.py`.
*   **`sd_manager.py`:** Provides abstracted functions for interacting with the SD card: reading/writing script files (`.py`), user data files (`/sd/data/*`), reading config (`/sd/config.json`), writing history (`/sd/history.jsonl`), writing logs (`/sd/calculator.log`), and reading the I2C map (`/sd/i2c_map.json`). Handles file system operations and errors.
*   **`i2c_manager.py`:** Initializes and manages the configured external I2C hardware bus. Provides low-level functions (`scan`, `read`, `write`) used by `calculator_api.py`. May incorporate logic to use the I2C Device Map loaded via `config_manager.py` to translate device/register names.
*   **`lib/`:** Contains third-party drivers (OLED, `sdcard.py`) and libraries (`urequests.py`, `ujson.py`, fonts, etc.). These should be treated as external dependencies.

**4. Data Flow Examples**

*   **Interactive Calculation:** Keyboard Scan -> `keyboard_manager` -> Key Code -> `main.py` -> Update Input Buffer -> `display_manager` -> Display Update. On '=' press: `main.py` -> `math_engine.evaluate(input_buffer)` -> `math_engine` updates `variables['Ans']` -> `main.py` gets result -> `display_manager` -> Display Result.
*   **Script Execution:** User selects "Run" in Script Mode -> `main.py` -> `script_manager.get_script_content()` -> `script_manager` calls `sandbox_exec.execute(script_code, api_instance)` -> `sandbox_exec` runs `exec()` -> Script calls `calculator_api.display("Hello")` -> `calculator_api` calls `display_manager.draw_text(...)` -> OLED Update.
*   **API Call from Script:** Script calls `calculator_api.wiki_search("ESP32")` -> `calculator_api` calls `network_manager.http_get(...)` -> `network_manager` uses `urequests` -> WiFi -> Internet -> Response -> `network_manager` -> `calculator_api` -> Script receives result string.
*   **I2C Read from Script:** Script calls `calculator_api.i2c_read_reg("TEMP_SENSOR", "READ_TEMP", 2)` -> `calculator_api` calls `i2c_manager.read_device_register("TEMP_SENSOR", "READ_TEMP", 2)` -> `i2c_manager` looks up address/register in map -> `i2c_manager` performs hardware I2C transaction -> `i2c_manager` returns bytes -> `calculator_api` -> Script receives data.

**5. Key Interfaces**

*   **User Script <-> System:** `calculator_api` module (strictly defined functions).
*   **Modules <-> SD Card:** `sd_manager` (file operations), `config_manager` (config loading).
*   **Modules <-> Display:** `display_manager` (drawing primitives, layout functions).
*   **Modules <-> Network:** `network_manager` (HTTP requests).
*   **Modules <-> I2C:** `i2c_manager` (bus operations).
*   **Config <-> Modules:** `config_manager` provides loaded settings.

**6. State Management**

*   **Core Calculator State:** `variables` dictionary (A-Z, Ans), current angle mode (DEG/RAD/GRAD) - Managed primarily by `math_engine`, accessible read-only (or via API setters) by scripts.
*   **UI State:** Current operational mode, input buffers, cursor positions, menu selections - Managed by `main.py` and mode-specific logic (e.g., `script_manager` for editor state).
*   **Network State:** WiFi connection status - Managed by `network_manager`.
*   **Configuration:** Loaded once at startup by `config_manager`.

**7. Error Handling Strategy**

*   **Module-Level Exceptions:** Modules should raise specific exceptions for errors (e.g., `SDCardError`, `NetworkError`, `I2CError`, `SandboxViolationError`, `ValueError`, `TypeError`).
*   **API Layer:** `calculator_api` functions must catch exceptions from underlying managers and either return specific error codes/values (e.g., `None`, `False`) or re-raise a curated set of API-specific exceptions for user scripts to handle.
*   **Sandbox:** `sandbox_exec` must catch all exceptions originating from the user script during `exec()`, log them, and report them clearly to the user via the `display_manager`, indicating the script name and line number. Script execution terminates on unhandled exceptions.
*   **Main Loop:** The main loop should have top-level error handling to catch unexpected crashes in core UI/dispatch logic, log the error, and attempt to recover or display a fatal error message.
*   **Logging:** Use the `sd_manager` (or a dedicated logger module) to log significant events and all errors to `/sd/calculator.log` for debugging.

---

**Document 2: API Specification (`calculator_api`)**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document specifies the Application Programming Interface (API) exposed to user-written Micropython scripts (`.py` files) running within the SPNC's sandboxed environment. This API is the *only* sanctioned way for scripts to interact with the calculator's hardware and software resources. All functions are accessed via the implicitly imported `calculator_api` module.

**2. Execution Environment**

Scripts run via `exec()` with a restricted set of builtins and allowed imports (`math`, `random`, `ujson`, `time`, `calculator_api`). Direct hardware access, file I/O (except via this API), and unrestricted imports are prohibited.

**3. API Functions**

**(Note:** Type hints are illustrative for Micropython)**

**3.1. Variable Management**

*   `def get_variable(name: str) -> object | None:`
    *   **Description:** Retrieves the value of a calculator variable (A-Z, 'Ans'). Case-insensitive lookup for 'Ans'.
    *   **Parameters:**
        *   `name`: Name of the variable (string, e.g., "A", "ANS").
    *   **Returns:** The value of the variable, or `None` if the variable doesn't exist.
    *   **Raises:** `TypeError` if name is not a string.
    *   **Example:** `current_ans = calculator_api.get_variable("Ans")`

*   `def set_variable(name: str, value: object) -> bool:`
    *   **Description:** Sets the value of a calculator variable (A-Z). 'Ans' cannot be set directly. Variable name must be a single uppercase letter.
    *   **Parameters:**
        *   `name`: Name of the variable (string, e.g., "A", "B"). Must be A-Z.
        *   `value`: The value to assign (numeric types recommended, others may work but use with caution).
    *   **Returns:** `True` on success, `False` if the variable name is invalid (not A-Z).
    *   **Raises:** `TypeError` if name is not a string.
    *   **Example:** `calculator_api.set_variable("A", 10.5)`

*   `def variables_dict() -> dict:`
    *   **Description:** Returns a *copy* of the current dictionary containing all calculator variables (A-Z, Ans). Modifying this returned dictionary does *not* affect the calculator's state.
    *   **Returns:** A dictionary mapping variable names (str) to their values.
    *   **Example:** `all_vars = calculator_api.variables_dict()`

**3.2. Display Output**

*   `def display(text_or_value: object, line: int = -1) -> None:`
    *   **Description:** Displays the string representation of `text_or_value` on the main output area of the screen. If `line` is -1 (default), appends to the display (scrolling if needed). If `line` >= 0, attempts to write to that specific line (0-indexed from top of main area), overwriting previous content on that line.
    *   **Parameters:**
        *   `text_or_value`: The object to display (will be converted to string).
        *   `line`: The target line number, or -1 to append/scroll.
    *   **Example:** `calculator_api.display("Processing step 1")`, `calculator_api.display(f"Result: {my_result}", line=0)`

*   `def clear_display() -> None:`
    *   **Description:** Clears the main output area of the display (does not affect status bar).
    *   **Example:** `calculator_api.clear_display()`

**3.3. User Input**

*   `def prompt_number(message: str = "?") -> float | int | None:`
    *   **Description:** Pauses script execution, displays the `message` string on the screen, and waits for the user to enter a number using the calculator keypad. Input is terminated by the ENTER/OK key.
    *   **Parameters:**
        *   `message`: The prompt message to display (string).
    *   **Returns:** The number entered by the user (int or float), or `None` if the user cancels (e.g., presses ESC/BACK).
    *   **Raises:** `TypeError` if message is not a string.
    *   **Example:** `radius = calculator_api.prompt_number("Enter Radius:")`

*   `def prompt_string(message: str = "?") -> str | None:`
    *   **Description:** Pauses script execution, displays the `message` string, and waits for the user to enter text using the calculator keypad (requires mapping for alpha chars). Input terminated by ENTER/OK.
    *   **Parameters:**
        *   `message`: The prompt message to display (string).
    *   **Returns:** The string entered by the user, or `None` if the user cancels.
    *   **Raises:** `TypeError` if message is not a string.
    *   **Example:** `user_name = calculator_api.prompt_string("Enter Name:")`

**3.4. SD Card Data I/O** (Operates within `/sd/data/`)

*   `def save_data(filename: str, data_object: object) -> bool:`
    *   **Description:** Saves the `data_object` to a file named `filename` within the `/sd/data/` directory. Uses `ujson.dump` for serialization, overwriting the file if it exists. `filename` should not contain path separators.
    *   **Parameters:**
        *   `filename`: The name of the file (string).
        *   `data_object`: The Python object to save (must be JSON serializable - lists, dicts, str, int, float, bool, None).
    *   **Returns:** `True` on success, `False` on failure (e.g., SD error, serialization error).
    *   **Raises:** `TypeError` if filename is not string or object is not serializable. `OSError` on file system issues.
    *   **Example:** `status = calculator_api.save_data("results.json", {"run1": 123, "run2": 456})`

*   `def load_data(filename: str) -> object | None:`
    *   **Description:** Loads data from the file named `filename` within the `/sd/data/` directory. Uses `ujson.load` for deserialization.
    *   **Parameters:**
        *   `filename`: The name of the file (string).
    *   **Returns:** The deserialized Python object, or `None` if the file doesn't exist or a deserialization/SD error occurs.
    *   **Raises:** `TypeError` if filename is not string. `OSError` on file system issues. `ValueError` on JSON decoding error.
    *   **Example:** `loaded_config = calculator_api.load_data("config.json")`

*   `def delete_data(filename: str) -> bool:`
    *   **Description:** Deletes the file named `filename` from the `/sd/data/` directory.
    *   **Parameters:**
        *   `filename`: The name of the file (string).
    *   **Returns:** `True` on success, `False` if the file doesn't exist or an SD error occurs.
    *   **Raises:** `TypeError` if filename is not string. `OSError` on file system issues.
    *   **Example:** `calculator_api.delete_data("old_log.txt")`

*   `def list_data_files() -> list[str] | None:`
    *   **Description:** Lists the names of files present in the `/sd/data/` directory.
    *   **Returns:** A list of filenames (strings), or `None` if an SD error occurs.
    *   **Example:** `files = calculator_api.list_data_files()`

**3.5. Networking**

*   `def wiki_search(search_term: str) -> str | None:`
    *   **Description:** Performs a search on Wikipedia (using the configured endpoint) for the `search_term`. Attempts to retrieve a short summary or the first paragraph. Requires WiFi connection.
    *   **Parameters:**
        *   `search_term`: The term to search for (string).
    *   **Returns:** A string containing the summary/extract, or `None` if the search fails, no result is found, or a network/API error occurs.
    *   **Raises:** `TypeError` if search_term is not string. `NetworkError` (custom exception) on connection issues.
    *   **Example:** `summary = calculator_api.wiki_search("Ohm's Law")`

*   `def llm_query(prompt: str) -> str | None:`
    *   **Description:** Sends the `prompt` string to the configured Large Language Model API endpoint using the configured API key. Requires WiFi connection.
    *   **Parameters:**
        *   `prompt`: The prompt to send to the LLM (string).
    *   **Returns:** The response string from the LLM, or `None` if the query fails or a network/API error occurs.
    *   **Raises:** `TypeError` if prompt is not string. `NetworkError` on connection issues. `APIKeyError` (custom) if key missing/invalid.
    *   **Example:** `answer = calculator_api.llm_query("Explain relativity simply")`

*   `def http_get(url: str, **kwargs) -> object | None:`
    *   **Description:** Performs an HTTP(S) GET request to the specified `url` using `urequests.get()`. Accepts keyword arguments supported by `urequests.get` (e.g., `headers`, `params`, `timeout`). Requires WiFi connection.
    *   **Parameters:**
        *   `url`: The URL to request (string).
        *   `**kwargs`: Additional arguments for `urequests.get`.
    *   **Returns:** The `urequests.Response` object on success, or `None` on failure (network error, timeout). Scripts need to check `.status_code` and handle `.text` or `.json()` on the response object.
    *   **Raises:** `TypeError` if url is not string. `NetworkError` on connection issues.
    *   **Example:** `response = calculator_api.http_get("http://example.com/data")`
                 `if response and response.status_code == 200: data = response.json()`

*   `def http_post(url: str, **kwargs) -> object | None:`
    *   **Description:** Performs an HTTP(S) POST request to the specified `url` using `urequests.post()`. Accepts keyword arguments supported by `urequests.post` (e.g., `data`, `json`, `headers`, `timeout`). Requires WiFi connection.
    *   **Parameters:**
        *   `url`: The URL to request (string).
        *   `**kwargs`: Additional arguments for `urequests.post`.
    *   **Returns:** The `urequests.Response` object on success, or `None` on failure.
    *   **Raises:** `TypeError` if url is not string. `NetworkError` on connection issues.
    *   **Example:** `response = calculator_api.http_post("http://example.com/api", json={"value": 10})`

**3.6. External I2C Communication**

*   `def i2c_scan() -> list[int] | None:`
    *   **Description:** Scans the configured external I2C bus for connected devices.
    *   **Returns:** A list of integer I2C addresses found, or `None` if an I2C bus error occurs.
    *   **Example:** `devices = calculator_api.i2c_scan()`

*   `def i2c_read(addr: int, nbytes: int) -> bytes | None:`
    *   **Description:** Reads `nbytes` directly from the device at I2C address `addr`.
    *   **Parameters:**
        *   `addr`: The 7-bit I2C address of the device (integer).
        *   `nbytes`: The number of bytes to read (integer > 0).
    *   **Returns:** A `bytes` object containing the data read, or `None` if the device does not acknowledge (NACK) or an I2C error occurs.
    *   **Raises:** `TypeError` for invalid parameter types. `ValueError` if nbytes <= 0. `I2CError` (custom) on bus issues.
    *   **Example:** `raw_data = calculator_api.i2c_read(0x48, 2)`

*   `def i2c_write(addr: int, data: bytes) -> bool:`
    *   **Description:** Writes the `data` (bytes object) directly to the device at I2C address `addr`.
    *   **Parameters:**
        *   `addr`: The 7-bit I2C address of the device (integer).
        *   `data`: The `bytes` object containing data to write.
    *   **Returns:** `True` if the write was acknowledged by the device, `False` on NACK or I2C error.
    *   **Raises:** `TypeError` for invalid parameter types. `I2CError` on bus issues.
    *   **Example:** `status = calculator_api.i2c_write(0x68, b'\x01\x10')`

*   `def i2c_read_reg(device: int | str, register: int | str, nbytes: int) -> bytes | None:`
    *   **Description:** Reads `nbytes` from a specific `register` of an I2C `device`. `device` and `register` can be integer addresses/register numbers or string names defined in the I2C Device Map (if configured). This typically involves an I2C write of the register address followed by a read.
    *   **Parameters:**
        *   `device`: Device address (int) or name from map (str).
        *   `register`: Register address (int) or name from map (str).
        *   `nbytes`: Number of bytes to read (integer > 0).
    *   **Returns:** A `bytes` object containing the data read from the register, or `None` on NACK or error.
    *   **Raises:** `TypeError`, `ValueError`, `I2CError`, `KeyError` (if names not found in map).
    *   **Example:** `temp_bytes = calculator_api.i2c_read_reg("TEMP_SENSOR", "READ_TEMP", 2)` or `temp_bytes = calculator_api.i2c_read_reg(0x48, 0x00, 2)`

*   `def i2c_write_reg(device: int | str, register: int | str, data: bytes) -> bool:`
    *   **Description:** Writes `data` (bytes) to a specific `register` of an I2C `device`. `device` and `register` can be integer addresses/numbers or string names from the I2C map. This typically involves an I2C write of the register address followed by the data bytes.
    *   **Parameters:**
        *   `device`: Device address (int) or name (str).
        *   `register`: Register address (int) or name (str).
        *   `data`: `bytes` object containing data to write to the register.
    *   **Returns:** `True` if the write was acknowledged, `False` on NACK or error.
    *   **Raises:** `TypeError`, `ValueError`, `I2CError`, `KeyError`.
    *   **Example:** `calculator_api.i2c_write_reg("TEMP_SENSOR", "CONFIG", b'\x01')` or `calculator_api.i2c_write_reg(0x48, 0x01, b'\x01')`

**3.7. System Functions**

*   `def delay_ms(milliseconds: int) -> None:`
    *   **Description:** Pauses script execution for the specified duration. Uses `time.sleep_ms()`.
    *   **Parameters:**
        *   `milliseconds`: Duration to pause in milliseconds (integer >= 0).
    *   **Raises:** `TypeError`, `ValueError`.
    *   **Example:** `calculator_api.delay_ms(500)`

*   `def get_mode() -> str:`
    *   **Description:** Returns the current calculator angle mode.
    *   **Returns:** String: "DEG", "RAD", or "GRAD".
    *   **Example:** `current_mode = calculator_api.get_mode()`

*   `def set_mode(mode: str) -> bool:`
    *   **Description:** Sets the calculator angle mode.
    *   **Parameters:**
        *   `mode`: The desired mode ("DEG", "RAD", "GRAD"). Case-insensitive.
    *   **Returns:** `True` on success, `False` if the mode string is invalid.
    *   **Raises:** `TypeError`.
    *   **Example:** `calculator_api.set_mode("RAD")`

*   `def log_message(message: str) -> None:`
    *   **Description:** Appends a timestamped `message` string to the system log file (`/sd/calculator.log`). Useful for script debugging.
    *   **Parameters:**
        *   `message`: The message to log (string).
    *   **Raises:** `TypeError`, `OSError` (if SD write fails).
    *   **Example:** `calculator_api.log_message(f"Sensor reading: {value}")`

*   `def get_timestamp() -> int:`
    *   **Description:** Returns the current system time as seconds since the epoch (requires system time to be set, e.g., via RTC or NTP sync if implemented). Returns 0 or raises an error if time is not set.
    *   **Returns:** Integer timestamp, or 0.
    *   **Example:** `ts = calculator_api.get_timestamp()`

---

**Document 3: UI/UX Design Document**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document describes the user interface (UI) layout, navigation principles, and user experience (UX) goals for the SPNC, utilizing the 256x128 pixel monochrome OLED display and the 8x8 matrix keyboard.

**2. General Principles**

*   **Clarity:** Information should be presented clearly and unambiguously. Avoid clutter.
*   **Responsiveness:** UI must respond immediately to key presses. Feedback for actions (calculations, saves, network calls) should be provided.
*   **Consistency:** Navigation patterns (e.g., use of MODE, ENTER, ESC, UP/DOWN keys) should be consistent across different modes.
*   **Readability:** Use clear fonts. Leverage the larger display height for better spacing and potentially larger fonts where appropriate.
*   **Context:** Utilize the extra screen space to provide relevant context (e.g., view variables while calculating, see file info in browser).

**3. Font Usage**

*   Primary Font: A clear, fixed-width or proportional sans-serif font (e.g., 8px height minimum for main text).
*   Status Bar Font: May use the same font or a slightly smaller variant if needed.
*   Larger Fonts: Consider using larger fonts (e.g., 12-16px height) for displaying primary results or prominent messages, if performance and memory allow. Font choice might be configurable later.

**4. Screen Layout Structure (256x128)**

The screen is conceptually divided into zones:

*   **Zone 1: Status Bar (Top ~10 pixels):** Persistent across most modes.
*   **Zone 2: Primary Interaction Area (Variable Height):** Location for calculator input, current editor line, file selection highlight, main prompt.
*   **Zone 3: Main Content/Output Area (Largest Area):** Displays calculation results, script listings, file browser content, API results, multi-line script output. Should be scrollable using UP/DOWN keys when content exceeds the area.
*   **Zone 4: Context/Info Area (Optional, Bottom or Side):** Could display current variable values, history snippets, help text, depending on the mode and available space.

**5. Status Bar Layout (Zone 1)**

*(Approximate Horizontal Positions)*

*   `[ 0-50 ]`: Current Mode (e.g., "CALC", "SCRIPT", "FILE", "WIFI", "I2C")
*   `[ 55-85 ]`: Angle Mode ("DEG", "RAD", "GRAD")
*   `[ 90-120 ]`: Modifier State ("SHIFT", "FN", "ALPHA") - Only shown when active.
*   `[ ~180-210 ]`: Script Name (`script.py` - if in SCRIPT mode)
*   `[ 220-255 ]`: WiFi Status Icon (e.g., Bars: None, Low, Med, High) / Network Activity Indicator

**6. Mode-Specific Layouts & Navigation**

*   **6.1. Calculator Mode (CALC)**
    *   **Zone 2:** Displays the current input expression being typed (potentially wrapping/scrolling horizontally). A cursor indicates the insertion point.
    *   **Zone 3:** Displays the result of the last calculation ('=' pressed). May show previous calculations (history scrolling up).
    *   **Zone 4 (Optional):** Could show the value of 'Ans' and maybe 2-3 other key variables (A, B, C?).
    *   **Navigation:** Numbers/Operators append to input. '=' evaluates. AC clears all. CE backspaces. STO/RCL keys initiate variable operations. MODE key cycles to other modes.

*   **6.2. Script Edit Mode (SCRIPT-EDIT)**
    *   **Zone 2:** Displays the *current line* being edited, with a cursor. Edits happen here.
    *   **Zone 3:** Displays a scrollable view of the script lines (with line numbers). The currently selected line (via UP/DN) is highlighted.
    *   **Zone 4 (Optional):** Could show script filename, total lines, cursor position (row/col).
    *   **Navigation:** UP/DOWN scrolls through lines in Zone 3 and loads the selected line into Zone 2 for editing. ENTER confirms changes to the line in Zone 2 and moves to the next line. Alpha/Symbol keys edit in Zone 2. Specific keys for SAVE, RUN, LOAD (enter FILE mode). ESC might exit editor (prompt to save?).

*   **6.3. File Browser Mode (FILE-BROWSE)**
    *   **Context:** Entered via LOAD/SAVE commands, or dedicated file mode key. Shows files in `/sd/scripts/` or `/sd/data/`.
    *   **Zone 2:** Shows the currently selected filename.
    *   **Zone 3:** Displays a scrollable list of files in the current directory. Selection highlighted.
    *   **Zone 4 (Optional):** Shows file size, modification date (if feasible).
    *   **Navigation:** UP/DOWN moves selection. ENTER performs the action (Load, Save with this name, Run, View?). ESC/BACK goes back or cancels. Specific keys might allow creating/deleting files.

*   **6.4. Script Run Mode (SCRIPT-RUN)**
    *   **Zone 2:** May show the currently executing line number or prompts from `prompt_xxx` API calls.
    *   **Zone 3:** Displays output generated by the script (`calculator_api.display`). Scrollable.
    *   **Zone 4:** Might show execution status ("Running", "Paused", "Waiting Input").
    *   **Navigation:** Input keys active only during `prompt_xxx` calls. A specific key (e.g., ESC, AC) might be assigned to interrupt/stop script execution (needs confirmation prompt).

*   **6.5. API Interaction Modes (API-WIKI, API-LLM)**
    *   **Zone 2:** Displays the prompt (e.g., "Wiki Search:", "LLM Prompt:") and the user's typed input.
    *   **Zone 3:** Displays the multi-line response received from the API. Scrollable.
    *   **Zone 4:** Shows status ("Fetching...", "Processing...", "Error").
    *   **Navigation:** Alpha/Symbol keys for input. ENTER submits the query. ESC/BACK cancels or returns to previous mode. UP/DOWN scrolls the result area.

*   **6.6. I2C Monitor Mode (Optional - I2C-MONITOR)**
    *   **Purpose:** Direct interaction with the I2C bus for debugging/testing external hardware.
    *   **Layout:** Could allow entering device address, register address, data to write. Display scan results or read data.
    *   **Navigation:** Requires careful key mapping for hex input, read/write commands.

**7. Keyboard Interaction**

*   **Text Entry:** Requires mapping alphabet keys (likely via ALPHA modifier) and common Python symbols (`=`, `(`, `)`, `[`, `]`, `{`, `}`, `"`, `'`, `:`, `_`, `.`, `,`) often using SHIFT/FN modifiers.
*   **Indentation (Scripting):** Simulating indentation might require dedicated keys (INDENT/DEDENT) or context-aware behavior (auto-indent after ':'). This is complex via matrix keyboard.
*   **Modifiers:** Clear visual indication in the status bar when SHIFT, FN, ALPHA are active (latched or momentary?).
*   **Key Rollover/Debouncing:** Handled by `keyboard_manager`.

**8. Navigation Flow**

*   **MODE Key:** Primary method for cycling through major operational modes (CALC -> SCRIPT -> FILE -> NETWORK/API -> I2C -> CALC...). Sequence TBD.
*   **ENTER/OK Key:** Confirms selections, executes actions (calculate, run script, submit API query, select file).
*   **ESC/BACK Key:** Cancels current operation, goes back in menus/modes, clears prompts.
*   **UP/DOWN Keys:** Used for scrolling lists (scripts, files, results) and navigating menus/selections.
*   **LEFT/RIGHT Keys:** Used for cursor movement within input lines/editor.

---

**Document 4: Hardware Design Document / Bill of Materials (BOM)**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document specifies the core hardware components, their interconnections, and pin assignments for the SPNC project based on requirements v0.3.

**2. Core Components (Bill of Materials - Examples)**

| Component          | Specification                                   | Interface | Qty | Notes                                            |
| ------------------ | ----------------------------------------------- | --------- | --- | ------------------------------------------------ |
| **Microcontroller** | ESP32 Module **with PSRAM** (e.g., ESP32-WROVER-E, ESP32-S3-WROOM-1U) | -         | 1   | **PSRAM is Mandatory**. Select module with sufficient Flash/PSRAM (e.g., 16MB/8MB). |
| **Display**        | 256x128 Monochrome OLED Module                 | SPI / I2C | 1   | Specify controller (e.g., SSD1322, SH1107 variant). SPI preferred for refresh rate. Verify driver compatibility. |
| **Keyboard**       | Custom 8x8 (64 key) Matrix PCB w/ Tactile Switches | GPIO      | 1   | Requires diodes for anti-ghosting.                |
| **Storage**        | Micro SD Card Module (incl. level shifting if needed) | SPI       | 1   | Standard module (LC Studio, etc.).              |
| **Power Supply**   | USB Type-C Connector / Cable                     | Power     | 1   | For 5V input.                                   |
| **Voltage Reg.**   | 3.3V LDO Regulator                              | Power     | 1   | Sufficient current capacity for ESP32+Peripherals (e.g., >800mA). |
| **I2C Connector**  | 4-pin Header/Connector (e.g., JST-XH, Pin Header) | I2C       | 1   | For VCC, GND, SDA, SCL connection.             |
| **Pull-up Resistors**| 4.7k Ohm Resistors (or configurable value)     | I2C       | 2   | For external I2C SDA, SCL lines.                |
| **Diodes**         | 1N4148 or similar signal diodes                 | Keyboard  | 64  | For keyboard matrix anti-ghosting.               |
| **Capacitors**     | Decoupling capacitors for ESP32, LDO, etc.      | Power     | ~10 | Various values (e.g., 10uF, 0.1uF).              |
| **PCB**            | Custom PCB to mount components                  | -         | 1   | Optional but recommended for robustness.         |
| **Enclosure**      | Custom 3D Printed or Machined Enclosure         | -         | 1   | Optional.                                        |

**3. Schematics / Connections (Textual Description)**

*   **Power:** USB 5V -> 3.3V LDO -> ESP32 VDD, Display VCC, SD Card VCC, I2C Connector VCC. Appropriate decoupling capacitors placed near power pins of ICs.
*   **ESP32 <-> Display (SPI Example):**
    *   ESP32 SPI CLK (e.g., GPIO 18) -> Display SCK
    *   ESP32 SPI MOSI (e.g., GPIO 23) -> Display MOSI/DIN/SDA
    *   ESP32 GPIO (e.g., GPIO 5) -> Display CS (Chip Select)
    *   ESP32 GPIO (e.g., GPIO 17) -> Display DC (Data/Command)
    *   ESP32 GPIO (e.g., GPIO 16) -> Display RST (Reset) - Optional if using software reset
*   **ESP32 <-> Display (I2C Example):**
    *   ESP32 I2C SDA (e.g., GPIO 21) -> Display SDA
    *   ESP32 I2C SCL (e.g., GPIO 22) -> Display SCL
*   **ESP32 <-> SD Card Module (SPI):**
    *   ESP32 SPI CLK (Shared w/Display if same bus, e.g., GPIO 18) -> SD SCK
    *   ESP32 SPI MOSI (Shared w/Display if same bus, e.g., GPIO 23) -> SD MOSI/DI
    *   ESP32 SPI MISO (e.g., GPIO 19) -> SD MISO/DO
    *   ESP32 GPIO (Unique, e.g., GPIO 4) -> SD CS (Chip Select)
*   **ESP32 <-> Keyboard Matrix:**
    *   ESP32 GPIO x 8 (Configurable, e.g., Rows R0-R7) -> Keyboard Rows (Outputs)
    *   ESP32 GPIO x 8 (Configurable, e.g., Cols C0-C7) -> Keyboard Columns (Inputs w/ Internal Pull-ups)
*   **ESP32 <-> External I2C Connector:**
    *   ESP32 I2C SDA (Configurable, e.g., GPIO 25) -> I2C Connector SDA Pin (with pull-up resistor to 3.3V)
    *   ESP32 I2C SCL (Configurable, e.g., GPIO 26) -> I2C Connector SCL Pin (with pull-up resistor to 3.3V)
    *   3.3V -> I2C Connector VCC Pin
    *   GND -> I2C Connector GND Pin

**4. Pin Mapping Table (Example - MUST be finalized & configurable in `/sd/config.json`)**

| Function         | ESP32 GPIO | Connected To        | Notes                                     |
| ---------------- | ---------- | ------------------- | ----------------------------------------- |
| **Display SPI**  |            |                     |                                           |
| DISP_SCK         | 18         | OLED SCK            | Shared SPI Bus (VSPI)?                     |
| DISP_MOSI        | 23         | OLED MOSI           | Shared SPI Bus (VSPI)?                     |
| DISP_CS          | 5          | OLED CS             | Unique                                    |
| DISP_DC          | 17         | OLED DC             | Unique                                    |
| DISP_RST         | 16         | OLED RST            | Unique (Optional)                         |
| **SD Card SPI**  |            |                     |                                           |
| SD_SCK           | 18         | SD Card SCK         | Shared SPI Bus (VSPI)?                     |
| SD_MOSI          | 23         | SD Card MOSI        | Shared SPI Bus (VSPI)?                     |
| SD_MISO          | 19         | SD Card MISO        | Unique to SPI Bus                         |
| SD_CS            | 4          | SD Card CS          | Unique                                    |
| **Keyboard**     |            |                     | Verify pins don't conflict with others! |
| KBD_ROW_0        | 32         | Keyboard Row 0      | Output                                    |
| KBD_ROW_1        | 33         | Keyboard Row 1      | Output                                    |
| KBD_ROW_2        | 27         | Keyboard Row 2      | Output                                    |
| KBD_ROW_3        | 14         | Keyboard Row 3      | Output                                    |
| KBD_ROW_4        | 12         | Keyboard Row 4      | Output                                    |
| KBD_ROW_5        | 13         | Keyboard Row 5      | Output                                    |
| KBD_ROW_6        | 15         | Keyboard Row 6      | Output                                    |
| KBD_ROW_7        | 2          | Keyboard Row 7      | Output                                    |
| KBD_COL_0        | 39         | Keyboard Column 0   | Input, Pull-up (Check if input-only pin) |
| KBD_COL_1        | 34         | Keyboard Column 1   | Input, Pull-up (Check if input-only pin) |
| KBD_COL_2        | 35         | Keyboard Column 2   | Input, Pull-up (Check if input-only pin) |
| KBD_COL_3        | 36         | Keyboard Column 3   | Input, Pull-up (Check if input-only pin) |
| KBD_COL_4        | 21         | Keyboard Column 4   | Input, Pull-up                            |
| KBD_COL_5        | 22         | Keyboard Column 5   | Input, Pull-up                            |
| KBD_COL_6        | 25         | Keyboard Column 6   | Input, Pull-up                            |
| KBD_COL_7        | 26         | Keyboard Column 7   | Input, Pull-up                            |
| **External I2C** |            |                     |                                           |
| EXT_I2C_SDA      | 25         | I2C Connector SDA   | With 4.7k Pull-up to 3.3V                 |
| EXT_I2C_SCL      | 26         | I2C Connector SCL   | With 4.7k Pull-up to 3.3V                 |
| *(Other)*        |            |                     |                                           |
| USB D+/D-        | -          | USB Connector       | Handled by ESP32 module's USB peripheral (if applicable) or external UART chip |
| Built-in LED?    | (e.g., 2)  | Onboard LED         | Optional for status indication            |

**5. Power Budget (Estimate)**

*   ESP32 (WiFi Active): ~150-300mA peak
*   OLED Display: ~20-80mA (content dependent)
*   SD Card (Write): ~50-100mA peak
*   External I2C Devices: Variable (User dependent)
*   **Total Estimated (excluding external I2C):** ~220-480mA (peaks higher).
*   **Recommendation:** Use a 3.3V LDO capable of >800mA, and ensure the 5V USB source can supply >1A.

**6. External I2C Connector Pinout (Example)**

1.  VCC (3.3V)
2.  GND
3.  SDA
4.  SCL

---

**Document 5: Testing Plan (Initial Draft)**

**Project:** ESP32 Scientific Programmable Networked Calculator (SPNC)
**Version:** 0.1
**Date:** 2023-10-27

**1. Introduction**

This document outlines the initial testing strategy for the SPNC project to ensure functionality, reliability, and adherence to requirements.

**2. Testing Levels**

*   **Level 1: Unit Testing:** Testing individual software modules or functions in isolation.
*   **Level 2: Integration Testing:** Testing the interaction and data flow between different modules.
*   **Level 3: System Testing:** Testing the fully assembled hardware and software system against the functional and non-functional requirements.
*   **Level 4: User Acceptance Testing (UAT):** Testing by potential end-users (if applicable) focusing on usability and real-world scenarios.

**3. Unit Testing Approach**

*   Focus on core logic modules where possible using Micropython's `unittest` or similar framework, potentially running tests on a host machine or the ESP32 itself.
*   **Key Modules for Unit Tests:**
    *   `math_engine`: Test evaluation of various expressions, operator precedence, function accuracy (vs standard library), DEG/RAD/GRAD conversions, variable storage/retrieval.
    *   `sandbox_exec`: **Critical.** Test sandbox restrictions (preventing forbidden builtins/imports), correct injection of `calculator_api`, exception handling and reporting. Test execution of simple valid/invalid scripts.
    *   `config_manager`: Test loading valid/invalid/missing config files, default value handling.
    *   Utility functions within managers (e.g., data parsing/formatting).

**4. Integration Testing Approach**

*   Test interactions between modules, often requiring running code on the actual ESP32 hardware.
*   **Key Integration Scenarios:**
    *   `calculator_api` <-> Backend Managers: Write scripts that call every function in `calculator_api` and verify the expected interaction with `display_manager`, `sd_manager`, `network_manager`, `i2c_manager`, `math_engine`.
    *   `main.py` <-> Mode Managers: Test transitions between modes, passing of control, and UI updates.
    *   Keyboard Input -> `main.py` Dispatch -> Module Action -> Display Update.
    *   SD Card Operations: Test saving/loading scripts, data, history, logs via the relevant managers/API calls. Verify file contents.
    *   Network Operations: Test WiFi connection, making HTTP GET/POST calls via `calculator_api`, handling responses and errors. Requires network access.
    *   I2C Operations: Connect known test I2C devices (e.g., another ESP32, EEPROM, sensor) and test `calculator_api.i2c_...` functions. Test scan, read, write, register access, error handling (NACK).

**5. System / Manual Testing Approach**

*   Execute end-to-end scenarios on the fully assembled calculator.
*   Use checklists based directly on the Functional Requirements (FR-*).
*   **Key Test Areas:**
    *   **Core Calculation:** Enter complex scientific calculations, verify results against known good calculator/tool. Test all functions, modes, variable usage.
    *   **Scripting:** Create scripts covering all `calculator_api` functions. Edit, save, load, run scripts. Test error reporting for script syntax/runtime errors. Test sandbox limitations.
    *   **UI/UX:** Verify all UI layouts match design. Test navigation flow, responsiveness, readability, status bar indicators.
    *   **SD Card:** Verify history logging, system logging (check file contents). Test file browser functionality. Test data saving/loading from scripts. Test behavior with SD card full/removed (if applicable).
    *   **Networking:** Test WiFi setup, API calls (Wikipedia, LLM), handling of network down/API errors.
    *   **I2C:** Connect sample I2C sensors/devices, write scripts to interact with them, verify data exchange, test I2C error handling.
    *   **Stress Testing:** Leave scripts running complex loops or network requests, perform rapid key presses, check for crashes or freezes. Test with large files/long history.
    *   **Configuration:** Test changing settings in `/sd/config.json` (Pins, Keys, WiFi, etc.) and verify behavior after reboot.

**6. Test Environment**

*   **Primary:** Actual SPNC hardware prototype.
*   **Secondary:** Host machine execution for pure-Python unit tests where possible.
*   **Network Access:** WiFi network with internet connectivity, potentially local HTTP/LLM test servers.
*   **I2C Test Devices:** Known-good I2C peripherals (e.g., temperature sensor, RTC, EEPROM).
