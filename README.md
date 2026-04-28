# CLIProxyAPI Linux Installers

This repository provides two Linux installers for [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI):

- `cliproxyapi-installer` for user-level installs with `systemctl --user`
- `cliproxyapi-systemd-installer` for system-level installs managed by root with `systemctl`

## Features

- 🚀 **Automatic Installation** - Detects your Linux architecture and downloads the latest version
- 🔄 **Smart Upgrades** - Preserves your configuration and automatically manages systemd service during upgrades
- 🔑 **API Key Management** - Automatically generates secure API keys
- 🛡️ **Two Service Modes** - Supports both user-level and system-level systemd installation flows
- 📊 **Status Monitoring** - Check installation status and configuration
- 🧹 **Cleanup** - Automatically removes old versions (keeps latest 2)
- 📚 **Documentation Management** - Built-in documentation tools
- ⚡ **Zero-Downtime Updates** - Service is properly stopped and restarted during upgrades

## Quick Start

### Choose an Installer

| Mode | Script | Install Directory | systemd Unit | Service User |
|------|--------|-------------------|--------------|--------------|
| User-level | `cliproxyapi-installer` | `~/cliproxyapi` | `~/.config/systemd/user/cliproxyapi.service` | Your login user |
| System-level | `cliproxyapi-systemd-installer` | `/opt/cliproxyapi` | `/etc/systemd/system/cliproxyapi.service` | `root` |

### User-Level Install

```bash
# Download and run the user-level installer
curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-installer | bash

# Or clone and run manually
git clone https://github.com/brokechubb/cliproxyapi-installer.git
cd cliproxyapi-installer
./cliproxyapi-installer
```

### System-Level Install

```bash
# Download and run the system-level installer
curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-systemd-installer | sudo bash

# Or clone and run manually
git clone https://github.com/brokechubb/cliproxyapi-installer.git
cd cliproxyapi-installer
sudo bash ./cliproxyapi-systemd-installer
```

### After User-Level Installation

1. **Configure API keys**:
   ```bash
   cd ~/cliproxyapi
   nano config.yaml
   ```

2. **Set up authentication**:
   ```bash
   ./cli-proxy-api --login
   ./cli-proxy-api --codex-login
   ./cli-proxy-api --claude-login
   ./cli-proxy-api --qwen-login
   ./cli-proxy-api --iflow-login
   ```

3. **Start the user service**:
   ```bash
   systemctl --user enable cliproxyapi.service
   systemctl --user start cliproxyapi.service
   systemctl --user status cliproxyapi.service
   ```

### After System-Level Installation

1. **Configure API keys**:
   ```bash
   cd /opt/cliproxyapi
   sudo nano config.yaml
   ```

2. **Set up authentication as root**:
   ```bash
   sudo ./cli-proxy-api --login
   sudo ./cli-proxy-api --codex-login
   sudo ./cli-proxy-api --claude-login
   sudo ./cli-proxy-api --qwen-login
   sudo ./cli-proxy-api --iflow-login
   ```

3. **Enable and start the system service**:
   ```bash
   sudo systemctl enable cliproxyapi.service
   sudo systemctl start cliproxyapi.service
   sudo systemctl status cliproxyapi.service
   ```

4. **Follow logs**:
   ```bash
   sudo journalctl -u cliproxyapi.service -f
   ```

> **Important**: The system-level installer runs CLIProxyAPI as `root` with `HOME=/root`. Existing auth or token files under a regular user's home directory are not migrated automatically.

> **💡 Pro Tip**: The installer automatically manages the systemd service during upgrades. If the service is running when you upgrade, it will be gracefully stopped, updated, and restarted automatically.

## Usage

The repository ships two installer scripts:

```bash
./cliproxyapi-installer [COMMAND]
sudo bash ./cliproxyapi-systemd-installer [COMMAND]
```

### Commands

| Command | Description |
|---------|-------------|
| `install` / `upgrade` | Install or upgrade CLIProxyAPI (default) |
| `status` | Show current installation status |
| `auth` | Display authentication setup information |
| `check-config` | Verify configuration and API keys |
| `generate-key` | Generate a new API key |
| `manage-docs` | Manage documentation and check consistency |
| `uninstall` | Remove CLIProxyAPI completely |
| `-h` / `--help` | Show help message |

