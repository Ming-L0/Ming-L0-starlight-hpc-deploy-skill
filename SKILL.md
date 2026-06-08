---
name: "starlight-hpc-deploy-skill"
description: "Deploy and run any code on NSCC-GZ Starlight HPC via SCP + SSH. Invoke when user wants to run code on 星光超算/Starlight/NSCC-GZ, or mentions 超算部署/HPC training."
---

# Starlight HPC Deploy

将任意代码部署到 NSCC-GZ 星光超算并运行。

---

## 整体流程

```
① starlight.nscc-gz.cn 创建作业 → 分配 GPU 计算节点
② 本地 PowerShell: ssh 建目录 + scp 上传
③ 本地 PowerShell: ssh 登录
④ 远程 bash: 逐步输命令 → cd → source venv → python xxx
```

> 不需要 sbatch / SLURM / Web E-Shell。节点已由平台分配好，SSH 进去直接跑。

---

## 连接信息（沿用上次值，用户不提不改变）

| 项目 | 值 |
|------|-----|
| SSH Host | `proxy.nscc-gz.cn` |
| SSH Port | `8022` |
| SSH User | `cityu_pengzhang_1` |
| 远程项目根目录 | `/XYAIFS00/HDD_POOL/cityu_pengzhang/cityu_pengzhang_1/AI4Medicine/eeg_project` |
| 远程代码子目录 | **每次部署时向用户确认**（默认用当前本地文件夹名） |
| Python 环境 | `source ../venv/bin/activate`（相对于子目录） |
| 代理 | `source /app/bin/proxy.sh` |

> 远程子目录规则：默认取当前项目文件夹名。例如在 `SpikeNet2` 目录下就默认传 `SpikeNet2/`，在 `starlight_hpc` 下就默认传 `starlight_hpc/`。用户可随时覆盖。

### 关键规则

| 规则 | 说明 |
|------|------|
| **SCP 只能在本地 PowerShell 跑** | 计算节点没有外网 DNS，SSH 进去后跑不了 scp |
| **SSH 进去后逐条给命令** | 每次只给一条，等用户说"ok"再给下一条 |
| **`mkdir -p logs` 按需执行** | 如果运行命令需要写日志到 logs/ 目录就创建，否则跳过 |

---

## 执行步骤（逐条给命令）

> **每个阶段给一条命令 + 告诉用户下一步是什么。等待用户说"ok"后再给下一条。**

### 阶段一：确认参数

向用户确认以下信息：

1. **要上传哪些文件**：列出当前目录下需要传到超算的文件（先 `ls` 或 `glob` 看一眼当前目录再推荐）
2. **远程子目录**：默认取当前文件夹名，用户可改
3. **要在超算上执行的命令**：比如 `python train.py --epochs 50`、`bash run.sh` 等
4. **是否需要后台运行**（默认 nohup 后台）

> 不要假定用户跑什么脚本，一切以用户说的为准。

### 阶段二：创建远程目录

如果远程子目录不存在，先在本地 PowerShell 用 SSH 创建：

```powershell
ssh <SSH_User>@<SSH_Host> -p <SSH_Port> "mkdir -p <Remote_Base>/<Remote_Subdir>"
```

> 提醒：**"目录已存在则可跳过。下一步 SCP 上传文件。"**

### 阶段三：上传（本地 PowerShell，1 条命令）

```powershell
scp -P <SSH_Port> -o StrictHostKeyChecking=no <file1> <file2> <SSH_User>@<SSH_Host>:"<Remote_Base>/<Remote_Subdir>/"
```

> 提醒：**"输密码上传。完成后下一步是 SSH 登录。"**

### 阶段四：SSH 登录（本地 PowerShell，1 条命令）

```powershell
ssh <SSH_User>@<SSH_Host> -p <SSH_Port>
```

> 提醒：**"输密码登录。登录后我会逐步给你 bash 命令。"**

### 阶段五：远程逐条执行（按顺序，一条一条给）

**① 进入项目目录**
```bash
cd <Remote_Base>/<Remote_Subdir>
```
> 提醒：**"进入项目目录。下一步激活 Python 环境。"**

**② 激活环境**
```bash
source /app/bin/proxy.sh 2>/dev/null || true && source ../venv/bin/activate
```
> 提醒：**"看到 (venv) 就是成功了。下一步执行用户命令。"**

**③ 创建日志目录（如需要）**
```bash
mkdir -p logs
```
> 仅在用户命令涉及日志输出时执行。

**④ 执行用户命令**

根据阶段一确认的命令，拼出完整 shell 命令。后台运行用 nohup：

```bash
nohup <user_command> > logs/<run_name>.out 2>&1 &
```

前台运行就直接：

```bash
<user_command>
```

> 提醒：**"命令已启动。下一步查看实时输出。"**

**⑤ 查看实时输出**
```bash
tail -f logs/<run_name>.out
```
> 提醒：**"Ctrl+C 退出 tail，后台任务继续跑。"**

### 阶段六：后续监控（用户需要时）

```bash
nvidia-smi              # GPU 状态
tail -f logs/xxx.out    # 实时输出
ps aux | grep python    # 进程
```

---

## 执行指南总结

1. **确认**：要传哪些文件 + 要跑什么命令 + 远程子目录（默认 = 当前文件夹名）
2. **mkdir**：SSH 创建远程目录
3. **SCP**：上传文件
4. **SSH**：登录
5. **逐条**：cd → source venv → （mkdir logs）→ nohup xxx & → tail -f
6. **SCP 绝不在 SSH 里跑**：计算节点无外网 DNS
