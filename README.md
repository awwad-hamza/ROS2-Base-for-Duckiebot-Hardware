# ROS2-Base-for-Duckiebot-Hardware
![Nvidia Logo](https://images.fastcompany.com/image/upload/f_auto,q_auto,c_fit,w_1024,h_1024/wp-cms-2/2024/06/i-2-91135380-nvidia-logo.jpg)
![Duckietown Logo](https://github.com/duckietown/duckietown-lx/raw/mooc2022/braitenberg/assets/images/dtlogo.png)

## 🦆 About

ROS2 Base for Duckiebot Hardware is a pre-configured ROS 2 Foxy environment optimized for Duckiebot robotics platforms running on the NVIDIA Jetson Nano Developer Kit. It provides a ready-to-use setup that streamlines development, deployment, and experimentation with ROS 2 on embedded systems.


/////////////////////////////////
Go back to these later hamza

## Prerequisites

Jetson Nano 2 GB with JetPack 4.x (L4T R32.x) installed.

Docker installed (comes with JetPack).

NVIDIA container runtime available (installed by default).

Optional: USB or CSI camera (for camera passthrough).

//////////////////////////////////

## Step 1: Verify Docker and NVIDIA runtime
1. Check Docker version:
```
docker --version
```
It should display the installed Docker Engine version on your Jetson Nano.

2. Check available runtimes:
```
sudo docker info | grep Runtimes
```
It should display something like:
```
Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux nvidia runc
```
If `nvidia` is missing, fix it:
```
sudo apt-get update
sudo apt-get install -y nvidia-container-runtime
sudo nano /etc/docker/daemon.json
```
Add (or edit) the file to include:
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
Then restart Docker:
```
sudo systemctl restart docker
```

## Step 2: Step 2: Verify GPU devices on the host
```
ls /dev/nv*
```
- You should see multiple `nvhost-*` devices.
- This confirms the Jetson GPU is visible to Docker containers.




//////////////

next talk about how we make ros2 foxy work on the jetson after these steps