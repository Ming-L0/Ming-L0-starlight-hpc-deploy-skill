---
name: "github-upload-skill"
description: "Upload local project files to GitHub. Handles git init, commit, and push. Invoke when user wants to upload code to GitHub, push to remote, or share project on GitHub."
---

# GitHub Upload

将本地项目文件上传到 GitHub。

---

## 整体流程

```
① 检查是否 Git 仓库 → 否 → git init
② 配置 git user.name / user.email（如未配置）
③ 创建 .gitignore（按需排除）
④ 确认要上传的文件
⑤ git add → git commit
⑥ GitHub 创建仓库 → git remote add → git push
```

---

## 执行步骤

### 阶段一：检查 Git 环境

```bash
git status
```

> **如果报 `not a git repository`**：执行 `git init`
> **如果报 `Author identity unknown`**：向用户询问 GitHub 用户名和邮箱，执行：
> ```bash
> git config user.name "<username>"
> git config user.email "<email>"
> ```

### 阶段二：确认上传文件

列出当前目录文件（排除 `.trae/`、`__pycache__/`、`*.pyc` 等），向用户确认：

1. **要上传哪些文件/目录**
2. **仓库名称**（默认用当前文件夹名）
3. **是否需要 .gitignore**（默认生成，排除 `.trae/` `__pycache__/` `*.pyc` `*.log`）

### 阶段三：创建 .gitignore

如需要，创建包含以下内容的 `.gitignore`：
```
.trae/
__pycache__/
*.pyc
*.log
```

可根据项目语言追加（如 `node_modules/`、`*.pyc`、`*.tar` 等）。

### 阶段四：提交

```bash
git add <file1> <file2> ...
git status
```

确认无误后：

```bash
git commit -m "<commit message>"
```

### 阶段五：推送到 GitHub

1. **请用户在浏览器中创建 GitHub 仓库**：
   - 打开 https://github.com/new
   - Repository name 填确认的名称
   - **不要**勾选 "Add a README file"
   - 点击 Create repository

2. **等用户确认后，获取实际仓库名**（可能 GitHub 自动加前缀，如 `Ming-L0-starlight-hpc-deploy-skill`）。建议用 `WebFetch` 访问 `https://github.com/<username>?tab=repositories` 确认。

3. **设置远程并推送**：

先用 SSH 方式测试认证：
```bash
ssh -T git@github.com
```

如果成功（显示 `Hi <username>!`），用 SSH 推送：
```bash
git remote add origin git@github.com:<username>/<repo-name>.git
git branch -M main
git push -u origin main
```

如果 SSH 失败，用 HTTPS：
```bash
git remote add origin https://github.com/<username>/<repo-name>.git
git branch -M main
git push -u origin main
```

> **PowerShell 注意**：`&&` 在 PowerShell 中不是有效分隔符，改用 `;`。

---

## 关键规则

| 规则 | 说明 |
|------|------|
| **不主动提交** | 只在用户明确说"上传"时才 commit + push |
| **确认远程仓库存在** | 用 WebFetch 检查 GitHub 确认仓库名 |
| **不传敏感文件** | `.env`、`credentials.json` 等不要上传 |
| **SSH 优先** | 先试 SSH，失败再切 HTTPS |
