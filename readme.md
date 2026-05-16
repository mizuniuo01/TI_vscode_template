# TI MSPM0 VS Code 开发模板

基于 **VS Code + 独立 SysConfig + Makefile + Cortex-Debug** 的轻量级开发流，适用于 TI MSPM0 系列芯片，彻底抛弃 CCS。

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

| 工具 | 说明 |
|------|------|
| MSPM0 SDK | TI 官网下载，提供 DriverLib 等底层库 |
| TI Arm Clang 编译器 (`ti-cgt-armllvm`) | TI 官网下载，或从 CCS 安装目录提取 |
| 独立版 SysConfig | TI 官网下载，**不要用 CCS 内置的**（缺少 GUI framework），脱离 CCS 依赖 |
| OpenOCD | Cortex-Debug 调试所需，配合 XDS110 使用 |
| VS Code 扩展 | `ms-vscode.cpptools`、`marus25.cortex-debug` 等，务必安装 Cortex-Debug 和 TI 芯片相关插件 |

> Windows 额外需要 CCS 自带的 `gmake`（位于 CCS 安装目录下的 `utils/bin/`），或将其加入 `PATH`。
> macOS 可以使用系统自带的 `make`。

---

## 配置步骤

### 第一步：创建 VS Code Profile

建议在 VS Code 中为 TI 开发单独建一个 **Profile**，将 `settings.json`、`tasks.json` 配置在 Profile 级别，所有 MSPM0 工程共享，无需在每个工程内重复配置。

> 注意：提供的json模版仅供参考。

**创建方式：**
1. VS Code 左下角点击账户图标 → **Profiles** → **Create Profile**
2. 输入名称（如 `TI`），选择从空白创建
3. 创建后，在该 Profile 下打开命令面板 (`Ctrl+Shift+P` / `Cmd+Shift+P`)
4. 搜索 **Open User Settings (JSON)**，将对应平台 `settings.json` 的内容粘贴进去，替换路径占位符
5. 搜索 **Open User Tasks**，将对应平台 `tasks.json` 的内容粘贴进去

---

### 第二步：修改 `imports.mak`

SDK 根目录下的 `imports.mak` 存放工具链路径，需要修改以下两项：

```makefile
SYSCONFIG_TOOL      ?= /path/to/sysconfig/sysconfig_cli    # Windows 用 .bat，macOS 用 .sh
TICLANG_ARMCOMPILER ?= /path/to/ti-cgt-armllvm             # 末尾不要加 /bin
```

> **注意：`TICLANG_ARMCOMPILER` 末尾不要加 `/bin`**，否则编译时会报找不到文件。
>
> **macOS 用户可跳过此步骤**，mac/makefile 已在 `include imports.mak` 之前显式赋值覆盖，`imports.mak` 保持 Windows 默认值不变，Win/Mac 可共用同一份 SDK。

---

### 第三步：创建工程 makefile

将对应平台 `ticlang/makefile` 复制到自己工程的 `ticlang/` 目录下，修改头部路径为实际路径：

**Windows** — 只需改第一行：
```makefile
MSPM0_SDK_INSTALL_DIR ?= D:/path/to/mspm0_sdk
```
`TICLANG_ARMCOMPILER` 和 `SYSCONFIG_TOOL` 由 `imports.mak` 里的默认值提供，不需要在 makefile 里额外写。

**macOS** — 改头三行：
```makefile
MSPM0_SDK_INSTALL_DIR ?= /path/to/mspm0_sdk
TICLANG_ARMCOMPILER   := /path/to/ti-cgt-armllvm
SYSCONFIG_TOOL        := /path/to/sysconfig/sysconfig_cli.sh
```
在 `include imports.mak` 之前显式赋值，覆盖掉 `imports.mak` 里的 Windows 默认值。

> 如果采用这套模版进行跨平台同步开发，将 makefile 加入 `.gitignore`，Win/Mac 各自维护本地副本互不干扰。`.gitignore` 写法：
> ```
> ticlang/makefile
> ```
> 如果 makefile 已被 git 追踪，需先执行 `git rm --cached ticlang/makefile` 解除追踪再提交。

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
| 编译 | 触发默认 build task，建议在 keybindings.json 中绑定快捷键（推荐 `F7`） |
| 编译 + 烧录 + 调试 | 触发 launch 配置，建议绑定快捷键（推荐 `F8`）；preLaunchTask 自动编译，Cortex-Debug 烧录，停在 `main()` 第一行 |
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
