---2017-01-23 14:02---
This is a Windows 10 IoT Core project on the Raspberry Pi 2/3, coded by C#.

GitHub：<https://github.com/ZhangGaoxing/windows-iot-demo/tree/master/ADXL345>

## Connect
ADXL345 connect to SPI
* VCC - 3.3 V
* GND -  GND
* CS - CS0 - Pin24
* SDO - Pin21
* SDA - Pin19
* SCL - Pin23

## Reference
In Chinese : <http://wenku.baidu.com/view/87a1cf5c312b3169a451a47e.html>

In English : <https://github.com/ZhangGaoxing/windows-iot-demo/tree/master/ADXL345/Reference>

ms-iot some code : <https://github.com/ms-iot/samples/tree/develop/SPIAccelerometer/CS>

## How to Use
* First, you need to create a ADXL345 object. After that you should call InitializeAsync() to initialize. 
```C#
ADXL345 sensor = new ADXL345(0, GravityRange.Four);
await sensor.InitializeAsync();
```
* Second, ReadAcceleration(). 
```C#
var accel = sensor.ReadAcceleration();
```
* If you want to close the sensor, call Dispose().
```C#
sensor.Dispose();
```