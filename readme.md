**Project Title:** ESP32 Scientific Programmable Networked Calculator (SPNC)

**Version:** 0.1 (Consolidated Requirements with I2C & Micropython Scripting)

**Date:** 2023-10-27

**1. Introduction & Vision**

The SPNC aims to be a powerful, open-source handheld calculator built using ESP32 hardware, specifically targeting **ESP32 variants with PSRAM**. It combines standard scientific calculation capabilities with user programmability via **scripts written in a sandboxed subset of Micropython syntax**. Persistent storage via SD card is mandatory for scripts, data, history, and configuration. Network connectivity enables access to external APIs (e.g., Wikipedia, LLMs), callable from user scripts. The primary input method is an 8x8 matrix keyboard, and output is displayed on a **256x128 pixel monochrome OLED screen**. The device supports interfacing with external sensors and peripherals via a dedicated **I2C bus**. The device is intended to be tethered (powered via USB).

**2. Goals & Objectives**

* Provide a functional scientific calculator meeting standard mathematical needs.
* Enable users to write, save, load, and execute custom **Micropython scripts (`.py` files)** within a **secure, sandboxed environment**.
* Utilize an SD card for mandatory persistent storage of scripts, user data, calculation history, configuration, and system logs.
* Connect to WiFi networks to access defined external web APIs, callable from user scripts.
* Offer a clear, responsive, and informative user interface leveraging the larger 256x128 display.
* Allow user scripts to **monitor and control external devices via an I2C interface**.
* Maintain a modular and well-documented codebase in Micropython for extensibility.
* Provide a familiar and powerful Pythonic programming environment for users.

**3. Non-Goals (Out of Scope for Initial Versions)**

* Real-time graphical function plotting.
* Computer Algebra System (CAS) capabilities.
* **Unrestricted Micropython execution** outside the defined sandbox.
* Battery operation and power management optimization.
* Advanced Python debugging tools within the programming environment (basic error reporting only).
* Direct hardware manipulation (GPIO, SPI, internal peripherals) from user scripts (except via the dedicated, controlled I2C API).
* Support for multiple simultaneous I2C buses (one primary external bus).

**4. Hardware Requirements**

* **HR-PROC-01:** **Processor:** ESP32 variant **with integrated PSRAM is mandatory** (e.g., ESP32-WROVER, ESP32-S3 with sufficient PSRAM) to handle OS, display buffer, networking, and script execution demands.
* **HR-DISP-01:** **Display:** **256x128 Monochrome OLED**. Specify interface (SPI preferred for performance, I2C acceptable) and ensure a compatible Micropython driver library exists and is verified (e.g., for SSD1322, SH1107, or similar controllers).
* **HR-INPT-01:** **Input:** Custom 8x8 (64 keys) Matrix Keyboard.
* **HR-STOR-01:** **Storage:** Micro SD Card Module (SPI interface). **SD card presence is mandatory for full operation.**
* **HR-CONN-01:** **Connectivity:** Built-in WiFi on the ESP32 module.
* **HR-POWR-01:** **Power:** USB Tethered (5V supply). Onboard 3.3V regulation required for ESP32, display, SD card.
* **HR-I2C-01:** **External I2C Bus:** Dedicated ESP32 I2C peripheral/pins (configurable) routed to an external connector/header for user devices. Pull-up resistors on SDA/SCL lines are required (configurable value, typically 4.7kOhm).
* **HR-PINS-01:** **Pin Assignments:** All GPIO pin assignments for Keyboard Rows/Cols, Display, SD Card, and the External I2C Bus must be clearly defined and configurable, likely within `/sd/config.json`. Ensure no pin conflicts.

**5. Functional Requirements**

**5.1. Core Calculator Mode**
    *   **FR-CALC-01:** Support standard arithmetic operations: +, -, *, /, %.
    *   **FR-CALC-02:** Support calculation with proper operator precedence and parentheses: (, ).
    *   **FR-CALC-03:** Support standard scientific functions (Trig, Hyperbolic, Log, Exp, Power, Sqrt, Constants, Number Manipulation, Factorial - see v0.1 for full list).
    *   **FR-CALC-04:** Support DEG, RAD, GRAD modes for trigonometric functions, with clear indication on the display.
    *   **FR-CALC-05:** Provide memory functions: Store (STO), Recall (RCL) for multiple variables (e.g., A-Z, Ans).
    *   **FR-CALC-06:** Maintain an "Ans" (Last Answer) variable.
    *   **FR-CALC-07:** Handle numerical input (int, float, scientific notation).
    *   **FR-CALC-08:** Provide clear error reporting for mathematical and syntax errors.
    *   **FR-CALC-09:** The UI should leverage the 256x128 display for enhanced context (e.g., multi-line input/output, variable watch area).

