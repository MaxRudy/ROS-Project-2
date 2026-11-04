# ROS-Project-2
arm6dof_pick_place - 6-DOF robotic arm in ROS + RViz (no MoveIt)
================================================================

WHERE EACH FILE GOES  (inside ~/catkin_ws/src/arm6dof_pick_place/)

  package.xml
  CMakeLists.txt
  urdf/arm6dof.xacro            robot model (6 joints + gripper + place pad)
  scripts/arm_mover.py          auto sine-wave motion node
  scripts/pick_place.py         scripted pick-and-place node
  scripts/cube_marker.py        draggable gold cube (interactive marker)
  launch/display.launch         manual sliders + draggable cube   <- main demo
  launch/demo.launch            arm moves by itself
  launch/pick_place.launch      scripted pick-and-place
  rviz/arm.rviz                 saved RViz layout

  (config/controllers.yaml and launch/gazebo.launch are from your original
   package and are not needed for this plain-RViz version.)

ONE-TIME INSTALL
  sudo apt install -y ros-noetic-joint-state-publisher-gui ros-noetic-interactive-markers
  chmod +x ~/catkin_ws/src/arm6dof_pick_place/scripts/*.py
  cd ~/catkin_ws && catkin_make

RUN  (run only ONE launch file at a time)
  source ~/catkin_ws/devel/setup.bash
  export LIBGL_ALWAYS_SOFTWARE=1
  roslaunch arm6dof_pick_place display.launch
