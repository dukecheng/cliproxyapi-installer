# CLIProxyAPI Linux 安装器

[English](README.md) | [简体中文](README.zh-CN.md)

这个仓库提供了两种适用于 [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) 的 Linux 安装器：

- `cliproxyapi-installer`：用户级安装，使用 `systemctl --user`
- `cliproxyapi-systemd-installer`：系统级安装，由 `root` 管理，使用 `systemctl`

## 功能特性

- 🚀 **自动安装** - 自动检测 Linux 架构并下载最新版本
- 🔄 **智能升级** - 在升级过程中保留配置并自动管理 systemd 服务
- 🔑 **API Key 管理** - 自动生成安全的 API Key
- 🛡️ **双服务模式** - 同时支持用户级和系统级 systemd 安装流程
- 📊 **状态监控** - 检查安装状态和配置情况
- 🧹 **自动清理** - 自动删除旧版本（保留最新 2 个）
- 📚 **文档管理** - 内置文档管理工具
- ⚡ **零停机更新** - 升级时会正确停止并重新启动服务

## 快速开始

### 选择安装器

| 模式 | 脚本 | 安装目录 | systemd Unit | 服务运行用户 |
|------|------|----------|--------------|--------------|
| 用户级 | `cliproxyapi-installer` | `~/cliproxyapi` | `~/.config/systemd/user/cliproxyapi.service` | 当前登录用户 |
| 系统级 | `cliproxyapi-systemd-installer` | `/opt/cliproxyapi` | `/etc/systemd/system/cliproxyapi.service` | `root` |

### 用户级安装

```bash
# 下载并运行用户级安装器
curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-installer | bash

# 或者克隆仓库后手动运行
git clone https://github.com/brokechubb/cliproxyapi-installer.git
cd cliproxyapi-installer
./cliproxyapi-installer
```

### 系统级安装

```bash
# 下载并运行系统级安装器
curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-systemd-installer | sudo bash

# 或者克隆仓库后手动运行
git clone https://github.com/brokechubb/cliproxyapi-installer.git
cd cliproxyapi-installer
sudo bash ./cliproxyapi-systemd-installer
```

### 用户级安装后的操作

1. **配置 API keys**：
   ```bash
   cd ~/cliproxyapi
   nano config.yaml
   ```

2. **进行认证设置**：
   ```bash
   ./cli-proxy-api --login
   ./cli-proxy-api --codex-login
   ./cli-proxy-api --claude-login
   ./cli-proxy-api --qwen-login
   ./cli-proxy-api --iflow-login
   ```

3. **启动用户级服务**：
   ```bash
   systemctl --user enable cliproxyapi.service
   systemctl --user start cliproxyapi.service
   systemctl --user status cliproxyapi.service
   ```

### 系统级安装后的操作

1. **配置 API keys**：
   ```bash
   cd /opt/cliproxyapi
   sudo nano config.yaml
   ```

2. **以 root 身份进行认证设置**：
   ```bash
   sudo ./cli-proxy-api --login
   sudo ./cli-proxy-api --codex-login
   sudo ./cli-proxy-api --claude-login
   sudo ./cli-proxy-api --qwen-login
   sudo ./cli-proxy-api --iflow-login
   ```

3. **启用并启动系统级服务**：
   ```bash
   sudo systemctl enable cliproxyapi.service
   sudo systemctl start cliproxyapi.service
   sudo systemctl status cliproxyapi.service
   ```

4. **查看日志**：
   ```bash
   sudo journalctl -u cliproxyapi.service -f
   ```

> **重要说明**：系统级安装器会以 `root` 身份运行 CLIProxyAPI，并设置 `HOME=/root`。普通用户 home 目录下已有的 auth 或 token 文件不会被自动迁移。

> **💡 提示**：安装器会在升级过程中自动管理 systemd 服务。如果服务在升级前正在运行，安装器会先优雅停止，再更新，最后自动重启。

## 用法

这个仓库提供两个安装脚本：

```bash
./cliproxyapi-installer [COMMAND]
sudo bash ./cliproxyapi-systemd-installer [COMMAND]
```

### 命令

