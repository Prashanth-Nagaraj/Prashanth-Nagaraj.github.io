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

![loading image...]({{site.baseurl}}/img/1/raspberry-pi-connection.jpg)

# **Create Azure IoT Hub**

IoT Hub is a managed service that acts as a central message hub for bi-directional communication between your IoT application and the devices it manages.  

Go to [Azure Portal](https://portal.azure.com/) | Create a resource | Internet of Things | IoT Hub.

![loading image...]({{site.baseurl}}/img/1/create-iot-hub.gif)

**Subscription** 	- Choose your subscription    
**Resource Group** 	– Create a new resource group and name it “electricity-usage-resource-group”    
**Region** 			– Choose nearest region to you    
**IoT Hub Name** 	– Enter “electricity-usage-iot-hub” or choose the name you like     

![loading image...]({{site.baseurl}}/img/1/create-iot-hub-subscription.jpg)

Click Next | Select “F1: Free Tier” under Pricing and scale tier and leave rest as it is | Review and Create.    

![loading image...]({{site.baseurl}}/img/1/create-iot-hub-size-and-scale.jpg)

Next, add an IoT device to your IoT Hub application. IoT device is a device identity registry in IoT Hub. Ideally every physical IoT device will have a new IoT device registry added in IoT Hub. IoT device on IoT Hub can be perceived as a logical entity for a physical IoT device. The data exchange between IoT Hub and Physical IoT device happens through this device registry endpoints.   

Select All resources | electricity-usage-iot-hub resource | IoT devices | Click Add



