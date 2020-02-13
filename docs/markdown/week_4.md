# I. Learning ROS Environment

## I-1. ROS Installation and environment configuration

## I-2. Basic ROS commands


# II. TurtleBot3 Operation

## II-1. Network Setup

## II-2. Bringup

## II-3. Teleoperation


# III. Gazebo Simulator & Switching Sim - Real robot

## III-1. Simulation

## III-2. SimulationとReal Robotのクロス開発

### III-2-A. 目標
20台のタートルボットと40台のsimulationの間に円滑なクロス開発が可能な環境およびネットワーク構成案、交互開発のための実行方法の提示

### III-2-B. ネットワーク構成案
- 仮定
  - すべてのTB3とユーザーPC（Simulation）は、固定IPアドレスを持つ。
    ユーザーPCは固定IPアドレスでない場合でも動作可能だと思われるが、ROSネットワーク設定を容易にするために固定IPの使用を推奨。
  - 各ユーザーは、自分の使用するTB3のIPアドレスを知っている。

1. 第1案  
ルータを複数台使用（5台）、4台のTB3と8人（Simulator含む）が一つのルータを使用する（推奨）

2. 第2案  
高性能ルータに40人と20台のTB3すべてを接続（クライアントの要求）

### III-2-C. ネットワーク設定(第1案)

1. ルータ設定  
    - SSIDおよびルータIP（IP帯域）：ルータごとに設定方法が異なるため、詳細な説明は省略
        - Turtlebot_Server_1 : 10.17.1.1 (10.17.1.x)
        - Turtlebot_Server_2 : 10.17.2.1 (10.17.2.x)
        - Turtlebot_Server_3 : 10.17.3.1 (10.17.3.x)
        - Turtlebot_Server_4 : 10.17.4.1 (10.17.4.x)
        - Turtlebot_Server_5 : 10.17.5.1 (10.17.5.x)

2. Turtlebot3 ネットワーク設定  
    - IPアドレスの例 (Turtlebot_Server_1の場合)
        - [Turtlebot3] IP : 4台
            - 10.17.1.11
            - 10.17.1.12 
            - 10.17.1.13
            - 10.17.1.14
        - [User PC] IP : 8台
            - 10.17.1.21
            - 10.17.1.22
            - 10.17.1.23
            - 10.17.1.24
            - 10.17.1.25
            - 10.17.1.26
            - 10.17.1.27
            - 10.17.1.28
        - 上記のような方式のアドレスで、残り4台のサーバーに接続しているTurtlebot3とユーザーPCのIPアドレスを設定

    - ネットワーク設定
        - [Turtlebot3] (10.17.1.11の場合)
            1. Turtlebotにキーボード、マウス、モニタを接続した後、Raspberry Piを起動
            2. ターミナルウィンドウを開き、/etc/dhcpcd.confファイルのstatic IP部分を変更  
                ```
                $ sudo nano /etc/dhcpcd.conf
                ```
            3. 以下の内容を追加
                ```                
                interface wlan0
                static ip_address=10.17.1.11/24
                static routers=10.17.1.1
                static domain_name_servers=8.8.8.8
                ```
            4. ファイルを保存して再起動
                ```
                $ sudo reboot
                ```
            5. ルータに接続し、IPアドレスを確認  
            ターミナルを開いて以下のように入力
                ```
                $ ifconfig
                ```

        - [User PC] (10.17.1.21の場合)
            1. ネットワークマネージャを開いて以下のように設定

            2. ルータに接続した後、IPアドレスを確認
                ```
                $ ifconfig
                ```

### III-2-D. ネットワーク設定(第2案)
1. ルータ設定
    - SSIDおよびルータIP（IP帯域）：ルータごとに設定方法が異なるため、詳細な説明は省略
        - Turtlebot_Server : 10.17.1.1 (10.17.1.x)
