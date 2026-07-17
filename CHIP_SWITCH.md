# TI Chip Switch Reference / TI 换芯片速查表

When switching the target MCU, update the following fields in your Profile settings. / 更换目标芯片时，修改 Profile 配置中的以下字段。

> **OpenOCD cfg files do NOT change** when switching within the MSPM0 family — all MSPM0 devices share the same target cfg (`ti_mspm0.cfg`) and debug probe interface (`xds110.cfg`). Only `device` and `svdFile` need updates. / 在同一 MSPM0 系列内换芯片时，**OpenOCD cfg 文件无需修改**——所有 MSPM0 型号共用同一个 target cfg（`ti_mspm0.cfg`）和调试器接口（`xds110.cfg`）。只需更新 `device` 和 `svdFile` 字段。

---

## Profile `settings.json` / Profile 配置文件

### G3507 → G150X（e.g. MSPM0G3507 → MSPM0G1506）

| Field / 字段 | Before / 改前 | After / 改后 |
|-------------|--------------|--------------|
| `C_Cpp.default.defines[0]` | `__MSPM0G3507__` | `__MSPM0G1506__` |
| `launch.configurations[0].device` | `MSPM0G3507` | `MSPM0G1506` |
| `launch.configurations[0].svdFile` | `.../MSPM0G350X.svd` | `.../MSPM0G150X.svd` |

### G3507 → G3107

| Field / 字段 | Before / 改前 | After / 改后 |
|-------------|--------------|--------------|
| `C_Cpp.default.defines[0]` | `__MSPM0G3507__` | `__MSPM0G3107__` |
| `launch.configurations[0].device` | `MSPM0G3507` | `MSPM0G3107` |
| `launch.configurations[0].svdFile` | `.../MSPM0G350X.svd` | `.../MSPM0G310X.svd` |

---

## `ticlang/makefile` / 工程 Makefile

| Field / 字段 | Notes / 说明 |
|-------------|-------------|
| SDK path / SDK 路径 | Unchanged / 不变 — same SDK covers all series / 同 SDK 支持全系列 |
| `-mcpu` / `-march` / `-mfloat-abi` | Unchanged / 不变 — all series are Cortex-M0+ / 全系列 Cortex-M0+ |
| Last line of `LFLAGS` / `LFLAGS` 最后一行 | No trailing `\` / 末尾不含 `\` — fixed in template / 模板已修正 |

> After switching chips, regenerate SysConfig: `cd ticlang && make syscfg` (macOS) / `mingw32-make syscfg` (Windows). Then F6 for a full rebuild. / 换芯片后重新生成 SysConfig：`make syscfg`（macOS）/ `mingw32-make syscfg`（Windows），然后 F6 全量重编。
