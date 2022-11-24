# How to make 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241057411.png) 
<center>总体框架</center> 

![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241232653.png)

用Arduino做实时的数据采集和控制，树莓派做上位机负责管理系统，Arduino 做下位机负责控制其他硬件，实现优势互补。 

### 故事触发模块 
+ #### 采用模块
<table><tr><td bgcolor=DarkSeaGreen>RC522+rfid标签</td></tr></table>

+ #### 技术原理与方案 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241133461.png) 
<center>接线方式（需要焊接）</center>  

### 发声讲故事模块
+ #### 采用模块 
<table><tr><td bgcolor=DarkSeaGreen>DFPlaye Mini MP3模块</td></tr></table> 

DFPlayer Mini MP3模块，可以直接接驳扬声器。模块配合供电电池、扬声器、按键可以单独使用，也可以通过串口控制，作为Arduino UNO或者是任何有串口的单片机的一个模块。模块本身集成了MP3、WAV、WMA的硬解码。同时软件支持TF卡驱动，支持FAT16、FAT32文件系统。通过简单的串口指令即可完成播放指定的音乐，以及如何播放音乐等功能。 

![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241116867.png) 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241118700.png) 
<center>引脚说明</center> 

+ #### 技术原理与方案
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241120386.png)
<center>接线方式</center> 

![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241131218.png) 
<center>方案</center>  

### 驱动循迹模块
+ #### 驱动模块 
<table><tr><td bgcolor=DarkSeaGreen>电机驱动模块 L298N</td></tr></table> 

![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241136236.png)  
L298N是一个内部有两个H桥的驱动芯片，这样电机的运转只需要用三个信号控制：两个方向信号和一个使能信号。(输入的电压不可超过它的额定电压）
L298N芯片的工作电压需要两路：  
第一路：输出供给电机回路的工作电源  
第二路：输入逻辑控制回路电源5V（电源出/入）

+12V：该引脚接的电压是驱动模块所能输出给电机的最大电压，一般 直接接电池。12V是由L298N芯片所能接受最大电压而定，一般介入5~12V电压。在此我们接入的电压为两节18650串联的电压，即3.7+3.7=7.4V； 

GND： 电源的负极，同时要保证Arduino开发板，驱动模块等所有模块的GND连在一起才可以正常工作。 

+5V： L298N模块内含稳压电路，在模块内部将"+12V"引脚输入的电压转化为可供开发板使用的+5V电压，一般将次输出接入到开发板为开发板供电。
L298N有两路输出，所以可以控制小车前进、后退、转弯，其中： 

ENA： 代表第一路输出的电压大小。驱动模块输出电压越高，电机转速越快。 
1.当其输入为0V的时候，驱动模块输出对第一路电机输出电压为0V； 
2.当其输入为3.3V的时候，驱动模块对第一路电机输出电压为"+12V"引脚的输入电压。 
3.由于ENA输入电压的高低控制驱动对电机的输出电压，因此当我们需要对小车运动速度进行控制的时候，一般通过PWM对"ENA"引脚进行控制。 

IN1/IN2：这两个引脚控制电机正反转方向。 

OUT1/OUT2：这两个引脚分别接电机的两极。 

ENB，IN3/IN4，OUT3/OUT4引脚控制第二路输出，与上述ENB，IN3/IN4，OUT3/OUT4功能相似

+ #### 循迹模块 
循迹原理： 利用红外线对于不同颜色具有不同的反射性质的特点。行驶过程中传感器的红外发射二极管不断发射红外光，当红外光遇到白色地面时发生漫反射，红外对管接收管接收反射光；如果遇到黑线则红外光被吸收，则红外管接收不到信号。
红外对管采集回来的信号通过2路循迹传感器模块里面的LM339比较器后输出高或低电平，从而实现信号的检测。 
### 摄像头识别模块
+ #### 编程开发环境、语言 
<table><tr><td bgcolor=DarkSeaGreen>linux-python-opencv</td></tr></table> 

+ #### “监听”，识别是否贴贴纸，以判断是否前进去识别  
此处参考检测物体移动/动作检测的方法
>##### 1、摄像头识别
>[参考方法](https://blog.csdn.net/Wangguang_/article/details/89875170?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-16-89875170-blog-65440952.pc_relevant_aa2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-16-89875170-blog-65440952.pc_relevant_aa2&utm_relevant_index=17) 
>结果如图： 
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241219937.png) 
>
>##### 2、红外传感器
>[参考方法1](http://wjhsh.net/jingxinbk-p-12409029.html)
>[参考方法2](https://www.eda365.com/portal.php?mod=view&aid=168601)
 
+ #### 识别颜色  
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241247965.png) 
><center>流程</center> 
>
>[在opencv框架下使用C++实现物料颜色识别1](https://blog.csdn.net/kilotwo/article/details/86744741?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-86744741-blog-127303956.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-86744741-blog-127303956.pc_relevant_default&utm_relevant_index=19) 
>
>[在opencv框架下使用C++实现物料颜色识别2](https://it1995.blog.csdn.net/article/details/83056346?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-83056346-blog-88137314.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-83056346-blog-88137314.pc_relevant_aa&utm_relevant_index=5) 
>
>[在opencv框架下使用python实现物料颜色识别](https://blog.csdn.net/m0_52250472/article/details/120425725?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-120425725-blog-106411746.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-120425725-blog-106411746.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=9)

+ #### 识别手写体 
> ##### 1、用OpenCV自带的神经网络
>[题主选用的BP算法进行训练，准确咧90%](https://blog.csdn.net/sheng_ai/article/details/23956919?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-23956919-blog-59727212.pc_relevant_recovery_v2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-23956919-blog-59727212.pc_relevant_recovery_v2&utm_relevant_index=3)  
>
>训练函数如下图 
>![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241303667.png)
> ##### 2、KNN
>[链接(opencv官方给的手写体数字识别的例子也用的KNN)](https://blog.csdn.net/zzlwl/article/details/125216262)
> ##### 3、其他
>[演示：基于Python+OpenCV+TensorFlow手写汉字识别系统](https://www.bilibili.com/video/av216101007/) 
>
>[使用C++结合Opencv库实现简易汉字识别](https://blog.csdn.net/weixin_44297922/article/details/121496280)


### 通信模块 
1.将树莓派与arduino通过usb线进行连接  
2.在树莓派终端输入 ls /dev/tty*查看两者连接端口的名字。查看有没有ttyACM0 这个文件(注只有在两个硬件USB互连的情况下才会有这个。如果两者没有连接是不会有的) 最新的系统一般都会自动生成  
3.编写树莓派与arduino通信代码

