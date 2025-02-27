Visualization node for ROS using OpenVR
=======================================

[![Example Screenshot](turtlebot_demo.jpg)](http://www.youtube.com/watch?v=M0u4YXW0DFU "VRViz Turtlebot Demo")

This code was built from the [openvr](https://github.com/ValveSoftware/openvr) 
example code, and adapted to run in catkin and to display ROS messages in 
virtual space running natively on Ubuntu. There are currently not many message types implemented, and in the future it may be more sensible to turn this into an rviz plugin rather than continue re-implimenting message types here.

If you end up using it, please cite us :) 

```
@inproceedings{ONeill2019,
author = {O'Neill, J. and Ourselin, S. and Vercauteren, T. and {Da Cruz*}, L. and Bergeles*, C.},
booktitle = {Joint Workshop on New Technologies for Computer/Robot Assisted Surgery},
title = {{VRViz: Native VR Visualization of ROS Topics}},
year = {2019}
}
```

Prerequisites
-------------

The main dependancy is SteamVR, which can be installed from [Steam](https://store.steampowered.com/). Additionally you will need a VR headset compatable with openvr. This code has only been tested with the HTC Vive, but it is possible that other headsets supported by steamvr such as the Oculus Rift could work as well.

Several library dependancies are included in the the repo, the [openvr](https://github.com/ValveSoftware/openvr) 
library, which is included in `openvr_library` and sdl2 which is included in `sdl2_library`.

The code is designed to be used in ROS, and has been tested in ROS Kinetic on Ubuntu 16.04. Instructions for installing ROS can be found [here](http://wiki.ros.org/ROS/Installation) and other than the turtlebot demo, all ROS dependencies should be covered by `ros-kinetic-desktop-full`.

The non-ROS dependencies include GLEW for rendering and [assimp](http://www.assimp.org/) for loading URDF robot models with Collada meshes.
These should be able to be installed with rosdep or with:
```
sudo apt-get install libglew-dev libassimp-dev
```

## BIRDLab installation notes

**The instructions in this subsection were only tested in the BIRDLab's high performance workstation and with the AMD Radeon HD 7850 graphic's card.**

Add the following option to your GRUB (you can use grub-customizer)
```
nano /etc/default/grub
```
Paste the following:
```
GRUB_CMDLINE_LINUX_DEFAULT="radeon.cik_support=0 radeon.si_support=0 amdgpu.cik_support=1 amdgpu.si_support=1"
```

Then run the following command:
```
update-grub
reboot
```

When starting Steam/SteamVR/VRviz do not forget to prioritize the drivers but prefixing the driver order:

```
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json steam
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/radeon_icd.x86_64.json roslaunch vrviz vrviz.launch
```

### Common issue
In the following link common issues have been reported:

https://old.reddit.com/r/linuxquestions/comments/abt1cr/vulkan_not_using_my_radeon_gpu/

http://forums.debian.net/viewtopic.php?f=6&t=142110

https://wiki.archlinux.org/index.php/Vulkan


Running with Steam Runtime
--------------------------

For now, this node requires being run as part of the steam runtime (This can be automated, as shown in vrviz.launch):
```
rosrun --prefix '~/.steam/ubuntu12_32/steam-runtime/run.sh' vrviz vrviz_gl
```

Demonstration Launch Files
--------------------------

For a demo of showing a Turtlebot in Gazebo, install `ros-kinetic-turtlebot-gazebo` and run:
```
roslaunch vrviz turtlebot_demo.launch
```
This should load up the robot, and it can be controlled by pulling the trigger of the controller and then moving/rotating the wand while the trigger is depressed. Pressing the touchpad and moving the wand will move the world around relative to the user. This launch file will fix the grid to the `odom` frame. See an example youtube video [here](http://www.youtube.com/watch?v=M0u4YXW0DFU "VRViz Turtlebot Demo").

For a demo showing a bagfile download the `demo_mapping.bag` file from [here](http://wiki.ros.org/rtabmap_ros) and run:
```
roslaunch vrviz point_cloud_demo.launch bagfile:=/path/to/demo_mapping.bag
```

For a demo showing a stereoscopic video, download the `bbb_clip_sbs.mp4` file from [here](https://www.dropbox.com/s/pjnyp77bv93qjiw/bbb_clip_sbs.mp4?dl=1) and install `ros-kinetic-video-stream-opencv` and then run:
```
roslaunch vrviz video_demo.launch video_file:=/path/to/bbb_clip_sbs.mp4
```

Features
--------
 - The default RViz 1m grid
 - Scaling the VR world relative to the ROS world (set by rosparam at startup)
 - Loading a robot model from the parameter server with `load_robot:=true`
 - Visualizing TF's (currently only TF's that have been referenced somewhere)
 - Visualizing PointCloud2 messages (works with color or intensity, otherwise sets constant color)
 - Visualizing stereo pair image (currently expects one side-by-side image, or duplicates the same image to each eye)
 - Visualizing camera image (projects out from camera location)
 - Visualizing visualization messages (All [types](http://wiki.ros.org/rviz/DisplayTypes/Marker) are at least basically supported, but may not perform identically to rviz)
   - to see a variety of markers, run `roslaunch vrviz turtlebot_demo.launch silly_shapes:=true`

Limitations
-----------
 - The code is very much a work in progress, and many features are partially or inefficiently implemented.
 - The [SteamVR support for Ubuntu](https://github.com/ValveSoftware/SteamVR-for-Linux) is still in Beta, so be careful.
 - Currently only supports one of each message type. This can be worked around by, for example, concatenating a bunch of point clouds in another node and then sending the big cloud into VRViz. Also multiple visualization messages could be mapped to the one receiver, as it should update based on namespace and ID.
 - Please feel free to open a feature request or add a pull request, there are lots of little improvements that we have not gotten around to but if there's a desire for them we would be happy to try.

Vulkan
------
As much as possible, the code has been created to keep the opengl specific things separate in order to allow building either with vulkan or opengl (based on the fact that the `open_vr` examples of opengl and vulkan, which this was built from, are very similar). However, while the vulkan executable (`vrviz_vk`) builds and links, it does NOT have any real functionality to speak of, and would require some work to bring up to the level of `vrviz_gl`. It is currently commented out of the CMakeLists and does not get built.

