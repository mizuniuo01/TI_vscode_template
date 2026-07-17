# TI MSPM0 VS Code Development Environment Setup Guide

This document guides users through setting up an independent TI MSPM0 development environment on Windows / macOS based on **VS Code + standalone SysConfig + Make + Cortex-Debug**.

This workflow does not depend on Code Composer Studio (CCS). All core tools are installed and managed independently.

The template files referenced in this guide (`settings.json`, `tasks.json`, `keybindings.json`, etc.) are located in the `win/` or `mac/` directories of this repository.

---

# 1. Development Environment Components

The development environment consists of the following components:

| Component             | Purpose                                                |
| --------------------- | ------------------------------------------------------ |
| VS Code               | Development environment and workflow entry point       |
| VS Code Profile       | Manages platform-specific configuration                |
| MSPM0 SDK             | Provides device support, drivers and example code      |
| TI Arm Clang Compiler | Firmware compilation                                   |
| SysConfig standalone  | Graphical hardware configuration and source generation |
| Make                  | Build automation                                       |
| OpenOCD               | Flash programming and debug server                     |
| ARM GNU GDB           | Debug backend                                          |
| Cortex-Debug          | VS Code debug integration                              |

---

# 2. Preparation

> Before installing: if a tool listed below is already installed and working on your system, skip its section and move to the next one.

## 2.1 Install VS Code

Install the latest version of Visual Studio Code:

