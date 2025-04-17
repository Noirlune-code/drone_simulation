# Drone_simulation Environment setup

This repository helps you set up a simulation environment for drone simulations in your projects. It can also be used for other projects involving simple robots, but it is primarily focused on drone simulation.  
We will be setting up our environment on **Ubuntu 22.04.5 LTS** - Jammy.  
ROS 2 Humble works best with this Ubuntu distro, so we will be using that. Gazebo Fortress is recommended with this ROS version, but we also need the ardupilot_gazebo plugin, which is not available for Gazebo Fortress. Therefore, we will use Gazebo Harmonic.

## Prerequisites
- Ubuntu 22.04.5 LTS  
**Note:** Gazebo Harmonic may not run in virtual machines, so dual-boot is recommended. If a virtual machine is the only option, one should stick to ROS 1 and Gazebo Classic.

## Step 1 - ROS 2 installation
You can refer detailed installation from this guide : https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html .
Steps for Ubuntu are mentioned here only. 

### Check for locale
Ros 2 humble needs a locale that supports UTF-8 encoding.
Run:
```
locale
```
Using GPT just verify once if your output is UTF-8 supported or not. Most probably it would but it is good to make sure.

⚠️ **If it is not supported then run these commands to set locale to UTF-8.**
```
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```
### Setup sources
You will need to add the ROS 2 apt repository to your system.

First ensure that the Ubuntu Universe repository is enabled.

```
sudo apt install software-properties-common
sudo add-apt-repository universe
```
Now add the ROS 2 GPG key with apt.
```
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

Then add the repository to your sources list.
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### Install ROS 2 packages
Update your apt repository caches after setting up the repositories.
```
sudo apt update
```
ROS 2 packages are built on frequently updated Ubuntu systems. It is always recommended that you ensure your system is up to date before installing new packages.
```
sudo apt upgrade
```

Desktop Install (Recommended): ROS, RViz, demos, tutorials.
```
sudo apt install ros-humble-desktop
```

### Environment setup
Place source line in the `.bashrc` file so that you do not have to source it everytime.
```
echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
```
Reload using
```
source ~/.bashrc
```

### Verify installation
#### Talker-Listener example

In one terminal, source the setup file and then run a C++ talker:
```
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```
In **another** terminal source the setup file and then run a Python listener:

```
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_py listener
```
You should see the talker saying that it’s Publishing messages and the listener saying I heard those messages. This verifies both the C++ and Python APIs are working properly. Hooray!


## Step 2 : Gazebo installation and integration with ROS 2 Humble
We would be installing Gazbo harmonic due to the reasons mentioned earlier.
Refer to : https://gazebosim.org/docs/latest/ros_installation/ for detailed guide.

### Install Gazebo Harmonic
First install some necessary tools:
```
sudo apt-get update
sudo apt-get install curl lsb-release gnupg
```
Then install Gazebo Harmonic:
```
sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
sudo apt-get update
sudo apt-get install gz-harmonic
```
All libraries should be ready to use and the gz sim app ready to be executed.

### Verify installation
Run:
```
gz sim
```
You would see a gazebo harmonic window. 

### Integration with ROS

Install ros_gz from the non official binary packages from apt:
```
apt-get install ros-humble-ros-gzharmonic
```

### Verify integration
Run:
```
ros2 launch ros_gz_sim_demos camera.launch.py
```
Gazebo would launch with some spheres and cubes in the environment. 
**If the GUI fails to load close and re-run the command**


In a new terminal, check the available ROS 2 topics:
```
ros2 topic list
```

You should see topics like /camera, /clock, and other related topics.

Try viewing the camera feed:
```
ros2 run rqt_image_view rqt_image_view
```
If everything is working correctly, you'll be able to **select the camera topic from the dropdown menu** and see the video feed from Gazebo.


## Step 3 : Install Ardupilot SITL and integration with gazebo
For detailed installation guide of Ardupilot SITL refer to: https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html

For detailed setup of Ardupilot_gazebo refer to: https://github.com/ArduPilot/ardupilot_gazebo

### Setup Ardupilot

#### Install git 
```
sudo apt-get update && sudo apt-get install git
```

####Clone the repository
```
cd ~
git clone --recurse-submodules https://github.com/your-github-userid/ardupilot
cd ardupilot
```
Just run the bash script and all processing will be done.
```
Tools/environment_install/install-prereqs-ubuntu.sh -y
```

Reload the path (log-out and log-in to make it permanent):
```
. ~/.profile
```
### Verify Installation
```
cd ~/ardupilot
sim_vehicle.py -v copter --console --map -w
```
### Setup Arudpilot_gazebo module
#### Add some dependencies

Gazebo Harmonic Dependencies:
```
sudo apt update
sudo apt install libgz-sim8-dev rapidjson-dev
sudo apt install libopencv-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl
```

#### Clone the repo and build:

```
cd ~
git clone https://github.com/ArduPilot/ardupilot_gazebo
cd ardupilot_gazebo
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
```

#### Setup environment variables
```
export GZ_SIM_SYSTEM_PLUGIN_PATH=$HOME/ardupilot_gazebo/build:$GZ_SIM_SYSTEM_PLUGIN_PATH
export GZ_SIM_RESOURCE_PATH=$HOME/ardupilot_gazebo/models:$HOME/ardupilot_gazebo/worlds:$GZ_SIM_RESOURCE_PATH
```

Reload your terminal with 
```
source ~/.bashrc
```

### Verify Integration with gazebo

#### Iris quad-copter
Run Gazebo
```
gz sim -v4 -r iris_runway.sdf
```
The -v4 parameter is not mandatory, it shows additional information and is useful for troubleshooting.

Run ArduPilot SITL
To run an ArduPilot simulation with Gazebo, the frame should have gazebo- in it and have JSON as model. Other commandline parameters are the same as usual on SITL.
```
sim_vehicle.py -v ArduCopter -f gazebo-iris --model JSON --map --console
```

Arm and takeoff.
Run the commands in the SITL_window
```
STABILIZE> mode guided
GUIDED> arm throttle
GUIDED> takeoff 5
```
Drone in gazebo will takeoff and fly. **If it does not then reload try the verification again.**


## Step 4: Mission Planner 
Mission Planner is not natively supported in LINUX so we would use mono to run it as on virtual machine.
Mission Planner does run under MONO but will have occasional issues and/or crashes.
You can use other softwares like QGroundControl that are directly available for linux.
For detailed setup refer to: https://ardupilot.org/planner/docs/mission-planner-installation.html

### Install mono

This method is to download the Mono from its official repository, for which we will first import the repository key:
```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
```
The next step is to add the repository to the list of our Ubuntu 22.04 repositories by using the command:
```
$ echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list
```
Third step is to update the repository of the Ubuntu 22.04 using the update command:
```
$ sudo apt update
```
Finally, we will install the Mono again using the apt package manager:
```
$ sudo apt install mono-complete -y
```
To check installation and the version of installed Mono:
```
$ mono --version
```

### Download Mission Planner
Download the files for Mission Planner from this link: 
https://firmware.ardupilot.org/Tools/MissionPlanner/MissionPlanner-latest.zip

Unzip the files to a directory and then locate to the directory.
And then Run:
```
mono MissionPlanner.exe
```