**5.2. Programming Mode (Micropython Scripting)**
    *   **FR-PROG-01:** Allow entry and line-by-line editing of scripts using **Micropython syntax**.
    *   **FR-PROG-02:** Scripts must be savable/loadable as `.py` text files to/from `/sd/scripts/` via a file browser UI.
    *   **FR-PROG-03:** Allow execution of `.py` scripts using a **sandboxed execution environment** (`sandbox_exec.py`).
    *   **FR-PROG-04:** Display script execution status clearly (Running, Paused, Done, Error + Line).
    *   **FR-PROG-05:** Provide scripts access to calculator state and functionality via a **well-defined `calculator_api` module** (see Section 8).
    *   **FR-PROG-06:** Catch and display Python exceptions from user scripts, including script name and line number.
    *   **FR-PROG-07:** The sandboxing mechanism must restrict access to harmful builtins and prevent direct hardware access (except via `calculator_api.i2c_...` functions). Define allowed builtins/imports (see Section 8).
    *   **FR-PROG-08:** Implement the API functions callable from user scripts as defined in Section 8.

**5.3. Display & User Interface**
    *   **FR-UI-01:** Utilize the **256x128 pixel monochrome OLED display** effectively.
    *   **FR-UI-02:** Implement distinct modes (e.g., CALC, SCRIPT-EDIT, SCRIPT-RUN, FILE-BROWSE, API-WIKI, API-LLM, I2C-MONITOR?) with clear status bar indication.
    *   **FR-UI-03:** Status bar displays: Mode, WiFi status, DEG/RAD/GRAD, SHIFT/FN status, current script name (if applicable).
    *   **FR-UI-04:** Display calculator input expression clearly (multi-line/scrolling).
    *   **FR-UI-05:** Display calculation results clearly (potentially with more precision or context).
    *   **FR-UI-06:** Display script listings with line numbers, maximizing vertical space for context.
    *   **FR-UI-07:** Provide a functional file browser for `/sd/scripts/` and `/sd/data/`.
    *   **FR-UI-08:** Display multi-line text results from API calls or script outputs effectively.
    *   **FR-UI-09:** Provide visual feedback for key presses.
    *   **FR-UI-10:** Use clear, readable fonts, potentially configurable or context-dependent (larger fonts possible).
    *   **FR-UI-11:** Design UI layouts that utilize the vertical space (e.g., variable watch pane, persistent history view).

**5.4. Keyboard Input**
    *   **FR-KEY-01:** Accurately scan and decode the 8x8 matrix keyboard.
    *   **FR-KEY-02:** Implement modifier keys (e.g., SHIFT, FN, ALPHA) to access the full range of functions, programming keywords, Python symbols (`:`, `_`, `"` etc.), and variable names.
    *   **FR-KEY-03:** Provide essential editing keys: Backspace/CE, AC.
    *   **FR-KEY-04:** Provide navigation keys (UP, DOWN, LEFT, RIGHT, ENTER/OK, ESC/BACK).
    *   **FR-KEY-05:** Key mapping must be clearly defined and configurable (e.g., in `/sd/config.json`). A dedicated UI for viewing the keymap might be useful.

**5.5. SD Card Storage (Mandatory)**
    *   **FR-SD-01:** Detect SD card presence on boot. Display persistent error and enter limited mode if absent/failed.
    *   **FR-SD-02:** Store user scripts as `.py` files in `/sd/scripts/`.
    *   **FR-SD-03:** Store calculation history persistently (e.g., `/sd/history.jsonl`). Implement rotation/size limiting.
    *   **FR-SD-04:** Store system logs persistently (e.g., `/sd/calculator.log`). Implement rotation/size limiting.
    *   **FR-SD-05:** Allow scripts to save/load data via API calls (`calculator_api.save_data`, `load_data`) to/from `/sd/data/` (defaulting to JSON format).
    *   **FR-SD-06:** Store configuration (WiFi, API keys/endpoints, I2C settings, keymaps) in `/sd/config.json`.

