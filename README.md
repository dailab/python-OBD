python-OBD-wifi
==========

A python module for handling realtime sensor data from OBD-II vehicle
ports. It is derived from the python-OBD repo for serial data. This is an adaption for ELM327 Wifi compatible modules.


```

Basic Usage
-----------

```Python
import obd

connection = obd.OBD("192.168.0.10", 35000) # connect to specific OBD-II-wifi Adapter. In our case it is a Panda by comma.ai

cmd = obd.commands.SPEED # select an OBD command (sensor)

response = connection.query(cmd) # send the command, and parse the response

print(response.value) # returns unit-bearing values thanks to Pint
print(response.value.to("mph")) # user-friendly unit conversions
```

Documentation
-------------

Available at [python-obd.readthedocs.org](http://python-obd.readthedocs.org/en/latest/)

Commands
--------

Here are a handful of the supported commands (sensors). For a full list, see [the docs](http://python-obd.readthedocs.io/en/latest/Command%20Tables/)

*note: support for these commands will vary from car to car*

-   Calculated Engine Load
-   Engine Coolant Temperature
-   Fuel Pressure
-   Intake Manifold Pressure
-   Engine RPM
-   Vehicle Speed
-   Timing Advance
-   Intake Air Temp
-   Air Flow Rate (MAF)
-   Throttle Position
-   Engine Run Time
-   Fuel Level Input
-   Number of warm-ups since codes cleared
-   Barometric Pressure
-   Ambient air temperature
-   Commanded throttle actuator
-   Time run with MIL on
-   Time since trouble codes cleared
-   Hybrid battery pack remaining life
-   Engine fuel rate
-   Vehicle Identification Number (VIN)

License
-------

GNU GPL v2

This library is forked from:

-   <https://github.com/peterh/pyobd>
-   <https://github.com/Pbartek/pyobd-pi>
-   <https://github.com/brendan-w/python-OBD>

Enjoy and drive safe!
