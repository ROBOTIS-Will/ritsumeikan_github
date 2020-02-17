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


## ナビゲーションノードの実行 
1. [遠隔PC] ロスコアを実行してください。 
  ```bash
  $ roscore
  ```
2. [タートルボット] タートルボット３の応用プログラムを始めるための基本パッケージを持ってきてください。 
  ```bash
  $ roslaunch turtlebot3_bringup turtlebot3_robot.launch
  ```
3. [遠隔PC] 探索ファイルを実行してください。  
  当コマンドを行う前にタートルボット３のモデル名を指定しなければなりません。$ {TB3_MODEL}は、ハンバーガー、ワッフル、ワッフルパイ(waffle_pi)で使用するモデル名です。エキスポートの設定を永久設定するためには、タートルボット３_MODELのエキスポートページを参照してください。 
  ```bash
  $ export TURTLEBOT3_MODEL=${TB3_MODEL}
  $ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
  ```
  上記の命令を実行すると、ビジュアル化ツールRVizが実行されます。RVizを別途実行するためには、下記の命令のコマンドをを使用してください。 
  ```bash
  $ rviz -d `rospack find turtlebot3_navigation`/rviz/turtlebot3_navigation.rviz
  ```

## 初期ポーズの推定 
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

警告 : 初期ポーズの推定に使用されるturtlebot3_teleop_keyboardノードは使用後に終了しなければなりません。そうしないと、トピックが次のステージの探索ノードで/ cmd_velのトピックと重なるため、ロボットが異常作動します。 

## ナビゲーション目標の送信 
[遠隔PC] すべてが用意されると、探索GUIで移動コマンドをチャレンジします。2Dを押すと、RVizメニューでNav Goalを選択すると、大きな緑の矢印が表示されます。この緑の矢印は、ロボットの目的地が指定できるマーカーです。矢印の後ろはロボットのXとYの位置で、先端はロボットのセタ方向です。ロボットが動く位置でこの矢印をクリックし、下記のような方向を設定するためにはドラッグします。 

1. 2D Nav Goalボタンをクリックしてください。
2. マップで特定地点をクリックして目標位置を設定し、カーサーをタートルボットが終わる方向へドラッグします。 

ロボットはマップをベースに目的地までの障害物が避けられる経路を作ります。その後、ロボットはその径路にそって移動します。その時に突然、障害物が感知されても障害物を避けて目標地点まで移動します。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/2d_nav_goal.png)

目標位置への経路が作成でいない場合は、目標位置の設定に失敗することがあります。目標位置に到達する前にロボットを停止させるためには、タートルボット３の現在位置を目標位置として設定してください。 

## チューニングガイド 
ナビゲーションスタックにはロボット性能を変更することができるたくさんのパラメータがあります。ROS　WIKIで情報を得るか、ROSロボットプログラミングの第１１章を参照にしてください。 
チューニングガイドは重要なパラメータを設定できるようにいくつかのTIPを提供します。パフォーマンスを環境によって変更するためには、TIPが役に立つのはもちろん、事案節約にもなります。 

turtlebot3_navigation/param/costmap_common_param_$(model).yaml 

### inflation_radius
当パラメータは障害物で膨張エリアを作ります。経路はこのエリアを横切ることがないように設計されます。この値をロボット半径より大きく設定することが安全です。それに関する詳細は、costmap_2d wikiの次のページを参照にしてください。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tuning_inflation_radius.png)

### cost_scaling_factor 
当要素にコスト値をかけます。それは相互比例のため、当パラメータが増加し、コストは減少します。
最善の経路はロボットが障害物の間の中心を通過することです。障害物と遠く離れるよう、この要素を小さく設定してください。 

![](http://emanual.robotis.com/assets/images/platform/turtlebot3/navigation/tuning_cost_scaling_factor.png)

### max_vel_x 
当係数には並進スピードの最大値を設定します。 
 
### min_vel_x 
当係数には並進スピードの最小値を設定します。当値をマイナスに設定すると、ロボットが後ろへ移動します。 

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