**5.6. Networking & API Integration**
    *   **FR-NET-01:** Configure WiFi via `/sd/config.json`.
    *   **FR-NET-02:** Connect to WiFi automatically or on demand.
    *   **FR-NET-03:** Display WiFi connection status (e.g., icon, IP address).
    *   **FR-NET-04:** Implement generic HTTP/S request functions accessible via `calculator_api`.
    *   **FR-NET-05:** Implement Wikipedia and LLM API interaction logic within `calculator_api`. Securely load API keys from config.
    *   **FR-NET-06:** Provide user feedback during network operations and handle errors gracefully.

**5.7. External I2C Device Support**
    *   **FR-I2C-01:** Configure the external I2C bus parameters (pins, frequency) via `/sd/config.json`.
    *   **FR-I2C-02:** Provide an API function (`calculator_api.i2c_scan()`) for user scripts to detect connected device addresses.
    *   **FR-I2C-03:** Provide API functions (`calculator_api.i2c_read(...)`, `calculator_api.i2c_write(...)`) for user scripts to communicate with devices at specified addresses, reading/writing byte data or registers.
    *   **FR-I2C-04:** Implement basic error handling for I2C communication (e.g., device not responding NACK).
    *   **FR-I2C-05:** Optionally allow defining an "I2C Device Map" (see Section 9) in `/sd/config.json` or `/sd/i2c_map.json` to allow scripts to reference devices/registers by name instead of raw addresses. The API should support using these names.

**6. Non-Functional Requirements**

* **NFR-PERF-01:** UI responsiveness is paramount.
* **NFR-PERF-02:** Scientific calculations must be near-instantaneous.
* **NFR-PERF-03:** Script execution performance should be reasonable; long operations (I2C, network) should provide feedback and not permanently freeze the UI.
* **NFR-PERF-04:** Network/I2C calls require timeouts.
* **NFR-USAB-01:** Key mappings need careful design for both calculation and coding efficiency.
* **NFR-USAB-02:** Consistent UI navigation.
* **NFR-RELI-01:** Graceful error handling across all operations (Math, Python, File I/O, Network, I2C).
* **NFR-RELI-02:** Robust SD card handling.
* **NFR-RELI-03:** I2C operations initiated by user scripts should be handled such that bus errors or misbehaving external devices minimally impact core calculator stability (e.g., use timeouts, catch exceptions).
* **NFR-SECU-01:** Secure loading of API keys from `/sd/config.json`.
* **NFR-SECU-02:** **User script execution environment MUST be strictly sandboxed.** Prevent access to unauthorized builtins, modules, file paths, and hardware peripherals (except via the defined `calculator_api`).
* **NFR-MAINT-01:** Modular codebase.
* **NFR-MAINT-02:** Adequate code commenting and documentation.

**7. System Architecture**

**7.1. Hardware Summary**
    *   ESP32 with PSRAM (Mandatory)
    *   256x128 OLED Display (SPI or I2C)
    *   8x8 Matrix Keyboard
    *   Micro SD Card Module (SPI)
    *   External I2C Header/Connector

**7.2. Software Modules (Micropython)**
    *   `main.py`: Initialization, main loop, mode dispatcher.
    *   `config_manager.py`: Loads and manages configuration from `/sd/config.json` (Pins, WiFi, APIs, I2C settings, Keymap, I2C Map).
    *   `keyboard_manager.py`: Matrix scanning, key decoding based on keymap from config, modifier state.
    *   `display_manager.py`: OLED driver wrapper (for 256x128), UI layout engine, font management.
    *   `math_engine.py`: Handles interactive expression evaluation, DEG/RAD/GRAD logic, shared `variables` dict.
    *   `script_manager.py`: Script editor UI, loading/saving `.py` files from `/sd/scripts/`, initiating script execution.
    *   `sandbox_exec.py`: **Crucial.** Implements the sandboxed `exec()` environment, restricts builtins/imports, injects `calculator_api`.
    *   `calculator_api.py`: **Defines the functions exposed to user scripts.** Acts as a facade to other modules.
    *   `network_manager.py`: WiFi connection, generic HTTP/S requests (via `urequests`).
    *   `sd_manager.py`: Manages history log, system log, user data files (`/sd/data/`).
    *   `i2c_manager.py`: Manages the external I2C bus based on config, provides core read/write/scan functions used by `calculator_api`.
    *   `lib/`: Contains drivers (OLED, `sdcard.py`) and libraries (`urequests.py`, `ujson.py`, etc.).

