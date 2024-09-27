---
title: "ros学习记录"
date: 2024-03-24 19:00:43
categories: "机器人"
tags: ["Cpp", "ros"]
summary: "ros学习记录，Cpp"
---

## 安装

### 配置软件源

打开软件与更新，配置 ubuntu 的软件和更新，并且允许安装未经认证的软件

官方默认安装源

```shell
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

清华的安装源

```shell
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

中科大安装源

```shell
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

### 设置key

在终端中输入

```shell
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

### 安装ros

首先需要更新源和 apt，终端中输入

```shell
sudo apt update
```

然后开始安装 ros，终端输入

```shell
sudo apt install ros-noetic-desktop-full
```

### 配置环境变量

在终端中输入指令

```shell
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 卸载指令

```shell
sudo apt remove ros-noetic-*
```

### 初始化 rosdep

在命令行中输入指令

```shell
sudo rosdep init
rosdep update
```

有些版本老的ros会报错，因为是缺少依赖，那就需要安装构建依赖

首先安装构建依赖的相关工具

```shell
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo apt install python3-rosdep
```

如果不是依赖的问题，就可以看看 [ros安装](http://www.autolabor.com.cn/book/ROSTutorials/chapter1/12-roskai-fa-gong-ju-an-zhuang/124-an-zhuang-ros.html)


## ros介绍

### ros的文件系统

![](http://www.autolabor.com.cn/book/ROSTutorials/assets/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F.jpg)

### ros的指令

**rosnode**获取节点信息的指令

- `rosnode ping` 测试到节点的连接状态
- `rosnode list` 列出活动节点
- `rosnode info` 打印节点信息
- `rosnode machine` 列出指定设备上节点
- `rosnode kill` 杀死某个节点
- `rosnode cleanup` 清除不可连接的节点

**rostopic**获取话题信息的指令

- `rostopic bw` 显示主题使用的带宽
- `rostopic delay` 显示带有 header 的主题延迟
- `rostopic echo` 打印消息到屏幕
- `rostopic find` 根据类型查找主题
- `rostopic hz` 显示主题的发布频率
- `rostopic info` 显示主题相关信息
- `rostopic list` 显示所有活动状态下的主题
- `rostopic pub` 将数据发布到主题
- `rostopic type` 打印主题类型

**rosmsg**获取有关消息类型信息的指令

- `rosmsg show` 显示消息描述
- `rosmsg info` 显示消息信息
- `rosmsg list` 列出所有消息
- `rosmsg md5` 显示 md5 加密后的消息
- `rosmsg package` 显示某个功能包下的所有消息
- `rosmsg packages` 列出包含消息的功能包

**rosservice**获取服务信息的指令

- `rosservice args` 打印服务参数
- `rosservice call` 使用提供的参数调用服务
- `rosservice find` 按照服务类型查找服务
- `rosservice info` 打印有关服务的信息
- `rosservice list` 列出所有活动的服务
- `rosservice type` 打印服务类型
- `rosservice uri` 打印服务的 ROSRPC uri

**rossrv**显示有关服务类型信息的指令

- `rossrv show` 显示服务消息详情
- `rossrv info` 显示服务消息相关信息
- `rossrv list` 列出所有服务信息
- `rossrv md5` 显示 md5 加密后的服务消息
- `rossrv package` 显示某个包下所有服务消息
- `rossrv packages` 显示包含服务消息的所有包

**rosparam**获取和设置参数信息的指令

- `rosparam set` 设置参数
- `rosparam get` 获取参数
- `rosparam load` 从外部文件加载参数
- `rosparam dump` 将参数写出到外部文件
- `rosparam delete` 删除参数
- `rosparam list` 列出所有参数

**设置命名空间和重映射**

在 `rosrun` 时的节点名称之后添加 `__ns:=namespace` 设置命名空间
  
```shell
rosrun turtlesim turtlesim_node __ns:=/xxx
```

在 `rosrun` 时的节点名称之后添加 `__name:=newname`

```shell
rosrun turtlesim turtlesim_node __name:=t1
rosrun turtlesim turtlesim_node /turtlesim:=t1(不适用于python)
```

**设置话题重映射**

rosrun名称重映射语法: rorun 包名 节点名 话题名:=新话题名称

**设置参数**

rosrun 包名 节点名称 _参数名:=参数值

### 一些需要注意的

在每一种通信都需要依赖于 ros 的句柄，所以就是在实例化各种通信端之前都需要初始化一个句柄

```C++
ros::NodeHandle nh;
```

## 话题通信

话题操作基本思想就是创建两个节点，然后这两个节点需要有一致的话题，然后作为发布方会发送该话题的消息，而接收放就接收该话题的消息，以此实现通信

### 发布方

```C++
// 初始化发布方，话题名称，保存的最大消息数
ros::Publisher pub = nh.advertise<std_msgs::String>("chatter",10);
// 初始化消息
std_msgs::String msg;
msg.data = "message";
// 发布话题
pub.publish(msg);
```

### 订阅方

```C++
// 订阅回调函数
void doMsg(const std_msgs::String::ConstPtr& msg_p);
// 初始化订阅方 话题名称，保存的最大消息数，绑定订阅回调函数
ros::Subscriber sub = nh.subscribe<std_msgs::String>("chatter",10,doMsg);
// 循环接收消息
ros::spin();
```

### 自定义msg

msgs是一个文本文件，每一行分别是字段类型和字段名称

首先在功能包下新建 msg 目录，添加文件 `xxx.msg` xxx是自定义的名称，内容可以自定义，这里暂定为

```CODE
string name
uint16 age
```

在 `package.xml` 中添加编译依赖与执行依赖

```XML
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
```

在 `CMakeLists.txt` 编辑相关配置

- 在 `find_package` 中添加 `message_generation` 与 `std_msgs`
- 在 `add_message_files` 中添加 msg 文件 `xxx.msg`
- 在 `generate_messages` 中添加 `std_msgs`

编译之后会生成一个头文件，位于 `\devel\include\包名\xxx.h` ，并且1需要配置该头文件的路径，然后就可以像 `std_msgs` 一样使用了

## 服务通信

ros 系统负责存储服务端和客户端的注册信息，并且匹配话题相同的服务端和客户端，并且建立连接，之后客户端请求，服务端响应

### 自定义服务器

功能包下新建 srv 目录，添加 `xxx.srv` 文件，内容为（根据自己需求来定义）

```CODE
# 客户端请求时发送的两个数据
int32 num1
int32 num2
---
# 服务器响应发送的数据
int32 sum
```

中间记得加 `---` 来做划分

编辑配置文件，在 `package.xml` 中添加编译依赖与执行依赖

```XML
  <build_depend>message_generation</build_depend>
  <exec_depend>message_runtime</exec_depend>
