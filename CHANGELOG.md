# 更新日志

本项目所有值得注意的变更都记录在这里。格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。`version.json`、脚本文件头 `-- @version:` 与 Release tag 三者必须一致。

## [未发布]

暂无。

## [1.0.12] - 2026-11-08

### 修复

- **切换内封字幕轨后字号不匹配**：写进 `sub-ass-style-overrides` 的 `Fontsize=` / `PlayResY=` 是按**当前轨** ASS 头算出的绝对值。有多种内封字幕的视频切轨后 `[Script Info] PlayResY` 与 `Default.Fontsize` 可能完全不同，旧绝对值会把上一条轨的物理字号带过去，表现为「字体样式变了、也落进黑边了，但大小不对」。现在按每条轨的原生字号占比 `base_fontsize / playres_y` 反解补偿倍率，使归一化物理字号 `Fontsize / PlayResY` 在切轨前后保持一致；`sub-ass-extradata` 变化与 `sid` / `secondary-sid` 切换都会补一次重算（同一个头只应用一次，用 `nudge_reconfigure()` 触发 `configure_ass`，不重建解码器）。账本在 `file-loaded` 清空，不跨文件泄漏。
- **多行对白最下面一行被裁切、字号被大标题样式带偏**：黑边贴合的盒高估算原先取 `[V4+ Styles]` 全表 `max_fontsize`，常被顶部标题 / signs 样式拉高。现改为只按最下面一行对白的 `Default` / `base_fontsize` 作字号基准；行数另由 `sub-text` / `sub-text-ass` / `secondary-sub-text` 三路观察器统计（认 `\N` / `\n` / 换行，上限 6 行，0.3s 去抖）累计进整块盒高，让 `compute_barfit()` 按整块限缩，并把盒底目标钳在画面内（`clamp(target, box_frac, 1 - box_frac × 0.1)`）。字号基准与行数解耦：只按最下行匹配字号，同时保证所有行都在视频内。
- **点「编辑目标」后自动适配的字体恢复默认**：`FontName` / `sub-font-size` 是解码器级共享值，切到另一条空目标轨时 `cur()` 无样式，`FontName=` 会被移出 overrides 列表，字体回到 ASS 原生。现在当来源轨是自动适配（`auto=true`）且新目标轨没有用户手动样式时，把自动样式与 `auto` 标记一起带过去；新目标已有手动样式时仍以用户选择为准。

### 变更

- 单元测试新增 **T58**（切轨物理字号一致 / 三轨 / 切回还原 / per-file 账本重置 / 最下行基准不取 `max_fontsize` / 多行整块盒限缩 / `sub-text-ass` 与 `secondary-sub-text` 备用文本通道 / 编辑目标保留自动样式 / 手动目标对照），373 → **388 条全绿**。
- **夹具拓展**：`tools/sub-style-mkv-fixture.py` 新增 `styles`（额外 ASS `Style:` 行）与 `extra_subs`（任意多条内封轨，各自指定 `fontsize` / `playres_y` / `events` / `styles`），`events` 可带 `\N` 多行；不传新参数时行为与旧夹具完全一致。
- 真机门禁新增 `tools/sub-style-probe-tracks.lua` 与 E2E **5i**（三条内封轨逐轨切换，绝对覆盖逐轨重算、归一化物理字号恒为 `0.0833`、切回还原）、**5j**（`Default=20` / `Sign=60` 多样式表下 `Fontsize≈17.02`，误用 `max_fontsize` 会是 `≈5.67`）。5i 不带黑边贴合通道（无 `PlayResY` 覆盖），故其物理字号为 `0.0833`；§21.1 表中 `0.05` 来自带黑边 + `\pos` 的早期诊断夹具，两者都是「切轨前后恒定」这一不变量。全套 E2E 由 226 → **250 条 PASS**（生产整合包 233 → 257），`SUMMARY sub-style-e2e PASS fail=0`。
- ⚠ 真机边界：MJPEG 单帧 + 内封 ASS 夹具下 mpv **不暴露** `sub-text` / `sub-text-ass`（不带主脚本实测也一样），因此行数统计另挂 `sub-text-ass` / `secondary-sub-text` 两路备用观察器（只参与行数，不改 `inspect_*` 检测时序）；多行盒高的正确性由 T58 覆盖，真机探针只打印诊断值。
- 架构文档新增 **§21**（切轨补偿公式与真机三轨实测数据、生命周期、多行盒高模型、自动样式随编辑目标迁移、夹具拓展与门禁）。
- 本次**不新增** conf 配置键、不新增 `script-message`、不新增 API 字段；对外 API 版本仍为 `1.0.0`（无破坏性变更）。`sub-ass-extradata` / `sub-text-ass` / `secondary-sub-text` 只是多了只读观察器。

