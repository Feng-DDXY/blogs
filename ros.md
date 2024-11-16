**使用版本：Ubuntu 16.04  ROS Kinetic（已安装）**



## 在turtlesim_node上生成三只小乌龟

>以下步骤自行完成（仅附终端命令）
>1.在已有工作空间下创建功能包ch10_tutorials1
>2.导入std_msgs和roscpp作为该功能包的依赖项
>3.在功能包ch10_tutorials1的src文件夹下创建并打开turtle2.cpp文件

```bash
catkin_create_pkg ch10_tutorials1 std_msgs roscpp
gedit turtle2.cpp
```

输入以下代码：

```cpp
#include <string>
#include <turtlesim/Kill.h>
#include <turtlesim/Spawn.h>
#include <ros/ros.h>

using namespace std;
int main(int argc, char * * argv)
{
    string names[] ={"turtle1_0","turtle2_0", "turtle3_0"}; //定义小海龟的名称（3只）
    ros::init(argc, argv, "turtle2"); 
    ros::NodeHandle n; //初始化节点句柄
    ros::service::waitForService("spawn"); //等待调用服务生成小海龟
    ros::ServiceClient client_kill = n.serviceClient <turtlesim::Kill>("kill");
    turtlesim::Kill kill_name;
    kill_name.request.name ="turtle1";
    client_kill.call(kill_name);
    ros::ServiceClient client = n.serviceClient <turtlesim::Spawn>("spawn");
    turtlesim::Spawn turtle; 
    for(int i =0;i<3;i++) //设置三只小海龟的位置（改循环数量）
    {
        turtle.request.name="x"+names[i];
        turtle.request.x =i+5;
        turtle.request.y =5;
        turtle.request.theta = 1.57;
        client.call(turtle); //发送服务请求，以调用服务
    }
}

```

打开功能包ch10_tutorials1下的CMakeList.txt添加如下语句

```bash
add_executable(turtle2 src/turtle2.cpp)
target_link_libraries(turtle2 ${catkin_LIBRARIES})
```

切换到工作空间下编译工作空间：
```powershell
cd ~/catkin_ws
catkin_make
```

同时显示3只小乌龟已经完成，分别在3个终端输入以下代码展示效果：

```bash
roscore
rosrun turtlesim turtlesim_node
rosrun ch10_tutorials1 turtle2
```
效果如下：
![在这里插入图片描述](./assets/39e7c6371cb3449fbce28f880daafcd3.png)

## 实现2只小乌龟同时编队运动（路径固定）

>以下步骤自行完成（仅附终端命令）
>1.在功能包ch10_tutorials1下新建消息文件夹msg
>2.在消息文件夹msg下新建msg10.msg
>3.在msg10.msg内写入以下消息类型

```bash
float64 linear_velocity1
float64 angular_velocity1
float64 linear_velocity2
float64 angular_velocity2
```
![在这里插入图片描述](./assets/350400dac582477586746c8a87afda93.png)
在功能包内的package.xml 创建以下语句：

```xml
<build_depend>message_generation</build_depend> 
<build_export_depend>message_runtime</build_export_depend>
<exec_depend>message_runtime</exec_depend>
```
![在这里插入图片描述](./assets/1786cd9623ce4d27909aa91c4fa907cf.png)

在功能包内的CMakelists.txt 创建以下语句：（建议ctrl+f搜索）
  ```
find_package(catkin REQUIRED COMPONENTS 
roscpp 
std_msgs 
message_generation
)
  ```
 ```
add_message_files( 
FILES 
msg10.msg 
) 
 ```
  ```
generate_messages( 
DEPENDENCIES 
std_msgs
)
  ```

  ```
catkin_package( 
CATKIN_DEPENDS roscpp std_msgs message_runtime 
)
  ```

切换到工作空间下编译工作空间：
```powershell
cd ~/catkin_ws
catkin_make
```
在\ch10_tutorials1\src下新建ch6.cpp文件，并编写以下代码：

