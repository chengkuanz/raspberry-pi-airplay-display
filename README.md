# MacBook Screen Mirroring to Raspberry Pi Using AirPlay (UxPlay)

This guide shows how to configuring a Raspberry Pi 4 as an AirPlay receiver using UxPlay to mirror a MacBook, iPhone, or iPad to an HDMI display, effectively acting as a wireless external monitor. The Raspberry Pi will start the AirPlay receiver automatically at boot.

<img width="1470" height="635" alt="Screenshot 2026-03-08 at 5 08 34 PM copy 2" src="https://github.com/user-attachments/assets/3b24e35e-525b-42e0-9daf-de169c618afd" />

## Install and Connect to the Raspberry Pi 

Download Raspberry Pi OS; Use `Raspberry Pi Imager` or `balenaEtcher` to write the OS image to your microSD card. Insert the card into the Raspberry Pi and power it on.

Open Terminal on the Raspberry Pi, or connect remotely over SSH: 
`ssh <PI_USER>@<PI_IP_ADDRESS>`

## Install UxPlay

``` 
sudo apt update
sudo apt install uxplay
``` 

Project repository: <https://github.com/FDH2/UxPlay>

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


<img width="783" height="357" alt="Screenshot 2026-03-08 at 7 29 17 PM" src="https://github.com/user-attachments/assets/5598ece4-8662-42fd-84cb-34fae908a84c" />



```ini
[Unit]
Description=UxPlay AirPlay Receiver
After=network-online.target
Wants=network-online.target

[Service]
# Fullscreen with explicit resolution/FPS
ExecStart=/usr/bin/uxplay -fs -s<WIDTHxHEIGHT@FPS>

Restart=always
RestartSec=5
User=<PI_USER>
Environment=DISPLAY=:0

[Install]
WantedBy=multi-user.target
```

Notes:

- `<PI_USER>`: your Raspberry Pi username (for example, `cz` or `pi`).
- `<WIDTHxHEIGHT@FPS>`: optional, your target output mode (for example, `1920x1080@60`, `2560x1440@30`).
- `-fs` runs UxPlay in fullscreen mode.
- `Restart=always` restarts the service if it stops.
- `DISPLAY=:0` sends output to the main display.


## Reload systemd

```bash
sudo systemctl daemon-reload
```

This tells `systemd` to read the new `uxplay.service` file.

## Enable Service at Boot

```bash
sudo systemctl enable uxplay.service
```


## Start the Service Configuration Now (Without Rebooting)

```bash
sudo systemctl start uxplay.service
```

## Check Service Status

```bash
systemctl status uxplay.service
```

Look for:
`
Active: active (running)
`

If you see that, UxPlay is running correctly as a service.

<img width="788" height="171" alt="image" src="https://github.com/user-attachments/assets/beeee4a1-cd0f-4927-b39a-fa66cd08b567" />


## Reboot to Confirm Automatic Start

```bash
sudo reboot
```

After reboot, UxPlay should start automatically. Check Service Status if necessary. 




<details>
  <summary>Disable and stop the service</summary>

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

</details>

## Prerequisites

- Raspberry Pi 
- MacBook, iPhone, or iPad with AirPlay support
- Both devices connected to the same Wi-Fi network
- HDMI display connected to the Raspberry Pi


## References

- [UxPlay project ](https://github.com/FDH2/UxPlay "UxPlay GitHub repository")
- [systemd service documentation](https://wiki.debian.org/systemd/Services "Debian systemd Services documentation")
