# ROS2-Base-for-Duckiebot-Hardware
![Nvidia Logo](https://images.fastcompany.com/image/upload/f_auto,q_auto,c_fit,w_1024,h_1024/wp-cms-2/2024/06/i-2-91135380-nvidia-logo.jpg)
![Duckietown Logo](https://github.com/duckietown/duckietown-lx/raw/mooc2022/braitenberg/assets/images/dtlogo.png)

## 🦆 About

ROS2 Base for Duckiebot Hardware is a pre-configured ROS 2 Humble environment optimized for the duckiebot DB21M robotics platform running on the NVIDIA Jetson Nano Developer Kit. It provides a ready-to-use setup that streamlines development, deployment, and experimentation with ROS 2 on embedded systems.

## SD Card Preparation for Jetson Nano Developer Kit

Follow the steps below to prepare your SD card and set up your Jetson Nano 2GB Developer Kit.

### 1. Download the Official SD Card Image
- Visit NVIDIA’s [official download page](https://developer.nvidia.com/embedded/downloads).
- Download the image file for the Jetson Nano 2GB Developer Kit.

### 2. Flash the Image to a microSD Card
- Use [Balena Etcher](https://etcher.balena.io/) or any other reliable flashing tool to write the downloaded image to a **new microSD card**.
- Ensure the flashing process completes successfully before removing the SD card.

### 3. Boot the Jetson Nano
- Insert the newly flashed microSD card into your Jetson Nano.
- Power on the Jetson Nano and follow the on-screen setup instructions.

### 4. System Configuration
To align your setup with the **Duckietown shell environment** (`duckie@HOSTNAME`):

- **Username:** enter `duckie` in the *Pick a username* field.  
- **Hostname:** choose a suitable hostname in the *Your computer’s name* field.  
- **Password:** use `quackquack` to match the default Duckietown (`dts`) setup.

> **Note:** These configuration choices are optional but recommended to maintain compatibility with Duckietown conventions for future use.

## PC–Jetson Communication Setup
This step will let you connect to your Jetson Nano from your PC using either Ethernet or Wi-Fi.
You’ll need a keyboard, screen, and (optionally) a mouse connected to the Jetson for this setup — at least until you can SSH into it remotely.
> Note: You can always operate the Jetson directly using a screen and keyboard, but setting up SSH makes development much more convenient.
### On the Jetson
1. Create a new Ethernet connection with a static IP
```
nmcli connection add type ethernet ifname eth0 con-name duckie-eth ip4 10.42.0.2/24 gw4 10.42.0.1
```
2. Set the DNS server for name resolution
```
nmcli connection modify duckie-eth ipv4.dns "8.8.8.8"
```
3. Bring the new Ethernet connection online
```
nmcli connection up duckie-eth
```
4. Enable automatic reconnection at boot
```
nmcli connection modify duckie-eth connection.autoconnect yes
```

### On Host PC
1. Before setting up the Ethernet connection, clear any previous SSH key entries associated with the Jetson’s IP (this can happen if you’ve reflashed the Jetson or changed its configuration):
```
ssh-keygen -f "$HOME/.ssh/known_hosts" -R "10.42.0.2"
```
This ensures SSH will not complain about changed host keys.

2. Next, configure a static IP on your PC’s Ethernet interface so it’s on the same subnet as the Jetson:
```
sudo nmcli connection add type ethernet ifname enp2s0 con-name jetson-link ip4 10.42.0.1/24
sudo nmcli connection up jetson-link
```
---
Once the connection is active, you can SSH into your Jetson from your PC using:
```
ssh duckie@10.42.0.2
```

## Networking
Having an active internet connection on the Jetson is essential for most development tasks.

### Option 1: Wi-Fi
> Make sure the Wi-Fi dongle is connected to the jetson before this task.
Access your Jetson with a keyboard and monitor, or SSH into it over Ethernet.

Turn on Wi-Fi:
```
nmcli radio wifi on
```
Ensure NetworkManager manages the interfaces:
```
sudo sed -i 's/managed=false/managed=true/' /etc/NetworkManager/NetworkManager.conf
```
Restart NetworkManager:
```
sudo systemctl restart NetworkManager
```

List available Wi-Fi networks:
```
nmcli device wifi list
```
Connect to your desired network:
```
nmcli device wifi connect "SSID" password "your_password"
```
Replace `SSID` and `your_password` with your actual Wi-Fi network name and password.

> Note: Connecting to enterprise or campus networks (e.g., `eduroam`) may require additional configuration files or authentication steps.
### Option 2: Ethernet Bridge (via your PC)
This method shares your computer’s internet connection with the Jetson over Ethernet.

#### On your pc

Enable IPv4 forwarding
```
sudo sysctl -w net.ipv4.ip_forward=1
```
Make it persistent:
```
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```
Add NAT (masquerade) rules to forward traffic from the Jetson (Ethernet) to your PC’s internet interface (`wlp3s0` = Wi-Fi, `enp2s0` = Ethernet):
```
sudo iptables -t nat -A POSTROUTING -o wlp3s0 -j MASQUERADE
sudo iptables -A FORWARD -i wlp3s0 -o enp2s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp2s0 -o wlp3s0 -j ACCEPT
```
Make the rules persistent
```
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```
#### On your Jetson
Restart the network interface:
```
sudo dhclient -r eth0
sudo dhclient eth0
```
Test internet connectivity:
```
ping -c 4 8.8.8.8
```
You should see replies similar to:
```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=XX ms
```
If that works, you’re online, You can then verify DNS resolution:
```
ping -c 4 google.com
```
## Docker Installation
Follow the steps below to install and verify Docker on your Jetson Nano.
### Install Docker on the Jetson
```
sudo apt-get update
sudo apt-get install docker.io
sudo systemctl enable docker
sudo usermod -aG docker $USER
sudo reboot
```
Verify the Installation
```
docker --version
```
You should see the installed **Docker Engine version** displayed on your Jetson Nano.
To confirm Docker is working properly, run the Hello World container:
```
docker run hello-world
```
If it runs successfully, Docker is correctly installed and functioning.

### Verify NVIDIA runtine
Check whether the NVIDIA runtime is available:
```
sudo docker info | grep Runtimes
```
It should display an output similar to:
```
Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux nvidia runc
```
If `nvidia` is missing, follow the steps below to fix it:
```
sudo apt-get update
sudo apt-get install -y nvidia-container-runtime
sudo nano /etc/docker/daemon.json
```
Update the file contents as follows:
```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia"
}
```
Save the file, then restart the Docker service:
```
sudo systemctl restart docker
```

## Verify GPU devices on the host
```
ls /dev/nv*
```
- You should see multiple `nvhost-*` devices.
- This confirms the Jetson GPU is visible to Docker containers.


## ROS 2 Humble Base Container on the Jetson
To keep your Docker container immutable while preserving your ROS 2 work outside the container, you can use a host-mounted workspace.
1. Create a workspace on the Jetson host
```
mkdir -p ~/ros_ws/src
```
This folder will store all your ROS 2 packages, builds, and installations.
2. Run the ROS 2 Humble container with the mounted workspace
```
sudo docker run -it --rm \
  --runtime nvidia \
  --network host \
  -v ~/ros2_ws:/workspace/ros2_ws \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  dustynv/ros:humble-ros-base-l4t-r32.7.1
```
3. Work inside the container
```
cd /workspace
colcon build
source install/setup.bash
```
- Any changes, packages, or builds inside /workspace persist on the host, even after the container is removed.

- The container remains clean and reusable, while your development work is safely stored outside.
### Re-accessing the Docker Container on Jetson
You can easily re-access your Docker container by either rerunning the previous run command or by creating a convenient alias.

Open your Bash configuration
```
nano ~/.bashrc
```
Scroll to the end of the file and add an alias for your container (for example, `openws`):
```
alias openws="sudo docker run -it --rm --runtime nvidia --network host -v ~/ros2_ws:/workspace/ros2_ws -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY dustynv/ros:humble-ros-base-l4t-r32.7.1"
```
Apply the changes
```
source ~/.bashrc
```
You can now access your container anytime by simply running: `openws`.