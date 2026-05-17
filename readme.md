# TI MSPM0 VS Code 开发模板

基于 **VS Code + 独立 SysConfig + Makefile + Cortex-Debug** 的轻量级开发流，适用于 TI MSPM0 系列芯片。

> macOS 可彻底脱离 CCS。Windows 若自行安装 GNU Make 并加入 PATH，也可完全不用 CCS；如不想折腾，安装 CCS 可直接获得 `gmake`。

支持 Windows 和 macOS 双平台。

---

## 目录结构

```
TI_vscode_template/
├── win/                    # Windows 平台模版工程（路径为占位符）
│   ├── main.c
│   ├── template.syscfg
│   ├── settings.json       ← 粘贴到 VS Code TI profile 全局设置
│   ├── tasks.json          ← 粘贴到 VS Code TI profile 全局任务
│   └── ticlang/
│       ├── makefile
│       └── device_linker.cmd
├── mac/                    # macOS 平台模版工程（路径为占位符）
│   ├── main.c
│   ├── template.syscfg
│   ├── settings.json       ← 粘贴到 VS Code TI profile 全局设置
│   ├── tasks.json          ← 粘贴到 VS Code TI profile 全局任务
│   └── ticlang/
│       ├── makefile
│       └── device_linker.cmd
└── readme.md
```

`win/` 和 `mac/` 是完整的可用模版工程，路径占位符为 `/path/to/...`（Windows 用 `D:/path/to/...`）。`settings.json` 和 `tasks.json` 需粘贴到 VS Code 的 **TI profile** 全局配置中，不是工程内的 `.vscode/`。通过此举可以不用每次新建工程都重新配置 json。

---

## 前置依赖

### TI 官方工具（Win / Mac 均需安装）

| 工具 | 说明 | 获取方式 |
|------|------|----------|
| MSPM0 SDK | 提供 DriverLib 等底层库 | TI 官网下载 |
| TI Arm Clang 编译器 (`ti-cgt-armllvm`) | 编译工具链 | TI 官网下载 |
| 独立版 SysConfig | 引脚/外设配置工具 | TI 官网下载，**不要用 CCS 内置的**（缺少 GUI framework） |

> 以上三个是 TI 开发的核心依赖，`settings.json` 和 `makefile` 中的路径占位符都指向它们。

### 平台特有依赖

| 工具 | Win | Mac | 说明 |
|------|-----|-----|------|
| Make | 方案一：自行安装 GNU Make（Chocolatey: `choco install make`，或 MSYS2: `pacman -S make`），加入 PATH；方案二：安装 CCS，使用其自带的 `gmake`（位于 `utils/bin/`） | 系统自带，无需额外安装 | 构建后端 |
| OpenOCD | 需要安装 | 暂不涉及 | Cortex-Debug 调试烧录 |

### VS Code 扩展

| 扩展 | 用途 |
|------|------|
| `ms-vscode.cpptools` | C/C++ 语法分析 |
| `marus25.cortex-debug` | 调试烧录（Win） |
| TI 芯片相关扩展 | 按需 |

---

## 配置步骤

### 第一步：安装 TI 官方工具

1. 从 TI 官网下载 **MSPM0 SDK**，解压到本地，记下路径
2. 下载 **TI Arm Clang 编译器** (`ti-cgt-armllvm`)，解压到本地，记下路径
3. 下载 **独立版 SysConfig**，记下 `sysconfig_cli.sh`（macOS）或 `sysconfig_cli.bat`（Windows）的路径

### 第二步：安装平台依赖

**macOS：** 系统自带 `make`，无需额外操作。

**Windows：** 安装 GNU Make（`choco install make` 或 MSYS2 等，确保 `make` 在 PATH 中）；如已安装 CCS 则可直接用其自带的 `gmake`。另需安装 OpenOCD 用于调试烧录。

### 第三步：修改 `imports.mak`

SDK 根目录下的 `imports.mak` 存放工具链路径：

```makefile
SYSCONFIG_TOOL      ?= /path/to/sysconfig/sysconfig_cli    # Win 用 .bat，macOS 用 .sh
TICLANG_ARMCOMPILER ?= /path/to/ti-cgt-armllvm             # 末尾不要加 /bin
```

> **注意：** `TICLANG_ARMCOMPILER` 末尾不要加 `/bin`，否则编译时会报找不到文件。
>
> **macOS 用户可跳过此步骤：** `mac/ticlang/makefile` 已在 `include imports.mak` 之前显式赋值覆盖，`imports.mak` 保持默认值不变即可，Win/Mac 共用同一份 SDK。

### 第四步：创建 VS Code Profile

在 VS Code 中为 TI 开发单独建一个 **Profile**，`settings.json`、`tasks.json` 配置在 Profile 级别，所有 MSPM0 工程共享。

1. VS Code 左下角齿轮 → **Profiles** → **Create Profile**
2. 输入名称（如 `TI`），选择从空白创建

### 第五步：安装扩展

在该 Profile 下安装：**C/C++**（`ms-vscode.cpptools`）、**Cortex-Debug**（`marus25.cortex-debug`，Win 需要）等。