```cpp
#include <ros/ros.h>
#include "geometry_msgs/Twist.h"
#include "ch10_tutorials1/msg10.h"
#include <iostream>

using namespace ros;

int main(int argc, char **argv)
{
    // 初始化ROS节点
    init(argc, argv, "ch6");

    // 创建一个节点句柄
    NodeHandle it1;
    NodeHandle it2;
    NodeHandle n;
    // 创建发布者
    Publisher pub1 = it1.advertise<geometry_msgs::Twist>("/xturtle1_0/cmd_vel", 100);
    Publisher pub2 = it2.advertise<geometry_msgs::Twist>("/xturtle2_0/cmd_vel", 100);
    Publisher pub = n.advertise<ch10_tutorials1::msg10>("message", 1000);

    // 设置循环频率
    Rate rate(1);

    // 主循环
    while (ok())
    {
        // 创建并设置turtle1的运动参数
        geometry_msgs::Twist msg1_0;
        msg1_0.linear.x = 0.5;
        msg1_0.angular.z = 1.0;
        pub1.publish(msg1_0);

        // 创建并设置turtle2的运动参数
        geometry_msgs::Twist msg2_0;
        msg2_0.linear.x = 2;
        msg2_0.angular.z = 1;
        pub2.publish(msg2_0);

        // 创建自定义消息并设置参数
        ch10_tutorials1::msg10 msg;
        msg.linear_velocity1 = msg1_0.linear.x;
        msg.angular_velocity1 = msg1_0.angular.z;
        msg.linear_velocity2 = msg2_0.linear.x;
        msg.angular_velocity2 = msg2_0.angular.z;

        // 发布自定义消息
        pub.publish(msg);

        // 打印信息到终端（可选）
        ROS_INFO("turtle1: [%f] [%f]", msg.linear_velocity1, msg.angular_velocity1);
        ROS_INFO("turtle2: [%f] [%f]", msg.linear_velocity2, msg.angular_velocity2);

        // 睡眠以保持频率
        rate.sleep();
    }

    return 0;
}
```

将turtle2.cpp代码改成这个：（只需要2只小乌龟）

```cpp
#include <string>
#include <turtlesim/Kill.h>
#include <turtlesim/Spawn.h>
#include <ros/ros.h>

using namespace std;
int main(int argc, char * * argv)
{
    string names[] ={"turtle1_0","turtle2_0"}; //定义小海龟的名称
    ros::init(argc, argv, "turtle2"); 
    ros::NodeHandle n; //初始化节点句柄
    ros::service::waitForService("spawn"); //等待调用服务生成两只小海龟
    ros::ServiceClient client_kill = n.serviceClient <turtlesim::Kill>("kill");
    turtlesim::Kill kill_name;
    kill_name.request.name ="turtle1";
    client_kill.call(kill_name);
    ros::ServiceClient client = n.serviceClient <turtlesim::Spawn>("spawn");
    turtlesim::Spawn turtle; 
    for(int i =0;i<2;i++) //设置两只小海龟的位置
    {
        turtle.request.name="x"+names[i];
        turtle.request.x =i+5;
        turtle.request.y =5;
        turtle.request.theta = 1.57;
        client.call(turtle); //发送服务请求，以调用服务
    }
}

```
打开功能包ch10_tutorials1下的CMakeList.txt添加如下语句

```bash
add_executable(ch6 src/ch6.cpp)
target_link_libraries(ch6 ${catkin_LIBRARIES})
add_dependencies(ch6 ch10_tutorials1_generate_messages_cpp)
```

切换到工作空间下编译工作空间：
```powershell
cd ~/catkin_ws
catkin_make
```

分别在4个终端输入以下指令，即可看到2个小海龟绕圈运行：

```bash
roscore
rosrun turtlesim turtlesim_node
rosrun ch10_tutorials1 turtle2
rosrun ch10_tutorials1 ch6
```
![在这里插入图片描述](./assets/e33d54bf8c91418188d5c908490e2348.png)


## 两只小乌龟跟随运动（单文件实现）
本文使用TF功能包实现乌龟坐标广播以及跟随，故需要加入TF相关依赖
在功能包内的CMakelists.txt 编写以下语句：（建议ctrl+f搜索）
  ```
find_package(catkin REQUIRED COMPONENTS 
roscpp 
std_msgs 
message_generation
tf2_ros
)
  ```
 ```
catkin_package( 
CATKIN_DEPENDS roscpp  rospy std_msgs message_runtime tf2_ros 
)
 ```

  ```
add_executable(turtle3 src/turtle3.cpp)
target_link_libraries(turtle3 ${catkin_LIBRARIES})
  ```

