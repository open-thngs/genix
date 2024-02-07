# GeniX

GeniX is a Generic inter-integrated Circuit Communication Protocoll on top of the I2C protocol to make the communication between sensor modules and main MCU’s or CPU’s more predictable, structured and abstract. The objective is to establish a universal language compatible with various sensor types, thereby minimizing the need for extensive software development and fostering modularization. This approach enables seamless integration of sensors into systems without any coding requirements, facilitating effortless sensor replacement for maintenance purposes or upgrading from less accurate models to superior ones.

- **Flexibility**: By utilizing a standardized communication interface and abstraction layer, various sensor modules can be seamlessly integrated regardless of their specific features or manufacturers.

- **Modularity**: The system's modularity allows for easy swapping or addition of individual sensor modules without needing to reconfigure the entire architecture. This facilitates system scalability and enables agile adaptation to changing requirements.

- **Development Efficiency**: Abstraction at the MCU level simplifies and accelerates the implementation of sensor modules since developers don't need to recode everything from scratch each time a new sensor device is added.

- **Interoperability**: Since all sensor modules follow the same communication standard and abstraction layer, they can communicate seamlessly with each other and with the main control unit. This streamlines integration and sensor swapping, ensuring smooth system functionality.

- **Enhanced Maintainability and Reliability**: Clear task separation between the GeniX controller and the sensor modules simplifies system maintenance and enables easy isolation and resolution of faults. Additionally, standardization minimizes the likelihood of compatibility issues, improving overall system reliability.

## GeniX Block

<img src="/img/genix-block-diagram.png" width="70%">

The image above illustrates the abstraction of the sensor modules and the main MCU. Each sensor module is equipped with its own MCU, which serves to abstract business logic, handle data analysis tasks, and manage communication with the sensor via its dedicated protocol, such as I2C or SPI (the most common ones). The MCU within the sensor node operates as a GeniX Device, facilitating communication with the main MCU. This approach renders the sensor hardware interchangeable and highly abstract, allowing it to transmit only the necessary data. Functions such as calibration, driver logic, calculations, and data analysis are performed on the sensor module MCU, thereby relieving the main MCU of these tasks and allocating more resources for other functions such as wireless communication and data storage.

### GeniX Device

A GeniX Device encompasses any sensor module that adheres to the GeniX communication standard. Each GeniX Device can integrate multiple functions or sensor types. Moreover, each device can operate as both an actor and/or a sensor. This versatility allows a GeniX Device to incorporate various sensors such as temperature, pressure, analog or digital sensors, as well as communication protocols like I2C, SPI, UART, actuators, or any other supported feature. The MCU embedded within the GeniX Device abstracts the logic required to communicate with sensors or actuators and translates the sensor-specific details into the GeniX Language, known as a Feature.

### GeniX Controller

The GeniX Controller serves as a device that connects all GeniX Devices to the Processing Unit. Essentially, it functions as a GPIO expander, expanding the functionality of the Processing Unit to receive interrupts from a GeniX Device or to enable/disable a GeniX Device, which is necessary for the discovery process, as explained later. Although the GeniX Controller itself is a GeniX Device, it possesses specialized functionality. A processing unit will recognize the controller device and utilize it to control other GeniX Devices.

### GeniX Processing Unit

The GeniX Processing Unit serves as the main processor in the entire modular sensor system. It leverages the GeniX Controller to control and discover all GeniX Devices within the system. Once this initialization process is complete, data from the sensors can be gathered and transmitted to an external system. The method of transmission is determined by the processing unit and can include options such as WiFi, LTE, Thread, LoRaWAN, or any other communication protocol, provided that the processing unit can communicate with the GeniX Devices via the GeniX protocol, which is based on I2C. This architecture offers a highly flexible system capable of functioning and communicating in various ways.

## GeniX Discovery Process

To discover all modules and features in the system the GeniX Processing Unit needs to run a procedure to identify all modules. The procedure steps need to run as follows:
1. Discover the GeniX Controller Module by scanning for I2C Devices and find the device in the range of 0x60 to 0x77. There is a defined range of I2C Address a GeniX Controller can have to identify itself to the Processing Unit. 
2. Once the GeniX Controller is available a discovery command is send to receive all its features. This will usually multiple Binary Inputs and Outputs to controll the Enable and Interrupt Pins of each GeniX Device.
3. The Processing Unit will use the GeniX Controller to disable all GeniX Devices in the system by enabling the EN Feature for each GeniX Device which will disable all devices in the system
4. note: If the Processing Unit scans for I2C Modules it should only show the GeniX Controller module.
5. the Processing Unit enables one (or the first) GeniX Device by disabling the EN Feature of one of the discovered features
6. then the Processing Unit scans for I2C Devices and will discover a new and unknown device which will be the first GeniX Device 
7. now the Processing Unit sets a new address to this device (the default address for all devices after boot is 0x08 which should not be used). Which address is used will be determined by the Processing Unit
8. the Processing Unit can now send teh command to discover the features of this GeniX Device.
9. Repeat step 4 to 8 until you discovered and setup all GeniX Deviecs

