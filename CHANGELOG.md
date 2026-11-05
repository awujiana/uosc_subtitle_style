# 更新日志

本项目所有值得注意的变更都记录在这里。格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。`version.json`、脚本文件头 `-- @version:` 与 Release tag 三者必须一致。

## [未发布]

暂无。

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
