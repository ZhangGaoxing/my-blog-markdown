---2017-01-22 18:43---

BH1750FVI 是一款 IIC 接口的数字型光强度传感器集成电路。下面介绍一下其在 Windows 10 IoT Core 环境下的用法。

项目运行在 Raspberry Pi 2/3 上，使用 C# 进行编码。

## 1. 准备

包含 BH1750FVI 的传感器，这里选择的是淘宝上最多的 GY-30；Raspberry Pi 2/3 一块，环境为 Windows 10 IoT Core；公母头杜邦线 4-5 根 

## 2. 连线

Raspberry Pi 2/3 的引脚如图

![](http://images2015.cnblogs.com/blog/1085877/201701/1085877-20170122175036973-2075058115.png)

由于采用的是 IIC 通信方式，因此我们需要把 GY-30 上的 SDA 与 Pin3 相连，SCL 与 Pin5 相连。VCC 接 3.3V，GND 接地。ADD 决定了传感器的地址，将其连接至 VCC ≥ 0.7 V 的时候，地址为 0x5C，接地时为 0x23。可以不连接。

* SDA - Pin3
* SCL - Pin5
* VCC - 3.3V
* GND - GND

![](http://images2015.cnblogs.com/blog/1085877/201701/1085877-20170122180458363-1169633855.png)

## 3. 代码

GitHub : <https://github.com/ZhangGaoxing/windows-iot-demo/tree/master/BH1750FVI>

需要新建一个 Windows 通用 项目 ，并且添加引用 Windows IoT Extensions for the UWP

![](http://images2015.cnblogs.com/blog/1085877/201701/1085877-20170122182806535-2119326185.png)

在项目中添加一个 C# 代码文件 BH1750FVI.cs，代码如下

```c#
using System;
using System.Threading.Tasks;
using Windows.Devices.I2c;

namespace BH1750FVIDemo
{
    /// <summary>
    /// I2C Address Setting
    /// </summary>
    enum AddressSetting
    {
        /// <summary>
        /// ADD Pin connect to high power level
        /// </summary>
        AddPinHigh = 0x5C,
        /// <summary>
        /// ADD Pin connect to low power level 
        /// </summary>     
        AddPinLow = 0x23            
    };

    /// <summary>
    /// The mode of measuring
    /// </summary>
    enum MeasurementMode
    {
        /// <summary>
        /// Start measurement at 1 lx resolution
        /// </summary>
        ContinuouslyHighResolutionMode = 0x10,
        /// <summary>
        /// Start measurement at 0.5 lx resolution
        /// </summary>
        ContinuouslyHighResolutionMode2 = 0x11,
        /// <summary>
        /// Start measurement at 4 lx resolution
        /// </summary>
        ContinuouslyLowResolutionMode = 0x13,
        /// <summary>
        /// Start measurement at 1 lx resolution once
        /// </summary>
        OneTimeHighResolutionMode = 0x20,
        /// <summary>
        /// Start measurement at 0.5 lx resolution once
        /// </summary>
        OneTimeHighResolutionMode2 = 0x21,
        /// <summary>
        /// Start measurement at 4 lx resolution once
        /// </summary>
        OneTimeLowResolutionMode = 0x23
    }

    /// <summary>
    /// Setting light transmittance
    /// </summary>
    enum LightTransmittance
    {
        Fifty,
        Eighty,
        Hundred,
        Hundred_Twenty,
        Hundred_Fifty,
        Two_Hundred
    }

    class BH1750FVI
    {
        I2cDevice sensor;
        private byte sensorAddress;                             
        private byte sensorMode;
        private byte sensorResolution = 1;
        private double sensorTransmittance = 1;

        private byte registerHighVal = 0x42;
        private byte registerLowVal = 0x65;

        /// <summary>
        /// Constructor
        /// </summary>
        /// <param name="address">Enumeration type of AddressSetting</param>
        /// <param name="mode">Enumeration type of MeasurementMode</param>
        public BH1750FVI(AddressSetting address, MeasurementMode mode)
        {
            sensorAddress = (byte)address;
            sensorMode = (byte)mode;

            if (mode == MeasurementMode.ContinuouslyHighResolutionMode2 || mode == MeasurementMode.OneTimeHighResolutionMode2)
            {
                sensorResolution = 2;
            }
        }

        /// <summary>
        /// Constructor
        /// </summary>
        /// <param name="address">Enumeration type of AddressSetting</param>
        /// <param name="mode">Enumeration type of MeasurementMode</param>
        /// <param name="transmittance">Enumeration type of LightTransmittance</param>
        public BH1750FVI(AddressSetting address, MeasurementMode mode, LightTransmittance transmittance)
        {
            sensorAddress = (byte)address;
            sensorMode = (byte)mode;

            if (mode == MeasurementMode.ContinuouslyHighResolutionMode2 || mode == MeasurementMode.OneTimeHighResolutionMode2)
            {
                sensorResolution = 2;
            }

            switch (transmittance)
            {
                case LightTransmittance.Fifty:
                    {
                        registerHighVal = 0x44;
                        registerLowVal = 0x6A;
                        sensorTransmittance = 0.5;
                    }
                    break;
                case LightTransmittance.Eighty:
                    {
                        registerHighVal = 0x42;
                        registerLowVal = 0x76;
                        sensorTransmittance = 0.8;
                    }
                    break;
                case LightTransmittance.Hundred:
                    {
                        registerHighVal = 0x42;
                        registerLowVal = 0x65;
                    }
                    break;
                case LightTransmittance.Hundred_Twenty:
                    {
                        registerHighVal = 0x41;
                        registerLowVal = 0x7A;
                        sensorTransmittance = 1.2;
                    }
                    break;
                case LightTransmittance.Hundred_Fifty:
                    {
                        registerHighVal = 0x41;
                        registerLowVal = 0x7E;
                        sensorTransmittance = 1.5;
                    }
                    break;
                case LightTransmittance.Two_Hundred:
                    {
                        registerHighVal = 0x41;
                        registerLowVal = 0x73;
                        sensorTransmittance = 2;
                    }
                    break;
            }
        }

        /// <summary>
        /// Initialize BH1750FVI
        /// </summary>
        public async Task InitializeAsync()
        {
            var settings = new I2cConnectionSettings(sensorAddress);
            settings.BusSpeed = I2cBusSpeed.FastMode;                     

            var controller = await I2cController.GetDefaultAsync();
            sensor = controller.GetDevice(settings);

            sensor.Write(new byte[] { 0x01 });
            sensor.Write(new byte[] { registerHighVal });
            sensor.Write(new byte[] { registerLowVal });
        }

        /// <summary>
        /// Read data from BH1750FVI
        /// </summary>
        /// <returns>A double type contains data</returns>
        public double Read()
        {
            byte[] readBuf = new byte[2];

            sensor.WriteRead(new byte[] { sensorMode }, readBuf);

            byte temp = readBuf[0];
            readBuf[0] = readBuf[1];
            readBuf[1] = temp;

            double result = BitConverter.ToUInt16(readBuf, 0) / (1.2 * sensorResolution * sensorTransmittance);

            return result;
        }

        /// <summary>
        /// Cleanup
        /// </summary>
        public void Dispose()
        {
            sensor.Dispose();
        }
    }
}
```

## 4. 如何使用

代码包含三个枚举类型，两个构造函数，三个方法。

* 第一步调用构造函数将 BH1750FVI 实例化。
* 第二步调用 InitializeAsync() 初始化 I2C 设备
* 第三步调用 Read() 读取数据，返回的是一个 double 类型的值
* 当需要关闭设备时，调用 Dispose() 