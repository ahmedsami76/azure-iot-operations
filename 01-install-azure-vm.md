# azure-iot-operations

### Step 1: Create Resource Group
1. Log in to Azure portal and open Azure Shell
2. Run the below AZ Cli command to create a resource group
NOTE: The below command randomizes the name of the resource group but you can fix your own 
```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="rg-aio-$RANDOM_ID"
export REGION=WestUS2
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION
```

### Step 2: Create Azure Linux VM

1. Create a Ubuntu 24.04 vm with a minimum of 32GB RAM and make sure it supports nested virtualization (ex, Standard_D8s_v3)
2. Enable/request JIT access to port 22 on the VM

### Step 3: Update Your System
Ensure your system is up-to-date.
1. Use a terminal to connect to the VM using ssh (ex, Azure shell)
2. Once logged in to the VM, run the following command
```bash
sudo apt update && sudo apt upgrade -y
```

### (OPTIONAL) Step 4: Install the GNOME Desktop Environment
Install the GNOME desktop environment. You can choose the full desktop or a minimal install.

For a minimal GNOME installation:
```bash
sudo apt install gnome-core
```

### (OPTIONAL) Step 5: Install and Configure xrdp
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

### (OPTIONAL) Step 6: Configure xrdp to Use GNOME
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

### (OPTIONAL) Step 7: Restart xrdp Service
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

### Step 7: Disable UFW if installed
```bash
sudo ufw disable
```