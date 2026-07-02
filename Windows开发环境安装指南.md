# Windows 开发环境安装指南

> 适用场景：重装系统后的个人开发机，三盘结构（C 系统 / D 应用 / E 开发）  
> 环境变量策略：**全部使用用户变量 + 用户 Path，不修改系统变量**

---

## 目录

1. [磁盘规划](#一磁盘规划)
2. [目录初始化](#二目录初始化)
3. [安装顺序总览](#三安装顺序总览)
4. [Git](#四git)
5. [JDK 21 + JDK 17](#五jdk-21--jdk-17)
6. [Maven](#六maven)
7. [IntelliJ IDEA](#七intellij-idea)
8. [Node.js（nvm-windows）](#八nodejsnvm-windows)
9. [npm 与 pnpm](#九npm-与-pnpm)
10. [7-Zip 与 Everything](#十常用小工具7-zip--everything)
11. [MySQL 8.0](#十一mysql-80)
12. [Redis](#十二redis)
13. [Docker Desktop](#十三docker-desktop)
14. [Navicat](#十四navicat)
15. [Android Studio](#十五android-studio)
16. [Flutter](#十六flutter)
17. [环境变量汇总](#十七环境变量汇总)
18. [IDEA 配置清单](#十八idea-配置清单)
19. [常见问题](#十九常见问题)
20. [Cursor](#二十cursor)
21. [Claude Code 与 Codex](#二十一claude-code-与-codex)
22. [CC Switch](#二十二cc-switch)
23. [本地 AI 模型（Ollama）](#二十三本地-ai-模型ollama)
24. [后续按需安装](#二十四后续按需安装)

---

## 一、磁盘规划

| 盘符 | 角色 | 原则 |
|------|------|------|
| **C:** | 系统盘 | 仅 Windows + 驱动 + 少量必须装 C 盘的软件 |
| **D:** | 应用盘 | 正式安装软件、绿色工具、安装包归档 |
| **E:** | 工作盘 | 源码、SDK、运行时、数据、缓存、备份 |

### D 盘结构

```
D:\
├── Apps\              # 正式安装（IDEA、Android Studio、Navicat、7-Zip 等）
│   ├── JetBrains\     # IDEA、Android Studio
│   ├── Database\      # Navicat
│   ├── Communication\ # 微信、QQ、钉钉等
│   └── Utilities\     # 7-Zip、Everything
├── Portable\          # 绿色/便携工具（Git 等）
├── Packages\          # 安装包归档
│   └── 2026\
│       ├── Dev\
│       ├── OS\
│       ├── DB\
│       └── Tools\
└── Drivers\           # 驱动备份（可选）
```

### E 盘结构

```
E:\
├── Workspace\         # 项目源码
│   ├── Company\
│   ├── Personal\
│   ├── OpenSource\
│   ├── Sandbox\
│   └── _Templates\
├── Envs\              # 语言/构建工具（Java、Maven、Node 等）
├── SDK\               # 平台 SDK（Android SDK、Flutter 等）
│   ├── Android\       # sdk、ndk、home（AVD）
│   └── Flutter\       # flutter SDK
├── Services\          # 本地服务（MySQL、Redis 等）
├── Data\              # 持久化数据（MySQL、Redis、微信/QQ 聊天记录等）
│   ├── wechat\        # 微信文件管理目录
│   └── qq\            # QQ 文件/缓存目录
├── Cache\             # 构建与包管理缓存（Gradle、pub、AndroidStudio、Cursor 等）
│   └── Cursor\        # Roaming / Local（Junction 目标目录）
├── Containers\        # Docker / WSL
├── Tools\             # 开发 CLI
├── Config\            # 配置文件
├── Secrets\           # 密钥（注意权限）
├── Logs\              # 日志
├── AI\                # 模型与推理（可选）
├── Backup\            # 备份
└── Temp\              # 临时文件
```

### 核心原则

1. C 盘只放系统
2. D 盘放软件和工具
3. E 盘放开发、数据、项目
4. SDK / Cache / Data 必须独立
5. 所有缓存迁移出 C 盘
6. **环境变量一律用用户变量，不动系统变量**

---

## 二、目录初始化

### 方式 A：直接拷贝目录树

将 `copy-to-disks\` 中的内容复制到 Windows：

- `copy-to-disks\D\` 里的内容 → `D:\`
- `copy-to-disks\E\` 里的内容 → `E:\`

> 注意：复制「里面的内容」，不要变成 `D:\D\Apps` 多一层。

空目录中的 `.gitkeep` 拷贝后可删除。

### 方式 B：PowerShell 脚本

```powershell
powershell -ExecutionPolicy Bypass -File .\bootstrap.ps1
# 仅创建目录，不设置环境变量
```

附带配置模板：

| 文件 | 用途 |
|------|------|
| `E:\Config\maven\settings.xml` | Maven 本地仓库 |
| `E:\Config\ide\jetbrains\idea.properties.example` | IDEA 路径重定向 |
| `E:\Config\git\.gitconfig.example` | Git 全局配置模板 |

---

## 三、安装顺序总览

```
① 目录结构（D/E 盘）
② Git
③ JDK 21 + JDK 17
④ Maven
⑤ IntelliJ IDEA
⑥ Node.js（nvm-windows）+ npm + pnpm
⑦ 7-Zip + Everything
⑧ MySQL 8.0
⑨ Redis
⑩ Docker Desktop（需先装 WSL2）
⑪ Navicat
⑫ Android Studio（Flutter / Android 开发前置）
⑬ Flutter
⑭ Cursor（AI IDE）
⑮ Claude Code + Codex（AI 终端 CLI）
⑯ CC Switch（API 供应商切换）
⑰ Ollama（本地大模型，可选）
⑱ 按需：Postman / Apifox ...
```

---

## 四、Git

### 下载

https://git-scm.com/download/win

安装包保存到：`D:\Packages\2026\Dev\`

### 安装路径

```
D:\Portable\VCS\Git
```

### 安装选项建议

| 选项 | 建议 |
|------|------|
| PATH | **Git from the command line and also from 3rd-party software** |
| HTTPS | Use the OpenSSL library |
| 换行 | Checkout Windows-style, commit Unix-style |
| 终端 | Use Windows' default console window |

### Git 配置文件

1. 将 `E:\Config\git\.gitconfig.example` 重命名为 `.gitconfig`
2. 填写 `user.name` 和 `user.email`

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `GIT_CONFIG_GLOBAL` | `E:\Config\git\.gitconfig` |
| `TEMP` | `E:\Temp` |
| `TMP` | `E:\Temp` |

> **注意**：`TEMP` / `TMP` 值前后不能有空格或隐藏换行符，否则 IDEA 会报警。

### 用户 Path 追加

```
D:\Portable\VCS\Git\cmd
```

### 验证

```powershell
git --version
git config --global user.name
echo "[$env:TEMP]"
where.exe git
```

---

## 五、JDK 21 + JDK 17

### 下载

推荐 **Eclipse Temurin（Adoptium）**：https://adoptium.net/zh-CN/temurin/releases

| 版本 | 用途 |
|------|------|
| JDK 21 | 新项目默认 |
| JDK 17 | 老项目 / 兼容 |

安装包保存到：`D:\Packages\2026\Dev\`

### 安装路径

```
E:\Envs\Java\
├── jdk-21.0.x          # 按实际版本号命名
├── jdk-17.0.x
└── current             # 软链接，指向默认 JDK
```

### 安装注意

- 安装程序中 **不要勾选** 自动设置 JAVA_HOME
- 安装程序中 **不要勾选** 自动添加 PATH

### 创建 current 软链接（管理员 CMD）

默认指向 JDK 21：

```cmd
mklink /J E:\Envs\Java\current E:\Envs\Java\jdk-21.0.x
```

切换默认 JDK（示例：切到 17）：

```cmd
rmdir E:\Envs\Java\current
mklink /J E:\Envs\Java\current E:\Envs\Java\jdk-17.0.x
```

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `JAVA_HOME` | `E:\Envs\Java\current` |
| `JDK21_HOME` | `E:\Envs\Java\jdk-21.0.x`（可选） |
| `JDK17_HOME` | `E:\Envs\Java\jdk-17.0.x`（可选） |

> `JAVA_HOME` 指向 JDK **根目录**，不要带 `\bin`。

### 用户 Path 追加

```
%JAVA_HOME%\bin
```

### 验证

```powershell
java -version
javac -version
echo $env:JAVA_HOME
Test-Path "$env:JAVA_HOME\bin\java.exe"
```

---

## 六、Maven

### 下载

https://maven.apache.org/download.cgi

下载 **Binary zip archive**，例如 `apache-maven-3.9.15-bin.zip`

### 解压路径

```
E:\Envs\Maven\apache-maven-3.9.15
```

确认存在：`bin\mvn.cmd`、`lib\launcher.jar`

> 不要多一层目录，避免路径变成 `...\apache-maven-3.9.15\apache-maven-3.9.15\bin`

### Maven 配置

`E:\Config\maven\settings.xml`：

```xml
<localRepository>E:\Cache\Maven</localRepository>
```

可复制到 `E:\Envs\Maven\apache-maven-3.9.15\conf\settings.xml`，或在 IDEA 里指定该文件。

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `MAVEN_HOME` | `E:\Envs\Maven\apache-maven-3.9.15` |

> **不要设置** `MAVEN_OPTS`、`CLASSPATH`、`JAVA_OPTS`，除非明确需要。这些变量若格式错误会导致 `mvn` 失败。

### 用户 Path 追加

```
%MAVEN_HOME%\bin
```

### 验证

```powershell
mvn -version
echo $env:MAVEN_HOME
```

期望输出包含 `Apache Maven 3.9.15` 和 `Java version: 21.x.x`。

---

## 七、IntelliJ IDEA

### 下载

https://www.jetbrains.com/idea/download/

| 版本 | 适用 |
|------|------|
| Ultimate | 商业 / 全功能 |
| Community | 个人学习 / 开源 |

### 安装路径

```
D:\Apps\JetBrains\IntelliJ IDEA 2025.x
```

### 安装选项

| 选项 | 建议 |
|------|------|
| Add bin to PATH | 不勾 |
| 右键「用 IDEA 打开」 | 建议勾 |

### 缓存迁出 C 盘（首次启动前）

创建目录：

```
E:\Config\ide\jetbrains\IntelliJIdea\config
E:\Config\ide\jetbrains\IntelliJIdea\plugins
E:\Cache\JetBrains\IntelliJIdea\system
E:\Logs\ide\jetbrains
```

创建或编辑 `E:\Config\ide\jetbrains\idea.properties`：

```properties
idea.config.path=E:/Config/ide/jetbrains/IntelliJIdea/config
idea.system.path=E:/Cache/JetBrains/IntelliJIdea/system
idea.plugins.path=E:/Config/ide/jetbrains/IntelliJIdea/plugins
idea.log.path=E:/Logs/ide/jetbrains
```

> 路径使用正斜杠 `/`。

### 用户环境变量（可选）

| 变量名 | 变量值 |
|--------|--------|
| `GRADLE_USER_HOME` | `E:\Cache\Gradle` |

---

## 八、Node.js（nvm-windows）

### 前置：卸载直接安装的 Node

若曾用安装包装过 Node.js，先在「设置 → 应用」中卸载，避免与 nvm 冲突。

### 下载

https://github.com/coreybutler/nvm-windows/releases

下载 `nvm-setup.exe`，保存到 `D:\Packages\2026\Dev\`

### 安装路径

| 项 | 路径 |
|----|------|
| nvm 安装目录 | `E:\Envs\Node\nvm` |
| symlink 目录 | `E:\Envs\Node\nodejs` |

> nvm-windows 惯例使用 `nodejs` 作为软链目录名（类似默认的 `C:\Program Files\nodejs`）。  
> `E:\Envs\Node\nodejs` 应为空目录或不存在，由安装器创建。

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `NVM_HOME` | `E:\Envs\Node\nvm` |
| `NVM_SYMLINK` | `E:\Envs\Node\nodejs` |

### 用户 Path（安装器通常自动添加）

```
%NVM_HOME%
%NVM_SYMLINK%
```

### 安装 Node 版本

```powershell
nvm install 22
nvm install 20
nvm use 22
```

### 版本切换

```powershell
nvm use 22    # 切换到 Node 22
nvm use 20    # 切换到 Node 20
nvm list      # 查看已安装版本
```

### 验证

```powershell
node -v
where.exe node
# 应指向 E:\Envs\Node\nodejs\node.exe
```

---

## 九、npm 与 pnpm

### npm

npm 随 Node 一起安装，**无需单独安装**。

### PowerShell 执行策略

若 `npm -v` 报「禁止运行脚本」，执行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

或临时使用：

```powershell
npm.cmd -v
```

### npm 配置

```powershell
npm config set cache E:\Cache\npm
npm config set prefix E:\Envs\Node\npm-global
```

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `npm_config_cache` | `E:\Cache\npm` |

### 用户 Path 追加

```
E:\Envs\Node\npm-global
```

### pnpm 安装

推荐 corepack：

```powershell
corepack enable
corepack prepare pnpm@latest --activate
```

或：

```powershell
npm install -g pnpm
```

### pnpm 配置

```powershell
pnpm config set store-dir E:\Cache\pnpm\store
pnpm config set global-dir E:\Cache\pnpm\global
pnpm config set cache-dir E:\Cache\pnpm\cache
```

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `PNPM_HOME` | `E:\Cache\pnpm` |

### 用户 Path 追加

```
%PNPM_HOME%
```

### 验证

```powershell
node -v
npm -v
pnpm -v
nvm list
```

---

## 十、常用小工具（7-Zip / Everything）

### 7-Zip

#### 下载

https://www.7-zip.org/ → **64-bit Windows x64**

安装包保存到：`D:\Packages\2026\Dev\`

#### 安装路径

```
D:\Apps\Utilities\7-Zip
```

#### 用户 Path 追加

```
D:\Apps\Utilities\7-Zip
```

#### 验证

```powershell
7z
where.exe 7z
```

---

### Everything

#### 下载

https://www.voidtools.com/zh-cn/downloads/

推荐 **安装版（x64）**，路径：

```
D:\Apps\Utilities\Everything
```

#### 安装选项建议

| 选项 | 建议 |
|------|------|
| 开机启动 | 建议勾 |
| 安装为服务 | 按需 |
| 集成资源管理器 | 按需 |

#### 环境变量

Everything **无需配置环境变量**，开始菜单或托盘启动即可。

#### 首次设置建议

**工具 → 选项**：

| 设置 | 建议 |
|------|------|
| 索引 → NTFS | 勾选 C/D/E 盘 |
| 随 Windows 启动 | 勾 |

---

## 十一、MySQL 8.0

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `E:\Services\MySQL\MySQL Server 8.0` |
| 数据 | `E:\Data\mysql` |
| 配置文件 | `E:\Data\mysql\my.ini`（安装器生成，与服务 `--defaults-file` 绑定） |
| 日志（建议） | `E:\Logs\services\mysql` |
| 安装包 | `D:\Packages\2026\Dev\` |

### 下载

https://dev.mysql.com/downloads/installer/

下载 **MySQL Installer**（`mysql-installer-community-8.0.x.x.msi`）

### 安装步骤

1. 选择 **Custom（自定义）**
2. 勾选 **MySQL Server 8.0**
3. 程序安装路径改为：`E:\Services\MySQL\MySQL Server 8.0`
4. Config Type 选 **Development Computer**
5. Port：`3306`
6. 设置 **root 密码**
7. Windows Service Name：`MySQL80`（默认）

### Data Directory 填什么

安装器默认显示：

```
C:\ProgramData\MySQL\MySQL Server 8.0   ← 不要用这个
```

**应改为：**

```
E:\Data\mysql
```

> 安装前可先创建空目录：`New-Item -ItemType Directory -Path "E:\Data\mysql" -Force`

### 安装器已知问题：自动多建 `Data` 子目录

填 `E:\Data\mysql` 后，安装器可能仍生成：

```ini
datadir=E:/Data/mysql\Data
```

**不必反复重装**，装完后修复一次即可（见下方「装完后修复」）。

### 重装前：删除残留服务

若提示 **Windows service name 已存在**，以**管理员**执行：

```powershell
sc.exe stop MySQL80
sc.exe delete MySQL80
```

确认删除后再安装。

### 装完后修复（datadir 多 `\Data` 时）

**1. 以管理员身份**停服务：

```powershell
net stop MySQL80
```

> 启停服务需要管理员权限，否则会报「系统错误 5，拒绝访问」。

**2. 若数据在 `E:\Data\mysql\Data\`，将里面所有文件移到 `E:\Data\mysql\`**（不要覆盖 `my.ini`），删除空的 `Data` 文件夹。

**3. 编辑 `E:\Data\mysql\my.ini`，修改关键项：**

```ini
[client]
port=3306
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
port=3306
basedir=E:/Services/MySQL/MySQL Server 8.0
datadir=E:/Data/mysql

character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

log-error=E:/Logs/services/mysql/error.log
slow-query-log=1
slow_query_log_file=E:/Logs/services/mysql/slow.log
long_query_time=2
```

**4. 创建目录：**

```powershell
New-Item -ItemType Directory -Path "E:\Logs\services\mysql" -Force
New-Item -ItemType Directory -Path "E:\Data\mysql\Uploads" -Force
```

**5. 启动服务：**

```powershell
net start MySQL80
```

### my.ini 在数据目录下是否有问题

**没问题。** 服务通过 `--defaults-file` 读取配置，与 `datadir` 独立：

```
mysqld.exe --defaults-file="E:\Data\mysql\my.ini" MySQL80
```

`my.ini` 放在 `E:\Data\mysql\` 下是正常做法，改 `datadir` 不影响配置文件位置。

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `MYSQL_HOME` | `E:\Services\MySQL\MySQL Server 8.0` |

### 用户 Path 追加

```
%MYSQL_HOME%\bin
```

### 创建开发账号（推荐）

```sql
mysql -u root -p

CREATE USER 'dev'@'localhost' IDENTIFIED BY '你的密码';
GRANT ALL PRIVILEGES ON *.* TO 'dev'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

### 验证

```powershell
sc.exe qc MySQL80
mysql -u root -p -e "SELECT @@datadir, @@basedir, @@character_set_server, @@port;"
```

期望：

| 项 | 值 |
|----|-----|
| `@@datadir` | `E:\Data\mysql\` |
| `@@basedir` | `E:\Services\MySQL\MySQL Server 8.0\` |
| `@@character_set_server` | `utf8mb4` |
| `@@port` | `3306` |

### Spring Boot 连接示例

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/你的库名?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: dev
    password: 你的密码
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 常用维护命令

```powershell
net start MySQL80
net stop MySQL80
mysql -u dev -p
mysqldump -u root -p --all-databases > E:\Backup\Database\all_backup.sql
```

---

## 十二、Redis

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `E:\Services\Redis` |
| 数据 | `E:\Data\redis` |
| 日志 | `E:\Logs\services\redis` |
| 安装包 | `D:\Packages\2026\Dev\` |

### 下载

Windows 推荐使用社区维护版（稳定、免费）：

https://github.com/tporadowski/redis/releases

下载最新 **`.zip`**，例如 `Redis-x64-5.0.14.1.zip`

> 官方 Redis 在 Windows 上更推荐 WSL/Docker；本地开发用此 Windows 版最省事。

### 解压路径

解压到：

```
E:\Services\Redis
```

确认存在：`redis-server.exe`、`redis-cli.exe`、`redis.windows-service.conf`

> 不要多一层目录，避免 `E:\Services\Redis\Redis-x64-5.0.14.1\...`

### 创建数据和日志目录

```powershell
New-Item -ItemType Directory -Path "E:\Data\redis" -Force
New-Item -ItemType Directory -Path "E:\Logs\services\redis" -Force
```

### 修改配置文件

编辑 `E:\Services\Redis\redis.windows-service.conf`（**服务用这个文件**）：

```conf
bind 127.0.0.1
port 6379

dir E:/Data/redis
logfile E:/Logs/services/redis/redis.log

appendonly yes
appendfilename "appendonly.aof"
```

> 路径建议使用正斜杠 `/`。

### 安装并启动 Windows 服务（管理员）

**以管理员身份**打开 PowerShell，进入目录后执行：

```powershell
cd E:\Services\Redis

.\redis-server.exe --service-install redis.windows-service.conf --service-name Redis
.\redis-server.exe --service-start
```

> **PowerShell 必须加 `.\` 前缀**，否则报「无法将 redis-server.exe 项识别为 cmdlet」。  
> 也可用完整路径：`E:\Services\Redis\redis-server.exe --service-install ...`

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `REDIS_HOME` | `E:\Services\Redis` |

### 用户 Path 追加

```
%REDIS_HOME%
```

### 验证

**新开普通 PowerShell**：

```powershell
redis-cli ping
```

应返回 `PONG`。

```powershell
redis-cli set test hello
redis-cli get test
dir E:\Data\redis
```

应能看到 `appendonly.aof` 等持久化文件。

### Spring Boot 连接示例

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      database: 0
```

### IDEA 连接（可选）

**Database → + → Redis**

| 项 | 值 |
|----|-----|
| Host | `127.0.0.1` |
| Port | `6379` |
| Password | 留空（默认无密码） |

### 常用维护命令

```powershell
# 服务管理（管理员）
.\redis-server.exe --service-start    # 在 Redis 目录下
.\redis-server.exe --service-stop
net start Redis
net stop Redis

# 客户端
redis-cli
redis-cli -h 127.0.0.1 -p 6379
```

---

## 十三、Docker Desktop

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `C:\Program Files\Docker\Docker` 或 `D:\Apps\Docker`（安装器若允许） |
| 镜像/容器数据（WSL 虚拟盘） | `E:\Containers\Docker\wsl` |
| Compose 项目 | `E:\Containers\Docker\compose` |
| 安装包 | `D:\Packages\2026\Dev\` |

> **关键**：程序可在 C/D，**镜像和容器数据必须在 E 盘**。

### 前置：安装 WSL2

Docker Desktop 默认使用 WSL2 后端。若提示「未安装适用于 Linux 的 Windows 子系统」：

**管理员 PowerShell**：

```powershell
wsl --install
```

重启电脑后确认：

```powershell
wsl --version
wsl --set-default-version 2
wsl -l -v
```

应显示 **VERSION 2**。首次会安装 Ubuntu，按提示设置 Linux 用户名和密码。

> 需在 BIOS 开启虚拟化（Intel VT-x / AMD-V）。

### 下载

https://www.docker.com/products/docker-desktop/

下载 **Docker Desktop for Windows**，保存到 `D:\Packages\2026\Dev\`

### 安装

1. 双击安装包
2. 勾选 **Use WSL 2 instead of Hyper-V**（推荐）
3. 装完按提示重启（若需要）

### 首次启动：数据盘迁到 E 盘（最重要）

先建目录：

```powershell
New-Item -ItemType Directory -Path "E:\Containers\Docker\wsl" -Force
New-Item -ItemType Directory -Path "E:\Containers\Docker\compose" -Force
```

打开 **Docker Desktop → Settings → Resources → Advanced**（或 **Disk image location**）：

**Disk image location** 改为：

```
E:\Containers\Docker\wsl
```

点 **Apply & Restart**。

### WSL 集成（可选）

**Settings → Resources → WSL Integration**

- 开启 **Enable integration with my default WSL distro**
- 按需开启 Ubuntu 等发行版

### 资源限制（可选，16G 内存参考）

| 项 | 建议 |
|----|------|
| CPUs | 4 |
| Memory | 4 GB |
| Disk image size | 64 GB+ |

### 环境变量

Docker Desktop 通常**自动加入 Path**，一般无需手动配置。Compose 项目统一放：

```
E:\Containers\Docker\compose\{项目名}\docker-compose.yml
```

### 验证

**新开 PowerShell**（Docker Desktop 需在运行）：

```powershell
docker version
docker compose version
docker run hello-world
```

### 容器访问宿主机 MySQL/Redis

| 宿主机服务 | 容器内地址 |
|------------|------------|
| MySQL | `host.docker.internal:3306` |
| Redis | `host.docker.internal:6379` |

### 常用命令

```powershell
docker images
docker ps -a
docker compose up -d
docker compose down
docker system prune -a
```

---

## 十四、Navicat

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `D:\Apps\Database\Navicat Premium 17`（版本号按实际） |
| 安装包 | `D:\Packages\2026\Dev\` |

> Navicat 配置和数据量小，装 D 盘即可；连接本地 MySQL/Redis 走 localhost。

### 下载

https://www.navicat.com.cn/download/navicat-premium

或 **Navicat for MySQL**（仅 MySQL 时够用）

### 安装路径

安装时改路径为：

```
D:\Apps\Database\Navicat Premium 17
```

### 环境变量

**无需配置环境变量**，开始菜单启动即可。

### 连接本地 MySQL

**连接 → MySQL**

| 项 | 值 |
|----|-----|
| 连接名 | `Local MySQL` |
| 主机 | `localhost` 或 `127.0.0.1` |
| 端口 | `3306` |
| 用户名 | `dev`（或 `root`） |
| 密码 | 安装 MySQL 时设置的密码 |

点 **测试连接** → **确定**。

### 连接本地 Redis（Premium 版）

**连接 → Redis**

| 项 | 值 |
|----|-----|
| 连接名 | `Local Redis` |
| 主机 | `127.0.0.1` |
| 端口 | `6379` |
| 密码 | 留空（默认无密码） |

### 与 IDEA Database 的分工

| 工具 | 适合 |
|------|------|
| IDEA Database | 项目内快速查表、写 SQL |
| Navicat | 日常管理、导入导出、多库切换、数据对比 |

---

## 十五、Android Studio

> **Flutter 开发的前置依赖**：先装 Android Studio 和 Android SDK，再装 Flutter。

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `D:\Apps\JetBrains\Android Studio` |
| Android SDK | `E:\SDK\Android\sdk` |
| NDK（按需） | `E:\SDK\Android\ndk\版本号` |
| AVD / `.android` | `E:\SDK\Android\home` |
| Gradle 缓存 | `E:\Cache\Gradle`（已有） |
| IDE 缓存 | `E:\Cache\AndroidStudio` |
| 安装包 | `D:\Packages\2026\Dev\` |

### 下载

https://developer.android.com/studio

保存到 `D:\Packages\2026\Dev\`

### 安装路径

```
D:\Apps\JetBrains\Android Studio
```

安装选项默认即可。

### 首次启动：SDK 路径（重要）

**Setup Wizard → SDK Components Setup** 或 **Settings → Languages & Frameworks → Android SDK**：

| 项 | 值 |
|----|-----|
| Android SDK Location | `E:\SDK\Android\sdk` |

先建目录：

```powershell
New-Item -ItemType Directory -Path "E:\SDK\Android\sdk" -Force
New-Item -ItemType Directory -Path "E:\SDK\Android\ndk" -Force
New-Item -ItemType Directory -Path "E:\SDK\Android\home" -Force
New-Item -ItemType Directory -Path "E:\Cache\AndroidStudio" -Force
```

### SDK 组件建议安装

**SDK → SDK Platforms**（按需勾选）：

| 组件 | 说明 |
|------|------|
| Android 14 (API 34) | 较新 |
| Android 13 (API 33) | 常用 |

**SDK → SDK Tools**：

| 组件 | 说明 |
|------|------|
| Android SDK Build-Tools | 必装 |
| Android SDK Platform-Tools | 必装（含 adb） |
| Android Emulator | 模拟器 |
| Android SDK Command-line Tools | 必装 |
| NDK (Side by side) | Flutter/原生需要时再装 |

### IDE 缓存迁出 C 盘

**Help → Edit Custom Properties**，添加（文件不存在则新建）：

```properties
idea.config.path=E:/Config/ide/androidstudio/config
idea.system.path=E:/Cache/AndroidStudio/system
idea.plugins.path=E:/Config/ide/androidstudio/plugins
idea.log.path=E:/Logs/ide/androidstudio
```

先建目录：

```powershell
New-Item -ItemType Directory -Path "E:\Config\ide\androidstudio\config" -Force
New-Item -ItemType Directory -Path "E:\Config\ide\androidstudio\plugins" -Force
New-Item -ItemType Directory -Path "E:\Cache\AndroidStudio\system" -Force
New-Item -ItemType Directory -Path "E:\Logs\ide\androidstudio" -Force
```

保存后**重启 Android Studio**。

### Gradle 配置

**Settings → Build, Execution, Deployment → Build Tools → Gradle**

| 项 | 值 |
|----|-----|
| Gradle user home | `E:\Cache\Gradle` |
| Gradle JDK | JDK 17 或 JDK 21 |

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `ANDROID_HOME` | `E:\SDK\Android\sdk` |
| `ANDROID_SDK_ROOT` | `E:\SDK\Android\sdk` |
| `ANDROID_SDK_HOME` | `E:\SDK\Android\home` |
| `ANDROID_AVD_HOME` | `E:\SDK\Android\home\avd` |

> `ANDROID_SDK_HOME` 让 `.android` 等目录走 E 盘，避免 AVD 占 C 盘。

### 用户 Path 追加

```
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\cmdline-tools\latest\bin
%ANDROID_HOME%\emulator
```

> `cmdline-tools\latest\bin` 路径需在 SDK Manager 安装 Command-line Tools 后确认实际目录。

### 验证

**新开 PowerShell**：

```powershell
echo $env:ANDROID_HOME
adb version
sdkmanager --list
```

### 创建模拟器（AVD）

**Tools → Device Manager → Create Device**

选机型 → 选系统镜像（Download 若未装）→ Finish。

数据保存在 `E:\SDK\Android\home\avd`（配置了 `ANDROID_AVD_HOME` 后）。

---

## 十六、Flutter

> **前置**：JDK、Android Studio、Android SDK 已装好。

### 路径规划

| 用途 | 路径 |
|------|------|
| Flutter SDK | `E:\SDK\Flutter\flutter` |
| Dart/Flutter 包缓存 | `E:\Cache\pub` |
| 项目源码 | `E:\Workspace\Personal\` 或 `Company\` |

### 下载 Flutter SDK

**方式 A：Git 克隆（推荐）**

```powershell
cd E:\SDK\Flutter
git clone https://github.com/flutter/flutter.git -b stable
```

**方式 B：zip 解压**

https://docs.flutter.dev/get-started/install/windows

解压到 `E:\SDK\Flutter\flutter`（确保 `flutter\bin\flutter.bat` 存在）

### 用户环境变量

| 变量名 | 变量值 |
|--------|--------|
| `PUB_CACHE` | `E:\Cache\pub` |
| `FLUTTER_STORAGE_BASE_URL` | `https://storage.flutter-io.cn`（国内镜像，可选） |
| `PUB_HOSTED_URL` | `https://pub.flutter-io.cn`（国内镜像，可选） |

> 国内网络建议设置 Flutter / Pub 镜像，加速 SDK 和包下载。

### 用户 Path 追加

```
E:\SDK\Flutter\flutter\bin
```

### 配置 Android SDK 路径

```powershell
flutter config --android-sdk E:\SDK\Android\sdk
```

### 接受 Android 许可

```powershell
flutter doctor --android-licenses
```

全部输入 `y` 接受。

### 验证（核心命令）

```powershell
flutter doctor -v
```

期望（Android 开发）：

| 检查项 | 期望 |
|--------|------|
| Flutter | ✅ stable channel |
| Android toolchain | ✅ |
| Android Studio | ✅ |
| Chrome | ✅ 或 ⚠️（Web 开发需要） |
| Network resources | ✅ 或 ⚠️（国内可能需镜像） |

### 创建并运行测试项目

```powershell
cd E:\Workspace\Sandbox
flutter create hello_flutter
cd hello_flutter
flutter run
```

连接真机或启动 Android 模拟器后执行 `flutter run`。

### 常用命令

```powershell
flutter doctor
flutter upgrade
flutter pub get
flutter clean
flutter devices
flutter emulators
flutter emulators --launch <emulator_id>
```

### VS Code / IDEA 插件（可选）

| IDE | 插件 |
|-----|------|
| Android Studio | 内置 Flutter / Dart 支持，**Plugins → Flutter** |
| IntelliJ IDEA | **Plugins → Flutter** + **Dart** |

IDEA 中 **Settings → Languages & Frameworks → Flutter**：

| 项 | 值 |
|----|-----|
| Flutter SDK path | `E:\SDK\Flutter\flutter` |

---

## 十七、环境变量汇总

> 全部配置在 **用户变量** 和 **用户 Path**，**不要修改系统变量**。

### 用户变量

| 变量名 | 变量值 |
|--------|--------|
| `GIT_CONFIG_GLOBAL` | `E:\Config\git\.gitconfig` |
| `TEMP` | `E:\Temp` |
| `TMP` | `E:\Temp` |
| `JAVA_HOME` | `E:\Envs\Java\current` |
| `MAVEN_HOME` | `E:\Envs\Maven\apache-maven-3.9.15` |
| `GRADLE_USER_HOME` | `E:\Cache\Gradle` |
| `NVM_HOME` | `E:\Envs\Node\nvm` |
| `NVM_SYMLINK` | `E:\Envs\Node\nodejs` |
| `npm_config_cache` | `E:\Cache\npm` |
| `PNPM_HOME` | `E:\Cache\pnpm` |
| `MYSQL_HOME` | `E:\Services\MySQL\MySQL Server 8.0` |
| `REDIS_HOME` | `E:\Services\Redis` |
| `ANDROID_HOME` | `E:\SDK\Android\sdk` |
| `ANDROID_SDK_ROOT` | `E:\SDK\Android\sdk` |
| `ANDROID_SDK_HOME` | `E:\SDK\Android\home` |
| `ANDROID_AVD_HOME` | `E:\SDK\Android\home\avd` |
| `PUB_CACHE` | `E:\Cache\pub` |
| `FLUTTER_STORAGE_BASE_URL` | `https://storage.flutter-io.cn`（国内可选） |
| `PUB_HOSTED_URL` | `https://pub.flutter-io.cn`（国内可选） |
| `OLLAMA_MODELS` | `E:\AI\Models\ollama` |

### 用户 Path（完整参考）

```
D:\Portable\VCS\Git\cmd
D:\Portable\bin
D:\Apps\Utilities\7-Zip
%USERPROFILE%\.local\bin
%JAVA_HOME%\bin
%MAVEN_HOME%\bin
%MYSQL_HOME%\bin
%REDIS_HOME%
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\cmdline-tools\latest\bin
%ANDROID_HOME%\emulator
E:\SDK\Flutter\flutter\bin
D:\Apps\Cursor\resources\app\bin
%NVM_HOME%
%NVM_SYMLINK%
E:\Envs\Node\npm-global
%PNPM_HOME%
E:\Tools\bin
```

### 用 PowerShell 修复 TEMP（若含隐藏字符）

```powershell
[Environment]::SetEnvironmentVariable('TEMP', 'E:\Temp', 'User')
[Environment]::SetEnvironmentVariable('TMP', 'E:\Temp', 'User')
```

验证：

```powershell
echo "[$env:TEMP]"
[int][char]$env:TEMP[0]   # 应输出 69（字母 E）
```

---

## 十八、IDEA 配置清单

| 位置 | 配置项 | 值 |
|------|--------|-----|
| Project Structure → SDKs | JDK 21 | `E:\Envs\Java\jdk-21.0.x` |
| Project Structure → SDKs | JDK 17 | `E:\Envs\Java\jdk-17.0.x` |
| Settings → Maven | Maven home | `E:\Envs\Maven\apache-maven-3.9.15` |
| Settings → Maven | User settings | `E:\Config\maven\settings.xml` |
| Settings → Maven | Local repository | `E:\Cache\Maven` |
| Settings → Gradle | Gradle user home | `E:\Cache\Gradle` |
| Settings → System Settings | Default project directory | `E:\Workspace` |
| Settings → Git | Git executable | `D:\Portable\VCS\Git\cmd\git.exe` |
| Settings → Node.js | Node interpreter | `E:\Envs\Node\nodejs\node.exe` |
| Settings → Flutter | Flutter SDK path | `E:\SDK\Flutter\flutter` |
| Settings → Dart | Dart SDK path | 随 Flutter 自动识别 |
| Database | MySQL 数据源 | `localhost:3306`，用户 `dev` |
| Database | Redis | `127.0.0.1:6379` |
| Settings → Docker | Docker | Docker for Windows（自动识别） |
| Plugins | Lombok | 安装并启用 Annotation Processing |
| Plugins | Flutter + Dart | Flutter 开发时安装 |

---

## 十九、常见问题

### 1. `mvn` 报 Java 用法说明 / `' '` 不是内部命令

**原因**：`CLASSPATH`、`MAVEN_OPTS`、`JAVA_OPTS` 等环境变量有脏值。

**处理**：

1. 删除用户变量和系统变量中的 `CLASSPATH`、`MAVEN_OPTS`、`JAVA_OPTS`
2. 确认 `MAVEN_HOME` 版本号与实际目录一致（如 `3.9.15`）
3. 临时测试：

```powershell
cmd.exe /c "set MAVEN_OPTS=& set CLASSPATH=& set JAVA_HOME=E:\Envs\Java\current& E:\Envs\Maven\apache-maven-3.9.15\bin\mvn.cmd -version"
```

### 2. IDEA 提示 TEMP 目录不存在

**原因**：`TEMP` 值含前导空格或换行（如 ` E:\Temp` 或 `\nE:\Temp`）。

**处理**：

```powershell
[Environment]::SetEnvironmentVariable('TEMP', 'E:\Temp', 'User')
[Environment]::SetEnvironmentVariable('TMP', 'E:\Temp', 'User')
```

确认 `E:\Temp` 目录存在，重启 IDEA。

### 3. `npm` 报 PowerShell 禁止运行脚本

**处理**：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 4. `java` 正常但 `mvn` 不行

**原因**：`java` 走 Path，但 `JAVA_HOME` 未设或指错。

**处理**：确认 `JAVA_HOME=E:\Envs\Java\current` 且 `Test-Path "$env:JAVA_HOME\bin\java.exe"` 为 `True`。

### 5. 尽量使用普通 PowerShell

`C:\WINDOWS\system32>` 通常是**管理员终端**，用户环境变量可能表现异常。日常开发用**普通 PowerShell**。

### 6. 环境变量修改后

必须**新开终端**或**重启 IDEA** 才生效。

### 7. MySQL Data Directory 默认是 ProgramData

安装器 **Data Directory** 默认显示 `C:\ProgramData\MySQL\MySQL Server 8.0`，必须手动改为 `E:\Data\mysql`。默认路径是 C 盘数据目录，不是配置文件专用路径。

### 8. MySQL 装完 datadir 变成 `E:\Data\mysql\Data`

安装器已知行为，**不必反复重装**。停服务后把 `Data` 子目录里的文件上移到 `E:\Data\mysql\`，修改 `my.ini` 中 `datadir=E:/Data/mysql`，再启动服务。

### 9. `net stop MySQL80` 拒绝访问（系统错误 5）

需要**以管理员身份**运行 PowerShell 或 CMD，或使用 `services.msc` 图形界面启停服务。

### 10. MySQL 重装提示服务名已存在

```powershell
sc.exe stop MySQL80
sc.exe delete MySQL80
```

删除残留服务后再安装。

### 11. PowerShell 复制命令时带上提示符

不要复制 `PS C:\...>`、`>>`、`True` 等输出内容，只复制命令本身。

### 12. PowerShell 找不到 `redis-server.exe`

**原因**：PowerShell 默认不从当前目录加载可执行文件。

**处理**：在 Redis 目录下使用 `.\` 前缀：

```powershell
cd E:\Services\Redis
.\redis-server.exe --service-install redis.windows-service.conf --service-name Redis
.\redis-server.exe --service-start
```

配置 `%REDIS_HOME%` 到用户 Path 后，新开终端可直接用 `redis-cli`。

### 13. Docker 提示未安装 WSL

**处理**：管理员 PowerShell 执行 `wsl --install`，重启后 `wsl --set-default-version 2`，再启动 Docker Desktop。

### 14. Docker 镜像仍占 C 盘

**处理**：Docker Desktop → Settings → Resources → **Disk image location** 改为 `E:\Containers\Docker\wsl`，Apply & Restart。

### 15. `flutter doctor` Android licenses 未接受

**处理**：

```powershell
flutter doctor --android-licenses
```

全部输入 `y`。

### 16. `adb` 找不到

**处理**：确认用户 Path 含 `%ANDROID_HOME%\platform-tools`，且 `ANDROID_HOME=E:\SDK\Android\sdk`。

### 17. Android 模拟器 / AVD 占 C 盘

**处理**：设置用户变量 `ANDROID_SDK_HOME=E:\SDK\Android\home` 和 `ANDROID_AVD_HOME=E:\SDK\Android\home\avd`，在 Device Manager 新建 AVD。

### 18. Flutter 下载慢

**处理**：设置国内镜像（用户变量）：

```
FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
PUB_HOSTED_URL=https://pub.flutter-io.cn
```

### 19. `claude` 命令找不到

**处理**：用户 Path 追加 `%USERPROFILE%\.local\bin`，**新开终端**再试。

### 20. Codex 装错包

**处理**：确认安装的是 `npm install -g @openai/codex`，不是 `codex`。

### 21. CC Switch 切换后 API 仍不通

**处理**：Codex 切换后新开终端；检查 API Key、Base URL；Claude Code 可在 CC Switch 里重新点「使用」。

---

## 二十、Cursor

> Cursor 是基于 VS Code 的 AI 代码编辑器，与 IDEA 互补：后端 Java 用 IDEA，全栈/前端/AI 辅助编码用 Cursor。

### 路径规划

| 用途 | C 盘默认路径 | E 盘目标路径 |
|------|--------------|--------------|
| 程序 | — | `D:\Apps\Cursor`（安装器 Browse 改路径） |
| 应用数据（Roaming） | `%APPDATA%\Cursor` | `E:\Cache\Cursor\Roaming` |
| 本地缓存（Local） | `%LOCALAPPDATA%\Cursor` | `E:\Cache\Cursor\Local` |
| 用户配置（可选） | `%USERPROFILE%\.cursor` | `E:\Config\ide\cursor` |
| 安装包 | — | `D:\Packages\2026\Dev\` |

> Cursor **没有**像 IDEA 那样的官方缓存路径配置项。迁出 C 盘需用 **NTFS Junction（目录联接）**：C 盘保留原路径，实际数据写到 E 盘。这是目前最稳定的方案。

### 下载

https://cursor.com/download

下载 **Cursor User Setup x64**（用户安装版，无需管理员）

### 安装

1. 双击 `CursorUserSetup-x64.exe`
2. **Destination Location** 改为：

```
D:\Apps\Cursor
```

3. 建议勾选：
   - **Add to PATH**（命令行 `cursor` 可用）
   - **Create a desktop icon**
   - 注册右键菜单（按需）

> **只装一份**：不要同时存在「系统安装（Program Files）」和「用户安装（AppData）」，否则更新容易冲突。

### 用户 Path（若安装时未自动添加）

```
D:\Apps\Cursor\resources\app\bin
```

### 缓存迁出 C 盘（Junction）

Cursor 的扩展、索引、缓存主要占用 `%APPDATA%\Cursor` 和 `%LOCALAPPDATA%\Cursor`。通过 Junction 迁到 E 盘后，C 盘只保留联接，不再堆积数据。

**方式 A：首次启动前（推荐）**

安装完 Cursor、尚未首次打开时执行：

```powershell
# 管理员 PowerShell
New-Item -ItemType Directory -Path "E:\Cache\Cursor\Roaming" -Force
New-Item -ItemType Directory -Path "E:\Cache\Cursor\Local" -Force

New-Item -ItemType Junction -Path "$env:APPDATA\Cursor" -Target "E:\Cache\Cursor\Roaming"
New-Item -ItemType Junction -Path "$env:LOCALAPPDATA\Cursor" -Target "E:\Cache\Cursor\Local"
```

**方式 B：已安装并使用过 Cursor**

1. **完全退出 Cursor**（任务管理器确认无 `Cursor.exe`）
2. **管理员 PowerShell** 执行：

```powershell
New-Item -ItemType Directory -Path "E:\Cache\Cursor\Roaming" -Force
New-Item -ItemType Directory -Path "E:\Cache\Cursor\Local" -Force

# Roaming（扩展、索引、workspaceStorage，体积最大）
if (Test-Path "$env:APPDATA\Cursor") {
    robocopy "$env:APPDATA\Cursor" "E:\Cache\Cursor\Roaming" /E /MOVE /R:1 /W:1
    Remove-Item "$env:APPDATA\Cursor" -Recurse -Force -ErrorAction SilentlyContinue
}
New-Item -ItemType Junction -Path "$env:APPDATA\Cursor" -Target "E:\Cache\Cursor\Roaming"

# Local（运行时缓存）
if (Test-Path "$env:LOCALAPPDATA\Cursor") {
    robocopy "$env:LOCALAPPDATA\Cursor" "E:\Cache\Cursor\Local" /E /MOVE /R:1 /W:1
    Remove-Item "$env:LOCALAPPDATA\Cursor" -Recurse -Force -ErrorAction SilentlyContinue
}
New-Item -ItemType Junction -Path "$env:LOCALAPPDATA\Cursor" -Target "E:\Cache\Cursor\Local"
```

**可选：`.cursor` 用户配置一并迁出**

```powershell
New-Item -ItemType Directory -Path "E:\Config\ide\cursor" -Force

if (Test-Path "$env:USERPROFILE\.cursor") {
    robocopy "$env:USERPROFILE\.cursor" "E:\Config\ide\cursor" /E /MOVE /R:1 /W:1
    Remove-Item "$env:USERPROFILE\.cursor" -Recurse -Force -ErrorAction SilentlyContinue
}
New-Item -ItemType Junction -Path "$env:USERPROFILE\.cursor" -Target "E:\Config\ide\cursor"
```

> `.cursor` 体积通常较小，不迁也不影响 C 盘空间。

**验证 Junction 是否生效**

```powershell
Get-Item "$env:APPDATA\Cursor" | Select-Object FullName, LinkType, Target
Get-Item "$env:LOCALAPPDATA\Cursor" | Select-Object FullName, LinkType, Target
```

期望输出：

```
FullName                              LinkType Target
--------                              -------- ------
C:\Users\<用户名>\AppData\Roaming\Cursor Junction {E:\Cache\Cursor\Roaming}

FullName                            LinkType Target
--------                            -------- ------
C:\Users\<用户名>\AppData\Local\Cursor Junction {E:\Cache\Cursor\Local}
```

**注意事项**

| 项 | 说明 |
|----|------|
| 权限 | 创建 Junction 需**管理员 PowerShell** |
| 更新 | Cursor 更新前建议完全退出，避免写入冲突 |
| 清理缓存 | 只删 E 盘下的 `Cache`、`CachedData`；**不要删**整个 `Roaming` 目录或 C 盘 Junction 本身 |
| 索引重建 | 迁移后首次打开大项目，索引可能重建，属正常现象 |

### 首次启动

1. 登录 Cursor 账号
2. 选择主题、快捷键（可选 VS Code 键位）
3. **Settings → 可安装中文语言包**（扩展搜 Chinese）

> 若采用「方式 A」，Junction 应在**首次启动前**建好；若已启动过，用「方式 B」。

### 与 E 盘工作区配合

```powershell
cd E:\Workspace\Company\your-project
cursor .
```

### 企业环境安装失败（TEMP 权限）

若报「Unable to execute file in the temporary directory」：

```powershell
New-Item -ItemType Directory -Path "E:\Temp" -Force
$env:TEMP = "E:\Temp"
$env:TMP = "E:\Temp"
.\CursorUserSetup-x64.exe
```

确保 `TEMP`/`TMP` 用户变量指向 `E:\Temp`（见 Git 章节）。

### 验证

```powershell
cursor --version

# Junction 应指向 E 盘
Get-Item "$env:APPDATA\Cursor" | Select-Object LinkType, Target
Get-Item "$env:LOCALAPPDATA\Cursor" | Select-Object LinkType, Target
```

---

## 二十一、Claude Code 与 Codex

> 两款 **AI 终端编程 CLI**，在命令行里做代码生成、重构、调试。  
> **前置**：Node.js 22+（nvm `nvm use 22`）、Git（已装）。

### 路径规划

| 工具 | 程序/配置位置 |
|------|---------------|
| Claude Code | 程序 `%USERPROFILE%\.local\bin\claude.exe` |
| Claude 配置 | `%USERPROFILE%\.claude\` |
| Codex CLI | npm 全局（`E:\Envs\Node\npm-global`） |
| Codex 配置 | `%USERPROFILE%\.codex\config.toml` |

> CLI 配置目录默认在 C 盘用户目录，体积小；**用 CC Switch 统一管理**，无需手改 JSON。

---

### Claude Code 安装

**官方推荐：原生安装（自动更新）**

**普通 PowerShell**：

```powershell
irm https://claude.ai/install.ps1 | iex
```

或使用 WinGet：

```powershell
winget install Anthropic.ClaudeCode
```

**备选：npm 安装**

```powershell
npm install -g @anthropic-ai/claude-code
```

### Claude Code 用户 Path 追加

原生安装后需确保（安装器通常自动添加）：

```
%USERPROFILE%\.local\bin
```

### Claude Code 验证

**关掉终端，重新打开**，然后：

```powershell
claude --version
claude
```

首次运行按提示登录 Anthropic 账号。

> 若提示 `'claude' 不是内部命令`：检查 Path 是否含 `%USERPROFILE%\.local\bin`，新开终端再试。

---

### Codex CLI 安装

**前置：Node.js 22+**

```powershell
nvm use 22
npm install -g @openai/codex
```

> 包名必须是 **`@openai/codex`**，不是 `codex`（后者是无关旧包）。

### Codex 配置（Windows 沙箱）

创建或编辑 `%USERPROFILE%\.codex\config.toml`：

```toml
[windows]
sandbox = "elevated"   # 推荐，需管理员权限配置一次
# sandbox = "unelevated"  # 无管理员权限时的备选
```

### Codex 验证

```powershell
codex --version
codex
```

首次运行通过浏览器登录 ChatGPT 或配置 API Key。

---

### 三者分工

| 工具 | 场景 |
|------|------|
| **Cursor** | 图形界面 AI 编码，日常写代码 |
| **Claude Code** | 终端里 Claude 驱动，适合 Agent 式改项目 |
| **Codex** | 终端里 OpenAI 驱动，ChatGPT 生态 |

---

## 二十二、CC Switch

### 这是什么

**CC Switch** 是一款开源桌面工具，用来**可视化管理、一键切换** AI 编程 CLI 的 API 供应商配置。

官网：https://ccswitch.io/  
GitHub：https://github.com/farion1231/cc-switch

| 功能 | 说明 |
|------|------|
| 一键切换 Provider | 官方 API、国内镜像、第三方代理，点一下切换 |
| 多工具统一管理 | Claude Code、Codex、Gemini CLI、OpenCode 等 |
| MCP 管理 | 可视化配置 MCP 服务器 |
| 用量统计 | Token 消耗与费用（视供应商） |
| 配置备份 | 自动备份，防误操作 |

**适合你如果**：需要在多个 API 供应商之间频繁切换，不想手改 `~/.claude/settings.json` 或 `~/.codex/config.toml`。

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序（MSI 安装） | `D:\Apps\Utilities\CC-Switch` |
| 程序（便携版） | `D:\Portable\Sys\CC-Switch` |
| 自身数据 | 应用数据目录（SQLite，体积小） |
| 管理的 CLI 配置 | 写入 `%USERPROFILE%\.claude\`、`%USERPROFILE%\.codex\` 等 |

### 前置条件

1. **先手动装好** Claude Code、Codex 等 CLI（CC Switch **不负责安装** CLI，Windows 版已禁用一键安装）
2. Node.js 18+ 已装（nvm use 22）

### 下载

https://github.com/farion1231/cc-switch/releases

| 文件 | 说明 |
|------|------|
| `CC-Switch-vX.X.X-Windows.msi` | 推荐，支持自动更新 |
| `CC-Switch-vX.X.X-Windows-Portable.zip` | 便携版，解压即用 |

### 安装

**MSI 版**：双击安装，路径可改为 `D:\Apps\Utilities\CC-Switch`

**便携版**：解压到 `D:\Portable\Sys\CC-Switch`，运行 `CC-Switch.exe`

### 使用流程

1. 打开 CC Switch
2. 顶部选择要管理的工具（**Claude Code** 或 **Codex**）
3. 点 **添加供应商**，填写：
   - 名称（如 `官方`、`DeepSeek`、`第三方代理`）
   - API Key
   - Base URL（若用镜像/代理）
   - 模型名
4. 选中供应商 → 点 **使用 / Enable**
5. CC Switch 自动写入对应配置文件

> **Claude Code** 支持热切换，多数情况**不用重启终端**。Codex 切换后建议**新开终端**。

### 与 Claude Code / Codex 的关系

```
你手动安装 Claude Code、Codex
         ↓
CC Switch 管理它们的 API 配置（不写代码，点 GUI）
         ↓
终端里正常运行 claude / codex 命令
```

### 验证

1. 在 CC Switch 里添加并启用一个供应商
2. 新开 PowerShell：

```powershell
claude --version
codex --version
```

3. 运行 `claude` 或 `codex`，确认能连上 API

---

## 二十三、本地 AI 模型（Ollama）

> 若需要**离线/本地**跑大模型（Llama、Qwen、DeepSeek 等），用 Ollama。与 Claude Code/Codex **不冲突**——本地模型通过 Ollama API 供其他工具调用。

### 路径规划

| 用途 | 路径 |
|------|------|
| 程序 | `D:\Apps\AI\Ollama` 或默认安装路径 |
| 模型文件 | `E:\AI\Models\ollama` |
| 缓存 | `E:\AI\Cache` |

### 下载

https://ollama.com/download/windows

### 用户环境变量（安装前设置，模型存 E 盘）

| 变量名 | 变量值 |
|--------|--------|
| `OLLAMA_MODELS` | `E:\AI\Models\ollama` |

先建目录：

```powershell
New-Item -ItemType Directory -Path "E:\AI\Models\ollama" -Force
New-Item -ItemType Directory -Path "E:\AI\Cache" -Force
```

### 安装

双击安装包，若可改路径选 `D:\Apps\AI\Ollama`。

### 拉取模型

```powershell
ollama pull qwen2.5:7b
ollama pull llama3.2
ollama list
```

### 验证

```powershell
ollama run qwen2.5:7b
```

另开终端测试 API：

```powershell
curl http://localhost:11434/api/tags
```

### 在 Cursor 中使用 Ollama（可选）

Cursor **Settings → Models** 可配置 OpenAI 兼容端点：

```
Base URL: http://localhost:11434/v1
```

具体可用模型名以 Ollama 实际拉取的为准。

---

## 二十四、后续按需安装

| 软件 | 安装位置 | 何时装 |
|------|----------|--------|
| 微信 / QQ | `D:\Apps\Communication\` | 日常通讯 |
| Postman / Apifox | `D:\Portable\API\` | 接口调试 |
| DBeaver | `D:\Portable\DB\` 或 `D:\Apps\Database\` | 开源数据库客户端 |
| Python | `E:\Envs\Python\` | Python 开发 |
| Go | `E:\Envs\Go\` | Go 开发 |

### 微信 / QQ

> 程序装 **D 盘**，聊天记录和文件改到 **E 盘**。C 盘只保留小体积配置（`%APPDATA%\Tencent\` 等），无需 Junction。

#### 路径规划

| 用途 | 路径 |
|------|------|
| 微信程序 | `D:\Apps\Communication\WeChat` |
| QQ 程序 | `D:\Apps\Communication\QQ` |
| 微信聊天记录/文件 | `E:\Data\wechat` |
| QQ 消息/文件/缓存 | `E:\Data\qq` |
| 安装包 | `D:\Packages\2026\Tools\` |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Apps\Communication\WeChat" -Force
New-Item -ItemType Directory -Path "D:\Apps\Communication\QQ" -Force
New-Item -ItemType Directory -Path "E:\Data\wechat" -Force
New-Item -ItemType Directory -Path "E:\Data\qq" -Force
```

#### 安装程序（D 盘）

安装时 **Destination** 改为：

```
D:\Apps\Communication\WeChat
D:\Apps\Communication\QQ
```

> 安装器默认常指向 C 盘，务必点 **浏览** 改路径。同类软件（钉钉、飞书）也可放 `D:\Apps\Communication\`。

#### 聊天记录迁到 E 盘（必做）

程序在 D 盘后，聊天文件默认仍可能写入 C 盘，需在软件内修改：

**微信**：**设置 → 文件管理 → 更改** → `E:\Data\wechat`

**QQ（QQNT）**：**设置 → 存储管理 / 文件管理** → `E:\Data\qq`

#### 注意事项

| 项 | 说明 |
|----|------|
| 不要放 E:\Workspace | E 盘工作区专用于源码，不与聊天数据混放 |
| C 盘残留 | `%APPDATA%\Tencent\` 等配置目录体积小，可保留 |
| 空间占用 | 聊天记录、图片、文件传输是主要体积，改完文件管理路径后 C 盘不再堆积 |

### 后续环境变量（按需添加，均为用户变量）

| 变量名 | 变量值 | 何时加 |
|--------|--------|--------|
| `GOPATH` | `E:\Envs\Go\gopath` | Go 开发 |
| `GOMODCACHE` | `E:\Cache\go-mod` | Go 开发 |
| `PIP_CACHE_DIR` | `E:\Cache\pip` | Python 开发 |

> `OLLAMA_MODELS` 已移至 [Ollama 章节](#二十三本地-ai-模型ollama)。

---

## 安装完成检查清单

- [ ] D/E 盘目录结构已就绪
- [ ] Git 可用，`git config` 已配置
- [ ] JDK 21 / 17 已装，`java -version` 正常
- [ ] Maven 可用，`mvn -version` 正常
- [ ] IDEA 已装，缓存/config 在 E 盘
- [ ] IDEA 中 Maven / Git / JDK 已配置
- [ ] nvm + Node 22/20 已装，`nvm use` 可切换
- [ ] npm / pnpm 可用，缓存在 `E:\Cache`
- [ ] `TEMP` / `TMP` 指向 `E:\Temp` 且无隐藏字符
- [ ] 7-Zip、Everything 已装
- [ ] MySQL 已装，`@@datadir` 为 `E:\Data\mysql\`
- [ ] MySQL 已创建 `dev` 账号，IDEA 可连接
- [ ] Redis 已装，`redis-cli ping` 返回 `PONG`
- [ ] Redis 数据在 `E:\Data\redis`
- [ ] WSL2 已装，`wsl -l -v` 显示 VERSION 2
- [ ] Docker Desktop 已装，Disk image 在 `E:\Containers\Docker\wsl`
- [ ] `docker run hello-world` 成功
- [ ] Navicat 已装，可连接 MySQL / Redis
- [ ] Android Studio 已装，SDK 在 `E:\SDK\Android\sdk`
- [ ] `adb version` 正常，AVD 数据在 E 盘
- [ ] Flutter SDK 在 `E:\SDK\Flutter\flutter`
- [ ] `flutter doctor` 无阻塞性错误
- [ ] Cursor 已装到 `D:\Apps\Cursor`，`cursor` 命令可用
- [ ] Cursor 缓存已通过 Junction 迁到 `E:\Cache\Cursor\Roaming` / `Local`
- [ ] Claude Code 已装，`claude --version` 正常
- [ ] Codex 已装，`codex --version` 正常
- [ ] CC Switch 已装，可切换 API 供应商
- [ ] Ollama 已装（可选），模型在 `E:\AI\Models\ollama`
- [ ] 微信 / QQ 程序在 `D:\Apps\Communication\`，聊天文件在 `E:\Data\wechat` / `qq`
- [ ] 在 `E:\Workspace` 下成功打开并运行过一个项目

---

*文档版本：2026-07-02（含 Cursor Junction / 微信 QQ / Claude Code / Codex / CC Switch / Ollama）*
