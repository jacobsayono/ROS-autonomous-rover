# ROS Architecture

Upgraded to ROS 2 Humble on Ubuntu 22.04 LTS.
Feb 1st, 2024

## For University Capstone Project:

Shell scripts (navigation logic & robotic mechanism control): http://github.com/jacobsayono/capstone

ROS 2 custom package (line following using computer vision): http://github.com/jacobsayono/line_follower

## For DevX BruinBot:

![BruinBot Simulation](assets/bruinbot_simulation.png)
![BruinBot Simulation](assets/bruinbot_sim.png)
![BruinBot Simulation](assets/bruinbot_slam.png)
![BruinBot Simulation](assets/bruinbot_nav2.png)
![BruinBot Simulation](assets/bruinbot_amcl.png)

```
sudo apt-get install ros-humble-gazebo-ros-pkgs
sudo apt install ros-humble-gazebo-ros2-control
sudo apt install python3-colcon-common-extensions
sudo apt install ros-humble-xacro
sudo apt install ros-humble-joint-state-publisher-gui
sudo apt install ros-humble-image-transport-plugins
sudo apt install ros-humble-rqt-image-view
sudo apt install ros-humble-ros2-control
sudo apt install ros-humble-ros2-controllers
sudo apt install ros-humble-slam-toolbox
sudo apt install ros-humble-navigation2
sudo apt install ros-humble-nav2-bringup
sudo apt install ros-humble-twist-mux
sudo apt install joystick jstest-gtk evtest
```
#### Note that gazebo in humble is not available in arm64. Only amd64 at the moment (as of Feb 2024). This means that your PC should be running on amd or intel chip (old macbooks have them).

Check out this link for available ROS packages:

http://packages.ros.org/ros2/ubuntu/pool/main/r/

~~TODO: Put these in requirements.txt for robot version~~

## Run this command instead if you are cloning repo into physical robot (Raspberry Pi)
```
sudo apt update && cat requirements.txt | xargs sudo apt install -y
```
This is basically the same as above, but we don't install gazebo on the RPi since we will be doing visualization on our development PC.

Go back to robot_ws, source humble and this repo (i.e. source /opt/ros/humble/setup.bash && source install/setup.bash), and `colcon build --symlink-install`

```
robot_ws/
    |___src/
        |___a-ros/  (cloned this repo)
```

## Raspberry Pi commands for system checks
Under-voltage/over-throttling feedback: `sudo vcgencmd get_throttled`

Temperature feedback: `sudo vcgencmd measure_temp`

## SSH to Raspberry Pi
Wifi auto connect config stored here in Raspberry Pi. Change the ssid and psk to your own router. This file can also be newly added in /boot when you etch the RPi imager the first time(?):

`/etc/wpa_supplicant/wpa_supplicant.conf`

Some useful commands to figure out your ipaddress or devices connected to the network. Try whichever:
```
ifconfig
ip addr
arp -a
nmap -sn 192.168.1.0/24
```
(or whatever the ipaddress is)

SSH into RPi:

`ssh pi@172.20.10.4` (or whatever the ipaddress is)

RPi Password:

`raspberry`

Safe shutdown via terminal:

`sudo shutdown -h now`

## Serial Comms (Arduino-RPi) + VSCode SSH & Extension
```
sudo adduser $USER dialout
sudo apt install python3-serial
sudo snap install arduino-cli
```
Reboot RPi. On Dev machine, install VSCode SSH extension and connect to host pi@172.20.10.4 and then type in the Pi's password.

Install the VSCode's Arduino extension (which will be on the RPi). Then click on the extension settings, click on Remote[SSH:172.20.10.4] tab at the top (next to User), and click the check box for `Arduino: Use Arduino Cli`.

Create an arduino sketch (e.g. Blink.ino or check the section after this), and make sure the file is the same name as the directory (just how Arduino works I suppose...)

Bottom right corner: select port (there should be arduino note; in my case it was /dev/ttyACM0), select board (install arduino avr boards), select programmer (I chose AVRISP mkII). Verify & upload code.

In .vscode/arduino.json you can add `"output": "build"` below the list (don't forget a comma before the last item since you're adding a new item now) to speed up compile time in the future.

## ROS-Arduino Bridge
The purpose of this code is to flash some code from the RPi to the arduino so that it can read and write to the motor driver properly.

To test, go to your home directory of your robot_ws (via SSH), and clone this repo:

