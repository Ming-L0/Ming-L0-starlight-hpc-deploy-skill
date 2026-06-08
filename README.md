# Trae IDE Skills

A collection of Trae IDE skills for development workflows.

## Skills

### 1. Starlight HPC Deploy

Deploy and run code on **NSCC-GZ Starlight HPC** (星光超算) via SCP + SSH — no SLURM or sbatch needed.

> See `.trae/skills/starlight-hpc-deploy-skill/SKILL.md`

### 2. GitHub Upload

Upload local project files to GitHub. Handles `git init`, commit, and push.

> See `.trae/skills/github-upload-skill/SKILL.md`

## Installation

Copy the entire `.trae` folder into your project root directory, then restart Trae IDE.

Structure:
```
your-project/
├── .trae/
│   └── skills/
│       ├── starlight-hpc-deploy-skill/
│       │   └── SKILL.md
│       └── github-upload-skill/
│           └── SKILL.md
└── ... (your code)
```

## Configuration

Each skill has its own configuration section in its `SKILL.md`. Edit with your credentials before use.
