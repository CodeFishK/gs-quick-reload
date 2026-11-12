# 更新日志

## v2.3 —— 夜间模式改成「绝对判据 + 周期复查」（versionCode 5）

修一个会让「日落到日出」整晚不生效的判定缺陷。

- **根因**：v2.1 的守护拿「重启前那一刻的昼夜判定」当基准，重启前若已经是错的，
  纠正逻辑就会把错的当成对的。11-12 20:19 那次：日落 17:11，重启前后都是浅色，
  守护写「判定正确」就收工，整晚没变深色，只能手动重设。
- **现在**：每次从 `dumpsys uimode` 读 `sunrise` / `sunset`（拿不到按 19:00–07:00），
  算出「此刻应该深色还是浅色」，不再参考重启前的值。
- 重启后 45 秒、90 秒各查一次；之后每 10 分钟复查、持续 12 小时，
  日落/日出两个边界即使系统没生效也会在 10 分钟内被拉回。
- 你一旦自己改了模式（切成始终浅色/深色），守护立即退出，不跟用户抢。
- 新增 `NIGHT_STEP` / `NIGHT_SPAN` 两个环境变量可调节奏；`night` 子命令同样改为按 Twilight 判定。
- 读取 `dumpsys` 失败的轮次直接跳过，不会因为读不到状态而乱改模式。

## v2.2 —— 机型定位与系统版本订正（versionCode 4）

**修正说明文字**：系统实际为 **Android 17（One UI 9）**，面向机型明确为 **三星 Galaxy Z Fold7**。
脚本逻辑无改动，只改元信息与文档，装了 v2.1 可以不升级。

- `module.prop` / `module.yaml` 的名称改为「一键重载模块（Fold7）」，说明开头即写明
  三星 Galaxy Z Fold7（SM-F966B / One UI 9 / Android 17）专用
- `compatibility.androidMinApi` 由 35（Android 15）改为 **37（Android 17）**，
  与文档一致；内核分支名 `android15-6.6` 是 KMI 命名，与系统版本无关，文档里已注明
- README 新增「适用机型：三星 Galaxy Z Fold7（专用）」一节，说明实测环境与未实测机型的边界

## v2.1 —— 初始发布（versionCode 3）

首个公开版本。一个按钮让 KernelSU 模块生效，而不用拿折叠屏的副屏去换。

- 按 KernelSU 自己的流程执行 `post-fs-data` / `services` 两个阶段，然后只 `setprop ctl.restart zygote`，
  **不执行 `stop` / `start`**，SurfaceFlinger 全程存活，副屏（`port=148`）不掉
- `boot_completed` 看门狗：先扫描模块脚本里有没有 `resetprop -w` 等待者，有才在重启后制造一次
  0→1 边沿；没有就只保留「卡 0 太久 → 置 1」的安全网
- lspd 健康检查：三条硬指标（有 daemon、`lspbridge` 监听 ≥1、pid 不在旧快照）不满足就放弃重启
  zygote，避免「要点两次才生效」
- 夜间模式自愈：重启后若昼夜判定被改，先顶一个固定值再还原用户原本的模式；`trap` 保证任何
  退出路径（终端断开 / 进程被杀）都会还原
- 子命令：`check` 体检、`mods` 清单、`dump <id>` 打包、`trim` 清理重复实例、`night` 一条命令还原自动
- 日志落在 `/sdcard/GS/logs/`，不需要 root 就能读

作者：CodeFishK。实测环境：三星 Galaxy Z Fold7（SM-F966B，One UI 9 / Android 17）+
[GhostSam](https://github.com/snothin/GhostSam) 临时 root。

> 说明：v1.x、v2.0 是作者本机的内部迭代版本，从未对外发布，故不在此列出。
