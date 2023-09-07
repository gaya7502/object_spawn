# 煙霧彈投擲方法
## 0. 電腦配置
* Ubuntu 18.04
* ROS Melodic
## 1. 導入煙霧彈模型
* 欲達到投擲結果，系統需先導入投擲物模型，目前以煙霧彈模型為例，可於交付之資料夾中找到"smoke"資料夾，並將該資料夾複製並於路徑:drone_ws/src/asv_wave_sim-gazebo9/asv_wave_sim_gazebo/models 位置進行貼上動作。

* 接著，可開啟smoke資料夾並開啟"model.sdf"文件，可以看見該文件內容之相關程式碼，因本系統具有海水，可使系統中之物件具有漂浮效果，目前該文件內之程式碼是將漂浮效果進行關閉，若要開啟可進行修改，下方之程式碼為關閉狀態。
```xml=
...
    <!--plugin name="hydrodynamics" filename="libHydrodynamicsPlugin.so">

      <wave_model>ocean_waves</wave_model>

      <damping_on>true</damping_on>

      <viscous_drag_on>true</viscous_drag_on>

      <pressure_drag_on>true</pressure_drag_on>

      <markers>

        <update_rate>30</update_rate>

        <water_patch>false</water_patch>

        <waterline>false</waterline>

        <underwater_surface>false</underwater_surface>

      </markers>

    </plugin-->
...
...
    <!--inertial>

        <pose>0 0 0 0 0 0</pose>

        <mass>20000</mass>

        <inertia>

          <ixx>33333</ixx>

          <ixy>0.0</ixy>

          <ixz>0.0</ixz>

          <iyy>173333</iyy>

          <iyz>0.0</iyz>

          <izz>193333</izz>

        </inertia>

    </inertial-->
```
可於程式碼中找到上述內容，若要修改其功能並開啟漂浮效果可修改為下方程式碼樣式。

```xml=
...
    <plugin name="hydrodynamics" filename="libHydrodynamicsPlugin.so">

      <wave_model>ocean_waves</wave_model>

      <damping_on>true</damping_on>

      <viscous_drag_on>true</viscous_drag_on>

      <pressure_drag_on>true</pressure_drag_on>

      <markers>

        <update_rate>30</update_rate>

        <water_patch>false</water_patch>

        <waterline>false</waterline>

        <underwater_surface>false</underwater_surface>

      </markers>

    </plugin>
...
...
    <inertial>

        <pose>0 0 0 0 0 0</pose>

        <mass>20000</mass>

        <inertia>

          <ixx>33333</ixx>

          <ixy>0.0</ixy>

          <ixz>0.0</ixz>

          <iyy>173333</iyy>

          <iyz>0.0</iyz>

          <izz>193333</izz>

        </inertia>

    </inertial>
```

## 2. 導入投擲程式

* 可於交付之資料夾中找到"spawn_model.py"之程式，並將該程式複製並於路徑:drone_ws/src/YCH_drone/src/python 位置進行貼上動作。

* 接著，可開啟"spawn_model.py"程式內容，為求正確定義系統中無人機之名稱與位置，以及定義投擲之煙霧彈位置，需將程式碼進行相應的修改。下方程式碼為"spawn_model.py"之段落內容，可以看到"model_name = 'iris'"，其中iris為無人機名稱，請查看gazebo介面左側model下之無人機模型名稱是否為該名稱，若否，請依照名稱進行修改，舉例應為:model_name = 'XXXX'。
* 接著進行煙霧彈模型定義位置修改，可以看到程式碼中"model_file = '/home/gaya/drone_ws/src/asv_wave_sim-gazebo9/asv_wave_sim_gazebo/models/smoke/model.sdf'"這段內容，其中之位置請填上上述煙霧彈模型之完整位置，舉例應為:XXX/XXX/drone_ws/src/asv_wave_sim-gazebo9/asv_wave_sim_gazebo/models/smoke/model.sdf。

* 另外，我們可以修改煙霧彈模型的投放位置來達到需求之樣式，可在程式碼中看到"smoke_pose.position.z = model_pose.position.z - 0.3"，其中之數值-0.3意思為，於無人機下方0.3單位進行投放，可根據需要之位置進行修改，舉例:smoke_pose.position.z = model_pose.position.z - xx，即為於無人機下方xx單位進行投擲。

```py=
...
def spawn_smoke(self):

        model_name = 'iris'  

        model_pose = self.get_model_pose(model_name)

 

        if model_pose is not None:

            smoke_name = 'smoke{}'.format(self.smoke_count)

            self.smoke_count += 1

 

            model_file = '/home/gaya/drone_ws/src/asv_wave_sim-gazebo9/asv_wave_sim_gazebo/models/smoke/model.sdf'

 

            with open(model_file, 'r') as f:

                model_xml = f.read()

 

            smoke_pose = Pose()

            smoke_pose.position.x = model_pose.position.x

            smoke_pose.position.y = model_pose.position.y

            smoke_pose.position.z = model_pose.position.z - 0.3

 

            spawn_model_proxy = rospy.ServiceProxy('/gazebo/spawn_sdf_model', SpawnModel)

            spawn_model_proxy(smoke_name, model_xml, "", smoke_pose, "world")

 

        else:

            rospy.logerr("Failed to get pose of {}. Smoke spawning aborted.".format(model_name))
...
```



## 3. 執行結果
* 請先開啟專案內容，並另外開啟一個terminal，輸入以下指令:cd XXX/drone_ws/src/YCH_drone/src/python，此為絕對目錄位置，請依照確切的位置內進行輸入，輸入後按下Enter鍵，接著輸入指令:python spawn_model.py，並可看到以下按鈕顯示於畫面中，此時請將無人機進行升空，於確定位置後再按下"Drop"鍵，即可看到煙霧彈進行投擲動作。

![image](https://github.com/gaya7502/object_spawn/blob/main/object_spawn0.png)

![image](https://github.com/gaya7502/object_spawn/blob/main/object_spawn1.png)