Once this process is done the GeniX Processing Unit will know about every GeniX Device and its Features.

## GeniX Device Features

A Feature is one or multiple "things" a device can do. This includes being an actor wit ha binary output or input or simply being a temperature sensor. One GeniX Device can have many differnet features and the system is not limited to just have one feature. A Feature is always represented by a Feature ID or Sensor ID to identify a feature within the GeniX Device and a Feature Type Code or Sensor Type Code which identifies the type of the Feature. The GeniX System has a defined list of features which are possible. This list is not complete and can be extened if needed but contains the most common types known so far. Together with a Feature Type Code the system also knows about the Unit and the values to expect. Here is a list of possible features:

| Type ID | Name           | Unit                  |
|---------|----------------|-----------------------|
| 0x01    | Temperature    | °C                    |
| 0x02    | Pressure       | Pa                    |
| 0x03    | Humidity       | %                     |
| 0x04    | Light          | Lux                   |
| 0x05    | Color          | RGBW HEX String       |
| 0x06    | Current        | mA                    |
| 0x07    | Encoder        | 0-360° Deg            |
| 0x08    | Binary In      | 1 or 0                |
| 0x09    | Binary Out     | 1 or 0                |
| 0x0A    | Distance       | mm                    |
| 0x0B    | Gas            | ppb                   |
| 0x0C    | Magnetometer   | Microtesla (µT) or Gauss (G) |
| 0x0D    | Sound          | dB                    |
| 0x0E    | Force Sensor   | N (Newton)            |
| 0x0F    | Flow Sensor    | m/s                   |
| 0x10    | Gyro X         | °/s or rad/s          |
| 0x11    | Gyro Y         | °/s or rad/s          |
| 0x12    | Gyro Z         | °/s or rad/s          |
| 0x13    | Mag X          | Microtesla (µT) or Gauss (G) |
| 0x14    | Mag Y          | Microtesla (µT) or Gauss (G) |
| 0x15    | Mag Z          | Microtesla (µT) or Gauss (G) |
| 0x16    | Accel X        | m/s² or G-Force (g)   |
| 0x17    | Accel Y        | m/s² or G-Force (g)   |
| 0x18    | Accel Z        | m/s² or G-Force (g)   |


## GeniX Connection Interface

To achieve such abstraction, a connection interface is essential to link a GeniX Device with its base GeniX Controller and the Processing Module. The image below illustrates this connection interface.

<img src="/img/genix-wire-interface-pinout.png" width="50%">

| Description       | Name |-| Name | Description      |
|-------------------|------|-|------|------------------|
| 3.3V Input/Output | 3V3  |-|  12V | 12V Input/Output |
| GROUND            | GND  |-|  GND | GROUND           |
| Not Connected     | NC   |-|  5V  | 5V Input/Output  |
| Not Connected     | NC   |-|  GND | GROUND           |
| Not Connected     | NC   |-|  SRC | Source Input/Output for any type of Input Voltage like Solar |

| Description                    | Name |-| Name  | Description                    |
|--------------------------------|------|-|-------|--------------------------------|
| Transmit Line                  | Tx   |-|  SDA  | Serial Data  |
| Receive Line                   | Rx   |-|  SCL  | Serial Clock |
| Not Connected                  | NC   |-|  EN   | Enable Pin of the GeniX Device |
| Interrupt of this GeniX Device | INT  |-|  USB+ | USB Positive Line |
| GROUND                         | GND  |-|  USB- | USB Negative Line |

Each GeniX Module has to have this type of connection interface to connect to the system. The connections for SDA, SCL, any type of needed Power, EN and INT are mendatory to make the system work properly. The rest are optional functions which can be used if needed.

# Prototype

I created a prototype for this GeniX System based on the ESP32 Microcontroller and the RP2040 of the Raspberry Pi Foundation. These microcontrollers were chosen for their ease of programming using languages like Toit or Micropython, eliminating the need for heavy C code.

The prototype comprises the following modules:
- **Processing Unit**: An ESP32-based module serving as the processing unit.
- **GeniX Controller**: A 2x2 matrix of GeniX Connection Interfaces based on an RP2040 and a GPIO Expander.
- **GeniX Device**: An RP2040-based module containing a Temperature sensor (STS4X) and a pressure sensor (DPS368). I simulated multiple sensors within one GeniX Device by enabling/disabling the sensors on different modules.
- **Power Module**: a module using the SRC input and the 3V3 output to generate the needed power for the system. Here I only used a USB-C 5V Input as SRC and an LDO to lower it to 3.3V. It is possible to also add multiple LDOs or Buck/Boost converters to achieve 5V or 12V if needed with different Power Modules.

This prototype demonstrates the feasibility of the GeniX System architecture and showcases its potential for integration and scalability using readily available microcontrollers.

<img src="/img/prototype-single.png" width="50%">

<img src="/img/prototype-together.png" width="50%">