# Setting Up MicroROS on ESP32 with Arduino IDE and ESP-IDF

This Repository elaborates the detailed Instructions used to setup MicroROS with ESP32. We can setup Pub/Sub Network using MicroROS on ESP32 where we can send commands from ROS2 Interface and Control GPIO Pins on ESP32.

For this setup both ESP-IDF (Recommended) and Arduino IDE are used.

## System Overview

MicroROS with ROS2 can be visualized as Server/Client Architecture. The system consists of following important components:

- MicroROS Agent
- MicroROS Client
- Communication Interface (UART, WiFi, Custom Interface etc)

**MicroROS Agent** runs on Host Computer that is responsible for communication with MicroROS Client using the communication interface or protocol defined in the firmware. In this system, MicroROS Agent can be thought as a server which listens on specific socket address (IP Address and Port Address). The MicroROS Agent can work in bidirectional manner, it can receive data from MicroROS Client or it can receive data from main ROS2 network and send it to MicroROS Client.

**MicroROS Client** is a Firmware that is flashed into the Microcontroller. The microcontroller is very resource constrained device, it cannot run a complete operating system. Microcontrollers are programmed for very specific type of tasks, for example, ESP32 is a microcontroller which has Bluetooth and WIFI modules, so ESP32 can be controlled using WiFi. We can flash MicroROS Firmware into the microcontroller, in this way the microcontroller will act as MicroROS Client.

**Communication Interface** is, simply put, a communication protocol used for communication between client and server, or MicroROS Agent and MicroROS Client. Since, ESP32 has WiFi capabilities, we can use WIFI as communication protocol. Other interfaces such as UART (Wired Serial Communication), or even custom interface (using external modules, such as Radio) can be used. The custom interfaces are useful in terms of mission critical or long range communications.

The Visual Representation of Architecture is as follows:

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/MicroROS-Architecture.png)

In the above image, NVIDIA Jetson Orin is Host Computer, since it is powerful device that can run an Operating System. The Host Computer is running MicroROS Agent. On the other side, ESP32 is used as client, it has Firmware that is flashed. The Communication protocol is USB (Serial Communication) in this case

### Resources Used

- ROS2 Humble (Version should be consistent for things to work properly)
- Arduino IDE (Latest Version) - For Arduino IDE based setup
- ESP-IDF (Docker Version) - For ESP-IDF based Setup (Recommended)
- Ubuntu 22.04

## 1) MicroROS with ESP32 and Arduino IDE

Firstly, we will setup MicroROS with ESP32 using Arduino IDE. Since this is easier and straight forward way.

#### Arduino IDE Installation

[Arduino IDE Download Page](https://www.arduino.cc/en/software)

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/arduino-installation.png)

Choose Linux Zip File (64 Bits) as an option. Next, extract the Zip File in any directory and then run arduino-ide. It will run Arduino IDE

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/arduino-interface.png)

#### Setup Arduino IDE for ESP32 Programming

Since Arduino IDE is mostly built for Arduino Based boards, we need to install some additional packages for programming ESP32 with Arduino IDE.

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/arduino-ide-open-preferences.webp)

Go to preferences and Enter the following into "Addtional Board Manager URLs" field

`https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`

Then, click the "OK " button.

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/ESP32-URL-Arduino-IDE.webp)

Next, Open the Boards Manager. Go to **Tools > Boards > Boards Manager**

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/boardsManager.webp)

Search for **ESP32** and press Install button for **ESP32 by Espressif Systems**

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/installing.webp)

That's all! The Board Manager will be installed in some moments.

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/esp32_manager_installed.png)

Now, restart your Arduino IDE for changes to take effect.

Then, we can test the installation by selecting board in **Tools > Board** 

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/board_selection.png)

In Our Case, it is DOIT ESP32 DEVKIT V1

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/devkit_selected.png)

Now, we are ready to program ESP32 using Arduino IDE!

