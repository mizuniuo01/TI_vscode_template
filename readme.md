# MSPM0 Project Template for VS Code

> A transparent and customizable VSCode-based development workflow for TI MSPM0 series microcontrollers.

A minimal and reproducible project template based on VSCode, TI development tools and an independent embedded toolchain.

It provides reference configurations and project templates rather than a single fixed application project.

This template includes:

- TI Arm Clang Compiler  
- SysConfig (standalone)  
- MSPM0 SDK  
- OpenOCD + GDB (ARM GNU Toolchain)  
- Make (MinGW on Windows, system make on macOS)

It is designed to work without Code Composer Studio (CCS), providing a lightweight, layered and fully controllable development workflow.

The template supports both Windows and macOS through profile-level VSCode configuration that can be reused across multiple MSPM0 projects.

Windows provides the complete build, flash and debug workflow.

macOS is currently supported for project development and build verification only. Flash and debug support on macOS is not included yet.

This template is intended for embedded developers who prefer a clean and IDE-independent workflow.

---

## Highlights

- **Fully independent toolchain**
  All components (compiler, SysConfig, SDK, Make, GDB, OpenOCD) are installed and managed separately, without relying on CCS.

- **CCS-free development workflow**  
  Build, flash and debug are implemented using Make, OpenOCD and Cortex-Debug, providing a lightweight and fully controllable environment.

- **Template-driven project setup**
  Provides reusable project templates and reference configurations with pre-configured SysConfig integration and build system setup.

- **VSCode profile-based configuration**  
  Toolchain paths, tasks and keybindings are managed at profile level and can be reused across multiple projects.

- **Cross-platform development support (Windows / macOS)**
  Both platforms provide a consistent development experience, with Windows supporting the complete build, flash and debug workflow, while macOS currently focuses on project development and build verification.

- **Platform-independent project structure**
  The workflow is portable because both platforms share the same core toolchain components and project structure.
  Platform-specific differences are isolated into separate VSCode profiles and configuration files, while project files remain consistent. This allows projects to be synchronized and developed across platforms through Git.

- **Customizable workflow**
  The provided configuration serves as both a ready-to-use setup and a reference design.
  Users can customize keybindings, tasks, build options, toolchain paths and other VSCode configurations according to their own workflow.

- **Customized Makefile**  
  The project Makefile is based on the TI SDK template but heavily customized: toolchain paths use `:=` overrides before `include imports.mak` to enforce standalone installations; `user/Src/` and `user/Inc/` are added to the source and include paths; all build outputs (`.obj`, `.out`, `.map`) are redirected to `ticlang/build/` instead of scattering alongside the makefile; Windows directory cleanup uses `$(RMDIR)` instead of Unix `rm -rf`; and `wildcard` / `VPATH` enable automatic source discovery under `user/Src/`.

- **Clear debug architecture**  
  Based on VSCode + GDB + OpenOCD pipeline, enabling standard SWD debugging without vendor IDE dependency.

---

## Project Structure

> Project files are generated from the reference configuration and maintained locally. Platform-specific build files should not be shared directly between Windows and macOS.

The repository is organized to separate platform-specific configuration, reusable reference configurations and documentation.

- `win/`  
  VSCode profile configuration for Windows, including settings, tasks, keybindings, and toolchain-specific build files.

- `mac/`  
  VSCode profile configuration for macOS, aligned with the Windows setup with minimal platform-specific differences.

- `CHIP_SWITCH.md`  
  Quick reference for switching between different MSPM0 devices.

- `GUIDE.md`  
  English setup and usage guide, covering detailed configuration and onboarding.

- `GUIDE(CHS).md`  
  Chinese version of the setup and usage guide.

> The `win/` and `mac/` Profile configurations provide pre-configured templates with placeholder paths. Users only need to replace the paths to get started. However, the configuration surface is large and the templates may not cover every setting — users should review and supplement the configuration as needed.

---

## Toolchain

This workflow separates vendor-specific development tools from the build and debug infrastructure.

The development environment is based on a fully independent toolchain composed of the following components:

- **MSPM0 SDK**  
  Provides device headers, drivers and example code.

- **SysConfig (standalone)**  
  Provides graphical hardware configuration. The SysConfig CLI is invoked during the build process to generate device initialization source files.

- **TI Arm Clang Compiler (tiarmclang)**  
  Used for building firmware.

- **Make**  
  Handles project build automation (MinGW on Windows, system make on macOS).

- **GDB (ARM GNU Toolchain)**  
  Used as the debugging backend.

- **OpenOCD**  
  Provides flash programming and debug server support.

This setup avoids dependency on Code Composer Studio (CCS) and enables a fully controllable build and debug workflow.

---

## Workflow Overview

The workflow separates the development environment into independent layers:

- VSCode provides the user interface and workflow orchestration.
- VSCode tasks and profiles manage project-level operations.
- The project Makefile (`ticlang/makefile`) drives the build process: it invokes SysConfig CLI for code generation, compiles sources under `user/Src/` with headers from `user/Inc/`, and outputs build artifacts to `ticlang/build/`.
- TI tools and open-source tools provide compilation, SysConfig-based configuration generation and debugging.

The workflow keeps each component visible and independently managed, allowing users to customize, replace or extend individual components.

---

## Cross-Platform Workflow

This template is designed for cross-platform development through Git:

- **Project source files** — shared via Git. All source files under `user/`, `.syscfg` configuration files, and `ticlang/device_linker.cmd` are platform-independent and can be committed directly.
- **Profile configuration** — machine-local, not committed. `settings.json`, `tasks.json` and `keybindings.json` contain absolute toolchain paths and are maintained separately on each machine using the `win/` and `mac/` reference templates.
- **Makefile** — platform-specific, added to `.gitignore`. The Makefile is copied from `win/ticlang/` or `mac/ticlang/` and customized with local tool paths. Each platform maintains its own copy.

