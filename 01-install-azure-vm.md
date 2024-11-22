# azure-iot-operations

### Step 1: Update Your System
Ensure your system is up-to-date.
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install the GNOME Desktop Environment
Install the GNOME desktop environment. You can choose the full desktop or a minimal install.

For a minimal GNOME installation:
```bash
sudo apt install gnome-core
```

### Step 3: Install and Configure xrdp
1. **Install xrdp**:
   ```bash
   sudo apt install xrdp
   ```

2. **Start and enable xrdp**:
   ```bash
   sudo systemctl enable xrdp
   sudo systemctl start xrdp
   ```

3. **Allow xrdp through the firewall**:
   ```bash
   sudo ufw disable
   ```

### Step 4: Configure xrdp to Use GNOME
1. Create or edit the file `~/.xsession` and set it to start GNOME:
   ```bash
   echo gnome-session > ~/.xsession
   ```

2. Configure the xrdp session to start GNOME. This might require editing the `/etc/xrdp/startwm.sh` script:
   - Open `startwm.sh` for editing:
     ```bash
     sudo nano /etc/xrdp/startwm.sh
     ```

   - Comment out any session commands (such as those meant for other desktop environments) and add `gnome-session` before `./.xsession-default` if needed.

     ```bash
     # Any existing commands (e.g., for environments like MATE or XFCE)
     # test -x /etc/X11/Xsession && exec /etc/X11/Xsession
     gnome-session
     ```

### Step 5: Restart xrdp Service
Restart the xrdp service to apply your changes.
```bash
sudo systemctl restart xrdp
```

### Step 6: Install `az` CLI
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
The latest version of the connectedk8s extension for Azure CLI
```bash
az extension add --upgrade --name connectedk8s
```
