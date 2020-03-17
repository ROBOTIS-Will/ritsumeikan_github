---
layout: splash
lang: en
ref: ritsumeikan3-2
permalink: /docs/ritsumeikan/tb3_omx_temp/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan3-2"
---

# TurtleBot3 Manipulation

## Software 설정
[Remote PC] TurtleBot3에 조립된 OpenMANIPULATOR-X를 사용하기 위해 패키지를 다운로드하고 빌드합니다.

```bash
$ cd ~/catkin_ws/src/
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_manipulation.git
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_manipulation_simulations.git
$ cd ~/catkin_ws && catkin_make
```

## Hardware 설정

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/assemble_points.png)

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/assemble.png)

## OpenCR 설정
OpenMANIPULATOR-X가 TurtleBot3 Waffle Pi의 OpenCR에 연결되면, OpenCR이 연결된 모든 다이나믹셀을 제어할 수 있도록 펌웨어를 변경해 주어야 합니다.

[TurtleBot3 SBC] OpenCR 펌웨어를 TurtleBot3의 Raspberry Pi에서 업로드하려면 아래와 같이 명령어를 입력합니다.
```bash
$ export OPENCR_PORT=/dev/ttyACM0
$ export OPENCR_MODEL=om_with_tb3
$ rm -rf ./opencr_update.tar.bz2
$ wget https://github.com/ROBOTIS-GIT/OpenCR-Binaries/raw/master/turtlebot3/ROS1/latest/opencr_update.tar.bz2 
$ tar -xvf opencr_update.tar.bz2 
$ cd ./opencr_update && ./update.sh $OPENCR_PORT $OPENCR_MODEL.opencr && cd ..
```
명령어를 입력하고 잠시 기다리면 새로운 펌웨어가 OpenCR에 업로드 되고, 성공적으로 업로드되면 터미널 창의 마지막에 `jump_to_fw`라는 문구가 표시됩니다.

{% capture danger01 %}
OpenCR의 펌웨어를 성공적으로 업데이트하면 OpenCR이 재부팅되며 OpenMANIPULATOR-X가 기본위치로 움직이게 됩니다. 따라서, OpenMANIPULATOR-X의 전선이 꼬이거나 기본위치로 움직이는 동안 본체나 다른 물체와 부딪히지 않도록 아래와 같은 자세를 만들어준 뒤 펌웨어를 업데이트 하시기 바랍니다.

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/open_manipulator_gazebo_1.png)
{% endcapture %}
<div class="notice--danger">{{ danger01 | markdownify }}</div>

## TurtleBot3 Bringup

### roscore 실행하기
[Remote PC] ROS 1을 구동하기 위한 roscore를 사용자의 PC에서 구동시켜줍니다.
```bash
$ roscore
```

### TurtleBot3 모델 정의하기
[TurtleBot3 SBC] `.bashrc` 파일에 TURTLEBOT3_MODEL에 대한 정의를 해두지 않았다면, 아래 명령어로 사용중인 TurtleBot3 모델을 정의해주어야 합니다. `waffle_pi` 이외에도 제품과 하드웨어의 구성에 따라 `burger`, `waffle`을 사용할 수 있습니다.
```bash
$ export TURTLEBOT3_MODEL=waffle_pi
```

