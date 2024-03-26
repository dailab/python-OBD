# Command Lookup

`OBDCommand`s are objects used to query information from the vehicle. They contain all of the information necessary to perform the query and decode the car's response. Python-OBD has [built in tables](Command Tables.md) for the most common commands. They can be looked up by name or by mode & PID.

```python
import obd

c = obd.commands.RPM

# OR

c = obd.commands['RPM']

# OR

c = obd.commands[1][12] # mode 1, PID 12 (RPM)
```

The `commands` table also has a few helper methods for determining if a particular name or PID is present.

---

### has_command(command)

Checks the internal command tables for the existance of the given `OBDCommand` object. Commands are compared by mode and PID value.

```python
import obd
obd.commands.has_command(obd.commands.RPM) # True
```

---

### has_name(name)

Checks the internal command tables for a command with the given name. This is also the function of the `in` operator.

```python
import obd

obd.commands.has_name('RPM') # True

# OR

'RPM' in obd.commands # True
```

---

### has_pid(mode, pid)

Checks the internal command tables for a command with the given mode and PID.

```python
import obd
obd.commands.has_pid(1, 12) # True
```

---

<br>