```

在 `CMakeLists.txt` 中编辑相关配置

- 在 `find_package` 中添加 `message_generation` 与 `std_msgs`
- 在 `add_service_files` 中添加 msg 文件 `xxx.srv`
- 在 `generate_messages` 中添加 `std_msgs`

编译之后会生成一个头文件，位于 `\devel\include\包名\xxx.h` ，并且1需要配置该头文件的路径，然后就可以作为头文件被引入使用了

### 服务端

```C++
// 服务器响应回调函数
bool doReq(demo03_server_client::AddInts::Request& req,demo03_server_client::AddInts::Response& resp);
// 初始化服务端 服务话题名称，接收到请求的回调函数
ros::ServiceServer server = nh.advertiseService("AddInts", doReq);
```

### 客户端

```C++
// 创建客户端对象
ros::ServiceClient client = nh.serviceClient<demo03_server_client::AddInts>("AddInts");
// 等待服务器启动
// 1
ros::service::waitForService("AddInts");
// 2
client.waitForExistence();
// 发送请求的客户端数据
demo03_server_client::AddInts ai;
// 发送请求,返回 bool 值，标记是否成功
bool flag = client.call(ai);
```

## 参数服务器

有两种增删改查的参数操作

### 设置参数

```C++
// 1
ros::NodeHandle::setParam(key, value);
// 2
ros::param::set(key, value);
```

### 获取参数

第一种方式

```C++
ros::NodeHandle::param(key, default_value);
ros::NodeHandle::getParam(key, &value);
ros::NodeHandle::getParamCache(key, &value);
// 获取所有并且存放在vector中
ros::NodeHandle::getParamNames(std::vector<std::string>&);
ros::NodeHandle::hasParam(key);
ros::NodeHandle::searchParam(key, &value);
```

第二种方式

```C++
ros::param::get(key, &value);
ros::param::getCache(key, &value);
ros::param::getParamNames(std::vector<std::string>&);
ros::param::has(key);
ros::param::search(key, &value);```
```

