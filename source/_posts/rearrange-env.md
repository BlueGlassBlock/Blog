---
title: 整理自己的工作环境
date: 2023-01-20 11:33:27
tags: [python, conda, venv, pdm, wsl]
---

最近又受蛊惑，所以想把开发机上的工作环境整理下。

<!-- more -->

# Python

之前我是直接装在 Windows 上的，现在全部删了换成 [`miniconda`](https://docs.conda.io/en/latest/miniconda.html)

```text
$ py -0
 -V:3.11 *        Python 3.11 (64-bit)
 -V:3.10          Python 3.10 (64-bit)
 -V:3.9           Python 3.9 (64-bit)
 -V:3.8           Python 3.8 (64-bit)
```

```bash
winget uninstall Python.Python.3.8
winget uninstall Python.Python.3.9
winget uninstall Python.Python.3.10
winget uninstall Python.Python.3.11
```

接着把 `pipx` 的环境删了，再删除所有的 `.venv`。

因为之前 `PDM` 修了 `install.cache` 的 bug，所以重装后所有的环境都启用中心缓存。

安装 `miniconda`。

```text
$ py -0p
 -V:3.10 *        C:\ProgramData\miniconda3\python.exe
```

换 `conda` 的源为[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)。

用管理员 shell 运行 `conda init`.

用 `conda` 自带的 Python 安装 `pipx` 等工具，再用其安装 `PDM` `pre-commit` `maturin` 等工具。

最后，进入项目后切换到对应 Python 版本直接 `pdm sync` 即可。

# 终端

之前我使用的是 [`Oh My Posh`](https://ohmyposh.dev/) 的 slim 主题。

现在更换到用 Rust 编写的 [`Starship`](https://starship.rs/zh-cn/)。

```bash
winget install --id Starship.Starship
```

```toml
"$schema" = 'https://starship.rs/config-schema.json'

format = """
$time\
$git_branch\
$git_status\
$fill\
$all\
$character\
"""

[time]
format = '\[[$time]($style)\]'
disabled = false

[git_status]
format = '([\[$all_status:$ahead_behind\]]($style))'
conflicted = "" # nf-fa-warning
ahead = "⇡"
behind = "⇣"
diverged = "⇕"
up_to_date = "" # nf-oct-sync
untracked = "?"
stashed = "" # nf-fa-bookmark
modified = "" # nf-fa-pencil
staged = "" # nf-fa-check_square_o 
renamed = "" # nf-fa-tags
deleted = "" # nf-fa-trash

[character]
success_symbol = "[➜](bold green)"
error_symbol = "[➜](bold red)"

[directory]
truncation_symbol = "../"
use_os_path_sep = false
truncation_length = 2
truncate_to_repo = false
```

# WSL

鉴于 WSL 我不怎么用，所以先删了。