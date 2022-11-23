# 1.与processing类似的新工具  
NodeBox是创建二维图形和可视化的应用程序，用Python语言编写，可以帮助用户以便捷的方式创建和生成设计，还能帮助用户将数据进行可视化处理。NodeBox内置多种绘画工具支持自定义字体的颜色、大小和形状,还有各种矢量图、剪贴画等, 它还支持导入各种格式的数据文件，比如Excel等。用户通过添加节点和更改选定节点的端口值进行可视化编程，从而创建和生成设计。  
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211231836647.png)  
# 2.Processing demo 
### 目标 
图片按照像素色块的方式分散开来，像素的明度越高，偏移程度越高。鼠标越往右移动，分散程度越大。 
### 代码 
```
PImage img;
int cellsize=4;
int cols,rows;

void setup()
{
    size(564, 675,P3D);//3d渲染器
    img=loadImage("pic.jpg");
    cols=width/cellsize;
    rows=height/cellsize;
}

void draw()
{
    background(255);
    img.loadPixels();//加载像素
    
    for(int i=0;i<cols;i++)
    {
        for(int j=0;j<rows;j++)
        {
            int x=i*cellsize+cellsize/2;
            int y=j*cellsize+cellsize/2;
            
            int loc=x+y*width; // 获取该像素点在序列中的位置
            color c=img.pixels[loc];//获取该像素点的颜色
            
            float z=map(brightness(img.pixels[loc]),0,255,0,mouseX);//重新构造映射，把明度和鼠标x坐标关联
            
            pushMatrix();
            translate(x,y,z);//平移
            fill(c);
            noStroke();
            rectMode(CENTER);
            rect(0,0,cellsize,cellsize);//画方块
            popMatrix();
        }
    }
}
```  
### 效果 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211231841439.gif) 
# 3.Processing与Arduino通信
### 目标 
点击按钮圆圈变为绿色，松开按钮圆圈变为红色。
### 代码 
+ Processing 
``` 
import processing.serial.*;
Serial port;
void setup()
{
 size(300,300);
  ellipse(100,100,100,100);
//
   port=new Serial(this,"COM3",9600); //Arduino's com
}
void draw(){
  // fill(0,255,0);
    while(port.available()>0)  //监听端口          
    {
        char inByte =port.readChar();//读取字节
        println(inByte);             //显示接收到的字节
        switch(inByte)
        {
            case 'a':                    //当接收的字母为a时填充红色
                    fill(0,255,0);
                    ellipse(100,100,100,100);//重新绘圆
                    break;
            case 'b':
                    fill(255,0,0);         //当接收到的字母为b时填充绿色
                    ellipse(100,100,100,100);
            default:break;
        }
    }
}
``` 
+ Arduino
```
boolean button; //定义布尔类型变量
void setup() 
{
    button =false;//初始值为false
    pinMode(8,INPUT);//定义8引脚为输入模式
    Serial.begin(9600);//设置比特率为9600bps
}
void loop()
{
    button =digitalRead(8);//读取第8引脚
    if(button)
    {
        Serial.write("a");//读到高电平发送‘a’
    }
    else
    {
        Serial.write("b");//读到低电平发送‘b’
    }
}  
```  
### 效果 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211231851435.gif)