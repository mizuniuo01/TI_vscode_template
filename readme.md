# TI MSPM0 VS Code 开发模板

基于 **VS Code + 独立 SysConfig + Makefile + Cortex-Debug** 的轻量级开发流，适用于 TI MSPM0 系列芯片，彻底抛弃 CCS。

支持 Windows 和 macOS 双平台。

---

## 目录结构

```
TI_vscode_template/
├── win/                    # Windows 平台模版工程（路径为占位符）
│   ├── template.c
│   ├── template.syscfg
│   ├── settings.json       ← 粘贴到 VS Code TI profile 全局设置
│   ├── tasks.json          ← 粘贴到 VS Code TI profile 全局任务
│   └── ticlang/
│       ├── makefile
│       └── device_linker.cmd
├── mac/                    # macOS 平台模版工程（路径为占位符）
│   ├── template.c
│   ├── template.syscfg
│   ├── settings.json       ← 粘贴到 VS Code TI profile 全局设置
│   ├── tasks.json          ← 粘贴到 VS Code TI profile 全局任务
│   └── ticlang/
│       ├── makefile
│       └── device_linker.cmd
└── readme.md
```

`win/` 和 `mac/` 是完整的可用模版工程，路径占位符为 `/path/to/...`（Windows 用反斜杠 `D:/path/to/...`）。`settings.json` 和 `tasks.json` 需粘贴到 VS Code 的 **TI profile** 全局配置中，不是工程内的 `.vscode/`。通过此举可以不用每次新建工程都重新配置json。

---

## 前置依赖

| 工具 | 说明 |
|------|------|
| MSPM0 SDK | TI 官网下载，提供 DriverLib 等底层库 |
| TI Arm Clang 编译器 (`ti-cgt-armllvm`) | TI 官网下载，或从 CCS 安装目录提取 |
| 独立版 SysConfig | TI 官网下载，**不要用 CCS 内置的**（缺少 GUI framework）脱离 CCS 依赖 |
| OpenOCD | Cortex-Debug 调试所需，配合 XDS110 使用 |
| VS Code 扩展 | `ms-vscode.cpptools`、`marus25.cortex-debug`等。（务必安装Cordex-Debug和TI芯片相关插件等插件。） |

> Windows 额外需要 CCS 自带的 `gmake`（位于 CCS 安装目录下的 `utils/bin/`），或将其加入 `PATH`。
> macOS 可以使用 macOS 自带的 `make`。

---

## 配置步骤

### 第一步：修改 `imports.mak`

SDK 根目录下的 `imports.mak` 存放工具链路径，需要修改以下两项：

```makefile
SYSCONFIG_TOOL      ?= /path/to/sysconfig/sysconfig_cli    # Windows 用 .bat，macOS 用 .sh
TICLANG_ARMCOMPILER ?= /path/to/ti-cgt-armllvm             # 末尾不要加 /bin
```

> **注意：`TICLANG_ARMCOMPILER` 末尾不要加 `/bin`**，否则编译时会报找不到文件。

---

### 第二步：创建工程 makefile

将 `local/ticlang/makefile` 复制到自己工程的 `ticlang/` 目录下，修改第一行 `MSPM0_SDK_INSTALL_DIR` 为 SDK 实际路径：

**Windows**
```makefile
MSPM0_SDK_INSTALL_DIR ?= D:/path/to/mspm0_sdk
```

**macOS**
```makefile
MSPM0_SDK_INSTALL_DIR ?= /path/to/mspm0_sdk
TICLANG_ARMCOMPILER   := /path/to/ti-cgt-armllvm
SYSCONFIG_TOOL        := /path/to/sysconfig/sysconfig_cli.sh
```

> macOS 上在 `include imports.mak` 之前显式赋值 `TICLANG_ARMCOMPILER` 和 `SYSCONFIG_TOOL`，覆盖掉 `imports.mak` 里的 Windows 默认值。如果采用这套模版进行跨平台同步开发，注意 makefile 加入 `.gitignore`，Win/Mac 各自维护本地副本互不干扰。

---

### 第三步：配置 VS Code profile

建议在 VS Code 中为 TI 开发单独建一个 **Profile**，将 `settings.json`、`tasks.json` 配置在 Profile 级别，所有 MSPM0 工程共享，无需在每个工程内重复配置。

将对应平台的示例文件（`win/` 或 `mac/` 下）内容复制到 Profile 的全局 `settings.json` 和 `tasks.json`，并将路径占位符替换为实际路径：

| 占位符 | 替换为 |
|--------|--------|
| `/path/to/mspm0_sdk` | MSPM0 SDK 根目录 |
| `/path/to/ti-cgt-armllvm` | 编译器根目录（不含 `/bin`） |

**Windows** 使用 `gmake`（CCS 自带），**macOS** 使用系统 `make`。

> 本模版里提供的 `json` 配置仅供参考，具体请依照自己的需求进行配置。

---

### 第四步：工程目录结构约定

```
MyProject/
├── main.c
├── project.syscfg
├── user/
│   ├── Inc/
│   └── Src/
└── ticlang/
    ├── makefile              ← 本地维护，加入 .gitignore
    ├── device_linker.cmd
    └── ti_msp_dl_config.*    ← SysConfig 自动生成，勿手动修改
```

---

## 常用操作

| 操作 | 方式 |
|------|------|
| 编译 | 建议自行指定快捷键，推荐 `F7` |
| 编译 + 烧录 + 调试 | `F8`（preLaunchTask 自动编译，Cortex-Debug 烧录，停在 `main()` 第一行） |
| 仅清理 | 运行 `Clean` task |
| 详细编译输出 | `cd ticlang && make VERBOSE=1` |
| 重新生成 SysConfig 文件 | `cd ticlang && make syscfg` |
| 打开 SysConfig GUI | `cd ticlang && make syscfg-gui` |

---

## 硬件注意事项

- **PA19 (SWDIO) 和 PA20 (SWCLK)** 是 SWD 调试引脚，在 SysConfig 中**绝对不能**复用为其他功能，否则锁芯片
- 建议在 SysConfig 中勾选 **Board → Configure Unused Pins**，避免悬空引脚漏电
- 默认栈大小为 512 字节，代码量较大时可以通过手动创建.cmd文件，覆盖默认栈大小，以扩大

---

## 常见问题

**Q: 编译报找不到 `tiarmclang`**
检查 `imports.mak` 中 `TICLANG_ARMCOMPILER` 末尾是否多写了 `/bin`。

**Q: makefile 被 git 追踪导致 Win/Mac 互相污染**
在工程 `.gitignore` 中加入 `ticlang/makefile`，并执行 `git rm --cached ticlang/makefile` 解除追踪。
