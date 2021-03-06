Making pin changes in the code can allow you to do timing using a logic analyzer or a scope. You can use it for some minor debugging or debugging where the timing is hard to tell with a debugger.

Pin changes are *not* a substitute for a proper debugger.

## How to debug with pin changes in G2

1. Go to the motate_pin_assignments.h file and swap the pin you want to "steal from" with the debug pin you want. The path is something like:

  `TinyG2/platform/atmel_sam/v9_3x8c/motate_pin_assignments.h`

  Say spindle change:

  ```c++
  pin_number kCoolant_EnablePinNumber         = 116;
  ```
  to
  ```c++
  pin_number kCoolant_EnablePinNumber         = -1; // was 116
  ```

  __Warning__ -- don't use the input pins on a TinyG board for timing via pin changes. They have an R/C filter on them and will slew the output significantly. You should limit the debug pins to those that are normally outputs, and have a screw terminal or other pinout.

1. Assign one or more debug pins to pins you want to temporarily "take over".

  ```c++
  pin_number kDebug1_PinNumber                =  116; // using the coolant pin temporarily
  pin_number kDebug2_PinNumber                =  -1;
  pin_number kDebug3_PinNumber                =  -1;
  pin_number kDebug4_PinNumber                =  -1;
  ```

1. Then at the top of the file you want to instrument, put (or, more likely, uncomment) the `OutputPin<>` definition. For example, at the top of plan_exec.cpp now is:

  ```c++
  // This next "using" line just has to be in the file before you use OutputPin<>. It likely already is.
  // You don't want it in the file twice, or the compiler will bark at you.
  using namespace Motate;
  OutputPin<kDebug1_PinNumber> exec_debug_pin1;
  //OutputPin<kDebug2_PinNumber> exec_debug_pin2;
  //OutputPin<kDebug3_PinNumber> exec_debug_pin3;
  //OutputPin<-1> exec_debug_pin3;
  ```

1. Now, in that file, anywhere you want to set the pin:

  ```c++
  exec_debug_pin1 = 1;
  ```

  and the clear it:

  ```c++
  exec_debug_pin1 = 0;
  ```

  and to toggle the pin:

  ```c++
  exec_debug_pin1.toggle();
  ```

__PLEASE__ -- don't git commit the changes to the `motate_pin_assignments.h` file.

__ALSO__ -- avoid putting more than just the pin changes in. `if...then` statements around the pins can easily have side effects, and it you're just timing or doing light debugging, you don't want that.

### Tips:

- Make sure the that pin isn't already being used elsewhere as well.
Just search for the name: kDebug1_PinNumber (for example)
