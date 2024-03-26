# Custom Commands

If the command you need is not in python-OBDs tables, you can create a new `OBDCommand` object. The constructor accepts the following arguments (each will become a property).

| Argument             | Type     | Description                                                                |
|----------------------|----------|----------------------------------------------------------------------------|
| name                 | string   | (human readability only)                                                   |
| desc                 | string   | (human readability only)                                                   |
| command              | bytes    | OBD command in hex (typically mode + PID                                   |
| bytes                | int      | Number of bytes expected in response (zero means unknown)                  |
| decoder              | callable | Function used for decoding messages from the OBD adapter                   |
| ecu (optional)       | ECU      | ID of the ECU this command should listen to (`ECU.ALL` by default)         |
| fast (optional)      | bool     | Allows python-OBD to alter this command for efficieny (`False` by default) |
| header (optional)    | string   | If set, use a custom header instead of the default one (7E0)               |


## Example

```python
from obd import OBDCommand, Unit
from obd.protocols import ECU
from obd.utils import bytes_to_int

def rpm(messages):
    """ decoder for RPM messages """
    d = messages[0].data # only operate on a single message
    d = d[2:] # chop off mode and PID bytes
    v = bytes_to_int(d) / 4.0  # helper function for converting byte arrays to ints
    return v * Unit.RPM # construct a Pint Quantity

c = OBDCommand("RPM", \          # name
               "Engine RPM", \   # description
               b"010C", \        # command
               4, \              # number of return bytes to expect
               rpm, \            # decoding function
               ECU.ENGINE, \     # (optional) ECU filter
               True)             # (optional) allow a "01" to be added for speed
```

By default, custom commands will be treated as "unsupported by the vehicle". There are two ways to handle this:

```python
o = obd.OBD()

# use the `force` parameter when querying
o.query(c, force=True)

# OR

# add your command to the set of supported commands
o.supported_commands.add(c)
o.query(c)
```

<br>

Here are some details on the less intuitive fields of an OBDCommand:

---

## OBDCommand.decoder

The `decoder` argument is a function of following form.

```python
def <name>(<list_of_messages>):
    ...
    return <value>
```

The return value of your decoder will be loaded into the `OBDResponse.value` field. Decoders are given a list of `Message` objects as an argument. If your decoder is called, this list is garaunteed to have at least one message object. Each `Message` object has a `data` property, which holds a parsed bytearray, and is also garauteed to have the number of bytes specified by the command. This bytearray includes any mode and PID bytes in the vehicle's response.

*NOTE: If you are transitioning from an older version of Python-OBD (where decoders were given raw hex strings as arguments), you can use the `Message.hex()` function as a patch.*

```python
def <name>(messages):
    _hex = messages[0].hex()
    ...
    return <value>
```

*You can also access the original string sent by the adapter using the `Message.raw()` function.*

---

## OBDCommand.ecu

The `ecu` argument is a constant used to filter incoming messages. Some commands may listen to multiple ECUs (such as DTC decoders), where others may only be concerned with the engine (such as RPM). Currently, python-OBD can only distinguish the engine, but this list may be expanded over time:

- `ECU.ALL`
- `ECU.ALL_KNOWN`
- `ECU.UNKNOWN`
- `ECU.ENGINE`

---

## OBDCommand.fast

The optional `fast` argument tells python-OBD whether it is safe to append a `"01"` to the end of the command. This will instruct the adapter to return the first response it recieves, rather than waiting for more (and eventually reaching a timeout). This can speed up requests significantly, and is enabled for most of python-OBDs internal commands. However, for unusual commands, it is safest to leave this disabled.

---

## OBDCommand.header

The optional `header` argument tells python-OBD to use a custom header when querying the command. If not set, python-OBD assumes that the default 7E0 header is needed for querying the command. The switch between default and custom header (and vice versa) is automatically done by python-OBD.

---

<br>
