---
layout: splash
lang: en
ref: ritsumeikan2-2
permalink: /docs/ritsumeikan/week2-2/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan2-2"
---

# [Class 4] Self navigation by LRF(Navigation)
ナビゲーションは、特定環境である位置で指定された対象へロボットを移動させるものです。そのために与えられた環境にある障害物や壁などのジオメトレック情報が含まれたマップが必要です。以前のスラム部分で説明した通り、マップはセンサーが獲得した距離情報やロボットのポーズ情報で作成されました。 
ナビゲーションを使用すると、マップ、ロボットインコーダー、IMUセンサーおよび距離センサーを使用してロボットが現在ポーズでマップの指定されたゴールポールに移動することができます。

성공적인 내비게이션을 위해서는 costmap의 계산이 필요합니다. costmap은 앞서 언급된 로봇의 위치와 방향을 표현하는 pose 정보와 센서값, 장애물 정보, SLAM으로 얻어진 지도를 기반으로 계산될 수 있습니다. 계산된 결과는 로봇이 충돌하는 영역, 충돌 가능성이 있는 영역, 자유롭게 이동이 가능한 영역 등으로 표현되며, 이러한 계산결과를 토대로 로봇의 안전한 이동 경로를 생성하는데 도움을 줍니다. costmap은 두 종류로 나뉠 수 있는데 로봇의 시작위치에서부터 목적지까지의 경로 계획을 수립하는데 사용되는 global costmap과 로봇의 주변부에서 장애물 등을 회피하기 위한 안전한 경로를 계산하는 local costmap이 있습니다. ROS에서 이러한 costmap은 값으로 표현될 수 있는데, 0은 로봇이 자유롭게 이동할 수 있는 영역이며, 255에 가까워질수록 로봇과 충돌이 발생할 수 있는 영역으로 구분됩니다.

  - 0 : 자유영역
  - 1 ~ 127 : 낮은 충돌 가능성이 있는 영역
  - 128 ~ 252 : 높은 충돌 가능성이 있는 영역
  - 253 ~ 254 : 충돌 영역
  - 255 : 로봇이 이동할 수 없는 영역

