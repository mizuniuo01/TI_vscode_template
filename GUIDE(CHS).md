# TI MSPM0 VS Code 开发环境配置指南

本文档用于指导 Windows / macOS 平台搭建基于 **VS Code + 独立 SysConfig + Make + Cortex-Debug** 的 TI MSPM0 独立开发环境。

本配置流程不依赖 Code Composer Studio (CCS)，所有核心工具均独立安装和管理。

本指南中引用的模板文件（`settings.json`、`tasks.json`、`keybindings.json` 等）位于本仓库的 `win/` 或 `mac/` 目录下。

---

# 1. 环境组成

开发环境由以下组件组成：

| 组件                    | 作用             |
| --------------------- | -------------- |
| VS Code               | 开发环境与工作流入口     |
| VS Code Profile       | 管理平台相关配置       |
| MSPM0 SDK             | 提供芯片支持、驱动和示例代码 |
| TI Arm Clang Compiler | 固件编译           |
| SysConfig standalone  | 图形化硬件配置与源码生成   |
| Make                  | 构建自动化          |
| OpenOCD               | 烧录与调试服务器       |
| ARM GNU GDB           | 调试后端           |
| Cortex-Debug          | VS Code 调试集成   |

---

# 2. 安装前准备

> 注意：如果下列工具已经安装且可用，跳过对应章节，继续下一项即可。

## 2.1 安装 VS Code

安装最新版 Visual Studio Code：

