## 意义
- 无须手动多次进行ros2 run，一键启动要运行的东西
  `ros2 launch [<package-name>] <launch-file-name>`
- 可设置启动条件、设置节点参数、设置环境变量
- 可包含其他 launch 脚本
- 应位于src/\<package\>/launch中

会涉及到.py .xml .yaml格式文件
## 基本格式
- 导入LaunchDescription类
```python
from launch import LaunchDescription
```
- 进行全局常量、函数、工具定义
- 定义回调函数（就是launch启动过程中跑的函数）
```python
def generate_launch_description( ):
```
- 在回调函数中，写所有需要进行的动作，如启动节点（使用Node类）等，这些动作称为actions
- 返回LaunchDescription对象，括号里面装刚才定义的 actions
```python
return LaunchDescription([ … ])
```

在此过程中可能用到的函数：
![[机器人操作系统/assets/Pasted image 20251211013223.png]]
## 具体使用与编程
### 使用launch文件启动Node节点
- 导入所需的类
```python
from launch import LaunchDescription
from launch_ros.actions import Node
```
- 在回调函数generate_launch_description()中生成node实例
```python
def generate_launch_description( ):
      <node-ins>=Node(<初始化列表参数>)
```
	列表参数要有：
	•package='<package-name>'功能包名
	•executable='<executable-name>'可执行文件名
	•name='node-name'节点名（可选）
	•parameters=[{<param-dic>},{<param-dic>},…]参数列表（可选）
	•arguments=['<arg1>','<arg2>',…]命令行参数（可选）
	•output='log'（默认）或'screen'或'both'输出控制（可选）
- return返回，结束函数
```python
return LaunchDescription([
      <node-ins>,
      …,
])
```
- 设置setup.py
```python
data_files=[
    ('<目标文件夹路径>', ['<待复制文件路径>','<…>']),
    (…),
]
```
- 构建，运行
### 使用launch文件运行 shell 命令
同样是在回调函数generate_launch_description()中编写。

格式：
launch文件最开头需要引入：
`from launch.actions import ExecuteProcess`
回调函数内：
```python
<cmd-ins>= ExecuteProcess(
    cmd = ['<cmd-str>',
           '<cmd-str>',
           …
           ]
     output = 'screen'  or 'log' or 'both',
     shell = True or False
```
cmd-str是要输入运行的字符串，shell是决定是否按 shell 特性运行

例程：
```python
from launch import LaunchDescription
from launch.actions import ExecuteProcess

def generate_launch_description():
    cmd1_action = ExecuteProcess(
        cmd = [
            'echo',
            '$AMENT_PREFIX_PATH',
        ],
        output = 'screen', shell = True
    )
    cmd2_action = ExecuteProcess(
        cmd = [
            'ros2',
            'run',
            'py_yanghui_triangle','yanghui'
        ],
        output = 'screen', shell = True
    )
    return LaunchDescription([
        cmd1_action,
        cmd2_action
    ])
```
### 实现启动参数配置
要用到的两个关键的类：
```DeclareLaunchArgument```-定义参数
`LaunchConfiguration`-获取实际参数值

- 声明定义参数
```python
from launch.actions import DeclareLaunchArgument

# 声明一个背景色红色分量的参数
decl_bg_r = DeclareLaunchArgument(
    name='bg_r',                    # 参数名称
    default_value='255',            # 默认值（字符串类型）
    description='Red color for background',  # 参数描述
    choices=['0', '128', '255']     # 可选值限制（可选）
)
```
- 获取参数值
```python
from launch.substitutions import LaunchConfiguration

# 获取参数的值
red_value = LaunchConfiguration('bg_r')  # 获取名为'bg_r'的参数值
green_value = LaunchConfiguration('bg_g')
blue_value = LaunchConfiguration('bg_b')
```
- 使用参数值
```python
from launch_ros.actions import Node

# 在节点配置中使用参数
turtle_node = Node(
    package='turtlesim',
    executable='turtlesim_node',
    parameters=[{
        'background_r': red_value,    # 使用参数值
        'background_g': green_value,
        'background_b': blue_value
    }]
)
```
### 实现文件包含
例如包含其他launch文件

两个需要用到的关键的类：
`IncludeLaunchDescription`- 包含动作
`PythonLaunchDescriptionSource`-launch文件源，将launch文件包装成标准格式

