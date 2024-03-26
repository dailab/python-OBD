# Debug

python-OBD uses python's builtin logging system. By default, it is setup to send output to `stderr` with a level of WARNING. The module's logger can be accessed via the `logger` variable at the root of the module. For instance, to enable console printing of all debug messages, use the following snippet:

```python
import obd

obd.logger.setLevel(obd.logging.DEBUG) # enables all debug information
```

Or, to silence all logging output from python-OBD:

```python
import obd

obd.logger.removeHandler(obd.console_handler)
```

---

<br>