[https://code.visualstudio.com/](https://code.visualstudio.com/)

安装完成后确认 VS Code 可以正常启动。

---

# 3. 安装 TI 官方工具

以下三个工具是 MSPM0 开发必须组件。

注意：

**不要使用 CCS 内置版本。**

建议全部安装独立版本。

---

## 3.1 安装 MSPM0 SDK

前往 TI 官网（ti.com）→ 找到 "MSPM0 SDK" → 下载最新版本。

安装完成后记录 SDK 路径。

示例：

Windows：

```
D:/ti/mspm0_sdk_2_10_00_04/
```

macOS：

```
/Applications/ti/mspm0_sdk_2_10_00_04/
```

SDK 目录应包含：

```
source/
examples/
imports.mak
...
```

---

## 3.2 安装 TI Arm Clang Compiler

前往 TI 官网（ti.com）→ 找到 "TI Arm Clang Compiler"（ti-cgt-armllvm）→ 下载最新版本。

示例路径：

Windows：

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

macOS：

```
/Applications/ti/ti-cgt-armllvm_5.1.0.LTS/
```

注意：

配置路径时填写编译器根目录。

正确：

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

错误：

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/bin
```

---

## 3.3 安装独立 SysConfig

前往 TI 官网（ti.com）→ 找到 "SysConfig GUI"（独立版，非 CCS 内置版本）→ 下载最新版本。

示例路径：

Windows：

```
D:/ti/sysconfig_1.27.0/
```

macOS：

```
/Applications/ti/sysconfig_1.27.1/
```

不要使用 CCS 中附带的 SysConfig。

---

# 4. 安装平台相关工具

### Windows

#### 4.1 安装 Make（MinGW）

```powershell
winget install MSYS2.MSYS2
```

安装完成后，打开 MSYS2 终端（UCRT64），执行：

```bash
pacman -S mingw-w64-x86_64-make
```

将 MinGW 的 `bin` 目录加入系统 PATH（通常为 `C:\msys64\mingw64\bin`）。

验证：

```bash
mingw32-make --version
```

---

#### 4.2 安装 ARM GNU Toolchain

安装 ARM GNU Toolchain，用于提供 `arm-none-eabi-gdb`（TI 工具链不自带 GDB）。

```powershell
winget install Arm.GnuArmEmbeddedToolchain
```

验证：

```bash
arm-none-eabi-gdb --version
```

---

#### 4.3 安装 OpenOCD

前往 OpenOCD 官网（openocd.org）→ 下载最新 Windows 版本 → 解压到：

```
D:/OpenOCD/
```

标准 OpenOCD 发行版已包含 MSPM0 目标支持（`ti_mspm0.cfg`）。

验证：

```bash
openocd --version
```

---

### macOS

#### 4.4 Make

macOS 自带 Make。

验证：

```bash
make --version
```

---

#### 4.5 GDB 与 OpenOCD

当前模板：

* macOS 支持开发和编译验证
* 暂不支持烧录和调试

因此无需安装 OpenOCD 和 ARM GDB。

---

# 5. 创建 VS Code Profile

打开 VS Code：

```
左下角齿轮
    ↓
Profiles
    ↓
Create Profile
```

创建新的 Profile。

建议名称：

```
TI
```

选择：

```
Create Empty Profile
```

后续所有配置均放入该 Profile。

---

# 6. 安装 VS Code 扩展

进入 TI Profile，安装以下扩展：

| 扩展 | 用途 |
|------|------|
| `ms-vscode.cpptools` | C/C++ IntelliSense |
| `marus25.cortex-debug` | ARM Cortex-M 调试 |
| `ms-vscode.makefile-tools` | Makefile 项目支持 |
| `xaver.clang-format` | C/C++ 代码格式化 |
| `mcu-debug.memory-view` | 内存查看 |
| `mcu-debug.peripheral-viewer` | 外设寄存器查看 |
| `mcu-debug.debug-tracker-vscode` | 调试轨迹跟踪 |
| `mcu-debug.rtos-views` | RTOS 任务/队列视图 |
| `ti-development-tools.cortex-debug-dp-mspm0` | MSPM0 SVD + OpenOCD 配置 |
| `ti-development-tools.ti-embedded-development` | TI 项目模板 |

> 如果已安装 `ti-embedded-debug`、`file-downloader` 等无关插件，可以直接卸载。

---

# 7. 配置 VS Code Profile

将模板中的配置文件复制到 Profile：

```
settings.json
tasks.json
keybindings.json
```

`Ctrl + Shift + P` → **Open User Settings (JSON)**，粘贴 `settings.json`，替换占位符。

`Ctrl + Shift + P` → **Open User Tasks**，粘贴 `tasks.json`。

`Ctrl + Shift + P` → **Open Keyboard Shortcuts (JSON)**，粘贴 `keybindings.json`。

> 模板提供了预配置的开箱即用方案，只需替换占位符路径即可使用。但配置项较多，模板可能未覆盖所有场景——使用者应自行检查并根据需要补充配置。

---

## 7.1 修改路径

替换以下变量：

```
<path-to-ti-cgt-armllvm>
```

TI Arm Clang 路径。

---

```
<path-to-mspm0-sdk>
```

MSPM0 SDK 路径。

---

Windows：

```
<path-to-openocd>
```

OpenOCD 路径。

---

Windows：

```
<path-to-arm-none-eabi-gcc>
```

ARM GNU Toolchain 路径。

---

Windows：

```
<path-to-home>/.clang-format
```

用户目录下的 `.clang-format` 文件路径。格式化规则文件需用户自行配置，本模板不提供。

macOS 使用固定路径 `clang-format.executable`（`/opt/homebrew/bin/clang-format`），无需此占位符。

---

# 8. 配置 SDK imports.mak

进入：

```
MSPM0 SDK 根目录
```

打开：

```
imports.mak
```

修改（macOS 用 `sysconfig_cli.sh`，Windows 用 `sysconfig_cli.bat`）：

```makefile
SYSCONFIG_TOOL ?= <path-to-sysconfig>/sysconfig_cli.sh    # macOS
SYSCONFIG_TOOL ?= <path-to-sysconfig>/sysconfig_cli.bat   # Windows

TICLANG_ARMCOMPILER ?= <path-to-ti-cgt-armllvm>
```

注意：

`TICLANG_ARMCOMPILER`

不要包含：

```
/bin
```

---

# 9. 创建工程

拷贝 `win/ticlang/makefile`（Windows）或 `mac/ticlang/makefile`（macOS）到新工程的 `ticlang/` 目录。

创建以下工程文件：

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

# 10. 配置 Makefile

进入：

```
ticlang/
```

修改（macOS 用 `sysconfig_cli.sh`，Windows 用 `sysconfig_cli.bat`）：

```makefile
MSPM0_SDK_INSTALL_DIR ?= /path/to/mspm0_sdk_x_xx_xx_xx
TICLANG_ARMCOMPILER   := /path/to/ti-cgt-armllvm_x.x.x.LTS
SYSCONFIG_TOOL        := /path/to/sysconfig_x.xx.x/sysconfig_cli.sh
```

注意：

工程 Makefile 属于平台相关文件。

不要在 Windows 和 macOS 之间共享。

建议加入：

```
.gitignore
```

例如：

```
ticlang/makefile
```

完整的 `.gitignore` 参考：

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

# 11. 跨平台 Git 工作流

本模板通过 Profile 隔离 + Git 共享实现 Windows / macOS 跨平台开发：

- **工程源码**（`user/`、`.syscfg`、`main.c`、`ticlang/device_linker.cmd`）通过 Git 共享。这些文件不含本地路径，跨平台通用。
- **Profile 配置**（`settings.json`、`tasks.json`、`keybindings.json`）包含绝对路径，不提交 Git。Windows 和 macOS 分别在对应 Profile 下维护。
- **Makefile** 加入 `.gitignore`，每台机器维护本地副本。

克隆工程到新机器后，只需拷贝对应平台的 Profile 配置和 Makefile 即可直接编译。

---

# 12. 验证环境

编译前先确认工具链就绪：

```bash
mingw32-make --version        # Windows
make --version                # macOS
arm-none-eabi-gdb --version   # Windows
openocd --version             # Windows
```

全部正常输出版本号即可继续。

---

# 13. 第一次编译

打开工程。

增量编译：

```
F7
```

完整重新编译：

```
F6
```

清理：

```
F5
```

---

# 14. SysConfig 使用

生成配置源码：

```bash
make syscfg          # macOS
mingw32-make syscfg  # Windows
```

打开 GUI：

```bash
make syscfg-gui          # macOS
mingw32-make syscfg-gui  # Windows
```

注意：

SysConfig 是工具。

`.syscfg` 是配置文件。

关系：

```
.syscfg
   |
   ↓
SysConfig GUI
   |
   ↓
ti_msp_dl_config.c/h
```

不要手动修改生成文件。

---

# 15. 烧录与调试（Windows）

选择 VS Code 调试配置：

仅烧录：

```
MSPM0-download
```

调试：

```
MSPM0-debug
```

启动：

```
F8
```

停止调试：

```
Shift + F8
```

不要使用 VS Code 调试工具栏中的红色停止按钮。

该操作可能导致 OpenOCD 进程残留。

---

# 16. 换芯片

更换目标芯片时，Profile 配置中的 `defines`、`device`、`svdFile` 等字段需要同步修改。详见 `CHIP_SWITCH.md`。

---

# 17. IntelliSense 配置

本模板使用：

```
Microsoft C/C++ IntelliSense
```

不是 clangd。

原因：

clangd 是 clang 前端，不是 tiarmclang。跨编译器解析 TI/GCC 特有语法（`__attribute__`、编译器内置宏、内联汇编等）会产生大量假阳性。Microsoft C/C++ IntelliSense 通过 `compilerPath` 直接 query tiarmclang，自动获取真实的内置宏和系统头文件路径，对 TI 编译器兼容性更好。

---

配置：

```
C_Cpp.default.compilerPath
```

指向：

```
tiarmclang
```

同时：

```
intelliSenseMode
```

设置为：

```
gcc-arm
```

---

# 18. 硬件注意事项

## 18.1 SWD 调试引脚

MSPM0 默认使用：

| 引脚   | 功能    |
| ---- | ----- |
| PA19 | SWDIO |
| PA20 | SWCLK |

不要在 SysConfig 中将这两个引脚复用为其他功能。

否则可能导致无法再次连接调试器。

---

## 18.2 未使用引脚配置

建议在 SysConfig 中启用：

```
Board → Configure Unused Pins
```

避免悬空引脚产生额外功耗。

---

# 19. 常见问题

## Q: 命令行工具未找到（`mingw32-make`、`arm-none-eabi-gdb`、`openocd` 等命令无法识别）

安装工具后需将其 `bin` 目录加入系统 PATH。在终端中运行 `mingw32-make --version` 等命令验证。如果无法识别，重新安装或手动添加路径到系统环境变量。

## Q: 编译提示找不到 tiarmclang

检查：

```
TICLANG_ARMCOMPILER
```

路径。

确保：

* 路径指向编译器根目录
* 没有添加 `/bin`

正确：

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/
```

错误：

```
D:/ti/ti-cgt-armllvm_5.1.1.LTS/bin
```

---

## Q: SysConfig GUI 无法启动

确认：

* 使用独立版 SysConfig
* 不是 CCS 内置 SysConfig
* `SYSCONFIG_TOOL` 路径正确

---

## Q: 新增 `.c` 文件后没有参与编译

确认源码位于：

```
user/Src/
```

如果使用子目录，需要修改 Makefile：

```
USER_C_FILES
CFLAGS
VPATH
```

---

## Q: Windows 编译出现 `-rf` 错误

原因：

SDK 默认 Makefile 使用 Unix 风格 clean 命令。

Windows 下：

```
$(RM) -rf
```

无法正常执行。

修改为：

```makefile
$(RMDIR)
```

---

## Q: Makefile 被 Git 追踪导致 Windows/macOS 冲突

工程级 Makefile 属于平台相关文件。

建议加入：

```
ticlang/makefile
```

到：

```
.gitignore
```

如果已经被 Git 管理：

```bash
git rm --cached ticlang/makefile
```

---

## Q: 调试退出后 OpenOCD 残留

退出调试时使用：

```
Shift + F8
```

不要点击 VS Code 调试工具栏红色停止按钮。

如果已经残留：

Windows：

打开任务管理器，结束：

```
openocd.exe
```

---

## Q: Flash 失败，提示 XDS110 failed to connect

检查：

1. XDS110 USB 连接
2. OpenOCD 路径是否正确
3. 是否使用支持 MSPM0 的 OpenOCD

确认：

```
cortex-debug.openocdPath
```

指向：

```
D:/OpenOCD/bin/openocd.exe
```

---

# 20. 完成

完成以上步骤后，即可使用：

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

完成 MSPM0 的独立开发流程。

该环境提供完整透明的：

* 工程配置
* 编译流程
* SysConfig 生成流程
* 烧录流程
* 调试流程

并保持与 CCS 工作流解耦。