| 命令 | 说明 |
|------|------|
| `install` / `upgrade` | 安装或升级 CLIProxyAPI（默认） |
| `status` | 显示当前安装状态 |
| `auth` | 显示认证设置说明 |
| `check-config` | 检查配置和 API keys |
| `generate-key` | 生成新的 API key |
| `manage-docs` | 管理文档并检查一致性 |
| `uninstall` | 完全卸载 CLIProxyAPI |
| `-h` / `--help` | 显示帮助信息 |

### 示例

```bash
# 安装或升级用户级服务
./cliproxyapi-installer

# 安装或升级系统级服务
sudo bash ./cliproxyapi-systemd-installer

# 查看当前安装状态
./cliproxyapi-installer status
sudo bash ./cliproxyapi-systemd-installer status

# 校验配置
./cliproxyapi-installer check-config
sudo bash ./cliproxyapi-systemd-installer check-config

# 生成新的 API key
./cliproxyapi-installer generate-key
./cliproxyapi-systemd-installer generate-key

# 查看认证设置说明
./cliproxyapi-installer auth
sudo bash ./cliproxyapi-systemd-installer auth

# 完全卸载
./cliproxyapi-installer uninstall
sudo bash ./cliproxyapi-systemd-installer uninstall
```

## 配置

### 安装目录

用户级安装：
```text
~/cliproxyapi/
├── cli-proxy-api
├── config.yaml
├── cliproxyapi.service
├── version.txt
├── x.x.x/
└── config_backup/
```

系统级安装：
```text
/opt/cliproxyapi/
├── cli-proxy-api
├── config.yaml
├── cliproxyapi.service
├── version.txt
├── x.x.x/
└── config_backup/
```

实际生效的系统级 unit 文件位于 `/etc/systemd/system/cliproxyapi.service`。

### API Keys

两个安装器都会自动生成 OpenAI 格式（`sk-...`）的安全 API keys。这些 keys 用于对你的代理服务请求进行认证，**不是** 用于上游 provider 认证。

查看或修改 API keys：

```bash
cd ~/cliproxyapi && nano config.yaml
sudo sh -c 'cd /opt/cliproxyapi && nano config.yaml'
```

### 认证 Provider

CLIProxyAPI 支持多个 AI Provider：

- **Gemini (Google)**：`./cli-proxy-api --login`
- **OpenAI (Codex/GPT)**：`./cli-proxy-api --codex-login`
- **Claude (Anthropic)**：`./cli-proxy-api --claude-login`
- **Qwen (Qwen Chat)**：`./cli-proxy-api --qwen-login`
- **iFlow**：`./cli-proxy-api --iflow-login`

你可以给任意登录命令加上 `--no-browser`，让它打印 URL，而不是自动打开浏览器。

## 系统要求

- **操作系统**：Linux（amd64, arm64）
- **依赖工具**：`curl` 或 `wget`，以及 `tar`
- **Shell**：Bash

### 安装依赖

**Ubuntu/Debian：**
```bash
sudo apt-get install curl wget tar
```

**CentOS/RHEL：**
```bash
sudo yum install curl wget tar
```

**Fedora：**
```bash
sudo dnf install curl wget tar
```

## Systemd 服务

两个安装器都会创建并管理 `cliproxyapi.service`，但它们对应的是不同的 systemd 作用域。

### 用户级服务命令

```bash
systemctl --user enable cliproxyapi.service
systemctl --user start cliproxyapi.service
systemctl --user status cliproxyapi.service
journalctl --user -u cliproxyapi.service -f
```

### 系统级服务命令

```bash
sudo systemctl enable cliproxyapi.service
sudo systemctl start cliproxyapi.service
sudo systemctl status cliproxyapi.service
sudo journalctl -u cliproxyapi.service -f
```

### 系统级 Unit 细节

`cliproxyapi-systemd-installer` 会把实际生效的 unit 写入 `/etc/systemd/system/cliproxyapi.service`，其关键特征如下：

- `WorkingDirectory=/opt/cliproxyapi`
- `ExecStart=/opt/cliproxyapi/cli-proxy-api --config /opt/cliproxyapi/config.yaml`
- `User=root`
- `Group=root`
- `Environment=HOME=/root`
- `WantedBy=multi-user.target`

### ✨ 智能服务管理

两个安装器在升级时都提供智能服务管理：

- **自动检测**：升级前检测服务是否正在运行
- **优雅停机**：在应用更新前安全地停止服务
- **自动重启**：升级成功后自动重启服务
- **状态保持**：尽量保持升级前的服务运行状态

### 升级时的服务行为

