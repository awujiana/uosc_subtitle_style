<div align="center">

[![协议](https://img.shields.io/badge/协议-AGPL--3.0-blue)](LICENSE)
[![语言](https://img.shields.io/badge/语言-简体中文-blueviolet)](https://zh.wikipedia.org/wiki/中文)
[![平台](https://img.shields.io/badge/平台-mpv%20%3E%3D%200.41-orange)](https://mpv.io)
[![版本](https://img.shields.io/badge/版本-v1.0.1-green)](https://github.com/awujiana/uosc_subtitle_style/releases)
[![作者](https://img.shields.io/badge/作者-awujiana-orange)](https://github.com/awujiana)

# uosc_subtitle_style

**mpv 内封字幕样式快速切换 —— 基于 uosc 菜单，主 / 次字幕（双语）分轨管理**

</div>

---

## 目录

- [uosc\_subtitle\_style](#uosc_subtitle_style)
  - [目录](#目录)
  - [项目简介](#项目简介)
  - [核心功能](#核心功能)
  - [快速开始](#快速开始)
    - [安装（30 秒）](#安装30-秒)
    - [绑定按键（可选）](#绑定按键可选)
    - [运行依赖](#运行依赖)
  - [使用说明](#使用说明)
    - [uosc 菜单](#uosc-菜单)
    - [预览](#预览)
    - [命令消息（script-message）](#命令消息script-message)
    - [数据文件](#数据文件)
  - [在 uosc 控制栏加按钮（uosc.conf）](#在-uosc-控制栏加按钮uoscconf)
    - [语法](#语法)
    - [在哪改](#在哪改)
    - [示例一：加一个「字幕样式」按钮（呼出菜单）](#示例一加一个字幕样式按钮呼出菜单)
    - [示例二：加多个按钮（菜单 / 字体选择器 / 重新扫描字体）](#示例二加多个按钮菜单--字体选择器--重新扫描字体)
    - [示例三：状态切换按钮（cycle，进阶）](#示例三状态切换按钮cycle进阶)
  - [配置项](#配置项)
  - [兼容性与注意事项](#兼容性与注意事项)
  - [测试](#测试)
  - [来源与许可](#来源与许可)
  - [关联仓库](#关联仓库)
  - [作者联系方式](#作者联系方式)
  - [开源协议](#开源协议)

---

## 项目简介

**uosc_subtitle_style** 是一个 mpv Lua 脚本，让你在播放途中通过 uosc 菜单一键调整**内封（内嵌）字幕**的样式：位置、缩放、字体、字号、粗体、主色、描边粗细与描边颜色。不需要记命令行属性，不需要重载字幕，点几下菜单即可。

它要解决的真问题有三个：

1. **内封字幕改样式会卡住**。mpv 的 `sub-reload` 与解码器重建会让已消费的字幕包丢失——外挂字幕因为 `sub_preload` 全量重载而侥幸无恙，**内封字幕必然中招**（表现为「字幕定住一句不跳」）。本脚本的核心工程约束就是**绝不轻易重建解码器**：位置/缩放走属性即时生效；字体/字号/描边在已是 `force` 时只写属性 + 一次 OSD 微扰；只有进出 `force` 的瞬间才允许重建一次。
2. **双语字幕（双轨）没有分轨样式通道**。mpv 的字体系列属性是单份值，主/次解码器共享。本脚本采用「编辑目标」制：菜单顶部切换主字幕/次字幕，样式区只作用于目标轨，另一轨恢复原样（各自的记忆保留，切回即恢复）。位置与缩放则是两轨完全独立的属性，并带**自动避让**（两轨至少相差一档，不重叠）。
3. **预设改色容易误伤原版字幕**。载入时可自动套用预设样式（白字/标准描边），但仅当字幕本身是彩色的才动；**双语字幕（双轨）默认保持原样不改颜色**——两条都是可配置、可热切换的。

## 核心功能

| 功能 | 说明 |
| --- | --- |
| 位置一键切换 | 原位 / 上移 / 中下 / 中部 / 中上 / 顶部 六档预设，另支持 ±5% 微调与整体缩放 |
| 主 / 次字幕分轨 | 菜单顶部切换编辑目标；位置/缩放/字体/字号/描边分轨生效，另一轨恢复原样 |
| 位置自动避让 | 两轨 `sub-pos` 至少相差 `pos_min_gap`（默认 25），目标轨设新位置时另一轨自动让位 |
| 字体选择器 | 枚举本机全部 sfnt/TTC 字体（含 CJK 标记），懒加载 + 签名缓存，支持重新扫描 |
| 样式覆盖 | 字体、字号（720p 等效像素）、粗体、主色、描边粗细、描边颜色 |
| 预设样式 | 载入时自动套用；仅彩色字幕生效（主色或描边任一为彩即算）、双语（双轨）默认跳过；可改可存可恢复出厂 |
| 彩色预设不串台 | 自动套用预设只对彩色字幕生效；切到黑白字幕自动恢复该字幕原样，不继承上个彩色视频的预设（内封/外挂均适用） |
| 中日双语布局 | 双轨按语言识别为一中一日时，日文轨自动置底 + 缩小（默认 ×0.8）、中文轨在其上；带 `\pos` 的注释幕样式/位置不变 |
| 样式检测报告 | 载入后自动解析外挂 ASS / 内封字幕配色，左上角报告「已套用 / 保持原样 / 双语跳过」及原因 |
| 状态持久化 | 当前样式跨文件、跨重启记忆（`persist=yes`，数据文件在 mpv 配置目录） |
| 无 uosc 降级 | 未检测到 uosc 时菜单降级为位置循环切换 + OSD 文本列表 |

## 快速开始

### 安装（30 秒）

**方式一：直接拖文件夹（推荐）**

把本仓库 `scripts/` 文件夹里的 `uosc_subtitle_style.lua`，拖进 mpv 配置目录的 `scripts/` 文件夹：

```text
portable_config/                  ← mpv 配置目录（便携版在 mpv.exe 旁；
├── scripts/                         标准版在 ~/.config/mpv/）
│   └── uosc_subtitle_style.lua  ← 拖到这里
└── script-opts/
    └── uosc_subtitle_style.conf ← 可选：不放则全部用内置默认值
```

> [!IMPORTANT]
> 拖的是 `scripts/` **里面的那个 `.lua` 文件**，不要把整个 `scripts` 文件夹再套一层
> （`scripts/scripts/` 这种嵌套 mpv 不会加载）。

**方式二：Releases 下载**

到 [Releases](https://github.com/awujiana/uosc_subtitle_style/releases) 页面下载
`uosc_subtitle_style-1.0.1.zip`，解压后按上面的目录树放到对应位置即可。

### 绑定按键（可选）

在 `input.conf` 里加一行，即可一键呼出字幕样式菜单：

```conf
ALT+Q script-message-to uosc_subtitle_style open-menu    # 字幕样式菜单
```

### 运行依赖

- **mpv ≥ 0.41**（用到 `secondary-sub-pos` 等分轨属性，低版本无此属性）。
- **uosc**（[tomasklaen/uosc](https://github.com/tomasklaen/uosc)）：菜单由它渲染。
  没装 uosc 也能用——自动降级为位置循环切换。

## 使用说明

### uosc 菜单

呼出菜单后，顶部是**编辑目标**（主字幕 / 次字幕），下面是位置预设、缩放、字体、字号、
粗体、主色、描边、预设样式等区。所有修改即时生效并写入记忆；次字幕加载与否自动决定
「次字幕」项是否可点。


### 预览
[![image.png](https://img.meituan.net/content/50ff6c2b3c5bbe2b66cc363ac948bc85356475.png)](https://img.meituan.net/content/50ff6c2b3c5bbe2b66cc363ac948bc85356475.png)
[![image.png](https://img.meituan.net/content/1e35c10e718a0c79f640fa255e3b1115376117.png)](https://img.meituan.net/content/1e35c10e718a0c79f640fa255e3b1115376117.png)
[![image.png](https://img.meituan.net/content/736a4a418027744a4cf2f8e23b2f233f429096.png)](https://img.meituan.net/content/736a4a418027744a4cf2f8e23b2f233f429096.png)
[![image.png](https://img.meituan.net/content/7856c0c121fb14cee2286b83d5d1772d417958.png)](https://img.meituan.net/content/7856c0c121fb14cee2286b83d5d1772d417958.png)
[![image.png](https://img.meituan.net/content/049ea738320986af80c11bcc7e461259322402.png)](https://img.meituan.net/content/049ea738320986af80c11bcc7e461259322402.png)


### 命令消息（script-message）

所有功能都可以用 `script-message-to uosc_subtitle_style <消息> [参数]` 驱动，
方便自绑按键或接入其它菜单：

| 消息 | 作用 |
| --- | --- |
| `open-menu` | 打开 uosc 菜单（无 uosc 时降级循环切换） |
| `cycle` | 循环切换目标轨的位置预设 |
| `target <main\|sec>` | 切换编辑目标轨道 |
| `pos <0-150>` | 设置目标轨 `sub-pos` |
| `scale <倍率>` | 设置目标轨 `sub-scale` |
| `nudge <±%>` | 以当前值为基准微调目标轨 `sub-pos` |
| `font <字体名\|reset>` | 字体覆盖（字体名含空格也不用引号，参数自动拼回） |
| `size <像素值\|reset>` | 字号覆盖（720p 等效像素） |
| `bold <on\|off\|reset>` | 粗体覆盖 |
| `color <#RRGGBB\|reset>` | 主颜色（`sub-color`） |
| `outline <粗细\|reset>` | 描边粗细（`sub-outline-size`） |
| `outline-color <#RRGGBB\|reset>` | 描边颜色（`sub-outline-color`） |
| `style-preset <big-bold\|original\|sync-other>` | 快捷预设 |
| `open-fonts` / `rescan-fonts` | 本机字体选择器 / 重新扫描 |
| `preset-apply` / `preset-save` / `preset-reset` | 套用 / 保存当前样式为预设 / 恢复出厂预设 |
| `preset-toggle-colored` | 切换「仅彩色字幕才套用预设」 |
| `preset-toggle-dual` | 切换「双语字幕（双轨）时保持原样」 |
| `preset-toggle-enabled` | 预设总开关 |
| `inspect` | 重新运行样式检测并出报告 |
| `reset` | 恢复默认并清除记忆 |
| `fix-override` | 把目标轨 `sub-ass-override` 设为 `scale`（位置/样式的前置条件） |

### 数据文件

运行时在 mpv 配置目录（`~~/`）写三个 JSON，删除即重置：

| 文件 | 内容 |
| --- | --- |
| `uosc-subtitle-style-state.json` | 当前各轨样式记忆（位置/缩放/字体/字号/粗体/颜色/描边） |
| `uosc-subtitle-style-preset.json` | 预设样式（颜色/描边/字体/字号/粗体/开关） |
| `uosc-subtitle-style-fonts.json` | 本机字体枚举缓存（按目录签名失效） |

## 在 uosc 控制栏加按钮（uosc.conf）

uosc 的播放控制栏（`controls`）是在它**自己的**配置文件 `script-opts/uosc.conf` 里定义的
（不是本脚本的 `uosc_subtitle_style.conf`）。本脚本的所有功能都通过
`script-message-to uosc_subtitle_style <消息>` 暴露（见上方「命令消息」表），所以只要在
uosc 的 `controls` 里加一个 `command:` 按钮、把点击动作指向这条消息，就能在控制栏上多一个
一键触发本脚本的按钮。

### 语法

`controls` 是一串**逗号分隔**的条目，每个条目可选带可见性条件前缀 `<条件>`，整体格式：

```text
<可见条件>command:<图标名>:<点击时执行的 mpv 命令>?<悬停提示>
```

- **可见条件（可选）**：`<has_sub>` 仅当有字幕时显示；常用还有 `<video>` / `<audio>` /
  `<idle>` / `<stream>`，取反加 `!`（如 `<!idle>` 表示非空闲时）。多个条件用 `,` 或
  ` *+ *` 组合成「或 / 与」组（例如 `<video,audio>`、`<audio+!video>`）。
- **kind**：`command:` 表示「点击执行一条 mpv 命令」；此外还有 `button:`（uosc 内置按钮名，
  如 `subtitles` / `fullscreen`）、`script-binding:`（脚本绑定）、`cycle:`（状态切换按钮）、
  `space` / `gap`（分隔）。本脚本用 `command:` 最合适。
- **图标名**：Material Symbols 图标名，例如 `tune`（调节）、`format_color_text`（文字颜色）、
  `text_fields`（字体）、`font_download`（字体下载）、`refresh`（重新扫描）。找不到的图标名
  会被渲染成文字，换一个即可。
- **命令**：填 `script-message-to uosc_subtitle_style <消息>`，`<消息>` 取上方表格任意一项。
- **悬停提示**：`?` 后面是鼠标悬停显示的文字（中文直接写，无需引号）。

### 在哪改

打开 mpv 配置目录下的 `script-opts/uosc.conf`，找到里面那一长行 `controls=...`，
把新条目**用逗号**拼进现有列表即可（建议放在靠近 `subtitles` 的位置）。

### 示例一：加一个「字幕样式」按钮（呼出菜单）

```conf
<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式
```
[![fbba665ca97f2816.png](https://img.meituan.net/content/4290cf0f128fad52ebe9db2cecdc99a48169.png)](https://img.meituan.net/content/4290cf0f128fad52ebe9db2cecdc99a48169.png)
- `<has_sub>`：只有加载了字幕（内封或外挂）才显示这个按钮；
- `tune`：用「调节」图标；
- 点击执行 `script-message-to uosc_subtitle_style open-menu`，即呼出本脚本菜单；
- `?字幕样式`：鼠标悬停提示。

拼到 `controls=` 里长这样（片段示意）：

```conf
controls=...,<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式,<has_sub>subtitles,...
```

### 示例二：加多个按钮（菜单 / 字体选择器 / 重新扫描字体）

```conf
<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式,
<has_sub>command:font_download:script-message-to uosc_subtitle_style open-fonts?字体选择器,
<has_sub>command:refresh:script-message-to uosc_subtitle_style rescan-fonts?重新扫描字体
```

（三个条目用逗号连写；`<has_sub>` 让它们在无字幕时自动隐藏。）

### 示例三：状态切换按钮（cycle，进阶）

若希望按钮能反映「预设开关」的开 / 关并随状态换图标，需要本脚本先注册一个可被 uosc 观察的
属性。本脚本当前**未**注册此类属性，所以最稳妥是用上面的 `command:` 按钮触发
`preset-toggle-enabled`（点击即切换，但图标不随状态变化）。等脚本注册了属性后，可用
`cycle:` 语法（4 个参数：kind、观察属性、点击命令、状态组）：

```conf
<has_sub>cycle:prop_name:script-message-to uosc_subtitle_style preset-toggle-enabled:on=toggle_on/off=toggle_off?预设开关
```

> [!TIP]
> - 改完保存 `uosc.conf` 后**重启 mpv** 才生效（uosc 不热加载 `controls` 配置）。
> - 图标名清单见 [uosc 仓库](https://github.com/tomasklaen/uosc)（Material Symbols）；
>   图标名写错只会显示成文字，换一个正确的即可，不影响功能。
> - 不想要按钮了，把对应条目从 `controls=` 里删掉即可。

## 配置项

全部配置在 `script-opts/uosc_subtitle_style.conf`（该文件带中文注释，25 项与脚本默认值一一对应）。
不放此文件 = 全部使用默认值。常用几项：

| 配置 | 默认 | 说明 |
| --- | --- | --- |
| `persist` | `yes` | 跨文件/重启记忆当前样式 |
| `pos_default` / `sec_pos_default` | `100` / `0` | 主/次字幕默认位置（mpv 原生默认值） |
| `pos_min_gap` | `25` | 两轨位置避让的最小间距（% 屏幕高度） |
| `dual_layout` | `yes` | 中日双语(双轨)自动布局：按语言识别，日文轨置底+缩小、中文轨其上；注释幕(`\pos`)不变 |
| `dual_layout_scale` | `0.8` | 双语布局中日文轨的缩放倍率（视觉上日文小于中文） |
| `nudge` | `5` | 菜单微调步长 |
| `fonts` | Noto CJK 四款 | 字体候选列表（逗号分隔） |
| `sizes` / `colors` / `outlines` / `outline_colors` | 见 conf | 各属性候选（`标签=值` 或裸值） |
| `preset_enabled` | `yes` | 载入时自动套用预设总开关 |
| `preset_only_colored` | `yes` | 仅当字幕本身是彩色时才套用 |
| `preset_skip_dual` | `yes` | **双语字幕（双轨）默认不套用、不改颜色** |
| `preset_color` / `preset_outline` / `preset_outline_color` | `#FFFFFF` / `2` / `#000000` | 预设主色/描边粗细/描边色 |
| `preset_font` / `preset_size` / `preset_bold` | 空 / `0` / `keep` | 预设字体/字号/粗体（空或 `keep` = 不动） |
| `colored_threshold` | `40` | 判定「彩色字幕」的最小饱和度（max-min(RGB)，主色与描边色分别判定，任一达标即算） |
| `inspect_delay` / `report_ms` | `1.2` / `6000` | 载入后检测等待（秒）/ 报告显示时长（毫秒） |
| `rebuild_delay` | `0.12` | override 切换的延迟设置（秒） |
| `fallback_cycle` | `yes` | 无 uosc 时降级为循环切换 |

> [!NOTE]
> mpv 的 conf **不支持行尾注释**（`key=value  # 注释` 会把整段当值），所以仓库里的
> conf 注释全部独立成行；自己改的时候请保持这个写法。

## 兼容性与注意事项

- **mpv ≥ 0.41**：依赖 `secondary-sub-pos` / `secondary-sub-scale` 等分轨属性。
- **ASS 字幕**功能完整；**图形字幕（PGS/VOBSUB）**不走 libass，位置/样式均不可调。
- **外挂 ASS 与内封字幕**都支持；样式检测对外挂解析 `[V4+ Styles]`，内封用截图差分。
- 字体/字号/描边的修改只对**目标轨**生效；**粗体是解码器级共享**（mpv 无分轨粗体通道，菜单已注明）。
- 带 `\pos` 的 signs/特效幕不受位置与样式修改影响（libass explicit 豁免）。
- Windows 实测（mpv 0.41）；Linux/macOS 的字体枚举路径已内置，理论可用但未经真机验证。
- 首次打开字体选择器会有一次全盘字体冷枚举（约 1-3 秒），之后走签名缓存。

## 测试

仓库自带完整的单测与真机端到端测试（改代码后请先跑门禁，判据是退出码与汇总行）：

```bash
# 单元测试（191 断言，stub 掉 mpv API，任意目录可跑）
../luajit.exe tools/sub-style-test.lua

# conf 静态门禁（按 mpv 解析语义复算：键集合/布尔/数值/字符串含 # ）
C:/Python314/python.exe tools/sub-style-conf-verify.py

# 真机 E2E（83 断言：分轨/状态/重置/字体选择器/字幕流连续性/预设六用例/截图视觉判据）
# 需要 mpv.exe；Pillow 视觉判据需要装 Pillow 的 Python
MPV_EXE=/path/to/mpv.exe python tools/sub-style-e2e.py
```

E2E 的定位顺序：`MPV_EXE` 环境变量 → PATH → 仓库同级目录的 `mpv.exe`。
uosc 是可选依赖：探测到（`UOSC_MAIN` 环境变量，或 mpv 旁 `portable_config/scripts/uosc/main.lua`）
时自动复制进 `scripts/uosc/` 由 mpv 自动加载，字体选择器三项断言参与实测；
探测不到时三项记 SKIP 并打印原因，不影响其余断言。

## 来源与许可

- 运行时依赖 [uosc](https://github.com/tomasklaen/uosc)（LGPL-3.0，**不随本仓库分发**，
  菜单经 `script-message` 与其交互）。
- 视觉测试判据依赖 [Pillow](https://python-pillow.org)（仅测试需要，运行时不依赖）。
- 本仓库自身的 Lua / Python / conf 代码与文档基于 [AGPL-3.0](LICENSE) 开源。

## 关联仓库

- [awujiana/mpv-awujiana](https://github.com/awujiana/mpv-awujiana) — mpv 整合包，本插件的实际运行环境
- [awujiana/uosc_danmaku](https://github.com/awujiana/uosc_danmaku) — 同生态的 uosc 弹幕插件
- [tomasklaen/uosc](https://github.com/tomasklaen/uosc) — 本插件依赖的 uosc UI 框架（第三方）

## 作者联系方式

- GitHub：[@awujiana](https://github.com/awujiana)
- 邮箱：[bingshanlengtie@qq.com](mailto:bingshanlengtie@qq.com)
- 问题反馈：[Issues](https://github.com/awujiana/uosc_subtitle_style/issues) ｜ [Discussions](https://github.com/awujiana/uosc_subtitle_style/discussions)

## 开源协议

本项目基于 [AGPL-3.0](LICENSE) 开源。任何人使用、修改或分发本项目的代码，
若以网络服务器形式向用户提供交互服务，必须同样以 AGPL-3.0 公开其完整对应源代码。
