# AN022 Tracker SHT3x Temperature/Humidity

The full code examples for this application note can be found in the [Github repository](https://github.com/particle-iot/app-notes/tree/master/AN022-Tracker-SHT3x-Temperature-Humidity) for this project.

## Introduction

This application note illustrates several hardware and software techniques:

- Expanding the Tracker One using the M8 connector
- Interfacing with 5V I2C devices
- Connecting a SHT30 or SHT31 I2C Temperature and Humidity Sensor
- Adding custom data to your location publishes

![Board Image](images/board-with-m8.jpg)

While the Tracker One board contains a precision thermistor, the sensor shown here is good for measuring temperature in locations separate from the Tracker, for example outside, in a refrigeration unit, etc..

## Connecting

The M8 (8mm) 8-pin connector on the Tracker One is standard, however it's not common. Some other connectors like M12 are more common, however, the 12mm connector would have required a taller enclosure to fit the larger connector. To simplify designs, Particle will provide a M8 female-to-wires cable, similar to this. This is for illustration only and the design may vary in the future.

![M8 cable](images/m8-cable.jpg)

The common use case will be to include a cable gland in your expansion enclosure, pass the wires through the gland, and terminate them on your custom expansion board.

You'd typically connect those wires to your custom expansion board using one of several methods:

- Terminate with pins in a PHR-8 to mate with a B8B-PH on your expansion board
- Terminate with screw terminals on your board
- Terminate by soldering the wires to your board

This example design is intended to be a prototype for illustration purposes only. It includes the same B8B-PH connector that is inside the Tracker One on the Tracker Carrier board. This allows the board to be connected to the carrier board using an inexpensive PHR-8 to PHR-8 cable.

## Hardware Design

![Board](images/board.jpg)

### Full Design

Schematic:

![Schematic](images/schematic.png)

Board:

![Board Layout](images/board-layout.png)

The Eagle CAD files for this design and the Gerber files are included in the **eagle** subdirectory.


### BoM (Bill of Materials)

| Quantity | Part | Description | Example | Cost |
| :---: | :--- | :--- | :--- | ---: |
| 1 | C1 | Capacitor Ceramic 4.7uF 6.3V 0603 | [Murata GRM188R60J475KE19J](https://www.digikey.com/product-detail/en/murata-electronics-north-america/GRM188R60J475KE19J/490-6407-1-ND/3845604) | |
| 1 | C2 | Capacitor Ceramic 10uF 16V 0805 | [Murata GRM21BR61C106KE15L](https://www.digikey.com/product-detail/en/murata-electronics-north-america/GRM21BR61C106KE15L/490-3886-1-ND/965928) | |
| 1 | C3 | Capacitor Ceramic 100pF 50V 0603 | [Kemet C0603C101J5GACTU](https://www.digikey.com/product-detail/en/kemet/C0603C101J5GACTU/399-1061-1-ND/411336) |  |
| 4 | R1, R2, R4, R5 | Resistor 10K 5% 1/4W 0603 | [Panasonic ERJ-PA3J103V](https://www.digikey.com/product-detail/en/panasonic-electronic-components/ERJ-PA3J103V/P10KBZCT-ND/5036237) | | 
| 1 | R3 | Resistor 200K 1% 1/10W 0603| [Panasonic ERJ-3EKF2003V](https://www.digikey.com/product-detail/en/panasonic-electronic-components/ERJ-3EKF2003V/P200KHCT-ND/198241) | |
| 1 | U1 | XCL224 3.3V regulator | [Torex XCL224A333D2-G](https://www.digikey.com/product-detail/en/torex-semiconductor-ltd/XCL224A333D2-G/893-1419-1-ND/8256121) |$ 2.43| 
| 1 | U2 | PCA9306SSOP | [TI](https://www.digikey.com/product-detail/en/texas-instruments/PCA9306DCTR/296-18509-1-ND/809938) | $0.67|  
| 1 | J3 | TERM BLK 4POS SIDE ENT 3.5MM PCB | [On Shore OSTTE040161](https://www.digikey.com/product-detail/en/on-shore-technology-inc/OSTTE040161/ED2637-ND/614586) | $1.01 |


Choose one of:

| Quantity | Part | Description | Example | Cost | 
| :---: | :--- | :--- | :--- | ---: | 
| 1 | J1 | Conn SMD 8 position 2.00mm | [JST B8B-PH-SM4-TB(LF)(SN)](https://www.digikey.com/product-detail/en/jst-sales-america-inc/B8B-PH-SM4-TB-LF-SN/455-1740-1-ND/926837) | $1.00 |
|   | J2 | Male Header Pins (8x0.1") | [Sullins PRPC040SAAN-RC](https://www.digikey.com/product-detail/en/PRPC040SAAN-RC/S1011EC-40-ND/2775214) | |
| 1 | J2 | Screw Terminal Block 8x0.1" PTH | [On Shore OSTVN08A150](https://www.digikey.com/product-detail/en/on-shore-technology-inc/OSTVN08A150/ED10566-ND/1588868) | $2.36 | 


### Regulator

![Regulator](images/regulator.png)

The M8 connector supplies 5V at 400 mA, and can be turned on and off using the `CAN_PWR` GPIO. There is a boost converter on the Tracker SoM and 5V is available off battery as well as USB and external VIN power.

Since the nRF52840 MCU only supports 3.3V logic levels on I2C, Serial, and GPIO, a 3.3V regulator is often required. This design uses a Torex XCL223 or XCL224. It's tiny, inexpensive, and does not require an external inductor, which saves space and BoM costs. It's a 700 mA regulator, but you'll be limited to the 400 mA on CAN_5V.

### PCA9306

![PCA9306](images/pca9306.png)

The SHT3x sensors require 5V, so we need to translate the 3.3V I2C on the Tracker SoM to 5V. 

**The nRF52 is not 5V tolerant! You cannot directly connect 5V I2C to it!**

We use a PCA9306 I2C level-shifter. This converts between 3.3V and 5V logic. Note that I2C is bi-directional on both pins (SDA and SCL), so you can't just use a simple level-shifter.

Note that I2C requires pull-up resistors, and this design includes two sets, one to 3.3V and one to 5V, on either side of the PCA9306.

### Adding Sensors

There are a few commonly available SHT30 sensors:

- [From Adafuit](https://www.adafruit.com/product/4099)
- [From Amazon](https://www.amazon.com/gp/product/B07F9X9Q37/ref=ppx_yo_dt_b_asin_title_o01_s00)

Note that the color code printed on the PCB is for the Adafruit sensor. The sensor I got from Amazon had a different color code, so be careful!

Also be sure to use the SHT30, SHT31, etc.. The earlier sensors like the SHT10 are not compatible! The SHT10 looks the same but uses a proprietary protocol instead of I2C.

![Board with Sensor](images/board-with-sensor.jpg)

## Firmware

### Getting the Tracker Edge Firmware

The Tracker Edge firmware can be downloaded from Github:

[https://github.com/particle-iot/tracker-edge](https://github.com/particle-iot/tracker-edge)

You will probably want to use the command line as there are additional commands you need to run after cloning the source:

```bash
git clone git@github.com:particle-iot/tracker-edge.git 
cd tracker-edge
git submodule update --init --recursive
```

- Open Particle Workbench.
- From the command palette, **Particle: Import Project**.
- Run **Particle: Configure Workspace for Device**, select version 1.5.4-rc.1, or 3.0.0 or later, Tracker, and your device.
- Run **Particle: Compile and Flash**.

Be sure to target 1.5.4-rc.1, or 3.0.0 or later, for your build. The 2.0.x LTS versions of Device OS do not have Tracker support. There will be versions of 3.0.x released concurrently with 2.0.x releases. 

### Add the sht3x-i2c library

From the command palette in Workbench, **Particle: Install Library** then enter **sht3x-i2c**.

The documentation for the library can be found [here](https://github.com/particle-iot/sht3x-i2c).

### Customize main.cpp

```cpp
#include "Particle.h"

#include "tracker_config.h"
#include "tracker.h"
#include "sht3x-i2c.h"

SYSTEM_THREAD(ENABLED);
SYSTEM_MODE(SEMI_AUTOMATIC);

PRODUCT_ID(TRACKER_PRODUCT_ID);
PRODUCT_VERSION(TRACKER_PRODUCT_VERSION);

SerialLogHandler logHandler(115200, LOG_LEVEL_TRACE, {
    { "app.gps.nmea", LOG_LEVEL_INFO },
    { "app.gps.ubx",  LOG_LEVEL_INFO },
    { "ncp.at", LOG_LEVEL_INFO },
    { "net.ppp.client", LOG_LEVEL_INFO },
});

Sht3xi2c sensor(Wire3, 0x44);

void locationGenerationCallback(JSONWriter &writer, LocationPoint &point, const void *context); // Forward declaration

void setup()
{
    Tracker::instance().init();

    // Register a location callback so we can add temperature and humidity information
    // to location publishes
    Tracker::instance().location.regLocGenCallback(locationGenerationCallback);
    
    // Turn on 5V output on M8 connector
    pinMode(CAN_PWR, OUTPUT);
    digitalWrite(CAN_PWR, HIGH);
    delay(500);

    sensor.begin(CLOCK_SPEED_400KHZ);
    sensor.start_periodic();

    Particle.connect();
}

void loop()
{
    Tracker::instance().loop();
}

void locationGenerationCallback(JSONWriter &writer, LocationPoint &point, const void *context)
{
    double temp, humid;

    int err = sensor.get_reading(&temp, &humid);
    if (err == 0)
    {
        writer.name("sh31_temp").value(temp);
        writer.name("sh31_humid").value(humid);

        Log.info("temp=%.2lf hum=%.2lf", temp, humid);
    }
    else {
        Log.info("no sensor err=%d", err);
    }
}

```

### Digging In

```cpp
Sht3xi2c sensor(Wire3, 0x44);
```

Note that the M8 I2C interface is `Wire3` not `Wire` as you might be used to on other Particle devices where `Wire` is on pins D0 and D1. On the Tracker M8 connector, `Wire3` is on the same physical pins as `Serial1` so you can only use one port or the other.


```cpp
Tracker::instance().init();

// Register a location callback so we can add temperature and humidity information
// to location publishes
Tracker::instance().location.regLocGenCallback(locationGenerationCallback);
```

In `setup()` initialize the Tracker firmware and add a callback for when the location publishes occur.

```cpp
// Turn on 5V output on M8 connector
pinMode(CAN_PWR, OUTPUT);
digitalWrite(CAN_PWR, HIGH);
delay(500);

sensor.begin(CLOCK_SPEED_400KHZ);
sensor.start_periodic();
```

The on `CAN_PWR` to power the regulator, level-shifter, and sensor. Then initialize the sensor.

```cpp
Particle.connect();
```

Finally connect to the Particle cloud at the end of setup.

```cpp
Tracker::instance().loop();
```

Make sure you give the Tracker edge firmware processor time on every call to `loop()`.

```cpp
void locationGenerationCallback(JSONWriter &writer, LocationPoint &point, const void *context)
{
    double temp, humid;

    int err = sensor.get_reading(&temp, &humid);
    if (err == 0)
    {
        writer.name("sh31_temp").value(temp);
        writer.name("sh31_humid").value(humid);

        Log.info("temp=%.2lf hum=%.2lf", temp, humid);
    }
    else {
        Log.info("no sensor err=%d", err);
    }
}
```

Finally, when there is a location publish, add the sensor data to it.


## Cloud Data

In the map view in the [console](https://console.particle.io), you should be able to see the additional custom data:

![Custom Data](images/custom-data.png)
