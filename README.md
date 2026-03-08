
# MacBook Screen Mirroring to Raspberry Pi using AirPlay (UxPlay)

This guide explains how to turn a Raspberry Pi into an AirPlay receiver so you can **mirror or extend your MacBook screen to the Raspberry Pi display** with Ux**UxPlay**.

After completing this setup:

MacBook → AirPlay → Raspberry Pi running UxPlay → HDMI Display

The Raspberry Pi will **automatically start as AirPlay receiver at boot**.

---

# Hardware Requirements

- Raspberry Pi 
- microSD card
- HDMI display connected to the Pi
- MacBook with AirPlay support
- Both devices connected to the **same network**

---

# Install Raspberry Pi OS

Download Raspberry Pi OS:

>https://www.raspberrypi.com/software/operating-systems/

Write the OS image to your SD card using:

- Raspberry Pi Imager
- balenaEtcher

Insert the card into the Raspberry Pi and boot.

---

# Connect to the Raspberry Pi

Open Terminal on the Raspberry Pi, or connect remotely using SSH.

```
ping 192.168.5.148
ssh cz@192.168.5.148
```
# **Install UxPlay**

Update packages:
```
sudo apt update
```

Install UxPlay:
```
sudo apt install uxplay
```

Project repository:
```
https://github.com/FDH2/UxPlay
```

# **Verify UxPlay Installation**
Check where uxplay is installed:
```
which uxplay
```


# **Create an Auto-Start Service**

To automatically start the AirPlay receiver when the Raspberry Pi boots, create a **systemd service**.

```
sudo nano /etc/systemd/system/uxplay.service
```
# **Service Configuration**
Paste the following configuration into the file.
```
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

Explanation

-   -fs runs UxPlay in fullscreen mode
    
-   -s2560x1440@30 sets display resolution
    
-   Restart=always restarts if the service stops
    
-   DISPLAY=:0 sends output to the main display


Make sure:

-   Spaces between arguments are correct
    
-   The username matches your system (User=username)

Save and Exit Nano

# **Reload systemd to apply the new service**
  
```
sudo systemctl daemon-reload
```
This tells systemd to read the new uxplay.service file.

# **Enable the service to start at boot**

  

Type:

  
```
sudo systemctl enable uxplay.service  
```
You should see messages about creating symlinks. This means the service is now set to start automatically on boot.

# **Start the service immediately (without rebooting)**

  

```
sudo systemctl start uxplay.service
```
 
  

This starts uxplay using the service configuration you created.


# **Check that the service is running**

  



  
```
systemctl status uxplay.service
```
  

  

Look for a line that says:

  

Active: active (running)

  

If you see that, it means uxplay is running correctly as a service.

  

To exit the status view, press the q key.

# **Reboot to confirm automatic start**

  

```

  

sudo reboot
```
  

Press Enter.

  

Your Raspberry Pi will restart. After it boots up, uxplay will automatically start with the command:

  
```
/usr/bin/uxplay -fs 
```

or 
```
/usr/bin/uxplay -fs -s2560x1440@30
  ```

You can confirm after reboot by opening Terminal again and typing:

  ```
systemctl status uxplay.service
```
  

You should again see:

  
```
Active: active (running)```
