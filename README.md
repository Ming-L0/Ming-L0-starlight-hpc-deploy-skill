# Starlight HPC Deploy Skill

A Trae IDE skill for deploying and running code on **NSCC-GZ Starlight HPC** (星光超算) via SCP + SSH — no SLURM or sbatch needed.

## How it works

```
① Create job on starlight.nscc-gz.cn → get a GPU node
② Local PowerShell: mkdir via SSH + scp upload
③ Local PowerShell: SSH login
④ Remote bash step-by-step: cd → activate venv → run your script
```

## Installation

1. Copy `SKILL.md` into your project's `.trae/skills/starlight-hpc-deploy-skill/` directory.
2. Restart Trae IDE (or reload skills).
3. Edit the connection info table in `SKILL.md` with your own credentials.

## Configuration

Edit the connection info table in `SKILL.md`:

| Field | What to set |
|-------|-------------|
| SSH Host | `proxy.nscc-gz.cn` |
| SSH Port | `8022` |
| SSH User | Your NSCC-GZ username |
| Remote Base Path | Your HDD pool path on Starlight |
| Python Env | Path to your venv |
| Proxy Script | Path to proxy setup script (optional) |

## Usage

Open any project in Trae IDE, type your deployment request (e.g., "deploy to HPC"), and the skill will guide you step-by-step:

1. Confirm files, remote directory, and command
2. Create remote directory
3. SCP upload
4. SSH login
5. Run your script

## Requirements

- NSCC-GZ Starlight account
- SSH access to `proxy.nscc-gz.cn:8022`
- Trae IDE with skill support