==其中turtle3是你的cpp文件名，如果以其他的命名需要修改（我这里用的turtle4）==

![在这里插入图片描述](./assets/159c9c9de14c497e809934a8b4eac7d4-1731713132743-31.png)
![在这里插入图片描述](./assets/1a92c4ffb20d496386725ed9f2bcc365-1731713132743-33.png)
![在这里插入图片描述](./assets/6fd18961bd2e41c1976b8ae495a4e31b-1731713132743-35.png)
![在这里插入图片描述](./assets/3804863b18e241498a4486ded9bd66b8-1731713132743-37.png)

在功能包内的package.xml 创建以下语句：
```xml
  <depend>tf2_ros</depend>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ab5fe7a26fdd4dfbb0d326e22892273d.png)
在\ch10_tutorials1\src下新建turtle3.cpp文件，并编写以下代码：

```cpp
#include "ros/ros.h"
#include "turtlesim/Pose.h"
#include "turtlesim/Spawn.h"
#include "tf2_ros/transform_broadcaster.h"
#include "tf2_ros/transform_listener.h"
#include "geometry_msgs/TransformStamped.h"
#include "geometry_msgs/Twist.h"
#include "tf2/LinearMath/Quaternion.h"
#include <cmath>

std::string turtle_name = "turtle1"; //代表要跟随乌龟的名字

void doPose(const turtlesim::Pose::ConstPtr& pose, const std::string& turtle_name) {
    static tf2_ros::TransformBroadcaster broadcaster;
    geometry_msgs::TransformStamped tfs;
    tfs.header.frame_id = "world";
    tfs.header.stamp = ros::Time::now();
    tfs.child_frame_id = turtle_name;

    //设定2只乌龟发布tf坐标的位置
    if (turtle_name == "turtle1") {
        tfs.transform.translation.x = (pose->x) + 1.0;
        tfs.transform.translation.y = (pose->y) + 1.0;
    } else if (turtle_name == "turtle2") {
        tfs.transform.translation.x = pose->x;
        tfs.transform.translation.y = pose->y;
    }
    tfs.transform.translation.z = 0.0;

    tf2::Quaternion qtn;
    qtn.setRPY(0, 0, pose->theta);
    tfs.transform.rotation.x = qtn.getX();
    tfs.transform.rotation.y = qtn.getY();
    tfs.transform.rotation.z = qtn.getZ();
    tfs.transform.rotation.w = qtn.getW();

    broadcaster.sendTransform(tfs);
}

int main(int argc, char *argv[]) {
    setlocale(LC_ALL, "");
    ros::init(argc, argv, "pub_sub_tf");
    ros::NodeHandle nh;
    ros::ServiceClient spawn_client = nh.serviceClient<turtlesim::Spawn>("/spawn");

    //新建2只乌龟
    turtlesim::Spawn spawn_msg;
    spawn_msg.request.x = 5.0;
    spawn_msg.request.y = 5.0;
    spawn_msg.request.theta = 0.0;
    spawn_msg.request.name = "turtle1";
    if (spawn_client.call(spawn_msg)) {
        ROS_INFO("Spawned turtle1 at (5, 5)");
    } else {
        ROS_ERROR("Failed to spawn turtle1");
        return -1;
    }

    spawn_msg.request.x = 8.0;
    spawn_msg.request.y = 5.0;
    spawn_msg.request.theta = 0.0;
    spawn_msg.request.name = "turtle2";
    if (spawn_client.call(spawn_msg)) {
        ROS_INFO("Spawned turtle2 at (8, 5)");
    } else {
        ROS_ERROR("Failed to spawn turtle2");
        return -1;
    }

    ros::Publisher pub = nh.advertise<geometry_msgs::Twist>("/turtle2/cmd_vel", 1000);

    tf2_ros::Buffer buffer;
    tf2_ros::TransformListener listener(buffer);

    //发布turtle1和turtle2的坐标（使用doPose函数）
    ros::Subscriber sub1 = nh.subscribe<turtlesim::Pose>("turtle1/pose", 1000, boost::bind(doPose, _1, "turtle1"));
    ros::Subscriber sub2 = nh.subscribe<turtlesim::Pose>("turtle2/pose", 1000, boost::bind(doPose, _1, "turtle2"));

    //ros::Duration(2.0).sleep();  //可以加个延迟，但是还是会有WARN（从3个变成1个）

    ros::Rate rate(10);
    while (ros::ok()) {
        try {
            // 获取 turtle1 相对 turtle2 的坐标信息
            geometry_msgs::TransformStamped tfs = buffer.lookupTransform("turtle2", "turtle1", ros::Time(0));

            // 根据坐标信息生成速度信息
            geometry_msgs::Twist twist;
            twist.linear.x = 0.5 * sqrt(pow(tfs.transform.translation.x, 2) + pow(tfs.transform.translation.y, 2));
            twist.angular.z = 4 * atan2(tfs.transform.translation.y, tfs.transform.translation.x);
            pub.publish(twist);
        } catch (const tf2::TransformException& e) {
            ROS_WARN("错误提示:%s", e.what());
        }

        rate.sleep();
        ros::spinOnce();
    }

    return 0;
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0447e9ddc6e945c1be82cc9bf9acbfd6.png)


