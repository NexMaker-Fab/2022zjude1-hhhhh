# Arduino Output
## LCD1602
LCD1602, also known as 1602 character liquid crystal, is a kind of dot-matrix liquid crystal module specially used to display letters, numbers and symbols, which can display 16X2, namely 32 characters at the same time. 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211261205609.png) 
VL(V0) : LCD contrast adjustment end, used to adjust the display contrast, generally connected. 

RS: Data/command selection. High level indicates data, low level indicates command. 

RW: Read/write selection. High level for read, low level for write.Normally we write data for display, so this pin is grounded. 

EN: Enables signals for data/command read and write. 

D0-D7: two-way data terminal. It can be operated with 8 data lines in parallel or 4 data lines in serial. 

The first, fifth and 16th pins of LCD1602 are connected to the development board GND; The second and 15th pins of LCD1602 are connected to the development board 5V; The 4th,6th, 11th, 12th, 13th and 14th of LCD1602 are respectively connected to the digital pins 7, 6, 5, 4, 3 and 2 of the development board; The pins at both ends of the potentiometer are connected to 5V and GND respectively, and the middle pin is connected to the 3rd pin of LCD1602. 
## writing diagram 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211261217952.png) 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211261218845.png) 
## code 
```
int LCD1602_RS = 7;
int LCD1602_EN = 6;
int DB[4] = { 2, 3, 4, 5};

/*
 * LCD写命令
 */
void LCD_Command_Write(int command)
{
  int i, temp;
  digitalWrite( LCD1602_RS, LOW);
  digitalWrite( LCD1602_EN, LOW);

  temp = command & 0xf0;
  for (i = DB[0]; i <= 5; i++)
  {
    digitalWrite(i, temp & 0x80);
    temp <<= 1;
  }

  digitalWrite( LCD1602_EN, HIGH);
  delayMicroseconds(1);
  digitalWrite( LCD1602_EN, LOW);

  temp = (command & 0x0f) << 4;
  for (i = DB[0]; i <= 5; i++)
  {
    digitalWrite(i, temp & 0x80);
    temp <<= 1;
  }

  digitalWrite( LCD1602_EN, HIGH);
  delayMicroseconds(1);
  digitalWrite( LCD1602_EN, LOW);
}

/*
 * LCD写数据
 */
void LCD_Data_Write(int dat)
{
  int i = 0, temp;
  digitalWrite( LCD1602_RS, HIGH);
  digitalWrite( LCD1602_EN, LOW);

  temp = dat & 0xf0;
  for (i = DB[0]; i <= 5; i++)
  {
    digitalWrite(i, temp & 0x80);
    temp <<= 1;
  }

  digitalWrite( LCD1602_EN, HIGH);
  delayMicroseconds(1);
  digitalWrite( LCD1602_EN, LOW);

  temp = (dat & 0x0f) << 4;
  for (i = DB[0]; i <= 5; i++)
  {
    digitalWrite(i, temp & 0x80);
    temp <<= 1;
  }

  digitalWrite( LCD1602_EN, HIGH);
  delayMicroseconds(1);
  digitalWrite( LCD1602_EN, LOW);
}

/*
 * LCD设置光标位置
 */
void LCD_SET_XY( int x, int y )
{
  int address;
  if (y == 0)    address = 0x80 + x;
  else          address = 0xC0 + x;
  LCD_Command_Write(address);
}

/*
 * LCD写一个字符
 */
void LCD_Write_Char( int x, int y, int dat)
{
  LCD_SET_XY( x, y );
  LCD_Data_Write(dat);
}

/*
 * LCD写字符串
 */
void LCD_Write_String(int X, int Y, char *s)
{
  LCD_SET_XY( X, Y );    //设置地址
  while (*s)             //写字符串
  {
    LCD_Data_Write(*s);
    s ++;
  }
}

void setup (void)
{
  int i = 0;
  for (i = 2; i <= 7; i++)
  {
    pinMode(i, OUTPUT);
  }
  delay(100);
  LCD_Command_Write(0x28);//显示模式设置4线 2行 5x7
  delay(50);
  LCD_Command_Write(0x06);//显示光标移动设置
  delay(50);
  LCD_Command_Write(0x0c);//显示开及光标设置
  delay(50);
  LCD_Command_Write(0x80);//设置数据地址指针
  delay(50);
  LCD_Command_Write(0x01);//显示清屏
  delay(50);

}

void loop (void)
{
  LCD_Write_String(2, 0, "Hello World!");

``` 
# Arduino Input and Output
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211261220763.png) 
## main
```
String inputString = "";         // a string to hold incoming data
boolean stringComplete = false;  // whether the string is complete

#include "Adafruit_NeoPixel.h"
#ifdef __AVR__
#include <avr/power.h>
#endif
#define PIN             4
#define NUMPIXELS      13
Adafruit_NeoPixel pixels = Adafruit_NeoPixel(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

#include <SoftwareSerial.h>
#include "DFPlayer_Mini_Mp3.h"
SoftwareSerial mySerialmp3(2, 8); // RX, TX
#define KEY A0

#include <Servo.h> //引入lib
Servo myservo1;  // 创建一个伺服电机对象
Servo myservo2;
Servo myservo3;

#include "RC522.h"
#include <SPI.h>

#define Relay A5

//4 bytes Serial number of card, the 5 bytes is verfiy bytes
unsigned char serNum[5];
int S;
unsigned char status;
unsigned char str[MAX_LEN];
int yanshi=1000;
void setup()
{
  pinMode(KEY, INPUT_PULLUP);
  myservo1.attach(5);  // 9号引脚输出电机控制信号
  myservo2.attach(6);
  myservo3.attach(7);
  myservo1.write(0);                     //仅能使用PWM引脚
  myservo2.write(0);
  myservo3.write(0);
  delay(1000);
  myservo1.write(90);
  myservo2.write(90);
  myservo3.write(90);

  Serial.begin(9600);
  attachInterrupt(1, key, RISING);

  Serial.print("Ilovemcu.taobao.com");
  pinMode(Relay, OUTPUT);
  SPI.begin();
  pinMode(chipSelectPin, OUTPUT); // Set digital pin 10 as OUTPUT to connect it to the RFID /ENABLE pin
  digitalWrite(chipSelectPin, LOW); // Activate the RFID reader
  pinMode(NRSTPD, OUTPUT); // Set digital Reset , Not Reset and Power-down

  MFRC522_Init();						//初始化RFID

  mySerialmp3.begin (9600);
  mp3_set_serial (mySerialmp3);  //set softwareSerial for DFPlayer-mini mp3 module
  delay(1);  //wait 1ms for mp3 module to set volume
  mp3_set_volume (20);
  delay(10);
  //  mp3_play(16);
  mp3palynum(111, yanshi);
  delay(300);
  mp3_play(104);//元
#if defined (__AVR_ATtiny85__)
  if (F_CPU == 16000000) clock_prescale_set(clock_div_1);
#endif
  pixels.begin();

  for (int i = 0; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(5, 5, 5)); // Moderately bright green color.
    pixels.show(); // This sends the updated pixel color to the hardware.
    delay(10);
  }
  delay(1000);
  for (int i = 0; i < NUMPIXELS; i++) {
    pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Moderately bright green color.
    pixels.show(); // This sends the updated pixel color to the hardware.
    delay(10);
  }
}

long timer1 = 0;
int xiangmu = 0;
int jine = 0;
void loop()
{
  if (millis() - timer1 > 200)
  {
    timer1 = millis();
    //Serial.println("sdf");
    if (xiangmu == 0)
    {
      readcar();

    }
    if (xiangmu == 1) //按头收款
    {
      if (digitalRead(KEY) == 1)
      {
        delay(10);
        if (digitalRead(KEY) == 1)
        {
          xiangmu = 0;
          Serial.println("按头收款");
          jine = jine + 100;
          mp3_play(109);//已收款100元
          for (int i = 0; i < NUMPIXELS; i++) {
            pixels.setPixelColor(i, pixels.Color(i, 0, 13)); // Moderately bright green color.
            pixels.show(); // This sends the updated pixel color to the hardware.
            delay(10);
          }
          delay(1000);
          myservo3.write(90);//恢复倾斜
          delay(1000);
          for (int i = 0; i < 3; i++)//翅膀扇动
          {
            myservo1.write(45);
            myservo2.write(130);
            delay(300);
            myservo1.write(90);
            myservo2.write(90);
            delay(300);
          }
          for (int i = 0; i < NUMPIXELS; i++) {
            pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Moderately bright green color.
            pixels.show(); // This sends the updated pixel color to the hardware.
            delay(10);
          }
        }
      }
    }
    if (xiangmu == 2) //摇摇
    {
      if (jine == 100)
        mp3_play(106);//100元哗啦声音
      else if (jine == 200)
        mp3_play(107);//200元哗啦声音
      else if (jine > 200)
        mp3_play(108);//300元哗啦声音
      delay(3000);
      mp3_play(103);//余额还有
      delay(2000);
      mp3palynum(jine, yanshi);
      delay(300);
      mp3_play(104);//元
      delay(1000);
      attachInterrupt(1, key, RISING);
      xiangmu = 0;
    }
  }
  serialEvent1();

}

void serialEvent1() {
  while (Serial.available()) {
    // get the new byte:
    char inChar = (char)Serial.read();
    Serial.write(inChar);
    // add it to the inputString:
    inputString += inChar;
    // if the incoming character is a newline, set a flag
    // so the main loop can do something about it:
    if (inChar == '\n')
    {
      stringComplete = true;
    }

    if (stringComplete)
    {
      Serial.println(inputString);
      //      char *p;
      //      p=inputString.c_str();
      //angle=inputString.toInt();//string 转 int
      // r=inputString.substring(10).toInt();
      if (strstr(inputString.c_str(), "send=100") != NULL)
      {
        xiangmu = 1;
        mp3_play(100);//小鸡叫
        for (int i = 0; i < NUMPIXELS; i++) {
          pixels.setPixelColor(i, pixels.Color(13, i, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
          delay(10);
        }
        delay(1000);
        myservo3.write(180);//存钱罐向前倾斜
        delay(1000);
        for (int i = 0; i < 3; i++)//翅膀扇动
        {
          myservo1.write(45);
          myservo2.write(130);
          delay(300);
          myservo1.write(90);
          myservo2.write(90);
          delay(300);
        }

      }
      // clear the string:
      inputString = "";
      stringComplete = false;
    }
  }
}

void key()
{
  S++;
  if (S > 10)
  {
    S = 0;
    xiangmu = 2;
    detachInterrupt(1);
    Serial.println("摇一摇");
  }

}
void readcar()
{
  // Search card, return card types
  status = MFRC522_Request(PICC_REQIDL, str);
  if (status == MI_OK)      //读取到ID卡时候
  {

    //Prevent conflict, return the 4 bytes Serial number of the card
    status = MFRC522_Anticoll(str);

    if (status == MI_OK)
    {
      memcpy(serNum, str, 5);
      Serial.print("ID:");
      ShowCardID(serNum);
      unsigned char* id = serNum;
      if ( id[0] == 0xb3 && id[1] == 0xce && id[2] == 0x26 && id[3] == 0xed )
      {
        Serial.println("5元!");
        if (jine > 5)
        {
          jine = jine - 5;
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 13, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
          mp3_play(105);//金属声音
          delay(3000);
          mp3_play(100);//小鸡叫
          delay(2000);
          mp3_play(101);//消费
          delay(2000);
          mp3palynum(5, yanshi);
          delay(300);
          mp3_play(104);//元
          delay(1000);
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
        }
        else
        {
          Serial.println("余额不足");
        }
      }
      else if ( id[0] == 0x33 && id[1] == 0x9d && id[2] == 0x4f && id[3] == 0xed )
      {
        Serial.println("10元!");
        if (jine > 10)
        {
          jine = jine - 10;
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 13, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
          mp3_play(105);//金属声音
          delay(3000);
          mp3_play(100);//小鸡叫
          delay(2000);
          mp3_play(101);//消费
          delay(2000);
          mp3palynum(10, yanshi);
          delay(300);
          mp3_play(104);//元
          delay(1000);
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
        }
        else
        {
          Serial.println("余额不足");
        }
      }
      if ( id[0] == 0x73 && id[1] == 0xce && id[2] == 0x26 && id[3] == 0xed )
      {
        Serial.println("15元!");
        if (jine > 15)
        {
          jine = jine - 15;
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 13, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
          mp3_play(105);//金属声音
          delay(3000);
          mp3_play(100);//小鸡叫
          delay(2000);
          mp3_play(101);//消费
          delay(2000);
          mp3palynum(15, yanshi);
          delay(300);
          mp3_play(104);//元
          delay(1000);
          for (int i = 0; i < NUMPIXELS; i++)
            pixels.setPixelColor(i, pixels.Color(0, 0, 0)); // Moderately bright green color.
          pixels.show(); // This sends the updated pixel color to the hardware.
        }
        else
        {
          Serial.println("余额不足");
        }
      }
      else
      {
        Serial.println("Stranger!");
      }

    }

  }

  MFRC522_Halt(); //command the card into sleep mode
}
void mp3palynum(int num, int yushu)
{
  if (num % 10000 / 1000 > 0)
  {
    mp3_play(num % 10000 / 1000);
    delay(yushu);
    mp3_play(13);
    delay(yushu);
  }
  if (num % 1000 / 100 > 0)
  {
    mp3_play(num % 1000 / 100);
    delay(yushu);
    mp3_play(11);
    delay(yushu );
  }
  if (num % 100 / 10 > 0)
  {
    mp3_play(num % 100 / 10);
    delay(yushu);
    mp3_play(10);
    delay(yushu );
  }
  if (num % 10 / 1 > 0)
  {
    mp3_play(num % 10 / 1);
    delay(yushu);
  }
}

``` 
## Ws2812 light code
```
#include "Adafruit_NeoPixel.h"

// Constructor when length, pin and type are known at compile-time:
Adafruit_NeoPixel::Adafruit_NeoPixel(uint16_t n, uint8_t p, neoPixelType t) :
  brightness(0), pixels(NULL), endTime(0), begun(false)
{
  updateType(t);
  updateLength(n);
  setPin(p);
}
``` 
## DFplayer mini code
```
#include <Arduino.h>
#include <SoftwareSerial.h>
//#include  "DFPlayer_Mini_Mp3.h"

extern uint8_t send_buf[10];
extern uint8_t recv_buf[10];

static void(*send_func)() = NULL;
static HardwareSerial * _hardware_serial = NULL;
static SoftwareSerial * _software_serial = NULL;
static boolean is_reply = false;

//
void mp3_set_reply (boolean state) {
	is_reply = state;
	send_buf[4] = is_reply;
}

//
static void fill_uint16_bigend (uint8_t *thebuf, uint16_t data) {
	*thebuf =	(uint8_t)(data>>8);
	*(thebuf+1) =	(uint8_t)data;
}


//calc checksum (1~6 byte)
uint16_t mp3_get_checksum (uint8_t *thebuf) {
	uint16_t sum = 0;
	for (int i=1; i<7; i++) {
		sum += thebuf[i];
	}
	return -sum;
}

//fill checksum to send_buf (7~8 byte)
void mp3_fill_checksum () {
	uint16_t checksum = mp3_get_checksum (send_buf);
	fill_uint16_bigend (send_buf+7, checksum);
}

//
void h_send_func () {
	for (int i=0; i<10; i++) {
		_hardware_serial->write (send_buf[i]);
	}
}

//
void s_send_func () {
	for (int i=0; i<10; i++) {
		_software_serial->write (send_buf[i]);
	}
}

//
//void mp3_set_serial (HardwareSerial *theSerial) {
void mp3_set_serial (HardwareSerial &theSerial) {
	_hardware_serial = &theSerial;
	send_func = h_send_func;
}

//
void mp3_set_serial (SoftwareSerial &theSerial) {
	_software_serial = &theSerial;
	send_func = s_send_func;
}

//
void mp3_send_cmd (uint8_t cmd, uint16_t arg) {
	send_buf[3] = cmd;
	fill_uint16_bigend ((send_buf+5), arg);
	mp3_fill_checksum ();
	send_func ();
}

//
void mp3_send_cmd (uint8_t cmd) {
	send_buf[3] = cmd;
	fill_uint16_bigend ((send_buf+5), 0);
	mp3_fill_checksum ();
	send_func ();
}


//
void mp3_play_physical (uint16_t num) {
	mp3_send_cmd (0x03, num);
}

//
void mp3_play_physical () {
	mp3_send_cmd (0x03);
}

//
void mp3_next () {
	mp3_send_cmd (0x01);
}

//
void mp3_prev () {
	mp3_send_cmd (0x02);
}

//0x06 set volume 0-30
void mp3_set_volume (uint16_t volume) {
	mp3_send_cmd (0x06, volume);
}

//0x07 set EQ0/1/2/3/4/5    Normal/Pop/Rock/Jazz/Classic/Bass
void mp3_set_EQ (uint16_t eq) {
	mp3_send_cmd (0x07, eq);
}

//0x09 set device 1/2/3/4/5 U/SD/AUX/SLEEP/FLASH
void mp3_set_device (uint16_t device) {
	mp3_send_cmd (0x09, device);
}
```
## RC522 code 
```
#include "RC522.h"
const int chipSelectPin = 10;
const int NRSTPD =9;
/*
 * Function��ShowCardID
 * Description��Show Card ID
 * Input parameter��ID string
 * Return��Null
 */
void ShowCardID(uchar *id)
{
    int IDlen=4;
    for(int i=0; i<IDlen; i++){
        Serial.print(0x0F & (id[i]>>4), HEX);
        Serial.print(0x0F & id[i],HEX);
    }
    Serial.println("");
}

/*
 * Function��ShowCardType
 * Description��Show Card type
 * Input parameter��Type string
 * Return��Null
 */
void ShowCardType(uchar* type)
{
    Serial.print("Card type: ");
    if(type[0]==0x04&&type[1]==0x00) 
        Serial.println("MFOne-S50");
    else if(type[0]==0x02&&type[1]==0x00)
        Serial.println("MFOne-S70");
    else if(type[0]==0x44&&type[1]==0x00)
        Serial.println("MF-UltraLight");
    else if(type[0]==0x08&&type[1]==0x00)
        Serial.println("MF-Pro");
    else if(type[0]==0x44&&type[1]==0x03)
        Serial.println("MF Desire");
    else
        Serial.println("Unknown");
}

/*
 * Function��Write_MFRC5200
 * Description��write a byte data into one register of MR RC522
 * Input parameter��addr--register address��val--the value that need to write in
 * Return��Null
 */
void Write_MFRC522(uchar addr, uchar val)
{
    digitalWrite(chipSelectPin, LOW);

    //address format��0XXXXXX0
    SPI.transfer((addr<<1)&0x7E); 
    SPI.transfer(val);
    
    digitalWrite(chipSelectPin, HIGH);
}


/*
 * Function��Read_MFRC522
 * Description��read a byte data into one register of MR RC522
 * Input parameter��addr--register address
 * Return��return the read value
 */
uchar Read_MFRC522(uchar addr)
{
    uchar val;

    digitalWrite(chipSelectPin, LOW);

    //address format��1XXXXXX0
    SPI.transfer(((addr<<1)&0x7E) | 0x80); 
    val =SPI.transfer(0x00);
    
    digitalWrite(chipSelectPin, HIGH);
    
    return val; 
}

/*
 * Function��SetBitMask
 * Description��set RC522 register bit
 * Input parameter��reg--register address;mask--value
 * Return��null
 */
void SetBitMask(uchar reg, uchar mask) 
{
    uchar tmp;
    tmp = Read_MFRC522(reg);
    Write_MFRC522(reg, tmp | mask); // set bit mask
}


/*
 * Function��ClearBitMask
 * Description��clear RC522 register bit
 * Input parameter��reg--register address;mask--value
 * Return��null
 */
void ClearBitMask(uchar reg, uchar mask) 
{
    uchar tmp;
    tmp = Read_MFRC522(reg);
    Write_MFRC522(reg, tmp & (~mask)); // clear bit mask
} 


/*
 * Function��AntennaOn
 * Description��Turn on antenna, every time turn on or shut down antenna need at least 1ms delay
 * Input parameter��null
 * Return��null
 */
void AntennaOn(void)
{
    uchar temp;

    temp = Read_MFRC522(TxControlReg);
    if (!(temp & 0x03))
    {
        SetBitMask(TxControlReg, 0x03);
    }
}


/*
 * Function��AntennaOff
 * Description��Turn off antenna, every time turn on or shut down antenna need at least 1ms delay
 * Input parameter��null
 * Return��null
 */
void AntennaOff(void)
{
    ClearBitMask(TxControlReg, 0x03);
}


/*
 * Function��ResetMFRC522
 * Description�� reset RC522
 * Input parameter��null
 * Return��null
 */
void MFRC522_Reset(void)
{
    Write_MFRC522(CommandReg, PCD_RESETPHASE);
}


/*
 * Function��InitMFRC522
 * Description��initilize RC522
 * Input parameter��null
 * Return��null
 */
void MFRC522_Init(void)
{
    digitalWrite(NRSTPD,HIGH);

    MFRC522_Reset();
         
    //Timer: TPrescaler*TreloadVal/6.78MHz = 24ms
    Write_MFRC522(TModeReg, 0x8D); //Tauto=1; f(Timer) = 6.78MHz/TPreScaler
    Write_MFRC522(TPrescalerReg, 0x3E); //TModeReg[3..0] + TPrescalerReg
    Write_MFRC522(TReloadRegL, 30); 
    Write_MFRC522(TReloadRegH, 0);
    
    Write_MFRC522(TxAutoReg, 0x40); //100%ASK
    Write_MFRC522(ModeReg, 0x3D); //CRC initilizate value 0x6363 ???

    //ClearBitMask(Status2Reg, 0x08); //MFCrypto1On=0
    //Write_MFRC522(RxSelReg, 0x86); //RxWait = RxSelReg[5..0]
    //Write_MFRC522(RFCfgReg, 0x7F); //RxGain = 48dB

    AntennaOn(); //turn on antenna
} 
```
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211261408624.png) 
## Introduce 
Parents can download the APP matching the piggy bank, register and log in, and pair the piggy bank with the Bluetooth module. The light band on the top of the piggy bank is green, indicating successful matching. When parents and children reach an agreement. Parents set the tasks to be completed by their children through the mobile phone. When the child completes the task, the parents give the child pocket money on the home page. At this time, the chicken's wings will quickly flap three times, and the top light band will show a gradual change of color from red to yellow from front to back, accompanied by the chick leaning forward. Multiple channels of information make the child feel that an allowance has arrived. When the child touches the chicken's head, the piggy bank confirms the payment and makes voice and acousto-visual prompts. The piggy bank leans back, restoring its swagger and self-satisfaction. (Internal steering engine drive wing rotation structure diagram). When the child has achieved certain savings goals, accompanied by the parents, the child carries the piggy bank to complete the independent shopping. 
Parents can also check the income and expenditure of their children's bills through the smart app. The growth of chickens in the home page is also directly related to the income and expenditure of the piggy bank. From another perspective, children can intuitively feel that the piggy bank is becoming "bigger". With the joint participation of families, parents can learn financial management courses with their children and jointly improve their financial knowledge. 
#  Prototyping 
This is a schematic diagram of Arduino nano based on DXP software. In the motherboard selection, select nano+nano extension to reduce the overall hardware size. 

The HC05 Bluetooth module communicates with the motherboard through the serial port, and can use the Android SPP Bluetooth serial assistant. Simulate the operation of parents handing out pocket money. 

Touch switch is used to realize a series of interactive actions such as children touching the chicken head to confirm payment. 

Three 9G steering engines to achieve wing and body movement changes. 

MP3 player module and speaker, built-in memory card, through the memory card into the related sound effects, sound playback through the speaker. 

Vibration sensor. Simulate the function of children shaking the piggy bank 

WS2812 light belt, realize the light change of chicken head 

RC522 RFID identification module, simulate the situation of children shopping

