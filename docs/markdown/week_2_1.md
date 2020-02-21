---
layout: splash
lang: en
ref: ritsumeikan2-1
permalink: /docs/ritsumeikan/week2-1/
sidebar:
  title: RITSUMEIKAN
  nav: "ritsumeikan2-1"
---

# [Class 3] LRF(LDS)センサー

TurtleBot3に使われているLDS(LASER Distance Sensor)は、2次元平面の360度距離値を読み取ることができるセンサーで、SLAMとNavigationに必要な重要情報を提供します。
LDSセンサーはUSBインターフェースであるUSB2LDSボードを通じ、Raspberry PiとUSBケーブルを介して接続されますが、UARTインタフェースを介してOpenCRとも直接接続することができます。

センサーの特性上、直射日光が強く当たる屋外環境では使用が困難で、10,000lux以下の明るさの屋内空間で走行するロボットに適しています。

TurtleBot3의 LDS 센서는 패키지의 형태로 설치하거나 소스코드를 다운로드 받아서 빌드할 수 있습니다.

## センサーパッケージをインストール
```bash
$ sudo apt-get install ros-kinetic-hls-lfcd-lds-driver
```
TurtleBot3のLDSセンサーを駆動するのに必要なドライバパッケージをインストールします。

### センサーの接続ポート権限を設定
```bash
$ sudo chmod a+rw /dev/ttyUSB0
```
センサーを、USBを用いてLinuxがインストールされたPCと接続する場合は、USBポートの権限を正しく設定することで、割り当てられたポートにアクセスすることができます。  
上のコマンドは、USB0番ポートに読み出し/書き込みアクセス権を設定します。

### hlds_laser_publisherノードを実行
```bash
$ roslaunch hls_lfcd_lds_driver hlds_laser.launch
```
{% capture capture00 %}
**roslaunch hls_lfcd_lds_driver hlds_laser.launch**
1. hlds_laser_publisher
    - publish : scan, rpms

hlds_laser.launchを実行するとhlds_laser_publisherノードが生成され、センサーのデータと回転速度をそれぞれscanとrpmsのtopicでpublishします。 sensor_msgsタイプのLaserScanメッセージであるscanにはLDSセンサーが回転し、獲得したロボット周辺の物体との距離データが配列の形で蓄積、保存されます。
{% endcapture %}
<div class="notice--success">{{ capture00 | markdownify }}</div>

### RVizとhlds_laser_publisherノードの実行
```bash
$ roslaunch hls_lfcd_lds_driver view_hlds_laser.launch
```

{% capture capture01 %}
**roslaunch hls_lfcd_lds_driver view_hlds_laser.launch**
1. hlds_laser_publisher
    - publish : scan, rpms

2. rviz
    - subscribe : scan

view_hlds_laser.launchを実行すると、hlds_laser.launchファイルとrvizノードが実行されます。  
hlds_laser.launchを実行するとhlds_laser_publisherノードが生成され、センサーのデータと回転速度をそれぞれscanとrpmsのtopicでpublishします。 sensor_msgsタイプのLaserScanメッセージであるscanにはLDSセンサーが回転し、獲得したロボット周辺の物体との距離データが配列の形で蓄積、保存されます。  
RVizノードでは、rvizの設定ファイルを読み込み、画面に表示してscanデータをsubscribeし、3次元グラフィックスで視覚化します。
{% endcapture %}
<div class="notice--success">{{ capture01 | markdownify }}</div>

## センサーのソースコードをダウンロード

ダウンロードされたLDS-01をサポートするドライバは、Windows、Linux、MacOSの開発環境で実行することができます。  
ソフトウェア条件は以下の通りです。

- GCC(Linux, MacOS使用時), MinGW(Windows使用時)
- Boost library(v1.66.0でテスト済み)

