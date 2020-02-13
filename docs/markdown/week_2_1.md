---
layout: splash
lang: en
ref: ritsumeikan2-1
permalink: /docs/ritsumeikan/week2-1/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan2-1"
---

# [Class 3] LRF(LDS) 센서

TurtleBot3에 사용된 LDS(LASER Distance Sensor)는 2차원 평면의 360도 거리값을 읽을 수 있는 센서로써 SLAM과 Navigation에 필요한 중요한 정보를 제공합니다.  
LDS 센서는 USB 인터페이스인 USB2LDS 보드를 통해 Raspberry Pi와 USB 케이블을 통해 연결되지만, UART 인터페이스를 통해 OpenCR과도 직접 연결될 수 있습니다.

센서의 특성상 직사광선이 강하게 내리쬐는 외부의 환경에서는 사용이 어렵기 때문에 10,000lux 이하 밝기의 실내공간에서 주행하는 로봇에 적합합니다.

TurtleBot3의 LDS 센서는 패키지의 형태로 설치하거나 소스코드를 다운로드 받아서 빌드할 수 있습니다.

## 센서 패키지 설치
```bash
$ sudo apt-get install ros-kinetic-hls-lfcd-lds-driver
```
TurtleBot3의 LDS 센서를 구동하는데 필요한 드라이버 패키지를 설치합니다.

### 센서 연결포트 권한 설정
```bash
$ sudo chmod a+rw /dev/ttyUSB0
```
센서를 USB로 Linux가 설치된 PC와 연결할 경우 USB 포트의 권한을 올바르게 설정해 주어야 할당된 포트에 접근할 수 있습니다.  
위 명령어는 USB0번 포트에 읽기 쓰기 권한을 설정합니다.

### hlds_laser_publisher 노드 실행
```bash
$ roslaunch hls_lfcd_lds_driver hlds_laser.launch
```
{% capture capture00 %}
**roslaunch hls_lfcd_lds_driver hlds_laser.launch**
1. hlds_laser_publisher
    - publish : scan, rpms

hlds_laser.launch 파일의 hlds_laser_publisher 노드가 생성되어 센서의 데이터와 회전속도를 각각 scan과 rpms의 topic으로 publish합니다. sensor_msgs타입의 LaserScan 메세지인 scan에는 LDS 센서가 회전하며 획득한 로봇의 주변 물체와의 거리 데이터가 배열의 형태로 누적되어 저장됩니다.
{% endcapture %}
<div class="notice--success">{{ capture00 | markdownify }}</div>

### RViz와 hlds_laser_publisher 노드 실행
```bash
$ roslaunch hls_lfcd_lds_driver view_hlds_laser.launch
```

{% capture capture01 %}
**roslaunch hls_lfcd_lds_driver view_hlds_laser.launch**
1. hlds_laser_publisher
    - publish : scan, rpms

2. rviz
    - subscribe : scan

view_hlds_laser.launch를 실행하면 hlds_laser.launch 파일과 rviz 노드가 실행됩니다.  
hlds_laser.launch 파일의 hlds_laser_publisher 노드가 생성되어 센서의 데이터와 회전속도를 각각 scan과 rpms의 topic으로 publish합니다. sensor_msgs타입의 LaserScan 메세지인 scan에는 LDS 센서가 회전하며 획득한 로봇의 주변 물체와의 거리 데이터가 배열의 형태로 누적되어 저장됩니다.  
RViz 노드에서는 rviz의 설정파일을 읽어들여 화면을 띄우고 scan 데이터를 subscribe하여 3차원 그래픽으로 시각화합니다.
{% endcapture %}
<div class="notice--success">{{ capture01 | markdownify }}</div>

## 센서 소스코드 다운로드

다운로드된 LDS-01을 지원하는 드라이버는 Windows, Linux, MacOS의 개발환경에서 실행할 수 있습니다.
소프트웨어 요구사항은 아래와 같습니다.
- GCC(Linux와 MacOS 사용시), MinGW(Windows 사용시)
- Boost library(v1.66.0에서 테스트됨)

