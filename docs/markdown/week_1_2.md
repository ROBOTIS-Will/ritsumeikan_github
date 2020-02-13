---
layout: splash
lang: en
ref: ritsumeikan1-2
permalink: /docs/ritsumeikan/week1-2/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan1-2"
---

# [Class 2] Gazebo Simulator & Switching Sim - Real robot

## Simulation

TurtleBot３はシミュレーションを行い、仮想ロボットでプログラミングや開発可能な環境を支援します。そのため、２つの開発環境が提案できます。一つ目な、フェイクノードと３D視覚化ツールであるRVizを使うこと、残りもう一つの方法は、３Dロボットシミュレーターガゼボを使用することです。 
フェイクノード方法はロボットモデルと動きをテストすることに適していますが、センサーを使用することはできません。スラムおよび探索をテストしなければならない場合、シミュレーションでMU、LDS およびカメラのようなセンサーが使用できるガゼボを使用することをおすすめします。 

**注意**  
当指針はUbuntu 16.04およびROSKinetic Kameでテストしました。  
当指針は遠隔PCで実行することを想定しています。下記の指針を遠隔PCで実行してください。 
{: .notice}

### フェイクノードを利用したTurtleBot３シミュレーション 

turtlebot3_fake_nodeを使用するためには、turtlebot3_simulationメタ―パッケージが必要です。次のコマンドの通りパッケージをインストールしてください。 turtlebot3_simulation メタパッケージは、turtlebot３メタパッケージおよびturtlebot3_msgsが必須条件です。

**TIP**  
ターミナル応用プログラムは画面の左上にあるウブトゥ検索アイコンで検索できます。ターミナルを実行するためのショートカットキーは、`Ctrl` - `Alt` - `T`です。
{: .notice}

```bash
$ cd ~/catkin_ws/src/
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ cd ~/catkin_ws && catkin_make
```

仮想ロボットをスタートするためには、下記のようにturtlebot3_fakeのturtlebot3_fake.launchファイルを実行してください。turtlebot3_fakeは実際のロボットを使用しなくても実行できる非常に簡単なシミュレーションノードです。RVizで仮想タートル３を遠隔操作ノードでコントロールできます。

当コマンドを行う前にTurtleBot３のモデル名を指定しなければなりません。$ {TB3_MODEL}は、burger、waffle、waffle_piで使用するモデル名です。

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_fake turtlebot3_fake.launch
```
{% capture capture01 %}
**roslaunch turtlebot3_fake turtlebot3_fake.launch**
- publish : odom, joint_states
- subscribe : cmd_vel

turtlebot3_fake 노드에서는 하드웨어의 실제 값이 아닌 계산된 값을 이용해 시뮬레이터가 실제 하드웨어와 유사하게 동작하도록 합니다. 입력된 cmd_vel로 계산된 가상의 Turtlebot3의 odom과 joint_states 값을 계산하여 시뮬레이터에 전달하기 위해 publish 합니다.
{% endcapture %}
<div class="notice--success">{{ capture01 | markdownify }}</div>

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

{% capture capture02 %}
**roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch**
- publish : cmd_vel

turtlebot3_teleop_key.launchファイルを実行して生成されたturtlebot3_teleop_keyboardノードでは、キーボードの入力を読み取ってlinearとangular値を更新し、linearとangularが含まれたtwist形式のtopicであるcmd_velをpublishします。  
その後、タートルボットのSBCで実行されたturtlebot3_robot.launchに含まれたturtlebot3_core.launchからcmd_velを受信します。  
cmd_velトピックはrosserialを介してOpenCRに伝達され、OpenCRにアップロードされたファームウェアでDYNAMIXELを制御するためのコマンドとして出力されます。  
受信されたコマンドに従って車輪と接続されたDYNAMIXELが駆動し、ロボットを動かします。
{% endcapture %}
<div class="notice--success">{{ capture02 | markdownify }}</div>

### Gazeboを使ったTurtlebot３シミュレーション 

ガゼボを使ったシミュレーションには２つがあります。まず、turtlebot3_gazeboパッケージでROSと一緒に使用することです。もうひこつは、ROSを使用しないでturtlebot3_gazebo_pluginを使用することです。

#### ガゼボ用ROSパッケージ 

遠隔PCでガゼボを初めて実行する場合は普段より時間がかかります。 

##### エンプティ―世界 
次の命令はガゼボ基本環境が空いているところで、仮想TurtleBot３をテストする際に使用できます。 

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
```

{% capture capture03 %}
**roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch**
- publish : joint_states, odom, scan, tf
- subscribe : cmd_vel

turtlebot3_empty_world.launchを実行すると、ファイル内部の設定に従ってGazeboシミュレータが実行され、設定されたTurtleBot3モデルがGazeboシミュレータに生成されます。この時使用されるxacroとURDFファイルの設定に従ってimu、scan、odom、joint_states、tfのtopicをpublishし、cmd_velをsubscribeします。
その後、teleopノードを介してcmd_velをpublishすると、Gazeboシミュレータがその値をsubscribeし、シミュレータに生成されたロボットが動くことを確認できます。
{% endcapture %}
<div class="notice--success">{{ capture03 | markdownify }}</div>


![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_empty_world.png)

##### TurtleBot３世界 
TurtleBot３ゾーンはTurtleBot３のシンボルを構成する簡単なもので構成されているマップです。TurtleBot３ゾーンは、主にスラムや探索のようなテストに使用されます。 
 