2. Turtlebot3 ネットワーク設定 (便宜上、固定IPを使用)
    - IPアドレスの例 
        - [Turtlebot3] IP : 20台
            - 10.17.1.11〜30
        - [User PC] IP : 40台
            - 10.17.1.101〜140
    - ネットワーク設定
        - [Turtlebot3] (10.17.1.11の場合)
            1. Turtlebotにキーボード、マウス、モニタを接続した後、Raspberry Piを起動
            2. ターミナルウィンドウを開き、`/etc/dhcpcd.conf`ファイルの`static IP`部分を変更
                ```
                $ sudo nano /etc/dhcpcd.conf
                ```
            3. 以下の内容を追加
                ```
                interface wlan0
                static ip_address=10.17.1.11/24
                static routers=10.17.1.1
                static domain_name_servers=8.8.8.8
                ```
            4. ファイルを保存して再起動
                ```
                $ sudo reboot
                ```
            5. ルータに接続し、IPアドレスを確認  
            ターミナルを開いて以下の通り入力
                ```
                $ ifconfig
                ```
        - [User PC] (10.17.1.101の場合)
            1. ネットワークマネージャを開いて以下のように設定し、保存

            2. ルータに接続し、IPアドレスを確認
                ```
                $ ifconfig
                ```

### III-2-E. ROS ネットワーク設定
1. [Turtlebot3]
    - 追加でROSネットワーク設定を行う必要なし

