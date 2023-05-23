# Readme for SOFAR
This is a short manual for all main commands and hint for ROS2. By Tabi43.

# Packages Section
To create a package:
``` bash
ros2 pkg create turtle_control --build-type ament_python --dependencies rclpy geometry_msgs std_msgs --node-name turtle controller
```
C++ package:
``` bash
ros2 pkg create turtle_control --build-type ament_cmake --dependencies rclcpp geometry_msgs std_msgs --node-name turtle controller
```

Make sure you source the enviroment with:
```bash
    source /opt/ros/humble/setup.bash
```

## Add a node
To add a node in a ROS2 package you can add a new file .py or .cpp inside your src folder and update the package.xml declaring the new file. Inside ***setup.py*** add for each new node inside:
``` python
    entry_points={
        'console_scripts': [
            '<node_name> = <pkg_name>.<node_name>:main',
        ],
```

# Compile
To compile a package use inside the ws folder:
```bash
    colcon build
```
To compile only a specific package:
``` bash
    colcon build --packages-select <my_package>
```

# Launch File 
Add this line inside ***setup.py*** :    
``` python
     data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        -----> (os.path.join("share", package_name, "launch"), glob("launch/*.launch.py")),
    ],
```
It should be the last element of the array ***data_files***.

Make sure you have added in ***setup.py***
``` python
    import os
    from glob import glob
```
The launch file is structured as:
``` python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='<pkg_name>',
            executable='<node_file_name>',
            name='<name>',
            remappings=[ ('/turtlesim2/turtle1/pose', '/remapped/pose'),
            ('/turtlesim2/turtle1/cmd_vel', '/remapped/cmd_vel')]
        ),  
        Node(
            package='<pkg_name>',
            executable='<node_file_name>',
            name='<name>',
            parameters=[
                {"<parameter_name>": <value>},
                {"<parameter_name>": <value>}
            ]
        ) 
    ])
```

# Servicies & Interfaces
***Messages needs to be defined in c++ packages!***
So we make a package containing interfaces, the convention is to name this package as "<pkg_name>_interface". Once we have created the package and maked the *srv* directory, we can write inside our ***.srv*** file:

The ***service.srv*** file is structured as:
``` python
    #request
    int64 x
    int64 y
    ---
    #response
    int64 sum
```
Before compile we need to add a line inside ***CMakeLists.txt*** :
``` python
    find_package(rosidl_default_generators REQUIRED)

    rosidl_generate_interfaces(${PROJECT_NAME}        
        "srv/<srv_file_name>.srv"
        DEPENDENCIES std_msgs geometry_msgs <etc>
    )
```
The add the dependencies in ***package.xml*** :
``` python
    <buildtool_depend>rosidl_default_generators</buildtool_depend>
    <exec_depend>rosidl_default_runtime</exec_depend>

    <member_of_group>rosidl_interface_packages</member_of_group>
```

NOTE:
<ol>
    <li> Use explicit types in req and res in the callback function in the code!</li>
    <li>Use wait_for_service() to ensure to access to a ready and existing service !</li>
    <li>TIP : close and reopen VS Code to see correctly the new interface builded</li>
</ol>

# Action & Interfaces
We need to define the package interface describe above... 

***The first letter of the File.action must be uppercase!***

The ***action.action*** is structured as:
``` python
    #Request
    ---  
    #Result
    std_msgs/Int64 blabla
    ---
    #Feedback
```
Before compile we need to add a line inside ***CMakeLists.txt*** :
``` python
    find_package(rosidl_default_generators REQUIRED)

    rosidl_generate_interfaces(${PROJECT_NAME}       
        "action/<action_file_name>.action"
        DEPENDENCIES std_msgs geometry_msgs <etc>
    )
```
The add the dependencies in ***package.xml*** :
``` python
    <buildtool_depend>rosidl_default_generators</buildtool_depend>
    <depend>action_msgs</depend>

    <member_of_group>rosidl_interface_packages</member_of_group>
```
# Parameters
To use a parameter always decleare it ! (in init)
``` python
    self.decleare_parameter('<parameter_name', <value>)
```

To read a parameter:
``` python
    my_param = self.get_parameter('my_parameter').get_parameter_value().string_value

    OR

    my_param = <type_cast>(self.get_parameter('my_parameter').value)
```
To decleare a parameter (seldom used):
``` python
    my_new_param = rclpy.parameter.Parameter(
        'my_parameter',
        rclpy.Parameter.Type.STRING,
        'world'
    )
    all_new_parameters = [my_new_param]
    self.set_parameters(all_new_parameters)
```

# Executor
Is an object whic you can store multipole nodes and run them in a multiple thread fation. 

# Note
When you have potentially overlapping callbacks put:
``` python
    from rclpy.callback_groups import MutuallyExclusiveCallbackGroup
    .
    .
    .
    self.ack_sub = self.create_subscription(
            Bool,
            "/ack", 
            self.on_ack, 
            10,
       ---> callback_group = MutuallyExclusiveCallbackGroup()
        )
````
To avoid deadlock.