```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_world.launch
```
![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_world_bugger.png)

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_world_waffle.png)

##### TurtleBot３ハウス 
TurtleBot３ハウスは住居の図面で制作したマップです。より複雑な作業性能と関連があるテストに適しています。 

```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_house.launch
```

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_house.png)

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_house1.png)

#### タートルボット3ドライブ

##### ガゼボの遠隔操作 
タートルボット３をキーボードでコントロールするためには、新しいターミナルで下記のコマンドを使って遠隔操作を実行します。 
```bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```
<div class="notice--success">{{ capture02 | markdownify }}</div>


##### 衝突回避 
タートルボット３世界でタートルボット３を自動運転するためには、新しいターミナルウィンドウを開いて下記のコマンドを入力してください。  
```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_world.launch
```
新しいターミナルウィンドウを一つ開いて下記のコマンドを入力してください。 
```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_simulation.launch
```

#### RVizの実行 
RVizはシミュレーションが実行される間に掲示されたトピックを視覚化します。下記のコマンドを入力し、新しいターミナルウィンドウでRVizをスタートすることができます。 
```bash
$ export TURTLEBOT3_MODEL=${TB3_MODEL}
$ roslaunch turtlebot3_gazebo turtlebot3_gazebo_rviz.launch
```
![](http://emanual.robotis.com/assets/images/platform/turtlebot3/simulation/turtlebot3_gazebo_rviz.png)


## SimulationとReal Robotのクロス開発

### 目標
20台のTurtleBotと40台のsimulationの間に円滑なクロス開発が可能な環境およびネットワーク構成案、交互開発のための実行方法の提示

### ネットワーク構成案
- 仮定
  - すべてのTB3とユーザーPC（Simulation）は、固定IPアドレスを持つ。
    ユーザーPCは固定IPアドレスでない場合でも動作可能だと思われるが、ROSネットワーク設定を容易にするために固定IPの使用を推奨。
  - 各ユーザーは、自分の使用するTB3のIPアドレスを知っている。

1. 第1案  
ルータを複数台使用（5台）、4台のTB3と8人（Simulator含む）が一つのルータを使用する（推奨）

2. 第2案  
高性能ルータに40人と20台のTB3すべてを接続（クライアントの要求）

### ネットワーク設定(第1案)

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

                ![](/assets/images/ritsumeikan/001.png)

        - [User PC] (10.17.1.21の場合)
            1. ネットワークマネージャを開いて以下のように設定

                ![](/assets/images/ritsumeikan/002.png)

            2. ルータに接続した後、IPアドレスを確認
                ```
                $ ifconfig
                ```

                ![](/assets/images/ritsumeikan/003.png)

### ネットワーク設定(第2案)
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

                ![](/assets/images/ritsumeikan/004.png)

        - [User PC] (10.17.1.101の場合)
            1. ネットワークマネージャを開いて以下のように設定し、保存

                ![](/assets/images/ritsumeikan/005.png)

            2. ルータに接続し、IPアドレスを確認
                ```
                $ ifconfig
                ```

                ![](/assets/images/ritsumeikan/006.png)

### ROS ネットワーク設定
1. [Turtlebot3]
    - 追加でROSネットワーク設定を行う必要なし

2. [User PC]
    - 参考 : [e-Manual](http://emanual.robotis.com/docs/en/platform/turtlebot3/pc_setup/#network-configuration)
    - ユーザーPCのIP例: 10.17.1.101
    - ユーザーPCのIPを確認（以下のコマンドを入力すると、上で設定したユーザーPCの静的IPが出力される。
        ```bash
        $ ifconfig
        ```

        ![](/assets/images/ritsumeikan/007.png)

    - `~/.bashrc` ファイルを修正：以下のコードがある場合は修正し、ない場合は追加
        ```
        export ROS_MASTER_URI=http://10.17.1.101:11311
        export ROS_HOSTNAME=10.17.1.101
        ```

### その他の設定
1. [Turtlebot3]
    - machineタグを利用したlaunchファイルで使用する`env.bash`ファイルの生成 (位置 : /home/pi)
        ```bash
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


### Turtlebot3, Simulation切り替え実行方法  
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

|      | TurtleBot3  | シミュレーション(Gazebo) |
|:----:|:----------------|:------------------------|
| 環境 | 多様な実際の環境 |ガゼボ環境<br />- 提供<br />&nbsp;&nbsp;- Empty World<br />&nbsp;&nbsp;- Turtlebot3 World<br />&nbsp;&nbsp;- Turtlebot3 House<br />- ユーザーが制作した環境<br />![](/assets/images/ritsumeikan/008.png)|
|モデル|Burger<br/>Waffle<br/>Waffle Pi|Burger<br/>Waffle<br/>Waffle Pi|
|センサーおよびトピック名|LIDAR : /scan<br />IMU : /imu<br />CAMERA(Waffle Pi) : /raspicam_node/image/compressed|LIDAR : /scan<br />IMU : /imu<br />CAMERA(Waffle, Waffle Pi) : <br />&nbsp;&nbsp;/camera/rgb/image_raw,<br />&nbsp;&nbsp;/camera/rgb/image_raw/compressed
|使用機器|Turtlebot3(Burger, Waffle, Waffle Pi)<br />Remote PC(User PC)|Remote PC(User PC)|