### 删除参数

```C++
ros::NodeHandle::deleteParam(key);
ros::param::del(key);
```

## 动作action

类似于服务通信，并且在运行过程中服务端不断地向客户端反馈数据与状态

### 自定义action消息

需要在导入工程包的时候还要额外导入 `actionlib actionlib_msgs` 等文件，在功能包下新建 `action` 目录，新增 `xxx.action` 文件。

`action` 文件内容由三部分组成

1. 请求目标值
2. 最终响应结果
3. 连续反馈

```CODE
#目标值
int32 num
---
#最终结果
int32 result
---
#连续反馈
float64 progress_bar
```

在 `CMakeLists.txt` 文件的 `add_action_files` 中添加 `xxx.action`

编译之后会在工作空间内生成对应的头文件

### 服务器

```C++
// 创建服务器
actionlib::SimpleActionServer(ros::NodeHandle n, 
                   std::string name, 
                   boost::function<void (const demo01_action::AddIntsGoalConstPtr &)> execute_callback, 
                   bool auto_start);
// 开始服务器
server.start();
```

### 客户端

```C++
actionlib::SimpleActionClient<demo01_action::AddIntsAction> client(nh,"addInts",true);
//等待服务启动
client.waitForServer();
// 目标值（也就是上述定义的action消息的数据
// 处理最终的反馈回调
// 服务器激活反馈回调
// 处理连续反馈回调
client.sendGoal(goal,&done_cb,&active_cb,&feedback_cb);
```

## launch文件

launch 文件是一个 XML 格式的文件，可以启动本地和远程的多个节点，还可以在参数服务器上设置参数

可以使用一个 `launch` 文件就打开多个节点，提高 ROS 程序的启动效果

### 使用步骤

1. 在功能包下添加 `launch` 目录，在目录下面新建 `xxx.launch` 文件，并且编辑文件
2. 在命令行中输入指令 `roslaunch 包名 xxx.launch` 来调用 `launch` 文件，使用 `roslaunch` 指令时会自动打开 `roscore`

### 注意

由于 ros 是多进程的，所以不能保证 `launch` 文件中的节点的启动顺序

### 介绍

**`launch` 标签**
  
是所有 `launch` 文件的根标签，充当其它标签的容器

**`node` 标签属性**

- `pkg="pkgname"` 节点所属的包名
- `type="nodetype"` 节点的类型
- `name="nodename"` 节点名称
- `args="xxx"` 参数可选，是传递给节点的参数
- `machine="machinename"` 机器名称，在指定的机器上启动节点
- `respawn="true or false"` 如果节点退出是否重新启动
- `respawn_delay="N"` 如果上述 `respawn` 为 `true` 则延迟 `N` 秒后重启节点
- `required="true or false"` 该节点是否必要，如果为 `true` 退出将杀死整个 `roslaunch`
- `ns="xxx"` 在指定命名空间中启动节点
- `clear_params="true or false"` 再启动前，删除节点的私有空间的所有参数
- `output="log or screen"` 日志发送的目标，发送到屏幕或者是文件

**`node` 子级标签**

- `env` 环境变量设置
- `remap` 重映射节点名称
- `rosparam` 参数设置
- `param` 参数设置

**`include` 标签属性**

