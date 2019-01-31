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

![loading image...]({{site.baseurl}}/img/1/add-iot-hub-device.jpg)

**Device Id** – “electricity-usage-simulated-sensor” or choose the name you like   

And leave rest of the options as it is. Click Save.    

After device is created, you can see the list of devices you created in ‘IoT Devices’ section. Click on the device which you just created and copy the Connection string (primary key) and store it. You need this later to connect from code to Azure IoT Hub.

# **Create UWP application**

Create a headless UWP application that can be deployed to Raspberry Pi. Headless applications are the background applications that doesn’t have any UI. The applications which has UI are called Headed application. An IoT device running Windows IoT Core can have any number of headless applications, but it can have only one headed application enabled.

Install Windows IoT project templates from visual studio market place. Launch Visual Studio | Go to Tools | Extensions and Updates | Search for “Windows IoT core project templates” and click Install.

![loading image...]({{site.baseurl}}/img/1/add-windows-iot-templates.jpg)

Go to File | New | Project | Select Background Application (IoT) under Windows IoT Core template.    

Enter “ElectricityUsageMonitor” for name and click OK.    

![loading image...]({{site.baseurl}}/img/1/create-uwp-application.jpg)

You will get a pop up to select Windows Target Version and Minimum version. Select the defaults and click OK.

![loading image...]({{site.baseurl}}/img/1/select-windows-versions.jpg)

You need Microsoft.Devices.Azure.Client NuGet package to interact with Azure IoT Hub. So, Install this package from the NuGet gallery.

# **Let’s look at Some Code**

In this example we will use a simulated electricity usage provider to collect electricity usage data, which we are going to send to Azure IoT Hub. But in real world there will be actual sensors that collect data.    

Replace StartupTask.cs file with the below code.

```csharp
using Microsoft.Azure.Devices.Client;
using Newtonsoft.Json;
using System;
using System.Diagnostics;
using System.Text;
using System.Threading.Tasks;
using Windows.ApplicationModel.Background;

namespace ElectricityUsageMonitor
{
    public sealed partial class StartupTask : IBackgroundTask
    {
        private BackgroundTaskDeferral _deferral;
        private SimulatedElectricityUsageProvider _electricityUsageProvider;
        private DeviceClient _deviceClient;
        private const string connectionString = "<replace>";
        private const int maxReadings = 5;

        public void Run(IBackgroundTaskInstance taskInstance)
        {
            Debug.WriteLine("Task Started");
            Initialize(taskInstance);
            SendElectricityUsageToCloud();
        }

        private void Initialize(IBackgroundTaskInstance taskInstance)
        {
            _electricityUsageProvider = new SimulatedElectricityUsageProvider();
            _deferral = taskInstance.GetDeferral();
            _deviceClient = DeviceClient.CreateFromConnectionString(connectionString);
        }
        private async void SendElectricityUsageToCloud()
        {
            Debug.WriteLine("Sending electricity usage to cloud");
            double currentUsage;
            for (int i = 0; i < maxReadings; i++)
            {
                currentUsage = _electricityUsageProvider.GetCurrentUsage();

                var electricityUsageData = new
                {
                    DeviceName = "SimulatedElectricityProvider",
                    Time = DateTime.Now.ToString(),
                    CurrentUsage = currentUsage
                };

                var electricityUsageJson = JsonConvert.SerializeObject(electricityUsageData);
                using (var electricityUsage = new Message(Encoding.UTF8.GetBytes(electricityUsageJson)))
                {
                    electricityUsage.ContentEncoding = "utf-8";
                    electricityUsage.ContentType = "application/json";

                    Debug.WriteLine("{0} - Sending usage - {1}", DateTime.Now, electricityUsageJson);
                    await _deviceClient.SendEventAsync(electricityUsage);
                }
                await Task.Delay(1000);
            }

            _deferral.Complete();
        }
    }
}
```

Create a new class and name it SimulatedElectricityUsageProvider.cs. Replace the file with the below contents.

```csharp
namespace ElectricityUsageMonitor
{
    internal class SimulatedElectricityUsageProvider
    {
        private double baseUsage = 10;

        public double GetCurrentUsage()
        {
            baseUsage = baseUsage += 2;
            return baseUsage;
        }
    }
}
```
Run method of StartupTask class is the first method that gets called soon after the application starts.

I have added a little bit of code to initialize SimulatedElectricityUsageProvider and DeviceClient objects. SimulatedElectricityUsageProvider class will provide the electricity usage data (like electricity meter readings).    

DeviceClient is a class from Microsoft.Devices.Azure.Client library contains methods that can send message from device and receive from a cloud service. Replace the ConnectionString value with the IoT Hub device’s connection string which you copied earlier.    