1. 아래의 GitHub 주소에서 소스코드를 다운로드합니다.
  ```bash
  $ git clone https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver.git
  ```
  또는, 아래의 사이트에서 초록색 `Clone or download` 버튼을 눌러 소스코드를 다운로드 할 수도 있습니다.
  - [https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver](https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver)

2. 개발환경에 필요한 소프트웨어와 라이브러리를 설치합니다.
  - GCC(Linux, MacOS) 또는 MinGW(Windows)
  - Boost library

### 소스코드 빌드하기
아래의 명령어는 Linux 환경에 맞도록 설정된 makefile을 이용하여 소스코드를 빌드합니다. Windows와 MacOS 환경의 경우 makefile을 적절히 수정하시기 바랍니다.
```bash
$ cd hls_lfcd_lds_driver/applications/lds_driver/
$ make
```

### CLI 환경에서 실행하기
소스코드가 성공적으로 빌드되면 아래와 같이 `./lds_driver` 파일을 실행해서 LDS 센서의 값을 확인할 수 있습니다.
```bash
$ ./lds_driver
```
```
r[359]=0.438000,r[358]=0.385000,r[357]=0.379000,...
```

### GUI 환경에서 실행하기
LDS 센서의 값을 시각적으로 확인하기 위해서는 Qt Creator와 Qt Libs를 추가로 설치해야 합니다.
- Qt Creator(v4.5.0에서 테스트됨)
- Qt Libs(v5.10.0에서 테스트됨)

1. 아래 Qt 사이트에서 Open Source 버전을 설치하세요
  - [https://www.qt.io/download](https://www.qt.io/download)

2. Qt Creator를 실행합니다.
3. 소스코드에서 아래 위치의 `lds_polar_graph.pro` 파일을 엽니다.  
  (hls_lfcd_lds_driver/applications/lds_polar_graph/lds_polar_graph.pro)
4. 소스코드에서 센서와 연결된 포트를 확인하고, 다른 포트에 할당되었다면 해당 포트 번호로 변경합니다.
5. `CTRL` + `SHIFT` + `B`를 눌러 소스코드를 빌드합니다.
6. 빌드가 성공적으로 완료되면 `CTRL` + `R`을 눌러 프로그램을 실행합니다.

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/appendix_lds/lds_gui.png)

### Embedded에서 실행하기

![](/assets/images/ritsumeikan/lds_lines.png)

# Making environmental map(SLAM)

SLAM(Simultaneous Localization and Mapping)이란 알려지지 않은 지역을 탐험하며 로봇에 부착된 센서를 통해 취득된 정보를 통해 로봇이 탐험하는 위치에 대한 지도를 작성하는 것을 의미합니다. SLAM은 내비게이션이나 무인 자동차의 자율 주행에 없어서는 안될 중요한 기술입니다.  
외부 정보를 취득하기 위해 사용되는 센서에는 거리를 측정할 수 있는 센서나 주변의 이미지를 얻어올 수 있는 센서들이 있습니다. 적외선 거리 센서(IR), 음파 센서(SONAR), 레이저 센서(LRF) 등이 많이 사용되며, 최근에는 영상을 해석하는 알고리즘의 발전으로 카메라가  많이 사용되기도 합니다.  
로봇의 위치를 예측하기 위해 로봇 바퀴와 연결된 엔코더의 값을 읽어 주행 거리(odometry)를 추측 항법(Dead Reckoning)을 사용해서 계산하게 되는데, 이때 바퀴와 지면간의 마찰 등으로 오차가 발생하게 됩니다. 실제 로봇의 위치와 예측된 로봇의 위치의 오차를 줄이기 위해서 관성 측정 센서(IMU)로부터 얻어진 데이터를 이용해 위치를 보정할 수 있습니다.  
또한, 거리 센서의 값을 이용해 위치 오차를 감소시키는데 사용되는 방법으로는 Kalman filter, Markov Localization, Monte Carlo Localization 등이 있습니다.

