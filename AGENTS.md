# AGENTS.md

> 本文件供 **AI 编码 / 写作 agent**（如 WorkBuddy、Copilot、Claude）阅读，用于了解本仓库的结构、约定与禁忌，从而在自动改动时保持风格一致。
> 人类贡献者请阅读 [`CONTRIBUTING.md`](CONTRIBUTING.md)（含完整命名规范）。

## 项目概述

- **名称**：uosc_subtitle_style（mpv 内封字幕样式快速切换）
- **GitHub 仓库名**：`awujiana/uosc_subtitle_style`
- **类型**：代码仓库（AGPL-3.0；uosc 本身为第三方 LGPL-3.0 运行时依赖，不随本仓库分发）
- **维护者**：awujian（GitHub `awujiana`）
- **一句话**：通过 uosc 菜单一键调整内封 / 外挂 ASS 字幕的位置、缩放、字体、字号、颜色与描边；主 / 次字幕分轨管理，支持上下内嵌黑边贴合与对外 API。
- **生产环境**：[awujiana/mpv-bingshan](https://github.com/awujiana/mpv-bingshan) 整合包。本仓库是该脚本的**独立公开仓库**，两边脚本、conf、架构与 API 文档、测试工具必须保持同源。

## 仓库结构

| 路径 | 用途 |
| --- | --- |
| `scripts/uosc_subtitle_style.lua` | 唯一运行时脚本；文件头 `-- @version: 1.0.12` 是本地版本依据 |
| `script-opts/uosc_subtitle_style.conf` | 50 项配置，键集合与脚本 `o` 表严格对齐 |
| `docs/uosc-subtitle-style-architecture.md` | 数据模型、mpv 能力矩阵、应用引擎、真机实证结论（§20 为 v1.0.11 位移 / 报告开关，§21 为 v1.0.12 切轨补偿 / 多行盒高 / 自动样式迁移） |
| `docs/api-uosc-subtitle-style.md` | 对外 API v1.0.0 契约（字段表 / 协议 / 白名单 30 个 / 错误码） |
| `tools/sub-style-test.lua` | 388 条单测，stub 掉 mpv API |
| `tools/sub-style-conf-verify.py` | conf 静态门禁（复算 mpv 解析语义） |
| `tools/sub-style-e2e.py` | 真机 E2E 编排，自动识别生产 / 独立两种布局 |
| `tools/sub-style-probe*.lua` | 各阶段真机探针（分轨 / 预设 / 定位 / 黑边 / 字号 / API / 位移与报告开关） |
| `tools/sub-style-probe-shift.lua` | v1.0.11 探针：`MODE=bars`（`\pos` 对白进黑边 / 点下移生效）与 `MODE=report`（`show_report` 两轮对照） |
| `tools/sub-style-probe-tracks.lua` | v1.0.12 探针：`MODE=switch`（三条内封轨切轨字号恒定）与 `MODE=styles`（多 Style 下只取 `Default` 基准） |
| `tools/sub-style-mkv-fixture.py` | 纯 Python 最小 Matroska 夹具，不依赖 ffmpeg；`styles` / `extra_subs` 可组合出多 Style、多内封轨、`\N` 多行对白 |
| `tools/lua-syntax-check.lua` | Lua 语法门禁 |
| `version.json` | 机器可读版本清单，必须与 `@version` / Release tag 一致 |
| `README.md` / `CONTRIBUTING.md` / `AGENTS.md` / `CHANGELOG.md` / `LICENSE` | 对外说明、贡献、AI 约定、更新日志、AGPL-3.0 全文 |

## 必须遵守的约定

1. **命名规范（不缩写）**：文件 / 函数全拼无缩写、名称不重复、Lua 脚本 `snake_case.lua`、`script-message` 入口 `kebab-case`。
2. **版本三处一致**：改脚本功能必须同步文件头 `-- @version: X.Y.Z`、`version.json` 的 `version` 与 `release_url`、GitHub Release tag `vX.Y.Z`；`script-modules/updater.lua`（在 `awujiana/mpv-script-updater`）以 `@version` 判定本地版本，同版本不更新、本地更新时禁止降级。
3. **内封 ASS 通道**：带 `\pos` 的 explicit ASS 事件对 `sub-pos` / `sub-scale` / `sub-font-size` / `sub-margin-y` **全部豁免**；样式只能写 `sub-ass-style-overrides` 的 `FontName` / `PrimaryColour` / `OutlineColour` / `Fontsize`（颜色 `&H00BBGGRR`），缩放与字号折成 `Fontsize` 并把两轨 `sub-scale` 归 1 防双叠加。**禁止** `ScaleX/ScaleY`（125% 起字幕消失），**禁止** `strip`（会丢 `\pos` 且让 style-overrides 整体失效），`factor == 1` 时不得写 `Fontsize`（会压平多 Style 原始字号层级）。
   **垂直位移（v1.0.11）**：唯一能改 `\pos` 的 Y 的是覆盖 `[Script Info]` 的 `PlayResY`，`y_render = y_ass × 帧高 / PlayResY`；写 `PlayResY = 原始/ratio` 时字体按 `ratio` 放大，必须同步把 `Fontsize` 除以 `ratio` 补回物理字号（补偿与字号通道是否开启无关）。`ass_y_ratio()` 即时求值不缓存、clamp `[0.5, 2.0]`；黑边贴合优先用反解的 `y_ratio`，否则按 `sub-pos` 偏移换算；`ratio == 1` 时两个覆盖都不写。⚠ 解码器级共享、以画面顶部为原点，次轨 `\pos` 无法同时进上黑边。
   **左上角报告（v1.0.11）**：`show_report=no` 只关显示不关功能；`flash_tl()` 是左上角 `show-text` 唯一出口，关闭时不 `show-text`、不临时改 `osd-align-*`；`refresh_barfit()` 的补报告路径也要判 `runtime.show_report`，否则会偷偷放回报告。
   **切轨字号 / 多行 / 自动样式（v1.0.12）**：`Fontsize=` / `PlayResY=` 是按当前轨 ASS 头算出的**绝对值**，切内封轨后必须按 `base_fontsize/playres_y` 反解补偿倍率重算（`ass_ref_units` per-file；`sub-ass-extradata` 观察器 + `sid` 观察器 0.1s 延迟补读双保险）；多行对白的字号基准只取 `Default`/`base_fontsize`，**禁止**用 `max_fontsize`（会被 signs / 大标题样式拉偏），行数另由 `sub-text` / `sub-text-ass` / `secondary-sub-text` 三路观察器统计进整块盒高；`switch_target()` 在来源轨 `auto=true` 且目标轨没有用户手动样式时必须把自动样式与 `auto` 标记一起带过去。后两路备用文本观察器**只用于行数统计**，不得参与 `inspect_*` 检测时序。
4. **技术禁令**：绝不轻易重建字幕解码器（会打断内封字幕包流，表现为「字幕定住一句」）；稳态编辑只写属性 + `show-text ' ' 60` 触发 `configure_ass`；`sub-margin-y` 是 INT 且不在 `reset-on-next-file`，贴合结束必须显式还原；`user-data` 是 node 属性，读取必须 `native`；`script-message-to` 参数含 `#RRGGBB` 必须加引号；`.conf` 禁止行尾注释；禁止 `load-script` 热重载。
5. **对外 API 边界**：API 版本 `1.0.0`，只做只读状态快照 + 白名单动作转发；`api` 自身不得进入 `call` 白名单；旧 `script-message` 入口参数与副作用必须原样保留；响应 JSON 用 `mp.command_native` 数组形式发送。
6. **内封轨全覆盖**：所有 `external=false` 的 `sub` 轨都要能被检查；切到未检查轨时必须先清自动预设再重检，否则 `wants_force()` 会让像素采样直接放弃。
7. **测试不得伪造 PASS**：没有 `uosc` 时字体选择器三项明确 SKIP；没有 `uosc_staff_credits` 时 API 探针 7 条职员表断言从期望表移除并打印跳过原因。新增行为必须同时新增单测与真机断言。
8. **README 风格**：顶部 `<div align="center">` + shields.io 徽章 → 标题 → `> [!WARNING]` AI 声明 → 稳定锚点目录 → 正文 → 底部三节（关联仓库 / 作者联系方式 / 开源协议）。标题不用 VS16 变体 emoji（`⚠️` / `🖼️`），会破坏自动锚点。
9. **协议与提交**：代码 AGPL-3.0，`LICENSE` 全文不许改；`git config user.name=awujian`、`user.email=bingshanlengtie@qq.com`；提交信息说明「做了什么 / 为什么」；每次改动同步 `CHANGELOG.md`。
10. **同源同步**：改完本仓库后，把脚本 / conf / 两份文档 / `tools/sub-style-*` 同步回 `mpv-bingshan` 的 `portable_config`，反之亦然；两端共享文件应保持字节一致（架构文档里的姊妹链接必须是公开绝对 URL，不能用只在一端有效的相对路径）。

## 改动前检查

- **通用门禁**（判据 = 退出码 + 汇总行，不要只看 tail）：
  - `luajit tools/lua-syntax-check.lua` → `FAIL 0`
  - `luajit tools/sub-style-test.lua` → `SUMMARY sub-style-test pass=388 fail=0 ALL_OK`
  - `python tools/sub-style-conf-verify.py` → `SUMMARY sub-style-conf-verify PASS fail=0`
  - `MPV_EXE=/path/to/mpv.exe python tools/sub-style-e2e.py` → `SUMMARY sub-style-e2e PASS fail=0`
- **改 README / 文档**：核对本地相对链接全部存在、标题无 VS16、目录锚点与 `sec-N` 一一对应。
- **发版**：`version.json` / `@version` / tag 三处一致 → 门禁全绿 → 提交推送 → `gh release create vX.Y.Z` 上传**两个直接文件**（`uosc_subtitle_style.lua`、`uosc_subtitle_style.conf`），不打包 zip。
- **提交**：`git status` 确认不含运行时 `uosc-subtitle-style-*.json`、`*.log`、临时复制的 `scripts/uosc/`。

## 不要做的事

- 不要手写 `⚠️ Warning` 代替 GitHub 警示块 `> [!WARNING]`。
- 不要把 `uosc-updater-state.json` 的 `tag` 当本地版本，不要把已发布 Release tag 的资产改掉（撤回应发更高版本）。
- 不要用 `sub-reload` 重建解码器来「刷新」样式，不要在 conf 值后面写注释。
- 不要把 uosc（第三方 LGPL）源码提交进本仓库；E2E 临时复制的 `scripts/uosc/` 已被 `.gitignore` 排除。
- 不要为了让门禁变绿而删断言、放宽阈值或把 SKIP 计成 PASS。
- 不要删除 `AGENTS.md` / `CONTRIBUTING.md` / `CHANGELOG.md`；它们承载协作与 AI 约定。