#### Adding MicroROS Library in Arduino IDE

In this step, we will add MicroROS Library in Arduino IDE to use MicroROS header file in our code sketch

- Download MicroROS Arduino Library from this link

`https://github.com/micro-ROS/micro_ros_arduino/tree/humble`

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/microROS-arduino-libary-download-page.png)

After Downloading this Zip File, we need to add this library to Arduino IDE.

- Open Arduino IDE and go to **Sketch > Include Library > Add .ZIP Library**

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/arduino-add-zip-library.png)

- Select that ZIP file downloaded in the previous step. 

Now the MicroROS Library is added in Arduino IDE and we are ready to write MicroROS Firmware into our ESP32 board using Arduino IDE.

At this point we are have successfully setup MicroROS Client. We can write code for our by adding MicroROS library header files. The next thing we need to do is to setup MicroROS Agent

#### Setup MicroROS Agent in ROS2 Humble

We need to create ROS2 Agent in our Host Computer. In our case the Host Computer is running ROS2 Humble on Ubuntu 22.04.

- Create the ROS2 Workspace

```bash
mkdir -p ros2_ws/src
```

- Go inside the src directory in ros2_ws

```bash
cd ros2_ws/src
```

- Clone MicroROS setup Repository from GitHub

```bash
git clone -b humble https://github.com/micro-ROS/micro_ros_setup.git
```

- Go to the root of ros2_ws

```bash
cd ..
```

- Build the Workspace

```bash
colcon build
```

- Source the Workspace after build process is finished

```bash
source install/setup.bash
```

Next create the MicroROS Agent

- Run the following command to create Agent Workspace

```bash
ros2 run micro_ros_setup create_agent_ws.sh
```

- Build the Agent Workspace

```
ros2 run micro_ros_setup build_agent.sh
```

- Then Source the workspace

```bash
source install/setup.bash
```

Now we are ready to start the server (MicroROS Agent) on Socket Address.

- Run the following command

```
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```

> Make sure to note down the port address "8888" in our case. We need to use the port address and IP address in our MicroROS Client Setup.
>
> To Identify the IP Address of your Host Machine running ROS2 run the command **ifconfig** or **ip a** in Terminal

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/ip-address.png) 

> Note down the IP Address

Now that our MicroROS Agent (Or Server) is running and listening on selected port, our client can communicate with it. For that we need to write code for MicroROS Client in Arduino IDE and flash that firmware into ESP32.

#### LED Blinking with ESP32 and MicroROS

Open the Code in "Codes" directory in Arduino IDE

![](/home/muneeb/Private/BrainSwarm/esp32-microROS/assets/subscriber_code_esp32.png)

On Line Number 40, Modify

- Noectic to Your SSID
- islamabad to your WIFI Password
- 192.168.158.163 to your IP Address of Host Computer
- 8888 to that port that your MicroROS Agent is listening on

> Make sure you have ESP32 and Host computer are connected to same Network or same WIFI.

Now, after this setup is done, we are ready to flash this firmware into ESP32. Select the Port from **Tools > Port** and Build and Upload the Code to ESP32.

This Firmware is Listening on Topic "micro_ros_arduino_subscriber". You can publish to this topic using ROS2 on Host Computer to see the LED Blink of ESP32.

```
ros2 pub /micro_ros_arduino_subscriber std_msgs/msg/Int32 "data: 0"
```

or

```
ros2 pub /micro_ros_arduino_subscriber std_msgs/msg/Int32 "data: 1"
```

These commands will toggle the LED to ON and OFF.

## Conclusion

We have successfully setup MicroROS using Arduino IDE for ESP32 and have run the example of subscriber to toggle LED to ON and OFF using ROS2 Command from Host Computer. We can extend this basic functionality to more sophisticated ones, such as controlling DC or Servo Motors, other peripherals, sensors and actuators connected to the GPIO pins of ESP32.