スラム(SLAM、Simultaneous Localization and Mapping)は、任意のスペースで現在位置を推定してマップを画く技術です。スラムは以前バージョンのタートルボットで知らされている機能です。この動画はタートルボット３が安価で小型のプラットフォームでどれほど正確にマップがかけるのかを見せます。 

## スラムノードの実行
1. [遠隔PC] ロスコアを実行してください。 
2. [タートルボット] タートルボット３の応用プログラムを始めるための基本パッケージを持ってきてください。 
3. [遠隔 PC] 新しいターミナルをオープンしてスラムファイルを実行してください。  
  当コマンドを行う前にタートルボット３のモデル名を指定しなければなりません。$ {TB3_MODEL}は、バーガー、ワッフル、ワッフルパイ(waffle_pi)で使用するモデル名です。エキスポートの設定を永久設定するためには、タートルボット３_MODELのエキスポートページを参照してください。 

  ```bash
  $ export TURTLEBOT3_MODEL=${TB3_MODEL}
  $ roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
  ```

{% capture capture02 %}
**roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping**
1. roslaunch turtlebot3_bringup turtlebot3_remote.launch
  - urdf：Unified Robot Description Formatの略で、ロボットの構成と接続形態を表すXML形式のファイルです。
  - robot_state_publisher : robot_state_publisherでは、ロボットの各関節の情報を受信し、得られた関節についての情報をurdfを参考にtfの形式でpublishします。
  - subscribe : joint_states 
  - publish : tf

    turtlebot3_remote.launchファイルを実行すると、ロボットのurdfを定義された位置から読み込みます。また、joint_statesとurdfを利用して、tfをpublishするrobot_state_publisherノードを生成します。  
    turtlebot3_slam.launch 파일 내부에 turtlebot3_remote.launch를 포함하기 때문에 turtlebot3_slam.launch가 실행되면 자동적으로 turtlebot3_remote.launch가 가장 먼저 실행됩니다.

2. turtlebot3_gmapping.launch
  - subscribe : scan, tf
  - publish : map, map_metadata

    실행문의 끝에서 slam_methods:=gmapping이라는 옵션을 입력했기 때문에 gmapping의 실행 관련된 파일인 turtlebot3_gmapping.launch가 실행됩니다.  
    이 파일 내부에서는 다시 LDS의 설정이 저장된 turtlebot3_lds_2d.lua를 불러오며 gmapping의 사용을 위해 필요한 각종 파라미터를 정의하고 gmapping 패키지의 slam_gmapping 노드를 실행합니다.  
    slam_gmapping 노드가 생성되면 scan과 tf 토픽을 subscribe하여 맵을 생성하는데 필요한 map_metadata와 map을 publish합니다.

3. rviz
  - subscribe : tf, scan, map

    마지막으로 rviz의 설정파일을 적용한 rviz가 실행되어 tf, scan, map데이터를 subscribe하여 로봇과 센서값, gmapping을 통해 생성된 맵을 시각화합니다.  
    완성된 지도는 두 개의 파일로 나뉘어 저장되는데, pgm은 Portable Gray Map 형식의 이미지 파일이며, yaml는 지도의 해상도 등 각종 설정을 저장하는 파일입니다.
{% endcapture %}
<div class="notice--success">{{ capture02 | markdownify }}</div>

- TurtleBot3様々なスラムメソッドを支援 
  - タートルボット３は様々なスラム方法でGmapping、cartographer、ヘクター、カルトを支援します。slam_methods:= xxxxx オプションを変更して行うことができます。
  - slam_methodsオプションには、`gmapping`, `cartographer`, `hector`, `karto`, `frontier_exploration`が含まれているため、、その一つを選択することができます。
  - 例えば、カルトは下記のように使用できます。
    ```bash
    $ roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=karto
    ```

- スラムパッケージに対するサブパッケージをインストール 
  - cartographerの場合： 
    ```bash
    $ sudo apt-get install ros-kinetic-cartographer ros-kinetic-cartographer-ros ros-kinetic-cartographer-ros-msgs ros-kinetic-cartographer-rviz 
    ```
  - Hector Mappingの場合： 
    ```bash
    $ sudo apt-get install ros-kinetic-hector-mapping
    ```
  - Kartoの場合 : 
    ```bash
    $ sudo apt-get install ros-kinetic-karto 
    ```
  - Frontier Explorationの場合 : 
    ```bash
    $ sudo apt-get install ros-kinetic-frontier-exploration ros-kinetic-navigation-stage
    ```