## [1.0.11] - 2026-11-06

### 新增

- **内封定位字幕可下移**：真机逐通道实测确认带 `\pos` 的 explicit ASS 事件对 `sub-pos` / `sub-scale` / `sub-font-size` / `sub-margin-y` 全部豁免，唯一能改 Y 的是覆盖 ASS `[Script Info]` 的 `PlayResY`。据此新增位移通道：把「上移/下移」与「上下黑边贴合」折成 `ratio` 写进 `PlayResY`，并用 `Fontsize` 除以 `ratio` 补回物理字号。对话轨道本身是 `\pos` 对白时也能整条推进下黑边、点下移即时生效。菜单「其它 → 内封定位字幕可下移」与消息 `ass-pos-shift-toggle` 可关回「保持原位」。
- **左上角报告可关闭**：conf `show_report=yes`（设 `no` 永久关闭）、菜单「显示左上角检测报告」与消息 `report-toggle`。`flash_tl()` 成为左上角 `show-text` 唯一出口，关闭时不输出、也不临时改 `osd-align-*`；检测 / 黑白预设 / 全内封轨检查 / 黑边贴合 / 位置字号通道全部照常执行。`runtime.report_shown` 计数供 API 与门禁验证。
- 对外 API 快照新增 `ass_pos_shift_active` / `ass_y_ratio` / `barfit.y_ratio` / `barfit.anchor_target` / `runtime.{ass_pos_shift,show_report,report_shown}`；`call` 白名单新增 `ass-pos-shift-toggle` / `report-toggle`（共 30 个）。API 版本仍为 `1.0.0`（只增可选字段）。
- 真机门禁：`tools/sub-style-probe-shift.lua` + `tools/sub-style-e2e.py` 第 **5g** 组（`\pos` 对白整条进黑边、点下移生效、关通道复现原报障、开关可逆，15 条）与第 **5h** 组（`show_report=yes|no` 两轮对照、关闭零输出、`report-toggle` 双向生效，各 11 条）。

### 变更

- 菜单步长 **上移/下移 5% → 1%**（conf `nudge=1`），便于把 `\pos` 内封字幕一点点推进黑边。
- conf 新增 `ass_pos_shift=yes` / `ass_anchor_ratio=0.94` / `show_report=yes`；`sub-style-conf-verify.py` 校验全部键的类型。
- 单元测试新增 **T56**（1% 步长 / PlayResY / Fontsize 补偿 / 黑边反解 / API 快照）与 **T57**（报告开关），328 → **373 条全绿**；`tools/sub-style-probe-fontsize.lua` 的旧「`\pos` 保持原位」断言同步改为「1% 同步位移」。
- 架构文档新增 **§20**（位移公式 `y_render = y_ass × 帧高 / PlayResY`、黑边反解、报告开关设计），API 文档 / README / AGENTS 同步。

### 修复

- 「内封字幕点击下移没有反应，只有外挂字幕有反应」：定位字幕不再保持原位，1% 微调通过 `PlayResY` 同步作用于 `\pos` 字幕。
- 「对话轨道字幕没有在黑边内输出」：黑边贴合对内封 ASS 由反解的 `y_ratio` 把对白盒底送进下黑边，整条字幕完整落入黑边。
- 关闭报告后 `refresh_barfit()` 的「OSD 微扰补回报告」路径曾会把关掉的报告偷偷放回来，已加 `runtime.show_report` 判断。

## [1.0.10] - 2026-11-05

### 新增

- **对外 API v1.0.0**：`user-data/uosc_subtitle_style/version` 与 `/state` 状态广播；
  `script-message-to uosc_subtitle_style api <reply_to> <request_id> <action>` 请求-响应协议，
  支持 `get-state` / `get` / `list-actions` / `call`；`call` 只转发表内白名单动作，禁止递归调用 `api`。
  完整契约见 [docs/api-uosc-subtitle-style.md](docs/api-uosc-subtitle-style.md)。
- **上下黑边双对话字幕字号自适应判据**（T55）：横向比较 `12/12`、`6/6`、`18/6`、`30/30` 黑边，
  证明限缩倍率由较薄黑边决定、上下两轨同倍率、字幕盒不溢出黑边，API 快照暴露 `barfit`。
- README / 架构文档同步；独立仓库门禁脚本补齐 `tools/lua-syntax-check.lua`。

### 变更

