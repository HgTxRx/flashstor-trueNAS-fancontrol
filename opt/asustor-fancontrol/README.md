# Asustor Fan Control - /opt Installation Files

This directory contains the files ready to be installed in `/opt/asustor-fancontrol/` on a Proxmox system.

## Directory Structure

```
asustor-fancontrol/
├── bin/
│   ├── temp_monitor.sh              # Fan monitoring & control script
│   └── check_asustor_it87.kmod.sh   # Kernel module check script
├── systemd/
│   ├── asustor-fancontrol.service           # Module setup service
│   └── asustor-fancontrol-monitor.service   # Fan monitor service
└── README.md                        # This file
```

## Installation

See [PROXMOX_INSTALLATION.md](../../PROXMOX_INSTALLATION.md) for complete installation instructions.

### Quick Install

```bash
# Copy this entire directory to /opt
sudo cp -r opt/asustor-fancontrol /opt/

# Make scripts executable
sudo chmod +x /opt/asustor-fancontrol/bin/*.sh

# Copy systemd service files
sudo cp /opt/asustor-fancontrol/systemd/*.service /etc/systemd/system/

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable asustor-fancontrol.service asustor-fancontrol-monitor.service
sudo systemctl start asustor-fancontrol.service asustor-fancontrol-monitor.service
```

## Files

### `bin/temp_monitor.sh`
Main temperature monitoring and fan control script. Runs continuously as a daemon.

**Features:**
- Monitors NVMe and system temperatures
- Adjusts fan speed based on temperature curves
- Hysteresis logic to prevent fan hunting
- Optional email alerts
- Configurable thresholds

### `bin/check_asustor_it87.kmod.sh`
Kernel module check/compile/install script. Runs once at boot.

**Features:**
- Checks if asustor_it87 kernel module is loaded
- Compiles and installs via DKMS if missing
- Logs output to `/var/log/asustor-fancontrol/asustor-fancontrol.log`
- Handles kernel updates automatically

### `systemd/asustor-fancontrol.service`
systemd unit for kernel module setup. Runs `check_asustor_it87.kmod.sh` at boot.

### `systemd/asustor-fancontrol-monitor.service`
systemd unit for continuous fan monitoring. Starts after module service, restarts on crash.

## Configuration

Edit `/opt/asustor-fancontrol/bin/temp_monitor.sh` to adjust:

- `frequency` - How often to check temperatures (seconds)
- `hdd_threshold` - NVMe temperature threshold for fan ramp (°C)
- `sys_threshold` - System temperature threshold for fan ramp (°C)
- `min_pwm` - Minimum fan speed (PWM 0-255)
- `hdd_delta_threshold` - Temperature change before responding (°C)
- `sys_delta_threshold` - Temperature change before responding (°C)
- `mailalerts` - Enable email notifications (1=on, 0=off)

After editing, restart the service:
```bash
sudo systemctl restart asustor-fancontrol-monitor.service
```

## Logs

Logs are written to:
```
/var/log/asustor-fancontrol/asustor-fancontrol.log
```

View real-time logs:
```bash
sudo journalctl -u asustor-fancontrol-monitor.service -f
```

## Troubleshooting

Check module status:
```bash
lsmod | grep asustor_it87
sudo dkms status
```

Check service status:
```bash
sudo systemctl status asustor-fancontrol.service
sudo systemctl status asustor-fancontrol-monitor.service
```

View logs:
```bash
sudo journalctl -u asustor-fancontrol.service -n 50
sudo journalctl -u asustor-fancontrol-monitor.service -f
sudo cat /var/log/asustor-fancontrol/asustor-fancontrol.log
```

## License

See LICENSE file in the root of the repository.
