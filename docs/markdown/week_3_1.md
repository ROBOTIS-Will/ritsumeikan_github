---
layout: splash
lang: en
ref: ritsumeikan3-1
permalink: /docs/ritsumeikan/week3-1/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan3-1"
---

# [Class 5] Manipulation

OpenMANIPULATOR-X는 ROS를 지원하는 ROBOTIS의 오픈소스 로봇팔입니다. 다이나믹셀과 3D 프린터를 이용해 제작된 부품으로 조립이 가능하기 때문에 만들기 쉽고 가격이 저렴한 장점이 있습니다.
특히 OpenMANIPULATOR-X는 TurtleBot3 Waffle이나 Waffle Pi 버전과 호환이 되도록 설계되었으며, 여기에서는 TurtleBot3 Waffle Pi에 조립된 매니퓰레이터를 사용하는 방법에 대해 알아봅니다. 

## Software 설정
[Remote PC] TurtleBot3에 조립된 OpenMANIPULATOR-X를 사용하기 위해 패키지를 다운로드하고 빌드합니다.

```bash
$ cd ~/catkin_ws/src/
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_manipulation.git
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_manipulation_simulations.git
$ cd ~/catkin_ws && catkin_make
```

## Hardware 설정
TurtleBot3 Waffle Pi의 LDS 센서는 로봇의 정 중앙에 위치하고 있습니다.
OpenMANIPULATOR-X를 부착하기 위해서는 LDS 센서의 위치를 아래 그림의 빨간 박스쪽으로 이동시키고, 노란 박스에 OpenMANIPULATOR-X의 첫번째 관절을 부착해야 합니다.
정확한 위치에 부착하지 않으면 로봇의 구성을 설명하는 URDF에 정의된 센서와 로봇 팔의 위치가 실제 로봇의 위치와 달라져 의도하지 않은 동작이 실행되거나 로봇 팔이 의도하지 않은 위치로 이동해 충돌이 생길 수 있습니다.

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/assemble_points.png)

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/assemble.png)

## OpenCR 설정
OpenMANIPULATOR-X가 TurtleBot3 Waffle Pi의 OpenCR에 연결되면, OpenCR이 연결된 모든 다이나믹셀을 제어할 수 있도록 펌웨어를 변경해 주어야 합니다. 여러분의 TurtleBot3 Waffle Pi에는 이미 이와 같은 펌웨어가 업로드 되어있습니다.

[TurtleBot3 SBC] OpenCR 펌웨어를 TurtleBot3의 Raspberry Pi에서 업로드하려면 아래와 같이 명령어를 입력합니다.
```bash
$ export OPENCR_PORT=/dev/ttyACM0
$ export OPENCR_MODEL=om_with_tb3
$ rm -rf ./opencr_update.tar.bz2
$ wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2 && tar -xvf opencr_update.tar.bz2 && cd ./opencr_update && ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr && cd ..
```
명령어를 입력하고 잠시 기다리면 새로운 펌웨어가 OpenCR에 업로드 되고, 성공적으로 업로드되면 터미널 창의 마지막에 `jump_to_fw`라는 문구가 표시됩니다.

{% capture danger01 %}
OpenCR의 펌웨어를 성공적으로 업데이트하면 OpenCR이 재부팅되며 OpenMANIPULATOR-X가 기본위치로 움직이게 됩니다. 따라서, OpenMANIPULATOR-X의 전선이 꼬이거나 기본위치로 움직이는 동안 본체나 다른 물체와 부딪히지 않도록 아래와 같은 자세를 만들어준 뒤 펌웨어를 업데이트 하시기 바랍니다.

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/open_manipulator_gazebo_1.png)
{% endcapture %}
<div class="notice--danger">{{ danger01 | markdownify }}</div>

## Bringup
1. [Remote PC] ROS를 구동하기 위한 roscore를 사용자의 PC에서 구동시켜줍니다.
```bash
$ roscore
```

2. [TurtleBot3 SBC] `.bashrc` 파일에 TURTLEBOT3_MODEL에 대한 정의를 해두지 않았다면, 아래 명령어로 사용중인 TurtleBot3 모델을 정의해주어야 합니다. `waffle_pi` 이외에도 제품과 하드웨어의 구성에 따라 `burger`, `waffle`을 사용할 수 있습니다.
```bash
$ export TURTLEBOT3_MODEL=waffle_pi
```