切换到工作空间下编译工作空间：
```powershell
cd ~/catkin_ws
catkin_make
```

使用方法：

分别在4个终端输入以下指令：

```bash
roscore
rosrun turtlesim turtlesim_node
rosservice call /kill "turtle1"
rosrun ch10_tutorials1 turtle4
```
**rosservice call /kill "turtle1"是因为代码里没有写kill小乌龟代码，手动在终端里去掉，不然rosrun turtlesim turtlesim_node会自动生成一只turtle1乌龟，导致重名报错**

此时窗口里会出现2个小海龟，打开另一个终端输入以下指令控制其中一只运动

```bash
rosrun turtlesim turtle_teleop_key
```
即可看到一个乌龟跟随另一个了
**我的代码里写的是跟随的(pose->x) + 1.0;即一直跟随在相对位置右上角**

![在这里插入图片描述](./assets/4bd8e6e442f5403395694622410916e5-1731713167602-63.png)**其中turtle3是你的cpp文件名，如果以其他的命名需要修改（我这里用的turtle4)**

![在这里插入图片描述](./assets/159c9c9de14c497e809934a8b4eac7d4.png)
![在这里插入图片描述](./assets/1a92c4ffb20d496386725ed9f2bcc365.png)
![在这里插入图片描述](./assets/6fd18961bd2e41c1976b8ae495a4e31b.png)
![在这里插入图片描述](./assets/3804863b18e241498a4486ded9bd66b8.png)

在功能包内的package.xml 创建以下语句：
```xml
  <depend>tf2_ros</depend>
```
![在这里插入图片描述](./assets/ab5fe7a26fdd4dfbb0d326e22892273d.png)
在\ch10_tutorials1\src下新建turtle3.cpp文件，并编写以下代码：