- 单元测试 310 → **328 条**；真机 E2E 新增 5f 组双脚本 API 探针（生产整合包 21 条 PASS）。
- `tools/sub-style-e2e.py` 支持**独立仓库布局**：`MPV_EXE` + `--config-dir` 隔离配置、
  临时复制 uosc、无 `uosc_staff_credits` 时收缩期望表而不是伪造 PASS。

## [1.0.9] - 2026-11-05

### 修复

- 修复「下移 5% / 整体缩放点击无效」：真机证实带 `\pos` 的内封 ASS 对 `sub-pos` / `sub-scale` /
  `sub-font-size` 全部豁免；缩放与字号改走 `sub-ass-style-overrides` 的 `Fontsize`。
- `factor == 1` 时不再写入 `Fontsize=20` 这类无效覆盖，避免压平 ASS 多 Style 的原始字号层级。
- nudge 到达边界时不再静默吞掉点击，给出 OSD 反馈；`\pos` 定位字幕保持原位并给出提示。

### 新增

- 字号 ±1px 连续微调，边界可配置；黑边贴合时的放大值自动存进全局预设。
- `tools/sub-style-mkv-fixture.py`：纯 Python 直接写最小 Matroska（MJPEG 单帧 + 内封 ASS `\pos` 事件），
  不依赖 ffmpeg 与 4GB 示例视频。
- `tools/sub-style-probe-fontsize.lua` 逐像素差分探针，量化 `Fontsize` 对 `\pos` 字幕的真实影响。

## [1.0.8] - 2026-11-05

### 新增

- **所有内封字幕轨都要被检查**：枚举 `track-list` 中全部 `external=false` 的 `sub` 轨，
  按轨 id 建 per-file 账本；`sid` / `secondary-sid` 切到未检查轨时自动重新检测。
- 报告新增「内封字幕轨：共 N 条，已检查 M 条（当前 X），待检查 …」，待检查最多列 6 个 id。

### 修复

- 内封字幕除「标准白字黑边」外一律套用黑白预设，不再依赖彩色 / 灰阶窄判据；
  灰阶亮度区间由 `[40,220]` 放宽到 `[30,235]`。
- 切轨重检前清除自动预设，避免 `wants_force()` 为真导致像素采样直接放弃。

## [1.0.7] - 2026-11-04

### 新增

- 视频**内嵌**上下黑边贴合：逐行亮度采样检测黑条，主 / 次对白分别放进下 / 上黑边，
  按较薄一侧自适应字号，不遮挡画面内容。
- 菜单新增「黑边贴合开关 / 重新检测黑边」；新增 `letterbox_*` 配置项。

### 修复

- `sub-margin-y` 是 INT 属性，必须四舍五入；关闭贴合与切换文件时显式还原。

## [1.0.6] - 2026-11-03

### 修复

- 颜色命令里的裸 `#RRGGBB` 被 mpv 命令解析器当注释吞掉：改为带引号参数。
- `sub-ass-style-overrides` 改写 ASS 样式表本身，让带 `\pos` 的顶部字幕 / signs 也能换字体、转黑白。
- 识别灰色字幕（如 `#8B8B8B`）；主色候选加入黑色与灰阶；菜单点击后保持打开并刷新选中态。

## [1.0.5] 及更早版本

- v1.0.5：打开即 OP/ED（无对白）后 seek 到正片不检测 —— seek 后刷新重试额度。
- v1.0.4：内封白字 + 细彩描边漏判彩色 —— 新增 `colored_outline_min` 阈值。
- v1.0.0 – v1.0.3：首个公开版本，双轨分轨样式、字体选择器、预设样式、状态持久化与基础门禁。

## 已知限制

- 图形字幕（PGS / VOBSUB）不走 libass，位置与样式均不可调。
- 带 `\pos` 的 explicit ASS 事件保留原定位，位置微调只作用于普通字幕。
- 粗体是解码器级共享属性，mpv 无分轨粗体通道。
- Linux / macOS 的字体枚举路径已内置，但未经真机验证。

[未发布]: https://github.com/awujiana/uosc_subtitle_style/compare/v1.0.10...HEAD
[1.0.10]: https://github.com/awujiana/uosc_subtitle_style/releases/tag/v1.0.10
[1.0.9]: https://github.com/awujiana/uosc_subtitle_style/compare/v1.0.8...v1.0.9
[1.0.8]: https://github.com/awujiana/uosc_subtitle_style/compare/v1.0.7...v1.0.8
[1.0.7]: https://github.com/awujiana/uosc_subtitle_style/compare/v1.0.6...v1.0.7
[1.0.6]: https://github.com/awujiana/uosc_subtitle_style/compare/v1.0.5...v1.0.6
[1.0.5]: https://github.com/awujiana/uosc_subtitle_style/releases/tag/v1.0.5