当你使用任一安装器执行升级时，它会：

1. **检查** 服务当前是否正在运行
2. **停止** 正在运行的服务
3. **执行** 升级（下载、解压、更新文件）
4. **重启** 升级前处于运行状态的服务
5. **报告** 最终服务状态

你会看到类似这样的输出：
```text
[INFO] Service is currently running and will be restarted after upgrade
[INFO] Stopping CLIProxyAPI service...
[SUCCESS] Service stopped
...
[INFO] Restarting CLIProxyAPI service...
[SUCCESS] Service restarted successfully
```

### 开机自启配置

用户级自启：

```bash
systemctl --user enable cliproxyapi.service
systemctl --user is-enabled cliproxyapi.service
```

系统级自启：
```bash
sudo systemctl enable cliproxyapi.service
sudo systemctl is-enabled cliproxyapi.service
```

禁用自启：
```bash
systemctl --user disable cliproxyapi.service
sudo systemctl disable cliproxyapi.service
```

**重要说明：**
- 用户级服务运行在当前登录用户的 systemd scope 中；如果希望在未登录时后台启动，通常还需要开启 lingering：`loginctl enable-linger $USER`
- 系统级服务通过 `/etc/systemd/system/cliproxyapi.service` 管理，并随系统正常启动流程启动
- 系统级日志通过 `journalctl -u cliproxyapi.service` 查看
- 如果 CLIProxyAPI 没有额外配置，系统级认证状态默认位于 `/root` 下

**如果服务无法正常工作：**
```bash
# 用户级排查
systemctl --user daemon-reload
systemctl --user status cliproxyapi.service
journalctl --user -u cliproxyapi.service -n 50
ls -la ~/.config/systemd/user/cliproxyapi.service

# 系统级排查
sudo systemctl daemon-reload
sudo systemctl status cliproxyapi.service
sudo journalctl -u cliproxyapi.service -n 50
ls -la /etc/systemd/system/cliproxyapi.service
```

## 故障排查

### 常见问题

1. **Permission Denied**
   ```bash
   chmod +x cliproxyapi-installer
   chmod +x cliproxyapi-systemd-installer
   ```

2. **缺少依赖**
   ```bash
   # 检查缺少什么
   ./cliproxyapi-installer status

   # 安装依赖
   sudo apt-get install curl wget tar  # Ubuntu/Debian
   ```

3. **API Keys 未配置**
   ```bash
   ./cliproxyapi-installer check-config
   # 按提示配置 API keys
   ```

4. **服务无法启动**
   ```bash
   # 用户级日志
   journalctl --user -u cliproxyapi.service -n 50

   # 系统级日志
   sudo journalctl -u cliproxyapi.service -n 50
   ```

5. **端口已被占用**
   ```bash
   # 查看谁占用了 8317 端口
   netstat -tlnp | grep 8317

   # 停掉已有进程
   pkill cli-proxy-api

   # 然后重启服务
   systemctl --user restart cliproxyapi.service
   sudo systemctl restart cliproxyapi.service
   ```

6. **Systemd 服务问题**
   ```bash
   # 用户级
   systemctl --user daemon-reload
   ls -la ~/.config/systemd/user/cliproxyapi.service
   systemctl --user disable cliproxyapi.service
   systemctl --user enable cliproxyapi.service
   systemctl --user start cliproxyapi.service

   # 系统级
   sudo systemctl daemon-reload
   ls -la /etc/systemd/system/cliproxyapi.service
   sudo systemctl disable cliproxyapi.service
   sudo systemctl enable cliproxyapi.service
   sudo systemctl start cliproxyapi.service
   ```

7. **升级后的服务问题**
   ```bash
   # 如果升级后服务没有重新启动
   systemctl --user status cliproxyapi.service
   sudo systemctl status cliproxyapi.service

   # 查看最近日志
   journalctl --user -u cliproxyapi.service -n 20
   sudo journalctl -u cliproxyapi.service -n 20

   # 如有需要可手动重启
   systemctl --user restart cliproxyapi.service
   sudo systemctl restart cliproxyapi.service
   ```

