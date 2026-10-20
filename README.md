# 梨课程 · Web 静态站点

南京理工大学课程表「梨课程」的 Web 端静态产物仓库。源码与 Android / iOS /
桌面端在私有工程 `Yuriel404/pear_schedule` 中维护，本仓库只保存构建产物，
方便站点托管、回滚与版本追溯。

- 在线访问：<https://course.njust.store>
- 部署分支：`main`（站点使用该分支的最新静态文件）
- 当前产物：`1.1.8preview`（构建号 10113，预览版）

## 目录说明

| 路径 | 内容 |
| --- | --- |
| `index.html` | 应用入口；首屏加载页的版本号读 `manifest.json` 的 `version` |
| `main.dart.js` / `flutter_bootstrap.js` / `flutter.js` | Flutter Web 运行时 |
| `canvaskit/` | CanvasKit 渲染引擎 |
| `assets/` | 字体、图标、`AssetManifest` 等应用资源 |
| `manifest.json` | PWA 清单，`version` 决定加载页显示的版本号 |
| `version.json` | App 内「关于 → 构建信息」读取的本机版本与构建号 |
| `.nojekyll` | 静态托管侧关闭 Jekyll 处理 |

## 不要手改产物

`main.dart.js`、`assets/`、`canvaskit/`、`manifest.json`、`version.json`
全部由源工程构建生成。需要修复功能或文案时请改源工程，重新构建后再覆盖本仓库；
直接改这些文件会在下一次发布时被整体覆盖，也无法回溯到源码。

## 构建与发布

在源工程根目录执行：

```powershell
# 1. 调整版本（pubspec.yaml 的 version 是唯一事实来源）
#    正式版：1.1.8+10112
#    预览版：源版本用合法的 semver 预发布格式 1.1.8-preview+10113，
#            Web 构建时再用 --build-name 注入展示名 1.1.8preview
dart run tool/release.dart --no-build   # 需要分配新构建号时
dart run tool/sync_web_version.dart     # 同步 web/manifest.json

# 2. 构建并发布到本仓库（脚本会覆盖产物、提交；--tag 打标签，--push 推送）
dart run tool/deploy_web.dart --display-version=1.1.8preview --tag
dart run tool/deploy_web.dart --no-build --display-version=1.1.8preview --tag --push
```

`tool/deploy_web.dart` 的默认部署目录是源工程下的
`.codebuddy/webpage-deploy`，可用 `--repo=<路径>` 指定其他克隆。
它会保留本仓库的 `.git`、`.nojekyll`、`README.md`、`CNAME`、`.gitignore`，
其余文件每次整体覆盖，避免上一版残留。

推送使用维护者配置的远端凭据；当前源工程通过 SSH 别名
`github-njust-openlib` 访问本仓库，必要时可显式指定：

```powershell
dart run tool/deploy_web.dart --no-build --tag --push `
  --push-url=git@github-njust-openlib:NJUST-OpenLib/NJUSTSchedule_WebPage.git
```

## 版本约定

- 文件名不带版本号，线上始终以 `main` 最新提交为准；历史版本通过提交和标签回滚。
- `version.json` 的 `version` + `build_number` 是 App 内显示的版本；
  `manifest.json` 的 `version` 是加载页显示的版本。两者应当一致。
- 正式版构建号由源工程 `tool/release_version.json` 统一分配，不要手改；
  预览版同样递增构建号，避免与正式版混淆。
- 站点不做 Service Worker 强制缓存更新，用户刷新页面即可拿到新产物；
  发布后如需确认，打开页面看加载页版本号是否变化。

## 回滚

```powershell
git log --oneline
git revert <commit>
git push origin main
```

也可以从历史提交恢复整棵产物树后再提交一次：

```powershell
git checkout <commit> -- .
git commit -m "revert(web): 回滚到 <版本>"
git push origin main
```

## 相关仓库

- 源工程：`Yuriel404/pear_schedule`（私有）
- 本仓库：`NJUST-OpenLib/NJUSTSchedule_WebPage`

## 声明

本应用为非官方课程表工具，不代表学校或任何官方机构。课表、成绩等数据来自
学校相关系统，使用前请阅读应用内的《使用协议与隐私政策》。