- `file="$(find pkgname)/xxx/xxx.launch"` 要包含的文件路径
- `ns="xxx"` 在指定的命名空间导入文件，可不填

**`include` 子级标签**

- `env` 环境变量设置
- `arg` 将参数传递给被包含的文件

**`remap` 标签**

用于话题重命名

- `from="xxx"` 原始话题名称
- `to="xxx"` 重命名目标名称

**`param` 标签**

- `name="namespace/paramname"` 参数名称，可以包含命名空间
- `value="xxx"` 可选参数，定义参数值，如果省略，必须指定外部文件作为参数来源
- `type="str or ..."` 指定参数类型，如未指定， `roslaunch` 会尝试确定参数类型

**`rosparam` 标签**

- `command="load or dump or delete"` 可选，默认为 `load` ，加载，导出或者删除参数
- `file="$(find xxx)/xxx/xxx..."` 加载或导出到的 `yaml` 文件
- `param="paramname"` 参数名称
- `ns="namespace"` 命名空间

**`group` 标签**

- `ns="namespace"` 可选，命名空间
- `clear_params="true or false"` 可选，启动前是否删除组名称空间内部的所有参数
- 除了 `launch` 标签以外的标签都是它的子标签

**`arg` 标签**

- `name="paramname"` 参数名称
- `default="defaultvalue"` 默认数值，可选
- `value="value"` 数值，可选，不可与 `default` 并存
- `doc="description"` 参数描述说明

## 其它的功能

### 回旋函数

- `spin()` 不断回旋，相当于 `while(1)`
- `spinOnce()` 只回旋一次，没有循环，相当于顺序语句

### 时间

**获取时刻**

```C++
ros::Time right_now = ros::Time::now();
```

**持续时间**

```C++
ros::Duration du(10); // 设置一个持续时间 s，相当于 sleep 函数
du.sleep();
```

**设置运行频率**

```C++
ros::Rate rate(1); // 指定频率
while (true)
{
  rate.sleep(); // 休眠，休眠时间 = 1 / 频率。
}
```

**定时器**

```C++
/**
* \brief 创建一个定时器，按照指定频率调用回调函数。
*
* \param period 时间间隔
* \param callback 回调函数
* \param oneshot 如果设置为 true,只执行一次回调函数，设置为 false,就循环执行
* \param autostart 如果为true，返回已经启动的定时器,设置为 false，需要手动启动，默认为true
*/
ros::Timer timer = nh.createTimer(ros::Duration(0.5), doSomeThing, true)

ros::spin(); //必须 spin 否则程序跑完了且不会运行回调函数
```

### 重映射和命名空间

**别名设置**

```C++
ros::init(argc,argv,"zhangsan",ros::init_options::AnonymousName);
```

**命名空间**

```C++
std::map<std::string, std::string> map;
map["__ns"] = "xxxx";
ros::init(map,"wangqiang");
```

### 话题重映射

rosrun名称重映射语法: rorun 包名 节点名 话题名:=新话题名称

话题的名称与节点的命名空间，节点的名称是有一定关系的，话题名称分为三种类型

**全局**，与节点命名空间平级，以 `/` 开头的节点名称

```C++
ros::Publisher pub = nh.advertise<std_msgs::String>("/chatter",1000);
```

**相对**，与节点名称平级，以非 `/` 开头的节点名称来确定话题名称

```C++
ros::Publisher pub = nh.advertise<std_msgs::String>("chatter",1000);
```

**私有**，是节点名称的子级，以 `~` 开头的名称

```C++
ros::NodeHandle nh("~");
ros::Publisher pub = nh.advertise<std_msgs::String>("chatter",1000);
```

## 后记

这些记录是我在学习 ros 时所记录下来的，可以看看赵老师所做的文档，写的很详细，很不错 [ros学习文档](http://www.autolabor.com.cn/book/ROSTutorials/di-5-zhang-ji-qi-ren-dao-hang/51-tfzuo-biao-bian-huan/512-jing-tai-zuo-biao-bian-huan.html)
