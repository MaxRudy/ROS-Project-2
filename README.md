  arm6dof_pick_place
  6-DOF Robotic Arm for Pick-and-Place Operation  (ROS + RViz, no MoveIt)
================================================================================

Author      : Kedar S Ajai  (ASI24RA035)
Course      : PERUT528 - Robot Operating System (ROS), Semester 5
Institution : Adi Shankara Institute of Engineering and Technology
              Dept. of Robotics and Automation, APJ Abdul Kalam Technological University
Environment : Ubuntu 20.04 (WSL2) | ROS 1 Noetic | RViz


--------------------------------------------------------------------------------
1. OVERVIEW
--------------------------------------------------------------------------------
This package models a 6-degree-of-freedom robotic arm with a two-finger parallel
gripper and visualizes it in RViz. The arm is described in URDF/Xacro. Joint
angles are published on /joint_states, and robot_state_publisher computes the
forward kinematics and broadcasts the TF tree that RViz draws.

No physical hardware, MoveIt or Gazebo is needed. The package only needs
URDF/Xacro, robot_state_publisher, joint_state_publisher_gui, interactive_markers
and RViz.

Three ways to run it:
  1. MANUAL  - drag a slider for each joint yourself, and drag the gold cube
               with the mouse.
  2. AUTO    - the arm sweeps through all its joints by itself.
  3. SCRIPTED PICK-AND-PLACE - the arm moves through a pick, lift, transport,
               place, release and return cycle (see Known Limitations).


--------------------------------------------------------------------------------
2. PACKAGE STRUCTURE
--------------------------------------------------------------------------------
arm6dof_pick_place/
 |-- package.xml              Package metadata and dependencies
 |-- CMakeLists.txt           Build rules (installs scripts, launch, urdf, rviz)
 |-- README.txt               This file
 |-- urdf/
 |    '-- arm6dof.xacro       Robot model: 6 joints, gripper, place pad
 |-- scripts/
 |    |-- arm_mover.py        Auto sine-wave motion node
 |    |-- pick_place.py       Scripted pick-and-place node
 |    '-- cube_marker.py      Draggable gold cube (interactive marker)
 |-- launch/
 |    |-- display.launch      Manual sliders + draggable cube  (MAIN DEMO)
 |    |-- demo.launch         Auto-moving arm
 |    '-- pick_place.launch   Scripted pick-and-place
 '-- rviz/
      '-- arm.rviz            Saved RViz layout (Grid, RobotModel, TF, CubeMarker)

Files from the original package that are NOT needed for this version (kept on
disk, untouched, not referenced by the build):
  config/controllers.yaml, launch/gazebo.launch, urdf/cube.urdf, urdf/table.urdf


--------------------------------------------------------------------------------
3. ROBOT MODEL
--------------------------------------------------------------------------------
Kinematic chain (root to tip):

  world -> base_link -> link1 -> link2 -> link3 -> link4 -> link5 -> link6
        -> tool0 -> gripper_base -> left_finger / right_finger

Joints:

  Joint               Type       Axis   Motion          Limits
  ------------------  ---------  -----  --------------  ----------------------
  joint1              revolute   Z      Waist yaw       -3.14 to +3.14 rad
  joint2              revolute   Y      Shoulder pitch  -1.57 to +1.57 rad
  joint3              revolute   Y      Elbow pitch     -2.20 to +2.20 rad
  joint4              revolute   Z      Wrist roll      -3.14 to +3.14 rad
  joint5              revolute   Y      Wrist pitch     -1.57 to +1.57 rad
  joint6              revolute   Z      Wrist yaw       -3.14 to +3.14 rad
  left_finger_joint   prismatic  Y      Gripper finger   0.00 to +0.02 m
  right_finger_joint  prismatic  Y      Gripper finger  -0.02 to  0.00 m

Link lengths (metres): base 0.05, link1 0.05, link2 0.10, link3 0.10,
link4 0.03, link5 0.03, link6 0.02. These are placeholder values. Replace them
with the measured values of a real arm if you build one.

Colours: links alternate Electric Violet and Cyber Teal. Dark spheres sit at
each joint to hide the seam between cylinders. The pick cube is gold and the
place pad is white.


