# Adding a New Model to PX4: X500 with Depth Camera and 2D LiDAR

This guide provides step-by-step instructions to add a new model (`x500_lidar_depth`) to PX4 for use with Gazebo Harmonic (Gz). The model includes an X500 drone equipped with a depth camera and a 2D LiDAR.

## Prerequisites
- PX4-Autopilot installed and set up.
- Gazebo Harmonic (Gz) installed and configured.
- Basic knowledge of ROS 2 and PX4.

## Steps to Add the New Model

### 1. Create the Model SDF File

#### Navigate to the Models Directory:
```bash
cd ~/PX4-Autopilot/Tools/simulation/gz/models
```

#### Create a New Directory for the Model:
```bash
mkdir x500_lidar_depth
cd x500_lidar_depth
```

#### Create the SDF File (`model.sdf`):
```bash
nano model.sdf
```

#### Add the following content:
```xml
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
```

#### Create the `model.config` File:
```bash
nano model.config
```

#### Add the following content:
```xml
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
```

### 2. Define the Airframe Configuration File

#### Navigate to the Airframes Directory:
```bash
cd ~/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes
```

#### Create a New Airframe File:
```bash
cp 4001_gz_x500 4021_gz_x500_lidar_depth
```

#### Edit the Airframe File:
```bash
nano 4021_gz_x500_lidar_depth
```

#### Update the file with the following content:
```bash
#!/bin/sh
#
# @name Gazebo x500 with Depth Camera and 2D LiDAR
#
# @type Quadrotor
#
PX4_SIM_MODEL=${PX4_SIM_MODEL:=x500_lidar_depth}

. ${R}etc/init.d-posix/airframes/4001_gz_x500
```

### 3. Add the Airframe to CMake

#### Edit the `CMakeLists.txt` File:
```bash
nano CMakeLists.txt
```

#### Add the following line to include the new airframe:
```cmake
px4_add_romfs_file(
  4021_gz_x500_lidar_depth
  
)
```

#### Rebuild PX4:
```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500_lidar_depth
```


#### Verify the Model:
```bash
ros2 topic list
ros2 topic echo /x500_lidar_depth/scan  # LiDAR data
ros2 topic echo /x500_lidar_depth/depth_image  # Depth camera data
```

### 5. Verify in QGroundControl

#### Open QGroundControl:
- Launch QGroundControl and connect to the simulation (usually via UDP on `localhost:14550`).

#### Select the Airframe:
- Go to **Setup > Airframe**.
- Search for your airframe (e.g., "Gazebo x500 with Depth Camera and 2D LiDAR").
- Select the airframe and click **Apply and Restart**.

#### Verify Parameters:
- Go to **Setup > Parameters** to view and modify parameters specific to your airframe.

## Directory Structure
```
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
```

## Conclusion
Following this guide, you have successfully added a new model (`x500_lidar_depth`) to PX4, equipped with a depth camera and 2D LiDAR for Gazebo Harmonic simulation. You can now test and refine your drone's sensor setup within PX4 and ROS 2.
