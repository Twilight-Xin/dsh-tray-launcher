---
id: "e38c38f2-db35-4a4a-a624-5c75e9e29c76"
title: "dsh-tray-launcher 编码守卫与安装目录环境变量"
description: "PowerShell 脚本 UTF-8 BOM 守卫协议、DSH_TRAY_LAUNCHER_PATH 安装目录机制与部署链路要点"
status: "active"
created_at: "2026-10-10T05:55:18.194Z"
updated_at: "2026-10-10T05:55:18.194Z"
content_hash: "2886e80c9c7125aa16010316ad7ad97be79ccab90df359e1eb83c9a53f23487b"
source_paths:
  - "tray.ps1"
  - "install.ps1"
  - "uninstall.ps1"
  - "scripts/check-bom.mjs"
  - "README.md"
session_ids:
  - "session-b2f45ded-59ee-4561-9e4a-d0067e2869e6"
memory_body_ids:
  []
---

# dsh-tray-launcher：编码守卫与安装目录环境变量

## 安装目录机制（feat 分支新增）

- 环境变量 `DSH_TRAY_LAUNCHER_PATH` 优先；`install.ps1` / `uninstall.ps1` 同步解析（Trim 后非空才生效，`GetFullPath` 归一化），未设置/为空回退默认 `%LOCALAPPDATA%\Programs\DSHTray`，生效时打印来源提示。
- 本机实际安装目录：`D:\apps\dshTray`（用户级环境变量已设置）。桌面快捷方式 `DeepSeek Harness.lnk` → `System32\wscript.exe` → `launch-hidden.vbs` → `powershell.exe`（PS 5.1）。

## UTF-8 BOM 规约（关键教训）

- 三个 .ps1（tray/install/uninstall）必须带 UTF-8 BOM。PS 5.1 读无 BOM 的 UTF-8 按系统 ANSI（GBK）解码，中文乱码破坏字符串字面量 → ParserError → 托盘静默起不来；pwsh 7 默认 UTF-8 不受影响——**手工 pwsh 测试通过不代表快捷方式可用**。
- DSH 会话的 write/edit 工具会剥掉 .ps1 的 BOM（带 BOM 探针文件实验证实）。每次修改 .ps1 后必须重补 BOM 并跑 `npm test`。
- 守卫：`scripts/check-bom.mjs`（`npm test` 第一环）；CI 只在 main/PR 触发，feat 分支必须本地跑。5.1 侧验证命令：`powershell.exe -NoProfile -Command "Parser::ParseFile(...)"`。

## 部署链路注意

- 托盘「更新启动器」会用全局 npm 包目录的 `tray.ps1` 覆盖部署副本（Sync-DeployedFiles）。改完仓库先提交并更新全局包，否则下次自更新把旧/无 BOM 文件带回来。
- 托盘互斥体 `Global\dsh-tray-launcher` 静默去重：已有实例在跑时新启动会无提示退出，看起来像"没反应"。

## 事故记录（2026-10-10）

feat 分支编辑三个 .ps1 剥掉 BOM → 快捷方式（5.1）GBK 乱码解析失败；pwsh 手工测试正常掩盖了问题。恢复 BOM、重新部署 `D:\apps\dshTray\tray.ps1`、npm test 全绿后修复。守卫脚本与 CI 本就存在（v1.5.1 同款事故后加的），缺口在 feat 分支未本地执行。
