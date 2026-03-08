# MacBook Screen Mirroring to Raspberry Pi Using AirPlay (UxPlay)

This guide shows how to turn a Raspberry Pi into an AirPlay receiver so you can mirror (or use as an extended display) from your MacBook to an HDMI display connected to the Pi using **UxPlay**.

After setup, the flow is:

`MacBook -> AirPlay -> Raspberry Pi running UxPlay -> HDMI display`

The Raspberry Pi will start the AirPlay receiver automatically at boot.
<img width="1470" height="956" alt="Screenshot 2026-03-08 at 5 08 34 PM copy" src="https://github.com/user-attachments/assets/74271894-c98f-4e75-9e62-7b407cd3f9fa" />

## Hardware Requirements

- Raspberry Pi
- microSD card
- HDMI display connected to the Pi
- MacBook with AirPlay support
- Both devices connected to the **same network**

## Install Raspberry Pi OS

Download Raspberry Pi OS:

<https://www.raspberrypi.com/software/operating-systems/>

Write the OS image to your microSD card using one of these tools:

- Raspberry Pi Imager
- balenaEtcher

Insert the card into the Raspberry Pi and boot.

## Connect to the Raspberry Pi

Open Terminal on the Raspberry Pi, or connect remotely over SSH:

```bash
ping <PI_IP_ADDRESS>
ssh <PI_USER>@<PI_IP_ADDRESS>
```

Example:

```bash
ssh cz@192.168.5.148
```

## Install UxPlay

Update package lists:

```bash
sudo apt update
```

Install UxPlay:

```bash
sudo apt install uxplay
```

Project repository:

<https://github.com/FDH2/UxPlay>

## Verify UxPlay Installation

Check where `uxplay` is installed:

```bash
which uxplay
```

Expected output is usually:

```text
/usr/bin/uxplay
```

## Create an Auto-Start Service

To automatically start the AirPlay receiver when the Raspberry Pi boots, create a `systemd` service:

```bash
sudo nano /etc/systemd/system/uxplay.service
```

### Service Configuration

Paste this into the file:

```ini
[Unit]
Description=UxPlay AirPlay Receiver
After=network-online.target
Wants=network-online.target

[Service]
# Option 1 (recommended to start): fullscreen with default output mode
ExecStart=/usr/bin/uxplay -fs

# Option 2 (optional): fullscreen with explicit resolution/FPS
# ExecStart=/usr/bin/uxplay -fs -s<WIDTHxHEIGHT@FPS>

Restart=always
RestartSec=5
User=<PI_USER>
Environment=DISPLAY=:0

[Install]
WantedBy=multi-user.target
```

Replace placeholders before saving:

- `<PI_USER>`: your Raspberry Pi username (for example, `cz` or `pi`).
- `<WIDTHxHEIGHT@FPS>`: your target output mode (for example, `1920x1080@60`, `2560x1440@30`).

Notes:

- `-fs` runs UxPlay in fullscreen mode.
- `Restart=always` restarts the service if it stops.
- `DISPLAY=:0` sends output to the main display.

Save and exit Nano.

## Reload systemd

```bash
sudo systemctl daemon-reload
```

This tells `systemd` to read the new `uxplay.service` file.

## Enable Service at Boot

```bash
sudo systemctl enable uxplay.service
```

You should see messages about creating symlinks. That means the service is configured to start automatically.

## Start the Service Now (Without Rebooting)

```bash
sudo systemctl start uxplay.service
```

This starts UxPlay using the service configuration.

## Check Service Status

```bash
systemctl status uxplay.service
```

Look for:

```text
Active: active (running)
```

If you see that, UxPlay is running correctly as a service.

To exit the status view, press `q`.

## Reboot to Confirm Automatic Start

```bash
sudo reboot
```

After reboot, UxPlay should start automatically.

To confirm:

```bash
systemctl status uxplay.service
```

You should again see:

```text
Active: active (running)
```

## Troubleshooting

If the service does not start or AirPlay is not visible, run:

```bash
systemctl status uxplay.service
journalctl -u uxplay.service -b
journalctl -u uxplay.service -f
```

Checks:

- MacBook and Raspberry Pi are on the same network/subnet.
- `User=<PI_USER>` is correct in the service file.
- `uxplay` exists at `/usr/bin/uxplay`.
- No typo in `ExecStart` arguments.

## Disable or Uninstall the Service

Disable and stop the service:

```bash
sudo systemctl disable --now uxplay.service
```

Remove the service file and reload `systemd`:

```bash
sudo rm /etc/systemd/system/uxplay.service
sudo systemctl daemon-reload
```

(Optional) Remove UxPlay package:

```bash
sudo apt remove uxplay
```

## References

- UxPlay project: <https://github.com/FDH2/UxPlay>
- Raspberry Pi OS downloads: <https://www.raspberrypi.com/software/operating-systems/>
- `systemd` service documentation: `man systemd.service`
- `journalctl` documentation: `man journalctl`
