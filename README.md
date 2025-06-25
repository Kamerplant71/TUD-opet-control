# OPET_control
 
## About
Multiple OPET loads connect to a single RS485 bus. This library provides an `OPETBus` object for the bus and an `OPET` object for each load on the bus.

**The best place to get started is in the Jupyter notebook `examples/basic_operations.ipynb`.**

The library has incomplete coverage of the OPET hardware's features. We'll add to it as we go.

The library has incomplete documentation. For now, refer to the OPET docx manual for information about the load.
## Set up
### Dependency list
see below for more detailed install instructions
fluke5522a_calibrator
pyserial
numpy
scipy

If desired set up an anaconda environment and activate it:
```
conda create --name OPETcalibration pyserial numpy scipy
conda activate OPETcalibration
```

otherwise install pyserial, numpy, and scipy

Then use pip to install both of the git modules with python
```
cd opet-control
pip install .
cd ../fluke5522a
pip install .
```

## Using a jupyter notebook with conda env
If not using a conda env skip this section
### Option 1 GUI + no new jupyter install
Set up a kernel using these commands:
```
conda install ipykernel --name OPETcalibration
python -m ipykernel install --user --name OPETcalibration --display-name "Python (OPETcalibration)"
```

When this is done open jupyter notebook using anaconda navigator. Open the example notebook and run the kernel Python (OPETcalibration) in order to run 
### Option 2 (requires admin rights)
Install jupyter within the environment 
```
conda install jupyter --name OPETcalibration
```
open jupyter notebook with in the active conda environment (either by selecting the environment in the gui or running from an active anaconda shell)

## Connecting PC to OPET and Calibrator
The calibrator requires 30 minutes of warmup time, so plan to allow time for it to start up. \n
Use an rs485 to usb cable with a null adapter and a gender changer to connect your PC to the fluke 5522a calibrator. Connect the OPET to your pc using the modbus to usb cable. \n 
Connect the 24V dc power to the OPET.

Now set up the calibrator. Follow on screen instructions to run 0 cal if prompted.

Use your PC to find which com port the calibrator and OPET are connected to. Edit the example notebook to change the referenced COM ports. If the notebook will not connect to the calibrator and throws an error like:
```
SerialException: could not open port 'COM6': FileNotFoundError(2, 'Access is denied.', None, 2)
```
Check that the fluke 5522a is set to communicate via serial and not GPIB
Alternatively see if any other processes may be using the COM port
## Connect OPET to Calibrator
When running a voltage calibration: connect the voltage output of the calibrator to the voltage sense of the OPET. Connect (+) to (+) and (-) to (-), connect the V- to S (this is the ALL OPET VOLTAGE cable).

When running a LC current calibration: connect the low current output (+) of the calibrator to the external MOSFET port position S and the low current output (-) to the voltage S input on the OPET.

When running a HC current calibration: 
1. connect the low current output (+) of the calibrator to the external MOSFET port position S and the low current output (-) to the PV curr C- input on the OPET. 
2. connect the high current output (+) of the calibrator to the external MOSFET port position S and the low current output (-) to the PV curr C- input on the OPET. 

## Running Calibration
Check the OPET EEPROM address against the sticker on the board, when running the cal routine you must write the correct OPET address for use with the modbus and you must imput the correct EEPROM address in order for the calibration routine to work properly.

For the first calibration of the day you may want to run the script without updating cal constants to verify that the cal constants are only slightly different. However when using the script *ensure that update_calibration_constants is set to True* in order to actually write the calibration to the OPETs

The high current range does not need to be run for LC OPETS but the low range script should be run for high current OPETs.

