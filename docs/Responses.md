# Responses

The `query()` function returns `OBDResponse` objects. These objects have the following properties:

| Property | Description                                                            |
|----------|------------------------------------------------------------------------|
| value    | The decoded value from the car                                         |
| command  | The `OBDCommand` object that triggered this response                   |
| message  | The internal `Message` object containing the raw response from the car |
| time     | Timestamp of response (as given by [`time.time()`](https://docs.python.org/2/library/time.html#time.time)) |



---

## is_null()

Use this function to check if a response is empty. Python-OBD will emit empty responses when it is unable to retrieve data from the car.

```python
r = connection.query(obd.commands.RPM)

if not r.is_null():
	print(r.value)
```

---


## Pint Values

The `value` property typically contains a [Pint](http://pint.readthedocs.io/en/latest/) `Quantity` object, but can also hold complex structures (depending on the request). Pint quantities combine a value and unit into a single class, and are used to represent physical values such as "4 seconds", and "88 mph". This allows for consistency when doing math and unit conversions. Pint maintains a registry of units, which is exposed in python-OBD as `obd.Unit`.

Below are common operations that can be done with Pint units and quantities. For more information, check out the [Pint Documentation](http://pint.readthedocs.io/en/latest/).

<span style="color:red">*NOTE: for backwards compatibility with previous versions of python-OBD, use `response.value.magnitude` in place of `response.value`*</span>

```python
import obd

>>> response.value
<Quantity(100, 'kph')>

# get the raw python datatype
>>> response.value.magnitude
100

# converts quantities to strings
>>> str(response.value)
'100 kph'

# convert strings to quantities
>>> obd.Unit("100 kph")
<Quantity(100, 'kph')>

# handles conversions nicely
>>> response.value.to('mph')
<Quantity(62.13711922373341, 'mph')>

# scaler math
>>> response.value / 2
<Quantity(50.0, 'kph')>

# non-scaler math requires you to specify units yourself
>>> response.value + (20 * obd.Unit.kph)
<Quantity(120, 'kph')>

# non-scaler math with different units
# handles unit conversions transparently
>>> response.value + (20 * obd.Unit.mph)
<Quantity(132.18688, 'kph')>
```

---

## Status

The status command returns information about the Malfunction Indicator Light (check-engine light), the number of trouble codes being thrown, and the type of engine.

```python
response.value.MIL              # boolean for whether the check-engine is lit
response.value.DTC_count        # number (int) of DTCs being thrown
response.value.ignition_type    # "spark" or "compression"
```

The status command also provides information regarding the availability and status of various system tests. These are exposed as `StatusTest` objects, loaded into named properties. Each test object has boolean flags for its availability and completion.

```python
response.value.MISFIRE_MONITORING.available    # boolean for test availability
response.value.MISFIRE_MONITORING.complete     # boolean for test completion
```

Here are all of the tests names that python-OBD reports:

| Tests                             |
|-----------------------------------|
| MISFIRE_MONITORING                |
| FUEL_SYSTEM_MONITORING            |
| COMPONENT_MONITORING              |
| CATALYST_MONITORING               |
| HEATED_CATALYST_MONITORING        |
| EVAPORATIVE_SYSTEM_MONITORING     |
| SECONDARY_AIR_SYSTEM_MONITORING   |
| OXYGEN_SENSOR_MONITORING          |
| OXYGEN_SENSOR_HEATER_MONITORING   |
| EGR_VVT_SYSTEM_MONITORING         |
| NMHC_CATALYST_MONITORING          |
| NOX_SCR_AFTERTREATMENT_MONITORING |
| BOOST_PRESSURE_MONITORING         |
| EXHAUST_GAS_SENSOR_MONITORING     |
| PM_FILTER_MONITORING              |


---

## Diagnostic Trouble Codes (DTCs)

Each DTC is represented by a tuple containing the DTC code, and a description (if python-OBD has one). For commands that return multiple DTCs, a list is used.

```python
# obd.commands.GET_DTC
response.value = [
    ("P0104", "Mass or Volume Air Flow Circuit Intermittent"),
    ("B0003", ""), # unknown error code, it's probably vehicle-specific
    ("C0123", "")
]

# obd.commands.FREEZE_DTC
response.value = ("P0104", "Mass or Volume Air Flow Circuit Intermittent")
```

---

## Fuel Status

The fuel status is a tuple of two strings, telling the status of the first and second fuel systems. Most cars only have one system, so the second element will likely be an empty string. The possible fuel statuses are:

| Fuel Status                                                                                   |
| ----------------------------------------------------------------------------------------------|
| `""`                                                                                          |
| `"Open loop due to insufficient engine temperature"`                                          |
| `"Closed loop, using oxygen sensor feedback to determine fuel mix"`                           |
| `"Open loop due to engine load OR fuel cut due to deceleration"`                              |
| `"Open loop due to system failure"`                                                           |
| `"Closed loop, using at least one oxygen sensor but there is a fault in the feedback system"` |

---

## Air Status

The air status will be one of these strings:

| Air Status                             |
| ---------------------------------------|
| `"Upstream"`                           |
| `"Downstream of catalytic converter"`  |
| `"From the outside atmosphere or off"` |
| `"Pump commanded on for diagnostics"`  |

---

## Oxygen Sensors Present

Returns a 2D structure of tuples (representing bank and sensor number), that holds boolean values for sensor presence.

```python
# obd.commands.O2_SENSORS
response.value = (
    (),                           # bank 0 is invalid, this is merely for correct indexing
    (True,  True,  True,  False), # bank 1
    (False, False, False, False)  # bank 2
)

# obd.commands.O2_SENSORS_ALT
response.value = (
    (),             # bank 0 is invalid, this is merely for correct indexing
    (True,  True),  # bank 1
    (True,  False), # bank 2
    (False, False), # bank 3
    (False, False)  # bank 4
)

# example usage:
response.value[1][2] == True # Bank 1, Sensor 2 is present
```
---

## Monitors (Mode 06 Responses)

All mode 06 commands return `Monitor` objects holding various test results for the requested sensor. A single monitor response can hold multiple tests, in the form of `MonitorTest` objects. The OBD standard defines some tests, but vehicles can always implement custom tests beyond the standard. Here are the standard Test IDs (TIDs) that python-OBD will recognize:

| TID | Name                     | Description                                        |
|-----|--------------------------|----------------------------------------------------|
| 01  | RTL_THRESHOLD_VOLTAGE    | Rich to lean sensor threshold voltage              |
| 02  | LTR_THRESHOLD_VOLTAGE    | Lean to rich sensor threshold voltage              |
| 03  | LOW_VOLTAGE_SWITCH_TIME  | Low sensor voltage for switch time calculation     |
| 04  | HIGH_VOLTAGE_SWITCH_TIME | High sensor voltage for switch time calculation    |
| 05  | RTL_SWITCH_TIME          | Rich to lean sensor switch time                    |
| 06  | LTR_SWITCH_TIME          | Lean to rich sensor switch time                    |
| 07  | MIN_VOLTAGE              | Minimum sensor voltage for test cycle              |
| 08  | MAX_VOLTAGE              | Maximum sensor voltage for test cycle              |
| 09  | TRANSITION_TIME          | Time between sensor transitions                    |
| 0A  | SENSOR_PERIOD            | Sensor period                                      |
| 0B  | MISFIRE_AVERAGE          | Average misfire counts for last ten driving cycles |
| 0C  | MISFIRE_COUNT            | Misfire counts for last/current driving cycles     |

Test results can be accessed by property name or TID (same as the `obd.commands` tables). All of the standard tests above will be present, though some may be null. Use the `MonitorTest.is_null()` function to determine if a test is null.

```python
response.value.MISFIRE_COUNT

# OR

response.value["MISFIRE_COUNT"]

# OR

response.value[0x0C] # TID for MISFIRE_COUNT
```

All `MonitorTest` objects have the following properties: (for null tests, these are set to `None`)

```python
result = response.value.MISFIRE_COUNT

result.tid      # integer Test ID for this test
result.name     # test name
result.desc     # test description
result.value    # value of the test (will be a Pint value, or in rare cases, a boolean)
result.min      # maximum acceptable value
result.max      # minimum acceptable value
result.passed   # boolean marking the test as passing
```

Here is an example of looking up live misfire counts for the engine's second cylinder:

```python
import obd

connection = obd.OBD()

response = connection.query(obd.commands.MONITOR_MISFIRE_CYLINDER_2)

# in the test results, lookup the result for MISFIRE_COUNT
result = response.value.MISFIRE_COUNT

# check that we got data for this test
if not result.is_null():
    print(result.value) # will be a Pint value
else:
    print("Misfire count wasn't reported")
```

---

<br>
