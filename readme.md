# 使用VS Code和Makefile简化的MSPM0工程开发模板

# 简要介绍：
# 本工程彻底抛弃了臃肿的CCS 采用**VS Code+独立SysConfig+Makefile+Cortex-Debug**的轻量级、自动化开发流
# 本readme使用copilot协助撰写
# 需要提前配置好全局的tasks.json settings.json
# template是英文的'模板'的意思
# 关键是使用json和makefile自行进行路径链接和配置 json的配置很容易链接错误但是报错不一定明说链接错误 并且makefile改写成自动化脚本才会方便以后使用

---

## 硬件与芯片避坑指南

### 1. 调试引脚
MSPM0G3507 依赖以下两个引脚进行SWD下载和在线调试（对应板载的 XDS110）：
* **PA20** : SWCLK (时钟)
* **PA19** : SWDIO (数据)

**致命警告**：在SysConfig中配置外设时 **绝对不要**将PA19和PA20复用为其他功能 一旦引脚被强行占用 单片机将无法再次连接仿真器 导致锁芯片变砖

### 2. 官方低功耗建议
为了达到最低功耗（防止悬空引脚漏电）请在SysConfig图形界面中：
导航到 **Board → Configure Unused Pins** 并勾选
SysConfig会自动生成代码，将所有未使用的引脚配置为内部下拉或输出低电平

---

## VS Code开发流说明

### 1. 核心架构原理
* **编译器**：`tiarmclang` (TI 官方基于 LLVM 的编译器)
* **底层库**：预编译的 `DriverLib.a` 静态库（通过绝对路径链接，工程体积仅几十 KB）
* **硬件配置**：`.syscfg` 文件 由独立的SysConfig GUI编辑 并在编译时由Makefile自动生成 `ti_msp_dl_config.c` 和 `.h`
* **构建系统**：魔改版万能 `Makefile` 支持自动搜刮目录下的源码 并自动根据syscfg的名字给固件命名

### 2. 工程目录结构
```
📁 My_MSPM0_Project
 ├── 📄 template.c       <-- 主函数 (main.c)
 ├── 📄 template.syscfg  <-- 硬件配置文件 (建议使用SysConfig GUI生成syscfg后保存到工程内使用)
 ├── 📁 user             <-- 自己的业务代码 (Makefile会自动识别并编译该文件夹下的所有 .c/.h)
 │    ├── 📄 example.c
 │    └── 📄 example.h
 └── 📁 ticlang          <-- 编译工作区 (执行make的地方，VS Code需以此为工作目录)
      └── 📄 makefile    <-- 核心构建脚本 (已配置VPATH和自动推导)
```
*(注：目前的 Makefile 默认抓取 `user/` 目录 如果想增加其他名字的文件夹如 `hardware/` 只需在 Makefile 的 `CFLAGS`、`USER_C_FILES` 和 `VPATH` 三处加上新路径即可)*

### 3. 日常开发流程

**第一步：配置硬件外设 (SysConfig)**
1. 打开在TI官网下载的独立SysConfig
2. 选择固件库路径和芯片后开始配置
3. 配置完毕后将 `.syscfg` 文件保存到当前工程的文件夹下(注意文件名必须和工程名对应 否则makefile会链接错误)

**第二步：编写业务逻辑**
1. 在 `user/` 文件夹下创建 `.c` 和 `.h` 文件
2. 魔改的 Makefile 包含了 `wildcard` 和 `VPATH` 规则 会自动发现并编译 `user/` 目录下的所有源码

**第三步：编译和烧录**
1. **仅编译检查(F7)**：在 VS Code 快捷键设置 (`keybindings.json`) 中将 `workbench.action.tasks.build` 绑定为 `F7` 敲代码时按 `F7` 即可编译出 `.out` 固件
2. **一键烧录与调试 (F5)**：连上开发板（XDS110）选择对应的launch配置 按下 `F5` 下载（本人改成了 `F8` ）
3. **丝滑连招**：得益于 `settings.json` 中的 `launch`模块中的 `preLaunchTask` 系统将全自动完成：保存代码 -> 调起 `gmake` 编译 -> 调起 OpenOCD 烧录 -> 停在 `main()` 函数第一行断点

---

## 附录
如果在其他电脑上还原此开发流，需确保以下几点：
1. 安装 CCS（提取里面的编译器和 gmake）和 MSPM0 SDK（提供底层库）
2. 安装独立版SysConfig
3. 修改核心路径：前往 SDK 目录下的 `imports.mak`，修改 `TICLANG_ARMCOMPILER`、`SYSCONFIG_TOOL` 等路径为当前电脑的绝对路径。*(注：��译器路径末尾千万不要多写一个 `/bin`，否则会报错找不到文件！)*
4. 在本工程的 `makefile` 第一行 确认 `MSPM0_SDK_INSTALL_DIR` 指向了正确的 SDK 绝对路径
5. 在当前VS Code配置中配置好全局的 `tasks.json` 和 `settings.json`便于日后跨工程快捷使用