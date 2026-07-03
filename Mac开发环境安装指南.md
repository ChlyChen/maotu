# Mac 开发环境安装指南

> 适用场景：重装 macOS 后的个人开发机（Apple Silicon / Intel 均可）  
> 与 [Windows 开发环境安装指南](./Windows开发环境安装指南.md) 配套，路径与工具链对齐，便于双机切换。  
> **Cursor** 负责 AI 编码；若还需官方扩展生态（Live Share 等），可另装 [VS Code](#vs-code可选与-cursor-并存)。
> 环境变量策略：**写入 `~/.zshrc` / `~/.zprofile`，不修改系统级 `/etc/*`（Docker 等除外）**

### 文档约定

| 约定 | 说明 |
|------|------|
| 数据根目录 | 默认 `~/Dev`（对应 Windows 的 `E:\`）；有外置盘可用 `/Volumes/Data` 替代 |
| Shell | macOS 默认 **zsh**；下文命令均在 **Terminal.app** 或 iTerm2 执行 |
| 包管理 | **Homebrew** 装 CLI 与部分 `.app`；大型 IDE 用官方 `.dmg` 拖入 `/Applications` |
| 路径写法 | macOS 用 `/`；`~` 表示当前用户主目录 |
| 迁出系统盘 | 用 **符号链接（symlink）** 代替 Windows 的 Junction |
| 安装包归档 | `~/Dev/Packages/2026/Dev/`（开发）、`Tools/`（通讯/影音） |
| 与 Windows 对照 | `~/Dev` ≈ `E:\`；`/Applications` ≈ `D:\Apps\`；`~/.cache` 等 ≈ C 盘用户缓存 |

---

## 目录

1. [目录规划](#一目录规划)
2. [目录初始化与 Homebrew](#二目录初始化与-homebrew)
3. [安装顺序总览](#三安装顺序总览)
4. [Git 与 Fork](#四git-与-fork)
5. [JDK 21 + JDK 17](#五jdk-21--jdk-17)
6. [Maven](#六maven)
7. [IntelliJ IDEA](#七intellij-idea)
8. [Node.js（nvm）](#八nodejsnvm)
9. [npm 与 pnpm](#九npm-与-pnpm)
10. [常用小工具](#十常用小工具)
11. [MySQL 8.0](#十一mysql-80)
12. [Redis](#十二redis)
13. [Docker Desktop](#十三docker-desktop)
14. [Navicat](#十四navicat)
15. [Android Studio 与 Flutter](#十五android-studio-与-flutter)
16. [环境变量汇总](#十六环境变量汇总)
17. [IDEA 配置清单](#十七idea-配置清单)
18. [常见问题](#十八常见问题)
19. [Cursor](#十九cursor)
20. [Claude Code 与 Codex](#二十claude-code-与-codex)
21. [CC Switch](#二十一cc-switch)
22. [本地 AI 模型（Ollama）](#二十二本地-ai-模型ollama)
23. [后续按需安装](#二十三后续按需安装)

---

## 一、目录规划

| 位置 | 角色 | 原则 |
|------|------|------|
| **内置 SSD（系统卷）** | 系统 + `/Applications` | 尽量少放大体积缓存与模型 |
| **`~/Dev`** | 开发数据根（≈ Windows `E:\`） | 源码、SDK、缓存、数据、配置 |
| **`/Applications`** | 正式安装的 `.app` | IDEA、Cursor、Android Studio 等 |
| **Homebrew** | CLI 与部分 cask | Apple Silicon：`/opt/homebrew`；Intel：`/usr/local` |

### `~/Dev` 结构（与 Windows E 盘对齐）

```
~/Dev/
├── Workspace/         # 项目源码
│   ├── Company/
│   ├── Personal/
│   ├── OpenSource/
│   ├── Sandbox/
│   └── _Templates/
├── Envs/              # Java、Maven、Node、Python 等
│   ├── Java/
│   ├── Maven/
│   ├── Node/          # nvm 可选迁此（见 Node 章节）
│   └── Python/
├── SDK/
│   ├── Android/
│   └── Flutter/
├── Data/              # MySQL、Redis、claude、codex 等持久化数据
│   ├── mysql/
│   ├── redis/
│   ├── claude/
│   └── codex/
├── Cache/             # Gradle、npm、pip、Homebrew、Cursor、JetBrains 等
│   ├── Cursor/
│   ├── Gradle/
│   ├── Maven/
│   ├── npm/
│   ├── pip/
│   └── homebrew/
├── Config/            # git、maven、idea.properties 等
├── Secrets/           # SSH 密钥等（chmod 600）
├── Logs/
├── AI/
│   └── Models/ollama/
├── Containers/        # Docker 磁盘镜像（可选迁此）
├── Packages/2026/
│   ├── Dev/
│   └── Tools/
└── Backup/
```

### 外置盘（可选）

若有 `/Volumes/Data`，可将整个 `~/Dev` 建在外置卷上，再软链回主目录：

```bash
# 仅在外置盘常挂载时使用
mkdir -p /Volumes/Data/Dev
# 将目录树建在 /Volumes/Data/Dev 后：
ln -s /Volumes/Data/Dev ~/Dev
```

### 核心原则

1. 系统卷只放 macOS 与 `/Applications`
2. 大块数据（SDK、缓存、模型、数据库）进 `~/Dev`
3. 环境变量进 `~/.zshrc`，**不改**系统 `/etc/paths`（除非你知道后果）
4. 必须写在 `~/Library` 的软件，用 **symlink** 把数据指到 `~/Dev`

### 默认在系统盘、需 symlink 或环境变量的项

| 软件 | 默认路径 | 迁出方式 |
|------|----------|----------|
| Cursor 缓存 | `~/Library/Application Support/Cursor` | symlink → `~/Dev/Cache/Cursor` |
| Claude / Codex | `~/.claude`、`~/.codex` | symlink → `~/Dev/Data/` |
| Ollama 模型 | `~/.ollama/models` | `OLLAMA_MODELS=~/Dev/AI/Models/ollama` |
| JetBrains | `~/Library/Application Support/JetBrains` | `idea.properties` 指到 `~/Dev` |
| Homebrew 缓存 | `~/Library/Caches/Homebrew` | `HOMEBREW_CACHE=~/Dev/Cache/homebrew` |
| nvm | `~/.nvm` | 可保留（体积不大）或 `NVM_DIR=~/Dev/Envs/Node/nvm` |
| Fork | `/Applications/Fork.app` | Mac 版可直接用系统 Git，无 `bash.exe` 问题 |

---

## 二、目录初始化与 Homebrew

### 创建目录树

```bash
mkdir -p ~/Dev/{Workspace/{Company,Personal,OpenSource,Sandbox,_Templates},Envs/{Java,Maven,Node,Python},SDK/{Android,Flutter},Data/{mysql,redis,claude,codex,obs},Cache/{Cursor,Gradle,Maven,npm,pip,homebrew,JetBrains},Config/{git,maven,ide/jetbrains},Secrets,Logs,AI/Models/ollama,Containers,Packages/2026/{Dev,Tools},Backup}
```

### 安装 Xcode Command Line Tools

```bash
xcode-select --install
```

弹出对话框点「安装」。验证：

```bash
git --version   # 先有 Apple Git，后面会换 Homebrew Git
clang --version
```

### 安装 Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Apple Silicon 安装完成后按提示把 brew 加入 PATH（通常写入 `~/.zprofile`）：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Intel Mac 一般为 `/usr/local/bin/brew`。

### Homebrew 缓存迁出（建议尽早设置）

在 `~/.zshrc` 追加：

```bash
export HOMEBREW_CACHE="$HOME/Dev/Cache/homebrew"
export HOMEBREW_DOWNLOADS="$HOME/Dev/Cache/homebrew/downloads"
```

### 配置模板

| 文件 | 用途 |
|------|------|
| `~/Dev/Config/maven/settings.xml` | Maven 本地仓库 |
| `~/Dev/Config/ide/jetbrains/idea.properties` | IDEA 路径重定向 |
| `~/Dev/Config/git/.gitconfig` | Git 全局配置（或 `~/.gitconfig` 软链到此） |

---

## 三、安装顺序总览

```
① 目录结构 + Homebrew + ~/.zshrc 基础变量
② Git（Homebrew）→ 可选 Fork
③ JDK 21 + JDK 17
④ Maven
⑤ IntelliJ IDEA
⑥ nvm + Node 22/20 + npm + pnpm
⑦ 常用小工具（Keka、Raycast 等，按需）
⑧ MySQL 8
⑨ Redis
⑩ Docker Desktop
⑪ Navicat（按需）
⑫ Android Studio → Flutter（按需）
⑬ Cursor（装后做缓存 symlink）
⑭ Claude Code + Codex（环境变量 → 装 CLI → symlink）
⑮ CC Switch（配 API → 再用 CLI）
⑯ Ollama（可选：先 OLLAMA_MODELS → brew install → pull）
⑰ Python（按需）
⑱ 按需：Clash / Termius / OBS / Steam / Discord / ToDesk …
```

> **流程要点**：`CLAUDE_CONFIG_DIR` / `CODEX_HOME` 与 symlink 在 CC Switch **之前**；`OLLAMA_MODELS` 在**第一次 pull 之前**；Cursor 缓存 symlink 在**大量扩展索引之前**更省事。

---

## 四、Git 与 Fork

### 安装 Git（Homebrew，推荐）

```bash
brew install git
```

Homebrew Git 通常在 `/opt/homebrew/bin/git`（Apple Silicon）。

### Git 配置

```bash
cp ~/Dev/Config/git/.gitconfig.example ~/Dev/Config/git/.gitconfig
# 编辑 user.name / user.email
ln -sf ~/Dev/Config/git/.gitconfig ~/.gitconfig
```

`~/.zshrc` 可选：

```bash
export GIT_CONFIG_GLOBAL="$HOME/Dev/Config/git/.gitconfig"
```

### 验证

```bash
which git
git --version
git config --global user.name
```

期望 `which git` 为 `/opt/homebrew/bin/git`（brew 优先于 Apple Git）。

### Fork（可选，Git 图形客户端）

> Mac 版 Fork 装在 `/Applications/Fork.app`，**无** Windows 上 `Missing bash.exe` 问题。  
> **Preferences → Git → Git Instance** 选系统 Git 即可：`/opt/homebrew/bin/git` 或 `$(which git)`。  
> **不必装** GitHub Desktop（与 Fork 重叠）。

1. 下载：[https://git-fork.com/](https://git-fork.com/)
2. 拖入 `/Applications`
3. **File → Preferences → Git**，确认指向 Homebrew Git
4. **File → Open Repository** → `~/Dev/Workspace/...`

| 场景 | 工具 |
|------|------|
| 编码 + AI | Cursor |
| Java 后端 | IDEA |
| 可视化 Git | Fork |
| 命令行 | `git` |

---

## 五、JDK 21 + JDK 17

### 安装（Temurin cask）

```bash
brew install --cask temurin@21
brew install --cask temurin@17
```

### 目录与 `JAVA_HOME`

brew 默认装到 `/Library/Java/JavaVirtualMachines/`。为与 Windows 习惯对齐，用 `~/Dev/Envs/Java/current` 作切换点：

```bash
# 查看实际路径
/usr/libexec/java_home -V

# 示例：链到当前默认 JDK 21（路径按 java_home 输出改）
ln -sfn "$(/usr/libexec/java_home -v 21)" ~/Dev/Envs/Java/current
```

`~/.zshrc`：

```bash
export JAVA_HOME="$HOME/Dev/Envs/Java/current/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
```

切换默认 JDK 17 时：

```bash
ln -sfn "$(/usr/libexec/java_home -v 17)" ~/Dev/Envs/Java/current
```

### 验证

```bash
java -version
echo $JAVA_HOME
```

---

## 六、Maven

### 安装

```bash
brew install maven
# 或手动解压到 ~/Dev/Envs/Maven/apache-maven-3.9.15
```

若用手动解压，在 `~/.zshrc` 设 `MAVEN_HOME`。

### Maven 配置

`~/Dev/Config/maven/settings.xml`：

```xml
<localRepository>/Users/你的用户名/Dev/Cache/Maven</localRepository>
```

（路径改成你的实际用户名，或用 `$HOME`。）

### 验证

```bash
mvn -version
```

---

## 七、IntelliJ IDEA

### 下载安装

[https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/) → **macOS Apple Silicon** 或 **Intel**

拖入 `/Applications`。

### 缓存迁出（首次启动前）

创建 `~/Dev/Config/ide/jetbrains/idea.properties`：

```properties
idea.config.path=/Users/你的用户名/Dev/Config/ide/jetbrains/IntelliJIdea/config
idea.system.path=/Users/你的用户名/Dev/Cache/JetBrains/IntelliJIdea/system
idea.plugins.path=/Users/你的用户名/Dev/Config/ide/jetbrains/IntelliJIdea/plugins
idea.log.path=/Users/你的用户名/Dev/Logs/ide/jetbrains
```

复制到 IDEA 配置目录：

```bash
mkdir -p ~/Library/Application\ Support/JetBrains
cp ~/Dev/Config/ide/jetbrains/idea.properties ~/Library/Application\ Support/JetBrains/idea.properties
```

`~/.zshrc`：

```bash
export GRADLE_USER_HOME="$HOME/Dev/Cache/Gradle"
```

---

## 八、Node.js（nvm）

### 安装 nvm

```bash
brew install nvm
```

创建 nvm 目录并按 brew 提示配置 `~/.zshrc`（brew 会输出需追加的几行），通常包括：

```bash
export NVM_DIR="$HOME/.nvm"
# 或迁到 Dev：export NVM_DIR="$HOME/Dev/Envs/Node/nvm"
[ -s "$(brew --prefix nvm)/nvm.sh" ] && \. "$(brew --prefix nvm)/nvm.sh"
```

### 安装 Node

```bash
nvm install 22
nvm install 20
nvm use 22
nvm alias default 22
```

### 验证

```bash
node -v
which node
```

---

## 九、npm 与 pnpm

### npm 缓存

`~/.zshrc`：

```bash
export npm_config_cache="$HOME/Dev/Cache/npm"
```

### pnpm

```bash
brew install pnpm
# 或 corepack enable && corepack prepare pnpm@latest --activate
```

```bash
export PNPM_HOME="$HOME/Dev/Cache/pnpm"
export PATH="$PNPM_HOME:$PATH"
```

### 验证

```bash
npm -v
pnpm -v
npm config get cache
```

---

## 十、常用小工具

| 工具 | 安装 | 说明 |
|------|------|------|
| **Keka** | `brew install --cask keka` | 解压，≈ Windows 7-Zip |
| **Raycast** | `brew install --cask raycast` | 启动器/搜索，≈ Everything（可选） |
| **wget / jq** | `brew install wget jq` | 常用 CLI |

---

## 十一、MySQL 8.0

### 安装

```bash
brew install mysql@8.0
brew services start mysql@8.0
```

### 数据目录迁到 `~/Dev/Data/mysql`

停服务后改配置（路径以 brew 提示为准，Apple Silicon 常见 ` /opt/homebrew/etc/my.cnf`）：

```ini
[mysqld]
datadir = /Users/你的用户名/Dev/Data/mysql
```

初始化并启动：

```bash
brew services stop mysql@8.0
mysqld --initialize-insecure --datadir="$HOME/Dev/Data/mysql"  # 仅首次空库
brew services start mysql@8.0
```

### 创建开发账号

```bash
mysql -u root -e "CREATE USER 'dev'@'localhost' IDENTIFIED BY '你的密码'; GRANT ALL ON *.* TO 'dev'@'localhost'; FLUSH PRIVILEGES;"
```

---

## 十二、Redis

```bash
brew install redis
```

数据目录：`~/Dev/Data/redis`，在 `redis.conf` 中设 `dir` 与 `dbfilename`，或用 `brew services` 默认后把 RDB 路径改过去。

```bash
brew services start redis
redis-cli ping   # 期望 PONG
```

---

## 十三、Docker Desktop

1. [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/) 下载 **Mac (Apple Silicon / Intel)**
2. 拖入 `/Applications`
3. **Settings → Resources → Advanced → Disk image location** → `~/Dev/Containers/Docker`

### 验证

```bash
docker run hello-world
```

---

## 十四、Navicat

官网下载 macOS 版 → 拖入 `/Applications`。连接本机 MySQL / Redis，与 Windows 文档相同。

---

## 十五、Android Studio 与 Flutter

### Android Studio

下载安装 → **Settings → Android SDK**：

- SDK：`~/Dev/SDK/Android/sdk`
- AVD / `.android`：`~/Dev/SDK/Android/home`

`~/.zshrc`：

```bash
export ANDROID_HOME="$HOME/Dev/SDK/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export ANDROID_AVD_HOME="$HOME/Dev/SDK/Android/home/avd"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"
```

### Flutter

```bash
cd ~/Dev/SDK/Flutter
git clone https://github.com/flutter/flutter.git -b stable
```

`~/.zshrc` 追加 `~/Dev/SDK/Flutter/flutter/bin`；国内可选镜像：

```bash
export PUB_CACHE="$HOME/Dev/Cache/pub"
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PUB_HOSTED_URL=https://pub.flutter-io.cn
```

```bash
flutter doctor
```

---

## 十六、环境变量汇总

将以下内容合并进 `~/.zshrc`（按已安装项取舍），然后 `source ~/.zshrc`：

```bash
# === Dev 根 ===
export DEV_ROOT="$HOME/Dev"

# === Java / Maven / Gradle ===
export JAVA_HOME="$HOME/Dev/Envs/Java/current/Contents/Home"
export MAVEN_HOME="$(brew --prefix maven 2>/dev/null)/libexec"   # 若 brew 装 maven
export GRADLE_USER_HOME="$HOME/Dev/Cache/Gradle"
export PATH="$JAVA_HOME/bin:$PATH"

# === Git ===
export GIT_CONFIG_GLOBAL="$HOME/Dev/Config/git/.gitconfig"

# === Node ===
export NVM_DIR="$HOME/.nvm"
export npm_config_cache="$HOME/Dev/Cache/npm"
export PNPM_HOME="$HOME/Dev/Cache/pnpm"
export PATH="$PNPM_HOME:$PATH"

# === Python ===
export PIP_CACHE_DIR="$HOME/Dev/Cache/pip"

# === Android / Flutter ===
export ANDROID_HOME="$HOME/Dev/SDK/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export ANDROID_AVD_HOME="$HOME/Dev/SDK/Android/home/avd"
export PUB_CACHE="$HOME/Dev/Cache/pub"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$HOME/Dev/SDK/Flutter/flutter/bin:$PATH"

# === AI CLI（装 Claude/Codex 前设置）===
export CLAUDE_CONFIG_DIR="$HOME/Dev/Data/claude"
export CODEX_HOME="$HOME/Dev/Data/codex"

# === Ollama（pull 前设置）===
export OLLAMA_MODELS="$HOME/Dev/AI/Models/ollama"

# === Homebrew 缓存 ===
export HOMEBREW_CACHE="$HOME/Dev/Cache/homebrew"

# === 代理（按需，端口以 Clash 为准）===
# export HTTP_PROXY="http://127.0.0.1:7897"
# export HTTPS_PROXY="http://127.0.0.1:7897"
```

> Apple Silicon 上 Homebrew 的 `eval "$(/opt/homebrew/bin/brew shellenv)"` 写在 `~/.zprofile`；nvm 等写在 `~/.zshrc`。

---

## 十七、IDEA 配置清单

| 位置 | 配置项 | 值 |
|------|--------|-----|
| Project Structure → SDKs | JDK 21 / 17 | `/Library/Java/JavaVirtualMachines/` 或 `~/Dev/Envs/Java/current` |
| Settings → Maven | Local repository | `~/Dev/Cache/Maven` |
| Settings → Maven | User settings | `~/Dev/Config/maven/settings.xml` |
| Settings → Gradle | Gradle user home | `~/Dev/Cache/Gradle` |
| Settings → Git | Path to Git | `/opt/homebrew/bin/git` |
| Settings → System Settings | Default directory | `~/Dev/Workspace` |
| Database | MySQL | `localhost:3306`，用户 `dev` |
| Database | Redis | `127.0.0.1:6379` |

---

## 十八、常见问题

### 1. `command not found: brew`

执行 `eval "$(/opt/homebrew/bin/brew shellenv)"` 并写入 `~/.zprofile`。

### 2. Apple Git 版本太旧

`brew install git`，确认 `which git` 为 `/opt/homebrew/bin/git`。

### 3. `mvn` 找不到 `JAVA_HOME`

确认 `JAVA_HOME` 指向 `.../Contents/Home`，且 `java -version` 正常。

### 4. nvm 在新终端不生效

检查 `~/.zshrc` 是否 source 了 `nvm.sh`；Terminal 是否默认登录 shell。

### 5. MySQL 权限 / 数据目录

确保 `~/Dev/Data/mysql` 属主为当前用户：`chown -R $(whoami) ~/Dev/Data/mysql`。

### 6. Docker 占满系统盘

改 Disk image location 到 `~/Dev/Containers/Docker`。

### 7. Claude Code 安装失败（网络）

```bash
# 开代理后
export HTTPS_PROXY=http://127.0.0.1:7897
curl -fsSL https://claude.ai/install.sh | bash
# 或 npm 全局安装
```

### 8. CC Switch 与 `CLAUDE_CONFIG_DIR` 分裂

CC Switch 只写 `~/.claude` / `~/.codex`，需 symlink（见二十、二十一章）。

### 9. Fork 在 Mac 上不必改 Git 路径

默认用 `$(which git)` 即可；无 Windows `cmd`/`bin` 之分。

---

## 十九、Cursor

### 路径规划

| 用途 | 默认路径 | 目标 |
|------|----------|------|
| 程序 | `/Applications/Cursor.app` | 拖入 Applications |
| 应用数据 | `~/Library/Application Support/Cursor` | symlink → `~/Dev/Cache/Cursor` |
| 用户配置 | `~/.cursor` | 可选 symlink → `~/Dev/Config/ide/cursor` |

### 安装

[https://cursor.com/download](https://cursor.com/download) → **Mac** → 拖入 `/Applications`。

### 缓存迁出（symlink，推荐首次启动前）

```bash
mkdir -p ~/Dev/Cache/Cursor

# 若已用过 Cursor，先退出应用再迁移：
if [ -d "$HOME/Library/Application Support/Cursor" ] && [ ! -L "$HOME/Library/Application Support/Cursor" ]; then
  mv "$HOME/Library/Application Support/Cursor" ~/Dev/Cache/Cursor/Application\ Support
fi
ln -sfn ~/Dev/Cache/Cursor/Application\ Support "$HOME/Library/Application Support/Cursor"
```

验证：

```bash
ls -la ~/Library/Application\ Support/Cursor
# 应显示 -> .../Dev/Cache/Cursor/Application Support
```

### 命令行

安装时勾选 **Install 'cursor' command**，或手动把 `Cursor.app` 内 bin 加入 PATH。

```bash
cd ~/Dev/Workspace/Personal/your-project
cursor .
```

---

## 二十、Claude Code 与 Codex

> 与 Windows 版逻辑一致：先环境变量，再装 CLI，再 symlink，最后 CC Switch。

### 环境变量（`~/.zshrc`，装 CLI 前）

```bash
export CLAUDE_CONFIG_DIR="$HOME/Dev/Data/claude"
export CODEX_HOME="$HOME/Dev/Data/codex"
mkdir -p "$CLAUDE_CONFIG_DIR" "$CODEX_HOME"
```

### 安装 Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
# 国内可开代理或使用 npm 安装
```

### 安装 Codex

按 OpenAI 官方 macOS 说明安装；`codex` 常在 `~/.local/bin` 或 npm global：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### symlink（CC Switch 必做）

```bash
# 先退出 claude / codex / CC Switch
ln -sfn ~/Dev/Data/claude ~/.claude
ln -sfn ~/Dev/Data/codex ~/.codex
```

验证：

```bash
ls -la ~/.claude ~/.codex
claude --version
codex --version
```

### 三者分工

| 工具 | 场景 |
|------|------|
| Cursor | 图形界面 AI 编码 |
| Claude Code | 终端 Agent 式改项目 |
| Codex | OpenAI 生态终端 CLI |

---

## 二十一、CC Switch

官网：[https://ccswitch.io/](https://ccswitch.io/)  
GitHub Releases 下载 **macOS `.dmg`（按芯片选 arm64 / x64）**

### 安装

拖入 `/Applications`。自身数据在 `~/.cc-switch`，体积小，可保留。

### 配置流程

1. 先装好 `claude`、`codex` 并建好 `~/.claude` / `~/.codex` symlink  
2. CC Switch → **Claude Code** 页添加供应商 → Enable  
3. **Codex** 页单独添加 → Enable  
4. 新开终端在 `~/Dev/Workspace` 下验证

> CC Switch **不读取** `CLAUDE_CONFIG_DIR` / `CODEX_HOME`，只写 `~/.claude`、`~/.codex`。

---

## 二十二、本地 AI 模型（Ollama）

### 路径

| 用途 | 路径 |
|------|------|
| 程序 | `brew install ollama` → `/opt/homebrew/bin/ollama` |
| 模型 | `~/Dev/AI/Models/ollama`（`OLLAMA_MODELS`） |

### 安装前（`~/.zshrc`）

```bash
export OLLAMA_MODELS="$HOME/Dev/AI/Models/ollama"
mkdir -p "$OLLAMA_MODELS"
```

### 安装与拉模型

```bash
brew install ollama
brew services start ollama
ollama pull qwen2.5:7b
ollama list
```

### 验证

```bash
ollama run qwen2.5:7b
curl http://localhost:11434/api/tags
```

### Cursor 接 Ollama（可选）

**Settings → Models** → OpenAI 兼容端点：`http://localhost:11434/v1`

---

## 二十三、后续按需安装

| 软件 | 安装位置 / 方式 | 何时装 |
|------|-----------------|--------|
| Fork | `/Applications/Fork.app` | 可视化 Git |
| Clash Verge Rev | `/Applications` 或官网 dmg | 需要代理时 |
| Termius | App Store / 官网 | SSH 图形客户端 |
| OBS Studio | `brew install --cask obs` | 录屏 |
| Steam | 官网 dmg | 游戏 |
| Discord | 官网 dmg | 海外社区（非必需） |
| VS Code | `/Applications/Visual Studio Code.app` | 官方扩展/Live Share，与 Cursor 分工 |
| 微信 / QQ | 官网 Mac 版 | 日常通讯 |

### VS Code（可选，与 Cursor 并存）

> 与 Cursor **可共存**；扩展目录分开（Cursor → `~/Library/Application Support/Cursor`，VS Code → `Code`）。  
> 常见理由：Live Share、仅支持 VS Code 的插件、团队统一编辑器。

#### 路径

| 用途 | 路径 |
|------|------|
| 程序 | `/Applications/Visual Studio Code.app` |
| 用户数据 | `~/Library/Application Support/Code` → symlink `~/Dev/Cache/VSCode` |
| 扩展 | `~/Dev/Cache/VSCode/extensions`（`VSCODE_EXTENSIONS`） |

#### 安装

```bash
brew install --cask visual-studio-code
# 或官网 dmg 拖入 /Applications
```

`~/.zshrc`（安装后、首次大量装扩展前）：

```bash
export VSCODE_EXTENSIONS="$HOME/Dev/Cache/VSCode/extensions"
mkdir -p "$VSCODE_EXTENSIONS"
```

#### 缓存 symlink（推荐）

```bash
mkdir -p ~/Dev/Cache/VSCode/Application\ Support
# 若已用过 VS Code，先退出再 mv 原目录内容
if [ -d "$HOME/Library/Application Support/Code" ] && [ ! -L "$HOME/Library/Application Support/Code" ]; then
  mv "$HOME/Library/Application Support/Code"/* ~/Dev/Cache/VSCode/Application\ Support/ 2>/dev/null
  rm -rf "$HOME/Library/Application Support/Code"
fi
ln -sfn ~/Dev/Cache/VSCode/Application\ Support "$HOME/Library/Application Support/Code"
```

#### 验证

```bash
code --version
cd ~/Dev/Workspace/Personal/your-project && code .
```

| 场景 | 工具 |
|------|------|
| AI 编码 | Cursor |
| Live Share / 官方扩展 | VS Code |
| Java | IDEA |

### Clash Verge Rev（macOS）

1. [GitHub Releases](https://github.com/clash-verge-rev/clash-verge-rev/releases) → macOS dmg  
2. 应用目录：**设置 → App Directory** 可在 `~/Library/Application Support/...`  
3. 系统代理 / TUN 按需开启；混合端口以软件显示为准

### OBS

```bash
brew install --cask obs
```

录像路径：**设置 → 输出 → 录像** → `~/Dev/Data/obs/recordings`

### Steam

安装后 **Steam → 设置 → 下载 → 内容库** 添加文件夹，例如外置盘或 `~/Dev/Games/SteamLibrary`（需自建）。

### Discord（可选）

官网安装，程序在 `/Applications/Discord.app`；数据在 `~/Library/Application Support/discord`。非开发必需。

---

## 安装完成检查清单

- [ ] `~/Dev` 目录树已创建，`brew` 可用
- [ ] `~/.zshrc` / `~/.zprofile` 已配置，`source ~/.zshrc` 无报错
- [ ] Git、`java`、`mvn`、`node` 正常
- [ ] IDEA 已装，`idea.properties` 指向 `~/Dev`
- [ ] MySQL / Redis / Docker 按需可用
- [ ] Cursor 缓存已 symlink 到 `~/Dev/Cache/Cursor`（可选但推荐）
- [ ] VS Code 已装（可选），`VSCODE_EXTENSIONS` 在 `~/Dev/Cache/VSCode/extensions`
- [ ] `.claude` / `.codex` 已 symlink，`claude` / `codex` 可用
- [ ] CC Switch 已配 API（若用第三方 Key）
- [ ] `OLLAMA_MODELS` 在 pull 前已设（若用 Ollama）
- [ ] Fork / Clash / Termius 等按需安装
- [ ] 在 `~/Dev/Workspace` 下成功打开并运行过一个项目

---

## 与 Windows 版路径对照（速查）

| 用途 | Windows | macOS |
|------|---------|-------|
| 工作区 | `E:\Workspace` | `~/Dev/Workspace` |
| Git | `D:\Portable\VCS\Git` | `/opt/homebrew/bin/git` |
| Fork Git | `...\Git\bin\git.exe` | `$(which git)` |
| Java | `E:\Envs\Java\current` | `~/Dev/Envs/Java/current` |
| Maven 仓库 | `E:\Cache\Maven` | `~/Dev/Cache/Maven` |
| Node (nvm) | `E:\Envs\Node\nvm` | `~/.nvm` 或 `~/Dev/Envs/Node/nvm` |
| Cursor 缓存 | Junction → `E:\Cache\Cursor` | symlink → `~/Dev/Cache/Cursor` |
| VS Code 扩展 | `VSCODE_EXTENSIONS` → `E:\Cache\VSCode\extensions` | 同上 → `~/Dev/Cache/VSCode/extensions` |
| Claude 数据 | Junction `E:\Data\claude` | symlink `~/Dev/Data/claude` |
| Ollama 模型 | `E:\AI\Models\ollama` | `~/Dev/AI/Models/ollama` |
| 安装包 | `D:\Packages\2026\` | `~/Dev/Packages/2026/` |

---

*文档版本：2026-07-02（与 Windows 指南配套初版）*