### Bringup 실행하기
[TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

# Gazebo 시뮬레이터에서 TurtleBot3 OpenMANIPULATOR 제어하기

## Gazebo 시뮬레이터 실행
[Remote PC] 아래 명령어를 새 터미널창에 입력해서 OpenMANIPULATOR가 적용된 TurtleBot3의 모델을 Gazebo환경으로 불러옵니다.
```bash
$ roslaunch turtlebot3_manipulation_gazebo turtlebot3_manipulation.launch
```

![](/assets/images/ritsumeikan/tb3_omx_gazebo.png)

## move_group 노드 실행

[Remote PC] MoveIt과 연동하기 위해서는 move_group 노드를 실행시켜 주어야 합니다. Gazebo 시뮬레이터에서 [▶] 실행 버튼을 눌러 시뮬레이션을 시작했다면, 다음 명령어를 입력한 후 아래 그림과 같이 "**You can start planning now!**" 메세지가 표시됩니다.

```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```

## RViz 실행
[Remote PC] MoveIt 환경이 설정된 `moveit.rviz` 파일을 읽어들여 RViz 상에서 MoveIt을 사용 가능하도록 합니다.
GUI에서 Interactive Marker를 활용한 로봇 팔을 제어할 수 있으며, 목표 위치로의 동작을 시뮬레이션 할 수 있기 때문에 충돌 등에 대비할 수 있습니다.

```bash
$ roslaunch turtlebot3_manipulation_moveit_config moveit_rviz.launch
```

![](/assets/images/ritsumeikan/tb3_omx_rviz.png)

## 로보티즈 GUI 컨트롤러 실행
[Remote PC] RViz를 사용하지 않고 Gazebo와 연결하여 로봇 팔을 제어해보려는 경우 로보티즈 GUI는 OpenMANIPULATOR의 첫번째 DYNAMIXEL을 기준으로 그리퍼의 유효한 파지 위치(그리퍼 사이의 빨간색 육면체)를 레퍼런스로 하는 Task Space Control이나 각 조인트 관절의 각도를 레퍼런스로 하는 Joint Space Control을 지원합니다.
필요에 따라 편리한 제어 방법을 사용할 수 있습니다.

```bash
$ roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch
```

![](/assets/images/ritsumeikan/tb3_omx_gui_controller.png)

# 실제 OpenMANIPULATOR 제어하기

## roscore 실행하기
[Remote PC] ROS 1을 구동하기 위한 roscore를 사용자의 PC에서 구동시켜줍니다.
```bash
$ roscore
```

## Bringup 실행하기
[TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

[Remote PC] OpenMANIPULATOR를 제어할 수 있는 turtlebot3_manipulation_bringup.launch을 실행합니다.

```bash
$ roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch
```

## move_group 실행하기
[Remote PC] MoveIt과 연동되는 사용자 인터페이스인 move_group 노드를 실행합니다.
```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```

## RViz 실행하기
[Remote PC] 각종 데이터의 시각화와 Interactive Marker를 활용한 OpenMANIPULATOR의 제어를 위해 RViz를 실행합니다.
```bash
$ roslaunch turtlebot3_manipulation_moveit_config moveit_rviz.launch
```

## ROBOTIS GUI 실행하기
[Remote PC] RViz와는 별개로 필요한 경우 ROBOTIS GUI를 통해 OpenMANIPULATOR를 제어할 수도 있습니다.
```bash
$ roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch
```

# TurtleBot3 OpenMANIPULATOR를 사용하여  SLAM 실행하기

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/manipulation/open_manipulator_slam.png)

## roscore 실행하기
[Remote PC] ROS 1을 구동하기 위한 roscore를 사용자의 PC에서 구동시켜줍니다.
```bash
$ roscore
```

## Bringup 실행하기
[TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.

```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

SLAM을 사용하여 지도를 작성할 때에는 Manipulation을 사용하지 않기 때문에 아래의 OpenMANIPULATOR를 제어하는 컨트롤러와 move_group 인터페이스는 실행할 필요가 없습니다.

```bash
$ roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch
```

```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```

## SLAM 노드 실행하기
[Remote PC] 여기에서는 Gmapping을 활용한 SLAM을 실행합니다.

```bash
$ roslaunch turtlebot3_slam turtlebot3_manipulation_slam.launch
```

## turtlebot3_teleop_key 노드 실행하기
[Remote PC] 작성되지 않은 지도의 위치로 로봇을 이동시켜 지도를 완성합니다.
```bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

[Remote PC] 지도가 완성되고 나면 map_saver 노드를 실행시켜 지도파일을 저장합니다.
```bash
$ rosrun map_server map_saver -f ~/map
```

# TurtleBot3 OpenMANIPULATOR를 사용하여  Navigation 실행하기

## roscore 실행하기
[Remote PC] ROS 1을 구동하기 위한 roscore를 사용자의 PC에서 구동시켜줍니다.
```bash
$ roscore
```

## Bringup 실행하기
[TurtleBot3 SBC] 아래 명령어로 rosserial과 LDS센서를 동작시키는 노드를 실행합니다.

```bash
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

## Navigation 실행하기
[Remote PC] 아래의 명령어를 실행하면 Navigation을 실행하기 위해 필요한 여러가지 파라미터와 지도, GUI 환경을 만들기 위한 URDF와 RViz 환경설정 등을 불러들입니다. 

```bash
$ roslaunch turtlebot3_manipulation_navigation navigation.launch
```

![](/assets/images/ritsumeikan/tb3_omx_nav.png)

## OpenMANIPULATOR 제어하기
Navigation을 실행할 때 OpenMANIPULATOR를 제어하는 노드를 생성하면, Navigation과 함께 로봇 팔을 제어할 수 있게 됩니다.
로봇이 움직이는 동안 OpenMANIPULATOR를 움직이는 경우 진동이나 무게중심의 이동으로 인해 로봇과 로봇팔의 동작이 불안정할 수 있습니다. 로봇 팔은 로봇이 움직이지 않는 상태에서 동작시키는 것을 권장합니다.

### turtlebot3_manipulation_bringup 노드 실행
[Remote PC] OpenMANIPULATOR만을 제어할 때와 마찬가지로 arm_controller와 gripper_controller를 실행합니다.
```bash
$ roslaunch turtlebot3_manipulation_bringup turtlebot3_manipulation_bringup.launch
```

### move_group 노드 실행
move_group 노드를 실행한 이후 MoveIt을 사용해서 OpenMANIPULATOR를 제어하거나 ROBOTIS GUI를 이용해 제어할 수 있습니다. 여기에서는 ROBOTIS GUI를 실행하는 방법을 기술합니다. 두 가지 방법 중 적절한 인터페이스를 사용하시기 바랍니다.

```bash
$ roslaunch turtlebot3_manipulation_moveit_config move_group.launch
```

### 로보티즈 GUI 컨트롤러 실행
[Remote PC] 로보티즈 GUI는 OpenMANIPULATOR의 첫번째 DYNAMIXEL을 기준으로 그리퍼의 유효한 파지 위치(그리퍼 사이의 빨간색 육면체)를 레퍼런스로 하는 Task Space Control이나 각 조인트 관절의 각도를 레퍼런스로 하는 Joint Space Control을 지원합니다.
필요에 따라 편리한 제어 방법을 사용할 수 있습니다.

```bash
$ roslaunch turtlebot3_manipulation_gui turtlebot3_manipulation_gui.launch
```