`git clone https://github.com/joshnewans/ros_arduino_bridge.git`

Make sure you flash the ROSArduinoBridge to your arduino (upload the .ino code).

Note: if you are using an `Arduino Nano` instead and there are some issues, try to `sudo apt-get remove brltty` that may be colliding with the USB port to pick up the nano. Furthermore, when selecting the board, you may need to pick the old bootloader processor. The port name would be `ttyUSB0` too.

Connect your motor driver to the arduino and check to see if the RPi can report motor states and command it.

`python3 -m serial.tools.miniterm -e /dev/ttyACM0 57600`

or

`python3 -m serial.tools.miniterm -e /dev/ttyUSB0 57600`

o <PWM1> <PWM2>: set raw PWM speed of each motor (-255 to 255)
e: motor responds with current encoder counts for each motor
r: reset encoder values
m <Speed1> <Speed2>: set closed-loop speed of each motor in *counts per loop* (default loop rate is 30), so `(counts per sec) / 30`

Try rotating the motor in one full revolution by hand, and record encoder counts per revolution. We got a result of 1950, so 1950/30 = 65.

E.g.:
`m 65 65` should rotate the motor at a speed of one revolution per second.

Go to your robot_ws/src/ directory and clone a demo package to test serial connection for motor control (do this for both dev and robot workspaces):

`git clone https://github.com/joshnewans/serial_motor_demo`

Note: Remove setup warning triggers by running this command (downgrade version):

`sudo apt install python3-pip`
`pip install setuptools==58.2.0`

Now build:

`cd .. && colcon build --symlink-install`

Test ROS2, driver is listening to a topic for motor speeds:

`ros2 run serial_motor_demo driver --ros-args -p serial_port:=/dev/ttyACM0 -p baud_rate:=57600 -p loop_rate:=30 -p encoder_cpr:=1950`

or

`ros2 run serial_motor_demo driver --ros-args -p serial_port:=/dev/ttyUSB0 -p baud_rate:=57600 -p loop_rate:=30 -p encoder_cpr:=1950`

On Dev machine:

`ros2 run serial_motor_demo gui`

Play around with motors.

## Tele-op with Joystick Controller

Clone these into robot_ws/src:

https://github.com/jacobsayono/diffdrive_arduino.git

https://github.com/jacobsayono/serial.git

Make sure RPi (robot_ws) has `sudo apt install ros-humble-ros2-control ros-humble-ros2-controllers`

You may have to do this but probably not: `sudo apt install ros-humble-hardware-interface`

*Make sure to change diffdrive_arduino to the Humble branch*

`sudo apt-get install libserial-dev`

`colcon build --symlink-install` should work!

## 2023 Notes and Progress

We use ROS2 Foxy on Ubuntu 20.04 LTS.

Dynamic Transforms:
```
sudo apt install ros-foxy-xacro ros-foxy-joint-state-publisher-gui
```

Compressed Images Plugin:
```
sudo apt install ros-foxy-image-transport-plugins
sudo apt install ros-foxy-rqt-image-view
```

Install ros2_control:
```
sudo apt install ros-foxy-ros2-control ros-foxy-ros2-controllers ros-foxy-gazebo-ros2-control
```
Install slam_toolbox (we will use online asynchronous):
```
sudo apt install ros-foxy-slam-toolbox
```

Install nav2:
```
sudo apt install ros-foxy-navigation2 ros-foxy-nav2-bringup ros-foxy-turtlebot3*
sudo apt install ros-foxy-twist-mux
```

### Implement package to your robot hardware or developer PC

In your home directory (~), clone the repo:
```
mkdir robot_ws/src
cd src
git clone git@github.com:bruinbot/a-ros.git
```

Go back to your robot_ws directory, then source ROS2 Foxy and build ROS:
```
cd ../..
source /opt/ros/foxy/setup.bash
colcon build --symlink-install
```

While in your robot_ws, source the a-ros package and launch BruinBot simulator:
```
source install/setup.bash
ros2 launch a-ros bruinbot_sim.launch.py world:=./src/a-ros/worlds/obstacles.world
```

### Use new terminals for the following commands. Remember to source both foxy and the a-ros package.

