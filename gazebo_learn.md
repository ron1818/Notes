### joint state publisher in .rviz ###
fake joint state publisher gui, later will get joint states from gazebo
by *gazebo_ros_control*

```xml
<!-- send fake joint values -->
  <node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="TRUE"/>
</node>

<!-- Combine joint values -->
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="state_publisher"/>
```
*todo*: find the plugin:
`<plugin name="kingfisher_dynamic" filename="libkingfisher_gazebo_plugins.so">`
### 
