## Setting Up a Kubernetes Cluster on Raspberry Pis Using k3s
### Prerequisites
- **Raspberry Pi 4 units** (at least 2 for a simple cluster)
- **MicroSD cards** for each Raspberry Pi
- **Router** with DHCP for initial setup, later configured with static IPs

### Software and Tools Required
1. **Raspberry Pi OS Lite 64Bit**: Download the OS image from [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

### Installation Steps

#### 1. Flash Raspberry Pi OS
- Download and install Raspberry Pi OS Lite 64Bit on each SD card using Raspberry Pi Imager.
- Make sure to user Raspberry Pi Imager to set the User's credentials, the computer's Hostname, the Time Zone, and enable SSH.

#### 2. Update All the things. (Optional)
- Make sure to update and upgrade the sodtware and rasp-condig tool if necessary.
- ```bash
  sudo apt-get update && sudo apt-get upgrade -y
  ```

#### 3. Configure Static IPs (Optional)
- This step is crucial for stable cluster management.
- Edit the `dhcpcd.conf` on each Raspberry Pi to assign static IP addresses:
  ```bash
  sudo nano /etc/dhcpcd.conf
  ```
- Add the following configuration for each node:
  ```bash
  interface eth0
  static ip_address=192.168.0.XX/24
  static routers=192.168.1
  static domain_name_servers=192.168.0.1 1.1.1.1 8.8.8.8
  ```
- Replace `192.168.0.XX` with the desired static IP.

#### 4. Disable Swap
- Turn off swap temporary:
  ```bash
  sudo swapoff -a
  ```
  - To turn of swap permanently we need to update the `CONF_SWAPSIZE` in `dphys-swapfile` file to `0`:
   ```bash
   sudo nano /etc/dphys-swapfile
   ```
   - Set the CONF_SWAPSIZE to 0 so it looks like below and save.
     ```bash
     CONF_SWAPSIZE=0
     ```

#### 5. Update `/boot/firmware/cmdline.txt`
- Adjust cgroup settings required by Kubernetes:
  ```bash
  sudo nano /boot/firmware/cmdline.txt
  ```
- Append the following at the end of the line:
  ```
  cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
  ```
- Save changes and reboot.

#### 6. Install k3s on Master Node
- Run the installation script with necessary parameters:
  ```bash
  curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--disable=servicelb"  sh -s -
  ```

#### 7. Join Worker Nodes
- Obtain the node token from the master:
  ```bash
  sudo cat /var/lib/rancher/k3s/server/node-token
  ```
- Use the token to join worker nodes:
  ```bash
  curl -sfL https://get.k3s.io | K3S_URL=https://192.168.0.5:6443 K3S_TOKEN=<YOUR-NODE-TOKEN> sh -
  ```