### Examples

```bash
# Install or upgrade the user-level service
./cliproxyapi-installer

# Install or upgrade the system-level service
sudo bash ./cliproxyapi-systemd-installer

# Check current installation status
./cliproxyapi-installer status
sudo bash ./cliproxyapi-systemd-installer status

# Verify your configuration
./cliproxyapi-installer check-config
sudo bash ./cliproxyapi-systemd-installer check-config

# Generate a new API key
./cliproxyapi-installer generate-key
./cliproxyapi-systemd-installer generate-key

# Show authentication setup info
./cliproxyapi-installer auth
sudo bash ./cliproxyapi-systemd-installer auth

# Uninstall completely
./cliproxyapi-installer uninstall
sudo bash ./cliproxyapi-systemd-installer uninstall
```

## Configuration

### Installation Directories

User-level install:
```text
~/cliproxyapi/
├── cli-proxy-api
├── config.yaml
├── cliproxyapi.service
├── version.txt
├── x.x.x/
└── config_backup/
```

System-level install:
```text
/opt/cliproxyapi/
├── cli-proxy-api
├── config.yaml
├── cliproxyapi.service
├── version.txt
├── x.x.x/
└── config_backup/
```

The active system-level unit is written to `/etc/systemd/system/cliproxyapi.service`.

### API Keys

Both installers automatically generate secure API keys in OpenAI format (`sk-...`). These keys are used for authenticating requests to your proxy server, **not** for provider authentication.

To view or modify your API keys:
```bash
cd ~/cliproxyapi && nano config.yaml
sudo sh -c 'cd /opt/cliproxyapi && nano config.yaml'
```

### Authentication Providers

CLIProxyAPI supports multiple AI providers:

- **Gemini (Google)**: `./cli-proxy-api --login`
- **OpenAI (Codex/GPT)**: `./cli-proxy-api --codex-login`
- **Claude (Anthropic)**: `./cli-proxy-api --claude-login`
- **Qwen (Qwen Chat)**: `./cli-proxy-api --qwen-login`
- **iFlow**: `./cli-proxy-api --iflow-login`

Add `--no-browser` to any login command to print the URL instead of opening a browser automatically.

## System Requirements

- **Operating System**: Linux (amd64, arm64)
- **Required Tools**: `curl` or `wget`, `tar`
- **Shell**: Bash

### Installing Dependencies

**Ubuntu/Debian:**
```bash
sudo apt-get install curl wget tar
```

**CentOS/RHEL:**
```bash
sudo yum install curl wget tar
```

**Fedora:**
```bash
sudo dnf install curl wget tar
```

## Systemd Service

Both installers create and manage a `cliproxyapi.service` unit, but they target different systemd scopes.

### User-Level Service Commands

```bash
systemctl --user enable cliproxyapi.service
systemctl --user start cliproxyapi.service
systemctl --user status cliproxyapi.service
journalctl --user -u cliproxyapi.service -f
```

### System-Level Service Commands

```bash
sudo systemctl enable cliproxyapi.service
sudo systemctl start cliproxyapi.service
sudo systemctl status cliproxyapi.service
sudo journalctl -u cliproxyapi.service -f
```

### System-Level Unit Details

The `cliproxyapi-systemd-installer` writes the active unit to `/etc/systemd/system/cliproxyapi.service` with these characteristics:

- `WorkingDirectory=/opt/cliproxyapi`
- `ExecStart=/opt/cliproxyapi/cli-proxy-api --config /opt/cliproxyapi/config.yaml`
- `User=root`
- `Group=root`
- `Environment=HOME=/root`
- `WantedBy=multi-user.target`

### ✨ Smart Service Management

Both installers provide intelligent service handling during upgrades:

- **Automatic Detection**: Detects if the service is running before upgrades
- **Graceful Shutdown**: Safely stops the service before applying updates
- **Auto-Restart**: Restarts the service after successful upgrades
- **State Preservation**: Maintains the service's previous running state

### Service Status During Upgrades

