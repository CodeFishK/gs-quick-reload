# 更新日志

## v2.2（versionCode 4）

仅元信息与文档变更，脚本逻辑与 v2.1 完全相同：

- 作者改为 **CodeFishK**
- 模块说明里补上适用的 root 环境：[GhostSam](https://github.com/snothin/GhostSam)
- `module.yaml` 补 `authors` / `homepage` / `support` 字段
- README 增加「相关项目与致谢」，实测机型处标注 GhostSam 链接

## v2.1（versionCode 3）

- 新增 `night` 子命令：一条命令把夜间模式还原成「自动（日落到日出）」，不需要 root
- 夜间守护加 `trap`：任何退出路径（终端断开 / 进程被 kill）都还原用户原本的模式，
  不再出现「夜间模式卡在固定值」
- 纠正改为最多 3 轮、每轮回读 `mComputedNightMode` 复验
- 实测并写入文档：`cmd uimode night auto` 在判定卡错时不会触发重算，
  必须先给固定值再回 auto

## v2.0（versionCode 2）

- 脚本随模块分发：`action.sh` 三级回退（模块目录 → /data/adb/gs → /data/local/tmp）
- `customize.sh` 自动备份脚本到 `/data/adb/gs/`、创建日志目录

## v1.x

- 首版：走 `ksud post-fs-data` / `services` 两阶段 + `setprop ctl.restart zygote`
- boot_completed 看门狗与边沿发生器
- lspd 健康检查，不健康则放弃重启 zygote（避免「要点两次才生效」）
