# OPET_control

[![License: BSD 3-Clause](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](LICENSE)
[![Python >= 3.10](https://img.shields.io/badge/Python-≥3.10-blue.svg)](https://www.python.org)

Python interface for controlling **OPET** (Open PV Electrical Tool) photovoltaic electronic loads over RS485.

Multiple OPET loads connect to a single RS485 bus. This library provides an `OPETBus` object for the bus and an `OPET` object for each load on the bus.

## Installation

```bash
pip install .
```

To include dependencies for calibration (`numpy`, `scipy`):

```bash
pip install ".[calibration]"
```

## Quick start

```python
import serial
from OPET_control import OPETBus, OPET

# Open the RS485 serial connection
ser = serial.Serial('/dev/ttyUSB0', baudrate=115200, timeout=1)

# Create the bus and a load object (address set by on-board jumpers, 0-31)
bus = OPETBus(ser)
opet = OPET(bus, address_integer=5)

# Read identification and a measurement sample
print(opet.identification)
print(opet.sample)

# Set mode to maximum power point tracking and enable output
opet.mode = 'mppt'
opet.output_enabled = True

# Take a single measurement
print(opet.sample)

# Run an I-V curve
opet.start_iv_curve(delay=True)
print(opet.iv_data)
```

See [`examples/basic_operations.ipynb`](examples/basic_operations.ipynb) for a more complete walkthrough.

## API reference

<details>
<summary><strong>OPET properties and methods</strong></summary>

### Measurement

| Member | Type | Description |
|---|---|---|
| `sample` | property | Dictionary with a single measurement (voltage, current, power, temperatures, status, etc.) |
| `start_iv_curve(delay=False)` | method | Requests an I-V curve measurement. If `delay=True`, blocks until complete. Returns estimated duration in seconds. |
| `iv_data` | property | Stored I-V curve data (`voltage`, `current` lists) from the latest measurement |

### Load control

| Member | Type | Description |
|---|---|---|
| `mode` | get/set | Load mode: `'off'` (0), `'voc'` (1), `'isc'` (2), `'vset'` (3), `'cset'` (4), `'mppt'` (5) |
| `output_enabled` | get/set | Enable or disable the load output (`True`/`False`) |
| `fixed_voltage` | get/set | Fixed voltage setpoint (used when mode is `'vset'`) |
| `fixed_current` | get/set | Fixed current setpoint (used when mode is `'cset'`) |

### Range control

| Member | Type | Description |
|---|---|---|
| `voltage_range` | property | Present voltage range, in V |
| `current_range` | property | Present current range, in A |
| `voltage_ranges` | property | All voltage ranges available for this load (dict) |
| `current_ranges` | property | All current ranges available for this load (dict) |
| `voltage_range_index` | get/set | Index of the voltage range. Set to `'auto'` for auto-ranging. |
| `current_range_index` | get/set | Index of the current range. Set to `'auto'` for auto-ranging. |
| `ranges` | property | Currently active (voltage, current) range as a tuple |

### ADC configuration

| Member | Type | Description |
|---|---|---|
| `n_adc_average_vc` | get/set | Number of voltage/current measurements averaged per cycle |
| `n_adc_cycles_vc` | get/set | Number of cycles averaged for voltage and current |
| `n_adc_average_other` | get/set | Number of non-V/I measurements averaged per cycle |

### Analog regulator

| Member | Type | Description |
|---|---|---|
| `gain_proportional` | get/set | Proportional gain resistor index (0–3). Higher = slower but more stable. |
| `gain_integral` | get/set | Integral gain capacitor index (0–3). Higher = slower but more stable. |

### Device info and status

| Member | Type | Description |
|---|---|---|
| `identification` | property | Dict with `hardware_version`, `software_version`, `serial_number` |
| `address_integer` | property | The OPET's bus address (0–31) |
| `hardware_configuration` | property | `'HC'` (high-current) or `'LC'` (low-current) |
| `status` | property | Human-readable dict of status flags |
| `status_integer` | property | Raw status register value |
| `available` | property | `True` if the OPET is ready for communication |
| `available_time` | get/set | Datetime before which no communication should be attempted |

### Low-level

| Member | Type | Description |
|---|---|---|
| `reset()` | method | Resets the OPET (300 ms delay) |
| `send_verify(message, ...)` | method | Send a raw command and verify the reply |
| `read_eeprom(address)` | method | Read an EEPROM address |
| `write_eeprom(address, value)` | method | Write to an EEPROM address |
| `operation_complete()` | method | Check if the OPET has finished its current operation |

### Calibration (used by `calibrate.py`)

| Member | Type | Description |
|---|---|---|
| `activate_voltage_calibration_mode()` | method | Enter voltage calibration mode (readings in counts) |
| `activate_current_calibration_mode()` | method | Enter current calibration mode (readings in counts) |
| `calibration_scale` | get/set | Scale factor for the active calibration range |
| `calibration_offset` | get/set | Offset for the active calibration range |

</details>

## Calibration

Calibration requires a Fluke 5522A calibrator and the separate [`fluke5522a_calibrator`](https://github.com/NatLabRockies/fluke5522a) package. Install with calibration dependencies:

```bash
pip install ".[calibration]"
pip install fluke5522a_calibrator   # or: cd ../fluke5522a && pip install .
```

See [`examples/opet_calibration_simple.ipynb`](examples/opet_calibration_simple.ipynb) for the step-by-step calibration notebook.

<details>
<summary><strong>Calibration hardware setup</strong></summary>

### Connecting PC to OPET and calibrator

The calibrator requires 30 minutes of warmup time, so plan to allow time for it to start up.

Use an RS485 to USB cable with a null adapter and a gender changer to connect your PC to the Fluke 5522A calibrator. Connect the OPET to your PC using the RS485 to USB cable.

Connect the 24V DC power to the OPET.

Now set up the calibrator. Follow on-screen instructions to run 0 cal if prompted.

Use your PC to find which COM port (or `/dev/tty*` device) the calibrator and OPET are connected to. Edit the example notebook to change the referenced ports. If the notebook throws an error like:

```
SerialException: could not open port 'COM6': FileNotFoundError(2, 'Access is denied.', None, 2)
```

Check that the Fluke 5522A is set to communicate via serial (not GPIB), and that no other process is using the port.

### Connecting OPET to calibrator

**Voltage calibration:** Connect the voltage output of the calibrator to the voltage sense of the OPET. Connect (+) to (+) and (-) to (-), connect V- to S. At NLR this is done with a cable labeled ALL OPET VOLTAGE.

**LC current calibration:** Connect the low current output (+) of the calibrator to the external MOSFET port position S and the low current output (-) to the voltage S input on the OPET.

**HC current calibration:**
1. Connect the low current output (+) of the calibrator to the external MOSFET port position S and the low current output (-) to the PV curr C- input on the OPET.
2. Connect the high current output (+) of the calibrator to the external MOSFET port position S and the high current output (-) to the PV curr C- input on the OPET.

### Running calibration

Check the OPET serial bus address against the jumper settings on the circuit board. The address is an integer from 0 to 31, set using jumpers on the board.

For the first calibration of the day you may want to run the script without updating cal constants to verify that the new constants are only slightly different. However, when using the script, **ensure that `update_calibration_constants` is set to `True`** in order to actually write the calibration to the OPETs.

The **Current (high ranges)** section does not need to be run for LC OPETs, but **Current (low ranges)** needs to be run for both low and high current OPETs.

</details>

## Disclaimer

NREL/Alliance for Sustainable Energy, LLC/DOE disclaim all warranties, express or implied, including the warranties of merchantability or fitness for a particular purpose, and make no warranty as to the accuracy, completeness, or usefulness of any information provided herein. Use of this package is at the user's own risk.