上記の命令を実行すると、ビジュアル化ツールRVizが実行されます。RVizを別途実行するためには、下記の命令の一つを使用してください。 

```bash
$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_gmapping.rviz
$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_cartographer.rviz
$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_hector.rviz
$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_karto.rviz
$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_frontier_exploration.rviz
```

cartographerバージョン0.3.0でテストしました。Googleで開発したCartographerパッケージは、ROSメロディック で0.3.0バージョンを支援しますが、ROSKineticでは0.2.0バージョンを支援します。ROSKineticでバイナリファイルをダウンロードする代わりに、下記のようにソースコードをダウンロードしてビルドしなければなりません。より詳細なインストール指針は[公式ウィキページ](https://google-cartographer-ros.readthedocs.io/en/latest/#building-installation)を参照にしてください。

```bash
$ sudo apt-get install ninja-build libceres-dev libprotobuf-dev protobuf-compiler libprotoc-dev
$ cd ~/catkin_ws/src
$ git clone https://github.com/googlecartographer/cartographer.git
$ git clone https://github.com/googlecartographer/cartographer_ros.git
$ cd ~/catkin_ws
$ src/cartographer/scripts/install_proto3.sh
$ rm -rf protobuf/
$ rosdep install --from-paths src --ignore-src -r -y --os=ubuntu:xenial
$ catkin_make_isolated --install --use-ninja
```

```bash
$ source ~/catkin_ws/install_isolated/setup.bash
$ roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=cartographer
```

{% capture capture03 %}
**roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=cartographer**
1. roslaunch turtlebot3_bringup turtlebot3_remote.launch
  - urdf：Unified Robot Description Formatの略で、ロボットの構成と接続形態を表すXML形式のファイルです。
  - robot_state_publisher : robot_state_publisherでは、ロボットの各関節の情報を受信し、得られた関節についての情報をurdfを参考にtfの形式でpublishします。
  - subscribe : joint_states 
  - publish : tf

    turtlebot3_remote.launchファイルを実行すると、ロボットのurdfを定義された位置から読み込みます。また、joint_statesとurdfを利用して、tfをpublishするrobot_state_publisherノードを生成します。  
    turtlebot3_slam.launch 파일 내부에 turtlebot3_remote.launch를 포함하기 때문에 turtlebot3_slam.launch가 실행되면 자동적으로 turtlebot3_remote.launch가 가장 먼저 실행됩니다.

2. turtlebot3_cartographer.launch
  - subscribe : scan, imu, odom
  - publish : submap_list, map
  
    실행문의 끝에서 slam_methods:=cartographer라는 옵션을 입력했기 때문에 cartographer의 실행 관련된 파일인 turtlebot3_cartographer.launch가 실행됩니다.  
    이 파일 내부에서는 다시 LDS의 설정이 저장된 turtlebot3_lds_2d.lua를 불러오며 cartographer의 사용을 위해 필요한 각종 파라미터를 정의하고 cartographer_ros 패키지의 cartographer_node 노드를 실행하여 scan, imu, odom 등의 topic을 subscribe하고 submap_list를 publish 합니다.  
    또한, cartographer_ros 패키지의 cartographer_occupancy_grid_node 노드를 실행하여 cartographer_node에서 publish된 submap_list를 subscribe하여 map을 publish합니다.

3. rviz
  - subscribe : tf, scan, map

    마지막으로 rviz의 설정파일을 적용한 rviz가 실행되어 tf, scan, map데이터를 subscribe하여 로봇과 센서값, gmapping을 통해 생성된 맵을 시각화합니다.  
    완성된 지도는 두 개의 파일로 나뉘어 저장되는데, pgm은 Portable Gray Map 형식의 이미지 파일이며, yaml는 지도의 해상도 등 각종 설정을 저장하는 파일입니다.
{% endcapture %}
<div class="notice--success">{{ capture03 | markdownify }}</div>


## 遠隔ノードの運営 
[遠隔 PC] 新しいターミナルをオープンして遠隔ノードを実行してください。次のコマンドを使用すると、使用者がスラム操作をマニュアルで行うようにロボットをコントロールすることができます。変更が速すぎたリ、回転を速くしたりする過激な動きは避けることをおすすめします。ロボットを使用してマップを作成する際、ロボットは測定する環境のあらゆるところをスキャンしなければなりません。クリーンマップを作成するためには、いくつかの経験が必要ですので、スラムを何回か練習して方法を身につけてください。マッピングプロセスは下記のイメージのようです。

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

```
Control Your TurtleBot3!
---------------------------
Moving around:
        w
    a    s    d
        x

w/x : increase/decrease linear velocity
a/d : increase/decrease angular velocity
space key, s : force stop

CTRL-C to quit
```

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/slam_running_for_mapping.png)

