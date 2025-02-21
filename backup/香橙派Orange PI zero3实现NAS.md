
### 概括：

- 香橙派+CasaOS+移动硬盘来实现内网访问存储
- Cpolar进行内网穿透实现外网访问

### 烧录香橙派系统系统

1. 硬件准备：

   1. 香橙派zero3
   2. 读卡器、sd卡（我用的32G）
   3. typeC线（香橙派电源）
   4. 外接显示器+转接头（香橙派是Micro HDMI口）
   5. 键盘

2. 烧录系统

   1. sd卡+读卡器插电脑
   2. 官网下载镜像：打开官网-服务于下载-下载-搜索zero3-下拉选择下载ubuntu镜像
   3. 下载完解压出img后缀镜像文件，用烧录软件ether烧录系统到sd卡中，烧录完成后将sd卡插到香橙派上
   4. 给香橙派插上电源、显示器、键盘，电源灯亮即可开机，屏幕显示OrangePI即为烧录成功

3. 配置系统

   1. 配置网络（若用网线则可忽略）
      1. 命令行输入sudo su获取root权限，密码默认为orangepi
      2. 输入orangepi-config进入配置界面，方向键进入network，再进入WIFI，输入密码连接Wifi
   2. 设置中国时区
      1. 返回到配置界面主菜单，选择Personal-Timezone-Asia-shanghai
   3. 获取Wifi分配给香橙派的ip地址
      1. 返回到命令行，输入ip address，输入的3：inet后边的即ip地址（我的是192.168.18.104）
      2. 现在就不需要显示器和键盘了
   4. 用自己的电脑ssh连接香橙派
      1. 使用putty连接：用户名为root，密码默认为orangepi，地址为刚刚获取的ip地址
      2. 能看到此页面即配置完成：

![Image](https://github.com/user-attachments/assets/b3480da6-a435-4206-b9e6-483423ab192c)

   ### 部署CasaOS

   1. 登录casaos的官网，下拉
   2. 找到`curl -fsSL https://get.casaos.io | sudo bash`，复制到香橙派的命令行，耐心等待部署
   3. 在同一个wifi下输入刚才获取的ip地址即可内网访问部署在香橙派上的CasaOS了，根据提示注册一个casaOS的账号
   4. 进入以下页面即部署完成（可以通过下方的Files进行文件存储与下载，若需扩大储存可外插移动硬盘）

![Image](https://github.com/user-attachments/assets/0f7d18cf-f07a-421f-bccb-28e53d23cc4e)

   ### Cpolar内网穿透

   1. 下载Cpolar

      ```
      #在香橙派命令行输入：
      curl -L https://www.cpolar.com/static/downloads/install-release-cpolar.sh | sudo bash		#下载cpolar
      
      cpolar version	#查看Cpolar是否下载成功
      
      ```

      

   2. token认证

      1. 登录cpolar官网，注册，登录

      2. 点击左侧的认证，复制自己的token

![Image](https://github.com/user-attachments/assets/1416e096-032f-45f8-81f3-6bccf827bd9f)

      3. 命令行中输入`cpolar authtoken abcd`，abcd为刚才复制的token

      4. 简单的穿透测试

         ```
         cpolar http 8080		#可以看到成功生成了两个指向本机8080端口的随机公网地址
         ```

   3. 向香橙派系统中添加cpolar服务

      ```
      systemctl enable cpolar
      systemctl start cpolar		#开启服务
      systemctl status cpolar		#查看服务
      ```

      
![Image](https://github.com/user-attachments/assets/f35dbf58-91f1-4123-9025-89f2cfb6db04)

4. 创建外网连接的地址

   1. 浏览器输入： http://香橙派的局域网ip:9200 

   2. 创建隧道

![Image](https://github.com/user-attachments/assets/37ca4ab4-6b99-4823-aa0b-41ca75015af9)

   3. 在状态-在线隧道列表中可以看到自己创建的隧道和公网地址

      
![Image](https://github.com/user-attachments/assets/3407bcaa-cc68-4434-b8c8-c1ba06c2604b)

5. 创建的是免费隧道，公网地址24小时内会随机变化，固定的地址需要付费升级







