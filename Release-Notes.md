<div align="center">

[![版本](https://img.shields.io/badge/版本-v1.0.2-blue)](https://github.com/awujiana/uosc_subtitle_style/releases)
[![发布日期](https://img.shields.io/badge/发布日期-2026--10--23-lightgrey)](https://github.com/awujiana/uosc_subtitle_style/releases)
[![类型](https://img.shields.io/badge/类型-正式版-green)](https://github.com/awujiana/uosc_subtitle_style/releases)
[![协议](https://img.shields.io/badge/协议-AGPL--3.0-blue)](LICENSE)
[![AI 辅助生成](https://img.shields.io/badge/AI辅助生成-是-orange)](https://github.com/awujiana)

<h1>uosc_subtitle_style · Release Notes</h1>

<strong>v1.0.2 发布说明</strong>

</div>

---

> [!WARNING]
> 本项目的相当一部分代码由 AI 辅助生成，并经过持续的人工作业、联调和重构，但它仍然可能存在遗漏、边界情况处理不足或行为与预期不完全一致的问题。如果你在使用过程中发现 Bug、兼容性问题、异常行为或文档缺失，欢迎积极提交 Issue。最好附上复现步骤、日志、截图和系统版本信息，这会非常有帮助。

<a id="sec-toc"></a>
## 📚 目录

- [🏷 版本信息](#sec-1)
- [🚀 更新亮点](#sec-2)
- [✨ 新增功能](#sec-3)
- [🐞 问题修复](#sec-4)
- [🔧 改进与调整](#sec-5)
- [📦 打包产物与校验](#sec-6)
- [🔄 升级指南](#sec-7)
- [🙏 致谢](#sec-8)
- [🔗 关联仓库](#sec-9)
- [📮 作者联系方式](#sec-10)
- [📄 开源协议](#sec-11)

---

<a id="sec-1"></a>
## 🏷 1. 版本信息

| 项目 | 内容 |
| --- | --- |
| 版本号 | `v1.0.2` |
| 发布日期 | `2026-10-23` |
| 发布类型 | `Stable`（正式版） |
| 对应提交 | `b8cf910`（功能）；文档规范化见同批后续提交 |
| 构建环境 | Windows 11 ｜ LuaJIT（mpv 0.41 内置）｜ Python 3.14 |
| 最低支持 | mpv ≥ 0.41；uosc（可选，缺失则降级） |

<a id="sec-2"></a>
## 🚀 2. 更新亮点

- **黑白字幕不再串台**：彩色字幕自动转黑白后，切到黑白字幕视频会恢复该字幕原样，不再继承上个彩色视频的预设（内封 / 外挂均适用）。
- **中日双语自动布局**：双轨按语言识别为一中一日时，日文轨自动置底 + 缩小（×0.8）、中文轨在其上；带 `\pos` 的注释幕样式与位置不受影响。
- **文档规范化**：README / CONTRIBUTING 按《我的 README 输出规范》v2.0 重写（AI 辅助生成声明、徽章分组、目录锚点、反馈建议等）。

<a id="sec-3"></a>
## ✨ 3. 新增功能

- **中日双语自动布局（`dual_layout`，默认开）**：`compute_layout()` 按轨语言识别一中一日（内封读 `track-list.lang`，外挂从文件名语言标记 `.chi.`/`.jpn.`/`(JP)` 等猜），日文轨置底（`sub-pos 100`）+ 缩放 `dual_layout_scale`（默认 0.8），中文轨在其上（`100 - pos_min_gap`）。布局是 per-file 覆盖层：不写状态文件、不跨文件泄漏，用户手动调位置 / 缩放即关闭。`\pos` 注释幕对位置与缩放都豁免（真机实测），样式 / 位置保持不变。conf 新增 `dual_layout` / `dual_layout_scale` 两项（`dual_layout=no` 可关闭）。

<a id="sec-4"></a>
## 🐞 4. 问题修复

- 修复 **彩色预设跨文件串台到黑白字幕**：`apply_preset` 写入 `state` 的预设样式此前无标记，`file-loaded` 时被 `apply_all` 重新套到非彩色字幕上，而检测报告却显示「保持原样」——画面与报告不一致；且泄漏的 `force`+白字会干扰内封轨的像素二次检测。现为轨加 `auto` 标记，`file-loaded` 先清除自动预设样式再应用，使新文件从原样起、检测可靠；用户手动样式依旧跨文件持久。

<a id="sec-5"></a>
## 🔧 5. 改进与调整

- 调整 **双轨默认行为**：`dual_layout=yes` 默认开启后，中日双语视频的次轨会被自动置底 + 缩小。如希望保持 mpv 原样（主底 / 次顶），在 `script-opts/uosc_subtitle_style.conf` 设 `dual_layout=no` 即可，无需改代码。
- 调整 **README / CONTRIBUTING**：按《我的 README 输出规范》v2.0 重排——顶部徽章分「静态必含 + 动态统计」两组、新增 AI 辅助生成 `> [!WARNING]` 声明、章节加 emoji 与 `<a id="sec-N">` 健壮锚点、安装改 `<details>` 折叠、新增「文档 / 反馈建议 / 参考与致谢」，测试计数更新为单测 191 / E2E 83。

<a id="sec-6"></a>
## 📦 6. 打包产物与校验

| 文件 | 平台 / 架构 | 大小 | SHA-256 |
| --- | --- | --- | --- |
| `uosc_subtitle_style-1.0.2.zip` | 跨平台（mpv 脚本） | 见资产页 | 见 [GitHub Release 资产页](https://github.com/awujiana/uosc_subtitle_style/releases/tag/v1.0.2) |

> 校验方式：`sha256sum <文件>`（Windows 可用 `certutil -hashfile <文件> SHA256`）。
> 产物内容：`scripts/uosc_subtitle_style.lua` + `script-opts/uosc_subtitle_style.conf` + `README.md` + `CONTRIBUTING.md` + `Release-Notes.md` + `LICENSE` + `docs/`。

<a id="sec-7"></a>
## 🔄 7. 升级指南

1. 备份当前 `script-opts/uosc_subtitle_style.conf` 与 mpv 配置目录下的三个数据文件（`uosc-subtitle-style-*.json`）。
2. 从 [Releases](https://github.com/awujiana/uosc_subtitle_style/releases) 下载 `uosc_subtitle_style-1.0.2.zip` 并解压。
3. 用新的 `scripts/uosc_subtitle_style.lua` 覆盖旧文件；`script-opts/uosc_subtitle_style.conf` 可选覆盖（不放则用内置默认值，新增的 `dual_layout` 项默认 `yes`）。
4. 重启 mpv，确认菜单可用、双语字幕按预期布局。

> 本版无破坏性变更；如不希望双语自动布局，设 `dual_layout=no` 即可回到旧行为。

<a id="sec-8"></a>
## 🙏 8. 致谢

- 运行时依赖 [uosc](https://github.com/tomasklaen/uosc)（LGPL-3.0，不随本仓库分发）。
- 视觉测试判据依赖 [Pillow](https://python-pillow.org)。
- 文档风格参考《我的 README 风格》输出规范。

<a id="sec-9"></a>
## 🔗 9. 关联仓库

- [awujiana/mpv-awujiana](https://github.com/awujiana/mpv-awujiana) — mpv 整合包，本插件的实际运行环境
- [awujiana/uosc_danmaku](https://github.com/awujiana/uosc_danmaku) — 同生态的 uosc 弹幕插件
- [tomasklaen/uosc](https://github.com/tomasklaen/uosc) — 本插件依赖的 uosc UI 框架（第三方）

<a id="sec-10"></a>
## 📮 10. 作者联系方式

- **作者**：awujiana
- **GitHub**：[@awujiana](https://github.com/awujiana)
- **邮箱**：[bingshanlengtie@qq.com](mailto:bingshanlengtie@qq.com)

<a id="sec-11"></a>
## 📄 11. 开源协议

本项目基于 **[AGPL-3.0](LICENSE)**（GNU Affero General Public License v3.0）开源。

根据 AGPL-3.0 的要求：任何人使用、修改或分发本项目代码，若以网络服务器形式向用户提供交互服务，必须同样以 AGPL-3.0 公开其完整对应源代码。

[![AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
