---
title: 整理自己的工作环境
date: 2023-01-20 11:33:27
tags: [install, python, conda, venv, pdm, wsl, windows, pipx]
---

开发机工作环境整理/安装记录。

<!-- more -->

# 2023-01

## Python

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

## 终端

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

## WSL

鉴于 WSL 我不怎么用，所以先删了。

# 2023-05


## 抛弃 miniconda

`miniconda` 虽然许多人交口称赞，但我还是觉得它太臃肿了。

其实臃肿倒还是其次，最大的问题是：

他自己是用 Python 写的。

这就导致了它一安装就会带有一个默认的 `base` 环境，而且这个环境是不可删除的。

然后各种工具就开始炸裂了：

PyO3 不能自动识别当前环境，因为存在 `CONDA_PREFIX`。

PDM 默认的策略是直接复用激活的环境，所以某位 Discord 用户就不小心把全局环境里的一堆包都删了。

因此我决定回到 <https://www.python.org/downloads/windows/> 上下载的 Python。

## 重建环境

感谢 `pipx`，我之前所有的全局 Python 工具都被他管理了。

重装 Python 后，利用内置的 `venv` 模块，我们可以轻松地升级那些工具的独立环境：

{% note primary %}
更正，其实可以直接 `pipx reinstall-all`
{% endnote %}

> ```powershell
> python -m venv --upgrade $env:USERPROFILE\.local\pipx\shared
> 
> foreach ($env_path in (Get-ChildItem -Path $env:USERPROFILE\.local\pipx\venvs)) {
>     python -m venv --upgrade $env_path 
> }
> ```
> 
> 可以继续使用原来装好的工具了。

## PIPX

等等等等，我之前是将 `pipx` 本体装在全局环境里的，现在怎么办？

其实 `pipx` 可以用来自己管理自己哦，[`pipx-in-pipx`](https://pypi.org/project/pipx-in-pipx/) 就是干这个的。

但是它对我不适用，所以我们可以尝试另一种方式：[`zipapp`](https://docs.python.org/zh-cn/3/library/zipapp.html)。

```powershell
curl -L https://github.com/pypa/pipx/releases/latest/download/pipx.pyz -o pipx.pyz
python pipx.pyz install pipx
Remove-Item pipx.pyz
```

来试试吧：

```powershell
pipx --help
```

```text
usage: pipx [-h] [--version]
            {install,uninject,inject,upgrade,upgrade-all,uninstall,uninstall-all,reinstall,reinstall-all,list,run,runpip,ensurepath,environment,completions}
            ...

Install and execute apps from Python packages.

Binaries can either be installed globally into isolated Virtual Environments
or run directly in a temporary Virtual Environment.

Virtual Environment location is C:\Users\blueg\.local\pipx\venvs.
Symlinks to apps are placed in C:\Users\blueg\.local\bin.

optional environment variables:
  PIPX_HOME             Overrides default pipx location. Virtual Environments will be installed to $PIPX_HOME/venvs.
  PIPX_BIN_DIR          Overrides location of app installations. Apps are symlinked or copied here.
  PIPX_DEFAULT_PYTHON   Overrides default python used for commands.
  USE_EMOJI             Overrides emoji behavior. Default value varies based on platform.

options:
  -h, --help            show this help message and exit
  --version             Print version and exit

subcommands:
  Get help for commands with pipx COMMAND --help

  {install,uninject,inject,upgrade,upgrade-all,uninstall,uninstall-all,reinstall,reinstall-all,list,run,runpip,ensurepath,environment,completions}
    install             Install a package
    uninject            Uninstall injected packages from an existing Virtual Environment
    inject              Install packages into an existing Virtual Environment
    upgrade             Upgrade a package
    upgrade-all         Upgrade all packages. Runs `pip install -U <pkgname>` for each package.
    uninstall           Uninstall a package
    uninstall-all       Uninstall all packages
    reinstall           Reinstall a package
    reinstall-all       Reinstall all packages
    list                List installed packages
    run                 Download the latest version of a package to a temporary virtual environment, then run an app
                        from it. Also compatible with local `__pypackages__` directory (experimental).
    runpip              Run pip in an existing pipx-managed Virtual Environment
    ensurepath          Ensure directories necessary for pipx operation are in your PATH environment variable.
    environment         Print a list of variables used in pipx.constants.
    completions         Print instructions on enabling shell completions for pipx
```

~~除了 `pipx list` 的 Python 版本显示不正常以外，其他都正常。~~

> 以及什么时候 pipx 才能用 [`ofek/pyapp`](https://github.com/ofek/pyapp) 啊