8. **配置保护相关问题**
   ```bash
   # 用户级备份
   ls -la ~/cliproxyapi/config_backup/
   cp ~/cliproxyapi/config_backup/config_YYYYMMDD_HHMMSS.yaml ~/cliproxyapi/config.yaml
   systemctl --user restart cliproxyapi.service

   # 系统级备份
   sudo ls -la /opt/cliproxyapi/config_backup/
   sudo cp /opt/cliproxyapi/config_backup/config_YYYYMMDD_HHMMSS.yaml /opt/cliproxyapi/config.yaml
   sudo systemctl restart cliproxyapi.service
   ```

### 获取帮助

```bash
# 显示所有可用命令
./cliproxyapi-installer --help
./cliproxyapi-systemd-installer --help

# 查看安装状态
./cliproxyapi-installer status
sudo bash ./cliproxyapi-systemd-installer status

# 校验配置
./cliproxyapi-installer check-config
sudo bash ./cliproxyapi-systemd-installer check-config
```

## 安全注意事项

- API keys 会自动生成，使用安全的随机字符串
- 配置文件存储在所选安装目录下
- systemd 服务会以合适的安全限制运行
- 升级过程中会自动创建配置备份
- **用户配置永远不会被覆盖** - 你的修改会在升级时被保留
- 系统级安装器会以 `root` 运行服务，因此从用户级安装切换过来前，请先确认文件权限以及 auth/token 存储位置的影响

## 更新与升级

安装器会自动检查是否有新版本：

```bash
# 检查更新并在可用时升级
./cliproxyapi-installer upgrade
sudo bash ./cliproxyapi-systemd-installer upgrade

# 或者直接运行（upgrade 是默认动作）
./cliproxyapi-installer
sudo bash ./cliproxyapi-systemd-installer
```

### 智能升级流程

升级时，安装器会提供智能服务管理：

- **🔄 服务管理**：如果服务正在运行，会先自动停止，升级完成后再启动
- **🛡️ 配置保护**：你的 `config.yaml` **绝不会被覆盖**，用户修改会被保留
- **💾 自动备份**：执行任何更改前都会先备份配置
- **🧹 版本清理**：旧版本会被清理（保留最新 2 个）
- **📋 服务更新**：如有需要会更新 systemd service 文件

### 升级行为

| 场景 | 服务动作 | 配置动作 |
|------|----------|----------|
| 服务运行中 | Stop → Upgrade → Restart | 备份并保留 |
| 服务已停止 | 仅升级 | 备份并保留 |
| 首次安装 | N/A | 从示例生成并写入自动生成的 keys |

> **🔒 你的配置是安全的**：安装器使用一套优先级机制，始终优先保留已有用户配置，而不是用示例文件覆盖它。

## 贡献

1. Fork 这个仓库
2. 创建功能分支
3. 提交你的更改
4. 完成充分测试
5. 发起 Pull Request

## 许可证

这个安装脚本使用与 CLIProxyAPI 相同的许可证。

## 支持

- **CLIProxyAPI 文档**：https://github.com/router-for-me/CLIProxyAPI
- **安装器问题反馈**：https://github.com/brokechubb/cliproxyapi-installer/issues
- **通用帮助**：运行 `./cliproxyapi-installer --help` 或 `./cliproxyapi-systemd-installer --help`

## 更新日志

### 最近改进

#### ✅ **智能服务管理**
- **自动检测服务状态**：升级前自动检测 CLIProxyAPI 服务是否正在运行
- **优雅处理服务**：升级前正确停止服务，升级后自动重启
- **保持原运行状态**：升级后尽量维持升级前的服务运行状态
- **增强日志反馈**：在整个升级流程中提供更清晰的服务状态反馈

#### ✅ **增强的配置保护**
- **绝不覆盖**：用户修改过的 `config.yaml` 不会在升级时被替换
- **优先级机制**：配置保护遵循清晰顺序（backup → existing → previous → example）
- **自动备份**：升级前自动创建配置备份
- **清晰提示**：当用户配置被保留时提供明确提示

#### ✅ **改进的 Systemd 集成**
- **修复服务文件**：解决了 systemd service 配置相关问题
- **更好的错误处理**：提升服务启动和重启的可靠性
- **简化安全限制**：移除了有问题的限制，同时保持必要安全性
- **新增系统级安装器**：新增 `cliproxyapi-systemd-installer`，支持 `/opt/cliproxyapi` 和 `/etc/systemd/system/cliproxyapi.service`

---

**说明**：这个安装器专门面向 Linux 系统。其他操作系统请参考 CLIProxyAPI 主仓库。
