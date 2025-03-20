Adding a New Model to PX4: X500 with Depth Camera and 2D LiDAR
This guide provides step-by-step instructions to add a new model (x500_lidar_depth) to PX4 for use with Gazebo Harmonic (Gz). The model includes an X500 drone equipped with a depth camera and a 2D LiDAR.

Prerequisites
PX4-Autopilot installed and set up.

Gazebo Harmonic (Gz) installed and configured.

Basic knowledge of ROS 2 and PX4.

Steps to Add the New Model
1. Create the Model SDF File
Navigate to the Models Directory:

bash
Copy
cd ~/PX4-Autopilot/Tools/simulation/gz/models
Create a New Directory for the Model:

bash
Copy
mkdir x500_lidar_depth
cd x500_lidar_depth
Create the SDF File (model.sdf):

Create a file named model.sdf:

bash
Copy
nano model.sdf
Add the following content:

xml
Copy
<?xml version="1.0" encoding="UTF-8"?>
<sdf version='1.9'>
  <model name='x500_lidar_depth'>
    <!-- Include the base X500 model -->
    <include merge='true'>
      <uri>x500</uri>
    </include>

    <!-- Include the OakD-Lite depth camera -->
    <include merge='true'>
      <uri>model://OakD-Lite</uri>
      <pose>.12 .03 .242 0 0 0</pose>
    </include>
    <joint name="CameraJoint" type="fixed">
      <parent>base_link</parent>
      <child>OakD-Lite/base_link</child>
      <pose relative_to="base_link">.12 .03 .242 0 0 0</pose>
    </joint>

    <!-- Include the 2D LiDAR -->
    <include merge='true'>
      <uri>model://lidar_2d_v2</uri>
      <pose>-.1 0 .26 0 0 0</pose>
    </include>
    <joint name="LidarJoint" type="fixed">
      <parent>base_link</parent>
      <child>link</child>
      <pose relative_to="base_link">-.1 0 .26 0 0 0</pose>
    </joint>
  </model>
</sdf>
Run HTML
Create the model.config File:

Create a file named model.config:

bash
Copy
nano model.config
Add the following content:

xml
Copy
<?xml version="1.0"?>
<model>
  <name>X500 with Depth Camera and 2D LiDAR</name>
  <version>1.0</version>
  <sdf version="1.9">model.sdf</sdf>

  <author>
    <name>Your Name</name>
    <email>your.email@example.com</email>
  </author>

  <description>
    X500 drone with OakD-Lite depth camera and 2D LiDAR for PX4 simulation.
  </description>
</model>
Run HTML
2. Define the Airframe Configuration File
Navigate to the Airframes Directory:

bash
Copy
cd ~/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes
Create a New Airframe File:

Copy an existing airframe file (e.g., 4010_x500) and modify it:

bash
Copy
cp 4010_x500 4021_gz_x500_lidar_depth
Edit the Airframe File:

Open the file for editing:

bash
Copy
nano 4021_gz_x500_lidar_depth
Update the file with the following content:

bash
Copy
#!/bin/sh
#
# @name Gazebo x500 with Depth Camera and 2D LiDAR
#
# @type Quadrotor
#

PX4_SIM_MODEL=${PX4_SIM_MODEL:=x500_lidar_depth}

. ${R}etc/init.d-posix/airframes/4001_gz_x500
3. Add the Airframe to CMake
Edit the CMakeLists.txt File:

Open the CMakeLists.txt file in the airframes directory:

bash
Copy
nano CMakeLists.txt
Add the following line to include the new airframe:

cmake
Copy
px4_add_airframe_file(
  PATH "4021_gz_x500_lidar_depth"
  CLASS "simulation"
  TYPE "quadrotor"
  MAINTAINER "Your Name <your.email@example.com>"
)
Rebuild PX4:

Rebuild PX4 to include the new airframe:

bash
Copy
cd ~/PX4-Autopilot
make px4_sitl gz_x500_lidar_depth
4. Test the New Airframe
Start the Simulation:

Use the new airframe ID to start the simulation:

bash
Copy
PX4_SYS_AUTOSTART=4021 ./build/px4_sitl_default/bin/px4
Verify the Model:

Check that the X500 drone with the depth camera and 2D LiDAR appears in Gazebo.

Use rviz2 or ros2 topic echo to verify sensor data:

bash
Copy
ros2 topic list
ros2 topic echo /x500_lidar_depth/scan  # LiDAR data
ros2 topic echo /x500_lidar_depth/depth_image  # Depth camera data
5. Verify in QGroundControl
Open QGroundControl:

Launch QGroundControl and connect to the simulation (usually via UDP on localhost:14550).

Select the Airframe:

Go to Setup > Airframe.

Search for your airframe (e.g., "Gazebo x500 with Depth Camera and 2D LiDAR").

Select the airframe and click Apply and Restart.

Verify Parameters:

Go to Setup > Parameters to view and modify parameters specific to your airframe.

Directory Structure
Copy
PX4-Autopilot/
├── ROMFS/
│   ├── px4fmu_common/
│   │   ├── init.d-posix/
│   │   │   ├── airframes/
│   │   │   │   ├── 4021_gz_x500_lidar_depth  # New airframe file
│   │   │   │   ├── CMakeLists.txt  # Updated CMake file
├── Tools/
│   ├── simulation/
│   │   ├── gz/
│   │   │   ├── models/
│   │   │   │   ├── x500_lidar_depth/
│   │   │   │   │   ├── model.sdf  # Model SDF file
│   │   │   │   │   ├── model.config  # Model config file
