## Install

Install ROS Kinetic with homebrew, following http://wiki.ros.org/kinetic/Installation/OSX/Homebrew/Source up to step 2.

This script might be preferred for installing everything at once https://github.com/mikepurvis/ros-install-osx

On OS X you will also need to run:

`brew install yaml-cpp`

Now, from the root dir install dependences:

`rosdep install --from-paths src --ignore-src -r -y`

And finally, build the project:

`catkin build`

and source the workspace:

`source devel/setup.bash`

You'll need to re-source the workspace every a new term is opened (or add it to ~/.bashrc).


## Tests

### Keyboard joystick

Follow instructions from https://github.com/ethz-asl/rotors_simulator/wiki/Setup-virtual-keyboard-joystick to setup a virtual joystick.

Then run a launch file that initiates a (virtual) joystick node and velocity controller node, runs gazebo, and spawns a MAV model in an empty world environment:

`roslaunch rotors_gazebo mav_velocity_control_with_keyboard.launch dev:=/dev/input/js0`

The `dev:=/dev/input/jso` part assigns a virtual joystick device to the parameter `dev`. Change this if the device is at a different location (you'll only see it when the keyboard script is run).

The virtual joystick only receives input when its window is active. The controls are mapped to:

* W / S -- thrust
* A / D -- yaw
* UP / DOWN -- pitch
* LEFT / RIGHT -- roll

Thrust will need to get to about 80% before it will take off.


### Launching other worlds

Other worlds are included in the folder `src/rotors_simulator/rotors_gazebo/worlds`. Not all of them are entirely working or have textures loaded. To load a world:

`roslaunch rotors_gazebo mav_velocity_control_with_keyboard.launch dev:=/dev/input/js0 world_name:=outdoor`

Worlds verified to work are:

* yosemite
* outdoor
* test_city
* pp
* 3boxes_room
* tum_kitchen
* ardrone_testworld
* competition


### Launching other MAVs

A couple of other vehicles are included under `src/rotors_simulator/rotors_description/urdf/`. E.g. to launch a parrot AR 2.0 model:

`roslaunch rotors_gazebo mav_velocity_control_with_keyboard.launch dev:=/dev/input/js0 mav_name:=ardrone`


## Running DroNet

First, launch the perception node:

`roslaunch dronet_perception dronet_launch.launch`

This will subscribe to the `/bebop2/camera_base/image_raw` topic and wait for images to run through the pre-trained DroNet. The output will be steering angle and collision prob published on the `cnn_predictions` topic.

Next, in a separate tab launch the control node:

`roslaunch dronet_control deep_navigation.launch`

This node subscribes to `cnn_predictions` and publishes forward (linear.x) and yaw (angular.z) velocities on the `/bebop/cmd_vel` topic as implemented in `src/dronet_control/src/deep_navigation.cpp`. If the real bebop_driver node were running it would subscribe to `/bebop/cmd_vel` and execute the velocity controls.

Finally, launch gazebo along with the keyboard and velocity controller nodes:

`roslaunch rotors_gazebo mav_velocity_control_with_keyboard.launch dev:=/dev/input/js0 world_name:=outdoor`

The perception node tab should now print the collision/steering predictions from DroNet. The control node tab should show the velocity controls. These values should change as you manuever the drone in the environment (remember to make the virtual joystick window active).


## Viewing Topics

Topics can be streamed on the console with the `rostopic echo <topic_name>` command. Tab-completing will show the list of available topics, e.g. to view the messages the joystick topic is publishing (after starting the simulation):

`rostopic echo /bebop2/joy`


## rqt

A custom dashboard can be put together using the `rqt` tool. Try this:

1. Start the simulation
2. Run `rqt`
3. Go to Plugins->Visualization->Image View to add an image widget.
4. In the top drop down choose `/bebop2/camera_base/image_raw`. This should stream from the forward-mounted camera.
