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



## 一、磁盘规划


| 盘符     | 角色  | 原则                            |
| ------ | --- | ----------------------------- |
| **C:** | 系统盘 | 仅 Windows + 驱动 + 少量必须装 C 盘的软件 |
| **D:** | 应用盘 | 正式安装软件、绿色工具、安装包归档             |
| **E:** | 工作盘 | 源码、SDK、运行时、数据、缓存、备份           |




### D 盘结构

```
D:\
├── Apps\              # 正式安装（IDEA、Android Studio、Navicat、7-Zip 等）
│   ├── JetBrains\     # IDEA、Android Studio
│   ├── Database\      # Navicat
│   ├── Communication\ # 微信、QQ、钉钉、ToDesk 等
│   ├── Games\         # Steam 客户端
│   └── Utilities\     # 7-Zip、Everything、Termius 等
├── Portable\          # 绿色/便携工具（Git、Clash、OBS 等）
│   ├── VCS\           # Git、Fork 相关
│   ├── Network\       # Clash Verge Rev 便携版
│   └── Media\         # OBS Studio 便携版
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
├── Envs\              # 语言/构建工具（Java、Maven、Node、Python 等）
│   └── Python\        # Python313 等
├── SDK\               # 平台 SDK（Android SDK、Flutter 等）
│   ├── Android\       # sdk、ndk、home（AVD）
│   └── Flutter\       # flutter SDK
├── Services\          # 本地服务（MySQL、Redis 等）
├── Data\              # 持久化数据（MySQL、Redis、微信/QQ 聊天记录等）
│   ├── wechat\        # 微信文件管理目录
│   ├── qq\            # QQ 文件/缓存目录
│   ├── obs\           # OBS 录像/输出（可选）
│   ├── claude\        # Claude Code 数据（CLAUDE_CONFIG_DIR）
│   └── codex\         # Codex 数据（CODEX_HOME）
├── Cache\             # 构建与包管理缓存（Gradle、pub、pip、AndroidStudio、Cursor 等）
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


| 文件                                                | 用途         |
| ------------------------------------------------- | ---------- |
| `E:\Config\maven\settings.xml`                    | Maven 本地仓库 |
| `E:\Config\ide\jetbrains\idea.properties.example` | IDEA 路径重定向 |
| `E:\Config\git\.gitconfig.example`                | Git 全局配置模板 |


---



## 三、安装顺序总览

```
① 目录结构（D/E 盘）
② Git（可选：Fork 图形客户端，见 [Git 章节](#fork可选git-图形客户端)）
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
⑮ Claude Code + Codex（先设数据目录环境变量 → 装 CLI → 验证命令）
⑯ CC Switch（配 API Key 供应商 → 再正式使用 CLI）
⑰ Ollama（本地大模型，可选）
⑱ Python（按需：装 E 盘 + pip 缓存 + Path）
⑲ 按需：Termius / OBS / Clash / Steam / Postman / Apifox / Go ...
```

---



## 四、Git



### 下载

[https://git-scm.com/download/win](https://git-scm.com/download/win)

安装包保存到：`D:\Packages\2026\Dev\`

### 安装路径

```
D:\Portable\VCS\Git
```



### 安装选项建议


| 选项    | 建议                                                             |
| ----- | -------------------------------------------------------------- |
| PATH  | **Git from the command line and also from 3rd-party software** |
| HTTPS | Use the OpenSSL library                                        |
| 换行    | Checkout Windows-style, commit Unix-style                      |
| 终端    | Use Windows' default console window                            |




### Git 配置文件

1. 将 `E:\Config\git\.gitconfig.example` 重命名为 `.gitconfig`
2. 填写 `user.name` 和 `user.email`



### 用户环境变量


| 变量名                 | 变量值                        |
| ------------------- | -------------------------- |
| `GIT_CONFIG_GLOBAL` | `E:\Config\git\.gitconfig` |
| `TEMP`              | `E:\Temp`                  |
| `TMP`               | `E:\Temp`                  |


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



### Fork（可选：Git 图形客户端）

> 与 Cursor / IDEA 内置 Git **互补**：适合可视化 diff、交互式 rebase、解决合并冲突、浏览历史。  
> 程序本体因 Velopack 自动更新机制**只能装在** `%LOCALAPPDATA%\Fork`（与 Codex 类似，安装器**无路径选项**）。  
> **必做**：在 Fork 里指向已装的 `D:\Portable\VCS\Git`，避免 Fork 再下载一份约 **400MB** 的 bundled Git 到 C 盘。

#### 路径规划

| 用途 | 路径 |
|------|------|
| Fork 程序 | `%LOCALAPPDATA%\Fork`（无法改到 D 盘，属正常） |
| 复用 Git | `D:\Portable\VCS\Git\cmd\git.exe` |
| 安装包归档 | `D:\Packages\2026\Dev\` |
| 仓库源码 | `E:\Workspace\...`（在 Fork 里打开/克隆到此） |

#### 前置条件

- 本章 Git 已装好，`git --version`、`git config --global user.name` 正常

#### 下载与安装

1. 打开 [https://git-fork.com/](https://git-fork.com/) → **Download Fork for Windows**
2. 安装包保存到 `D:\Packages\2026\Dev\`
3. 运行安装器，按向导完成（**没有**自定义安装路径）
4. 从开始菜单或 `%LOCALAPPDATA%\Fork\Fork.exe` 启动

> Fork 有免费试用期，之后需购买授权；试用期内功能完整，足够验证环境。

#### 配置：使用本机 Git（必做）

**File → Preferences → Git → Git Instance** 选 **Custom**，路径填：

```text
D:\Portable\VCS\Git\cmd\git.exe
```

或直接编辑 `%LOCALAPPDATA%\Fork\settings.json`（`CustomGitInstancePath` 必须用**正斜杠**）：

```json
"CustomGitInstancePath": "D:/Portable/VCS/Git/cmd/git.exe"
```

保存后**完全退出 Fork 再打开**。配置成功后：

- Fork 与终端 `git`、IDEA、Cursor **共用** `E:\Config\git\.gitconfig`
- 可删除 `%LOCALAPPDATA%\Fork\gitInstance\` 释放 C 盘（约 400MB）；若 Fork 更新后又拉回，改回 Custom Git 后再删即可

#### 首次使用建议

1. **File → Open Repository** → 选 `E:\Workspace\Personal\` 或具体项目目录  
2. **File → Clone** 时 **Directory** 也指向 `E:\Workspace\...`  
3. 日常写代码仍用 **Cursor**；提交、分支、rebase、看历史用 **Fork**

| 场景 | 工具 |
|------|------|
| 编码 + AI 辅助 | Cursor |
| Java / 后端工程 | IDEA |
| 可视化 Git 操作 | Fork |
| 脚本化 / CI 同款命令 | 终端 `git` |

#### 验证

**普通 PowerShell**（`PS E:\Workspace\...>`，不要用 `C:\WINDOWS\system32` 管理员窗口）：

```powershell
Test-Path "$env:LOCALAPPDATA\Fork\Fork.exe"
git --version
git config --global --get user.name
```

在 Fork 内：**Preferences → Git**，确认 Git Instance 为 `D:\Portable\VCS\Git\cmd\git.exe`；打开 `E:\Workspace` 下任意已有仓库，能正常显示提交历史即可。

#### 常见问题

| 现象 | 处理 |
|------|------|
| C 盘出现 `...\Fork\gitInstance\` 且很大 | Preferences 改 Custom Git；退出 Fork 后删 `gitInstance` 文件夹 |
| Fork 打不开 / 无响应 | 杀毒软件可能拦截 `%LOCALAPPDATA%\Fork`；加白名单后重装 |
| 与 Cursor 内置 Git 冲突？ | 不冲突；各用各的界面，底层同一套 `git.exe` 与 `.gitconfig` |
| `error launching git: filename too long` | **设置 → 系统 → 关于 → 高级系统设置** 勾选启用 Win32 长路径；并确认用 Custom Git |

---



## 五、JDK 21 + JDK 17



### 下载

推荐 **Eclipse Temurin（Adoptium）**：[https://adoptium.net/zh-CN/temurin/releases](https://adoptium.net/zh-CN/temurin/releases)


| 版本     | 用途       |
| ------ | -------- |
| JDK 21 | 新项目默认    |
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


| 变量名          | 变量值                           |
| ------------ | ----------------------------- |
| `JAVA_HOME`  | `E:\Envs\Java\current`        |
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

[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

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


| 变量名          | 变量值                                 |
| ------------ | ----------------------------------- |
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

[https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/)


| 版本        | 适用        |
| --------- | --------- |
| Ultimate  | 商业 / 全功能  |
| Community | 个人学习 / 开源 |




### 安装路径

```
D:\Apps\JetBrains\IntelliJ IDEA 2025.x
```



### 安装选项


| 选项              | 建议  |
| --------------- | --- |
| Add bin to PATH | 不勾  |
| 右键「用 IDEA 打开」   | 建议勾 |




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


| 变量名                | 变量值               |
| ------------------ | ----------------- |
| `GRADLE_USER_HOME` | `E:\Cache\Gradle` |


---



## 八、Node.js（nvm-windows）



### 前置：卸载直接安装的 Node

若曾用安装包装过 Node.js，先在「设置 → 应用」中卸载，避免与 nvm 冲突。

### 下载

[https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)

下载 `nvm-setup.exe`，保存到 `D:\Packages\2026\Dev\`

### 安装路径


| 项          | 路径                    |
| ---------- | --------------------- |
| nvm 安装目录   | `E:\Envs\Node\nvm`    |
| symlink 目录 | `E:\Envs\Node\nodejs` |


> nvm-windows 惯例使用 `nodejs` 作为软链目录名（类似默认的 `C:\Program Files\nodejs`）。  
> `E:\Envs\Node\nodejs` 应为空目录或不存在，由安装器创建。



### 用户环境变量


| 变量名           | 变量值                   |
| ------------- | --------------------- |
| `NVM_HOME`    | `E:\Envs\Node\nvm`    |
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


| 变量名                | 变量值            |
| ------------------ | -------------- |
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


| 变量名         | 变量值             |
| ----------- | --------------- |
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

[https://www.7-zip.org/](https://www.7-zip.org/) → **64-bit Windows x64**

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

[https://www.voidtools.com/zh-cn/downloads/](https://www.voidtools.com/zh-cn/downloads/)

推荐 **安装版（x64）**，路径：

```
D:\Apps\Utilities\Everything
```



#### 安装选项建议


| 选项      | 建议  |
| ------- | --- |
| 开机启动    | 建议勾 |
| 安装为服务   | 按需  |
| 集成资源管理器 | 按需  |




#### 环境变量

Everything **无需配置环境变量**，开始菜单或托盘启动即可。

#### 首次设置建议

**工具 → 选项**：


| 设置           | 建议         |
| ------------ | ---------- |
| 索引 → NTFS    | 勾选 C/D/E 盘 |
| 随 Windows 启动 | 勾          |


---



## 十一、MySQL 8.0



### 路径规划


| 用途     | 路径                                                     |
| ------ | ------------------------------------------------------ |
| 程序     | `E:\Services\MySQL\MySQL Server 8.0`                   |
| 数据     | `E:\Data\mysql`                                        |
| 配置文件   | `E:\Data\mysql\my.ini`（安装器生成，与服务 `--defaults-file` 绑定） |
| 日志（建议） | `E:\Logs\services\mysql`                               |
| 安装包    | `D:\Packages\2026\Dev\`                                |




### 下载

[https://dev.mysql.com/downloads/installer/](https://dev.mysql.com/downloads/installer/)

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

**2. 若数据在** `E:\Data\mysql\Data\`**，将里面所有文件移到** `E:\Data\mysql\`（不要覆盖 `my.ini`），删除空的 `Data` 文件夹。

**3. 编辑** `E:\Data\mysql\my.ini`**，修改关键项：**

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


| 变量名          | 变量值                                  |
| ------------ | ------------------------------------ |
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


| 项                        | 值                                     |
| ------------------------ | ------------------------------------- |
| `@@datadir`              | `E:\Data\mysql\`                      |
| `@@basedir`              | `E:\Services\MySQL\MySQL Server 8.0\` |
| `@@character_set_server` | `utf8mb4`                             |
| `@@port`                 | `3306`                                |




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


| 用途  | 路径                       |
| --- | ------------------------ |
| 程序  | `E:\Services\Redis`      |
| 数据  | `E:\Data\redis`          |
| 日志  | `E:\Logs\services\redis` |
| 安装包 | `D:\Packages\2026\Dev\`  |




### 下载

Windows 推荐使用社区维护版（稳定、免费）：

[https://github.com/tporadowski/redis/releases](https://github.com/tporadowski/redis/releases)

下载最新 `.zip`，例如 `Redis-x64-5.0.14.1.zip`

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


| 变量名          | 变量值                 |
| ------------ | ------------------- |
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


| 项        | 值           |
| -------- | ----------- |
| Host     | `127.0.0.1` |
| Port     | `6379`      |
| Password | 留空（默认无密码）   |




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


| 用途               | 路径                                                          |
| ---------------- | ----------------------------------------------------------- |
| 程序               | `C:\Program Files\Docker\Docker` 或 `D:\Apps\Docker`（安装器若允许） |
| 镜像/容器数据（WSL 虚拟盘） | `E:\Containers\Docker\wsl`                                  |
| Compose 项目       | `E:\Containers\Docker\compose`                              |
| 安装包              | `D:\Packages\2026\Dev\`                                     |


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

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

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


| 项               | 建议     |
| --------------- | ------ |
| CPUs            | 4      |
| Memory          | 4 GB   |
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


| 宿主机服务 | 容器内地址                       |
| ----- | --------------------------- |
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


| 用途  | 路径                                            |
| --- | --------------------------------------------- |
| 程序  | `D:\Apps\Database\Navicat Premium 17`（版本号按实际） |
| 安装包 | `D:\Packages\2026\Dev\`                       |


> Navicat 配置和数据量小，装 D 盘即可；连接本地 MySQL/Redis 走 localhost。



### 下载

[https://www.navicat.com.cn/download/navicat-premium](https://www.navicat.com.cn/download/navicat-premium)

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


| 项   | 值                         |
| --- | ------------------------- |
| 连接名 | `Local MySQL`             |
| 主机  | `localhost` 或 `127.0.0.1` |
| 端口  | `3306`                    |
| 用户名 | `dev`（或 `root`）           |
| 密码  | 安装 MySQL 时设置的密码           |


点 **测试连接** → **确定**。

### 连接本地 Redis（Premium 版）

**连接 → Redis**


| 项   | 值             |
| --- | ------------- |
| 连接名 | `Local Redis` |
| 主机  | `127.0.0.1`   |
| 端口  | `6379`        |
| 密码  | 留空（默认无密码）     |




### 与 IDEA Database 的分工


| 工具            | 适合                  |
| ------------- | ------------------- |
| IDEA Database | 项目内快速查表、写 SQL       |
| Navicat       | 日常管理、导入导出、多库切换、数据对比 |


---



## 十五、Android Studio

> **Flutter 开发的前置依赖**：先装 Android Studio 和 Android SDK，再装 Flutter。



### 路径规划


| 用途               | 路径                                 |
| ---------------- | ---------------------------------- |
| 程序               | `D:\Apps\JetBrains\Android Studio` |
| Android SDK      | `E:\SDK\Android\sdk`               |
| NDK（按需）          | `E:\SDK\Android\ndk\版本号`           |
| AVD / `.android` | `E:\SDK\Android\home`              |
| Gradle 缓存        | `E:\Cache\Gradle`（已有）              |
| IDE 缓存           | `E:\Cache\AndroidStudio`           |
| 安装包              | `D:\Packages\2026\Dev\`            |




### 下载

[https://developer.android.com/studio](https://developer.android.com/studio)

保存到 `D:\Packages\2026\Dev\`

### 安装路径

```
D:\Apps\JetBrains\Android Studio
```

安装选项默认即可。

### 首次启动：SDK 路径（重要）

**Setup Wizard → SDK Components Setup** 或 **Settings → Languages & Frameworks → Android SDK**：


| 项                    | 值                    |
| -------------------- | -------------------- |
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


| 组件                  | 说明  |
| ------------------- | --- |
| Android 14 (API 34) | 较新  |
| Android 13 (API 33) | 常用  |


**SDK → SDK Tools**：


| 组件                             | 说明              |
| ------------------------------ | --------------- |
| Android SDK Build-Tools        | 必装              |
| Android SDK Platform-Tools     | 必装（含 adb）       |
| Android Emulator               | 模拟器             |
| Android SDK Command-line Tools | 必装              |
| NDK (Side by side)             | Flutter/原生需要时再装 |




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


| 项                | 值                 |
| ---------------- | ----------------- |
| Gradle user home | `E:\Cache\Gradle` |
| Gradle JDK       | JDK 17 或 JDK 21   |




### 用户环境变量


| 变量名                | 变量值                       |
| ------------------ | ------------------------- |
| `ANDROID_HOME`     | `E:\SDK\Android\sdk`      |
| `ANDROID_SDK_ROOT` | `E:\SDK\Android\sdk`      |
| `ANDROID_SDK_HOME` | `E:\SDK\Android\home`     |
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


| 用途               | 路径                                    |
| ---------------- | ------------------------------------- |
| Flutter SDK      | `E:\SDK\Flutter\flutter`              |
| Dart/Flutter 包缓存 | `E:\Cache\pub`                        |
| 项目源码             | `E:\Workspace\Personal\` 或 `Company\` |




### 下载 Flutter SDK

**方式 A：Git 克隆（推荐）**

```powershell
cd E:\SDK\Flutter
git clone https://github.com/flutter/flutter.git -b stable
```

**方式 B：zip 解压**

[https://docs.flutter.dev/get-started/install/windows](https://docs.flutter.dev/get-started/install/windows)

解压到 `E:\SDK\Flutter\flutter`（确保 `flutter\bin\flutter.bat` 存在）

### 用户环境变量


| 变量名                        | 变量值                                      |
| -------------------------- | ---------------------------------------- |
| `PUB_CACHE`                | `E:\Cache\pub`                           |
| `FLUTTER_STORAGE_BASE_URL` | `https://storage.flutter-io.cn`（国内镜像，可选） |
| `PUB_HOSTED_URL`           | `https://pub.flutter-io.cn`（国内镜像，可选）     |


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


| 检查项               | 期望               |
| ----------------- | ---------------- |
| Flutter           | ✅ stable channel |
| Android toolchain | ✅                |
| Android Studio    | ✅                |
| Chrome            | ✅ 或 ⚠️（Web 开发需要） |
| Network resources | ✅ 或 ⚠️（国内可能需镜像）  |




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


| IDE            | 插件                                         |
| -------------- | ------------------------------------------ |
| Android Studio | 内置 Flutter / Dart 支持，**Plugins → Flutter** |
| IntelliJ IDEA  | **Plugins → Flutter** + **Dart**           |


IDEA 中 **Settings → Languages & Frameworks → Flutter**：


| 项                | 值                        |
| ---------------- | ------------------------ |
| Flutter SDK path | `E:\SDK\Flutter\flutter` |


---



## 十七、环境变量汇总

> 全部配置在 **用户变量** 和 **用户 Path**，**不要修改系统变量**。



### 用户变量


| 变量名                        | 变量值                                    |
| -------------------------- | -------------------------------------- |
| `GIT_CONFIG_GLOBAL`        | `E:\Config\git\.gitconfig`             |
| `TEMP`                     | `E:\Temp`                              |
| `TMP`                      | `E:\Temp`                              |
| `JAVA_HOME`                | `E:\Envs\Java\current`                 |
| `MAVEN_HOME`               | `E:\Envs\Maven\apache-maven-3.9.15`    |
| `GRADLE_USER_HOME`         | `E:\Cache\Gradle`                      |
| `NVM_HOME`                 | `E:\Envs\Node\nvm`                     |
| `NVM_SYMLINK`              | `E:\Envs\Node\nodejs`                  |
| `npm_config_cache`         | `E:\Cache\npm`                         |
| `PNPM_HOME`                | `E:\Cache\pnpm`                        |
| `MYSQL_HOME`               | `E:\Services\MySQL\MySQL Server 8.0`   |
| `REDIS_HOME`               | `E:\Services\Redis`                    |
| `ANDROID_HOME`             | `E:\SDK\Android\sdk`                   |
| `ANDROID_SDK_ROOT`         | `E:\SDK\Android\sdk`                   |
| `ANDROID_SDK_HOME`         | `E:\SDK\Android\home`                  |
| `ANDROID_AVD_HOME`         | `E:\SDK\Android\home\avd`              |
| `PUB_CACHE`                | `E:\Cache\pub`                         |
| `FLUTTER_STORAGE_BASE_URL` | `https://storage.flutter-io.cn`（国内可选）  |
| `PUB_HOSTED_URL`           | `https://pub.flutter-io.cn`（国内可选）      |
| `OLLAMA_MODELS`            | `E:\AI\Models\ollama`                  |
| `CLAUDE_CONFIG_DIR`        | `E:\Data\claude`（装 Claude Code 前设置，可选） |
| `CODEX_HOME`               | `E:\Data\codex`（装 Codex 前设置，可选）        |
| `PIP_CACHE_DIR`            | `E:\Cache\pip`（装 Python 前设置）            |




### 用户 Path（完整参考）

```
D:\Portable\VCS\Git\cmd
D:\Portable\bin
D:\Apps\Utilities\7-Zip
%USERPROFILE%\.local\bin
%LOCALAPPDATA%\Programs\OpenAI\Codex\bin
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
E:\Envs\Python\Python313
E:\Envs\Python\Python313\Scripts
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


| 位置                         | 配置项                       | 值                                   |
| -------------------------- | ------------------------- | ----------------------------------- |
| Project Structure → SDKs   | JDK 21                    | `E:\Envs\Java\jdk-21.0.x`           |
| Project Structure → SDKs   | JDK 17                    | `E:\Envs\Java\jdk-17.0.x`           |
| Settings → Maven           | Maven home                | `E:\Envs\Maven\apache-maven-3.9.15` |
| Settings → Maven           | User settings             | `E:\Config\maven\settings.xml`      |
| Settings → Maven           | Local repository          | `E:\Cache\Maven`                    |
| Settings → Gradle          | Gradle user home          | `E:\Cache\Gradle`                   |
| Settings → System Settings | Default project directory | `E:\Workspace`                      |
| Settings → Git             | Git executable            | `D:\Portable\VCS\Git\cmd\git.exe`   |
| Settings → Node.js         | Node interpreter          | `E:\Envs\Node\nodejs\node.exe`      |
| Settings → Flutter         | Flutter SDK path          | `E:\SDK\Flutter\flutter`            |
| Settings → Dart            | Dart SDK path             | 随 Flutter 自动识别                      |
| Database                   | MySQL 数据源                 | `localhost:3306`，用户 `dev`           |
| Database                   | Redis                     | `127.0.0.1:6379`                    |
| Settings → Docker          | Docker                    | Docker for Windows（自动识别）            |
| Plugins                    | Lombok                    | 安装并启用 Annotation Processing         |
| Plugins                    | Flutter + Dart            | Flutter 开发时安装                       |


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

**原因**：`TEMP` 值含前导空格或换行（如  `E:\Temp` 或 `\nE:\Temp`）。

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

**处理**：

1. WinGet 安装后一般已自动加 Path，执行 `where.exe claude` 查看
2. 若仍没有，WinGet 路径示例：`%LOCALAPPDATA%\Microsoft\WinGet\Packages\Anthropic.ClaudeCode_...\claude.exe`
3. 原生脚本安装则追加 `%USERPROFILE%\.local\bin`
4. **新开终端**再试



### 20. Claude Code 安装报 `ECONNREFUSED` / `Failed to fetch version from downloads.claude.ai`

**原因**：国内网络常无法直连 `downloads.claude.ai`。

**处理**（按顺序试）：

1. **WinGet**（国内优先推荐）：

```powershell
winget install Anthropic.ClaudeCode --accept-source-agreements --accept-package-agreements
```

首次可能提示同意 `msstore` 源协议，输入 `Y`。

1. **开代理后重跑官方脚本**：

```powershell
$env:HTTPS_PROXY = "http://127.0.0.1:7890"   # 按你的代理端口改
$env:HTTP_PROXY  = "http://127.0.0.1:7890"
irm https://claude.ai/install.ps1 | iex
```

1. **npm 备选**（需 Node.js 22+）：

```powershell
nvm use 22
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

诊断连通性：

```powershell
curl.exe -sI https://downloads.claude.ai/claude-code-releases/latest
```



### 21. Codex 装错包

**处理**：确认安装的是 `npm install -g @openai/codex`，不是 `codex`。

### 22. Codex 首次启动逼你登录 ChatGPT，但我想用 API Key

**处理**：按 `Ctrl + C` 退出 → 先在 [CC Switch](#二十二cc-switch) 里为 **Codex** 添加并启用 API Key 供应商 → **新开终端**再 `codex`。若仍弹出登录界面，选 **3. Provide your own API key**。

### 23. CC Switch 切换后 API 仍不通

**处理**：Codex 切换后新开终端；检查 API Key、Base URL、模型名；Claude Code 可在 CC Switch 里重新点「使用」。

### 24. `codex` 命令找不到，但 `codex.exe` 存在

**原因**：原生安装未自动加 Path。

**处理**：用户 Path 追加 `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`，新开终端。验证：`where.exe codex`。

### 25. Codex 报找不到 `codex-windows-sandbox-setup.exe`

**处理**：`config.toml` 设 `sandbox = "unelevated"`，或首次沙箱菜单选 **2. Use non-admin sandbox**。

### 26. CC Switch 配置在 C 盘、`E:\Data\codex` 只有沙箱配置

**原因**：CC Switch 不写 `CODEX_HOME` 目录。

**处理**：对 `%USERPROFILE%\.codex` 和 `%USERPROFILE%\.claude` 做 Junction 到 `E:\Data\codex` / `E:\Data\claude`（见第二十一章方式 C）。

### 27. `claude doctor` 提示 `.claude.json not found`

**处理**：从 `E:\Data\claude\backups\` 选最新 `.claude.json.backup.`* 复制为 `E:\Data\claude\.claude.json`。Remote Control ‼ 对 API Key 用户可忽略。

### 28. `where python` 指向 `WindowsApps\python.exe`

**原因**：Windows「应用执行别名」占位，不是真 Python。

**处理**：

1. **设置 → 应用 → 高级应用设置 → 应用执行别名** → 关闭 `python.exe`、`python3.exe`
2. 用户 Path **最前面**加（版本号按实际目录改）：
   ```text
   E:\Envs\Python\Python313
   E:\Envs\Python\Python313\Scripts
   ```
3. 新开终端验证：`where.exe python` 应指向 `E:\Envs\Python\...`

### 29. `pip cache dir` 不是 `E:\Cache\pip`

**处理**：安装前设用户变量 `PIP_CACHE_DIR=E:\Cache\pip`；或 `python -m pip config set global.cache-dir E:\Cache\pip`。**新开终端**后再 `pip install`。

---



## 二十、Cursor

> Cursor 是基于 VS Code 的 AI 代码编辑器，与 IDEA 互补：后端 Java 用 IDEA，全栈/前端/AI 辅助编码用 Cursor。



### 路径规划


| 用途            | C 盘默认路径                 | E 盘目标路径                          |
| ------------- | ----------------------- | -------------------------------- |
| 程序            | —                       | `D:\Apps\Cursor`（安装器 Browse 改路径） |
| 应用数据（Roaming） | `%APPDATA%\Cursor`      | `E:\Cache\Cursor\Roaming`        |
| 本地缓存（Local）   | `%LOCALAPPDATA%\Cursor` | `E:\Cache\Cursor\Local`          |
| 用户配置（可选）      | `%USERPROFILE%\.cursor` | `E:\Config\ide\cursor`           |
| 安装包           | —                       | `D:\Packages\2026\Dev\`          |


> Cursor **没有**像 IDEA 那样的官方缓存路径配置项。迁出 C 盘需用 **NTFS Junction（目录联接）**：C 盘保留原路径，实际数据写到 E 盘。这是目前最稳定的方案。



### 下载

[https://cursor.com/download](https://cursor.com/download)

下载 **Cursor User Setup x64**（用户安装版，无需管理员）

### 安装

1. 双击 `CursorUserSetup-x64.exe`
2. **Destination Location** 改为：

```
D:\Apps\Cursor
```

1. 建议勾选：
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

**可选：**`.cursor` **用户配置一并迁出**

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


| 项    | 说明                                                                    |
| ---- | --------------------------------------------------------------------- |
| 权限   | 创建 Junction 需**管理员 PowerShell**                                       |
| 更新   | Cursor 更新前建议完全退出，避免写入冲突                                               |
| 清理缓存 | 只删 E 盘下的 `Cache`、`CachedData`；**不要删**整个 `Roaming` 目录或 C 盘 Junction 本身 |
| 索引重建 | 迁移后首次打开大项目，索引可能重建，属正常现象                                               |




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
> **前置**：Git（已装）；**Codex 若用 npm 安装**还需 Node.js 22+（`nvm use 22`）。  
> **Claude Code 原生安装不需要 Node.js。**



### 推荐流程（核对版）

```
①（可选）设用户环境变量：CLAUDE_CONFIG_DIR、CODEX_HOME → E:\Data\
② 安装 Claude Code（国内优先 WinGet）
③ 安装 Codex CLI（原生 install.ps1）→ 手动加 Path（见下文）
④ 验证：claude --version、codex --version（不必先登录）
⑤ 安装 CC Switch（x64 选 Windows.msi，非 arm64）
⑥ 分别为 Claude Code / Codex 添加 API Key 供应商并「启用」
⑦ 数据迁 E 盘 + Junction（CC Switch 写 C 盘用户目录，必须做）
⑧ 新开普通 PowerShell → cd E:\Workspace\... → claude / codex 验证
```


| 步骤          | 做什么                                  | 不要做什么                                     |
| ----------- | ------------------------------------ | ----------------------------------------- |
| 装 CLI       | 只装程序，确认命令可用                          | 不要在 Codex 里选 ChatGPT 登录（若打算用 API Key）     |
| 装 CC Switch | 图形界面配 Key、Base URL、模型                | CC Switch **不代替**安装 CLI                   |
| 首次使用        | CC Switch 启用供应商后再 `codex` / `claude` | 不要手改 `~/.codex/config.toml`（交给 CC Switch） |




### 路径规划


| 工具                  | 程序位置（常见）                                                  | CC Switch 写入（C 盘）        | E 盘真实数据          |
| ------------------- | --------------------------------------------------------- | ------------------------ | ---------------- |
| Claude Code（WinGet） | `%LOCALAPPDATA%\Microsoft\WinGet\Packages\...\claude.exe` | `%USERPROFILE%\.claude\` | `E:\Data\claude` |
| Claude Code（原生脚本）   | `%USERPROFILE%\.local\bin\claude.exe`                     | 同上                       | 同上               |
| Codex（原生安装）         | `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin\codex.exe`      | `%USERPROFILE%\.codex\`  | `E:\Data\codex`  |
| Codex（npm）          | `E:\Envs\Node\npm-global\codex.cmd`                       | 同上                       | 同上               |


> **重要**：`CLAUDE_CONFIG_DIR` / `CODEX_HOME` 只影响 CLI 自身读写的部分配置；**CC Switch 始终写入** `%USERPROFILE%\.claude\` **和** `%USERPROFILE%\.codex\`。  
> 要统一到 E 盘，必须再做 **Junction**（见下文「数据迁出 C 盘 → 方式 C」），与 Cursor 同理。



### 账号与认证方式


| 工具          | 认证方式                               | 本文推荐                    |
| ----------- | ---------------------------------- | ----------------------- |
| Claude Code | Anthropic 账号 / Claude 订阅 / API Key | **API Key + CC Switch** |
| Codex       | ChatGPT 订阅登录 / API Key             | **API Key + CC Switch** |


> 有 ChatGPT 订阅也可在 Codex 里选「Sign in with ChatGPT」；本文按 **API Key 统一管理** 写法。

---



### Claude Code 安装



#### 国内网络：优先 WinGet

国内直连 `downloads.claude.ai` 常失败（`ECONNREFUSED`），**优先用 WinGet**：

```powershell
winget install Anthropic.ClaudeCode --accept-source-agreements --accept-package-agreements
```

- 首次可能提示同意 `msstore` 源协议 → 输入 `Y`
- 包 ID 是 `Anthropic.ClaudeCode`（CLI），不是桌面 GUI 包
- WinGet **不自动更新**，升级：`winget upgrade Anthropic.ClaudeCode`



#### 官方脚本（网络畅通时）

**普通 PowerShell**：

```powershell
irm https://claude.ai/install.ps1 | iex
```

**CMD 用户**：

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

**有代理时**（先设代理再执行官方脚本）：

```powershell
$env:HTTPS_PROXY = "http://127.0.0.1:7890"   # 按实际端口改
$env:HTTP_PROXY  = "http://127.0.0.1:7890"
irm https://claude.ai/install.ps1 | iex
```



#### npm 安装（备选，需 Node.js 18+）

```powershell
nvm use 22
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

> 若曾用 npm 装过，建议先迁到原生：`claude install`，再 `npm uninstall -g @anthropic-ai/claude-code`，避免两套二进制冲突。



#### 安装失败诊断

```powershell
curl.exe -sI https://downloads.claude.ai/claude-code-releases/latest
```

无 `HTTP/2 200` / `HTTP/1.1 200` → 网络被挡，用 WinGet 或代理。

### Claude Code 用户 Path


| 安装方式             | Path 是否自动添加 | 手动追加                       |
| ---------------- | ----------- | -------------------------- |
| **WinGet**（国内推荐） | 通常已自动加入     | 一般无需                       |
| 原生 `irm` 脚本      | 通常自动        | `%USERPROFILE%\.local\bin` |


验证：

```powershell
claude --version
where.exe claude
# WinGet 常见：...\WinGet\Packages\Anthropic.ClaudeCode_...\claude.exe
```



### Claude Code 验证

**关掉终端，重新打开**：

```powershell
claude --version
claude doctor    # 可选
```

> 此阶段只需确认**命令可用**；API Key 在 [CC Switch](#二十二cc-switch) 配置后再正式使用。  
> WinGet 安装时 `where claude` 指向 WinGet 目录是**正常现象**，不必强求 `.local\bin`。  
> `claude doctor` 里 **Remote Control ‼** 可忽略（API Key 用户无需登录 claude.ai）。  
> 若报 `.claude.json not found` 且有 backup，恢复：  
> `Copy-Item "E:\Data\claude\backups\.claude.json.backup.*" "E:\Data\claude\.claude.json"`（选最新一个）。  
> Windows 上建议已装 **Git for Windows**，Claude Code 才能用 Bash 工具。

---



### Codex CLI 安装

**方式 A：原生安装（推荐，无需 npm）**

**普通 PowerShell**：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

**方式 B：npm 安装**

前置：Node.js 22+，且 npm 全局目录已配到 E 盘（见 [npm 章节](#九npm-与-pnpm)）：

```powershell
nvm use 22
npm install -g @openai/codex
```

> 包名必须是 `@openai/codex`，不是 `codex`（后者是无关旧包）。



### Codex 用户 Path（原生安装常需手动添加）

原生安装后 **WinGet/安装器不一定自动加 Path**，需确认：

```powershell
Test-Path "$env:LOCALAPPDATA\Programs\OpenAI\Codex\bin\codex.exe"
```

若为 `True`，将下面路径加入 **用户 Path**：

```text
%LOCALAPPDATA%\Programs\OpenAI\Codex\bin
```

PowerShell 一次性添加：

```powershell
$codexBin = "$env:LOCALAPPDATA\Programs\OpenAI\Codex\bin"
$p = [Environment]::GetEnvironmentVariable("Path", "User")
if ($p -notlike "*$codexBin*") {
    [Environment]::SetEnvironmentVariable("Path", "$p;$codexBin", "User")
}
```

**新开终端**后验证：`codex --version`、`where.exe codex`。

### Codex 验证（仅确认安装）

**新开 PowerShell**：

```powershell
codex --version
```

> 只需确认命令可用。**不要在此完成 ChatGPT 登录**（若打算用 API Key + CC Switch）。  
> 若已误进入登录界面，按 `Ctrl + C` 退出，先去 CC Switch 配供应商。



### Codex 配置（Windows 沙箱）

配置文件路径（设了 `CODEX_HOME` 时用 E 盘路径）：

```text
E:\Data\codex\config.toml          # 已设 CODEX_HOME
%USERPROFILE%\.codex\config.toml   # 默认
```

首次运行前创建或编辑，**国内实测建议直接用 unelevated**（elevated 可能报找不到 `codex-windows-sandbox-setup.exe`）：

```toml
[windows]
sandbox = "unelevated"
# sandbox = "elevated"   # 更强隔离；若 setup.exe 报错再改回 unelevated
```


| 模式           | 说明                                 |
| ------------ | ---------------------------------- |
| `unelevated` | **推荐默认**。个人开发机够用；无需管理员配沙箱          |
| `elevated`   | 更强隔离；首次弹 UAC；部分版本有 helper 找不到的 bug |


首次启动若出现沙箱菜单：

```text
1. Set up default sandbox (Administrator)   ← 可能报找不到 setup.exe
2. Use non-admin sandbox                    ← 选这个
3. Quit
```

选 **2**，或与 `config.toml` 中 `unelevated` 保持一致。

> 项目目录建议放在 `E:\Workspace\...`，在 PowerShell 里 `cd` 进去后运行 `codex`。  
> API Key、Base URL、模型名由 **CC Switch 写入**，不要与沙箱配置混在一起手改。



### 数据与缓存迁出 C 盘（环境变量）

Claude Code 和 Codex 运行时会写入会话、工具输出、图片/粘贴缓存等。与 Cursor 不同，两者都支持**官方环境变量**指定数据根目录。

#### 会增长的内容


| 工具          | 典型目录                                  | 内容        |
| ----------- | ------------------------------------- | --------- |
| Claude Code | `.claude\projects\`                   | 对话记录、工具结果 |
| Claude Code | `.claude\file-history\`               | 文件修改前快照   |
| Claude Code | `.claude\paste-cache\`、`image-cache\` | 粘贴/图片缓存   |
| Codex       | `.codex\sessions\`                    | 会话记录      |
| Codex       | `.codex\cache\`、`attachments\`        | 缓存与附件     |
| Codex       | `.codex\sqlite\`、`logs\`              | 本地数据库与日志  |


> C 盘不紧、轻度使用可先不迁；C 盘紧张或打算长期重度使用，建议**安装前**设好环境变量。



#### 方式 A：安装前设置（推荐）

用户环境变量：


| 变量名                 | 变量值              |
| ------------------- | ---------------- |
| `CLAUDE_CONFIG_DIR` | `E:\Data\claude` |
| `CODEX_HOME`        | `E:\Data\codex`  |


```powershell
New-Item -ItemType Directory -Path "E:\Data\claude" -Force
New-Item -ItemType Directory -Path "E:\Data\codex" -Force
```

设好后安装 CLI；但 **CC Switch 仍写 C 盘用户目录**，务必再做「方式 C」Junction。

#### 方式 B：已使用后迁移（robocopy）

1. 完全退出 `claude` / `codex`、CC Switch
2. 设好上述用户环境变量
3. 迁移已有数据：

```powershell
# Claude Code
if (Test-Path "$env:USERPROFILE\.claude") {
    robocopy "$env:USERPROFILE\.claude" "E:\Data\claude" /E /MOVE /R:1 /W:1
}

# Codex
if (Test-Path "$env:USERPROFILE\.codex") {
    robocopy "$env:USERPROFILE\.codex" "E:\Data\codex" /E /MOVE /R:1 /W:1
}
```



#### 方式 C：Junction 统一到 E 盘（**配合 CC Switch 必做**）

CC Switch 写入 `%USERPROFILE%\.claude\settings.json` 和 `%USERPROFILE%\.codex\auth.json`，与 `CLAUDE_CONFIG_DIR` / `CODEX_HOME` 可能分裂。**推荐做法**：

```powershell
# 管理员 PowerShell；先退出 claude / codex / CC Switch

# Claude Code
robocopy "$env:USERPROFILE\.claude" "E:\Data\claude" /E /MOVE /R:1 /W:1
Remove-Item "$env:USERPROFILE\.claude" -Recurse -Force -ErrorAction SilentlyContinue
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude" -Target "E:\Data\claude"

# Codex
robocopy "$env:USERPROFILE\.codex" "E:\Data\codex" /E /MOVE /R:1 /W:1
Remove-Item "$env:USERPROFILE\.codex" -Recurse -Force -ErrorAction SilentlyContinue
New-Item -ItemType Junction -Path "$env:USERPROFILE\.codex" -Target "E:\Data\codex"
```

验证（应显示 `Junction` 且 `Target` 指向 E 盘）：

```powershell
Get-Item "$env:USERPROFILE\.claude" | Select-Object LinkType, Target
Get-Item "$env:USERPROFILE\.codex" | Select-Object LinkType, Target
Test-Path "E:\Data\claude\settings.json"    # CC Switch 写的 Claude 配置
Test-Path "E:\Data\codex\config.toml"       # CC Switch 写的 Codex 配置
```

1. 新开终端，运行 `claude` / `codex` 验证



#### 清理缓存（保留配置和登录）

先退出 CLI，再删缓存类目录（会丢失部分会话/附件，慎用）：

```text
E:\Data\claude\paste-cache\
E:\Data\claude\image-cache\
E:\Data\codex\cache\
E:\Data\codex\attachments\
```

> **不要**删除整个 `claude` / `codex` 目录，否则会丢登录信息和配置。



### 常见问题


| 现象                                           | 处理                                                        |
| -------------------------------------------- | --------------------------------------------------------- |
| Claude 安装 `ECONNREFUSED downloads.claude.ai` | 国内优先 **WinGet**；或开代理；或 npm + npmmirror                    |
| WinGet 提示同意 msstore 协议                       | 输入 `Y` 继续                                                 |
| `claude` 找不到                                 | WinGet 装后一般自动有 Path；否则查 `where.exe claude`                |
| `codex` 找不到                                  | 用户 Path 加 `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`，新开终端 |
| `codex` 找不到（npm 装）                           | Path 含 `E:\Envs\Node\npm-global`                          |
| `找不到 codex-windows-sandbox-setup.exe`        | `config.toml` 改 `sandbox = "unelevated"` 或沙箱菜单选 **2**     |
| Codex 弹出 ChatGPT 登录                          | `Ctrl+C` → CC Switch 启用供应商 → 新开终端；或选 **3. API key**       |
| CC Switch 写了 C 盘、E 盘没配置                      | 做 **Junction**（方式 C）                                      |
| `.claude.json not found`                     | 从 `E:\Data\claude\backups\` 最新 backup 复制恢复                |
| `claude doctor` Remote Control ‼             | API Key 用户可忽略                                             |
| 换 API 供应商                                    | CC Switch 切换；Codex 需新开终端                                  |
| C 盘被 CLI 数据占满                                | `CLAUDE_CONFIG_DIR` / `CODEX_HOME` + Junction             |


---



### 三者分工与数据布局（汇总）


| 工具              | 程序                                         | 数据 Junction                                    |
| --------------- | ------------------------------------------ | ---------------------------------------------- |
| **Cursor**      | `D:\Apps\Cursor`                           | `%APPDATA%\Cursor` → `E:\Cache\Cursor\Roaming` |
| **Codex**       | `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin` | `%USERPROFILE%\.codex` → `E:\Data\codex`       |
| **Claude Code** | WinGet 或 `%USERPROFILE%\.local\bin`        | `%USERPROFILE%\.claude` → `E:\Data\claude`     |



| 工具              | 场景                          |
| --------------- | --------------------------- |
| **Cursor**      | 图形界面 AI 编码，日常写代码            |
| **Claude Code** | 终端里 Claude 驱动，适合 Agent 式改项目 |
| **Codex**       | 终端里 OpenAI 驱动，ChatGPT 生态    |


---



## 二十二、CC Switch



### 这是什么

**CC Switch** 是一款开源桌面工具，用来**可视化管理、一键切换** AI 编程 CLI 的 API 供应商配置。

官网：[https://ccswitch.io/](https://ccswitch.io/)  
GitHub：[https://github.com/farion1231/cc-switch](https://github.com/farion1231/cc-switch)


| 功能            | 说明                                      |
| ------------- | --------------------------------------- |
| 一键切换 Provider | 官方 API、国内镜像、第三方代理，点一下切换                 |
| 多工具统一管理       | Claude Code、Codex、Gemini CLI、OpenCode 等 |
| MCP 管理        | 可视化配置 MCP 服务器                           |
| 用量统计          | Token 消耗与费用（视供应商）                       |
| 配置备份          | 自动备份，防误操作                               |


**适合你如果**：需要在多个 API 供应商之间频繁切换，不想手改 `~/.claude/settings.json` 或 `~/.codex/config.toml`。

### 路径规划


| 用途           | 路径                                                        |
| ------------ | --------------------------------------------------------- |
| 程序（MSI 安装）   | `D:\Apps\Utilities\CC-Switch`                             |
| 程序（便携版）      | `D:\Portable\Sys\CC-Switch`                               |
| 自身数据         | `%USERPROFILE%\.cc-switch\`（SQLite，体积小；迁 E 盘可选）              |
| 写入 Claude 配置 | `%USERPROFILE%\.claude\`（Junction 后实际在 `E:\Data\claude\`） |
| 写入 Codex 配置  | `%USERPROFILE%\.codex\`（Junction 后实际在 `E:\Data\codex\`）   |


> **CC Switch 不读取** `CLAUDE_CONFIG_DIR` **/** `CODEX_HOME`，只写 `%USERPROFILE%\.claude\` 和 `%USERPROFILE%\.codex\`。  
> 配合 E 盘规划时，务必对这两个目录做 [Junction](#数据与缓存迁出-c-盘环境变量)（方式 C）。  
> CC Switch **自身**数据在 `%USERPROFILE%\.cc-switch\`（供应商数据库），体积通常很小；C 盘不紧可保留，也可在 CC Switch **设置 → 应用配置目录** 改为 `E:\Config\cc-switch`。



### 前置条件

1. **先手动装好** Claude Code、Codex 等 CLI（CC Switch **不负责安装** CLI，Windows 版已禁用一键安装）
2. `claude --version`、`codex --version` 已能跑通（**不必先完成登录**）



### 下载

[https://github.com/farion1231/cc-switch/releases](https://github.com/farion1231/cc-switch/releases)


| 文件                                      | 说明                         |
| --------------------------------------- | -------------------------- |
| `CC-Switch-vX.X.X-Windows.msi`          | **x64 电脑选这个**（Intel / AMD） |
| `CC-Switch-vX.X.X-Windows-arm64.msi`    | 仅 ARM 版 Windows 设备         |
| `CC-Switch-vX.X.X-Windows-Portable.zip` | 便携版                        |


> 在「设置 → 系统 → 关于」看 **系统类型**：`基于 x64 的处理器` → 选 **Windows.msi**，不要选 arm64。



### 安装

**MSI 版**：双击安装，路径可改为 `D:\Apps\Utilities\CC-Switch`

**便携版**：解压到 `D:\Portable\Sys\CC-Switch`，运行 `CC-Switch.exe`

### 推荐配置流程（API Key 用户）



#### 1. 配置 Claude Code 供应商

1. 打开 CC Switch → 顶部选 **Claude Code**
2. **添加供应商** → 填名称、API Key、Base URL（第三方代理必填）、模型
3. 保存 → 点 **使用 / Enable**

写入示例位置：`E:\Data\claude\settings.json`（或默认 `%USERPROFILE%\.claude\`）

#### 2. 配置 Codex 供应商

1. 顶部切换到 **Codex**（与 Claude Code **分开配置**）
2. **添加供应商** → 填：
  - **API Key**：`sk-...`
  - **Base URL**：官方 `https://api.openai.com/v1` 或供应商文档给的地址
  - **模型**：供应商支持的模型 ID
3. 保存 → 点 **使用 / Enable**

写入位置（Junction 后均在 E 盘）：


| 工具          | 文件              | 内容                                      |
| ----------- | --------------- | --------------------------------------- |
| Claude Code | `settings.json` | API Key、Base URL、模型                     |
| Codex       | `auth.json`     | API Key                                 |
| Codex       | `config.toml`   | `model_provider`、`base_url`、`model`、沙箱等 |




#### 3. 首次启动 Codex（配合 CC Switch）

```powershell
# 新开 PowerShell（CC Switch 启用供应商之后）
cd E:\Workspace\Sandbox
codex
```

若仍出现登录菜单：

```text
> 1. Sign in with ChatGPT
> 2. Sign in with Device Code
> 3. Provide your own API key
```

- **API Key 用户**：选 `3`，或 `Ctrl+C` 退出后确认 CC Switch 已启用供应商再试
- **不要选 1**（除非你有 ChatGPT 订阅且想走订阅额度）



#### 4. 首次启动 Claude Code

```powershell
cd E:\Workspace\Sandbox
claude
```

按 CC Switch 已写入的配置连接 API；若提示登录，检查 CC Switch 里 Claude Code 供应商是否已 **启用**。

### 使用注意


| 项            | 说明                                                    |
| ------------ | ----------------------------------------------------- |
| 切换供应商后       | **Codex 必须新开终端**；Claude Code 多数情况可热切换                 |
| 纯 API Key 计费 | **不要**开「保留官方 ChatGPT 登录」类选项（避免计费走 ChatGPT 订阅而非 API）   |
| 手改配置         | 交给 CC Switch，避免与 GUI 写入冲突                             |
| 第三方代理        | Base URL、模型名必须与供应商文档一致；Codex 需 **Responses API** 兼容端点 |




### 与 Claude Code / Codex 的关系

```
安装 Claude Code、Codex（只验证命令）
         ↓
安装 CC Switch → 分别为两者添加 API Key 供应商并「启用」
         ↓
新开终端 → cd E:\Workspace\... → claude / codex
```



### 验证

1. CC Switch 里 Claude Code、Codex 各启用一个供应商
2. **新开 PowerShell**：

```powershell
cd E:\Workspace\Sandbox
claude --version
codex --version
claude    # 发一条消息，确认 API 通
codex     # 发一条消息，确认 API 通
```

1. 切换供应商后，Codex 再 **新开终端** 验证



### 如何确认在用 CC Switch 的 API

**Claude Code**（Junction 后路径如下）：

```powershell
Get-Content "E:\Data\claude\settings.json"   # 应有 API 相关配置
```

**Codex**：

```powershell
Get-Content "E:\Data\codex\config.toml"        # model_provider = "custom"、base_url 为你的代理
Get-Content "E:\Data\codex\auth.json"        # 应有 OPENAI_API_KEY（勿外泄）
```


| 检查项               | 走 CC Switch API    | 走 ChatGPT 订阅                    |
| ----------------- | ------------------ | ------------------------------- |
| Codex `base_url`  | 第三方代理地址            | 官方或空                            |
| Codex `auth.json` | 有 `OPENAI_API_KEY` | `auth_mode: chatgpt`、Key 为 null |
| 供应商后台             | 有调用记录              | 走 OpenAI/ChatGPT 账单             |


---



## 二十三、本地 AI 模型（Ollama）

> 若需要**离线/本地**跑大模型（Llama、Qwen、DeepSeek 等），用 Ollama。与 Claude Code/Codex **不冲突**——本地模型通过 Ollama API 供其他工具调用。



### 路径规划

> **说明**：双击 `OllamaSetup.exe` **没有**路径选项，默认装到 `%LOCALAPPDATA%\Programs\Ollama`（C 盘用户目录）。  
> 要迁出 C 盘分两步：**程序**用安装器 `/DIR=` 参数；**模型**（占空间大头，几十～上百 GB）用 `OLLAMA_MODELS` 指到 E 盘。


| 用途 | 路径 |
|------|------|
| 程序（自定义安装） | `D:\Apps\AI\Ollama`（需命令行 `/DIR=`，见下文） |
| 程序（默认，未改时） | `%LOCALAPPDATA%\Programs\Ollama` |
| 模型文件（必迁 E 盘） | `E:\AI\Models\ollama`（`OLLAMA_MODELS`） |
| 日志/更新缓存 | `%LOCALAPPDATA%\Ollama`（体积小，可留 C） |
| 安装包 | `D:\Packages\2026\Dev\` |



### 下载

[https://ollama.com/download/windows](https://ollama.com/download/windows)

安装包保存到 `D:\Packages\2026\Dev\`



### 用户环境变量（拉模型前必设）

| 变量名 | 变量值 |
|--------|--------|
| `OLLAMA_MODELS` | `E:\AI\Models\ollama` |

> **在第一次 `ollama pull` 之前**设好，否则模型会先下到 `%USERPROFILE%\.ollama\models`（C 盘）。

先建目录：

```powershell
New-Item -ItemType Directory -Path "D:\Apps\AI\Ollama" -Force
New-Item -ItemType Directory -Path "E:\AI\Models\ollama" -Force
```

PowerShell 设用户变量：

```powershell
[Environment]::SetEnvironmentVariable('OLLAMA_MODELS', 'E:\AI\Models\ollama', 'User')
```

设完后**完全退出**托盘区 Ollama（右键 Quit），再重新打开；或**新开** PowerShell 验证：

```powershell
echo $env:OLLAMA_MODELS
# 期望：E:\AI\Models\ollama
```



### 安装（程序到 D 盘）

**不要**只双击安装包。在**普通 PowerShell** 中（路径按实际安装包名改）：

```powershell
cd D:\Packages\2026\Dev
.\OllamaSetup.exe /DIR="D:\Apps\AI\Ollama"
```

安装器会把程序写到 `D:\Apps\AI\Ollama`，并尝试把该目录加入用户 **Path**。

若已用默认方式装过（程序在 C 盘），可：

1. **设置 → 应用** 卸载 Ollama  
2. 确认已设 `OLLAMA_MODELS`（上节）  
3. 再用 `/DIR=` 重装到 D 盘  

> 若 `ollama` 命令找不到，手动把 `D:\Apps\AI\Ollama` 加到用户 Path，并**新开终端**。



### 若程序只能留 C 盘（退而求其次）

`OllamaSetup.exe` 的 `/DIR=` 若因权限/杀毒失败，可接受程序在 `%LOCALAPPDATA%\Programs\Ollama`（约几百 MB），**务必**仍设 `OLLAMA_MODELS=E:\AI\Models\ollama`——模型才是占盘主力。

若 C 盘已有 `%USERPROFILE%\.ollama\models` 且很大：

```powershell
# 完全退出 Ollama 后
robocopy "$env:USERPROFILE\.ollama\models" "E:\AI\Models\ollama" /E /MOVE
```

然后重启 Ollama，再 `ollama list` 确认模型仍在。

### 拉取模型

```powershell
ollama pull qwen2.5:7b
ollama pull llama3.2
ollama list
```



### 验证

```powershell
echo $env:OLLAMA_MODELS
where.exe ollama
# 程序期望：D:\Apps\AI\Ollama\ollama.exe（若用了 /DIR=）

ollama list
Get-ChildItem "E:\AI\Models\ollama" -ErrorAction SilentlyContinue | Select-Object -First 5 Name
# pull 后应有 blobs / manifests 等

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


| 软件               | 安装位置                                    | 何时装       |
| ---------------- | --------------------------------------- | --------- |
| Fork（Git 客户端）   | `%LOCALAPPDATA%\Fork`（程序）；Git 复用 `D:\Portable\VCS\Git` | Git 装好后，需要可视化 Git 时 |
| ToDesk（远程桌面）   | `D:\Apps\Communication\ToDesk`          | 需要远程连接本机或其它设备时 |
| Termius（SSH）     | `D:\Apps\Utilities\Termius`             | 常连服务器、要图形化 SSH 时 |
| OBS Studio       | `D:\Portable\Media\OBS-Studio`（便携推荐） | 录屏、直播、会议录制 |
| Clash Verge Rev  | `D:\Portable\Network\Clash-Verge-Rev`（便携推荐） | 需要系统代理 / TUN 时 |
| Steam            | `D:\Apps\Games\Steam`；游戏库 `D:\Apps\Games\SteamLibrary` | 玩游戏、Steam 下载 |
| 微信 / QQ          | `D:\Apps\Communication\`                | 日常通讯      |
| Postman / Apifox | `D:\Portable\API\`                      | 接口调试      |
| DBeaver          | `D:\Portable\DB\` 或 `D:\Apps\Database\` | 开源数据库客户端  |
| Python           | `E:\Envs\Python\`                       | Python 开发 |
| Go               | `E:\Envs\Go\`                           | Go 开发     |



### Python

> 程序与 pip 包装 **E 盘**；pip 下载缓存走 `E:\Cache\pip`。**不用 Junction**（无大块用户目录写 C 盘）。

#### 路径规划

| 用途 | 路径 |
|------|------|
| Python 本体 | `E:\Envs\Python\Python313`（示例，按版本号命名） |
| pip 缓存 | `E:\Cache\pip` |
| 安装包 | `D:\Packages\2026\Dev\` |
| 虚拟环境 | 项目内 `.venv` 或 `E:\Envs\Python\venvs` |

#### 安装前（用户变量 + 目录）

```powershell
New-Item -ItemType Directory -Path "E:\Envs\Python\Python313" -Force
New-Item -ItemType Directory -Path "E:\Cache\pip" -Force
```

| 变量名 | 变量值 |
|--------|--------|
| `PIP_CACHE_DIR` | `E:\Cache\pip` |

> **在第一次 `pip install` 之前**设好 `PIP_CACHE_DIR`，缓存才不会进 C 盘。

#### 安装（官方安装器）

1. 下载：https://www.python.org/downloads/windows/ → **Windows installer (64-bit)**  
   保存到 `D:\Packages\2026\Dev\`
2. 运行安装器：
   - 勾 **Add python.exe to PATH**
   - 选 **Customize installation**
   - Optional：勾 **pip**、**py launcher**；**不要**勾 Install for all users
   - Advanced：**Customize install location** → `E:\Envs\Python\Python313`
   - 建议勾 **Disable path length limit**
3. 安装完成

#### 关闭 Windows 应用执行别名（必做）

**设置 → 应用 → 高级应用设置 → 应用执行别名**

关闭：

- `python.exe`
- `python3.exe`

否则 `where python` 可能指向 `C:\Users\...\WindowsApps\python.exe` 占位符。

#### 用户 Path

若 `where python` 未指向 E 盘，手动把下面两项加到用户 Path **靠前位置**（版本号与安装目录一致）：

```text
E:\Envs\Python\Python313
E:\Envs\Python\Python313\Scripts
```

PowerShell 示例：

```powershell
$pyRoot    = "E:\Envs\Python\Python313"
$pyScripts = "E:\Envs\Python\Python313\Scripts"
$p = [Environment]::GetEnvironmentVariable("Path", "User")
$parts = $p -split ';' | Where-Object { $_ -and $_ -notlike "*Python313*" }
$newPath = ($pyRoot, $pyScripts + $parts) -join ';'
[Environment]::SetEnvironmentVariable("Path", $newPath, "User")
```

#### 验证

**新开普通 PowerShell**（不要用 `C:\WINDOWS\system32` 管理员窗口）：

```powershell
where.exe python
# 期望：E:\Envs\Python\Python313\python.exe

python --version
pip --version
python -m pip cache dir
# 期望：E:\Cache\pip

pip install requests
Get-ChildItem "E:\Cache\pip" -Recurse -ErrorAction SilentlyContinue | Select-Object -First 3 FullName
```

#### 国内 pip 镜像（可选）

```powershell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 虚拟环境

```powershell
cd E:\Workspace\Sandbox
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

若激活脚本报策略错误：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### 常见问题

| 现象 | 处理 |
|------|------|
| `python` 打开 Microsoft Store | 关应用执行别名 + 修正 Path |
| `pip` 找不到 | 用 `python -m pip`；或 Path 加 `...\Scripts` |
| `E:\Cache\pip` 为空 | 先 `pip install` 包；缓存在子目录 `http-v2`、`wheels` |
| `py` 找不到 | 非必须；用 `python` 即可 |

### 微信 / QQ

> 程序装 **D 盘**，聊天记录和文件改到 **E 盘**。C 盘只保留小体积配置（`%APPDATA%\Tencent\` 等），无需 Junction。



#### 路径规划


| 用途          | 路径                             |
| ----------- | ------------------------------ |
| 微信程序        | `D:\Apps\Communication\WeChat` |
| QQ 程序       | `D:\Apps\Communication\QQ`     |
| 微信聊天记录/文件   | `E:\Data\wechat`               |
| QQ 消息/文件/缓存 | `E:\Data\qq`                   |
| 安装包         | `D:\Packages\2026\Tools\`      |




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


| 项                | 说明                                  |
| ---------------- | ----------------------------------- |
| 不要放 E:\Workspace | E 盘工作区专用于源码，不与聊天数据混放                |
| C 盘残留            | `%APPDATA%\Tencent\` 等配置目录体积小，可保留   |
| 空间占用             | 聊天记录、图片、文件传输是主要体积，改完文件管理路径后 C 盘不再堆积 |




### ToDesk（远程桌面）

> 远程连接本机或其它设备。程序装 **D 盘**；用户配置在 `%APPDATA%\ToDesk\`，服务相关文件在 `%ProgramData%\ToDesk\`，体积通常很小，**无需 Junction**（与微信策略一致）。  
> 安装会注册系统服务/虚拟显示驱动，需 **管理员权限**（UAC 点「是」）。

#### 路径规划

| 用途 | 路径 |
|------|------|
| ToDesk 程序 | `D:\Apps\Communication\ToDesk` |
| 用户配置（自动） | `%APPDATA%\ToDesk\`（`config.ini`、设备列表等） |
| 服务/公共配置 | `%ProgramData%\ToDesk\`（安装器写入，勿手动挪） |
| 安装包 | `D:\Packages\2026\Tools\` |
| 文件传输/录像（可选） | `E:\Data\todesk`（若软件内可改默认保存路径） |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Apps\Communication\ToDesk" -Force
New-Item -ItemType Directory -Path "E:\Data\todesk" -Force
```

#### 下载与安装

1. 官网：[https://www.todesk.com/download](https://www.todesk.com/download) → **ToDesk 个人版**（认准官方域名，勿用第三方「破解版」）
2. 安装包保存到 `D:\Packages\2026\Tools\`
3. 右键安装包 → **以管理员身份运行**（UAC 点「是」）
4. 选 **自定义安装**（不要「快速安装」到 `C:\Program Files\ToDesk`）
5. 安装路径点 **浏览**，改为：

```text
D:\Apps\Communication\ToDesk
```

6. 建议勾选「创建桌面快捷方式」；**开机自启**按需（常作被控端可勾，仅偶尔远控可不勾）
7. 安装完成后登录（手机验证码 / 微信 / App 扫码均可）

#### 安全与使用建议

| 项 | 建议 |
|----|------|
| 被控端密码 | 设置强密码；或开启「仅允许临时验证码连接」 |
| 无人值守 | 仅在自己设备、且确有需要时开启 |
| 文件传输 | 若设置里有「默认保存路径」，改为 `E:\Data\todesk` |
| 与开发环境 | 与 Git / Cursor / Docker 无冲突；大文件传输时注意带宽 |

#### 验证

```powershell
Test-Path "D:\Apps\Communication\ToDesk\ToDesk.exe"
Get-Service -Name "*todesk*" -ErrorAction SilentlyContinue | Select-Object Name, Status
```

打开 ToDesk，界面显示本机 **设备代码**，能成功远程连接另一台设备（或用手机 App 连本机）即正常。

#### 常见问题

| 现象 | 处理 |
|------|------|
| 安装失败 / 无法写入 | 管理员运行安装包；路径用纯英文；确保 D 盘有 ≥1GB 空间 |
| 仍装到 C 盘 | 卸载后重装，必须选「自定义安装」并改路径 |
| 杀毒拦截 | 将 `D:\Apps\Communication\ToDesk` 加入信任/排除列表 |
| 远程黑屏 / 无画面 | 更新显卡驱动；ToDesk **设置** 里切换渲染/硬件加速选项 |
| 卸载后残留 | 删 `D:\Apps\Communication\ToDesk`；必要时清 `%APPDATA%\ToDesk`、`%ProgramData%\ToDesk` 后重装 |




### Termius（SSH 客户端）

> 图形化 SSH / SFTP，连云服务器、家里 NAS 比纯终端省事。程序可装 **D 盘**；账号同步后本地缓存多在 `%APPDATA%\Termius\`，体积小，**无需 Junction**。

#### 路径规划

| 用途 | 路径 |
|------|------|
| Termius 程序 | `D:\Apps\Utilities\Termius` |
| 本地配置（自动） | `%APPDATA%\Termius\` |
| 安装包 | `D:\Packages\2026\Tools\` |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Apps\Utilities\Termius" -Force
```

#### 下载与安装

1. 官网：[https://termius.com/free-download](https://termius.com/free-download) 或 [https://termi.us/win](https://termi.us/win)
2. 安装包保存到 `D:\Packages\2026\Tools\`
3. **方式 A（图形界面）**：运行安装器，若有路径选项，改为 `D:\Apps\Utilities\Termius`
4. **方式 B（静默指定路径）**：管理员 CMD 进入安装包目录后执行（`/D=` 必须在最后，路径勿加尾部 `\`）：

```cmd
Termius.exe /S /D=D:\Apps\Utilities\Termius
```

5. 登录 Termius 账号（免费版够用；团队功能需订阅）

> **勿装 Microsoft Store 版**（路径在 `WindowsApps`，难迁移）。

#### 验证

```powershell
Test-Path "D:\Apps\Utilities\Termius\Termius.exe"
```

打开 Termius，新建 SSH 连接，能连上一台 Linux 主机即正常。

#### 与开发环境

| 场景 | 工具 |
|------|------|
| 本机写代码 | Cursor / IDEA |
| 连服务器跑命令 | Termius 或 Cursor 内置终端 + `ssh` |
| 密钥 | 可用 Termius 管理，或与 `E:\Secrets\ssh\` 自建密钥配合 |




### OBS Studio（录屏 / 直播）

> 录屏、直播、会议录制。**推荐便携版 + portable 模式**：程序与配置都在 D 盘，录像输出指到 E 盘。

#### 路径规划

| 用途 | 路径 |
|------|------|
| OBS 程序（便携） | `D:\Portable\Media\OBS-Studio` |
| 场景/配置（便携模式） | 同上目录内 `config\` |
| 录像 / 回放输出 | `E:\Data\obs\recordings` |
| 安装包 | `D:\Packages\2026\Tools\` |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Portable\Media\OBS-Studio" -Force
New-Item -ItemType Directory -Path "E:\Data\obs\recordings" -Force
```

#### 下载与安装（便携版，推荐）

1. 官网：[https://obsproject.com/download](https://obsproject.com/download) → **Windows** → 选 **ZIP**（Portable），不要只下 Installer 若你想配置全在 D 盘
2. 或 GitHub Releases：[https://github.com/obsproject/obs-studio/releases](https://github.com/obsproject/obs-studio/releases) → `OBS-Studio-x.x.x-Windows-x64.zip`
3. 解压到 `D:\Portable\Media\OBS-Studio`（解压后应能看到 `bin\64bit\obs64.exe`）
4. 在 `D:\Portable\Media\OBS-Studio` **根目录**新建空文件 `portable_mode.txt`（启用便携模式，配置不写 `%APPDATA%\obs-studio`）
5. 首次运行：`D:\Portable\Media\OBS-Studio\bin\64bit\obs64.exe`
6. **设置 → 输出 → 录像路径** → `E:\Data\obs\recordings`
7. 可建桌面快捷方式指向 `obs64.exe`

#### 安装版（可选）

若坚持用安装器，管理员 CMD 可静默装到 D 盘：

```cmd
OBS-Studio-x.x.x-Windows-x64-Installer.exe /S /D=D:\Apps\Media\OBS-Studio
```

此时用户配置仍在 `%APPDATA%\obs-studio\`，C 盘会多一份配置；**更推荐上面的便携版**。

#### 验证

```powershell
Test-Path "D:\Portable\Media\OBS-Studio\bin\64bit\obs64.exe"
Test-Path "D:\Portable\Media\OBS-Studio\portable_mode.txt"
```

OBS 内试录 10 秒，确认文件出现在 `E:\Data\obs\recordings`。

#### 常见问题

| 现象 | 处理 |
|------|------|
| 游戏采集黑屏 | 以管理员运行 OBS；或改用窗口采集 / 显示器采集 |
| UWP 游戏采不到 | 非 Program Files 安装时，给 OBS 目录加 `ALL APPLICATION PACKAGES` 权限（见 OBS 官方文档） |
| 配置丢了的错觉 | 确认根目录有 `portable_mode.txt`，且从同一 `obs64.exe` 启动 |




### Clash Verge Rev（代理客户端）

> Windows 上维护中的 Clash 图形客户端（原 Clash for Windows 已停更）。**推荐便携版**装 D 盘，订阅与配置在程序目录内 `.config\`，便于备份。  
> 订阅链接属敏感信息，自行保管，建议备份到 `E:\Secrets\` 或 `E:\Config\clash-verge\`。

#### 路径规划

| 用途 | 路径 |
|------|------|
| 程序（便携） | `D:\Portable\Network\Clash-Verge-Rev` |
| 配置（便携） | `D:\Portable\Network\Clash-Verge-Rev\.config\io.github.clash-verge-rev.clash-verge-rev\` |
| 安装包 | `D:\Packages\2026\Tools\` |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Portable\Network\Clash-Verge-Rev" -Force
```

#### 下载与安装（便携版，推荐）

1. GitHub：[https://github.com/clash-verge-rev/clash-verge-rev/releases](https://github.com/clash-verge-rev/clash-verge-rev/releases)
2. 下载 **`Clash.Verge_x64_portable.zip`**（便携版，不要与安装版混用同目录）
3. 解压到 `D:\Portable\Network\Clash-Verge-Rev`
4. 运行 `Clash Verge.exe`（或目录内主程序）
5. **设置** 中确认 **应用目录（App Directory）** 在便携目录下的 `.config\...`（设置里可「打开应用目录」核对）
6. **配置订阅**：配置 / Profiles → 导入机场订阅 URL 或本地 YAML
7. 常用选项（按需）：
   - **系统代理（System Proxy）**：让浏览器等走代理
   - **TUN 模式**：全局接管（需管理员；与部分 VPN/公司网络可能冲突）
   - **开机自启**：按需

> 安装版默认配置在 `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\`（C 盘 Roaming）。若已用安装版想迁 D 盘：设置里打开应用目录 → 整夹复制到便携版 `.config\` 对应位置后改用便携包。

#### 与开发环境

| 项 | 说明 |
|----|------|
| 终端 `git` / `npm` | 开系统代理后多数能直连；若不行，在 Clash 规则里为 `github.com` 等设代理，或临时设用户变量 `HTTP_PROXY` / `HTTPS_PROXY` 为 `http://127.0.0.1:<混合端口>`（端口以软件 **设置** 为准，常见 7897） |
| Docker | TUN 与 Docker 网络偶发冲突；出问题先关 TUN，仅用系统代理 |
| Claude / Codex | 走系统代理或规则分流即可，无需单独为 CLI 改安装路径 |

#### 验证

打开 Clash Verge → **设置 → 应用目录** 确认在 D 盘 → 选节点 → **系统代理** 开启 → 浏览器能打开 [https://www.google.com](https://www.google.com)（或你常用的检查站点）即正常。

#### 常见问题

| 现象 | 处理 |
|------|------|
| 需要 WebView2 | 按提示安装 [Microsoft Edge WebView2](https://developer.microsoft.com/microsoft-edge/webview2/) |
| 升级后配置没了 | 升级前在设置里打开应用目录备份；大版本可先卸再装并保留配置夹 |
| 端口被占用 | 设置里改 **混合端口（Mixed Port）** |




### Steam（游戏平台）

> 客户端装 **D 盘**；**游戏库**单独建在 D 盘并设为默认，避免游戏堆满 C 盘。

#### 路径规划

| 用途 | 路径 |
|------|------|
| Steam 客户端 | `D:\Apps\Games\Steam` |
| 游戏库（默认） | `D:\Apps\Games\SteamLibrary` |
| 安装包 | `D:\Packages\2026\Tools\` |

#### 安装前建目录

```powershell
New-Item -ItemType Directory -Path "D:\Apps\Games\Steam" -Force
New-Item -ItemType Directory -Path "D:\Apps\Games\SteamLibrary" -Force
```

#### 下载与安装

1. 官网：[https://store.steampowered.com/about/](https://store.steampowered.com/about/) → 下载 `SteamSetup.exe`
2. 保存到 `D:\Packages\2026\Tools\`
3. 运行安装器，**安装路径**改为：

```text
D:\Apps\Games\Steam
```

4. 安装完成后登录 Steam 账号
5. **Steam → 设置 → 存储**：
   - 点 **+** 或 **添加驱动器**
   - 选 D 盘，或 **让我选择其他位置** → `D:\Apps\Games\SteamLibrary`
   - 将该库 **设为默认**（三点菜单 → 设为默认）
6. 以后新游戏默认装到 `D:\Apps\Games\SteamLibrary\steamapps\`

#### 已有游戏迁到 D 盘

**Steam → 设置 → 存储** → 选中游戏 → **转移**，目标选 `D:\Apps\Games\SteamLibrary`（不要手拷文件夹，用内置迁移）。

#### 验证

```powershell
Test-Path "D:\Apps\Games\Steam\steam.exe"
```

Steam 设置里默认存储为 D 盘库；下载一个小游戏或已装游戏，确认路径在 `D:\Apps\Games\SteamLibrary\steamapps\common\`。

#### 与开发环境

Steam 与 Git / Cursor / Docker 无冲突；注意游戏盘预留足够空间（大型 3A 单游戏可达 100GB+）。




### 后续环境变量（按需添加，均为用户变量）


| 变量名                 | 变量值                 | 何时加             |
| ------------------- | ------------------- | --------------- |
| `GOPATH`            | `E:\Envs\Go\gopath` | Go 开发           |
| `GOMODCACHE`        | `E:\Cache\go-mod`   | Go 开发           |
| `PIP_CACHE_DIR`     | `E:\Cache\pip`      | Python 开发       |
| `CLAUDE_CONFIG_DIR` | `E:\Data\claude`    | 装 Claude Code 前 |
| `CODEX_HOME`        | `E:\Data\codex`     | 装 Codex 前       |
| `HTTP_PROXY`        | `http://127.0.0.1:7897` | 仅终端不走系统代理时（端口以 Clash 设置为准） |
| `HTTPS_PROXY`       | `http://127.0.0.1:7897` | 同上              |


> `OLLAMA_MODELS` 已移至 [Ollama 章节](#二十三本地-ai-模型ollama)。

---



## 安装完成检查清单

- [ ] D/E 盘目录结构已就绪
- [ ] Git 可用，`git config` 已配置
- [ ] Fork 已装（可选），Preferences → Git 指向 `D:\Portable\VCS\Git\cmd\git.exe`，`gitInstance` 已删或不存在
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
- [ ] Claude Code 已装（国内可用 WinGet），`claude --version` 正常
- [ ] Codex 已装，Path 含 `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`，`codex --version` 正常
- [ ] Codex 沙箱为 `unelevated`（或沙箱菜单已选 2）
- [ ] `.claude` / `.codex` 已 Junction 到 `E:\Data\claude` / `E:\Data\codex`
- [ ] `E:\Data\claude\settings.json`、`E:\Data\codex\auth.json` 存在（CC Switch 已配置）
- [ ] CC Switch 已装（x64 选 `Windows.msi`），Claude Code / Codex 各启用 API Key 供应商
- [ ] `claude`、`codex` 在 `E:\Workspace` 项目里能正常对话
- [ ] Ollama：`OLLAMA_MODELS` 为 `E:\AI\Models\ollama`；程序在 `D:\Apps\AI\Ollama` 或已接受默认 C 盘程序路径（可选）
- [ ] Python 在 `E:\Envs\Python\Python313`，`where python` 指向 E 盘
- [ ] `PIP_CACHE_DIR` 为 `E:\Cache\pip`，`python -m pip cache dir` 一致
- [ ] 已关闭 Windows 应用执行别名中的 `python.exe` / `python3.exe`
- [ ] 微信 / QQ 程序在 `D:\Apps\Communication\`，聊天文件在 `E:\Data\wechat` / `qq`
- [ ] ToDesk 在 `D:\Apps\Communication\ToDesk`，能显示设备代码并正常远控（可选）
- [ ] Termius 在 `D:\Apps\Utilities\Termius`，SSH 可连服务器（可选）
- [ ] OBS 便携版在 `D:\Portable\Media\OBS-Studio`，有 `portable_mode.txt`，录像在 `E:\Data\obs\recordings`（可选）
- [ ] Clash Verge Rev 便携版在 `D:\Portable\Network\Clash-Verge-Rev`，应用目录在 D 盘（可选）
- [ ] Steam 客户端在 `D:\Apps\Games\Steam`，默认游戏库为 `D:\Apps\Games\SteamLibrary`（可选）
- [ ] 在 `E:\Workspace` 下成功打开并运行过一个项目

---

*文档版本：2026-07-02（含 Termius/OBS/Clash/Steam、Fork、ToDesk、Python、WinGet/Codex/Junction/CC Switch）*