When you run either installer in upgrade mode, it will:

1. **Check** if the service is currently running
2. **Stop** the service gracefully if it's active
3. **Apply** the upgrade (download, extract, update files)
4. **Restart** the service if it was running before
5. **Report** the final service status

You'll see output like:
```
[INFO] Service is currently running and will be restarted after upgrade
[INFO] Stopping CLIProxyAPI service...
[SUCCESS] Service stopped
...
[INFO] Restarting CLIProxyAPI service...
[SUCCESS] Service restarted successfully
```

### Autostart Configuration

User-level autostart:

```bash
systemctl --user enable cliproxyapi.service
systemctl --user is-enabled cliproxyapi.service
```

System-level autostart:
```bash
sudo systemctl enable cliproxyapi.service
sudo systemctl is-enabled cliproxyapi.service
```

Disable autostart:
```bash
systemctl --user disable cliproxyapi.service
sudo systemctl disable cliproxyapi.service
```

**Important Notes:**
- User-level services start in the logged-in user's systemd scope and typically need lingering for background startup without login: `loginctl enable-linger $USER`
- System-level services are managed through `/etc/systemd/system/cliproxyapi.service` and start through the normal system boot flow
- System-level logs are read with `journalctl -u cliproxyapi.service`
- System-level authentication state lives under `/root` unless CLIProxyAPI is configured otherwise

**If the service is not working:**
```bash
# User-level troubleshooting
systemctl --user daemon-reload
systemctl --user status cliproxyapi.service
journalctl --user -u cliproxyapi.service -n 50
ls -la ~/.config/systemd/user/cliproxyapi.service

# System-level troubleshooting
sudo systemctl daemon-reload
sudo systemctl status cliproxyapi.service
sudo journalctl -u cliproxyapi.service -n 50
ls -la /etc/systemd/system/cliproxyapi.service
```

## Troubleshooting

### Common Issues

1. **Permission Denied**
    ```bash
    chmod +x cliproxyapi-installer
    chmod +x cliproxyapi-systemd-installer
    ```

2. **Missing Dependencies**
    ```bash
    # Check what's missing
    ./cliproxyapi-installer status
    
    # Install required tools
    sudo apt-get install curl wget tar  # Ubuntu/Debian
    ```

3. **API Keys Not Configured**
    ```bash
    ./cliproxyapi-installer check-config
    # Follow the instructions to configure API keys
    ```

4. **Service Won't Start**
    ```bash
    # User-level logs
    journalctl --user -u cliproxyapi.service -n 50

    # System-level logs
    sudo journalctl -u cliproxyapi.service -n 50
    ```

5. **Port Already in Use**
    ```bash
    # Check what's using port 8317
    netstat -tlnp | grep 8317
    
    # Stop the existing process
    pkill cli-proxy-api
    
    # Then restart the service
    systemctl --user restart cliproxyapi.service
    sudo systemctl restart cliproxyapi.service
    ```

6. **Systemd Service Issues**
    ```bash
    # User-level
    systemctl --user daemon-reload
    ls -la ~/.config/systemd/user/cliproxyapi.service
    systemctl --user disable cliproxyapi.service
    systemctl --user enable cliproxyapi.service
    systemctl --user start cliproxyapi.service

    # System-level
    sudo systemctl daemon-reload
    ls -la /etc/systemd/system/cliproxyapi.service
    sudo systemctl disable cliproxyapi.service
    sudo systemctl enable cliproxyapi.service
    sudo systemctl start cliproxyapi.service
    ```

7. **Upgrade Service Issues**
    ```bash
    # If service doesn't restart after upgrade
    systemctl --user status cliproxyapi.service
    sudo systemctl status cliproxyapi.service
    
    # Check recent service logs
    journalctl --user -u cliproxyapi.service -n 20
    sudo journalctl -u cliproxyapi.service -n 20
    
    # Manually restart if needed
    systemctl --user restart cliproxyapi.service
    sudo systemctl restart cliproxyapi.service
    ```