Background applications will end when the Run method is executed completely unless you take a deferral.

    _deferral = taskInstance.GetDeferral();

Once the deferral is taken, the background application will continue until the Complete method of deferral object is called.

    _deferral.Complete();
    
We will use SendEventAsync method of _deviceClient object to send data to client. To send data to IoT Hub we need to construct Message that contains data. Message object can be initialized by passing the data as byte array or Stream. Here we are passing the byte array of the data.

# **Deploying application to Raspberry Pi**

Make sure you have connected the Raspberry Pi to the same network of your computer.    

Open Windows Core IoT Dashboard and go to My devices. If you have connected Raspberry Pi to the network correctly, you should see your device there. Right Click on the device and click on Open in Device Portal. This will bring up Windows Device Portal. This is one stop shop to manage your device.    

![loading image...]({{site.baseurl}}/img/1/iot-core-dashboard.jpg)

Deploying UWP application from Visual studio to Raspberry Pi is a straight forward process. With the application open in visual studio, set the Architecture to ARM in the toolbar dropdown and click on the Local Machine drop down and select Remote Machine.    

![loading image...]({{site.baseurl}}/img/1/select-architecture.jpg)

Select your device connection in Remote Connections dialog.   

![loading image...]({{site.baseurl}}/img/1/select-remote-connection.jpg)

Go to Build menu in Visual Studio and click Deploy Solution. This will deploy the application to Raspberry Pi. After the deployment is completed go to Windows Device Portal and click on Apps manager. This is similar to Task Manager in Windows. Here you will see all the applications that are running in the device.   

![loading image...]({{site.baseurl}}/img/1/app-manager.jpg)

Now the application starts sending ElectricUsageData to Azure IoT Device and stops. If you want to restart the application click on Actions dropdown and select start.   

# **Processing data on cloud**

Sending the data to cloud is only meaningful if we do something useful with the data. There are many ways you can process data that is sent to Azure IoT Hub. You can connect it to other azure services like Service Bus Queue, Blob Storage, Power BI etc.    

You can imagine some real-world scenarios like triggering an alert if the electricity usage exceeds a limit etc. But that’s out of scope of this blog.However, we shall make sure the data is successfully received in the cloud. Let’s retrieve the messages sent to cloud in a .NetCore console application.   

Create a .NetCore console application in Visual Studio. Add the WindowsAzure.ServiceBus library from the NuGet gallery and replace the generated Program.cs with the following code.   

```csharp
using Microsoft.Azure.EventHubs;
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace ElectricityUsageReader
{
    internal static class ReadElectricityUsageFromCloud
    {
        private static EventHubClient _eventHubClient;

        private static async Task Main(string[] args)
        {
            var connectionString = "replace";
            Console.WriteLine("Electricty Usage Reader - Reading messages from cloud");
            _eventHubClient = EventHubClient.CreateFromConnectionString(connectionString);
            var runtimeInfo = await _eventHubClient.GetRuntimeInformationAsync();
            var d2cPartitions = runtimeInfo.PartitionIds;

            CancellationTokenSource cts = new CancellationTokenSource();

            Console.CancelKeyPress += (s, e) =>
            {
                e.Cancel = true;
                cts.Cancel();
                Console.WriteLine("Exiting...");
            };

            var tasks = new List<Task>();
            foreach (string partition in d2cPartitions)
            {
                tasks.Add(ReadElectrictyUsageAsync(partition, cts.Token));
            }

            Task.WaitAll(tasks.ToArray());
        }

        private static async Task ReadElectrictyUsageAsync(string partition, CancellationToken ct)
        {
            var eventHubReceiver = _eventHubClient.CreateReceiver("$Default", partition, EventPosition.FromEnqueuedTime(DateTime.Now));
            Console.WriteLine("Create receiver on partition: " + partition);
            while (true)
            {
                if (ct.IsCancellationRequested) break;
                Console.WriteLine("Listening for messages on: " + partition);
                var events = await eventHubReceiver.ReceiveAsync(100);

                if (events == null) continue;

                foreach (EventData eventData in events)
                {
                    string data = Encoding.UTF8.GetString(eventData.Body.Array);
                    Console.WriteLine("Message received on partition {0}:", partition);
                    Console.WriteLine("  {0}:", data);
                }
            }
        }
    }
}
```

Replace the connection string with your Azure IoT Hub connection string from Azure Portal. This is not the device’s connection string which we used earlier.   

To get the Azure IoT Hub connection string, Go to Azure IoT Hub resource in azure portal | Shared access policy | click on iothubowner policy | use connection string – primary key value.   

Compile and run this program, this will start listening for the message received at IoT Hub. Keep this program running and go to Windows Device Portal and start the application on Raspberry Pi. You should be able to see messages received at IoT Hub on your console application.

# **Processing data on cloud**








