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