---
title: 'Win11 Ros2 & isaacsim bridge'
date: 2025-07-11
permalink: /posts//Ros2&isaacsim_bridge
tags:
  - isaacsim
  - ros2
---

Installing ROS 2 natively on Windows 11 can be complex and error-prone due to compatibility and dependency issues.
To simplify the setup and ensure smooth communication with Isaac Sim, I use WSL (Windows Subsystem for Linux) to run an Ubuntu 22.04 environment and install ROS 2 Humble inside it.

This setup allows seamless integration with Isaac Sim’s ROS 2 bridge, which supports ROS 2 Humble natively.

This guide assumes that you have already downloaded and installed Isaac Sim from the official GitHub release.


Install wsl and ubuntu22.04
======

Install wsl and ubuntu, it may take a while.
```powershell
wsl --install
# You can check available version, recommand 22.04
wsl --list --online
wsl --install -d Ubuntu-22.04
```

You need restart your computer!!!

Now, simply search for "Ubuntu" in the Start Menu and run it.
<img src="../images/posts/Launch_Ubuntu.png" alt="Search Ubuntu in Windows Search" width="600"/>

When you open Ubuntu for the first time, you'll be prompted to create a UNIX user account:
```bash
Installing, this may take a few minutes...
Please create a default UNIX user account:
Enter new UNIX username:
```
After setting your username and password, you'll be able to use Ubuntu through the terminal.

To open a new Ubuntu terminal in the future, simply search for "Ubuntu" in the Start Menu and run it again.


Install ROS2 in Ubuntu22.04
======

You just need to follow the instuction:

https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html


## 1. Set Local

```bash
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

## 2. Setup Sources

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

```bash
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb" # If using Ubuntu derivates use $UBUNTU_CODENAME
sudo dpkg -i /tmp/ros2-apt-source.deb
```

## 3. Install ROS 2 packages

```bash
sudo apt update
sudo apt install ros-humble-desktop
```

## 4. You can also install

ROS-Base Install (Bare Bones): Communication libraries, message packages, command line tools. No GUI tools.

```bash
sudo apt install ros-humble-ros-base
```

Development tools: Compilers and other tools to build ROS packages

```bash
sudo apt install ros-dev-tools
```

## 5. Auto load ROS when open terminal

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

## 6. Test your ros

In one terminal, source the setup file and then run a C++ talker:

```bash
#terminal 1
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```

In another terminal source the setup file and then run a Python listener：

```bash
#terminal 2
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_py listener
```

Windows network settings
======
After ROS 2 installation is complete, open WSL2 and run the following command to get the IP address of WSL2:

```bash
#Ubuntu terminal
hostname -I
```

Open Powershell as Admin and run the following command and retrieve the IPv4 address of the Windows host:

```bash
#Windows powershell
ipconfig /all
```

Set the variables in Powershell accordingly with the respective IP addresses:

```bash
#Windows powershell
$Windows_IP = "<WINDOWS_IP>"
$WSL2_IP = "<WSL2_IP>"
```

Setup port forwarding in Powershell for the specific ports used by default DDS (FastDDS) in ROS:

```bash
#Windows powershell
netsh interface portproxy add v4tov4 listenport=7400 listenaddress=$Windows_IP connectport=7400 connectaddress=$WSL2_IP
netsh interface portproxy add v4tov4 listenport=7410 listenaddress=$Windows_IP connectport=7410 connectaddress=$WSL2_IP
netsh interface portproxy add v4tov4 listenport=9387 listenaddress=$Windows_IP connectport=9387 connectaddress=$WSL2_IP
```

Enable specific internal ROS 2 libraries
======

Windows powershell
```bash
# Set environment variables
$env:isaac_sim_package_path = "cd PATH_TO_ISAAC_SIM"
$env:ROS_DISTRO = "humble"
$env:RMW_IMPLEMENTATION = "rmw_fastrtps_cpp"

# Only set this !!ONCE!! per session to avoid path conflicts
$env:PATH = "$env:PATH;$env:isaac_sim_package_path\exts\isaacsim.ros2.bridge\humble\lib"

# Run Isaac Sim with ROS 2 Bridge Enabled
& "$env:isaac_sim_package_path\isaac-sim.bat" --/isaac/startup/ros_bridge_extension=isaacsim.ros2.bridge
```

<img src="../images/posts/ROS2_Bridge_Enabled.png" alt="Search Ubuntu in Windows Search" width="800"/>