![](http://wiki.ros.org/costmap_2d?action=AttachFile&do=get&target=costmapspec.png)

> [http://wiki.ros.org/costmap_2d#Inflation](http://wiki.ros.org/costmap_2d#Inflation)

## ナビゲーションノードの実行 
1. [遠隔PC] ロスコアを実行してください。 
  ```bash
  $ roscore
  ```
2. [タートルボット] タートルボット３の応用プログラムを始めるための基本パッケージを持ってきてください。 
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

turtlebot3_robot.launchファイルを実行すると、turtlebot3_core.launchとturtlebot3_lidar.launchファイルが実行され、TurtleBot3の状態をチェックするノード（node）であるturtlebot3_diagnosticsが生成され、TurtleBot3の各種センサやハードウェアの状態についての情報をpublishします。turtlebot3_core.launchファイルでは、OpenCRと通信してjoint_states、odomをpublishし、cmd_velをsubscribeするノードが生成されます。turtlebot3_lidar.launchファイルでは、LIDARを作動させ、センサーから得られたscanデータをpublishするノード（node）が生成されます。
{% endcapture %}
<div class="notice--success">{{ capture01 | markdownify }}</div>

3. [遠隔PC] 探索ファイルを実行してください。  
  当コマンドを行う前にタートルボット３のモデル名を指定しなければなりません。$ {TB3_MODEL}は、ハンバーガー、ワッフル、ワッフルパイ(waffle_pi)で使用するモデル名です。エキスポートの設定を永久設定するためには、タートルボット３_MODELのエキスポートページを参照してください。 
  ```bash
  $ export TURTLEBOT3_MODEL=${TB3_MODEL}
  $ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
  ```

{% capture capture02 %}
**roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml**
1. roslaunch turtlebot3_bringup turtlebot3_remote.launch
  - urdf : Unified Robot Description Format의 약자로써 로봇의 구성과 연결형태를 표현하는 XML 형식의 파일입니다.
  - robot_state_publisher : robot_state_publisher에서는 로봇의 각 관절에 대한 정보를 수신하고 얻어진 관절에 대한 정보를 urdf를 참고해서 tf의 형식으로 publish 합니다.
    - subscribe : joint_states 
    - publish : tf
  - turtlebot3_remote.launch 파일을 실행시키면 로봇의 urdf를 정의된 위치에서 불러옵니다. 또한, joint_states와 urdf를 이용하여 tf를 publish하는 robot_state_publisher 노드를 생성합니다. turtlebot3_slam.launch 파일 내부에 turtlebot3_remote.launch를 포함하기 때문에 turtlebot3_slam.launch가 실행되면 자동적으로 turtlebot3_remote.launch가 가장 먼저 실행됩니다.

2. map_server 노드
  - publish : map_metadata, map
  - map_server 노드는 디스크에 저장되어있는 지도를 불러오는 역할을 합니다. 터미널에 입력된 명령어에서 map_file 파라미터는 지도의 정보가 저장된 파일의 위치를 전달합니다.

3. amcl.launch
  - publish : tf, amcl_pose, particlecloud
  - subscribe : scan, tf, initialpose, map
  - 지도와 센서의 scan값, 로봇의 initialpose와 tf를 읽어들인 다음 particle filter를 사용하여 지도상에서 로봇의 위치를 예측합니다.

4. move_base
  - subscribe : goal, cancel
  - publish : feedback, status, result, cmd_vel
  - move_base 패키지의 move_base 노드는 로봇의 Navigation stack에 접근하는 ROS 인터페이스를 제공합니다. move_base 노드는 global planner와 local planner를 연결해서 로봇을 목적지까지 움직이도록 하며, 이때 각각의 planner에 맞는 각각의 costmap 역시 보관합니다. 로봇의 목적지(goal)를 Action 형태의 토픽으로 수신받으면 현재 위치(feedback)와 상태(status), 이동 결과(result)를 업데이트 하기 위해 마찬가지로 Action 형태의 토픽을 사용합니다. 또한, 현재 상태에 맞춰 로봇을 움직이기 위한 cmd_vel 토픽이 지속적으로 publish 됩니다.

5. rviz
  - subscribe : tf, odom, map, scan
  - rviz의 설정파일을 적용한 rviz가 실행되어 tf, scan, map을 subscribe하여 데이터를 시각화합니다.
{% endcapture %}
<div class="notice--success">{{ capture02 | markdownify }}</div>

  上記の命令を実行すると、ビジュアル化ツールRVizが実行されます。RVizを別途実行するためには、下記の命令のコマンドをを使用してください。 
  ```bash
  $ rviz -d `rospack find turtlebot3_navigation`/rviz/turtlebot3_navigation.rviz
  ```

## 初期ポーズの推定 

Navigation을 할 때 가장 중요한 것은 로봇의 정확한 현재 위치를 지도상에 표현하는 것 입니다. 로봇의 엔코더와 IMU, LDS등 각종 센서로부터 얻어진 정보를 토대로 TurtleBot3의 위치를 예측하는 데에는 확률을 기반으로 하는 AMCL (Adaptive Monte Carlo Localization)이라는 particle filter가 사용됩니다. AMCL은 센서의 정보를 기반으로 로봇이 존재할 가능성이 있는 위치를 예측하는데 알고리즘의 파라미터 값에 설정된 이동량만큼 로봇이 이동할 때마다 이러한 예측된 위치값을 갱신하며 갱신이 반복될 때마다 예측된 위치값들의 오차가 줄어들게 됩니다.

[遠隔PC] まず、ロボットの初期ポーズを推定してください。RVizメニューで2D Pose Estimateを押すと、大きな緑の矢印が表示されます。実際のロボットがそれを与えられたマップにあるポーズで移し、マウスの左ボタンを押した状態で緑の矢印をロボットの正面が向かう方向へドラッグします。下記の指針に従ってください。 

  1. 2Dポーズの測定ボタンをクリックします。 
  2. タートルボット３があるマップで近い接点をクリックし、カーサーをドラッグしてタートルボット３が向かう方向を表示します。 

その後、turtlebot3_teleop_keyboard　ノードのようなツールを使用してロボットを前後に動かし、周辺環境の情報を収集し、ロボットがマップにある現在位置を探してください。 

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```
これが完了すると、ロボットは緑の矢印で指定された位置と方向を初期ポーズとして使用し、実際の位置と方向を推定します。緑の矢印はタートルボット３の予想位置を表示します。レーザースキャナーはマップに大まかな壁を表示します。図面に数字が誤って表示されていないことが確認できたら、2D Pose Estimateボタンをクリックしてタートルボット３をローカリゼーションします。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/2d_pose_estimate.png)

> 위 그림에서 로봇 주변에 광범위하게 표시된 수많은 작은 초록색 화살표는 로봇의 초기 위치를 지정했을 때, 로봇의 센서와 각종 정보를 통해 AMCL이 계산한 로봇의 현재 위치입니다. 로봇이 이동할 수록 분포되어 있던 현재 위치 예상값들이 로봇쪽으로 가깝게 수렴하는 모습을 볼 수 있습니다.

警告 : 初期ポーズの推定に使用されるturtlebot3_teleop_keyboardノードは使用後に終了しなければなりません。そうしないと、トピックが次のステージの探索ノードで/ cmd_velのトピックと重なるため、ロボットが異常作動します。 

## ナビゲーション目標の送信 
[遠隔PC] すべてが用意されると、探索GUIで移動コマンドをチャレンジします。2Dを押すと、RVizメニューでNav Goalを選択すると、大きな緑の矢印が表示されます。この緑の矢印は、ロボットの目的地が指定できるマーカーです。矢印の後ろはロボットのXとYの位置で、先端はロボットのセタ方向です。ロボットが動く位置でこの矢印をクリックし、下記のような方向を設定するためにはドラッグします。 

1. 2D Nav Goalボタンをクリックしてください。
2. マップで特定地点をクリックして目標位置を設定し、カーサーをタートルボットが終わる方向へドラッグします。 

ロボットはマップをベースに目的地までの障害物が避けられる経路を作ります。その後、ロボットはその径路にそって移動します。その時に突然、障害物が感知されても障害物を避けて目標地点まで移動します。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/2d_nav_goal.png)

目標位置への経路が作成でいない場合は、目標位置の設定に失敗することがあります。目標位置に到達する前にロボットを停止させるためには、タートルボット３の現在位置を目標位置として設定してください。 

## チューニングガイド 
Navigation stack에는 여러 파라미터들이 있으며, 이러한 파라미터의 설정을 통해 서로 다른 형태의 로봇에 최적화된 Navigation을 적용할 수 있습니다. 
여기에서는 중요하거나 자주 사용되는 파라미터에 대한 설명을 합니다. 다양한 로봇과 환경에 따른 더욱 심도있는 Navigation 튜닝은 [Basic Navigation Tuning Guide](http://wiki.ros.org/navigation/Tutorials/Navigation%20Tuning%20Guide)를 참고하시기 바랍니다.
아래의 파라미터는 costmap을 계산하는데 사용되는 파라미터이며, `turtlebot3_navigation/param/costmap_common_param_$(model).yaml` 파일에 저장되어 프로그램을 실행할 때 로드됩니다.

### Costmap 관련 파라미터

#### inflation_radius
지도상의 장애물로부터 설정된 값 만큼의 공간을 띄워 로봇이 이동할 경로를 생성할 때 최소한의 안전거리를 유지하는데 사용됩니다. 이 값은 로봇의 반지름보다 큰 값으로 설정해야 로봇과 장애물의 충돌을 피할 수 있습니다. 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tuning_inflation_radius.png)


#### cost_scaling_factor 
cost_scaling_factor는 아래와 같이 음수로 된 지수값을 이용해서 계수인자를 생성하기 때문에 cost_sacling_factor의 값을 키울수록 계산된 cost값은 작아지게 됩니다.
costmap_2d::INSCRIBED_INFLATED_OBSTACLE의 값은 254로 정의되어 있습니다.

- exp{-1.0 * cost_scaling_factor * (distance_from_obstacle - inscribed_radius)} * (costmap_2d::INSCRIBED_INFLATED_OBSTACLE - 1)

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tuning_cost_scaling_factor.png)


### AMCL 관련 파라미터

#### min_particles, max_particles
AMCL 노드에서 사용하는 particle의 개수를 조절합니다. 입자의 수가 많아지면 정확도가 높아지지만 계산량이 많아지기 때문에 연산속도가 느려집니다.

#### initial_pose_x, initial_pose_y, initial_pose_a
로봇의 초기 위치와 방향을 설정합니다. RViz의 2D Pose Estimate 버튼으로 로봇의 Pose를 수동으로 설정할 수 있습니다.

#### update_min_d, update_min_a
파티클 필터를 업데이트 하는데 필요한 최소 병진 거리(m)와 최소 회전 각도(rad)를 설정할 수 있습니다. 값이 작을수록 업데이트가 자주 이루어지지만, 계산량이 많아지게 됩니다.

### max_vel_x 
로봇의 최대 병진 속도를 정의합니다.
 
### min_vel_x 
로봇의 최소 병진 속도를 정의합니다. 이 값을 음수로 정의하게 되면 로봇이 후진할 수 있는 경로가 생성되며, 0으로 정의하게 되면 로봇은 항상 앞으로 나아가게 됩니다.

### max_trans_vel 
最大並進速度の実際数値。ロボットはこれよりはやく走ることができません。 
 
### min_trans_vel 
最少並進速度の実際数値。ロボットはこれより遅く走ることができません。 
 
### max_rot_vel 
最大回転速度の実際数値。ロボットはこれよりはやく走ることができません。 
 
### min_rot_vel 
最少回転速度の実際数値。ロボットはこれより遅く走ることができません。 
 
### acc_lim_x 
並進加速限界の実際数値。 
 
### acc_lim_theta 
回転加速限界の実際数値。 

### xy_goal_tolerance 
x、 yの距離は、ロボットが目標位置に到達すると、許可されます。 
 
### yaw_goal_tolerance 
ロボットが目標位置に到達すると、ヨー角が許可されます。 
 
### sim_time 
この要素は数秒位内に正方向シミュレーションに設定されます。低すぎると、狭いゾーンを通過するには十分ではありません。高すぎると、速やかに回転できますｌ。下記のイメージでは、イエローラインの長さの差を確認することができます。

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tuning_sim_time.png)