## チューニングガイド 
Gmappingには様々な環境に対して性能を変更させるたくさんのパラメータがあります。ROSウィキですべてのパラメータに対する情報を得るか、ROSロボットプログラミングの第１１章を参照にしてください。 
重要なパラメータを設定するチューニングガイドはいくつかのTIPを提供します。パフォーマンスを環境によって変更するためには、TIPが役に立つのはもちろん、事案節約にもなります。 

### maxUrange
このパラメータはライダーセンターの最大使用可能範囲を設定します。 
 
### map_update_interval 
マップをアップデートする期間(秒段位)この値が低いと、マップが頻繁にアップデートされます。しかし、より大きな計算負荷がかかります。環境に基づいてこのパラメータを設定してください。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/tuning_map_update_interval.png)

### minimumScore 
スキャンマッチの結果を考慮するための最小点数。当パラムはポーズ推定値をスキップすることを防止します。これが正しく設定されると、下記の情報を見ることができます。 

```
Average Scan Matching Score=278.965
neff= 100
Registering Scans:Done
update frame 6
update ld=2.95935e-05 ad=0.000302522
Laser Pose= -0.0320253 -5.36882e-06 -3.14142
```
この値が高すぎると、下記の警告を見ることができます。 
```
Scan Matching Failed, using odometry. Likelihood=0
lp:-0.0306155 5.75314e-06 -3.14151
op:-0.0306156 5.90277e-06 -3.14151
```

### linearUpdate
ロボットが翻訳される度にスキャンプロセスが行われます。 
 
### angularUpdate 
ロボットが回転される度にスキャンプロセスが行われます。これをlinearUpdateより小さく設定することをおすすめします。 

## マップの保存 
[遠隔PC] すべての作業が終わりましたので、map _saverノードを実行してマップファイルを作ります。マップはロボットが動く時にロボットの走行距離、tf 情報、センサーのスキャン情報をベースに作成されます。当データーは以前動画のRVizで見ることができます。作成されたマップはmap_saverが実行されるディレクトリに保存されます。ファイル名を指定しないと、マップ情報が入っているmap.pgmおよびmap.yamlファイルに保存されます。 

```bash
$ rosrun map_server map_saver -f ~/map
```

`-f` オプションは、マップファイルが保存されたフォルダーおよびファイル名が表示されます。~ / mapをオプションで使用すると、map.pgmとmap.yamlが使用者のホームフォルダー~ / ($HOME directory :  /home/\<username\>)のマップフォルダーに保存されます。 

## マップ 
ROSコミュニティーで一般的に使用される２次元OGMを使用します。下記のイメージのように以前のSave Mapセクションで得たマップで、ホワイトはロボットが動ける空きゾーン、ブラックはロボットが動けないゾーン、グレーは不明なゾーンです。当マップは探索に使用されます。

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/map.png)

下記のイメージはタートルボット３を使って大きなマップを作成したその結果です。約３５０ｍの走行距離のマップ作成には約１時間かかりました。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/large_map.png)
