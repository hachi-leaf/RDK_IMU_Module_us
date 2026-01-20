# <center>**RDK_IMU_Module_us**</center>
## <center>———— *Driver RDK IMU Module in User Space* ————</center>
&emsp;基于用户空间的RDK IMU Module软件包，完全不依赖iio相关驱动，仅使用linux标准通用I2C与SPI驱动<br>
&emsp;**Version: 1.0.0**

***

## **一、编译说明**

### **1.1.依赖说明**

项目分为两个部分：<br>

1. **rdk_imu_module_data:** 核心部分，用于调用Linux通用SPI与I2C驱动与IMU通信，并进行数据解析与时间同步，编译需`-lm`链接math<br>

2. **rdk_imu_module_gpio:** 次要部分，用于驱动**RDK IMU Module**载板上的GPIO设备（LED、蜂鸣器和1-wire温度传感器），依赖**RDK wiringPi**，需要按照以下教程进行**RDK wiringPi**的安装：[在旭日X3派上移植的 WiringPi](https://gitee.com/xin03liu/WiringPi)，并在编译时`-lwiringPi`链接**wiring Pi**<br>

- 不安装**RDK wiringPi**也可以使用**rdk_imu_module_data**的C部分内容，但不能使用**rdk_imu_module_gpio**和Python部分内容<br>

***

### **1.2.Makefile使用**

&emsp;项目目录下编写了Makefile，可以通过make快速编译项目，`cd /path/to/RDK_IMU_Module_us`来到项目目录，`make`或`make all`编译所有项目，或者：<br>

1. `make static`: 编译完整功能的`.a`静态链接库到`./lib`路径下<br>

2. `make static-Wo-gpio`: 仅编译**rdk_imu_module_data**部分内容的`.a`静态链接库到`./lib`路径下<br>

3. `make dynamic`: 编译完整功能的`.so`动态链接库到`./lib`路径下<br>

4. `make dynamic-Wo-gpio`: 仅编译**rdk_imu_module_data**部分内容的`.so`动态链接库到`./lib`路径下<br>

5. `make sample`: 编译完整功能的测试用例可执行文件到`./bin`路径下<br>

6. `make sample-Wo-gpio`: 仅编译**rdk_imu_module_data**部分内容的测试用例可执行文件到`./bin`路径下<br>

7. `make cython`: 编译Cython部分内容<br>

8. `make clean`: 清理所有内容<br>

***

## **二、Python-Sample快速体验**

### **2.1.环境准备**

#### **2.1.1.wiringPi**安装

&emsp;按照**1.1.依赖说明**中的链接安装`RDK wiringPi`，使用命令`gpio readall`检查，输出以下内容：<br>

```text
root@ubuntu:~/RDK_IMU_Module_us# gpio readall
 +-----+-----+-----------+--RDK X5--+-----------+-----+-----+
 | BCM | xPi |    Name   | Physical |   Name    | xPi | BCM |
 +-----+-----+-----------+----++----+-----------+-----+-----+
 |     |     |      3.3v |  1 || 2  | 5v        |     |     |
 |   2 | 390 |     SDA.5 |  3 || 4  | 5v        |     |     |
 |   3 | 389 |     SCL.5 |  5 || 6  | 0v        |     |     |
 |   4 | 420 | I2S1_MCLK |  7 || 8  | TxD.1     | 383 | 14  |
 |     |     |        0v |  9 || 10 | RxD.1     | 384 | 15  |
 |  17 | 380 |  GPIO. 17 | 11 || 12 | I2S1_BCLK | 421 | 18  |
 |  27 | 379 |  GPIO. 27 | 13 || 14 | 0v        |     |     |
 |  22 | 388 |  GPIO. 22 | 15 || 16 | GPIO. 23  | 382 | 23  |
 |     |     |      3.3v | 17 || 18 | GPIO. 24  | 402 | 24  |
 |  10 | 398 | SPI1_MOSI | 19 || 20 | 0v        |     |     |
 |   9 | 397 | SPI1_MISO | 21 || 22 | GPIO. 25  | 387 | 25  |
 |  11 | 395 | SPI1_SCLK | 23 || 24 | SPI1_CSN1 | 394 | 8   |
 |     |     |        0v | 25 || 26 | SPI1_CSN0 | 396 | 7   |
 |   0 | 355 |     SDA.0 | 27 || 28 | SCL.0     | 354 | 1   |
 |   5 | 399 |   GPIO. 5 | 29 || 30 | 0v        |     |     |
 |   6 | 400 |   GPIO. 6 | 31 || 32 | PWM6      | 356 | 12  |
 |  13 | 357 |      PWM7 | 33 || 34 | 0v        |     |     |
 |  19 | 422 | I2S1_LRCK | 35 || 36 | GPIO. 16  | 381 | 16  |
 |  26 | 401 |   GPIO.26 | 37 || 38 | I2S1_DIN  | 423 | 20  |
 |     |     |        0v | 39 || 40 | I2S1_DOUT | 424 | 21  |
 +-----+-----+-----------+----++----+-----------+-----+-----+
 | BCM | xPi |    Name   | Physical |   Name    | xPi | BCM |
 +-----+-----+-----------+--RDK X5--+-----------+-----+-----+
```

***

#### **2.1.2.动态链接库编译**

&emsp;项目自带一个编译好的、可用于python的`.so`动态链接库文件，位于`RDK_IMU_Module_us/Py_RDK_IMU/rdkimu.cpython-310-aarch64-linux-gnu.so`，如果确认`RDK wiringPi`已正确安装，可以直接运行sample<br>

&emsp;如果需要重新编译Cython动态链接库，回到Makefile目录下`make cython`即可<br>

***

### **2.2.sample执行**

&emsp;`python3 test_pyimu.py`执行测试用例，终端将依次输出imu地址扫描结果和imu初始化结果，随后伴随蜂鸣器的响声，**RDK IMU Module**载板上的3个LED依次闪烁，然后终端输出板载温度，最后程序将会循环读取并打印IMU的6轴数据和时间戳、温度数据<br>

*** 

## **三、C-Samples快速体验**

### **3.1.完整sample运行**


## **四、C接口介绍**

## **五、Python接口介绍**
