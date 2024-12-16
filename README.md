# OPET_control
 
Multiple OPET loads connect to a single RS485 bus. This library provides an `OPETBus` object for the bus and an `OPET` object for each load on the bus.

**The best place to get started is in the Jupyter notebook `examples/basic_operations.ipynb`.**

The library has incomplete coverage of the OPET hardware's features. We'll add to it as we go.

The library has incomplete documentation. For now, refer to the OPET docx manual for information about the load.

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