[https://code.visualstudio.com/](https://code.visualstudio.com/)

After installation, make sure VS Code starts correctly.

---

# 3. Install TI Official Tools

The following three tools are required for MSPM0 development.

Note:

**Do not use versions bundled with CCS.**

Install standalone versions instead.

---

## 3.1 Install MSPM0 SDK

Go to the TI website (ti.com) → find "MSPM0 SDK" → download the latest version.

After installation, record the SDK path.

Examples:

Windows:

```
D:/ti/mspm0_sdk_2_10_00_04/
```

macOS:

```
/Applications/ti/mspm0_sdk_2_10_00_04/
```

The SDK directory should contain:

```
source/
examples/
imports.mak
...
```

---

## 3.2 Install TI Arm Clang Compiler

Go to the TI website (ti.com) → find "TI Arm Clang Compiler" (ti-cgt-armllvm) → download the latest version.

Example paths:

Windows:

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

macOS:

```
/Applications/ti/ti-cgt-armllvm_5.1.0.LTS/
```

Note:

The configured path must point to the compiler root directory.

Correct:

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

Incorrect:

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/bin
```

---

## 3.3 Install Standalone SysConfig

Go to the TI website (ti.com) → find "SysConfig GUI" (standalone version, not the one bundled with CCS) → download the latest version.

Example paths:

Windows:

```
D:/ti/sysconfig_1.27.0/
```

macOS:

```
/Applications/ti/sysconfig_1.27.1/
```

Do not use the SysConfig bundled with CCS.

---

# 4. Install Platform-Specific Tools

### Windows

#### 4.1 Install Make (MinGW)

```powershell
winget install MSYS2.MSYS2
```

After installation, open the MSYS2 terminal (UCRT64) and install `make`:

```bash
pacman -S mingw-w64-x86_64-make
```

Add the MinGW `bin` directory to your system PATH (typically `C:\msys64\mingw64\bin`).

Verify:

```bash
mingw32-make --version
```

---

#### 4.2 Install ARM GNU Toolchain

Install ARM GNU Toolchain, which provides `arm-none-eabi-gdb` (TI toolchain does not include GDB).

```powershell
winget install Arm.GnuArmEmbeddedToolchain
```

Verify:

```bash
arm-none-eabi-gdb --version
```

---

#### 4.3 Install OpenOCD

Go to the OpenOCD website (openocd.org) → download the latest Windows release → extract to:

```
D:/OpenOCD/
```

The standard OpenOCD distribution already includes MSPM0 target support (`ti_mspm0.cfg`).

Verify:

```bash
openocd --version
```

---

### macOS

#### 4.4 Make

macOS provides Make by default.

Verify:

```bash
make --version
```

---

#### 4.5 GDB and OpenOCD

Current template status:

* macOS supports development and build verification
* Flash programming and debugging are not supported yet

Therefore, OpenOCD and ARM GDB are not required on macOS.

---

# 5. Create VS Code Profile

Open VS Code:

```
Gear icon (bottom-left)
    ↓
Profiles
    ↓
Create Profile
```

Create a new Profile.

Recommended name:

```
TI
```

Select:

```
Create Empty Profile
```

All following configurations will be stored in this Profile.

---

# 6. Install VS Code Extensions

Open the TI Profile and install the following extensions:

| Extension                                      | Purpose                           |
| ---------------------------------------------- | --------------------------------- |
| `ms-vscode.cpptools`                           | C/C++ IntelliSense                |
| `marus25.cortex-debug`                         | ARM Cortex-M debugging            |
| `ms-vscode.makefile-tools`                     | Makefile project support          |
| `xaver.clang-format`                           | C/C++ code formatting             |
| `mcu-debug.memory-view`                        | Memory viewer                     |
| `mcu-debug.peripheral-viewer`                  | Peripheral register viewer        |
| `mcu-debug.debug-tracker-vscode`               | Debug trace tracking              |
| `mcu-debug.rtos-views`                         | RTOS task/queue view              |
| `ti-development-tools.cortex-debug-dp-mspm0`   | MSPM0 SVD + OpenOCD configuration |
| `ti-development-tools.ti-embedded-development` | TI project templates              |

> If unrelated extensions such as `ti-embedded-debug` or `file-downloader` are installed, they can be removed.

---

# 7. Configure VS Code Profile

Copy the following files from the template:

```
settings.json
tasks.json
keybindings.json
```

Open:

```
Ctrl + Shift + P
```

Select:

```
Open User Settings (JSON)
```

Paste `settings.json` and replace the placeholders.

Use the same method for:

```
Open User Tasks
```

and:

```
Open Keyboard Shortcuts (JSON)
```

to configure `tasks.json` and `keybindings.json`.

> The templates provide a pre-configured setup — just replace the placeholder paths to get started. However, the configuration surface is large and the templates may not cover every scenario. Users should review and supplement the configuration as needed.

---

## 7.1 Replace Paths

Replace the following variables:

```
<path-to-ti-cgt-armllvm>
```

TI Arm Clang compiler path.

---

```
<path-to-mspm0-sdk>
```

MSPM0 SDK path.

---

Windows:

```
<path-to-openocd>
```

OpenOCD path.

---

Windows:

```
<path-to-arm-none-eabi-gcc>
```

ARM GNU Toolchain path.

---

Windows:

```
<path-to-home>/.clang-format
```

Path to the `.clang-format` file in the user directory. The formatting rules file must be configured by the user; it is not provided by this template.

macOS uses a fixed `clang-format.executable` path (`/opt/homebrew/bin/clang-format`) and does not require this placeholder.

---

# 8. Configure SDK `imports.mak`

Go to:

```
MSPM0 SDK root directory
```

Open:

```
imports.mak
```

Modify (`sysconfig_cli.sh` on macOS, `sysconfig_cli.bat` on Windows):

```makefile
SYSCONFIG_TOOL ?= <path-to-sysconfig>/sysconfig_cli.sh    # macOS
SYSCONFIG_TOOL ?= <path-to-sysconfig>/sysconfig_cli.bat   # Windows

TICLANG_ARMCOMPILER ?= <path-to-ti-cgt-armllvm>
```

Note:

`TICLANG_ARMCOMPILER`

must not include:

```
/bin
```

---

# 9. Create a Project

Copy `win/ticlang/makefile` (Windows) or `mac/ticlang/makefile` (macOS) to the `ticlang/` directory of your new project.

Create the following project structure:

```
MyProject/
├── main.c
├── project.syscfg
├── user/
│   ├── Inc/
│   └── Src/
└── ticlang/
    ├── makefile
    └── device_linker.cmd
```

---

# 10. Configure Makefile

Enter:

```
ticlang/
```

Modify (`sysconfig_cli.sh` on macOS, `sysconfig_cli.bat` on Windows):

```makefile
MSPM0_SDK_INSTALL_DIR ?= /path/to/mspm0_sdk_x_xx_xx_xx
TICLANG_ARMCOMPILER   := /path/to/ti-cgt-armllvm_x.x.x.LTS
SYSCONFIG_TOOL        := /path/to/sysconfig_x.xx.x/sysconfig_cli.sh
```

Note:

The project-level Makefile is platform-specific.

Do not share it between Windows and macOS.

It is recommended to add:

```
.gitignore
```

For example:

```
ticlang/makefile
```

A complete `.gitignore` reference:

```
*.obj
*.d
*.map
*.out
*.opt
*.dot
*.genlibs
ticlang/build/
ticlang/makefile
.DS_Store
```

---

# 11. Cross-Platform Git Workflow

This template enables cross-platform development through Profile isolation and Git sharing:

- **Project source files** (`user/`, `.syscfg`, `main.c`, `ticlang/device_linker.cmd`) are shared via Git. These files contain no local paths and are platform-independent.
- **Profile configuration** (`settings.json`, `tasks.json`, `keybindings.json`) contains absolute toolchain paths and is not committed. Windows and macOS each maintain their own Profile setup using the reference templates.
- **Makefile** is added to `.gitignore` and maintained locally on each machine.

After cloning a project on a new machine, simply copy the appropriate platform's Profile configuration and Makefile to start building immediately.

---

# 12. Verify the Environment

Before building, verify that all required tools are available:

```bash
mingw32-make --version        # Windows
make --version                # macOS
arm-none-eabi-gdb --version   # Windows
openocd --version             # Windows
```

If all commands output version information, the environment is ready.

---

# 13. First Build

Open the project.

Incremental build:

```
F7
```

Full rebuild:

```
F6
```

Clean:

```
F5
```

---

# 14. SysConfig Usage

Generate configuration source files:

```bash
make syscfg              # macOS
mingw32-make syscfg      # Windows
```

Open SysConfig GUI:

```bash
make syscfg-gui              # macOS
mingw32-make syscfg-gui      # Windows
```

Note:

SysConfig is the configuration tool.

`.syscfg` is the configuration file.

Relationship:

```
.syscfg
   |
   ↓
SysConfig GUI
   |
   ↓
ti_msp_dl_config.c/h
```

Do not manually modify generated files.

---

# 15. Flash and Debug (Windows)

Select the VS Code debug configuration:

Flash only:

```
MSPM0-download
```

Debug:

```
MSPM0-debug
```

Start:

```
F8
```

Stop debugging:

```
Shift + F8
```

Do not use the red stop button in the VS Code debug toolbar.

This may leave OpenOCD processes running in the background.

---

# 16. Switching Chips

When switching to a different target MCU, the `defines`, `device` and `svdFile` fields in the Profile configuration must be updated accordingly. See `CHIP_SWITCH.md` for details.

---

# 17. IntelliSense Configuration

This template uses:

```
Microsoft C/C++ IntelliSense
```

instead of clangd.

Reason:

clangd is based on the Clang frontend and is not the same compiler frontend as tiarmclang.

Parsing TI/GCC-specific features across different compiler environments, such as:

* `__attribute__`
* compiler built-in macros
* inline assembly

may generate many false-positive diagnostics.

Microsoft C/C++ IntelliSense uses `compilerPath` to query tiarmclang directly, automatically obtaining the real built-in macros and system include paths, providing better compatibility with the TI compiler environment.

---

Configuration:

```
C_Cpp.default.compilerPath
```

should point to:

```
tiarmclang
```

Also:

```
intelliSenseMode
```

should be set to:

```
gcc-arm
```

---

# 18. Hardware Notes

## 18.1 SWD Debug Pins

MSPM0 uses the following default SWD pins:

| Pin  | Function |
| ---- | -------- |
| PA19 | SWDIO    |
| PA20 | SWCLK    |

Do not assign these pins to other functions in SysConfig.

Otherwise, the debugger may no longer be able to connect to the device.

---

## 18.2 Configure Unused Pins

It is recommended to enable:

```
Board → Configure Unused Pins
```

in SysConfig.

This prevents floating pins from causing unnecessary power consumption.

---

# 19. Common Issues

## Q: Command-line tools not found (`mingw32-make`, `arm-none-eabi-gdb`, `openocd` not recognized)

After installation, the tool's `bin` directory must be added to the system PATH. Verify with `mingw32-make --version` in a new terminal. If not recognized, reinstall or manually add the path to system environment variables.

## Q: `tiarmclang` not found during build

Check:

```
TICLANG_ARMCOMPILER
```

path.

Make sure:

* The path points to the compiler root directory
* `/bin` is not appended

Correct:

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

Incorrect:

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/bin
```

---

## Q: SysConfig GUI cannot start

Check:

* Standalone SysConfig is installed
* CCS bundled SysConfig is not being used
* `SYSCONFIG_TOOL` path is correct

---

## Q: Newly added `.c` files are not compiled

Make sure source files are located under:

```
user/Src/
```

If subdirectories are used, update the Makefile:

```
USER_C_FILES
CFLAGS
VPATH
```

---

## Q: Windows build fails with `-rf` error

Reason:

The SDK default Makefile uses Unix-style clean commands.

On Windows:

```
$(RM) -rf
```

cannot execute correctly.

Replace with:

```makefile
$(RMDIR)
```

---

## Q: Makefile tracked by Git causes Windows/macOS conflicts

The project-level Makefile is platform-specific.

Add:

```
ticlang/makefile
```

to:

```
.gitignore
```

If it has already been tracked by Git:

```bash
git rm --cached ticlang/makefile
```

---

## Q: OpenOCD remains after debugging exits

Always exit debugging with:

```
Shift + F8
```

Do not click the red stop button in the VS Code debug toolbar.

If OpenOCD remains running:

On Windows:

Open Task Manager and terminate:

```
openocd.exe
```

---

## Q: Flash fails with `XDS110 failed to connect`

Check:

1. XDS110 USB connection
2. OpenOCD path configuration
3. Whether the installed OpenOCD supports MSPM0

Confirm:

```
cortex-debug.openocdPath
```

points to:

```
D:/OpenOCD/bin/openocd.exe
```

---

# 20. Completion

After completing the above steps, the MSPM0 independent development workflow is ready:

```
VS Code
+
TI Arm Clang
+
SysConfig
+
Make
+
OpenOCD
+
GDB
```

This environment provides a fully transparent workflow for:

* Project configuration
* Build process
* SysConfig generation process
* Flash programming
* Debugging

while keeping the entire workflow independent from CCS.
