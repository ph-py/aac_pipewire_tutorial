### Tutorial: Enabling AAC for Bluetooth Headphones on Debian 13 (Trixie)

This guide explains how to recompile the PipeWire Bluetooth plugin with **AAC** support. In official Debian builds, AAC is often disabled due to licensing and dependency restrictions between the `main` and `non-free` repositories.

By following this procedure, the following profile will become available in GNOME or Pavucontrol:
`High Fidelity Playback (A2DP Sink, codec AAC)`

---

#### 1. Requirements and Context
This tutorial was tested on:
- **OS:** Debian GNU/Linux 13 (trixie)
- **Audio Server:** PipeWire 1.4.2-1
- **Hardware:** Bluetooth headphones identifying as "TUNED" (or similar) supporting AAC.

#### 2. Enable Source Repositories
To download the source code, you need `deb-src` entries in your sources list.

Open the file:
```bash
sudo nano /etc/apt/sources.list
```

Add these lines (adjust if you use different mirrors):
```text
deb-src http://deb.debian.org/debian/ trixie main contrib non-free non-free-firmware
```

Update your package index:
```bash
sudo apt update
```

#### 3. Install Build Tools and Dependencies
Install the necessary tools to compile software and the specific AAC development library:
```bash
sudo apt install build-essential devscripts
sudo apt build-dep pipewire
sudo apt install libfdk-aac2t64 libfdk-aac-dev
```

Confirm the runtime library is installed:
```bash
dpkg -s libfdk-aac2t64 | grep '^Status'
```

#### 4. Download PipeWire Source Code
Create a workspace and download the source (do this as a regular user, **not** root):
```bash
mkdir -p ~/pipewire-aac
cd ~/pipewire-aac
apt source pipewire
```

Enter the created directory:
```bash
cd pipewire-1.4.2*
```

#### 5. Modify Build Rules to Enable AAC

##### 5.1 Edit `debian/rules`
```bash
nano debian/rules
```
Find the line containing `-Dbluez5-codec-aac=disabled` and change it to:
```text
-Dbluez5-codec-aac=enabled
```

##### 5.2 Edit `debian/control`
```bash
nano debian/control
```
1. Find and **delete** the line: `Build-Conflicts: libfdk-aac-dev`
2. In the `Build-Depends:` section, add `libfdk-aac-dev,` to the list (e.g., after `pkg-config,`).

#### 6. Compile the Package
Run the build command. This may take several minutes:
```bash
debuild -b -uc -us
```

#### 7. Install the Recompiled Bluetooth Plugin
Once finished, go to the parent directory and install only the Bluetooth SPA plugin:
```bash
cd ..
sudo dpkg -i libspa-0.2-bluetooth_1.4.2*.deb
```

#### 8. Restart Services
Restart the user audio services for the changes to take effect:
```bash
systemctl --user restart wireplumber pipewire pipewire-pulse
```

---

### Maintenance and Best Practices

#### Prevent Overwriting (APT Hold)
To prevent official Debian updates from replacing your custom package with one that lacks AAC support:
```bash
sudo apt-mark hold libspa-0.2-bluetooth
```

#### Cleanup
You can remove the build tools used only for this process:
```bash
sudo apt remove libfdk-aac-dev devscripts build-essential
sudo apt autoremove
```

#### Save the .deb File
It is recommended to save the generated `.deb` file in a safe folder so you can reinstall it without recompiling if needed:
```bash
mkdir -p ~/custom-packages
cp ~/pipewire-aac/libspa-0.2-bluetooth_*.deb ~/custom-packages/
```

*Technical reference: Based on [AAC and Debian by Tookmund](https://tookmund.com/2024/02/aac-and-debian).*