3. [TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```
{% capture capture01 %}
**roslaunch turtlebot3_bringup turtlebot3_robot.launch**
1. turtlebot3_core.launch
    - subscribe : cmd_vel
    - publish : joint_states, odom

2. turtlebot3_lidar.launch
    - publish : scan

OpenCR의 펌웨어가 변경되었기 때문에 turtlebot3_robot.launch 파일을 실행시키면 Week1에서 설명한 turtlebot3_robot.launch를 실행시켰을 때 publish되는 토픽들 이외에도 joint_trajectory_point, gripper_position 두 개의 토픽을 subscribe합니다. joint_trajectory_point는 OpenMANIPULATOR의 각 관절의 위치값을 전달하며 OpenCR을 통해 OpenMANIPULATOR를 구성하는 DYNAMIXEL 액츄에이터에 전달됩니다. gripper_position은 OpenMANIPULATOR 그리퍼의 위치값이며 joint_trajectory_point와 마찬가지로 OpenCR을 통해 그리퍼를 구성하는 DYNAMIXEL에 전달되어 그리퍼를 제어하게 됩니다. rqt의 Message Publisher를 사용하여 위치값을 전송하는 방법으로도 간단히 제어할 수 있습니다.
{% endcapture %}
<div class="notice--success">{{ capture01 | markdownify }}</div>

4. [TurtleBot3 SBC] TurtleBot3 Waffle Pi에 연결되어 있는 Raspberry Pi 카메라를 동작시키려면 아래의 명령어를 입력해서 카메라 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_bringup turtlebot3_rpicamera.launch
```

5. Rviz 실행해서 TB3+OMX 3D 확인 및 RPI 카메라 영상 확인

6. [Remote PC] 본체의 구동을 위해 새로운 터미널 창을 열고 turtlebot3_teleop_key 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

# Gazebo 시뮬레이터에서 MoveIt으로 OpenMANIPULATOR 제어하기

## Gazebo 시뮬레이터 실행

```bash
$ roslaunch turtlebot3_manipulation_gazebo turtlebot3_manipulation.launch
```
{% capture capture02 %}
**roslaunch turtlebot3_manipulation_gazebo turtlebot3_manipulation.launch**
Gazebo상의 로봇이 로드가 되며 그와 함께 로봇과 통신하는 두가지 로봇 컨트롤러 arm_controller, gripper_controller가 실행됩니다. 방식은 실제 로봇을 사용할 때와 동일합니다. 아래의 코드들을 실행시켜 move_group과 통신하여 로봇을 제어하게 됩니다.
{% endcapture %}
<div class="notice--success">{{ capture02 | markdownify }}</div>

![](/assets/images/ritsumeikan/tb3_omx_gazebo.png)

## move_group 노드 실행

MoveIt과 연동하기 위해서는 move_group 노드를 실행시켜 주어야 합니다. Gazebo 시뮬레이터에서 [▶] 실행 버튼을 눌러 시뮬레이션을 시작했다면, 다음 명령어를 입력한 후 아래 그림과 같이 "You can start planning now!" 메세지가 표시됩니다.

```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```
{% capture capture03 %}
**roslaunch turtlebot3_manipulation_moveit_config move_group.launch**
move_group.launch을 실행하면 move_group노드가 실행됩니다. move_group노드는 유저인터페이스를 통해 명령을 받아와 로봇 컨트롤러에게 action형식으로 전달합니다. 
{% endcapture %}
<div class="notice--success">{{ capture03 | markdownify }}</div>

![](/assets/images/ritsumeikan/tb3_omx_move_controller.png)

## RViz 실행
MoveIt 환경이 설정된 `moveit.rviz` 파일을 읽어들여 RViz 상에서 MoveIt을 사용 가능하도록 합니다.
GUI에서 Interactive Marker를 활용한 로봇 팔을 제어할 수 있으며, 목표 위치로의 동작을 시뮬레이션 할 수 있기 때문에 충돌 등에 대비할 수 있습니다.

```bash
$ roslaunch turtlebot3_manipulation_moveit_config moveit_rviz.launch
```
{% capture capture04 %}
**roslaunch turtlebot3_manipulation_moveit_config moveit_rviz.launch**
MoveIt이 활성화된 RViz가 실행됩니다. Motion Planning plugin이 실행되어 기존에 moveit_setup_assistant를 통해 미리 저장된 모션 또는 interactive marker을 통해 설정한 모션을 move_group에게 전달할 수 있습니다. 목표 위치를 설정한 후 Plan and Execute버튼을 누르면 로봇이 움직이게 됩니다.
{% endcapture %}
<div class="notice--success">{{ capture04 | markdownify }}</div>

## 로보티즈 GUI 컨트롤러 실행
RViz를 사용하지 않고 Gazebo와 연결하여 로봇 팔을 제어해보려는 경우 로보티즈 GUI는 OpenMANIPULATOR의 첫번째 DYNAMIXEL을 기준으로 그리퍼의 유효한 파지 위치(그리퍼 사이의 빨간 육면체)를 레퍼런스로 하는 Task Space Control이나 각 조인트 관절의 각도를 레퍼런스로 하는 Joint Space Control을 지원합니다.
필요에 따라 편리한 제어 방법을 사용할 수 있습니다.

```bash
$ roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch
```
{% capture capture05 %}
**roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch**
유저인터페이스로 c++ move_group_interface를 사용한 qt gui가 실행됩니다. 인터페이스를 통해 받아온 현재 조인트 위치 및 end-effector의 위치가 gui상에 표시됩니다. Send버튼을 클릭하면 설정된 위치값을 인터페이스를 통해 move_group에게 전달하여 컨트롤러에 전달, 로봇이 움직이게 됩니다.
{% endcapture %}
<div class="notice--success">{{ capture05 | markdownify }}</div>

## MoveIt!으로 OpenMANIPULATOR 제어하기

MoveIt의 move_group이라는 노드는 여러 컴포넌트들을 받아와 연산된 궤적을 ROS action 형식으로 로봇 컨트롤러에게 제공하는 적분기(intergrator, 積分器)로서의 역할을 합니다. 유저는 move_group노드에 moveit이 제공하는 세가지 인터페이스 (C++, Python, Rviz GUI) 를 통해 접근을 할 수 있습니다. 유저에게 이 인터페이스들을 통해 명령을 받으면 move_group노드는 moveit config 정보 (조인트제한, 기구학, 충돌 탐지) 및 로봇의 상태정보를 토대로 궤적을 생성하여 로봇 컨트롤러에 제공합니다.

![](/assets/images/ritsumeikan/move_group.png)

## Remote PC에서 Bringup 실행하기

기본적인 TurtleBot3의 플랫폼과는 달리 OpenMANIPULATOR를 제어할 수 있는 서비스 서버가 필요하기 때문에 여기의 Bringup에서는 기존에 PC에서 실행되고 있었던 turtlebot3_remote.launch를 종료하고 아래와 같이 Manipulation에 특화된 launch 파일을 실행합니다.

```bash
$ roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch
```
{% capture capture02 %}
**roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch**
1. **turtlebot3_manipulation_bringup 노드**
  - turtlebot3_manipulation_bringup.launch를 실행시키면 arm_controller와 gripper_controller 두 개의 컨트롤러가 실행이 됩니다. move_group과 통신하는 action server 컨트롤러의 역할로써 각각 move_group을 통해 팔과 그리퍼 관절의 목표궤적을 읽어들여 순서대로 publish합니다. publish된 토픽들은 OpenCR을 통해 로봇의 관절에 조립된 DYNAMIXEL에 전달되어 OpenMANIPULATOR를 움직이게 됩니다.

2. **turtlebot3_core.launch**
  - subscribe : cmd_vel
  - publish : joint_states, odom

3. **turtlebot3_lidar.launch**
  - publish : scan

turtlebot3_core.launchとturtlebot3_lidar.launchファイルが実行され、TurtleBot3の状態をチェックするノード(node)であるturtlebot3_diagnosticsが生成され、TurtleBot3の各種センサやハードウェアの状態についての情報をpublishします。turtlebot3_core.launchファイルでは、OpenCRと通信してjoint_states、odomをpublishし、cmd_velをsubscribeするノードが生成されます。turtlebot3_lidar.launchファイルでは、LIDARを作動させ、センサーから得られたscanデータをpublishするノード(node)が生成されます。
{% endcapture %}
<div class="notice--success">{{ capture02 | markdownify }}</div>


```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```
<div class="notice--success">{{ capture03 | markdownify }}</div>

```bash
$ roslaunch turtlebot3_manipulation_moveit_config moveit_rviz.launch
```
<div class="notice--success">{{ capture04 | markdownify }}</div>


![](/assets/images/ritsumeikan/tb3_omx_rviz.png)

```bash
$ roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch
```
<div class="notice--success">{{ capture05 | markdownify }}</div>

![](/assets/images/ritsumeikan/tb3_omx_gui_controller.png)






























## SLAM
OpenMANIPULATOR-X가 장착된 TurtleBot3의 SLAM은 앞에서 학습했던 SLAM과 조금 차이가 있습니다.
로봇 팔이 LDS 센서의 일정 부분을 가로막고 있기 때문에 SLAM에 사용되는 LDS센서의 범위를 제한해주어야 원활한 지도의 작성이 가능해집니다.
아래와 같이 `scan_data_filter.yaml` 파일에서 LDS 센서의 설정된 각도 범위 데이터를 필터링함으로써 유효하지 않은 각도의 값을 사용하지 않고 지도를 그릴 수 있습니다.

```
scan_filter_chain:

- name: Remove 120 to 240 degree
  type: laser_filters/LaserScanAngularBoundsFilterInPlace
  params:
    lower_angle: 2.0944
    upper_angle: 4.18879
```

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/open_manipulator_slam.png)