8. **Configuration Protection Issues**
    ```bash
    # User-level backups
    ls -la ~/cliproxyapi/config_backup/
    cp ~/cliproxyapi/config_backup/config_YYYYMMDD_HHMMSS.yaml ~/cliproxyapi/config.yaml
    systemctl --user restart cliproxyapi.service

    # System-level backups
    sudo ls -la /opt/cliproxyapi/config_backup/
    sudo cp /opt/cliproxyapi/config_backup/config_YYYYMMDD_HHMMSS.yaml /opt/cliproxyapi/config.yaml
    sudo systemctl restart cliproxyapi.service
    ```

### Getting Help

```bash
# Show all available commands
./cliproxyapi-installer --help
./cliproxyapi-systemd-installer --help

# Check installation status
./cliproxyapi-installer status
sudo bash ./cliproxyapi-systemd-installer status

# Verify configuration
./cliproxyapi-installer check-config
sudo bash ./cliproxyapi-systemd-installer check-config
```

## Security Considerations

- API keys are automatically generated using cryptographically secure random strings
- Configuration files are stored under the selected installation directory
- The systemd service runs with appropriate security restrictions
- Backups of configuration are created automatically during upgrades
- **User configurations are never overwritten** - your modifications are protected during upgrades
- The system-level installer runs the service as `root`, so review file ownership and auth/token storage expectations before switching from a user-level install

## Updates and Upgrades

The installer automatically checks for newer versions:

```bash
# Check for updates and upgrade if available
./cliproxyapi-installer upgrade
sudo bash ./cliproxyapi-systemd-installer upgrade

# Or simply run (upgrade is the default action)
./cliproxyapi-installer
sudo bash ./cliproxyapi-systemd-installer
```

### Smart Upgrade Process

During upgrades, the installer provides intelligent service management:

- **🔄 Service Management**: If the service is running, it's automatically stopped before upgrade and restarted afterward
- **🛡️ Configuration Protection**: Your `config.yaml` file is **never overwritten** - user modifications are preserved
- **💾 Automatic Backups**: Configuration backups are created automatically before any changes
- **🧹 Version Cleanup**: Old versions are cleaned up (latest 2 versions kept)
- **📋 Service Updates**: Systemd service file is updated if needed

### Upgrade Behavior

| Scenario | Service Action | Config Action |
|----------|----------------|---------------|
| Service running | Stop → Upgrade → Restart | Preserved with backup |
| Service stopped | Upgrade only | Preserved with backup |
| First install | N/A | Created from example with generated keys |

> **🔒 Your configuration is safe**: The installer uses a priority system that always preserves existing user configurations over example files.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This installer script is released under the same license as CLIProxyAPI.

## Support

- **CLIProxyAPI Documentation**: https://github.com/router-for-me/CLIProxyAPI
- **Installer Issues**: https://github.com/brokechubb/cliproxyapi-installer/issues
- **General Help**: Run `./cliproxyapi-installer --help` or `./cliproxyapi-systemd-installer --help`

## Changelog

### Recent Improvements

#### ✅ **Smart Service Management**
- **Automatic Service Detection**: Installer detects if CLIProxyAPI service is running before upgrades
- **Graceful Service Handling**: Service is properly stopped before upgrade and restarted afterward
- **State Preservation**: Service maintains its previous running state after upgrades
- **Enhanced Logging**: Clear feedback about service status throughout the upgrade process

#### ✅ **Enhanced Configuration Protection**
- **Never Overwrite**: User-modified `config.yaml` files are never replaced during upgrades
- **Priority System**: Clear hierarchy for configuration preservation (backup → existing → previous → example)
- **Automatic Backups**: Configuration backups created before any upgrade operations
- **User Notifications**: Clear messaging when user configurations are preserved

#### ✅ **Improved Systemd Integration**
- **Fixed Service File**: Resolved systemd service configuration issues
- **Better Error Handling**: Improved service startup and restart reliability
- **Simplified Security**: Removed problematic restrictions while maintaining security
- **System-Level Installer**: Added `cliproxyapi-systemd-installer` for `/opt/cliproxyapi` and `/etc/systemd/system/cliproxyapi.service`

---

**Note**: This installer is specifically for Linux systems. For other operating systems, please refer to the main CLIProxyAPI repository.
