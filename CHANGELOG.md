# 更新日志

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

作者：CodeFishK。实测环境：[GhostSam](https://github.com/snothin/GhostSam) 临时 root（三星 Z Fold7），
理论上任意 KernelSU / 兼容分支通用。

> 说明：v1.x、v2.0 是作者本机的内部迭代版本，从未对外发布，故不在此列出。
