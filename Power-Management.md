_[<<< Back to Configuring Version 0.99 Main Page](Configuring-Version-0.99)_<br>
_**Applies to fb 100.11 and above, which was pushed on 9/18/16**_ <br>

Power management is used to keep the steppers on when you need them and turn them off when you don't.

Stepper motors consume maximum power when idle. They hold torque and get hot. If you shut off power the motor has (almost) no holding torque. Most lead screw or geared machines will hold position if you shut off the power when the motor is not moving, but belt machines generally do not.

In either case it's usually a good idea to keep all motors powered during a machining cycle to maintain absolute machine position. Depending on your machine you either may want to remove power some number of seconds after the machining cycle, or leave the motors powered on for as long as the machine is on.

You also generally want to use power management to de-power the machine if it's left unattended for an extended time. You don't want to leave the steppers on for extended idle times such as walking away from your machine and leaving it on overnight with the motors idling.

The power management commands let you set up the right set of actions for your machine and use.

## Global Power Management Settings
These settings affect all motors.

Setting | Description | Notes
--------|-------------|-----------------------------
[{me:...}](#me-motor-enable) | Enable motors | {md:0} to enable all motors for default timeout (mt). Set non-zero value in seconds to override default timeout
[{md:...}](#md-motor-disable) | Disable motors | {md:0} to disable all motors. Set 1-6 to disable motor N only
[{mt:...}](#mt-motor-timeout) | Motor timeout | Default timeout in seconds
[{pwr1:n}](#pwr1n-power-level-readout) | Power level | Current power level for motor (read-only)
[{pwr:n}](#pwrn-power-level-group) | Power level group | Power levels for all motors (read-only)

JSON mode and Text mode examples:
<pre>
{me:180}      $me=180     Lock all motors for 3 minutes for a tooling operation
{md:n}        $md         Disable all motors
{mt:900}      $mt=900     Set global timeout to 15 minutes
{pwr:n}       $pwr        Query motor power level for all motors
{pwr2:n}      $pwr2       Query motor power level for motor 2
</pre>

### {me:...} Motor Enable
This will turn on any motor that is not disabled (i.e. `{1pm:0}`). Providing `{me:0}` will enable the motors for the timeout specified in the `mt` value. If a non-zero value is provided it will enable the motors for that many seconds. This is useful when attempting manual operations such as tool changes to ensure that the motors do not de-energize before the operation is complete. `{md:0}` can be used to disable the motors once the operation is complete. This value is "write-only" and returns an error if read (`{me:n}`).

### {md:...} Motor Disable
Providing `{md:0}` will disable all motors that are not permanently enabled (i.e. `{1pm:1}`). If a valid motor number is provided instead of zero it will disable that motor only (excepting permanently enabled motors). This value is "write-only" and returns an error if read (`{md:n}`).

### {mt:...} Motor Timeout
Sets the number of seconds before a motor will shut off automatically. Maximum about 4.2 million seconds, or about 7 weeks (1 dog-year). When the timeout starts is set by the [per-motor setting](#per-motor-settings):

- `{1pm:0}` Disabled motors are always off. They do not time out, and cannot be enabled using `me`
- `{1pm:1}` Always-on motors are always on. They do not time out, and cannot be disabled using `md`
- `{1pm:2}` Motors that are powered-in-cycle begin timeout at the end of the cycle, which is when the last motor stops moving. A new cycle before timeout occurs will re-enable the motors restart the timeout at the end of the new cycle.
- `{1pm:3}` Motors that are powered-while-moving begin timeout at the end of their movement. New movement before timeout occurs will re-enable the motors and restart the timeout at the end of the movement.

Motor timeouts are suspended during feedholds - i.e. they will not time out during a feedhold. This allows changing or adjusting tools without loss of position.

### {pwr1:n} Power Level Readout
Returns the power level from 0.0 to 1.0 that is currently being applied to the motor. If the motor is not energized it will be 0, otherwise it should the power level set for that motor. Read-only.

### {pwr:n} Power Level Group
Returns the power levels for all motors. Read-only.


## Per-Motor Settings

Setting | Description | Notes
------------------|-------------|-----------------------------
___________| |
[{1:{pm:n}}](#1pm-motor-power-modes) | Display power mode | Returns one of the power modes below
[{1:{pm:0}}](#1pm0-motor-disabled) | Disabled | Motor will not run, and is not enabled by `{me:N}`
[{1:{pm:1}}](#1pm1-motor-always-powered) | Always powered | Motor always powered and is not disabled by `{md:N}`
[{1:{pm:2}}](#1pm2-motor-powered-in-cycle) | Powered in cycle | Motor is powered during machining cycle (any axis is moving) and for `{mt:N}` seconds after cycle stops
[{1:{pm:3}}](#1pm3-motor-powered-when-moving) | Powered when moving | Motor is powered when it is moving and for `{mt:N}` seconds afterwards. Motors in this state can disable themselves during cycles if timeout is less than cycle time.

JSON mode and text mode examples:
<pre>
{1pm:0}     {1:{pm:0}}    $1pm=0     Motor 1 disabled
{2pm:1}     {2:{pm:1}}    $2pm=1     Motor 2 always powered
{1pm:2}     {1:{pm:2}}    $1pm=2     Motor 1 powered during a machining cycle (any motor moving)
{4pm:3}     {4:{pm:3}}    $4pm=3     Motor 4 only powered when it is moving
</pre>

These commands affect all motors and take effect as soon as they are issued.

### {1:{pm:...}} Motor Power Modes

#### {1:{pm:0}} Motor Disabled
This will turn off motor power and prevent the motor from turning on. Disabling the motor will prevent that axis from participating in any move. The motor will not be affected by `{me:N}` or `{md:t}` commands. This setting takes effect immediately.

#### {1:{pm:1}} Motor Always Powered
This will turn on motor power and leave it on until the board is shut down. The motor will not be affected by `{me:N}` or `{md:t}` commands. You generally do not want to use this mode as it will leave the motors on for extended periods of time if you do not power down the machine. Better to set a long motor power timeout and use Powered In Cycle (2)

#### {1:{pm:2}} Motor Powered In Cycle
This will turn on the motor power at the start of any move on any axis (a "cycle"), and will de-energize the motors `{mt:N}` seconds after the cycle is complete (i.e. all motion stops). The timeout interval is set by the `{mt:N}` value (see [Global Power Management Commands](#global-power-management-commands)).

#### {1:{pm:3}} Motor Powered When Moving
This will turn on the motor power only when that axis is moving, and will remove power `{mt:N}` seconds after that axis stops moving.