### TurtleBot3 Bringup
[TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.

```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```
<div class="notice--success">{{ capture01 | markdownify }}</div>

### Remote PC에서 Bringup 실행하기
[Remote PC] 기본적인 TurtleBot3의 플랫폼과는 달리 OpenMANIPULATOR를 제어할 수 있는 서비스 서버가 필요하기 때문에 여기의 Bringup에서는 기존에 PC에서 실행되고 있었던 turtlebot3_remote.launch를 종료하고 아래와 같이 Manipulation에 특화된 launch 파일을 실행합니다.

```bash
$ roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch
```
<div class="notice--success">{{ capture02 | markdownify }}</div>

### SLAM 노드 실행하기
[Remote PC] 여기에서는 Gmapping을 활용한 SLAM을 실행합니다.

```bash
$ roslaunch turtlebot3_slam turtlebot3_manipulation_slam.launch
```
{% capture capture06 %}
**roslaunch turtlebot3_slam turtlebot3_manipulation_slam.launch**
1. **urdf**：Unified Robot Description Formatの略で、ロボットの構成と接続形態を表すXML形式のファイルです。
2. **robot_state_publisher** : robot_state_publisherでは、ロボットの各関節の情報を受信し、得られた関節についての情報をurdfを参考にtfの形式でpublishします。
  - subscribe : joint_states 
  - publish : tf
3. **laser_filter 노드** : LDS센서의 유효하지 않은 값의 영역을 필터링하는 노드를 실행합니다. 여기에서는 OpenMANIPULATOR가 설치되어 있는 후방부에 대한 각도를 무시합니다.
4. **turtlebot3_gmapping.launch** : Gmapping을 이용한 SLAM을 실행하기 위해 필요한 파라미터 정보들이 파라미터 서버에 로딩됩니다. 이 설정을 이용해 gmapping을 설정합니다.
5. **turtlebot3_gmapping.rviz** : Gmapping을 적용한 SLAM을 RViz 화면에 표시하기 위해 필요한 RViz의 기본 설정을 적용하여 RViz를 실행시킵니다.

turtlebot3_manipulation_slam.launchファイルを実行すると、ロボットのurdfを定義された位置から読み込みます。また、joint_statesとurdfを利用して、tfをpublishするrobot_state_publisherノードを生成します。  
{% endcapture %}
<div class="notice--success">{{ capture06 | markdownify }}</div>

### turtlebot3_teleop_key 노드 실행하기
[Remote PC] 작성되지 않은 지도의 위치로 로봇을 이동시켜 지도를 완성합니다.
```bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

지도가 완성되고 나면 map_saver 노드를 실행시켜 지도파일을 저장합니다.
```bash
$ rosrun map_server map_saver -f ~/map
```

## Navigation
OpenMANIPULATOR-X를 장착한 TurtleBot3는 Navigation을 실행할 때에도 LDS 센서의 범위를 지정해 주어야 오류 없이 Navigation을 실행할 수 있습니다.

```bash
Navigation 실행 명령어
```

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/open_manipulator_navigation.png)

## MoveIt!
TurtleBot3의 본체에 장착된 로봇 팔을 움직이기 위해 MoveIt!이라는 플러그인을 사용할 수 있습니다.
