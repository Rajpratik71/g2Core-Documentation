_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_

**!!!! TAKE NOTE: The 3D printer extensions documented on this page are pre-release and experimental (hence the leading underscore). These settings will NOT be present in the release ultimate 3D printing releases. Please expect any UIs or code written to these specifications to require changes. The settings on this page will be removed and will not be backwards compatible. !!!!**

# Heater Groups
There are 3 heater groups. The examples below show heater group 1 - `{_he1:n}`

## Summary

Setting | Description | Notes
--------|-------------|-------
[{_he1:{e:...}}](#_he1e-heater-enable) | Heater Enable | 0=off, 1=on
[{_he1:{p:...}}](#_he1p-i-d-heater-pid-values) | Heater P (PID) | read/write
[{_he1:{i:...}}](#_he1p-i-d-heater-pid-values) | Heater I (PID) | read/write
[{_he1:{d:...}}](#_he1p-i-d-heater-pid-values) | Heater D (PID) | read/write
[{_he1:{st:...}}](#_he1st-setpoint-temperature) | Setpoint temperature | write-only
[{_he1:{t:...}}](#_he1t-current-temperature) | Current temperature | read-only
[{_he1:{at:...}}](#_he1at-at-temperature-flag) | "At temperature" flag | read-only
[{_he1:{op:...}}](#_he1op-pwm-output) | PWM output level | read-only
[{_he1:{tr:...}}](#_he1tr-thermistor-resistance) | Thermistor resistance | read-only
[{_he1:{an:...}}](#_he1an-heater-adc-reading) | Heater ADC reading | read-only
[{_he1:{fp:...}}](#_he1fp-fm-fl-fh-fan-controls) | Fan power | read/write
[{_he1:{fm:...}}](#_he1fp-fm-fl-fh-fan-controls) | Fan minimum power | read/write
[{_he1:{fl:...}}](#_he1fp-fm-fl-fh-fan-controls) | Fan low temperature | read/write
[{_he1:{fh:...}}](#_he1fp-fm-fl-fh-fan-controls) | Fan high temperature | read/write

### {_he1:{e:...}} Heater Enable

### {_he1:{p:..., i:..., d:...}} Heater PID Values

PID settings for heater feedback loop. Typical default values are:
```
For an extruder heater, 12volt:
  DEFAULT_P    7.0
  DEFAULT_I    0.05
  DEFAULT_D    150.0

For a heated bed:
  DEFAULT_P    9.0
  DEFAULT_I    0.12
  DEFAULT_D    400.0
```

### {_he1:{st:...}} Setpoint Temperature

### {_he1:{t:...}} Current Temperature

### {_he1:{at:...}} At-Temperature Flag

### {_he1:{op:...}} PWM Output

### {_he1:{tr:...}} Thermistor Resistance

### {_he1:{an:...}} Heater ADC Reading

### {_he1:{fp:..., fm:..., fl:..., fh:...}} Fan Controls

# PID Groups
There are 3 PID groups. The examples below show PID group 1 - `{_pid1:n}`

## Summary

Setting | Description | Notes
--------|-------------|-------
[{_pid1:{p:...}}](#_pid1p-i-d-pid-values) | Raw P Value | read-only
[{_pid1:{i:...}}](#_pid1p-i-d-pid-values) | Raw I Value | read-only
[{_pid1:{d:...}}](#_pid1p-i-d-pid-values) | Raw D Value | read-only

### {_pid1:{p:..., i:..., d:...}} Raw PID Values
These are Read-Only values from the live PID. They are not scaled to temperature

# Auto Bed Leveling Parameters

## Summary

Setting | Description | Notes
--------|-------------|-------
[{tram:t](#tramt-tram-command) | Tram Command | Set and enable tramming matrix

### {tram:t} Tram Command
Setting Tram will use the last three Probe values to define the plane of the Z axis. For operation, probe the bed (Z axis) at three disparate points, then invoke Tram.
