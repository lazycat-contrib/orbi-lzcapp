# Orbi - LazyCat App

[Orbi](https://github.com/orbi-build/orbi)（熄灯软件工厂：给 GitHub Issue 打 `ai-ready` 标签，自动交付经过独立评审、合并的 PR 与打 tag 的 Release）的懒猫微服（LazyCat）打包。

## 模式

- **镜像模式（mirror）**：`ghcr.io/orbi-build/orbi` 镜像经 `ghcr.nju.edu.cn` 加速拉取，`require_digest_match: true` 校验镜像内容与上游一致。
- **自动跟随上游**：每日检查上游 semver tag，发现新版本自动构建发布。
- **喵喵商店发布**：仅私有商店（MiaoMiao private store），不发布官方商店。

## 架构

Orbi 无自带 Web UI，本包提供一个静态引导页（应用入口）：快速开始、状态查看命令、标签含义。

| 服务 | 说明 |
| --- | --- |
| orbi | runner 主服务（systemd 作为 PID 1，单容器） |
| guide | nginx 静态引导页（包内 content 目录） |

## 设置向导参数

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| GitHub Token | ✅ | 对任务池仓库有写权限（classic 带 `repo` scope，或 fine-grained 勾选 Issues/PR/Contents 读写） |
| 任务池仓库 | ✅ | `OWNER/REPO` 形式 |
| 基准分支 | 可选 | 默认 `main` |
| Pi Provider / Model / Base URL / API Key | 可选 | 模型配置，**四项同时填写或同时留空**（部分填写会启动失败） |
| Pi API 协议 / 上下文窗口 / 最大输出 | 可选 | 默认 `openai-completions` / `128000` / `16384` |

## 说明

- **数据卷**：`/lzcapp/var/orbi/deploy`（deploy home：orbi checkout、orbi.toml、凭据）、`/lzcapp/var/orbi/work`（任务池克隆与 worktree）
- **systemd 运行需求**：挂载宿主 `/sys/fs/cgroup`（读写）+ `tmpfs /run /tmp`（已在 manifest 配置）；需要运行时支持 host cgroup namespace（上游要求 `--cgroupns=host`，lzc-manifest 暂无对应字段，取决于平台默认）
- GitHub Token 每次启动由注入变量重写进 `/orbi/.orbi/env`（权限 600）；修改参数重新部署即可生效
- 首次启动 entrypoint 幂等 setup：克隆 orbi 自身、生成 orbi.toml / pi-providers.json（已存在的文件绝不覆盖）
- 使用方式见应用入口引导页或[官方文档](https://docs.orbi.build/zh/)

## 所需 Secrets

| Secret | 说明 |
| --- | --- |
| `APPSTORE_URL` / `APPSTORE_TOKEN` | 喵喵商店 API 地址与发布令牌 |
| `APP_ID` | 可选 |
| `PRIVATE_STORE_GROUP_CODES` | 可选，私有分组码 |

## 许可

AGPL-3.0（上游）
