# azure-iot-operations

### Step 1: Update Your System
Ensure your system is up-to-date.
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install the GNOME Desktop Environment
Install the GNOME desktop environment. You can choose the full desktop or a minimal install.

For a full GNOME installation:
```bash
sudo apt install ubuntu-desktop
```

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
   sudo ufw allow 3389/tcp
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

### Step 6: Connect via RDP
Use your preferred RDP client to connect to your Ubuntu VM:
- **Hostname**: The public IP address or DNS name of your Azure VM.
- **Username and Password**: The credentials of a user account on your Ubuntu VM.

### Additional Notes
- Make sure your Network Security Group (NSG) rules in Azure allow inbound traffic on port 3389 to your VM.
- You might need to handle specific GNOME dependencies or settings if you encounter issues. GNOME can be more resource-intensive than lighter desktop environments like XFCE or MATE, so ensure that your VM has sufficient resources.
- If you experience any problems, checking the xrdp logs found in `/var/log/xrdp` can be helpful for troubleshooting.
