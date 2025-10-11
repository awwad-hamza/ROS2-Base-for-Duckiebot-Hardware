# ROS2-Base-for-Duckiebot-Hardware
![Nvidia Logo](https://images.fastcompany.com/image/upload/f_auto,q_auto,c_fit,w_1024,h_1024/wp-cms-2/2024/06/i-2-91135380-nvidia-logo.jpg)
![Duckietown Logo](https://github.com/duckietown/duckietown-lx/raw/mooc2022/braitenberg/assets/images/dtlogo.png)

## 🦆 About

ROS2 Base for Duckiebot Hardware is a pre-configured ROS 2 Foxy environment optimized for the duckiebot DB21M robotics platform running on the NVIDIA Jetson Nano Developer Kit. It provides a ready-to-use setup that streamlines development, deployment, and experimentation with ROS 2 on embedded systems.

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

### (Optional) Set Up Wi-Fi on the Jetson
You can either connect your Jetson Nano directly to Wi-Fi or share your PC’s internet connection through an Ethernet bridge.
In this section, we’ll go through the Wi-Fi setup method, which is often simpler and ensures your Jetson can access the internet for installing packages, pulling Docker images, and running updates.
> Note: Note: Having an active internet connection on the Jetson is essential for most development tasks.

> Make sure the Wi-Fi dongle is connected to the jetson before this task.

Access your Jetson with a keyboard and monitor, or SSH into it over Ethernet.
>Note: nano is not installed by default on the Jetson, so you’ll need to install it first using: `sudo apt install nano`

```
nmcli radio wifi on
sudo nano /etc/NetworkManager/NetworkManager.conf
```
Look for the following section:
```
[ifupdown]
managed=false
```
If `managed=false` is set, change it to:
```
[ifupdown]
managed=true
```
Save and close the file (Ctrl + O, Enter, Ctrl + X), then restart the NetworkManager service:
```
sudo systemctl restart NetworkManager
```

### Connect to a Wi-Fi Network
To view available Wi-Fi networks:
```
nmcli device wifi list
```
Then connect to your desired network:
```
nmcli device wifi connect "SSID" password "your_password"
```
Replace `SSID` and `your_password` with your actual Wi-Fi network name and password.

> Note: Connecting to a university or public Wi-Fi network (such as `eduroam`), may not be as straightforward as this.


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




//////////////

next talk about how we make ros2 foxy work on the jetson after these steps