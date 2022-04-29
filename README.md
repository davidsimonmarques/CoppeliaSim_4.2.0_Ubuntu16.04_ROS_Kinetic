## How to Install and Configure CoppeliaSim 4.2.0 on Ubuntu 16.04 with ROS Kinetic Kame

Install ROS Kinetic Kame
------

   * Go to the ROS Kinetic Kame official installation page and follow all provided instructions: http://wiki.ros.org/kinetic/Installation/Ubuntu .


Install Tutorial CoppeliaSim - Espeleo Simulation
------
- 1 - Download CoppeliaSim V4.2.0 for Ubuntu 16.04 (https://coppeliarobotics.com/files/CoppeliaSim_Edu_V4_2_0_Ubuntu16_04.tar.xz). Unzip into *Home* folder.
		
		$ wget -P /tmp https://coppeliarobotics.com/files/CoppeliaSim_Edu_V4_2_0_Ubuntu16_04.tar.xz
		$ cd /tmp && tar -xvf CoppeliaSim_Edu_V4_2_0_Ubuntu16_04.tar.xz
		$ mv CoppeliaSim_Edu_V4_2_0_Ubuntu16_04 ~/

- 2 - Prepare ".bashrc" for CoppeliaSim:

		$ echo 'export COPPELIASIM_ROOT_DIR="$HOME/CoppeliaSim_Edu_V4_2_0_Ubuntu16_04"' >> ~/.bashrc && source ~/.bashrc
		$ echo 'alias coppelia="$COPPELIASIM_ROOT_DIR/coppeliaSim.sh"' >> ~/.bashrc && source ~/.bashrc

- 3 - Go to your catkin workspace source folder ("~/catkin_ws/src/") and clone recursively the plugin repository. **Note: the simExtROSInterface that works with Ubuntu 16.04 is the coppeliasim-v4.0.0 branch. Until now, we have seen no problems.**

		$ cd ~/catkin_ws/src/
		$ git clone https://github.com/CoppeliaRobotics/simExtROSInterface --branch coppeliasim-v4.0.0
		
- 4 - **Fix a python requirement that is broken in CoppeliaSim 4.2.0**:

		$ cd $COPPELIASIM_ROOT_DIR/programming
		$ rm -rf libPlugin
		$ git clone https://github.com/CoppeliaRobotics/libPlugin.git
		$ cd libPlugin
		$ git checkout 1e5167079b84ca002a6197414d51c40eda583d01
		
- 5 - Install the support packages:

		$ sudo apt-get install -y python-catkin-tools xsltproc ros-$ROS_DISTRO-brics-actuator ros-$ROS_DISTRO-tf2-sensor-msgs		

- 6 - Use "catkin build" to compile your packages. To do so, you must "catkin clean", then "catkin build"

		$ cd ~/catkin_ws
		$ catkin clean -y && catkin build

- 7 - If your compilation finished succesfully, the library "libv_repExtRosInterface.so" compiled correctly. 
	This library makes CoppeliaSim recognize the ROS enviroment in your machine. Now, copy this library to the CoppeliaSim directory:
	
		$ cp ~/catkin_ws/devel/lib/libsimExtROSInterface.so $COPPELIASIM_ROOT_DIR
		
- 8 - It's necessary to install the package Coppeliasim Plugin Velodyne which is responsible for publishing the velodyne point cloud from a C++ plugin, increasing the simulation performance.
		
		$ cd ~/catkin_ws/src/ && git clone https://github.com/ITVRoC/coppeliasim_plugin_velodyne.git
		$ cd ~/catkin_ws && catkin build
		$ cp ~/catkin_ws/devel/.private/coppeliasim_plugin_velodyne/lib/libv_repExtRosVelodyne.so $COPPELIASIM_ROOT_DIR

	The scenes in this repository already have the other configurations.
	If you want to create a new scene with the plugin follow the steps of the link - https://github.com/ITVRoC/coppeliasim_plugin_velodyne.

- 9 - If you run CoppeliaSim at this point, you will receive an error message and the simulator will crash. This happens to force you to update to a newer version. However, there is a possible workaround for this problem (instead of *gedit*, you can also use *nano*). 

		$ gedit $COPPELIASIM_ROOT_DIR/system/usrset.txt

    Now include the following line in the beggining of the file:

		$ allowOldEduRelease=7775

    If you're trying to install version 4.1.0 of CoppeliaSim, you should change from 7775 to 7501. For more information about this problem check this topic posted on the forum: https://forum.coppeliarobotics.com/viewtopic.php?t=9322

- 10 - Everything is ready to run. To test the communication, run the ROS master:

		$ roscore

- 11 - Now run CoppeliaSim:

		$ coppelia
		
Note that there are multiple init messages from CoppeliaSim on terminal. An indication that your library was compiled correctly is the following message:

```
		Plugin 'RosInterface': loading...
		Plugin 'RosInterface': warning: replaced variable 'simROS'
		Plugin 'RosInterface': load succeeded.
```


- 12 - To confirm the interaction between ROS and CoppeliaSim, play the empty scene in the begin of the program. In other terminal type:

		$ rostopic list
		
	If the topic "/tf" appears, the ROS/CoppeliaSim is enabled and functional.

Troubleshooting
------
* If when compiling your packages an error occurs on simExtROSInterface, download the file 'libsimExtROSInterface.so' included on this repository and copy it to $COPPELIASIM_ROOT_DIR. 

* Now, remove simExtROSInterface from /src and compile again:
		$ rm -rf ~/catkin_ws/src/simExtROSInterface
		$ cd ~/catkin_ws
		$ catkin clean -y && catkin build
