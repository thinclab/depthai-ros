# Depth-AI ROS Fork

This fork contains the location of the Oak-D S2 in the current THINC lab setup, a topic remapping for the robot state publisher to avoid conflicts, and an additional `/udev` folder for read/write permissions.

The original DepthAI-ROS GitHub Repository and Documentation can be found at the links below.<br>
[Depth-AI ROS GitHub](https://github.com/luxonis/depthai-ros.git)<br>
[Depth-AI ROS Documentation](https://docs.luxonis.com/software/depthai/manual-install/)<br>

#### Table of Contents
[Installation](#Installation)<br>
[Setup](#Setup)<br>
[Use](#Use)<br>

## Installation

Use the following command to download dependencies for the Oak-D package.

    sudo wget -qO- https://docs.luxonis.com/install_dependencies.sh | bash

It is recommended to have a separate workspace for this driver to simplify the build process; use the commands below to create the `/cam_ws` and to clone the fork into the `/src` directory.

    mkdir -p ~/cam_ws/src
    cd ~/cam_ws/src
    git clone -b jazzy https://github.com/thinclab/depthai-ros.git

After you have cloned the fork, go to `~/cam_ws`, resolve dependencies, and build the package in the workspace using the commands below.

    cd ~/cam_ws
    rosdep install --from-paths src --ignore-src -r -y
    colcon build

Make sure to source the workspace after building. You can add the source commands to your `~/.bashrc` using the commands below so that this happens automatically.

    echo "source ~/cam_ws/install/setup.bash" >> ~/.bashrc
    echo "source ~/cam_ws/install/local_setup.bash" >> ~/.bashrc

In order to run this driver, you need to first give the camera read and write permissions for your machine. Go to the `/udev_rules` folder of this package, move the `99-oak.rules` file to your `/rules.d` folder, and reload the udev for the changes to take effect.

    cd ~/cam_ws/src/depthai-ros/udev_rules
    sudo mv 99-oak.rules /etc/udev/rules.d/
    sudo udevadm control -R

## Setup

This fork contains a number of things to simplify the use of the Oak-D S2 Camera. First, a custom configuration file,`/depthai-ros/depthai_ros_driver/config/camera.yaml`, has been included. Within this file, the location of the camera is defined by the following lines. The current values reflect the position of the Oak-D S2 Camera within THINC lab, but these values can be changed to your desire.

    camera:
      i_publish_tf_from_calibration: true
      i_tf_parent_frame: world
      i_tf_cam_pos_x: "0.355"
      i_tf_cam_pos_y: "-0.002"
      i_tf_cam_pos_z: "1.87"
      i_tf_cam_roll: "0.0"
      i_tf_cam_pitch: "1.570796"
      i_tf_cam_yaw: "0.0"

In addition to this, the RGB values can be modified with the following lines.

    rgb:
      r_set_man_exposure: true
      r_set_man_whitebalance: true
      r_exposure: 33000
      r_iso: 204
      r_whitebalance: 3200

The IMU is enabled with the inclusion of the following in the configuration file.

    pipeline_gen:
      i_enable_imu: true
    imu:
      i_enable_acc: true
      i_enable_gyro: true
      i_enable_rotation: true

A number of other parameters can be specified in this file. Please see the following link for a list of the parameters.<br>
[Depth-AI ROS Parameters](https://docs.luxonis.com/software/ros/depthai-ros/driver/#DepthAI%20ROS%20Driver-List%20of%20parameters)

In addition to the launch configuration file, this package has modified the original URDF launch file to include a topic remapping. The URDF was originally sent to the `/robot_description` topic, but now the `robot_state_publisher` has been remapped to the `/oak_camera_description` topic. Because this fork is usually used in conjunction with other packages that publish URDF's, this remapping is necessary to avoid conflicts. Feel free to open `/depthai-ros/depthai_descriptions/launch/urdf_launch.py` and change line 90 so that it publishes to a different topic of your choosing. 

    82   Node(
    83       package="robot_state_publisher",
    84       condition=UnlessCondition(use_composition),
    85       executable="robot_state_publisher",
    86       name=name + "_state_publisher",
    87       namespace=namespace,
    88       parameters=[robot_description],
    89       remappings=[
    90           ('/robot_description', '/oak_camera_description'),
    91       ],
    92   ),

## Use

Start the Oak-D S2 Camera Driver with the following launch command.

    ros2 launch depthai_ros_driver camera.launch.py

If it is successful, you should see a message like this,

    [component_container-3] [INFO] [1742215114.606896223] [oak]: Camera ready!

### Calibration

In order to calibrate the camera, you will need to clone the `depthai` repository. Then, create a Python environment and install dependencies with the `requirements.txt` file. Make sure that the `depthai-calibration` and `depthai-boards` repositories are also downloaded into the corresponding folders of the `depthai` repository. If everything is setup correctly, follow the instructions at the following documentation.
[Calibration Documentation](https://docs.luxonis.com/hardware/platform/depth/calibration)<br>

<!-- git clone luxonis/depthai
conda env create 
pip install /depthai/requirements.txt
conda install libstdcxx -c conda-forge
pip install opencv-contrib-python==4.7.0.72 opencv-python==4.7.0.72
pip install matplotlib
git clone luxonis/depthai-calibration
git clone luxonis/depthai-boards
mv depthai-calibration/calibration_utils.py into /depthai
mv depthai-boards into /depthai/resources
apply changes to calibrate.py

python3 calibrate.py -s 3.45 --board OAK-D-S2 -nx 13 -ny 7 -m process

https://docs.luxonis.com/software-v3/depthai/tools/oak-viewer/ -->