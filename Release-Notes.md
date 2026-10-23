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
- [📦 打包产物与校验](#sec-5)
- [🔄 升级指南](#sec-6)
- [🔗 关联仓库](#sec-7)
- [📮 作者联系方式](#sec-8)
- [📄 开源协议](#sec-9)

---

<a id="sec-1"></a>
## 🏷 1. 版本信息

| 项目 | 内容 |
| --- | --- |
| 版本号 | `v1.0.2` |
| 发布日期 | `2026-10-23` |
| 发布类型 | `Stable` |
| 对应提交 | `b8cf910`（功能） |
| 最低支持 | mpv ≥ 0.41；uosc（可选，缺失则降级） |

<a id="sec-2"></a>
## 🚀 2. 更新亮点

- **黑白字幕不再串台**：彩色字幕转黑白后，切到黑白字幕视频会恢复其原样，不再继承上个彩色视频的预设。
- **中日双语自动布局**：日文轨自动置底 + 缩小（×0.8）、中文轨在其上；`\pos` 注释幕样式 / 位置不变。

<a id="sec-3"></a>
## ✨ 3. 新增功能

- **中日双语自动布局（`dual_layout`，默认开）**：按轨语言识别一中一日，日文轨置底（`sub-pos 100`）+ 缩放 `dual_layout_scale`（默认 0.8），中文轨在其上。布局为 per-file 覆盖层，不写状态文件、不跨文件泄漏，手动调位置 / 缩放即关闭。不需要此行为时设 `dual_layout=no`，即回到 mpv 原样（主底 / 次顶）。

<a id="sec-4"></a>
## 🐞 4. 问题修复

- **彩色预设跨文件串台到黑白字幕**：此前自动预设样式无标记，`file-loaded` 时被重新套到非彩色字幕上（报告却显示「保持原样」）。现为轨加 `auto` 标记，`file-loaded` 先清除自动预设样式再应用，新文件从原样起、检测可靠；用户手动样式仍跨文件持久。

<a id="sec-5"></a>
## 📦 5. 打包产物与校验

| 文件 | 平台 / 架构 | 大小 | SHA-256 |
| --- | --- | --- | --- |
| `uosc_subtitle_style-1.0.2.zip` | 跨平台（mpv 脚本） | 72.5 KB（74261 字节） | `6ef39b9c5df52b0533f1331498ec812ab3c4f51eb2e96934604d5210c988b6d8` |

> 校验：`sha256sum <文件>`（Windows 用 `certutil -hashfile <文件> SHA256`）。产物含 lua + conf + README + CONTRIBUTING + Release-Notes + LICENSE + docs。

<a id="sec-6"></a>
## 🔄 6. 升级指南

1. 备份 `script-opts/uosc_subtitle_style.conf` 与三个 `uosc-subtitle-style-*.json` 数据文件。
2. 从 [Releases](https://github.com/awujiana/uosc_subtitle_style/releases) 下载并解压 `uosc_subtitle_style-1.0.2.zip`，用新的 `scripts/uosc_subtitle_style.lua` 覆盖旧文件（conf 可选覆盖）。
3. 重启 mpv 确认生效。本版无破坏性变更；如不需要双语自动布局，设 `dual_layout=no`。

<a id="sec-7"></a>
## 🔗 7. 关联仓库

- [awujiana/mpv-awujiana](https://github.com/awujiana/mpv-awujiana) — mpv 整合包，本插件的实际运行环境
- [awujiana/uosc_danmaku](https://github.com/awujiana/uosc_danmaku) — 同生态的 uosc 弹幕插件
- [tomasklaen/uosc](https://github.com/tomasklaen/uosc) — 本插件依赖的 uosc UI 框架（第三方）

<a id="sec-8"></a>
## 📮 8. 作者联系方式

- **作者**：awujiana
- **GitHub**：[@awujiana](https://github.com/awujiana)
- **邮箱**：[bingshanlengtie@qq.com](mailto:bingshanlengtie@qq.com)

<a id="sec-9"></a>
## 📄 9. 开源协议

本项目基于 **[AGPL-3.0](LICENSE)** 开源。任何人使用、修改或分发本项目代码，若以网络服务器形式向用户提供交互服务，必须同样以 AGPL-3.0 公开其完整对应源代码。

[![AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