例程：
```python
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch import LaunchDescription
from ament_index_python import get_package_share_directory
import os

def generate_launch_description():
    # 1. 包含第一个Launch文件（机器狗控制）
    include_dog_action = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(
                get_package_share_directory('spotmicro_dog'),  # 机器狗功能包
                'launch',           # launch文件夹
                'spot_launch.py'    # 机器狗的主Launch文件
            )
        )
    )
    
    # 2. 包含第二个Launch文件（乌龟模拟器），并传递参数
    include_turtle_action = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            os.path.join(
                get_package_share_directory('launch_study'),  # 当前功能包
                'launch',           # launch文件夹  
                'param_launch.py'   # 之前创建的参数化Launch文件
            )
        ),
        # 关键：向被包含的Launch文件传递参数
        launch_arguments=[
            ('bg_r', '100'),    # 设置背景红色=100
            ('bg_g', '128'),    # 设置背景绿色=128
            ('bg_b', '32')      # 设置背景蓝色=32
        ]
    )
    
    # 3. 返回包含动作
    return LaunchDescription([
        include_dog_action,      # 启动机器狗系统
        include_turtle_action    # 启动带特定参数的乌龟模拟器
    ])
```
### 实现分组设置
将节点的 action 分组，方便管理

所需的类：
`GroupAction`-分组动作
`PushRosNamespace`-命名空间推送,位组内的节点加上命名前缀

例程：
```python
from launch import LaunchDescription
from launch_ros.actions import Node
from launch_ros.actions import PushRosNamespace
from launch.actions import GroupAction

def generate_launch_description():
    # 1. 创建三个独立的乌龟模拟器节点
    turtle1 = Node(
        package='turtlesim',
        executable='turtlesim_node', 
        name='t1'  # 原本会叫 /t1
    )
    
    turtle2 = Node(
        package='turtlesim',
        executable='turtlesim_node',
        name='t2'  # 原本会叫 /t2  
    )
    
    turtle3 = Node(
        package='turtlesim',
        executable='turtlesim_node',
        name='t3'  # 原本会叫 /t3
    )
    
    # 2. 创建第一个分组：将turtle1和turtle2放入"Group_One"命名空间
    g1 = GroupAction([
        PushRosNamespace('Group_One'),  # 设置命名空间前缀
        turtle1,  # 现在全名: /Group_One/t1
        turtle2   # 现在全名: /Group_One/t2
    ])
    
    # 3. 创建第二个分组：将turtle3放入"Group_Two"命名空间  
    g2 = GroupAction([
        PushRosNamespace('Group_Two'),  # 不同的命名空间
        turtle3   # 现在全名: /Group_Two/t3
    ])
    
    # 4. 返回包含两个分组的Launch描述
    return LaunchDescription([g1, g2])
```
### 事件启动设置
略
## 作业
> 功能包”chd_pkg”中有节点执行脚本”chd_execute”，该节点中有话题名为”/chd_topic”。请用命令行ros2 run的方式执行该脚本，要求：
（1）指定节点名为”new_name_node”；
（2）指定命名空间名为”/chd_ns”。
（3）将原话题名重映射为”/chd_new_topic”
该命令行应如何撰写？
如果用launch.py运行该节点，请编写该python文件（注，本题有自学内容，请学习重映射相关参数的用法）。

命令行：
```python
ros2 run chd_pkg chd_execute --ros-args -r __node:=new_name_node -r __ns:=/chd_ns  -r  /chd_ns/chd_topic:=/chd_new_topic
```
launch.py：
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    the_node = Node(
        package = 'chd_pkg',
        executable = 'chd_execute',
        name = 'new_name_node',
        namespace= 'chd_ns',
        remappings=[
            ('chd_topic','/chd_new_topic'),
        ],
    )   

    return LaunchDescription([
        the_node
    ])
```
> 请撰写”chd_launch.py”文件内容，包含配置参数”my_arg”，并通过ExecuteProcess执行echo命令打印该参数的结果。要求该参数默认值为”1234”，通过命令行传入参数为”abcd”，请在代码后贴执行结果图。

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument,ExecuteProcess
from launch.substitutions import LaunchConfiguration

def generate_launch_description():
    launch_conf=LaunchConfiguration('my_arg')
    decl_my_arg=DeclareLaunchArgument(
        name='my_arg',
        default_value='1234',
    )
    e_ac=ExecuteProcess(
            cmd=['echo', 'Value of my_arg:', launch_conf],
            output='screen',
        )

    return LaunchDescription([
        decl_my_arg,
        e_ac
    ])
```
