# order_manager

OnePoint 桌面程序（Wails + Go）的**版本分发仓库**。这里只放 Release 安装包，不放源码。

## 客户端是怎么更新的

```
程序启动
  -> GET https://api.github.com/repos/Aemiser/order_manager/releases/latest
  -> 语义化比较 tag 与程序内 global.Version
  -> 有新版本、且 Release 带 OnePoint.exe 附件时，弹出「发现新版本」
  -> 下载 exe 到系统临时目录，校验文件大小
  -> 退出程序，由 bat 脚本替换 exe 并重新启动
```

配置位于程序目录下的 `settings.yaml`：

```yaml
version:
    repo: "Aemiser/order_manager"
```

## 发布一个新版本

1. 修改源码 `go/global/enter.go` 中的 `Version`，例如 `10.0.2`。
2. `wails build`，产物为 `build/bin/OnePoint.exe`。
3. 执行一键发布脚本（需 `GITHUB_TOKEN`，权限 `Contents: write`）：

   ```powershell
   $env:GITHUB_TOKEN = 'ghp_xxxxxxxxxxxx'
   powershell -ExecutionPolicy Bypass -File .\scripts\publish-release.ps1 -Version 10.0.2
   ```

   或者在 GitHub 网页上手动新建 Release 并拖入 exe。

### 必须遵守的规则

| 项目 | 要求 |
| --- | --- |
| Tag | 与程序内版本号一致，可用 `v` 前缀，如 `10.0.2` / `v10.0.2`；必须是语义化递增版本 |
| 附件 | 必须有 `OnePoint.exe`（`.exe` 结尾）；多个 exe 时客户端优先取同名文件 |
| 状态 | **不要**勾选 draft 或 pre-release，`/releases/latest` 不会返回它们 |
| 仓库可见性 | 必须保持 **Public**，客户端不带 token，私有仓库会得到 404 |

发布后可用匿名请求自检：

```
https://api.github.com/repos/Aemiser/order_manager/releases/latest
```

确认 `tag_name` 是刚发布的版本、`assets` 里有 `OnePoint.exe` 且 `browser_download_url` 可下载。

## 注意事项

- **不要**把 `settings.yaml` 上传到此仓库，其中含数据库密码、邮箱授权码、模型 API Key 等敏感配置。
- Release 单个附件上限 2 GB，当前 exe 约 24 MB，无压力。
- GitHub 匿名 API 有频率限制（每 IP 每小时 60 次），客户端每次启动只请求一次，正常使用不会超限。
- 历史 Release `1.0.0` 是空壳（无附件），可删除，避免混淆。
