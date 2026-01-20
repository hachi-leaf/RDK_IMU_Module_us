# **RDK_IMU_Module_us**
## ———— *Driver RDK IMU Module in User Space* ————
&emsp;基于用户空间的RDK IMU Module软件包，完全不依赖iio相关驱动，仅使用linux标准通用I2C与SPI驱动<br>
&emsp;**Version: 1.0.0**

## **一、编译说明**

### **1.1.依赖说明**

项目分为两个部分：<br>

1. **rdk_imu_module_data:** 核心部分，用于调用Linux通用SPI与I2C驱动与IMU通信，并进行数据解析与时间同步，编译需`-lm`链接math<br>

2. **rdk_imu_module_gpio:** 次要部分，用于驱动**RDK IMU Module**载板上的GPIO设备（LED、蜂鸣器和1-wire温度传感器），依赖**RDK wiringPi**，需要按照以下教程进行**RDK wiringPi**的安装：[在旭日X3派上移植的 WiringPi](https://gitee.com/xin03liu/WiringPi)，并在编译时`-lwiringPi`链接**wiring Pi**<br>

- 不安装**RDK wiringPi**也可以使用**rdk_imu_module_data**的C部分内容，但不能使用**rdk_imu_module_gpio**和Python部分内容<br>

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

#### **2.1.2.动态链接库编译**

&emsp;项目自带一个编译好的、可用于python的`.so`动态链接库文件，位于`RDK_IMU_Module_us/Py_RDK_IMU/rdkimu.cpython-310-aarch64-linux-gnu.so`，如果确认`RDK wiringPi`已正确安装，可以直接运行sample<br>

&emsp;如果需要重新编译Cython动态链接库，回到Makefile目录下`make cython`即可<br>

### **2.2.sample执行**

&emsp;`python3 test_pyimu.py`执行测试用例，终端将依次输出imu地址扫描结果和imu初始化结果，随后伴随蜂鸣器的响声，**RDK IMU Module**载板上的3个LED依次闪烁，然后终端输出板载温度，最后程序将会循环读取并打印IMU的6轴数据和时间戳、温度数据<br>

## **三、C-Samples快速体验**

### **3.1.完整sample运行**

&emsp;`cd /path/to/RDK_IMU_Module_us`来到项目目录下，通过`make sample`编译测试用例，确认编译成功后可直接运行`./bin/imu_test`文件，测试用例将会进行IMU设备的扫描与初始化，并驱动**RDK IMU Module**载板发出声光提示，输出以下内容：<br>

```text
root@ubuntu:~/RDK_IMU_Module_us# ./bin/imu_test 
[examples/imu_test.c]Successfully detected the BMI088 device.
[examples/imu_test.c]RDK_IMU_Accel_Reset 0
[examples/imu_test.c]RDK_IMU_Gyro_Reset 0
[examples/imu_test.c]IMU initialization successful 0
[examples/imu_test.c]GPIO initialization successful 0
[examples/imu_test.c]The 0th time obtaining the ambient temperature is 31.312500 Celsius.
[examples/imu_test.c]The 1th time obtaining the ambient temperature is 31.312500 Celsius.
[examples/imu_test.c]The 2th time obtaining the ambient temperature is 31.375000 Celsius.
```

&emsp;然后清屏，输出IMU数据，如下：<br>

```text
[imu data] accel: -0.015,  -0.012,  -1.008, timestamp: 1768881430201181
[imu data]  gyro:  0.153,   0.580,   0.443, timestamp: 1768881430200252
[temp data] temp: 28.875000
```

### **3.2.仅运行核心示例**

&emsp;不安装、不链接wiringPi，仅运行核心示例，只需要使用`make sample-Wo-gpio`编译不带gpio的测试用例，将会`./bin/imu_test_without_gpio`文件，运行后程序同样会进行IMU设备的扫描与初始化，但没有声光提示，然后清屏，循环打印IMU数据<br>

## **四、C接口介绍**

&emsp;`RDK_IMU_Module_us/include/rdk_imu_module.h`中定义了驱动imu的所有函数和枚举类型，`RDK_IMU_Module_us/include/bmi088_regs.h`中则定义了BMI088芯片的所有寄存器信息以及状态枚举<br>

### **4.1.错误枚举**

&emsp;`enum rdk_imu_error{}`枚举了所有的错误类型，该枚举类型作为大部分函数的返回值，值`RDK_IMU_OK`为无错误<br>

### **4.2.data接口说明**

- `struct imu_state`:<br>
- IMU状态结构体，是所有imu data相关函数的重要参数，记录了IMU模块的完整状态（不含中断）<br>
<br>

- `struct imu_data`:<br>
- IMU数据结构体，用于格式化存储imu数据包的完整数据<br>
<br>

- `struct imu_state RDK_IMU_Get_Initial_State()`:<br>
- 初始化结构体`imu_state`的标准方式<br>
    - `return`: 默认状态的`imu_state`结构体，使用一个`struct imu_state`类型变量来接收它<br>
<br>

- `struct imu_data RDK_IMU_Get_Initial_Data()`: 初始化结构体`imu_data`的标准方式<br>
    - `return`: 默认状态的`imu_data`结构体，使用一个`struct imu_data`类型变量来接收它<br>
<br>

- `enum rdk_imu_error RDK_IMU_All_Device_Scan(`
- &emsp;`struct imu_state *st)`:<br> 
- IMU设备扫描函数,支持I2C/SPI自动检测，支持I2C全总线全地址自动扫描<br>
    - `*st`: `struct imu_state`类型指针<br>
    - `return`: `enum rdk_imu_error`错误码<br>
<br>

- `enum rdk_imu_error RDK_IMU_Accel_Pwr_Set(`
- &emsp;`struct imu_state *st,`
- &emsp;`enum bmi088_acc_pwr_mode pwr_mode)`: 加速度计电源模式设置<br>
    - `*st`: `struct imu_state`类型指针<br>
    - `pwr_mode`: `enum bmi088_acc_pwr_mode`类型，可选值如下：<br>
        - `ACC_PWR_OFF`: 关闭电源<br>
        - `ACC_PWR_SUSPEND`: 挂起<br>
        - `ACC_PWR_ON`: 启动<br>
    - `return`: `enum rdk_imu_error`错误码<br>
<br>

- `enum rdk_imu_error RDK_IMU_Gyro_Pwr_Set(`<br>
- &emsp;`struct imu_state *st,`<br>
- &emsp;`enum bmi088_gyro_lpm1 pwr_mode)`:<br>
    - `*st`: `struct imu_state`类型指针<br>
    - `pwr_mode`: `enum bmi088_gyro_lpm1`类型，可选值如下：<br>
        - `GYRO_LPM1_NORMAL`: 启动<br>
        - `GYRO_LPM1_SUSPEND`: 挂起<br>
        - `GYRO_LPM1_DEEP_SUSPEND`: 深度挂起<br>
    - `return`: `enum rdk_imu_error`错误码<br>



### **4.3.gpio接口说明**

## **五、Python接口介绍**
