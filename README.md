### WireGuard VPN Deployment Instructions

Follow these steps to deploy the WireGuard VPN server using the provided `docker-compose.yml` file:

---
###  **1. Enable IP Forwarding**
   WireGuard requires IP forwarding. Enable it permanently by modifying the system configuration:
   ```bash
   echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
   echo "net.ipv6.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

---

### **2. Set Up WireGuard with Docker Compose**

1. **Create a Deployment Directory**
   Create a directory to store the `docker-compose.yml` file and configuration:
   ```bash
   mkdir -p ~/wireguard && cd ~/wireguard
   ```

2. **Create the `docker-compose.yml` File**
   Save the provided `docker-compose.yml` file in the `~/wireguard` directory.

3. **Adjust File Permissions**
   Ensure the deployment directory and configuration files have the appropriate permissions:
   ```bash
   sudo chown -R $USER:$USER ~/wireguard
   ```

4. **Set the Environment Variables**
   - Replace `PUID` and `PGID` with your user and group IDs:
     ```bash
     id
     ```
     Use the values for `uid` and `gid` from the output.
   - Replace `SERVERURL` with your server’s public IP address or domain name.
   - Adjust `SERVERPORT` if necessary (default is `51820`).

#### Note: Change other settings in the docker-compose.yml as your needs.
---

### **3. Deploy the WireGuard Container for VPN Server**

1. **Launch the Container**
   Use Docker Compose to start the WireGuard server:
   ```bash
   docker-compose up -d
   ```

2. **Check the Container Status**
   Verify that the container is running:
   ```bash
   docker ps
   ```

3. **Inspect the Logs**
   Check the logs to ensure the WireGuard server has started correctly:
   ```bash
   docker logs wireguard
   ```

---

### **4. Retrieve Client Configurations**

1. **Access Generated Configs**
   The client configuration files will be located in the `config` directory inside the deployment directory (`~/wireguard/config/peerX/peerX.conf`).

2. **Download Configs**
   Use `scp` or another method to transfer the client configuration files to your client devices:
   ```bash
   scp ~/wireguard/config/peerX/peerX.conf user@client-machine:/etc/wireguard/
   ```
   Or just create a new file named `wg0.conf` or `peer.conf` and paste the context in there!

   ##### Note: Edit the AllowedIPs field in the configuration file. If you let it be 0.0.0.0 it will route all your traffic through VPN. If you only need it for maintenance purposes, change it to you VPN server IP range (e.g. 10.13.13.0/24).

---

### **5. Connect to the VPN**

1. **Install WireGuard on Clients**
   - For Linux:
     ```bash
     sudo apt install wireguard
     ```
   - For macOS:
     Install the WireGuard app from the App Store.
   - For Windows:
     Download and install the WireGuard client from [WireGuard’s website](https://www.wireguard.com/install/).

2. **Import the Configuration**
   Use the client configuration file (`peerX.conf`) to set up the connection on your device. This file should be in the `/etc/wireguard` directory.
   ```bash
   sudo wg-quick up peerX.conf
   ```
3. **Auto Startup at System Reboot (Necassary if you have access only through VPN to you systems.).**
   Just execute the command below to execute the client side (peer) VPN as a service:
   ```bash
   sudo systemctl enable wg-quick@peerX
   sudo systemctl start wg-quick@peerX
   ```

   #### Note: If a VPN is previously set, stop it using the command below:
   ```bash
   sudo wg-quick down peerX
   ```

5. **Check Connection to the VPN**
   ```bash
   sudo wg show
   ```

---

### **6. Verify the VPN Connection**

1. **Check the Connection**
   Once connected, confirm that your public IP matches the server’s public IP:
   ```bash
   curl ifconfig.me
   ```

2. **Test Internal Communication**
   Ensure you can communicate with other devices on the VPN, including the WireGuard server.

---

### **7. Manage and Monitor the Server**

1. **Stop the WireGuard Container**
   ```bash
   docker-compose down
   ```

2. **Restart the WireGuard Container**
   ```bash
   docker-compose up -d
   ```

3. **Update the WireGuard Image**
   To pull the latest WireGuard Docker image:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

---

You now have a fully deployed and operational WireGuard VPN server! 🎉
