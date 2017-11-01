# SonarI2C-RPi
Raspberry Pi Python library for the Octosonar breakout board by Alastair Young.

<b>Links to Alastair's pages:</b> <br>
The original Arduino library: [github.com](https://github.com/arielnh56/SonarI2C)<br>
Hackaday: [hackaday.io](https://hackaday.io/project/19950-hc-sr04-i2c-octopus-octosonar)<br>
Blog: [redhunter.com](http://redhunter.com/blog/2016/04/28/sonari2c-multiple-hc-sr04-sensors-on-arduino-i2c/)<br>
Buy it on Tindie: [tindie.com](https://www.tindie.com/products/10614/)<br>

## Summary

This is a Raspberry Pi Python library for the Octosonar by Alastair Young.

The Octosonar is a breakout board for connecting eight ultrasonic sensors (HC-SR04) to a microcontroller (Arduino). This library adds support for the Raspberry Pi as well. It's connected via I2C and only needs three pins (SCL/SDA and INT) on the Pi.

Will also work with a PCF8574 expander and tri-state buffers. 
See Alastair's links above for more information on how to set that up.

Note: This is not a direct port of the Arduino library. It works perfectly with the Octosonar but the functions and classes are not the same.

## Supported Platforms

Raspberry Pi (any model) with Python. Tested with Python 2.7 and Python 3.4

## Getting Started

### Hardware Set Up

This branch of the library is forked from the [work by ageir](https://github.com/ageir/SonarI2C-RPi) which also used level shifting hardware to interface to the V1 OctoSonar board which used a NOR gate to demux the echo signals. The V2 OctoSonar is compatible with 3.3V controllers such as the Raspberry Pi, though still needs a separate 5V input to drive the sensors. So ageir's diagrams regarding level shifters have been removed from this README.

### Software Requirements

This library requires the pigpio library. You can download it here:
[http://abyz.co.uk/rpi/pigpio/](http://abyz.co.uk/rpi/pigpio/)<br>

To install it with PIP:

Python 2.x
```c
pip install pigpio
```

Python 3.x
```c
pip3 install pigpio
```

The pigpio deamon needs to be started for the pigpio library to work.
```c
sudo pigpiod
```

### Example Code

Example code is available in the src directory in the repository.

The code triggers all sonars in order (0-7) and prints a list containing the results. Press CTRL-C to cancel.

You will need to adjust the line ```octosonar = SonarI2C(pi, int_gpio=25)``` to suit your setup. See documentation below for the class. Example: ```octosonar = SonarI2C(pi, int_gpio=25, bus=1, addr=0x3d, max_range_cm=400) ```

Adjust the ```time.sleep(0.01)``` delay to suit your needs. Might be needed if you get a lot of echoes and false readings as the echos might spread to other sonars.

 
```c
from SonarI2C import SonarI2C
import pigpio
import time

print("Press CTRL-C to cancel.")
pi = pigpio.pi()
if not pi.connected:
    exit(0)
try:
    octosonar = SonarI2C(pi, int_gpio=25)
    result_list = []
    while True:
        for i in range(8):
            sonar_result = octosonar.read_cm(i)
            time.sleep(0.01)
            if sonar_result is False:
                result_list.append("Timed out")
            else:
                result_list.append(round(sonar_result, 1))
        print(result_list)
        result_list = []

except KeyboardInterrupt:
    print("\nCTRL-C pressed. Cleaning up and exiting.")
finally:
    octosonar.cancel()
    pi.stop()
```

## Class Definitions

### Class SonarI2C

       Arguments:
        pi          -- pigpio instance
        int_gpio    -- the GPIO connected to the octosonars INT pin.
                       BCM numbering.
        bus         -- The i2c bus number, set 0 for the first Raspberry Pi
                       model with 256MB ram, else 1.
                       default: 1
        addr        -- i2c address of the octosonar.
                       default: 0x3d
        max_range_cm-- Maximum range for the sonars in centimeters.
                       default: 400

Example: ```octosonar = SonarI2C(pi, int_gpio=25, bus=1, addr=0x3d, max_range_cm=400)```

### SonarI2C.read()

        Takes a measurement on a port on the Octosonar.
        Not adjusted for round trip. This is the number of
        microseconds that the INT pin is high.

        Arguments:
        port    -- port on the Octosonar, Valid values: 0-7
        Returns: Distance in microseconds. False if timed out.

Example: ```octosonar.read(0)```

### SonarI2C.read_cm()

        Takes a measurement on a port on the Octosonar.
        Adjusted for round trip. Returns real distance to
        the object.

        Arguments:
        
        port    -- port on the Octosonar, Valid values: 0-7
        Returns: Distance in centimeters. False if timed out.

Example: ```octosonar.read_cm(0)```

### SonarI2C.read_inch()

        Takes a measurement on a port on the Octosonar.
        Adjusted for round trip. Returns real distance to
        the object.

        Arguments:
        
        port    -- port on the Octosonar, Valid values: 0-7
        Returns: Distance in inches. False if timed out.

Example: ```octosonar.read_inch(0)```

### SonarI2C.cancel()

        Cancels the Octosonar and cleans up resources.

Example: ```octosonar.cancel()```