To tele-operate the robot:
```
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Re-map tele-op to use ros2_control:
```
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r /cmd_vel:=/diff_cont/cmd_vel_unstamped
```

Run rviz2 with camera and lidar:
```
rviz2 -d src/a-ros/config/ros2_control.rviz 
```

To open image view:
```
ros2 run rqt_image_view rqt_image_view
```

(Optional) Take compressed image that is being published by Gazebo and uncompress into new topic:
```
ros2 run image_transport republish compressed raw --ros-args -r in/compressed:=/camera/image_raw/compressed -r out:=/camera/image_raw/uncompressed
```

### Gamepad Controller
Send inputs to dev machine and send to physical robot.

Install joystick driver to Ubuntu:
```
sudo apt install joystick jstest-gtk evtest
```
Test your gamepad:
```
evtest
```
Check devices and run joystick node:
```
ros2 run joy joy_enumerate_devices
ros2 run joy joy_node
```
Open new terminal and echo to check if ROS picks up joystick:
```
ros2 topic echo /joy
ros2 param list
ros2 run joy_tester test_joy
```
Launch joystick:
ros2 launch a-ros joystick.launch.py

### 2D SLAM (slam_toolbox) Online Async

Copy online async slam_toolbox into our config directory. Run this command from home directory (~):
```
cp /opt/ros/foxy/share/slam_toolbox/config/mapper_params_online_async.yaml dev_ws/src/a-ros/config
```

Launch slam_toolbox:
```
ros2 launch slam_toolbox online_async_launch.py  params_file:=./src/a-ros/config/mapper_params_online_async.yaml use_sim_time:=true
```
On rviz2, create a Map and set topic to /map

Save generated map (as you drive around), and then set the mapper_params_online_async.yaml `mode = localization` and uncomment the map file path and map_start_at_doc lines
Here we saved the map name as `map_save`.

### Adaptive Monte Carlo Localization (AMCL):

Note -- this is part of the nav2 stack
```
ros2 run nav2_map_server map_server --ros-args -p yaml_filename:=map_save.yaml -p use_sim_time:=true
```
And on new terminal, run lifecycle_bringup on map_server:
```
ros2 run nav2_util lifecycle_bringup map_server
```
On rviz2:
Map -> Topic -> Durability Policy = Transient Local

Run AMCL to localize our robot against the map:
```
ros2 run nav2_amcl amcl --ros-args -p use_sim_time:=true
```
Use the lifecycle_bringup terminal and put it on amcl instead:
```
ros2 run nav2_util lifecycle_bringup amcl
```
Then in rviz2, click on `2D Pose Estimate` and drag and drop the arrow where your robot is currently located and facing at

### Navigation2 (Nav2)

Controller publishes to: /diff_cont/cmd_vel_unstamped

Nav2 publishes to: /cmd_vel

We multiplex the /cmd_vel_joy from teleop_node and /cmd_vel from nav2 into twist_mux node and publish it to /cmd_vel_out (but we will remap this to /diff_cont/cmd_vel_unstamped and add to launch file):
```
ros2 run twist_mux twist_mux --ros-args --params-file ./src/a-ros/config/twist_mux.yaml -r cmd_vel_out:=diff_cont/cmd_vel_unstamped
```

Launch nav2:
```
ros2 launch nav2_bringup navigation_launch.py use_sim_time:=true
```
Localization with AMCL:
```
ros2 launch nav2_bringup localization_launch.py map:=./map_save.yaml use_sim_time:=true
```
Set Initial Pose, Durability Policy = Transient Local
```
ros2 launch nav2_bringup navigation_launch.py use_sim_time:=true map_subscript_transient_local:=true
```

Files from nav2 copied to this repository, so can use these commands instead on actual robot:
```
ros2 launch a-ros bruinbot_localization_launch.py map:=./map_save.yaml use_sim_time:=true
ros2 launch a-ros bruinbot_navigation_launch.py use_sim_time:=true map_subscript_transient_local:=true
```

### Ball Tracker
https://github.com/joshnewans/ball_tracker
```
ros2 launch a-ros bruinbot_ball_tracker.launch.py sim_mode:=true
```

Optional:
```
ros2 topic echo /detected_ball
```

### Data Collection (via rosbag)
https://github.com/bruinbot/a-datacollection


### Rosbags

We want to record bags in mcap format so they can be visualized on foxglove. Install the following to enable saving as mcap.
```
$ sudo apt install ros-humble-rosbag2-storage-mcap
```

Record bags as follows
```
$ ros2 bag record -s mcap /topic1 /topic2 ...
```
