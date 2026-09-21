# ROS 2 Simulation Setup & Safety Stop Controller

## Assignment 2
### Module: ROS 2 Sensing, Simulation & Reactive Control
### Target Platform: ROS 2 (Humble / Jazzy) & Gazebo

This assignment is designed for robotics students to gain practical experience with ROS 2, Gazebo simulation, URDF robot modeling, sensor integration, and reactive control logic. The goal is to build a small autonomous mobile robot simulation in which the robot moves forward until it encounters an obstacle within a safe zone, then stops automatically.

---

## Objective

Create a ROS 2 workspace and package structure, integrate a provided URDF robot model into a Gazebo world environment with necessary sensor and actuator plugins, visualize sensor data in RViz, and implement a custom reactive node that commands the robot forward until an obstacle enters a safe distance zone.

---

### 2. Clone the assignment repository into the src folder

```bash
cd ~
git clone https://github.com/RvtLabs/offline_training_ws.git
```

### 3. Create the ROS package

```bash
cd offline_training_ws && mkdir src
cd ~/offline_training_ws/src
ros2 pkg create --build-type ament_python sim_pkg
```

### 4. Add the launch, URDF, world, and node files
Place the required files into the appropriate folders:

- `sim_pkg/launch/sim.launch.py`
- `sim_pkg/urdf/simple_robot.urdf`
- `sim_pkg/worlds/custom_env.world`
- `sim_pkg/sim_pkg/obstacle_stop_node.py`

### 5. Build the workspace

```bash
cd ~/offline_training_ws
colcon build --symlink-install
source install/setup.bash
```

### 6. Launch the simulation

```bash
ros2 launch sim_pkg sim.launch.py
```

### 7. Visualize in RViz

```bash
rviz2
```

Add the LaserScan display and select the `/scan` topic. Verify the sensor data is visible in the scene.

---

## Verification Commands

Use these commands to check the robot behaviour and sensor output:

```bash
ros2 topic echo /scan
ros2 topic echo /cmd_vel
```

You should observe:

- laser range data on `/scan`
- positive forward command velocity while the path is clear
- zero or near-zero command velocity when an obstacle is detected within the safety threshold

---

## Suggested Implementation Logic for the Safety Node

The obstacle safety controller can be implemented as a ROS 2 Python node using the following logic:

```python
# Pseudocode
while True:
    cmd = Twist()
    cmd.linear.x = 0.5  # forward motion

    if obstacle_detected_in_forward_fov(range_data, min_angle=-30, max_angle=30, threshold=0.5):
        cmd.linear.x = 0.0

    pub.publish(cmd)
```

The key idea is to check only the ranges that are in front of the robot within the allowed angular window and stop if any nearby obstacle violates the safety distance.

---

## Final Submission Requirements

Students must submit the following:

1. Workspace Snapshot
   - A clear screenshot or tree layout showing `ros2_ws/src/sim_pkg` with all required files.

2. Demonstration Video
   - Show the robot driving forward in Gazebo/RViz.
   - Show a static or dynamic obstacle entering the restricted zone.
   - Show the robot stopping automatically before hitting the obstacle.

3. Source Package
   - Clean source code with launch files, URDF model, world configuration, and the Python safety node.

---

## Core Tasks

### Task 1: Workspace & Package Creation
Set up a standard ROS 2 workspace and create a custom package named `sim_pkg` using either `ament_python` or `ament_cmake`.

Expected package structure:

```text
offline_training_ws/
└── src/
    └── sim_pkg/
        ├── launch/
        │   └── sim.launch.py
        ├── urdf/
        │   └── simple_robot.urdf
        ├── worlds/
        │   └── custom_env.world
        ├── sim_pkg/
        │   └── obstacle_stop_node.py
        ├── package.xml
        └── CMakeLists.txt / setup.py
```

---

### Task 2: Gazebo World Launch & URDF Spawning
Develop a Python launch file (`sim.launch.py`) that:

- launches a Gazebo environment using `custom_env.world`
- spawns the robot URDF model into the world
- publishes the robot description using `robot_state_publisher`
- uses `ros_gz_sim` or `gazebo_ros` entity spawning tools to place the robot correctly in the environment

The robot should appear in the Gazebo world and be ready for motion and sensing.

---

### Task 3: Plugins Integration & Data Visualization
Verify and ensure the URDF includes the required plugin support:

- Differential Drive Plugin:
  - controls robot motion via `/cmd_vel`
  - publishes odometry data to `/odom`
- 2D LiDAR / distance sensor plugin:
  - publishes scan data to `/scan`

Visualization:

- launch RViz
- visualize the laser scan rays from `/scan`
- verify topic activity using:

```bash
ros2 topic echo /scan
ros2 topic echo /cmd_vel
```

---

## Final Task: Reactive Safety Stop Controller

Develop a custom ROS 2 node named `obstacle_stop_node.py` to control straight-line robot motion with obstacle safety checking.

### Functional Requirements

1. Straight Navigation
   - Continuously command the robot to move forward by publishing a constant positive linear velocity in the x-direction.
   - The robot should keep moving forward while the path is clear.

2. LiDAR FOV Filter
   - Subscribe to `/scan`.
   - Evaluate only the ranges inside the forward field of view from $-30^\circ$ to $+30^\circ$ relative to the robot heading.

3. Safety Threshold Trigger
   - If any obstacle is detected within a distance of less than 0.5 meters in the defined forward field-of-view region, stop the robot immediately.
   - Set linear velocity to zero until the obstacle clears.

4. Safety Behavior
   - The robot must halt before collision.
   - Once the path is clear again, motion can resume.

---

## Summary

The expected outcome is a working ROS 2 simulation where a robot drives forward in a Gazebo world, detects nearby obstacles with a forward-facing LiDAR sensor, and stops automatically when an obstacle enters the safety zone inside $-30^\circ$ to $+30^\circ$ and within 0.5 meters.

This assignment is an excellent foundation for more advanced ROS 2 perception and autonomous navigation tasks.
