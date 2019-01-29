---
published: true
---
IoT (Internet of Things) is a rapidly growing technology that can bring significant changes to modern life. Since its invention, the single board computers such as Raspberry Pi has been an integral part of IoT solutions.   

Although there are several operating systems in the market that can run on IoT devices, Windows IoT (formerly Windows Embedded) family of operating systems is rapidly gaining popularity. Windows IoT Core is an optimized operating system for low cost industry devices like Raspberry Pi and its free.   

In this article I’ll show you how to send data from Raspberry PI running Windows IoT Core to Azure IoT hub.

> This blog assumes that you have basic understanding of C#, .NET, .NET Core and Visual Studio.   

# **Source Code**

The source code used in this tutorial can be found at my [GitHub](https://github.com/prashanth-nagaraj/ElectricityUsageMonitor) repository.

# **Prerequisites**

- Raspberry Pi Model 2/3B (You can buy from official site [here](https://www.raspberrypi.org/))
- SD card suitable for your Pi model.
- Visual Studio 2017 with Universal Windows Platform tools (You can download free community edition [here](https://visualstudio.microsoft.com/downloads/))
- Enable Windows Developer Mode in your Desktop. Follow [here](https://docs.microsoft.com/en-us/windows/uwp/get-started/enable-your-device-for-development))
- Azure subscription (You can get free subscription [here](https://azure.microsoft.com/en-gb/free/))

# **Set up Raspberry Pi**

Follow the steps given [here](https://docs.microsoft.com/en-us/windows/iot-core/tutorials/quickstarter/devicesetup) and
- Install Windows IoT Core on to your SD card. 
- Once the OS is flashed on to SD card insert this SD card in to your Raspberry PI.
- Connect your Raspberry Pi to the network. Make sure your laptop and raspberry pi are connected to same network.

![Raspberry Pi Connection]({{site.baseurl}}/img/1-raspberry-pi-connection.jpg)

# **Create Azure IoT Hub**

IoT Hub is a managed service that acts as a central message hub for bi-directional communication between your IoT application and the devices it manages.  

Go to [Azure Portal](https://portal.azure.com/) | Create a resource | Internet of Things | IoT Hub.
