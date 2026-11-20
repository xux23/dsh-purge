<div align="center">

# dsh-purge Community

**让 AI 更专注地辅助你的学习与动手实验。**

给 DeepSeek Harness 的自托管控件（俗称「破甲」插件）· 88 条宿主补丁 · 25 层状态自检 · 左侧栏整页设置

[![Release](https://img.shields.io/github/v/release/kingcharleslzy-ai/dsh-purge?label=release)](https://github.com/kingcharleslzy-ai/dsh-purge/releases)
[![Checks](https://github.com/kingcharleslzy-ai/dsh-purge/actions/workflows/community.yml/badge.svg)](https://github.com/kingcharleslzy-ai/dsh-purge/actions/workflows/community.yml)

当前版本 **1.8.0** · 适配 **DSH Desktop 0.2.2-rc.5**（rc.4 / rc.3 / rc.2 / rc.1 / 0.2.1-rc.1 / rc.2 / rc.3 / rc.5 同样通过） · [English](docs/README.en.md)

1.8.0 把 0.2.2-rc.5 / rc.4 纳入已验证宿主：88 条补丁在真实 `app.asar` 解包副本上零漂移、二次应用 `wrote 0`。[发行说明](https://github.com/kingcharleslzy-ai/dsh-purge/releases/tag/v1.8.0)

原项目作者署名为小杨 / YuJunZhiXue；本仓库独立维护社区版，保留上游版权与 MIT 许可证。详见[署名记录](NOTICE.md)与[许可证](LICENSE)。

[立即安装](#安装) · [它会做什么](#它是什么) · [出问题怎么办](#排障)

</div>

---

## 它是什么

跑在你自己机器上的 Harness 附加组件，把日常使用里最耗注意力的事收敛到一处：

- **看清状态** — 安装、补丁、注入是否就绪，一条命令或一屏面板给结论
- **少些往返** — 反复出现的确认与打断，收敛成一次设置
- **随时能回去** — 应用、回滚、开关都留记录，操作前自动快照
- **方便做实验** — 提示词、规则、预设集中管理，改坏了能撤

它降低的是**交互层面的摩擦**，让 agent 能连续执行任务；不改模型本身，也不改变你要做的事。

| 设计原则 | 含义 |
| --- | --- |
| 本机优先 | 全部逻辑在本机运行，不改云端账号、不上传提示词或配置 |
| 可见 | 每个动作都有状态与证据，不靠「应该生效了」这种判断 |
| 可回滚 | 应用 / 安装 / 回滚前自动快照，保留原文件与校验清单 |
| 不替你做决定 | 只减少摩擦，不改变你的目标 |

适用本机学习、个人项目、自动化实验与维护工作。请在你拥有或已获授权的环境中使用。

## 安装

**推荐：让 AI 帮你装。** 把这句话发给能操作本机的 Agent：

> 帮我把 https://github.com/kingcharleslzy-ai/dsh-purge 的最新稳定版装到我正在用的 DSH。先读仓库 AGENTS.md 的安装章节，检查有没有旧版，有就保留设置原地迁移。装好告诉我实际版本和 profile，重启由我自己操作。

### 方式一：DSH 官方界面（首次安装）

1. 打开「插件 → 发现 → 添加索引源」。没有「发现」就先到「插件 → 实验性插件 → 更多」启用「自定义插件市场」。
2. 粘贴这一行，保存后找到 **dsh-purge Community**：

   ```text
   https://kingcharleslzy-ai.github.io/dsh-purge/index.json
   ```

3. 核对仓库为 `kingcharleslzy-ai/dsh-purge`，点「安装」，再到已安装列表确认版本。

「安装源」通常保持原值即可。若出现 `GET https://github.com/…/yaml` 和 404，在「插件 → 添加插件 → 安装源」改成 npm 官方源或你常用的私有源后重试。[三个地址各填哪里](docs/INSTALLATION.md#install-addresses)

### 方式二：DSH 官方 CLI

适用于已装 `dsh` 命令、使用 `web` profile 的 CLI/Web 用户。桌面版用户优先用界面，不要为了装插件再用 `npx` 拉另一套宿主。

```sh
dsh plugin --profile web add "https://github.com/kingcharleslzy-ai/dsh-purge/releases/download/v1.8.0/dsh-purge-community-1.8.0.tgz"
```

`web` 必须与实际启动的 profile 一致。本项目通过 GitHub 发行，**不要换成 `add dsh-purge` 或 `add dsh-purge-community`**，也不是 `npm install -g`。官方 CLI 不做我们的 SHA256 比对与迁移快照，需要这些就选方式三。

### 方式三：社区安装器（备份、旧版迁移、恢复）

先装好并打开一次 DSH，让 profile 初始化。脚本先下载再校验，**请先读过脚本**。`dry-run` 通过只代表前置检查、下载和 SHA256 过了，不代表依赖安装与启动已通过。

**macOS / Linux**

```sh
PURGE_INSTALL_DIR="$(mktemp -d)"
curl -fSL --connect-timeout 20 --max-time 90 "https://raw.githubusercontent.com/kingcharleslzy-ai/dsh-purge/master/scripts/install.sh" -o "$PURGE_INSTALL_DIR/install.sh"
sh "$PURGE_INSTALL_DIR/install.sh" --dry-run
```

校验成功、核对目标实例后，同一终端执行 `sh "$PURGE_INSTALL_DIR/install.sh"`。自定义路径先设 `DSH_APP` / `DSH_HOME`，自定义 profile 两次都加 `--profile=实际名称`。

**Windows Desktop（PowerShell 5.1 / 7）**

```powershell
$PurgeInstaller = Join-Path $env:TEMP ('purge-install-' + [guid]::NewGuid().ToString('N') + '.ps1')
Invoke-WebRequest -UseBasicParsing -TimeoutSec 90 -Uri 'https://raw.githubusercontent.com/kingcharleslzy-ai/dsh-purge/master/scripts/install.ps1' -OutFile $PurgeInstaller
powershell -NoProfile -ExecutionPolicy Bypass -File "$PurgeInstaller" -DryRun
```

退出码为 0、核对目标实例后，同一会话执行 `powershell -NoProfile -ExecutionPolicy Bypass -File "$PurgeInstaller"`。找不到应用或有多个副本时，两次都加 `-DshExe '完整路径'`；自定义目录用 `-DshHome`，profile 用 `-Profile`。

### 安装会改什么，怎么确认

- 迁移前自动备份，调用当前实例的官方 `dsh plugin add`，再核对版本与入口；保留提示词、规则、更新偏好及其他插件设置。
- **不是「完全不碰宿主目录」**：桌面布局的 `app/runtime` 链接缺失或损坏时会修复；Linux 系统宿主只读时可能创建用户副本并切换启动器。
- 不自动应用补丁、不自动重启、不改代理、不修改全局 npm 源、不清缓存。[安装指南](docs/INSTALLATION.md)
- 若市场安装失败（issue #7），失败发生在**插件加载前**，Purge 本身尚未运行；先读脱敏错误、核对依赖来源，不能指望安装器修好所有市场或网络问题。
- 退出码非零就是失败。成功后记下 `backup_id`；运行中的 DSH 仍可能显示旧版，待你方便时手动重开。
- 回滚走「Purge → 状态 → 本机备份」，或按 [CLI 回滚](docs/INSTALLATION.md#cli-rollback)。不要假设 `purge-community` 已在 PATH。

Agent 执行前必须先读 [AGENTS.md §12](AGENTS.md#installation)。

## 面板

左侧栏一整页，不做悬浮按钮：顶部一句话说状态、主操作是「应用」，其余分区折叠。

- 维护页可看最近动作、备份占用、更新偏好
- 会话输入框的回退按钮默认隐藏，在 **Purge → 更多 → 会话外观** 随时开关
- 技能包支持 ZIP / TAR / TGZ，校验路径、条目数与解压大小后才安装
- 反向代理访问时，用 `PURGE_TRUSTED_HOSTS` 列可信主机名；Origin 与跨站检查仍生效

<details>
<summary>在 dsh-go 手机端查看面板</summary>

手机显示的是当前所选宿主的状态与记录。先确认该宿主的 Web 已加载 Purge。

| 当前宿主 | 打开方式 |
| --- | --- |
| 通过 SSH 连接电脑 | 连接后「Chats → Switch to Web」，点左上角「Chats and workspaces」→「Global panels」→「Purge」 |
| 手机本机引擎 | 选「This phone」，等引擎显示「Running」，打开「Chats → All features」→「Purge」 |

验收范围仅限 Android 15 ARM64 模拟器上的**只读**功能（dsh-go 1.0.15 / DSH 0.2.2-rc.1）；真机与写入操作（应用、升级、回滚）尚未验收。

</details>

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `purge-community selfcheck [--json]` | 逐层状态与证据；不适用项单独标注 |
| `purge-community status` | 宿主路径与计数 |
| `purge-community host-status` | 只读查看安装布局与当前入口（不换包、不重启） |
| `purge-community diagnose` | 锚点漂移诊断，给出文件、行号与候选落点 |
| `purge-community reanchor …` | 把确认过的新锚点钉成本地覆盖，可撤销 |
| `purge-community probe [new]` | 生成运行时探针，验证注入确实送达模型 |
| `purge-community audit [--limit n]` | 最近动作记录 |
| `purge-community boost [on\|off]` | 尾部强化段开关 |
| `purge-community preset [status\|install]` | 「无限制模式」预设落盘 |
| `purge-community retry [on\|off]` | 回答被中断时的单次重试开关 |
| `purge-community usage` / `prune` | 备份占用与清理 |
| `purge-community backup` / `rollback <id>` | 快照 / 回滚插件 |

## 兼容性

| 组件 | 值 |
| --- | --- |
| 插件 | `dsh-purge-community` 1.8.0 |
| 已验证宿主 | DSH Desktop 0.2.2-rc.5 / rc.4 / rc.3 / rc.2 / rc.1，0.2.1-rc.1 / rc.2 / rc.3 / rc.5 |
| 0.2.2-rc.5 | 88 条补丁 76 applied / 零漂移，二次应用 `wrote 0`；源码取自真实 `app.asar` 解包副本，桌面壳 5 条同样命中。**本机未点「应用」未重启，不代表真机已加载。** |
| 0.2.2-rc.4 | 同批零漂移、二次应用 `wrote 0`；桌面壳 5 条未在 rc.4 桌面包上单独复测。 |
| 其它宿主 | 允许安装，自检标「未验证」；先看 `diagnose` |
| Node | ≥18（桌面版自带运行时即可）；红队工具需 ≥22.5（`node:sqlite`） |

## <a id="排障"></a>排障

```sh
purge-community selfcheck --json     # 先看哪一层不是 ok
purge-community diagnose             # 锚点漂移时看候选位置
purge-community audit --limit 20     # 最近动作
```

面板打不开时，先确认版本一致，再按[维护文档](docs/MAINTENANCE.md)恢复；**不要直接覆盖 `Resources/app`**。

> **下载时报 `fetch failed`？** 1.6.39 起独立安装器会显示失败阶段与错误码，有限重试后可切系统 curl，两条通道都保留来源、证书和 SHA256 校验。重新下载脚本再跑 dry-run，仍失败就提交脱敏诊断行。[下载排查](docs/maintenance-issue-52.md)

> **想让某些预设保留自己的提示词？** 用 `excludePresets`（默认空，行为不变）把 `roleplay`、`minimal` 等排除在注入之外，命令、面板与工具照常保留；不撤销全局宿主补丁。[按预设排除](docs/maintenance-issue-53.md)

## 更新

维护页分别设置「自动检查」与「自动安装」，默认只检查，间隔 6 小时。安装完成不会自动重启——什么时候重开由你决定。

## 开发与验证

```sh
npm test                 # 单元与契约测试
npm run check:docs       # 版本与文档一致性
npm run check:client     # 面板 bundle 与源码一致
npm run test:electron    # 真实 Electron 环境（临时目录，不改安装）
npm run smoke            # 发行包冒烟
```

发行版本号补丁位使用 0–99、不补前导零；历史版本保留原编号，见[发行版本号](docs/MAINTENANCE.md#发行版本号)。面向 AI agent 的操作说明在 [AGENTS.md](AGENTS.md)，两者冲突以它为准。

**关键词**：破甲 · 无限制模式 · unrestricted · prompt injection · jailbreak · DSH 插件 · 状态自检 · 备份回滚

## 贡献者

感谢原作者、代码贡献者，以及提供问题复现、分析与验证的朋友。具体贡献见 [CONTRIBUTORS.md](CONTRIBUTORS.md)，第三方来源见 [NOTICE.md](NOTICE.md)。

<!-- CONTRIBUTORS:START -->
<table>
  <tr>
    <td align="center" width="120"><a href="https://github.com/YuJunZhiXue"><img src="https://avatars.githubusercontent.com/u/188421097?s=112" width="56" height="56" alt="YuJunZhiXue" /><br /><sub>YuJunZhiXue</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/kingcharleslzy-ai"><img src="https://avatars.githubusercontent.com/u/260348840?s=112" width="56" height="56" alt="kingcharleslzy-ai" /><br /><sub>kingcharleslzy-ai</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/luowang-ui"><img src="https://avatars.githubusercontent.com/u/62333101?s=112" width="56" height="56" alt="luowang-ui" /><br /><sub>luowang-ui</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/heigit"><img src="https://avatars.githubusercontent.com/u/8845668?s=112" width="56" height="56" alt="heigit" /><br /><sub>heigit</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/OGHong"><img src="https://avatars.githubusercontent.com/u/130678941?s=112" width="56" height="56" alt="OGHong" /><br /><sub>OGHong</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Wangxiaolu0716"><img src="https://avatars.githubusercontent.com/u/110671501?s=112" width="56" height="56" alt="Wangxiaolu0716" /><br /><sub>Wangxiaolu0716</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/wtfsb19"><img src="https://avatars.githubusercontent.com/u/70636030?s=112" width="56" height="56" alt="wtfsb19" /><br /><sub>wtfsb19</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/MingShi350"><img src="https://avatars.githubusercontent.com/u/47762455?s=112" width="56" height="56" alt="MingShi350" /><br /><sub>MingShi350</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/loLollipop"><img src="https://avatars.githubusercontent.com/u/168881978?s=112" width="56" height="56" alt="loLollipop" /><br /><sub>loLollipop</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/youki258"><img src="https://avatars.githubusercontent.com/u/177468176?s=112" width="56" height="56" alt="youki258" /><br /><sub>youki258</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/SimonQvQ"><img src="https://avatars.githubusercontent.com/u/154972131?s=112" width="56" height="56" alt="SimonQvQ" /><br /><sub>SimonQvQ</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/alexopenfan-xiaxin"><img src="https://avatars.githubusercontent.com/u/267281196?s=112" width="56" height="56" alt="alexopenfan-xiaxin" /><br /><sub>alexopenfan-xiaxin</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/Minervaowl7"><img src="https://avatars.githubusercontent.com/u/127482476?s=112" width="56" height="56" alt="Minervaowl7" /><br /><sub>Minervaowl7</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/ShiTan-677"><img src="https://avatars.githubusercontent.com/u/91113050?s=112" width="56" height="56" alt="ShiTan-677" /><br /><sub>ShiTan-677</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Minggle"><img src="https://avatars.githubusercontent.com/u/35053182?s=112" width="56" height="56" alt="Minggle" /><br /><sub>Minggle</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Steooenwolf-666"><img src="https://avatars.githubusercontent.com/u/156271754?s=112" width="56" height="56" alt="Steooenwolf-666" /><br /><sub>Steooenwolf-666</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/a320583781-create"><img src="https://avatars.githubusercontent.com/u/234735967?s=112" width="56" height="56" alt="a320583781-create" /><br /><sub>a320583781-create</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/longyanbo1224-debug"><img src="https://avatars.githubusercontent.com/u/334267419?s=112" width="56" height="56" alt="longyanbo1224-debug" /><br /><sub>longyanbo1224-debug</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/0xnuu"><img src="https://avatars.githubusercontent.com/u/113989865?s=112" width="56" height="56" alt="0xnuu" /><br /><sub>0xnuu</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/21hbguo"><img src="https://avatars.githubusercontent.com/u/139569698?s=112" width="56" height="56" alt="21hbguo" /><br /><sub>21hbguo</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Akiiia"><img src="https://avatars.githubusercontent.com/u/74829345?s=112" width="56" height="56" alt="Akiiia" /><br /><sub>Akiiia</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/AkiraNyx"><img src="https://avatars.githubusercontent.com/u/176532870?s=112" width="56" height="56" alt="AkiraNyx" /><br /><sub>AkiraNyx</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/ankepoipoi"><img src="https://avatars.githubusercontent.com/u/314255825?s=112" width="56" height="56" alt="ankepoipoi" /><br /><sub>ankepoipoi</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/cracer4869"><img src="https://avatars.githubusercontent.com/u/46320829?s=112" width="56" height="56" alt="cracer4869" /><br /><sub>cracer4869</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/djs11eee-png"><img src="https://avatars.githubusercontent.com/u/357155267?s=112" width="56" height="56" alt="djs11eee-png" /><br /><sub>djs11eee-png</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/dropcake333"><img src="https://avatars.githubusercontent.com/u/291205689?s=112" width="56" height="56" alt="dropcake333" /><br /><sub>dropcake333</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/fomaper"><img src="https://avatars.githubusercontent.com/u/38674512?s=112" width="56" height="56" alt="fomaper" /><br /><sub>fomaper</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Kilig1058674225"><img src="https://avatars.githubusercontent.com/u/68159061?s=112" width="56" height="56" alt="Kilig1058674225" /><br /><sub>Kilig1058674225</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Legeo969"><img src="https://avatars.githubusercontent.com/u/79903511?s=112" width="56" height="56" alt="Legeo969" /><br /><sub>Legeo969</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/LinyiAIGC"><img src="https://avatars.githubusercontent.com/u/68159061?s=112" width="56" height="56" alt="LinyiAIGC" /><br /><sub>LinyiAIGC</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/lyufeng000"><img src="https://avatars.githubusercontent.com/u/168790556?s=112" width="56" height="56" alt="lyufeng000" /><br /><sub>lyufeng000</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Musan-34"><img src="https://avatars.githubusercontent.com/u/279201993?s=112" width="56" height="56" alt="Musan-34" /><br /><sub>Musan-34</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/nanirise"><img src="https://avatars.githubusercontent.com/u/222086913?s=112" width="56" height="56" alt="nanirise" /><br /><sub>nanirise</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/qianxuno"><img src="https://avatars.githubusercontent.com/u/179305118?s=112" width="56" height="56" alt="qianxuno" /><br /><sub>qianxuno</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/TLcmmmm"><img src="https://avatars.githubusercontent.com/u/167976723?s=112" width="56" height="56" alt="TLcmmmm" /><br /><sub>TLcmmmm</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/Tofu707"><img src="https://avatars.githubusercontent.com/u/178003727?s=112" width="56" height="56" alt="Tofu707" /><br /><sub>Tofu707</sub></a></td>
  </tr>
  <tr>
    <td align="center" width="120"><a href="https://github.com/xinspark"><img src="https://avatars.githubusercontent.com/u/109896752?s=112" width="56" height="56" alt="xinspark" /><br /><sub>xinspark</sub></a></td>
    <td align="center" width="120"><a href="https://github.com/YifanHere"><img src="https://avatars.githubusercontent.com/u/62582492?s=112" width="56" height="56" alt="YifanHere" /><br /><sub>YifanHere</sub></a></td>
  </tr>
</table>
<!-- CONTRIBUTORS:END -->

名单合并 GitHub 提交者与人工鸣谢记录，自动每日刷新并保留历史贡献者。头像顺序不代表贡献排名。

### DSH Store 收录信息

申请必须填 **Plugin path：`.`**（安装根目录的 `dsh-purge-community`），不要留空或选独立面板包。逐版本兼容声明在 manifest 里，未验证版本保持 `unknown`；元数据修正不等于市场已批准安装。[路径、证据与收录边界](docs/DSH-STORE.md)