```
Help on OPET in module OPET_control.OPET_control object:

class OPET(builtins.object)
 |  OPET(opet_bus, address_integer)
 |  
 |  Represents a single OPET
 |  
 |  Methods defined here:
 |  
 |  __init__(self, opet_bus, address_integer)
 |      `opet_bus` is an instance of OPETBus and address_integer is the
 |      OPET's address as set with the on-board jumpers.
 |  
 |  activate_current_calibration_mode(self)
 |  
 |  activate_voltage_calibration_mode(self)
 |  
 |  operation_complete(self)
 |      # @property
 |  
 |  parse_system_status_integer(self, integer)
 |      Parses the system status integer, returned as the `status` item
 |      of `get_sample()` or as the .status property, into a human-readable
 |      dictionary.
 |  
 |  read_eeprom(self, address)
 |      Returns the contents of an EEPROM `address`
 |  
 |  reset(self)
 |  
 |  send_verify(self, message, raw_reply=False, skip_verify=False, check_availability=True, block_until_available=False)
 |      This is a wrapper for the OPETBus's send_verify method.
 |      
 |      If `check_availability`, this checks if the OPET's `available_time`
 |      property is in the future. If it is, we raise NotAvailableError,
 |      except if `block_until_available` is also true, in which case we wait
 |      until `available_time` before attempting communication.
 |  
 |  start_iv_curve(self, delay=False)
 |      Requests an I-V curve measurement. The actual measurement is read
 |      back using the .iv_data property. Returns an estimated time, in ms, for
 |      the curve to complete. If `delay` is True, this blocks for this amount
 |      of time.
 |      
 |      If this method returns zero, the measurement has not been started.
 |      Possible reasons include:
 |      - bias voltage error is active
 |      - probably others
 |      
 |      After the curve is started, the unit can receive messages, but can't
 |      respond until the curve is finished. This method automatically sets
 |      the `available_time` property to the datetime when the measurement is
 |      expected to be finished. It is suggested that communication not be
 |      attempted before `available_time` is reached.
 |  
 |  ----------------------------------------------------------------------
 |  Readonly properties defined here:
 |  
 |  address_integer
 |  
 |  available
 |  
 |  current_range
 |      Present current range, in A
 |  
 |  current_ranges
 |      Current ranges, in A, available for this load
 |  
 |  hardware_configuration
 |      'HC' for the high-current configuration and 'LC' for low-current
 |  
 |  identification
 |  
 |  iv_data
 |      Stored I-V curve data from the latest IV or transient measurement.
 |      The measurement is updated using `start_iv_curve()`.
 |  
 |  max_current_range_index
 |  
 |  max_voltage_range_index
 |  
 |  ranges
 |      Currently active (voltage, current) range, in a tuple
 |  
 |  sample
 |      Dictionary with a single measurement sample
 |  
 |  status
 |  
 |  status_integer
 |  
 |  voltage_range
 |      Present current range, in A
 |  
 |  voltage_ranges
 |      Voltage ranges, in V, available for this load
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables (if defined)
 |  
 |  __weakref__
 |      list of weak references to the object (if defined)
 |  
 |  available_time
 |  
 |  calibration_offset
 |  
 |  calibration_scale
 |  
 |  current_range_index
 |      Index of the current range. See load.current_ranges_all for values.
 |      'auto' is a special value that sets the load to auto-range.
 |  
 |  fixed_current
 |      The fixed current setting, which is only used when .mode is 4
 |      ('cset') and output is 1 (on)
 |  
 |  fixed_voltage
 |      The fixed voltage setting, which is only used when .mode is 3
 |      ('vset') and output is 1 (on)
 |  
 |  gain_integral
 |      Index of the capacitor that sets the integral gain of the analog
 |      PI regulator. Possible values are (0, 1, 2, 3). Increasing the value
 |      increases the value of the capacitor, making the loop slower, but more
 |      stable.
 |  
 |  gain_proportional
 |      Index of the resistor that sets the proportional gain of the analog
 |      PI regulator. Possible values are (0, 1, 2, 3). Increasing the value
 |      increases the value of the resistor, making the loop slower, but more
 |      stable.
 |  
 |  mode
 |      The load mode of the OPET, set as a string or integer and returned
 |      as an integer:
 |      'off': 0
 |      'voc': 1
 |      'isc': 2
 |      'vset': 3
 |      'cset': 4
 |      'mppt': 5
 |  
 |  n_adc_average_other
 |      Number of measurements other than voltage and current averaged
 |      per cycle
 |  
 |  n_adc_average_vc
 |      Number of voltage and current measurements averaged per cycle
 |  
 |  n_adc_cycles_vc
 |      Number of cycles averaged for current and voltage
 |  
 |  output_enabled
 |  
 |  voltage_range_index
 |      Index of the voltage range. See load.voltage_ranges for values.
 |      'auto' is a special value that sets the load to auto-range.
 ```
