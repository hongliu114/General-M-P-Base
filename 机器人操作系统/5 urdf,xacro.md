## URDF
全称：统一机器人描述格式
### 概述
- 对机器人的刚体外观、物理属性等方面建模
- 结合动力学仿真器Gazebo，实现动力学仿真
- 描述语法：
	- 使用xml
	- 主要标签：
		- \<robot>-顶层标签
		- \<link>-刚体单元
		- \<joint>-关节
		- \<gazebo>-gazebo中的仿真参数
### 编程
- urdf文件编写
	- 格式：![[机器人操作系统/assets/Pasted image 20251211025605.png]]
	- 详细内容见[[机器人操作系统/5 urdf,xacro#urdf文件编写]]
- 创建功能包
- 创建子目录
  ![[机器人操作系统/assets/Pasted image 20251211025957.png]]
- 编写launch.py
  AI注释过的例程：
```python
# 导入ROS 2启动系统所需的模块
from launch import LaunchDescription  # 用于构建启动描述的主要类
from launch_ros.actions import Node   # 用于创建ROS节点动作
from launch_ros.parameter_descriptions import ParameterValue  # 参数值描述类
from ament_index_python import get_package_share_directory  # 获取ROS 2功能包路径
from launch.substitutions import Command  # 用于执行shell命令的替换类

# ROS 2启动文件的入口函数，必须命名为generate_launch_description
def generate_launch_description():
    # 【步骤1】获取机器人描述功能包的共享目录路径
    # 通过包名'myrobot_description'自动定位到该包的安装路径
    pkg_share_dir = get_package_share_directory('myrobot_description')
    
    # 【步骤2】拼接URDF模型文件的完整路径
    # 假设URDF文件位于包的urdf子目录下，文件名为myrobot.urdf
    urdf_file = pkg_share_dir + '/urdf/myrobot.urdf'
    
    # 【步骤3】处理URDF/Xacro文件
    # 使用xacro工具解析URDF文件（支持xacro宏和变量扩展）
    urdf_string = ParameterValue(
        # Command用于执行shell命令：'xacro 文件路径' 将xacro转换为URDF字符串
        Command(['xacro ', urdf_file]),
        value_type=str  # 明确指定返回值为字符串类型
    )
    # 此时urdf_string包含了机器人模型的完整URDF描述（已解析的字符串形式）
    
    # 【步骤4】创建robot_state_publisher节点
    # 这是ROS 2中负责发布机器人状态的核心节点
    robot_state_publisher_node = Node(
        package='robot_state_publisher',       # 节点所属的ROS包名
        executable='robot_state_publisher',    # 要执行的可执行文件名
        parameters=[{                           # 传递给节点的参数列表
            # 关键参数：将解析后的URDF模型字符串传递给节点
            'robot_description': urdf_string
        }]
    )
    # robot_state_publisher节点的作用：
    # 1. 从robot_description参数读取完整的URDF机器人模型
    # 2. 订阅/joint_states话题（获取各关节的实时状态）
    # 3. 发布/tf和/tf_static话题（计算并发布所有坐标系间的变换关系）
    # 4. 让RViz等可视化工具能够正确显示机器人模型及其运动
    
    # 【步骤5】构建并返回启动描述
    # 创建包含所有要启动节点的LaunchDescription对象
    return LaunchDescription([
        robot_state_publisher_node  # 将节点添加到启动列表中
    ])
```
- 修改setup.py
  AI注释过的代码：
```python
# 定义数据文件安装规则：将源文件复制到指定安装位置
data_files=[
    # 注册包到ROS 2资源索引
    ('share/ament_index/resource_index/packages', ['resource/' + package_name]),
    
    # 安装package.xml元数据文件
    ('share/' + package_name, ['package.xml']),
    # 安装launch目录下所有启动文件
    ('share/' + package_name + '/launch', glob('launch/*')),
    # 安装urdf目录下所有机器人模型文件
    ('share/' + package_name + '/urdf', glob('urdf/*')),
    # 安装rviz目录下所有可视化配置文件
    ('share/' + package_name + '/rviz', glob('rviz/*')),
    # 安装meshes目录下所有3D网格模型文件
    ('share/' + package_name + '/meshes', glob('meshes/*')),
],
```
- 为launch添加rviz节点![[机器人操作系统/assets/Pasted image 20251211032820.png]]
- 
#### urdf文件编写
##### \<robot>文件
```xml
<?xml version="1.0"?>
<robot name="edu_robot">
<!-- 这里包含定义这个机器人所有元素的代码 -->
</robot>
```
说明版本，机器人取名。

写了个包含各种要素的 AI 注释代码
```xml
<?xml version="1.0"?>
<robot name="simple_robot">

  <!-- 定义颜色材质 -->
  <material name="blue">
    <color rgba="0 0 0.8 1"/>  <!-- 蓝色 -->
  </material>
  <material name="red">
    <color rgba="0.8 0 0 1"/>  <!-- 红色 -->
  </material>
  
  <!-- 1. 底盘link - 基础圆柱体 -->
  <link name="base_link">
    <visual>
      <geometry>
        <cylinder length="0.1" radius="0.2"/>  <!-- 圆柱体 -->
      </geometry>
      <material name="blue"/>
    </visual>
  </link>
  <!-- 2. 身体link - 长方体，使用坐标偏移 -->
  <link name="body">
    <visual>
      <geometry>
        <box size="0.15 0.1 0.3"/>  <!-- 长方体 -->
      </geometry>
      <material name="red"/>
      <!-- 将身体向上移动，放在底盘上方 -->
      <origin xyz="0 0 0.2"/>
    </visual>
  </link>
  <!-- 3. 头部link - 球体 -->
  <link name="head">
    <visual>
      <geometry>
        <sphere radius="0.08"/>  <!-- 球体 -->
      </geometry>
      <!-- 没有指定材质，使用默认颜色 -->
    </visual>
  </link>

  <!-- 关节连接 -->
  <!-- 固定关节：身体固定在底盘上 -->
  <joint name="base_to_body" type="fixed">
    <parent link="base_link"/>
    <child link="body"/>
  </joint>
  <!-- 旋转关节：头部可以转动 -->
  <joint name="body_to_head" type="continuous">
    <parent link="body"/>
    <child link="head"/>
    <!-- 头部位于身体上方 -->
    <origin xyz="0 0 0.25"/>
    <!-- 绕Z轴旋转 -->
    <axis xyz="0 0 1"/>
  </joint>

</robot>
```

上面的link中还可补充碰撞属性<collision\>，物理属性\<inertial>

- \<joint>关节类型
	- continous-连续旋转
	- prismatic-沿轴滑动
	- revolute-有限旋转
	- fixed-固定
##### 检查urdf文件语法
```xml
check_urdf </path/to/file.urdf>
```
## XACRO
全称：可扩展标记语言(XML)宏(macro)
### 概述
优点：
- 更短小、更易读、便于复用代码（模块化）
- 提供编程接口(常量、变量、数学公式或条件语句)，增强XML的表达力
特点：
- 需要xacro工具对.xacro进行解析
### 三大核心语法
#### \<xacro:property>常量
定义可重复用的，统一修改复制的常量值

例程：
```xml
<!-- 定义常量 -->
<xacro:property name="wheel_radius" value="0.1" />
<xacro:property name="body_length" value="0.5" />
<xacro:property name="pi" value="3.14159" />

<!-- 使用常量 -->
<link name="wheel">
  <visual>
    <geometry>
      <cylinder length="0.05" radius="${wheel_radius}"/>
    </geometry>
  </visual>
</link>
```
#### \<xacro:macro>宏定义
将部件定义为可重复使用的宏，类似函数。

例程：
```xml
<!-- 定义一条机器腿的宏 -->
<xacro:macro name="robot_leg" params="prefix parent_link reflect">
  <link name="${prefix}_leg">
    <visual>
      <geometry><cylinder length="0.3" radius="0.05"/></geometry>
    </visual>
  </link>
  
  <joint name="${parent_link}_to_${prefix}_leg" type="fixed">
    <parent link="${parent_link}"/>
    <child link="${prefix}_leg"/>
    <origin xyz="0 ${0.2*reflect} 0.15"/>
  </joint>
</xacro:macro>

<!-- 使用宏实例化两条腿 -->
<xacro:robot_leg prefix="left" parent_link="base_link" reflect="1"/>
<xacro:robot_leg prefix="right" parent_link="base_link" reflect="-1"/>
```
#### \<xacro:include>文件包含
将机器人的不同部分拆分到不同文件，有一个主文件负责组装

例程：
```xml
<!-- 主文件: robot.xacro -->
<?xml version="1.0"?>
<robot name="edu_robot" xmlns:xacro="http://www.ros.org/wiki/xacro">

  <!-- 包含其他组件 -->
  <xacro:include filename="$(find myrobot_description)/urdf/pebble_camera_usb.xacro"/>
  <xacro:include filename="$(find myrobot_description)/urdf/pebble_leg.xacro"/>
  <xacro:include filename="$(find myrobot_description)/urdf/pebble_gripper.xacro"/>

  <!-- 底盘基础结构 -->
  <link name="base_link">...</link>

  <!-- 实例化组件 -->
  <xacro:pebble_leg base_link_name="base_link" prefix="left" reflect="1"/>
  <xacro:camera_usb base_link_name="head" joint_rpy="0 0 0" joint_xyz="0.15 0 0.11"/>
</robot>
```
