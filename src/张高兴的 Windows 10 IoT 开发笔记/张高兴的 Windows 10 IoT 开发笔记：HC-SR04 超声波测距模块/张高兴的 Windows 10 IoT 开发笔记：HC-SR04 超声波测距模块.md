---2017-01-23 11:57---

HC-SR04 采用 IO 触发测距。下面介绍一下其在 Windows 10 IoT Core 环境下的用法。

项目运行在 Raspberry Pi 2/3 上，使用 C# 进行编码。

## 1. 准备

* HC-SR04 ×1 
* Raspberry Pi 2/3 ×1
* 公母头杜邦线 ×4

## 2. 连线

* Vcc - 5V
* Gnd - GND
* Trig - GPIO 17 - Pin 11
* Echo - GPIO 27 - Pin 13

![](http://p42sif40h.bkt.clouddn.com/18-2-13/74433159.jpg)

## 3. 代码

GitHub : https://github.com/ZhangGaoxing/windows-iot-demo/tree/master/HC_SR04

你需要在项目中添加一个 C# 代码文件 HCSR04.cs，将下面的代码复制粘贴，并且不要忘记添加引用 **Windows IoT Extensions for the UWP**

```c#
using System.Diagnostics;
using System.Threading.Tasks;
using Windows.Devices.Gpio;

namespace HC_SR04Demo
{
    class HCSR04
    {
        private int sensorTrig;
        private int sensorEcho;

        private GpioPin pinTrig;
        private GpioPin pinEcho;

        Stopwatch time = new Stopwatch();

        /// <summary>
        /// Constructor
        /// </summary>
        /// <param name="trig">Trig Pin</param>
        /// <param name="echo">Echo Pin</param>
        public HCSR04(int trig, int echo)
        {
            sensorTrig = trig;
            sensorEcho = echo;
        }

        /// <summary>
        /// Initialize the sensor
        /// </summary>
        public void Initialize()
        {
            var gpio = GpioController.GetDefault();

            pinTrig = gpio.OpenPin(sensorTrig);
            pinEcho = gpio.OpenPin(sensorEcho);

            pinTrig.SetDriveMode(GpioPinDriveMode.Output);
            pinEcho.SetDriveMode(GpioPinDriveMode.Input);

            pinTrig.Write(GpioPinValue.Low);
        }

        /// <summary>
        /// Read data from the sensor
        /// </summary>
        /// <returns>A double type distance data</returns>
        public async Task<double> ReadAsync()
        {
            double result;

            pinTrig.Write(GpioPinValue.High);
            await Task.Delay(10);
            pinTrig.Write(GpioPinValue.Low);

            while (pinEcho.Read() == GpioPinValue.Low)
            {

            }
            time.Restart();
            while (pinEcho.Read() == GpioPinValue.High)
            {

            }
            time.Stop();

            result = (time.Elapsed.TotalSeconds * 34000) / 2;

            return result;
        }

        /// <summary>
        /// Cleanup
        /// </summary>
        public void Dispose()
        {
            pinTrig.Dispose();
            pinEcho.Dispose();
        }
    }
}
```
 
## 4. 如何使用

* 第一步调用构造函数将 HCSR04 实例化，请传入 Trig 和 Echo 的连接值
* 第二步调用 Initialize() 初始化设备
* 第三步调用 ReadAsync() 读取数据，返回的是一个 double 类型的值
* 当需要关闭设备时，调用 Dispose() 

<br>

 **详见 GitHub**