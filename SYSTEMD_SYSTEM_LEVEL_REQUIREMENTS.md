# System-Level systemd Migration Requirements

## Background

The current `cliproxyapi-installer` installs CLIProxyAPI as a **user-level systemd service**:

- Service file is written to `~/.config/systemd/user/cliproxyapi.service`
- Service lifecycle commands use `systemctl --user`
- Documentation instructs users to manage the service with `systemctl --user`
- Logs are expected to be read through `journalctl --user`

This behavior causes confusion for deployments that are expected to behave like a normal Linux system service:

- Service does not appear in `systemctl status` without `--user`
- `journalctl -u cliproxyapi.service` shows no logs for user-managed services
- Service lifecycle depends on the logged-in user session
- Installation path and service model are not aligned with typical server administration expectations

## Goal

Change the installer so that CLIProxyAPI is installed and managed as a **system-level systemd service**.

Target behavior:

- Service file is installed under `/etc/systemd/system/cliproxyapi.service`
- Service lifecycle commands use `systemctl` without `--user`
- Service can be enabled at boot with `systemctl enable cliproxyapi.service`
- Logs are available through `journalctl -u cliproxyapi.service`
- Installer output and documentation consistently describe system-level service management

## Scope

This change applies to the `cliproxyapi-installer` repository only.

In scope:

- Installer script behavior
- Generated systemd unit file
- Installer status/start/stop/restart logic
- Installer help and terminal output
- README updates
- Uninstall cleanup for system-level service
- Root privilege handling for installation and service management

Out of scope:

- Refactoring CLIProxyAPI itself
- Changing CLIProxyAPI logging internals
- Adding support for both user-level and system-level service modes in the same change
- Reworking the installer into a package manager integration

## Requirements

### 1. Service management must become system-level

All service operations must use system-level `systemctl` commands.

Required changes:

- Replace `systemctl --user is-active --quiet cliproxyapi.service` with `systemctl is-active --quiet cliproxyapi.service`
- Replace `systemctl --user stop cliproxyapi.service` with `systemctl stop cliproxyapi.service`
- Replace `systemctl --user start cliproxyapi.service` with `systemctl start cliproxyapi.service`
- Replace `systemctl --user restart cliproxyapi.service` with `systemctl restart cliproxyapi.service`
- Replace `systemctl --user daemon-reload` with `systemctl daemon-reload`

All installer messages, examples, and help text must also drop `--user`.

### 2. The service file must be installed in `/etc/systemd/system`

The generated unit file must no longer be copied into `~/.config/systemd/user`.

Required behavior:

- Generate `cliproxyapi.service` as before for visibility/debugging if desired
- Install the active systemd unit to `/etc/systemd/system/cliproxyapi.service`
- Run `systemctl daemon-reload` after writing the service file
- Show system-level enable/start/status instructions after installation

### 3. The generated unit must target system boot

The generated unit file must use:

```ini
[Install]
WantedBy=multi-user.target
```

It must not use `default.target`.

### 4. The installer must require root privileges

Because writing to `/etc/systemd/system` and managing system services requires elevated privileges, the installer must clearly require root.

Required behavior:

- Detect non-root execution early
- Exit with a clear error message when the script is run without root privileges
- The message must tell the user to rerun with `sudo` or as root

Example behavior:

- `sudo bash cliproxyapi-installer`
- `curl ... | sudo bash`

Root checking should cover install, upgrade, uninstall, and any code path that manages the system-level service.

### 5. The service should run as root in this change

For this iteration, the service will run as `root`.

Required unit characteristics:

- `User=root`
- `Group=root`
- `Environment=HOME=/root`

The change should not introduce a dedicated service account in this iteration.

### 6. The service unit must use stable paths

The generated systemd unit must reference the installed binary and config file using explicit absolute paths.

Recommended unit structure:

```ini
[Unit]
Description=CLIProxyAPI Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=<INSTALL_DIR>
ExecStart=<INSTALL_DIR>/cli-proxy-api --config <INSTALL_DIR>/config.yaml
Restart=always
RestartSec=10
User=root
Group=root
Environment=HOME=/root

[Install]
WantedBy=multi-user.target
```

The final implementation may keep the existing install directory layout, but the service file must not rely on user-level systemd conventions.

### 7. Installer documentation and CLI output must be updated

All references to user-level service management must be removed from:

