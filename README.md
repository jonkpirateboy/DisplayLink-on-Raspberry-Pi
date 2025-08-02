# DisplayLink on Raspberry Pi 5

This guide assumes you're using a DisplayLink USB adapter and want to get it working with a Raspberry Pi 5 running Raspberry Pi OS (Bookworm, 64-bit). It includes building and installing the EVDI module, setting up DisplayLinkManager, and ensuring everything works on boot.

## 1. System Requirements

- Raspberry Pi 5
- Raspberry Pi OS Bookworm 64-bit (with desktop)
- DisplayLink-based USB display

## 2. Enable Kernel Mode Setting (KMS) for Display Output

Edit `/boot/firmware/config.txt` and ensure:

```ini
dtoverlay=vc4-kms-v3d
max_framebuffers=2
gpu_mem=128
```

Reboot after editing:

```bash
sudo reboot
```

## 3. Enable GUI Login (Autologin)

From SSH, run:

```bash
sudo raspi-config
```

Navigate to:

```
1 System Options
  └─ S5 Boot
      └─ B2 Desktop Desktop GUI
1 System Options
  └─ S6 Auto Login
      └─ Yes / Yes
Finish
Reboot
```

## 4. Install Dependencies

```bash
sudo apt update
sudo apt install git dkms raspberrypi-kernel-headers build-essential libdrm-dev libusb-1.0-0-dev pkg-config x11-xserver-utils unzip
```

## 5. Clone and Build EVDI

```bash
git clone https://github.com/DisplayLink/evdi.git
cd evdi
make -C module CFLAGS_MODULE="-Wno-error"
sudo make -C module install
sudo depmod -a
```

## 6. Load the EVDI Module

```bash
sudo modprobe evdi
```

Confirm it loaded:

```bash
lsmod | grep evdi
```

To force it to load at boot:

```bash
echo evdi | sudo tee -a /etc/modules
```

## 7. Install DisplayLink Driver

Get the driver from [synaptics](https://www.synaptics.com/products/displaylink-graphics/downloads/ubuntu) with apt.

```bash
wget https://www.synaptics.com/sites/default/files/Ubuntu/pool/stable/main/all/synaptics-repository-keyring.deb
sudo dpkg -i synaptics-repository-keyring.deb
sudo apt update
sudo apt install displaylink-driver
```

## 8. Fix Session Type If Broken

The DisplayLink driver might reset your default desktop session. Run:

```bash
echo -e "[Desktop]\nSession=LXDE-pi-x" > ~/.dmrc
chmod 644 ~/.dmrc
```

## 9. Configure LightDM to Autologin and Run Setup Script

Edit the LightDM config:

```bash
sudo nano /etc/lightdm/lightdm.conf
```

Ensure the `[Seat:*]` section contains and change `[yourusername]` to your username:

```ini
[Seat:*]
greeter-session=pi-greeter
greeter-hide-users=false
user-session=LXDE-pi-x
display-setup-script=/usr/share/dispsetup.sh
autologin-user=[yourusername]
autologin-session=LXDE-pi-x
```

## 10. Create DisplayLink Display Setup Script

Create the script that sets the display resolution:

```bash
nano ~/.displaylink-setup.sh
```

Paste:

```bash
#!/bin/bash
xrandr --output DVI-I-1-1 --mode 1024x600 --primary
```

Make it executable:

```bash
chmod +x ~/.displaylink-setup.sh
```

## 11. Autostart the DisplayLink Setup Script

```bash
mkdir -p ~/.config/autostart
nano ~/.config/autostart/displaylink.desktop
```

Paste and change `[yourusername]` to your username:

```ini
[Desktop Entry]
Type=Application
Exec=/home/[yourusername]/.displaylink-setup.sh
Hidden=false
X-GNOME-Autostart-enabled=true
Name=DisplayLink Setup
Comment=Force screen mode for DisplayLink output
```

Make it executable:

```bash
chmod +x ~/.config/autostart/displaylink.desktop
```

## 12. Reapply Auto Login

Re-run `raspi-config` and ensure auto login is enabled **after** all previous steps:

```bash
sudo raspi-config
```

Navigate to:

```
1 System Options
  └─ S6 Auto Login
      └─ Yes / Yes
Finish
Reboot
```

## Finished!

The screen should now be working! :)