**8. Programmable API (Environment for User `.py` Scripts)**

Executed via `sandbox_exec.py`. Provides:

* **Sandboxed Execution:** `exec()` with restricted environment.
* **Allowed Builtins:** Restricted list (see FR-PROG-07).
* **Allowed Imports:** `math`, `random`, `ujson`, `time`, `calculator_api`.
* **`calculator_api` Module Functions:**
  * *Variables:* `get_variable(name)`, `set_variable(name, value)`, `variables_dict()` (returns copy).
  * *Display:* `display(text_or_value)`, `clear_display()`, `display_line(line_num, text)`.
  * *Input:* `prompt_number(message)`, `prompt_string(message)`.
  * *SD Card Data:* `save_data(filename, data_object)` (uses ujson), `load_data(filename)` (uses ujson), `delete_data(filename)`, `list_data_files()`.
  * *Networking:* `wiki_search(term)`, `llm_query(prompt)`, `http_get(url, **kwargs)`, `http_post(url, **kwargs)`.
  * *I2C:* `i2c_scan()` (returns list of addresses), `i2c_read(addr, nbytes_or_reg, ...)` (flexible read), `i2c_write(addr, data_bytes_or_reg, ...)` (flexible write), `i2c_read_reg(device_name_or_addr, reg_name_or_addr, nbytes)`, `i2c_write_reg(device_name_or_addr, reg_name_or_addr, data)`.
  * *System:* `delay_ms(ms)`, `get_mode()`, `set_mode(mode)`, `log_message(msg)`, `get_timestamp()`.

**9. I2C Device Map**

* To simplify script interaction with known I2C devices, an optional mapping can be defined in `/sd/config.json` or `/sd/i2c_map.json`.
* This allows scripts to use symbolic names instead of raw addresses and register numbers via the `calculator_api.i2c_read/write_reg` functions.
* **Example `i2c_map.json` Structure:**
  
  ```json
  {
    "TEMP_SENSOR": {
      "addr": "0x48", // or 72
      "registers": {
        "READ_TEMP": "0x00", // Register to read temperature
        "CONFIG": "0x01"
      },
      "protocol": "standard" // or specific device type if needed
    },
    "RTC": {
      "addr": "0x68",
      "registers": {
        "SECONDS": "0x00",
        "MINUTES": "0x01",
        "HOURS": "0x02"
      }
    }
  }
  ```
* The `i2c_manager.py` and `calculator_api.py` will use this map to translate names to addresses/registers.

**10. Data Formats (SD Card)**

* **Scripts:** `/sd/scripts/*.py` (UTF-8 Micropython text files).
* **History:** `/sd/history.jsonl` (JSON Lines format).
* **System Log:** `/sd/calculator.log` (Timestamped plain text).
* **User Data:** `/sd/data/*` (Default JSON via `ujson` for API calls).
* **Configuration:** `/sd/config.json` (JSON format).
* **I2C Map (Optional):** `/sd/i2c_map.json` (JSON format).

**11. UI/UX Concepts (High-Level)**

* Multi-pane layouts possible (e.g., main output + variable watch).
* Clear visual hierarchy using spacing, lines, and potentially font variations.
* Dedicated I2C monitor mode for raw bus interaction/debugging?
* File browser with basic file operations (view, run, delete).

**12. Future Considerations / Roadmap**

* Refining the `calculator_api` based on usage.
* Enhancing the script editor (syntax highlighting if feasible?).
* Adding more standard Python modules to the sandbox whitelist (e.g., `ure`).
* Advanced plotting capabilities.
* Unit conversions module/API.
* More sophisticated file manager (subdirectories?).
* Investigating `asyncio` for non-blocking I/O in scripts.
* Support for specific complex I2C device protocols via the API.