1. 以下のGitHubアドレスからソースコードをダウンロードします。
  ```bash
  $ git clone https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver.git
  ```
  または、以下のサイトで緑の`Clone or download`ボタンをクリックし、ソースコードをダウンロードすることも可能です。

  - [https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver](https://github.com/ROBOTIS-GIT/hls_lfcd_lds_driver)

2. 開発環境に必要なソフトウェアとライブラリをインストールします。
  - GCC(Linux, MacOS)またはMinGW(Windows)
  - Boost library

### ソースコードをビルド
以下のコマンドは、Linux環境に合うように設定されたmakefileを利用して、ソースコードをビルドします。WindowsとMacOS環境の場合、makefileを適切なものに修正してください。

```bash
$ cd hls_lfcd_lds_driver/applications/lds_driver/
$ make
```

### CLI環境で実行
ソースコードが正しくビルドされると、以下のように `./lds_driver`ファイルを実行してLDSセンサー値を確認することができます。

```bash
$ ./lds_driver
```
```
r[359]=0.438000,r[358]=0.385000,r[357]=0.379000,...
```

### GUI環境で実行
LDSセンサーの値を視覚的に確認するためには、Qt CreatorとQt Libsを追加でインストールする必要があります。
- Qt Creator(v4.5.0でテスト済み)
- Qt Libs(v5.10.0でテスト済み)

1. 以下のQtサイトでOpen Source版をインストールしてください。
  - [https://www.qt.io/download](https://www.qt.io/download)

2. Qt Creatorを実行します。
3. ソースコードから、以下の位置にある`lds_polar_graph.pro`ファイルを開きます。  
  (hls_lfcd_lds_driver/applications/lds_polar_graph/lds_polar_graph.pro)
4. ソースコードでセンサーと接続されたポートを確認し、他のポートに割り当てられている場合、該当するポート番号に変更します。
5. `CTRL` + `SHIFT` + `B`を押して、ソースコードをビルドします。
6. ビルドが正常に完了すると、`CTRL` + `R`を押してプログラムを実行します。

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/appendix_lds/lds_gui.png)

### Embeddedボードで実行

LDS-01センサーは、OpenCRまたはArduinoボードで動作させることが可能です。
この場合、センサーデータの視覚化のため、LCDパネルが必要です。

LDS-01センサーのTX、RXケーブルはembeddedボードのUARTピンと互換性があり、embeddedボードに電源とTX、RXピンを接続して使用することができます。  
実際のケーブルの色は下の図と異なる場合があるため、必ず製品のデータシートを参照してください。

![](/assets/images/ritsumeikan/lds_lines.png)

#### OpenCRからLDSセンサーを読み込む
OpenCRボードからセンサーの値を読み取るには、Arduino例題をOpenCRボードにアップロードする必要があります。

1. Arduino IDEのTools > Board > Boards ManagerでOpenCRを検索し、ライブラリをインストールします。
2. Tools > BoardからOpenCRを選択します。
3. Tools > PortでOpenCRが接続されたポートを選択します。
4. File > Examples > OpenCR > Etc > LDS > drawLDS例題を選択してOpenCRにアップロードします。

例題のアップロードが完了すると、OpenCRと接続されたLCDに、以下の画像のようにセンサーの値が視覚的に確認できるようになります。

![](/assets/images/ritsumeikan/011.png)

# SLAM

SLAM(Simultaneous Localization and Mapping)とは、未知の領域を探索し、ロボットに取り付けられたセンサーを通じて取得された情報を使用して、ロボットが自己位置推定及び探索する位置の地図を作成することを意味します。SLAMは、Navigationや無人自動車の自律走行に欠かせない重要な技術です。外部情報を取得するために使用されるセンサーには、距離を測定することができるセンサーや周辺の画像を取得できるセンサーがあります。赤外線距離センサー(IR)、音波センサー(SONAR)、レーザーセンサー(LRF)などがよく使用されており、最近では映像を解析するアルゴリズムの発展によって、カメラが数多く使用されてもいます。  
ロボットの位置を予測するため、ロボットの車輪と接続されたエンコーダの値を読み取り、推測航法(Dead Reckoning)を使用して走行距離(odometry)を計算することになりますが、この時、車輪と地面の間の摩擦などによって誤差が生じます。実際のロボットの位置と予測されたロボットの位置の誤差を減らすために、慣性測定センサ(IMU)から得られたデータを利用して位置補正を行うことができます。  
また、距離センサー値を用いて位置の誤差を低減するために使用される方法として、Kalman filter、Markov Localization、Monte Carlo Localizationなどがあります。

SLAMでマップを作成する際、いくつかの注意点があります。  
広い倉庫やホールのようにセンサーの範囲で届かない広い領域の地図を作成する場合、地図が正しく描画されません。これは、センサーの範囲を両腕の長さであると仮定したとき、ホールの中心で目を閉じて両腕を使って現在地を探ろうとすることと同じです。  
これと類似して、特徴点のない長い廊下も地図の描画が困難です。特徴のない両壁面からなる長い廊下で目を閉じ、壁を手でつたって歩くと、廊下のどの位置まで来たのか把握しづらいことと同様です。

このような場合、マッピングアルゴリズムが特徴点として参照できるよう、地図上のあちこちに物体や障害物を設置する方法を検討できます。このような特徴点は、一定のパターンや対称の形で置くよりも、パターンがない形で置くことが最もよいでしょう。

一般的によく使用されるGmappingとCatographerには、地図を作成する方法に若干の違いがあります。mapをすぐにpublishするGmappingとは異なり、Catographerはsubmapをpublishし、submapを集めてmapを生成します。これによって、互いに繋がった広範囲の地図を作成する場合には、Catographerがより正確な形の地図を作る助けになります。


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
1. **roslaunch turtlebot3_bringup turtlebot3_remote.launch**
  - urdf：Unified Robot Description Formatの略で、ロボットの構成と接続形態を表すXML形式のファイルです。
  - robot_state_publisher : robot_state_publisherでは、ロボットの各関節の情報を受信し、得られた関節についての情報をurdfを参考にtfの形式でpublishします。
  - subscribe : joint_states 
  - publish : tf

    turtlebot3_remote.launchファイルを実行すると、ロボットのurdfを定義された位置から読み込みます。また、joint_statesとurdfを利用して、tfをpublishするrobot_state_publisherノードを生成します。  
    turtlebot3_slam.launchファイル内部にturtlebot3_remote.launchが含まれているのでturtlebot3_slam.launchが実行されると自動的にturtlebot3_remote.launchが最初に実行されます｡

2. **turtlebot3_gmapping.launch**
  - subscribe : scan, tf
  - publish : map, map_metadata

    実行文の末尾にslam_methods:=gmappingというオプションを入力したため、gmappingの実行に関連するファイルであるturtlebot3_gmapping.launchが実行されます。このファイルの内部では、再びLDS設定が保存されたtu​​rtlebot3_lds_2d.luaを呼び出し、gmappingの使用に必要な各種パラメータを定義して、gmappingパッケージのslam_gmappingノードを実行します。 slam_gmappingノードが生成されると、scanとtfトピックをsubscribeして、マップの生成に必要なmap_metadataとmapをpublishします。

3. **rviz**
  - subscribe : tf, scan, map

    最後にrvizの設定ファイルを適用したrvizが実行され、tf、scan、mapデータをsubscribeしてロボットとセンサー値、gmappingによって生成されたマップを視覚化します。  
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
    turtlebot3_slam.launchファイル内部にturtlebot3_remote.launchが含まれているのでturtlebot3_slam.launchが実行されると自動的にturtlebot3_remote.launchが最初に実行されます｡

2. turtlebot3_cartographer.launch
  - subscribe : scan, imu, odom
  - publish : submap_list, map
  
    実行文の末尾にslam_methods:=cartographerというオプションを入力したため、cartographerの実行に関連するファイルであるturtlebot3_cartographer.launchが実行されます。このファイルの内部では、再びLDSの設定が保存されたtu​​rtlebot3_lds_2d.luaを呼び出し、cartographerの使用に必要な各種パラメータを定義して、cartographer_rosパッケージのcartographer_nodeノードを実行し、scan、imu、odomなどのtopicをsubscribeしてsubmap_listをpublishします。また、cartographer_rosパッケージのcartographer_occupancy_grid_nodeノードを実行し、cartographer_nodeでpublishされたsubmap_listをsubscribeしてmapをpublishします。

3. rviz
  - subscribe : tf, scan, map

    最後にrvizの設定ファイルを適用したrvizが実行され、tf、scan、mapデータをsubscribeしてロボットとセンサー値、gmappingによって生成されたマップを視覚化します。
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
Gmappingには様々な環境に対して性能を変更させるたくさんのパラメータがあります。ROSウィキですべてのパラメータに対する情報を得るか、ROSロボットプログラミングの第11章を参照にしてください。 
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
[遠隔PC] すべての作業が終わりましたので、map _saverノードを実行してマップファイルを作ります。マップはロボットが動く時にロボットの走行距離、tf 情報、センサーのスキャン情報をベースに作成されます。当データーは以前動画のRVizで見ることができます。作成されたマップはmap_saverが実行されるディレクトリに保存されます。ファイル名を指定しないと、マップ情報が入っているmap.pgmおよびmap.yamlファイルに保存されます。 完成した地図は2つのファイルに分けて保存され、そのうち、pgmはPortable Gray Map形式の画像ファイルであり、yamlは地図の解像度など各種設定を保存するファイルです。

```bash
$ rosrun map_server map_saver -f ~/map
```

`-f` オプションは、マップファイルが保存されたフォルダーおよびファイル名が表示されます。~ / mapをオプションで使用すると、map.pgmとmap.yamlが使用者のホームフォルダー~ / ($HOME directory :  /home/\<username\>)のマップフォルダーに保存されます。 

## マップ 
ROSコミュニティーで一般的に使用される２次元OGMを使用します。下記のイメージのように以前のSave Mapセクションで得たマップで、ホワイトはロボットが動ける空きゾーン、ブラックはロボットが動けないゾーン、グレーは不明なゾーンです。当マップは探索に使用されます。

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/map.png)

下記のイメージはタートルボット３を使って大きなマップを作成したその結果です。約３５０ｍの走行距離のマップ作成には約１時間かかりました。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/slam/large_map.png)