This separation means a project can be cloned on either Windows or macOS, and after copying the appropriate profile configuration and Makefile, it is ready to build immediately.

---

## VSCode Integration

This workflow uses VSCode as the primary development environment.

The workflow uses several VSCode extensions as integration layers:

- Cortex-Debug
- C/C++ Extension
- Makefile Tools
- clang-format
- TI MSPM0 extensions

These extensions connect VSCode with the underlying tools, while the toolchain itself remains independently managed and replaceable.

> The `.clang-format` formatting rules file is not provided by this template. Users should configure it according to their own coding style.

### IntelliSense

The template uses Microsoft C/C++ IntelliSense instead of clangd.

The IntelliSense compiler configuration points to TI Arm Clang, allowing VSCode to correctly resolve embedded headers, compiler macros and device-specific definitions.
This avoids inconsistencies caused by using a different compiler configuration, especially when handling compiler-specific macros and embedded SDK headers.

---

## Quick Start

For detailed installation and configuration instructions, please refer to:
- [GUIDE.md](GUIDE.md) (English)
- [GUIDE(CHS).md](GUIDE(CHS).md) (Chinese)

Basic workflow:

1. Install required dependencies.
2. Configure the VSCode profile.
3. Create a new project based on the provided reference configuration.
4. Build and debug using VSCode (Windows), or build and verify the project (macOS).

### New Project Setup

1. Copy `win/ticlang/makefile` (Windows) or `mac/ticlang/makefile` (macOS) to your new project's `ticlang/` directory. Name the project folder consistently with the `.syscfg` file.
2. Open the `.syscfg` file with the SysConfig GUI to configure pins, clocks and peripherals. Generate device configuration code (`mingw32-make syscfg` on Windows, `make syscfg` on macOS — or let the build process invoke it automatically).
3. Add application source files under `user/Src/` and headers under `user/Inc/`. The Makefile uses `wildcard` and `VPATH` to automatically discover new `.c` files.
4. Build with F7. On Windows, flash and debug with F8.

---

## Usage

### Build

- **F7** — Incremental build  
- **F6** — Full rebuild (clean + build)  
- **F5** — Clean build artifacts  

### Flash & Debug

- **F8** — Flash or start debugging (Windows only)
- Select `MSPM0-download` for flashing only (Windows only)
- Select `MSPM0-debug` for debugging (Windows only)

- **Shift+F8** — Stop debugging session (Windows only)

### Workflow

All operations are executed through VSCode tasks and keybindings defined in the project profile.

These tasks internally invoke the build system (Make) and debug tools (OpenOCD + GDB).
The SysConfig CLI is invoked by Make to generate device configuration source files from the `.syscfg` configuration file.

---

## Debug Architecture

The debug pipeline is built on a standard Cortex-M toolchain, orchestrated through VSCode:

```text
VSCode + Cortex-Debug
        ↓
       GDB
        ↓
     OpenOCD
        ↓
       SWD
        ↓
      MSPM0
```

- **VSCode + Cortex-Debug**  
  Acts as the orchestration and integration layer, providing a frontend for launching and managing debug sessions.

- **GDB**  
  Handles breakpoints, stepping and state inspection.

- **OpenOCD**  
  Serves as the debug server, connecting GDB to the target via SWD.

- **SWD (Serial Wire Debug)**  
  Physical interface to the MSPM0 device.

The VSCode debug extensions provide integration between the user workflow and the underlying tools, while keeping the complete development pipeline visible and configurable.

---

## FAQ

### Toolchain & Build

**Q: Command-line tools not found (`mingw32-make`, `arm-none-eabi-gdb`, `openocd` not recognized)**  
These tools must be added to the system PATH after installation. On Windows, verify with `where mingw32-make` or `mingw32-make --version` in a new terminal. If the command is not recognized, reinstall the tool or manually add its `bin` directory to the system environment variables.

**Q: Compiler not found (`tiarmclang` not found)**  
Check `TICLANG_ARMCOMPILER` path in your configuration. Make sure the path does not include `/bin`.

**Q: Build fails on Windows with `-rf` errors**  
Some SDK makefiles use Unix-style clean commands. Replace `$(RM) -rf` with `$(RMDIR)` in the template makefile.

---

### SysConfig

**Q: SysConfig GUI fails to launch or generated source files are missing**
Ensure you are using the standalone SysConfig installation instead of the version bundled in CCS, and verify that the SysConfig CLI path is correctly configured.

**Q: What is the difference between SysConfig and `.syscfg` files?**  
SysConfig is the graphical configuration tool provided by TI.  
The `.syscfg` file is the project configuration file created by SysConfig. During the build process, the SysConfig CLI reads the `.syscfg` file and generates device configuration source files.

---

### Project Structure

**Q: Newly added `.c` files are not compiled**  
Make sure source files are placed under the expected directory (e.g. `user/Src/`).  
If subdirectories are used, update the corresponding paths in the makefile.

**Q: Makefile conflicts between Windows and macOS**  
Do not share project-level makefiles across platforms. Keep them local and exclude them via `.gitignore`.

---

### Debug & Flash

**Q: Debug session does not exit cleanly**  
Always stop debugging using `Shift+F8`.  
Improper termination may leave OpenOCD processes running in the background.

**Q: Flash failed (`XDS110: failed to connect`)**  
Check USB connection and ensure OpenOCD is correctly configured and accessible.

---

### General

**Q: Why not use a fully integrated vendor IDE workflow?**
This project uses VSCode extensions as workflow integration components while keeping the underlying toolchain independent and transparent.
Core components such as build, debug and SysConfig-based configuration generation are explicitly defined and can be inspected or modified.