--------------------------------------------------------------------------------
4. REQUIREMENTS
--------------------------------------------------------------------------------
  - Ubuntu 20.04 (native or WSL2) with ROS Noetic installed
  - A catkin workspace at ~/catkin_ws
  - ROS packages: urdf, xacro, robot_state_publisher, joint_state_publisher,
    joint_state_publisher_gui, interactive_markers, rviz, rospy, sensor_msgs

Install the extra packages (one time):

  sudo apt update
  sudo apt install -y ros-noetic-joint-state-publisher-gui \
                      ros-noetic-interactive-markers \
                      ros-noetic-xacro \
                      ros-noetic-robot-state-publisher

For a different ROS distro, replace "noetic" with the output of: echo $ROS_DISTRO


--------------------------------------------------------------------------------
5. INSTALLATION AND BUILD
--------------------------------------------------------------------------------
1) Put the package folder in your workspace:

     ~/catkin_ws/src/arm6dof_pick_place/

   The folder that holds package.xml and CMakeLists.txt must sit directly
   inside src/. A nested arm6dof_pick_place/arm6dof_pick_place/ folder will
   break the build.

2) Make the scripts executable:

     chmod +x ~/catkin_ws/src/arm6dof_pick_place/scripts/*.py

3) Build and source:

     cd ~/catkin_ws
     catkin_make
     source devel/setup.bash

4) Optional: make the setup permanent so every new terminal is ready.

     echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
     echo "export LIBGL_ALWAYS_SOFTWARE=1"       >> ~/.bashrc

   LIBGL_ALWAYS_SOFTWARE=1 fixes RViz flickering or crashing under WSL.

5) Check the model before running (optional):

     rosrun xacro xacro ~/catkin_ws/src/arm6dof_pick_place/urdf/arm6dof.xacro > /tmp/arm.urdf
     check_urdf /tmp/arm.urdf


--------------------------------------------------------------------------------
6. RUNNING
--------------------------------------------------------------------------------
Open a WSL terminal, then:

     source ~/catkin_ws/devel/setup.bash
     export LIBGL_ALWAYS_SOFTWARE=1

Run ONE of the following. roslaunch starts roscore for you. Stop with Ctrl+C
before starting another.

  A) MANUAL CONTROL  (main demo)
       roslaunch arm6dof_pick_place display.launch

     Opens RViz and a "joint_state_publisher_gui" window with one slider per
     joint. Dragging a slider moves that joint in RViz. The gold cube is a
     draggable marker. Click it, then drag the red, green and blue arrows to
     move it. If the slider window is hidden behind RViz, check the taskbar.

  B) AUTO DEMO
       roslaunch arm6dof_pick_place demo.launch

     arm_mover.py publishes a sine wave for every joint, so the arm moves
     continuously with the gripper opening and closing.
     Optional arguments:  rate:=30.0   animate:=true|false

  C) SCRIPTED PICK-AND-PLACE
       roslaunch arm6dof_pick_place pick_place.launch

     pick_place.py steps through 10 waypoints in a loop:
       home -> above_cube -> at_cube -> grasp -> lift_cube -> above_place ->
       at_place -> release -> lift_after_place -> return_home

IMPORTANT: never run two modes at the same time. Both publish on /joint_states
and the arm will twitch as they fight each other.


--------------------------------------------------------------------------------
7. ROS NODES AND TOPICS
--------------------------------------------------------------------------------
  Node                         Role
  ---------------------------  ---------------------------------------------
  joint_state_publisher_gui    Publishes slider values to /joint_states
  arm_mover                    Publishes sine-wave joint values
  pick_place_demo              Publishes interpolated waypoint joint values
  robot_state_publisher        Forward kinematics, broadcasts TF
  cube_marker                  Interactive marker server ("cube_control")
  rviz                         3D visualization

  Topic                        Type                    Publisher
  ---------------------------  ----------------------  -----------------------
  /joint_states                sensor_msgs/JointState  GUI / mover / pick_place
  /tf, /tf_static              tf2_msgs/TFMessage      robot_state_publisher
  /cube_control/update         interactive marker      cube_marker

Useful inspection commands:

     rostopic echo /joint_states            # live joint angles
     rostopic hz /joint_states              # publish rate
     rosrun tf tf_echo world tool0          # live gripper-tip pose (forward kinematics)
     rosrun rqt_tf_tree rqt_tf_tree         # view the TF tree
     rosnode list                           # running nodes


--------------------------------------------------------------------------------
8. HOW IT WORKS
--------------------------------------------------------------------------------
Forward kinematics: robot_state_publisher reads the URDF (loaded as the
"robot_description" parameter from the Xacro file), takes the joint angles from
/joint_states, and multiplies the chain of joint transforms from the base to
every link. RViz subscribes to the resulting TF tree and redraws each link.

Waypoint motion (pick_place.py): each waypoint holds six arm angles, a gripper
state (open or closed) and a duration. Between waypoints the node linearly
interpolates every joint, value = start + (target - start) * t with t going from
0 to 1, and publishes at 30 Hz. This gives smooth motion without inverse
kinematics. The angles are hand-tuned joint-space poses, not computed from a
Cartesian target.

Interactive cube (cube_marker.py): the cube is an InteractiveMarker in the
"world" frame with three MOVE_AXIS drag handles. It is not part of the URDF, so
moving it does not affect the arm.


--------------------------------------------------------------------------------
9. SUGGESTED DEMO FLOW (for presenting)
--------------------------------------------------------------------------------
  1. Run display.launch. Explain that the URDF defines the links and joints.
  2. Drag joint1 to joint6 one at a time and name each motion: waist yaw,
     shoulder pitch, elbow pitch, wrist roll, wrist pitch, wrist yaw.
  3. Open and close the gripper with the two finger sliders.
  4. Drag the gold cube next to the gripper. Explain it is an interactive marker
     the operator places by hand.
  5. In a second terminal, run:  rosrun tf tf_echo world tool0
     This prints the live gripper-tip position as the sliders move.
  6. Stop it (Ctrl+C) and run demo.launch to show the automatic sweep.


--------------------------------------------------------------------------------
10. TROUBLESHOOTING
--------------------------------------------------------------------------------
  Problem                                      Fix
  -------------------------------------------  --------------------------------
  Resource not found: arm6dof_pick_place       Run: source ~/catkin_ws/devel/setup.bash
  catkin_make cannot find the package          Check package.xml sits directly in
                                               src/arm6dof_pick_place/ (no nesting)
  RViz flickers, crashes or is blank           export LIBGL_ALWAYS_SOFTWARE=1
  "No transform from [link] to [world]"        Nothing is publishing /joint_states.
                                               Check rosnode list.
  Arm frozen or no motion                      chmod +x scripts/*.py, then
                                               catkin_make again
  Arm twitches                                 Two modes are running. Stop one.
  No slider window                             Look behind RViz or in the taskbar.
                                               pick_place.launch and demo.launch
                                               do not open sliders by design.
  Gold cube not visible or not draggable       Confirm the "CubeMarker" display is
                                               ticked in RViz and the Interact tool
                                               (top toolbar) is selected. Confirm
                                               interactive_markers is installed.
  ImportError for interactive_markers          sudo apt install
                                               ros-noetic-interactive-markers
  Could not load robot model                   Run the xacro command in step 5 of
                                               Section 5 to see the real error.
  After editing files nothing changes          cd ~/catkin_ws && catkin_make


--------------------------------------------------------------------------------
11. KNOWN LIMITATIONS
--------------------------------------------------------------------------------
  - The waypoint angles in pick_place.py are approximate. The arm may not
    land exactly on the cube or pad, because the pick and place positions
    were estimated, not solved with inverse kinematics.
  - The gold cube is visual only. It does not attach to the gripper and there
    is no collision or grasp detection. In manual mode you move the cube by
    hand to simulate a grasp.
  - This is a kinematic visualization, not a physics simulation. There is no
    gravity, no contact and no dynamics.
  - On WSL2 you may see "Moved backwards in time" or "Detected jump back in
    time" warnings in the terminal. They come from WSL clock resyncs and are
    harmless. The arm keeps working.
  - The transmission and gazebo blocks in the Xacro are unused here. RViz
    ignores them.


--------------------------------------------------------------------------------
12. FUTURE WORK
--------------------------------------------------------------------------------
  - Inverse kinematics so the arm can reach a target XYZ position
  - MoveIt integration for collision-aware motion planning
  - Camera-based object detection to replace the manually placed cube
  - Gazebo physics simulation
  - Building the arm in hardware (servos plus a microcontroller such as ESP32)


--------------------------------------------------------------------------------
13. LICENSE
--------------------------------------------------------------------------------
MIT License (as declared in package.xml).
================================================================================
