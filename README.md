# MacBook Screen Mirroring to Raspberry Pi Using AirPlay (UxPlay)

This guide shows how to turn a Raspberry Pi into an AirPlay receiver so you can mirror (or use as an extended display) from your MacBook to an HDMI display connected to the Pi using **UxPlay**.

After setup, the flow is:

`MacBook -> AirPlay -> Raspberry Pi running UxPlay -> HDMI display`

The Raspberry Pi will start the AirPlay receiver automatically at boot.

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
ping 192.168.5.148
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

## Create an Auto-Start Service

To automatically start the AirPlay receiver when the Raspberry Pi boots, create a `systemd` service:

```bash
sudo nano /etc/systemd/system/uxplay.service
```

### Service Configuration

Paste the following into the file:

```ini
[Unit]
Description=UxPlay AirPlay Receiver
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/bin/uxplay -fs -s2560x1440@30
Restart=always
RestartSec=5
User=cz
Environment=DISPLAY=:0

[Install]
WantedBy=multi-user.target
```

Notes:

- `-fs` runs UxPlay in fullscreen mode.
- `-s2560x1440@30` sets the display resolution.
- `Restart=always` restarts the service if it stops.
- `DISPLAY=:0` sends output to the main display.

Make sure:

- Spacing between arguments is correct.
- `User=...` matches your Raspberry Pi username.

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
