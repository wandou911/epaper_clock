# epaper_clock
树莓派打造电子油墨屏时钟
本文参考[空华叔:全球首款配备电子纸屏幕的抽纸盒](https://bluehua.org/2016/04/25/2311.html)制作的电子时钟，效果图：



 

内部图：



 

 

制作方法

所用的硬件
1、树莓派3
2、微雪4.3寸串口电子墨水屏
3、DHT22温湿度传感模块

硬件连接：
电子屏
|屏幕|树莓派|物理接口|
|-|:-:|-:|
|DIN|TX(GPIO14)|8|
|DOUT|RX(GPIO15)|10|
|GND|GND|6|
|VCC|3V|1|

温湿度传感器

DHT22|树莓派|物理接口
-|:-:|-:|
DOUT|1-Wire(BCM4)|7
GND|GND|9
VCC|3V|17


关于GND表示的是电源的负极，VCC表示电源正极

DHT22 DOUT引脚也可以接到其他gpio脚上，不过要相应的修改home_air_sensor.py中read_retry第二个参数

在树莓派和屏幕接线根本不知道怎么接，请看下图对应关系：



 树莓派3BGPIO图：

![树莓派3BGPIO图](http://images2015.cnblogs.com/blog/1044995/201704/1044995-20170401110742242-471509557.jpg)


树莓派GPIO接口writing Pi, BCM, Board三种模式对应

![树莓派GPIO接口模式](http://images2017.cnblogs.com/blog/1044995/201712/1044995-20171217213946249-1431021000.png)

准备软件环境
禽兽，放开那个串口。。
树莓派的串口默认是用于linux串口终端登录用的，如果要通过串口控制屏幕，就需要把它解放出来～

树莓派3的串口BUG
在释放串口之前，我们要先解决一下树莓派3的BUG（如果用1,2代请忽略这一步）树莓派3的硬件串口被分配分配给了蓝牙模块，而GPIO14和GPIO15的串口是由内核模拟的，不稳定（可以说基本不能用)，所以首先要把GPIO14和GPIO15改成硬件驱动

第一步 确保SD卡刷了最新的raspbian jessie镜像
第二步 系统启动，并连接了网络
第三步 执行
```
sudo apt-get update
sudo apt-get upgrade
``` 

第四步 编辑 /boot/config.txt 添加一行
```
dtoverlay=pi3-miniuart-bt
```

 

最后 禁用自带蓝牙
```
sudo systemctl disable hciuart
``` 

释放串口


编辑 /boot/cmdline.txt，默认是下面这样
```
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait
```

或者这样
```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 kgdboc=serial0,115200 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
``` 

把console=ttyAMA0,console=serial0,kgdboc=***这两个参数删掉 变成下面这样
```
dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
```

之后sudo reboot重启系统 串口就可以正常使用了

安装软件依赖
```
1 sudo apt-get install python-requests python-lxml python-serial git build-essential python-dev
2 git clone https://github.com/adafruit/Adafruit_Python_DHT.git
3 cd Adafruit_Python_DHT
4 sudo python ./setup.py install
```
 

准备串口屏幕的图片和字体资源
这个串口屏是通过TF卡加载字体和图片资源的（好坑爹的设计。。），所以你需要准备一张TF卡，格式化为 FAT32 文件系统，分配单元大小选择 4096 字节，然后把tf_card文件夹中的文件全部copy到TF卡根目录，并把TF卡查到屏幕的卡槽里。串口屏的更多资料见：https://github.com/yy502/ePaperDisplay/blob/master/docs/4.3inch-e-Paper-UserManual-CN.pdf

终于可以运行了～～

在运行之前先编辑一下weather_time_render.py，找到下面2行，把注释取消掉，运行时会把屏幕TF卡中的文件加载到屏幕自带的NandFlash中，之后就不需要插TF卡了～～
```
1 # screen.load_pic()
2 # time.sleep(5)
```

运行脚本
```
1 sudo ./home_air_sensor.py
2 ./weather_fetcher.py
3 ./weather_time_render.py
```


 

没有特殊情况，屏幕将和成品显示同样的画面，第一次运行之后就可以把加载图片的2句代码再次注释掉了

 
自动定时任务，就可以一直运行：
```
crontab epaper_clock.cron
```
查看定时任务：
```
crontab -l
```

本文是参考：github上两篇博客实现

参考博客：1 https://bluehua.org/2016/04/25/2311.html

　　　　　2 http://blog.csdn.net/zhujunxxxxx/article/details/51944584

github：  1 https://github.com/emptyhua/epaper_clock

　　　　  2 https://github.com/zhujunxxxxx/weather_clock

　　　　  3 https://github.com/yy502/ePaperDisplay
