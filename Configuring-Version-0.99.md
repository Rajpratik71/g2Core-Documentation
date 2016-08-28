**The configuration and settings documentation on this page is for firmware version 0.99.** 
The version number can be found as the fv variable in the startup JSON message, or by typing $fv. Version 0.99 encompasses builds 100.00 and later.

If you have an older firmware version:
* [Configuration for Firmware Version 0.98](Configuration-for-Firmware-Version-0.98)

## Configuration Pages

- [System Configuration Group](Configuring 0.99 System Groups)
- [Motor Configuration](Configuring 0.99 Motors)
- [Axis Configuration](Configuring 0.99 Axes)
- [Other Configuration Groups](Configuring 0.99 Other Groups)
- [Actions and Reports](Configuring 0.99 Actions and Reports)
- [3DP Extensions Configuration](Configuring 0.99 3DP Extensions)

##Conventions Used for Config Documentation
- Examples show relaxed JSON input (e.g. `{fv:n}`). Strict JSON is also accepted in all cases (e.g. `{"fv":null}`)
- CMD means some command - aka the "name" of the name/value pair. CMDs are case insensitive.
- Underscore "_" means some numeric value
- "abcd" means some string value
- Motor examples use Motor 1, but any motor active for your platform is OK
- Axis examples use X axis, but any axis is OK unless otherwise noted
- All examples are in millimeter units

**A note about units**
- All data entry and display occurs in the prevailing units set in the Gcode mode. Set G20 for inches, G21 for mm. Query {unit:n} or $unit to find out where you are (G20=0, G21=1)
- Units entered in one system are correctly preserved and do not need to be re-entered or adjusted if the Gcode units mode changes.
- The Gcode default units setting `{gun:_}` sets the units the system "comes up in" during power-on or reset. So changing GUN will not change your units until you reset. Note: PROGRAM END (M2, M30) does not change the units. 

### JSON Cheat Sheet
All configuration commands are available using [JSON mode](JSON-Operation), which is the preferred access method if you are writing a UI or controller. Most commands are also available in [Text Mode](Text-Mode). The few commands that are available from only one or the other are noted, as are any commands the behave differently depending on the mode. The rough equivalence is:
<pre>
{"CMD":n} == $CMD            Read value for command "CMD" in strict JSON mode
{CMD} == $CMD                Read value in relaxed JSON mode
{"CMD":123.4} == $CMD=123.4  Set value to 123.4 in strict JSON mode
{CMD:123.4} == $CMD=123.4    Set value in relaxed JSON mode
{xvm:50000}                  Set X max velocity to 50000 as an example
</pre>
You can also use JSON to read an entire object - examples for motor and axis:
<pre>
{1:n}
{"r":{"1":{"ma":0,"sa":1.800,"tr":40.0000,"mi":8,"po":0,"pm":2,"pl":0.375}},"f":[1,0,5]}

{x:n}
{"r":{"x":{"am":1,"vm":50000,"fr":50000,"tn":0.000,"tm":420.000,"jm":10000,"jh":20000,"jd":0.1000,"hi":1,"hd":0,"sv":3000,"lv":100,"lb":20.000,"zb":3.000}},"f":[1,0,5]}
</pre>
The footer is an array of 3 elements:
- (1) Footer revision
- (2) [Status code](Status-Codes) - Short version: status=0 is OK, everything else is an exception.
- (3) RX buffer info (explained later)
- Note: The checksum that used to be in the footer has been removed as no-one was using it.