- `README.md`
- Quick-start output
- Status output
- `check-config` output
- Service setup hints
- Upgrade messages
- Help text where service instructions are shown

Examples must be updated from:

```bash
systemctl --user enable cliproxyapi.service
systemctl --user start cliproxyapi.service
systemctl --user status cliproxyapi.service
journalctl --user -u cliproxyapi.service -f
```

to:

```bash
systemctl enable cliproxyapi.service
systemctl start cliproxyapi.service
systemctl status cliproxyapi.service
journalctl -u cliproxyapi.service -f
```

### 8. Uninstall must remove the system-level unit

The uninstall flow must clean up the systemd unit.

Required behavior:

- Stop the service if present
- Disable the service if enabled
- Remove `/etc/systemd/system/cliproxyapi.service`
- Run `systemctl daemon-reload`
- Remove installer-managed files as before

The uninstall flow should be tolerant of partially installed states.

### 9. Upgrade behavior must remain correct

The current upgrade flow detects whether the service was running, stops it, upgrades files, and restarts it.

That behavior must be preserved, but using system-level service management.

Required behavior:

- Detect running state through `systemctl is-active`
- Stop service before upgrade if active
- Restart service after upgrade only if it was running before
- Keep config preservation behavior unchanged unless required by the service migration

## Notes and Risks

### Logging expectations

After this change, default operational logging expectations should be:

```bash
journalctl -u cliproxyapi.service -f
```

This does not automatically mean logs will appear under `/var/log`.

If file-based logs are needed later, that should be handled through CLIProxyAPI configuration, not through this installer change alone.

### Authentication directory behavior

Because the service will run as `root`, CLIProxyAPI may resolve `~` differently than when it ran as a user-level service.

This can affect:

- `auth-dir`
- OAuth token storage
- any path in config that relied on the previous user’s home directory

This installer change should not silently migrate auth data unless explicitly implemented.

At minimum:

- document the behavior in README or release notes
- avoid implying that existing user-level auth data will automatically carry over

If a compatibility safeguard is easy to implement, the installer may explicitly preserve or set the auth path, but that is optional for this requirement set.

## Acceptance Criteria

The change is complete when all of the following are true:

1. Running the installer as root creates `/etc/systemd/system/cliproxyapi.service`
2. The service can be managed with:
   - `systemctl start cliproxyapi.service`
   - `systemctl stop cliproxyapi.service`
   - `systemctl restart cliproxyapi.service`
   - `systemctl status cliproxyapi.service`
3. `systemctl enable cliproxyapi.service` works and configures boot-time startup
4. `journalctl -u cliproxyapi.service` shows service logs when the service emits stdout/stderr logs
5. No installer output or README instructions still reference `systemctl --user`
6. Uninstall removes the system-level unit cleanly
7. Upgrade preserves the prior running-state behavior using system-level service management

## Test Cases

### Fresh install

- Run installer as root on a clean Linux host
- Confirm the service file is created in `/etc/systemd/system`
- Confirm `systemctl daemon-reload` succeeds
- Confirm `systemctl enable cliproxyapi.service` succeeds
- Confirm `systemctl start cliproxyapi.service` succeeds

### Status checks

- Run installer status command
- Confirm service instructions use system-level `systemctl`
- Confirm no `--user` references remain

### Upgrade

- Start the service
- Run installer upgrade
- Confirm the installer detects the running service
- Confirm it stops and restarts the service correctly
- Confirm config is preserved

### Uninstall

- Uninstall after a completed install
- Confirm the service is stopped and disabled
- Confirm `/etc/systemd/system/cliproxyapi.service` is removed
- Confirm `systemctl daemon-reload` is called

### Documentation verification

- Search the repository for `systemctl --user`
- Search the repository for `journalctl --user`
- Confirm those references are removed or intentionally retained only in historical context

## Implementation Notes

Expected primary edit targets:

- `cliproxyapi-installer`
- `README.md`

Search targets for replacement:

- `systemctl --user`
- `journalctl --user`
- `~/.config/systemd/user`
- `WantedBy=default.target`

## Defaults for This Change

These defaults should be assumed unless explicitly overridden during implementation:

- Service mode: system-level
- Service user: `root`
- Service manager: `systemctl`
- Journal command: `journalctl -u`
- Unit location: `/etc/systemd/system/cliproxyapi.service`
- Install target: `multi-user.target`