2. [User PC]
    - 参考 : [e-Manual](http://emanual.robotis.com/docs/en/platform/turtlebot3/pc_setup/#network-configuration)
    - ユーザーPCのIP例: 10.17.1.101
    - ユーザーPCのIPを確認（以下のコマンドを入力すると、上で設定したユーザーPCの静的IPが出力される。
        ```
        $ ifconfig
        ```
    - `~/.bashrc` ファイルを修正：以下のコードがある場合は修正し、ない場合は追加
        ```
        export ROS_MASTER_URI=http://10.17.1.101:11311
        export ROS_HOSTNAME=10.17.1.101
        ```

### III-2-F. その他の設定
1. [Turtlebot3]
    - machineタグを利用したlaunchファイルで使用する`env.bash`ファイルの生成 (位置 : /home/pi)
        ```
        $ nano ~/env.bash
        ```

        ```
        #!/bin/bash

        # check if other turtlebot3_core is already running
        is_running=`ps ax | grep turtlebot3_core`
        IFS=' ' read -ra is_runnings <<< "$is_running"
        process_name=${is_runnings[4]}
        if [ ${process_name} == "python" ]
        then
        echo "other turtlebot3_core is already running."
        exit 1
        fi

        #### ROS ####
        source /opt/ros/kinetic/setup.bash
        source ~/catkin_ws/devel/setup.bash

        #### ROS Network ####
        ip_address=`hostname -I`
        ip_address_trim=${ip_address%% * }
        ip_address_no_space="$(echo -e "${ip_address_trim}" | tr -d '[:space:]')"
        export ROS_HOSTNAME=${ip_address_no_space}

        ##### Set TURTLEBOT3 Model ####
        export TURTLEBOT3_MODEL=waffle_pi

        exec "$@"
        ```

    - `env.bash`ファイルに実行権限を追加
        ```
        $ chmod +x ~/env.bash
        ```

2. [User PC]
    - 接続したTurtlebot3のssh keyを生成、以下で作成するscriptを利用
    - keygen script製作：ssh-keyscanで複数のアルゴリズムを利用したkey生成
        1. script生成
            ```
            $ nano ~/tb3_ssh_keygen
            ```

            ```
            #!/bin/bash
            argc=$#
            args=("$@")

            if [ 0 -eq $argc ]
            then   	 
            echo "need to argument that host ip for ssh connection"
            echo "Usage: $0 [ip address] ..."
            exit 1
            fi

            for((index = 0; index < $#; index++ ))
            do
            ssh-keygen -R ${args[$index]}
            ssh-keyscan ${args[$index]} >> ~/.ssh/known_hosts
            done
            ```

        2. 実行権限を追加
            ```
            $ chmod +x ~/tb3_ssh_keygen
            ```
        3. script実行（ex.TB3のIPアドレスが10.17.3.11〜30である場合、20台のTB3に固定IPを設定後、TB3が起動している状態でコマンドを実行する必要がある）
            ```
            $ ~/tb3_ssh_keygen 10.17.3.11 10.17.3.12 10.17.3.13 10.17.3.14 10.17.3.15\
            10.17.3.16 10.17.3.17 10.17.3.18 10.17.3.19 10.17.3.20\
            10.17.3.21 10.17.3.22 10.17.3.23 10.17.3.24 10.17.3.25\
            10.17.3.26 10.17.3.27 10.17.3.28 10.17.3.29 10.17.3.30
            ```
    - turtlebot3_robot_machine.launch 制作
        1. machineタグはnode用タグでありincludeタグでは動作しない。turtlebot3_robot.lauchファイルの修正が必要
        2. [Turtlebot3] turtlebot3_robot.launchの代替用途、実行時Turtlebotに直接接続する必要なし、Turtlebot3のROSネットワーク設定を変更する必要もなし。
        3. turtlebot3/turtlebot3_bringup/launchフォルダに以下のファイルを生成
            ```
            $ nano turtlebot3_robot_machine.launch
            ```

            ```
            <?xml version="1.0"?>
            <launch>
            <arg name="multi_robot_name" default=""/>
            <arg name="set_lidar_frame_id" default="base_scan"/>
            <arg name="address" default="10.17.3.91"/>
            <arg name="env_name" default="env.bash"/>
            <arg name="user_name" default="pi"/>
            <arg name="password" default="turtlebot"/>
            
            <!-- setting for machine -->
            <machine name="tb3" address="$(arg address)" env-loader="~/$(arg env_name)" user="$(arg user_name)" password="$(arg password)" />

            <!-- packages for turtlebot3 -->
            <node machine="tb3" pkg="rosserial_python" type="serial_node.py" name="turtlebot3_core" output="screen">
                <param name="port" value="/dev/ttyACM0"/>
                <param name="baud" value="115200"/>
                <param name="tf_prefix" value="$(arg multi_robot_name)"/>
            </node>
            
            <node machine="tb3" pkg="hls_lfcd_lds_driver" type="hlds_laser_publisher" name="turtlebot3_lds" output="screen">
                <param name="port" value="/dev/ttyUSB0"/>
                <param name="frame_id" value="$(arg set_lidar_frame_id)"/>
            </node>

            <node machine="tb3" pkg="turtlebot3_bringup" type="turtlebot3_diagnostics" name="turtlebot3_diagnostics" output="screen"/>

            </launch>
            ```


### III-2-G. Turtlebot3, Simulation切り替え実行方法  
すべて[User PC]（Remote PC）で実行
1. TB3実行方法
    - roscore  
    ターミナルを開いて以下のコマンドを入力
        ```
        $ roscore
        ```

    - Turtlebot (リモート)駆動 : turtlebot3_robot_machine.launch  
    別のターミナルを開いて、以下のコマンドを入力（接続しようとするturtlebot3のIPアドレスが10.17.1.11であると仮定）  
    ```
    $ roslaunch turtlebot3_bringup turtlebot3_robot_machine.launch address:=10.17.1.11
    ```  
    
    ![イメージリンク](http://emanual.robotis.com/assets/images/platform/turtlebot3/bringup/run_rviz.jpg)
    
    1. SSHエラー発生時は、以下のコマンドを入力して再実行 
        ```
        $ ~/tb3_ssh_keygen 10.17.3.11
        ```
    2. 他のユーザーがTurtlebotを使用している場合、launchファイル実行時に以下のメッセージが出て終了
        ```
        RLException: remote roslaunch failed to launch: tb3
        The traceback for the exception was written to the log file
        ```

    - Robot Model & TF : turtlebot3_remote.launch  
    別のターミナルを開いて、以下のコマンドを入力
        ```
        $ roslaunch turtlebot3_bringup turtlebot3_remote.launch
        ```

    - RVIZ  
    新たにターミナルを開いて、以下のコマンドを入力（Waffle Piを使用する場合は、$ {TB3_MODEL}の代わりにwaffle_piを入力）
        ```
        $ export TURTLEBOT3_MODEL=${TB3_MODEL}
        $ rosrun rviz rviz -d `rospack find turtlebot3_description`/rviz/model.rviz
        ```

2. Simulation(Gazebo) 実行方法
参考 : [e-Manual](http://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/#turtlebot3-simulation-using-gazebo)

    - roscore  
    ターミナルを開いて、以下のコマンドを入力
        ```
        $ roscore
        ```

    - gazebo実行  
    別のターミナルを開いて、以下のコマンドを入力（Waffle Piを使用する場合は、$ {TB3_MODEL}の代わりにwaffle_piを入力）
        ```
        $ export TURTLEBOT3_MODEL=${TB3_MODEL}
        $ roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
        ```

    - RVIZ  
    新たにターミナルを開いて、以下のコマンドを入力（Waffle Piを使用する場合は$ {TB3_MODEL}の代わりにwaffle_piを入力）
        ```
        $ export TURTLEBOT3_MODEL=${TB3_MODEL}
        $ roslaunch turtlebot3_gazebo turtlebot3_gazebo_rviz.launch
        ```
    ![イメージリンク](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_gazebo_rviz.png)


3. 使用切り替え
    - リアルタイムでの切り替え：gazeboは/ use_sim_timeパラメータを使用しており、リアルタイムでの切り替え（roscoreを維持した切り替え）が困難
    - 切り替え方法
        - roscoreを含むすべてのnodeを終了（実行したターミナルで`CTRL` + `C`キーを入力）
        - 上記の実行方法に従って切り替え
    - Turtlebot3 実ロボットとSimulationの比較

|      | タートルボット3  | シミュレーション(Gazebo) |
|:----:|:----------------|:------------------------|
| 環境 | 多様な実際の環境 |ガゼボ環境<br />- 提供<br />&nbsp;&nbsp;- Empty World<br />&nbsp;&nbsp;- Turtlebot3 World<br />&nbsp;&nbsp;- Turtlebot3 House<br />- ユーザーが制作した環境<br />![img]()|
|モデル|Burger<br/>Waffle<br/>Waffle Pi|Burger<br/>Waffle<br/>Waffle Pi|
|センサーおよびトピック名|LIDAR : /scan<br />IMU : /imu<br />CAMERA(Waffle Pi) : /raspicam_node/image/compressed|LIDAR : /scan<br />IMU : /imu<br />CAMERA(Waffle, Waffle Pi) : <br />&nbsp;&nbsp;/camera/rgb/image_raw,<br />&nbsp;&nbsp;/camera/rgb/image_raw/compressed
|使用機器|Turtlebot3(Burger, Waffle, Waffle Pi)<br />Remote PC(User PC)|Remote PC(User PC)|



## III-3. Machine Learning I : object_detector_3d

### III-3-A. 目標
Machine Learningフレームワークの一つであるchainerを利用して物体を認識し、Depth cameraと連動してその物体との距離を求める。(リンク : [chainer](https://chainer.org/))


### III-3-B. 動作環境
- Ubuntu 16.04
- ROS Kinetic
- Python 2.7.16
- Intel RealSense D435


### III-3-C. 設定(Setup)
1. ROS Kineticをインストール : [wiki.ros.org](http://wiki.ros.org/kinetic/Installation/Ubuntu)を参照

2. RealSense D435 ROSパッケージをインストール
    ```
    $ sudo apt install ros-kinetic-realsense2-camera
    ```

3. 依存パッケージをインストール
    - chainer, chainercv
        ```
        $ pip install chainer chainercv
        ```
        > pipがない場合はインストール
        ```
        $ sudo apt install python-pip
        ```
    - ros_numpy
        ```
        $ sudo apt install ros-kinetic-ros-numpy
        ```

4. object_detector_3dをインストール
    - catkin workspaceに移動
        ```
        $ cd ~/catkin_ws/src
        ```
    - object_detector_3d ソースコードをダウンロード
        ```
        $ git clone https://github.com/ROBOTIS-GIT/object_detector_3d.git
        ```
    - modelをダウンロード(github lfsを利用)
        - github lfs をインストール
            ```
            $ curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
            $ sudo apt install git-lfs
            ```
        - ダウンロード
            ```
            $ cd ~/catkin_ws/src/object_detector_3d
            $ git lfs pull
            ```
    - コンパイル
        ```
        $ cd ~/catkin_ws
        $ catkin_make
        ```
 
### III-3-D. 実行（PCによって異なるが、実行に数秒以上必要）
1. realsense & object_detector_3d node
    ```
    $ roslaunch object_detector_3d run.launch
    ```
2. rviz
    ```
    $ roslaunch object_detector_3d rviz.launch
    ```


### III-3-E. 実行画面

![img]()
> 画面左：rostopic echo /object_detection_3dによる出力が表示されている。

> 画面右上：rvizを介して/object_detection_3d/result_imageが表示されている。

画面右上を見ると、手前からキーボード、マグカップ、瓶、2台のモニターがそれぞれ検出されていることが分かる。
それぞれの探知にキャプションが付いており、以下の情報が表示されている。
- 物体の名前
- 検出スコア。 区間[0，1]に含まれる数値で、1に近いほど信頼度が高いことを意味する。
- 物体の中心点（後述）の3次元座標。 座標系の原点はカメラの中心であり、x、y、z軸方向は、それぞれ右、下、内側方向で、単位はメートル。

5つの検出結果のうち、深さ方向の距離であるz値を見ると、物体の位置が手前から奥に行くほど値が大きくなっていることを確認できる。

### III-3-F. ROS ノード
1. Topic
    - Subscribed Topics
        - /camera/color/image_raw [sensor_msgs/Image]  
        カラーイメージ、2Dオブジェクトの検出に利用

        - /camera/depth/color/points [sensor_msgs/PointCloud2]  
        3D PointCloud、3次元位置を求めるために使用  
        使用時、上記camera imageと時刻の同期が必要。

    - Published Topics
        - /object_detection_3d [object_detector_3d/Detection3DResult]
            ```
            int32 num_detections
            Detection3D[] detections
            ```
        このトピックは、検出した物体の数（num_detections）と検出情報（detections）で構成されている。 
        検出情報であるDetection3Dは、以下のような情報で構成されている。
        ```
        int32 class_id
        string class_name
        float32 score
        float32 y_min
        float32 x_min
        float32 y_max
        float32 x_max
        geometry_msgs/Point position
        ```

        class_id、class_nameは検出された物体の分類番号と名前で、scoreは検出の信頼度を意味する。y_min、x_min、y_max、x_maxは、検出された物体の境界ボックス（bounding box）の左上と右下の座標である。また、positionは物体の3次元位置である。

        - /object_detection_3d/result_image [sensor_msgs/Image]  
        検出に使用された画像に検出された物体の情報が含まれた結果イメージ。上記の右のような画像となる。  

    - その他  
        - sensor_msgs.CameraInfo:/camera/color/image_rawの内部パラメータ
        - realsense2_camera.Extrinsics:点群の座標系からカラーカメラ座標系に変換するための外部パラメータ


### III-3-G. 実装の詳細 (Implementation details)
1. 入出力データ
    - 入力
        - 2Dカメラ画像
        - 3D PointCloud
        - カメラの内部パラメータ
        - PointCloud座標系でカメラの位置を示す外部パラメータ

    - 出力
        - PointCloud座標系における物体の中心点の座標
        - 物体の種類
        - 検出(Detection)の信頼度

2. アルゴリズムの概要  
以下の手順に従って物体の3D座標を抽出
    - (1)カメラから得たイメージを入力として、物体検出器（Object Detector）を利用してイメージから複数の物体を検出する。
    - (2)各検出に対応するbbox（bounding box）のPointCloud座標系から錐台（frustum）を求める。
    - (3)各検出に対応する視錐台（view frustum）について、その中に含まれるpointのサブセットを抽出する。
    - (4)各pointサブセットグループについて、中心点の座標を求める。
    - (5) 2Dの検出結果と3D中心点の座標を統合し、3D検出結果を算出する。

3. 各アルゴリズムの説明
    - (1) 2D物体の検出(2D Object detection)  
    2D物体検出器は、イメージに含まれた（事前に定義された種類の）物体を検出するものを指す。 検索は、2D画像を入力すると以下のように情報が出力される。  
    これは、複数の物体それぞれについての予測値である。
        - 物体を囲むボックス（axis aligned bounding box, bbox）
        - 物体の種類
        - 予測の信頼度  

        具体的に使用する物体検出器（Object Detector）は、[MS COCO](http://cocodataset.org/)データセットによって学習された[SSD300](https://arxiv.org/abs/1512.02325)を利用しており、この物体検出器（Object Detector）はCNNを利用して80種の物体を検出する。
        具体的な検出項目は、リンク先のリストを参照のこと。
        実装には、pythonのdeep learning frameworkである [chainer](https://github.com/chainer/chainer)を使用しており、特に画像処理に関しては、[chainercv](https://github.com/chainer/chainercv)を使用する。

    - (2, 3) 検出した物体のpointサブセットを抽出  
    ここでは、前段階で得たbbox情報を利用する。個別bboxのPointCloudの全pointのうち、カメラ視点から見てbbox中に入る点のみ部分PointCloudとして抽出している。

    - (4)部分PointCloudでの中心点を計算  
    部分PointCloudは、対象物体と背景および遮蔽された物体から得られたpointによって構成される。これらの点を代表する一つの集約された点を求めることになるが、これを中心点とする。  
    中心点の定義は様々であるが、ソフトウェアは部分PointCloudの中心を中心点として定義する。  
    しかし、この方法は抽出した物体と物体でない部分を区別せず点データとして利用しているため、物体の形状やbboxの違いなどによって、物体そのものの中心から外れた位置が中心として算出されてしまう可能性がある。抽出した物体でない部分は部分PointCloudから除去した後、残ったPointCloudで中心を計算する方法が、より良い中心点の定義であると考えられる。

    - (5) 2D検出と3D中心点の座標の統合  
    単純であるため省略。


## III-4. Machine Learning II: YOLO

### III-4-A. 目標
ROS環境でYOLOを使用して物体の認識を試みる。YOLO（You Only Look Once）はリアルタイム物体探索システムで、他の物体認識エンジンに比べて高速性を誇る。YOLOはDNN（deep neural network）を学習させて実行するニューラルネットワークフレームワーク（neural network framework）であるdarknetを利用して駆動。

![img]()

### III-4-B. 動作環境
- Ubuntu 16.04
- ROS Kinetic
- Intel RealSense D435


### III-4-C. 設定(Setup)
- ROS Kinetic をインストール : [wiki.ros.org](http://wiki.ros.org/kinetic/Installation/Ubuntu)を参照のこと

- RealSense D435 ROS パッケージをインストール  
    ```
    $ sudo apt install ros-kinetic-realsense2-camera
    ```

- [darknet_ros](https://github.com/leggedrobotics/darknet_ros)(ROS用 YOLO) インストール
    - catkin workspaceに移動
        ```
        $ cd ~/catkin_ws/src
        ```
    - ソースコードをダウンロード
        ```
        $ git clone --recursive https://github.com/leggedrobotics/darknet_ros.git
        ```
    - コンパイル
        ```
        $ cd ~/catkin_ws
        $ catkin_make -DCMAKE_BUILD_TYPE=Release
        ```
     - yolo v3用 weightをダウンロード
        ```
        $ cd ~/catkin_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights
        $ wget http://pjreddie.com/media/files/yolov3.weights
        ```
    - darknet_ros 設定変更
        - darknet_ros/config/ros.yaml ファイルをエディタで開く。
            ```
            $ nano ~/catkin_ws/src/darknet_ros/darknet_ros/config/ros.yaml
            ```
        - camera_readingにある topicを /camera/color/image_raw に変更して保存。
            ```
            subscribers:

            camera_reading:
                topic: /camera/color/image_raw
                queue_size: 1
            ```

### III-4-D. 実行（すでに学習したモデルを使用）
1. realsense
    ```
    $ roslaunch realsense2_camera rs_camera.launch
    ```

2. YOLO(darket_ros)
    ```
    $ roslaunch darknet_ros darknet_ros.launch
    ```

### III-4-E. 実行画面

![img]()

> 上の写真のように複数の物体が同時に認識され、認識された物体の境界にボックスが表示され、物体の名前がボックスの左上に現れる。


### III-4-F. ROSノード
1. Topic
    - Subscribed Topics
        - /camera/color/image_raw [sensor_msgs/Image]  
        カラーイメージ、物体検出に利用

    - Published Topics
        - /darknet_ros/bounding_boxes [darknet_ros_msgs/BoundingBoxes]  
        認識した物体の情報を含むtopicで、以下のようにメッセージのheader、検出に使用したイメージのheader、検出した物体の情報であるBoundingBoxで構成されている。
            ```
            Header header
            Header image_header
            BoundingBox[] bounding_boxes
            ```
            物体の情報を示すBoundingBoxは、以下の通りである。(BoundingBox.msg)  
            ```
            float64 probability
            int64 xmin
            int64 ymin
            int64 xmax
            int64 ymax
            int16 id
            string class
            ```
            検出の精度を示すprobability、検出した物体を表すイメージ上境界ボックスのx、y位置、物体の区分番号であるid、物体の種類を表すclassで構成されている。

        - /darknet_ros/detection_image  [sensor_msgs/Image]  
        検出に使用されたイメージに、検出された物体の情報が含まれた結果イメージ

        - /darknet_ros/found_object [std_msgs/Int8]  
        検出された物体の個数を表示

2. Actions
    - camera_reading [sensor_msgs::Image]  
    イメージと結果値（検出された物体の境界ボックス）を含んだアクションを送る。

3. Parameters   
検出に関するパラメータの設定は、`darknet_ros/config/yolo.yaml`と類似した名前のファイルで行うことができる。 ROSに関するパラメータの設定は、`darknet_ros/config/ros.yaml`ファイルで行うことができる。

    - image_view/enable_opencv (bool)  
        bounding boxを含む検出画像を示すopen cv viewerを再起動。
    - image_view/wait_key_delay (int)  
        open cv viewerで wait key delay(ms)
    - yolo_model/config_file/name (string)  
        検出に使用するネットワークのcfg名。プログラムはdarknet_ros/yolo_network_config/cfgフォルダから、名称に合ったcfgファイルを読み込んで使用する。
    - yolo_model/weight_file/name (string)  
        検出に使用するネットワークのweightファイル名。プログラムはdarknet_ros/yolo_network_config/weightsフォルダから名称に合ったweightsファイルを読み込んで使用する。
    - yolo_model/threshold/value (float)    
        検出アルゴリズムのthreshold、0と1の間の値である。
    - yolo_model/detection_classes/names (array of strings) 
        ネットワークが検出可能な物体の名前(検出可能な物体のクラス)

### III-4-G. GPUアクセラレーション
NVidia GPUがある場合、CUDAを利用するとCPUのみを使用するよりも何倍も高速な検出が可能である。CUDAをインストールすると、CMakeLists.txtファイルから自動的に認識し、コンパイル（catkin_make）時にGPUモードでコンパイルされる。
- [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit)


### III-4-H. 参考サイト
- [darknet](https://pjreddie.com/darknet/yolo/)
- [darknet_ros](https://github.com/leggedrobotics/darknet_ros)
