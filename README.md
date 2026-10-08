# git-publish

> TRAE Skill —— 自动将项目/Skill 推送到 GitHub，包括 README 自动生成、代理探测、Token 认证、安全清理。

## 这是什么

这是一个 [TRAE](https://www.trae.cn/) Skill，用于将当前工作区的项目或 Skill 一键推送到 GitHub。它解决了以下痛点：

- **GitHub CLI 安装困难**（沙箱限制、交互式登录不友好）
- **中国大陆网络问题**（github.com 443 被墙，需要代理）
- **git config --global 无权限**（沙箱无法写 .gitconfig）
- **推送后 README 缺失**（仓库第一眼空荡荡）
- **Token 安全**（推送后明文 token 残留）

## 核心能力

| 能力 | 说明 |
|------|------|
| **API 创建仓库** | 用 GitHub REST API + Personal Access Token 创建仓库，不依赖 gh CLI |
| **自动代理探测** | 扫描本地常见代理端口（7890/7897/1080 等），自动设置环境变量 |
| **README 自动生成** | 推送前检查 README，缺失或不完整时按规范自动生成 |
| **LICENSE + .gitignore** | 自动生成 MIT LICENSE 和合适的 .gitignore |
| **Token 安全清理** | 推送后替换 remote URL 中的明文 token，并提醒用户撤销 |
| **全流程自动化** | 从创建仓库到推送完成，用户只需提供一个 Token |

## 怎么用

### 安装

将 `.trae/skills/git-publish/` 目录复制到你的 TRAE 工作区的 `.trae/skills/` 目录下。

### 触发

当你说以下内容时，Skill 自动激活：

- "把 skill 推到 GitHub 上"
- "提交到 GitHub"
- "上传到 GitHub"
- "发布到 GitHub"

### 使用流程

1. AI 确认要推送的目录、仓库名、描述
2. AI 检查并补全 README / LICENSE / .gitignore
3. 你按提示生成一个 GitHub Token（勾选 repo 权限）
4. AI 用 API 创建仓库 → git init → commit → push
5. 如遇网络问题，AI 自动探测代理并重试
6. 推送完成，AI 返回仓库地址并提醒你撤销 Token

## 完整流程图

```
用户："推到GitHub"
        │
        ▼
  确认目录/仓库名/描述
        │
        ▼
  检查 README/LICENSE/.gitignore
  (缺失则自动生成)
        │
        ▼
  用户提供 GitHub Token
        │
        ▼
  API 创建仓库 (Invoke-RestMethod)
        │
        ▼
  git init + remote + add + commit
        │
        ▼
  git push
        │
   ┌────┴────┐
   │ 成功？   │
   ├─是───────┤ → 清理 remote URL token → 验证 → 提醒撤销 token → 完成
   │         │
   └─否───────┘ → 探测代理端口 → 设置 $env:HTTPS_PROXY → 重试 push
```

## 示例

> **用户**：把这个 skill 推到 github 上

**AI**：
1. 我确认一下：要推送的目录是 `.trae/skills/your-skill`，仓库名叫 `your-skill`，描述是"xxx"，公开仓库，对吗？
2. 我检查了目录，README.md 已存在，LICENSE 和 .gitignore 缺失，我来补上...
3. 请去 https://github.com/settings/tokens/new 生成 token（勾选 repo），发给我。
4. 正在创建仓库... 仓库已创建：https://github.com/username/your-skill
5. 正在推送... 检测到网络问题，探测到代理 127.0.0.1:7897，重试中...
6. 推送成功！仓库地址：https://github.com/username/your-skill
7. 请记得去 https://github.com/settings/tokens 撤销你的临时 token。

## README 生成规范

Skill 推送时如果 README 缺失，会自动生成包含以下部分的 README：

1. **标题 + 一句话简介**
2. **这是什么**（背景、目的、核心价值）
3. **核心内容**（表格展示组件/功能）
4. **怎么用**（安装方式、触发条件）
5. **示例**（输入→输出）
6. **文件结构**
7. **License**

## 文件结构

```
git-publish/
├── .trae/
│   └── skills/
│       └── git-publish/
│           └── SKILL.md      # Skill 主体
├── README.md                  # 本文件
└── LICENSE                    # MIT
```

## 技术细节

### 为什么用 API 而不是 gh CLI？

- gh CLI 需要安装（MSI 安装包在沙箱中可能失败）
- gh CLI 的 `auth login --web` 是交互式命令，AI 无法操作
- GitHub REST API 只需要一个 token，用 `Invoke-RestMethod` 即可调用

### 为什么用环境变量而不是 git config？

- `git config --global` 需要写 `C:\Users\xxx\.gitconfig`，TRAE 沙箱可能无权限
- 环境变量 `$env:HTTP_PROXY` / `$env:HTTPS_PROXY` 只影响当前会话，更安全

### 代理探测原理

扫描常见代理软件的本地端口：

| 端口 | 代理软件 |
|------|---------|
| 7890 | Clash |
| 7897 | Clash Verge |
| 1080 | SOCKS5 通用 |
| 10808/10809 | V2Ray |
| 8118 | Privoxy |
| 8080 | HTTP 通用 |

## License

MIT
