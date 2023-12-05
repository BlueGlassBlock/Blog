---
title: 街机音游 HDD 研究记录
date: 2023-11-19 18:41:15
tags: [research, hdd, arcade, rhythm-game, maimai, chunithm, wacca]
---

部分街机音游部署研究记录。

<!-- more -->

{% note warning %}
如果本文内容侵犯到您的权益，请通过 About 页的邮箱与我取得联系以并删除侵权内容。
{% endnote %}

{% note primary %}
本文不提供任何“官方”文件下载渠道及链接，实际上这些文件在某种程度上都可直接通过搜索引擎查找到。

本文仅作踩坑记录，大部分操作还请自行摸索。
{% endnote %}

# ALLS 系服务器 - ARTEMiS 配置

## 原料

- ARTEMiS 源码及其 Python 虚拟环境
- MySQL 兼容的数据库

### 附：数据库配置

我用的是 MariaDB，MySQL 也是一样的。

#### 便携式配置

如果用安装包装了那就可以跳过。

```PowerShell
$ dir

    Directory: D:\mariadb

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----          2023/11/20    18:49                bin
d----          2023/11/20    18:49                include
d----          2023/11/20    18:49                lib
d----          2023/11/20    18:49                man
d----          2023/11/20    18:49                share
-a---           2023/8/17    23:25          17987 COPYING
-a---           2023/8/17    23:25           2104 CREDITS
-a---           2023/8/17    23:25           2721 README.md
-a---           2023/8/17    23:25          85250 THIRDPARTY
$ .\bin\mariadb-install-db.exe -p="ARTEMiS" # 用 ARTEMiS 作为 root 密码
Default data directory is D:\mariadb\data
Running bootstrap
Creating my.ini file
Removing default user
Setting root password
Creation of the database was successful
$ .\bin\mariadbd.exe --console # 阻塞式启动 MariaDB
2023-11-20 18:51:40 0 [Note] Starting MariaDB 11.1.2-MariaDB source revision 9bc25d98209df6810f7a7d5e7dd3ae677a313ab5 as process 22764
2023-11-20 18:51:40 0 [Note] InnoDB: Compressed tables use zlib 1.2.13
2023-11-20 18:51:40 0 [Note] InnoDB: Number of transaction pools: 1
2023-11-20 18:51:40 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2023-11-20 18:51:40 0 [Note] InnoDB: Initializing buffer pool, total size = 128.000MiB, chunk size = 2.000MiB
2023-11-20 18:51:40 0 [Note] InnoDB: Completed initialization of buffer pool
......
```

#### 初始化数据库

此处按照 ARTEMiS 文档来。

##### 访问数据库

`.\bin\mariadb.exe -u root -p="ARTEMiS"`

### 建表

```sql
CREATE USER 'aime'@'localhost' IDENTIFIED BY 'ARTEMiS';
CREATE DATABASE aime;
GRANT Alter,Create,Delete,Drop,Index,Insert,References,Select,Update ON aime.* TO 'aime'@'localhost';
FLUSH PRIVILEGES;
exit;
```

{% note warning %}
假定从这里开始你的 Python 环境已经搭建好并安装了 `requirements.txt` 内的依赖（直接安装和使用虚拟环境隔离都可以）。
{% endnote %}

将 `example_config` 文件夹复制为 `config` 并修改 `config/core.yaml`。

请将 `"HOSTNAME"` 自行替换为从 `ipconfig` 获取的 IP 地址，不可使用 `localhost`。

各个端口请不要随意更改（除非你用反代等手段），否则会导致游戏无法正常连接。

```yaml
server:
  listen_address: "HOSTNAME"
  allow_user_registration: True
  allow_unregistered_serials: True
  name: "ARTEMiS"
  is_develop: True
  threading: False # 使用多线程，但是会导致 Ctrl+C 无法正常退出
  log_dir: "logs"

title:
  loglevel: "info"
  hostname: "HOSTNAME"
  port: 8080

database:
  host: "localhost" # 此处用于连接数据库，所以允许使用 localhost
  username: "aime"
  password: "ARTEMiS" # 与上面的密码一致
  name: "aime"
  port: 3306
  protocol: "mysql"
  sha2_password: False
  loglevel: "info"
  user_table_autoincrement_start: 10000 # 用户 ID 起始值
  memcached_host: "localhost" # Linux 下使用

frontend:
  enable: False # 没啥用的前端，可以关闭
  port: 8090
  loglevel: "info"

allnet:
  loglevel: "info"
  port: 80
  allow_online_updates: False
  update_cfg_folder: ""

billing:
  port: 8443
  ssl_key: "cert/server.key"
  ssl_cert: "cert/server.pem"
  signing_key: "cert/billing.key"

aimedb:
  loglevel: "info"
  port: 22345
  key: "AIMEDB_KEY"

mucha:
  enable: False
  hostname: "localhost"
  loglevel: "info"
```

`AIMEDB_KEY` 为 `Copyright` + `(C)` + aime 卡发行公司的大写，共 16 个字符。

到此你应该可以正常通过 `python index.py` 启动服务器了，但是你仍需使用 `python read.py` 从你想要的游戏中读取数据。

### 读取游戏数据

读取请参考 ARTEMiS 的文档，这里给出部分游戏的示例。

```PowerShell
python read.py --series SDDT --version 7 --binfolder K:\Arcade\SDDT\App --optfolder K:\Arcade\SDDT\Option # O.N.G.E.K.I. bright MEMORY
python read.py --series SDHD --version 13 --binfolder K:\Arcade\SDBT\App --optfolder K:\Arcade\SDBT\Option # CHUNITHM SUN
python read.py --series SDEZ --version 19 --binfolder K:\Arcade\SDEZ\App\Sinmai_Data\StreamingAssets --optfolder K:\Arcade\SDEZ\Option # maimai DX Festival
python read.py --series SDFE --version 4 --binfolder K:\Arcade\SDFE\App\WindowsNoEditor\Mercury\Content # WACCA Reverse，没有 Option
```

### 进行游戏

使用虚拟数据卡进行游戏请设置 `segatools.ini` 中的 `aime.aimePath` 指向一个 20 位数字的文本文件（最好不要用记事本），并设置 `aime.felicaGen` 为 `0`。

使用实体卡请参考其他网络教程。

需要联网游戏请设置 `segatools.ini` 中的 `dns.default` 为服务器地址（上文 `HOSTNAME` 对应的 IP 地址）。

## 各个游戏踩坑记录

# 音击 / O.N.G.E.K.I.

`segatools` 安装后游戏下方仍会显示 `Credit: 0`，但是其实可以直接开始游戏（GP 是免费购买的）。

# 中二节奏 / CHUNITHM

分辨率锁定 1920x1080 无法修改。

# 华卡音舞 / WACCA

分辨率修改：修改 `WindowsNoEditor/Mercury/Config/DefaultGameUserSettings.ini` 内 `resolution` 相关项。 

启动卡 `4105-0` 错误代码：修改 `WindowsNoEditor/Mercury/Config/DefaultHardware.ini`

```ini
[/Script/Mercury.StartupSettings]
bEnableErrorWatcher=false
```

联网后无法进入游戏：修改 `WindowsNoEditor/Mercury/Config/DefaultHardware.ini`

```ini
[/Script/Mercury.StartupSettings]
IgnoreLogin=true
```

进入 test 页再退出也有可能解决问题。

触摸支持：请看我的[这个项目](https://github.com/BlueGlassBlock/toucca)。