### 第六步：编写 Profile JSON

1. `Ctrl+Shift+P` / `Cmd+Shift+P` → 搜索 **Open User Settings (JSON)**，将对应平台 `settings.json` 的内容粘贴进去
2. 将所有 `/path/to/...` 占位符替换为第一步记下的实际路径：
   - `mspm0_sdk` → MSPM0 SDK 路径
   - `ti-cgt-armllvm` → TI Arm Clang 编译器路径
   - `svdFile` → SDK 内对应芯片的 SVD 文件路径
3. `Ctrl+Shift+P` / `Cmd+Shift+P` → 搜索 **Open User Tasks**，将对应平台 `tasks.json` 的内容粘贴进去

> 路径在 `settings.json` 中出现了三处：`C_Cpp.default.includePath`（SDK 头文件）、`C_Cpp.default.compilerPath`（编译器）、`launch` 中的 `svdFile`（调试用 SVD 文件）。

### 第七步：创建工程 makefile

将对应平台的 `ticlang/makefile` 复制到自己工程的 `ticlang/` 目录下，修改头部路径：

**macOS** — 改头三行：
```makefile
MSPM0_SDK_INSTALL_DIR ?= /path/to/mspm0_sdk
TICLANG_ARMCOMPILER   := /path/to/ti-cgt-armllvm
SYSCONFIG_TOOL        := /path/to/sysconfig/sysconfig_cli.sh
```

**Windows** — 只需改第一行（其余由 `imports.mak` 提供）：
```makefile
MSPM0_SDK_INSTALL_DIR ?= D:/path/to/mspm0_sdk
```

> 如果采用这套模版进行跨平台同步开发，将 `makefile` 加入 `.gitignore`，Win/Mac 各自维护本地副本互不干扰。`.gitignore` 写法：
> ```
> ticlang/makefile
> ```
> 如果 makefile 已被 git 追踪，需先执行 `git rm --cached ticlang/makefile` 解除追踪再提交。

---

## 工程目录结构约定

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
    ├── stack_override.cmd    ← 可选，见下方说明
    └── ti_msp_dl_config.*    ← SysConfig 自动生成，勿手动修改
```

**关于自动编译：** makefile 使用 `wildcard` + `VPATH` 自动抓取源文件，`user/Src/` 下新建 `.c` 文件无需改动 makefile，直接编译即可。**但只有 `user/Src/` 这一层是自动的**，若在其下再建子目录（如 `user/Src/motor/`），需在 makefile 的以下三处手动添加新路径：

```makefile
USER_C_FILES := ... $(wildcard ../user/Src/motor/*.c)   # 新增源文件搜索
CFLAGS += ... -I../user/Src/motor                       # 新增头文件搜索
VPATH = ... ../user/Src/motor                           # 新增 make 搜索路径
```

**关于 `stack_override.cmd`：** SDK 默认栈大小为 512 字节，代码量较大时建议扩大。做法是在 `ticlang/` 下创建 `stack_override.cmd`，内容只需一行：

```
--stack_size=2048
```

makefile 的 `LFLAGS` 末尾已引用此文件，链接器取最后一个 `--stack_size` 值，不会被 `make syscfg` 覆盖。

---

## 常用操作

| 操作 | 方式 |
|------|------|
| 编译 | 触发默认 build task，建议在 keybindings.json 中绑定快捷键 |
| 编译 + 烧录 + 调试 | 触发 launch 配置；preLaunchTask 自动编译，Cortex-Debug 烧录，停在 `main()` 第一行 |
| 仅清理 | 运行 `Clean` task |
| 详细编译输出 | `cd ticlang && make VERBOSE=1` |
| 重新生成 SysConfig 文件 | `cd ticlang && make syscfg` |
| 打开 SysConfig GUI | `cd ticlang && make syscfg-gui` |

---

## 硬件注意事项

- **PA19 (SWDIO) 和 PA20 (SWCLK)** 是 SWD 调试引脚，在 SysConfig 中**绝对不能**复用为其他功能，否则锁芯片
- 建议在 SysConfig 中勾选 **Board → Configure Unused Pins**，避免悬空引脚漏电
- 默认栈大小为 512 字节，代码量较大时建议通过 `stack_override.cmd` 扩大（推荐 2048）

---

## 常见问题

**Q: 编译报找不到 `tiarmclang`**

检查 `imports.mak` 中 `TICLANG_ARMCOMPILER` 末尾是否多写了 `/bin`。

**Q: `make syscfg-gui` 报错**

需要使用独立版 SysConfig，不能用 CCS 内置版本。

**Q: makefile 被 git 追踪导致 Win/Mac 互相污染**

在工程 `.gitignore` 中加入 `ticlang/makefile`，并执行 `git rm --cached ticlang/makefile` 解除追踪后提交。

**Q: 新建 `.c` 文件后编译找不到**

确认文件放在 `user/Src/` 下（一层，不是子目录）。若放在子目录，需在 makefile 的 `USER_C_FILES`、`CFLAGS`、`VPATH` 三处手动添加路径。
