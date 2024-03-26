# Debug Output

If python-OBD is not working properly, the first thing you should do is enable debug output. Add the following line before your connection code to print all of the debug information to your console:

```python
obd.logger.setLevel(obd.logging.DEBUG)
```

Here are some common logs from python-OBD, and their meanings:

<br>

### Successful Connection

```none
[obd] ========================== python-OBD (v0.4.0) ==========================
[obd] Explicit port defined
[obd] Opening serial port '/dev/pts/2'
[obd] Serial port successfully opened on /dev/pts/2
[obd] write: 'ATZ\r\n'
[obd] wait: 1 seconds
[obd] read: 'ATZ\rELM327 v2.1\r'
[obd] write: 'ATE0\r\n'
[obd] read: 'ATE0\rOK\r'
[obd] write: 'ATH1\r\n'
[obd] read: 'OK\r'
[obd] write: 'ATL0\r\n'
[obd] read: 'OK\r'
[obd] write: 'ATSPA8\r\n'
[obd] read: 'OK\r'
[obd] write: '0100\r\n'
[obd] read: '7E8 06 41 00 FF FF FF FF FC\r'
[obd] write: 'ATDPN\r\n'
[obd] read: 'A8\r'
[obd] Connection successful
[obd] querying for supported PIDs (commands)...
[obd] Sending command: 0100: Supported PIDs [01-20]
[obd] write: '0100\r\n'
[obd] read: '7E8 06 41 00 FF FF FF FF FC\r'
[obd] Sending command: 0120: Supported PIDs [21-40]
[obd] write: '0120\r\n'
[obd] read: '7E8 06 41 20 FF FF FF FF FC\r'
[obd] Sending command: 0140: Supported PIDs [41-60]
[obd] write: '0140\r\n'
[obd] read: '7E8 06 41 40 FF FF FF FE FB\r'
[obd] finished querying with 93 commands supported
[obd] =========================================================================
```

<br>

### Unresponsive ELM

```none
[obd] ========================== python-OBD (v0.4.0) ==========================
[obd] Explicit port defined
[obd] Opening serial port '/dev/pts/2'
[obd] Serial port successfully opened on /dev/pts/2
[obd] write: 'ATZ\r\n'
[obd] wait: 1 seconds
[obd] __read() found nothing
[obd] __read() found nothing
[obd] __read() never recieved prompt character
[obd] read: ''
[obd] write: 'ATE0\r\n'
[obd] __read() found nothing
[obd] __read() found nothing
[obd] __read() never recieved prompt character
[obd] read: ''
[obd] Connection Error:
[obd]     ATE0 did not return 'OK'
[obd] Failed to connect
[obd] =========================================================================
```

This is likely a problem with the serial connection between the OBD-II adapter and your computer. Make sure that:

- bluetooth devices have been paired properly
- you are connecting to the right port in `/dev` (or that there is any port at all)
- you have the correct permissions to write to the port

You can use the `scan_serial()` helper function to determine which ports are available for writing.

```python
import obd

ports = obd.scan_serial()       # return list of valid USB or RF ports
print ports                    # ['/dev/ttyUSB0', '/dev/ttyUSB1']
```

<br>

### Unresponsive Vehicle

```none
[obd] ========================== python-OBD (v0.4.0) ==========================
[obd] Explicit port defined
[obd] Opening serial port '/dev/pts/2'
[obd] Serial port successfully opened on /dev/pts/2
[obd] write: 'ATZ\r\n'
[obd] wait: 1 seconds
[obd] read: 'ATZ\rELM327 v2.1\r'
[obd] write: 'ATE0\r\n'
[obd] read: 'ATE0\rOK\r'
[obd] write: 'ATH1\r\n'
[obd] read: 'OK\r'
[obd] write: 'ATL0\r\n'
[obd] read: 'OK\r'
[obd] write: 'ATSPA8\r\n'
[obd] read: 'OK\r'
[obd] write: '0100\r\n'
[obd] read: 'SEARCHING...\rUNABLE TO CONNECT\r'
[obd] write: 'ATDPN\r\n'
[obd] read: '0\r'
[obd] Connection Error:
[obd]     ELM responded with unknown protocol
[obd] Failed to connect
[obd] =========================================================================
```

This is a connection problem between the ELM adapter and your car. Make sure that you car is powered, and that the electrical connection between the adapter and your car's OBD-II port is sound.

---

<br>