```cpp
#include "ros/ros.h"
#include "turtlesim/Pose.h"
#include "turtlesim/Spawn.h"
#include "tf2_ros/transform_broadcaster.h"
#include "tf2_ros/transform_listener.h"
#include "geometry_msgs/TransformStamped.h"
#include "geometry_msgs/Twist.h"
#include "tf2/LinearMath/Quaternion.h"
#include <cmath>

std::string turtle_name = "turtle1"; //代表要跟随乌龟的名字

void doPose(const turtlesim::Pose::ConstPtr& pose, const std::string& turtle_name) {
    static tf2_ros::TransformBroadcaster broadcaster;
    geometry_msgs::TransformStamped tfs;
    tfs.header.frame_id = "world";
    tfs.header.stamp = ros::Time::now();
    tfs.child_frame_id = turtle_name;

    //设定2只乌龟发布tf坐标的位置
    if (turtle_name == "turtle1") {
        tfs.transform.translation.x = (pose->x) + 1.0;
        tfs.transform.translation.y = (pose->y) + 1.0;
    } else if (turtle_name == "turtle2") {
        tfs.transform.translation.x = pose->x;
        tfs.transform.translation.y = pose->y;
    }
    tfs.transform.translation.z = 0.0;

    tf2::Quaternion qtn;
    qtn.setRPY(0, 0, pose->theta);
    tfs.transform.rotation.x = qtn.getX();
    tfs.transform.rotation.y = qtn.getY();
    tfs.transform.rotation.z = qtn.getZ();
    tfs.transform.rotation.w = qtn.getW();

    broadcaster.sendTransform(tfs);
}

int main(int argc, char *argv[]) {
    setlocale(LC_ALL, "");
    ros::init(argc, argv, "pub_sub_tf");
    ros::NodeHandle nh;
    ros::ServiceClient spawn_client = nh.serviceClient<turtlesim::Spawn>("/spawn");

    //新建2只乌龟
    turtlesim::Spawn spawn_msg;
    spawn_msg.request.x = 5.0;
    spawn_msg.request.y = 5.0;
    spawn_msg.request.theta = 0.0;
    spawn_msg.request.name = "turtle1";
    if (spawn_client.call(spawn_msg)) {
        ROS_INFO("Spawned turtle1 at (5, 5)");
    } else {
        ROS_ERROR("Failed to spawn turtle1");
        return -1;
    }

    spawn_msg.request.x = 8.0;
    spawn_msg.request.y = 5.0;
    spawn_msg.request.theta = 0.0;
    spawn_msg.request.name = "turtle2";
    if (spawn_client.call(spawn_msg)) {
        ROS_INFO("Spawned turtle2 at (8, 5)");
    } else {
        ROS_ERROR("Failed to spawn turtle2");
        return -1;
    }

    ros::Publisher pub = nh.advertise<geometry_msgs::Twist>("/turtle2/cmd_vel", 1000);

    tf2_ros::Buffer buffer;
    tf2_ros::TransformListener listener(buffer);

    //发布turtle1和turtle2的坐标（使用doPose函数）
    ros::Subscriber sub1 = nh.subscribe<turtlesim::Pose>("turtle1/pose", 1000, boost::bind(doPose, _1, "turtle1"));
    ros::Subscriber sub2 = nh.subscribe<turtlesim::Pose>("turtle2/pose", 1000, boost::bind(doPose, _1, "turtle2"));

    //ros::Duration(2.0).sleep();  //可以加个延迟，但是还是会有WARN（从3个变成1个）

    ros::Rate rate(10);
    while (ros::ok()) {
        try {
            // 获取 turtle1 相对 turtle2 的坐标信息
            geometry_msgs::TransformStamped tfs = buffer.lookupTransform("turtle2", "turtle1", ros::Time(0));

            // 根据坐标信息生成速度信息
            geometry_msgs::Twist twist;
            twist.linear.x = 0.5 * sqrt(pow(tfs.transform.translation.x, 2) + pow(tfs.transform.translation.y, 2));
            twist.angular.z = 4 * atan2(tfs.transform.translation.y, tfs.transform.translation.x);
            pub.publish(twist);
        } catch (const tf2::TransformException& e) {
            ROS_WARN("错误提示:%s", e.what());
        }

        rate.sleep();
        ros::spinOnce();
    }

    return 0;
}
```
![在这里插入图片描述](./assets/0447e9ddc6e945c1be82cc9bf9acbfd6.png)


切换到工作空间下编译工作空间：
```powershell
cd ~/catkin_ws
catkin_make
```

使用方法：

分别在4个终端输入以下指令：

```bash
roscore
rosrun turtlesim turtlesim_node
rosservice call /kill "turtle1"
rosrun ch10_tutorials1 turtle4
```
**rosservice call /kill "turtle1"是因为代码里没有写kill小乌龟代码，手动在终端里去掉，不然rosrun turtlesim turtlesim_node会自动生成一只turtle1乌龟，导致重名报错**
如果有佬能修改一下更好（

此时窗口里会出现2个小海龟，打开另一个终端输入以下指令控制其中一只运动

```bash
rosrun turtlesim turtle_teleop_key
```
即可看到一个乌龟跟随另一个了
**我的代码里写的是跟随的(pose->x) + 1.0;即一直跟随在相对位置右上角）**

![在这里插入图片描述](./assets/4bd8e6e442f5403395694622410916e5.png)
