This page provides details of the internals of the dual endpoint USB function and the how G2 handles devices in general.

## General
The code is organized like so.
* The low-level devices are provided by Motate devices
* The device primitives are wrapped as C++ classes in xio.cpp/.h
* xio.cpp also provides higher level functions such as device / channel bindings and line reader semantics
* The interface presented to upper layers (i.e. the controller) provides a simple function that can be used to read the control and data channels from any type of device.

### xio.cpp Notes
_Notes from here out are unfinished. Please don't rely on them just yet_

Controller.cpp  controll_init():
```c++
// setup for USBserial0
SerialUSB.setConnectionCallback([&](bool connected) {
    usb0->next_state = connected ? DEVICE_CONNECTED : DEVICE_NOT_CONNECTED;
});
```
Yes, that's a lambda function, this might change later. Here's a good lambda function C++ reference:<br>
http://www.cprogramming.com/c++11/c++11-lambda-closures.html

This works exactly the same:
```c++
void changedStateCh0(bool connected) {
    usb0->next_state = connected ? DEVICE_CONNECTED : DEVICE_NOT_CONNECTED;
}

void controller_init(uint8_t std_in, uint8_t std_out, uint8_t std_err)
{
    ...
    SerialUSB.setConnectionCallback(changedStateCh0);
    ...
}
```

`Connected` is as it says, true for now connected, false for now disconnected. It should only get called on an edge — when it changes. You shouldn’t see two back-to-back connected=true calls with the same callback.


As for reading and writing: There are two `SerialUSB` objects: `SerialUSB`  (NO zero, but easily changed), and `SerialUSB1`. They are identical in function. From here on, when I say `SerialUSB`, I also mean `SerialUSB1` — they’re identical.

The interface is simple:
```c++
int16_t SerialUSB.readByte()

uint16_t read(uint8_t *buffer, const uint16_t length); // Blocking for now

int32_t write(const uint8_t *data, const uint16_t length); // Also blocking
```

There is also `readSome()` and `writeSome()` functions that are non-blocking, but otherwise have the same call syntax. read/readSome and write/writeSome all return the amount they read or wrote, which is possibly 0 in the Some cases.

We haven’t used the blocking `read()` yet, and the blocking write only blocks until another 256 (or 128 for dual USB) buffer becomes available and it can push the data to it. This is rarely very long.

`length` is required in all cases. I have fixed this in the Motate repo to allow `length==0` to means until the terminating `NULL`, but that’s not ported yet, and porting it’s not practical yet. So, always give a value length (`strlen(ptr)` works).

`readByte()` will return `-1` if there’s nothing to read. It’s been doing that this whole time, inside xio -> `read_char()`, which is in turn called by `read_line()`, where it’s called `_FDEV_ERR` and means the same thing.

I modified the `_write()` system call (that gets called from `printf`) to take file id == 1 as the `SerialUSB1`, and everything else as `SerialUSB`. It calls `SerialUSB.write()`, the blocking version.

## C++ Classes, Virtual Functions and Inheritance
This section provides an explanation of what's going on with the device classes, virtual functions and inheritance. It's intended for those who know C but not C++, or need a C++ refresher.

Starting at the beginning. We are constructing a C++ class for the primitive functions for a class. For this example we are only looking at reading characters from a device.

Each of the Motate devices have a distinct type, such as `SPI<0, 1, 2, 3>`, and `UART<...>`. This makes it impossible to put them in an array, much like you can't (easily) have an array that contains strings, `int`s, and `struct`s.

In order to put these diverse types in an array and treat them as equal types that can be called the same way, we will have to create an intermediate wrapper type base-class, and various subclasses for each of the device types.

Here's our base type (for simplicity, we only define one function in these examples):

```c++
struct xioDeviceWrapperBase {				// base class for the reading from a device
	virtual int16_t readchar() = 0;			// Pure virtual. Will be subclassed for every device
};
```

This class is the base class, or "parent class", and will be subclassed for each device type. It has a _pure virtual function_ that the subclasses will override to read characters from the device.

The `= 0` portion of the readchar() definition makes it a _pure virtual function_ and also makes `xioDeviceWrapperBase` an _abstract base class_, which means you cannot instantiate a `xioDeviceWrapperBase` object directly. However, you can have a pointer to a `xioDeviceWrapperBase`, and since you cannot instantiate one that means that it _must_ point to a subclass. (See this page on [C++ polymorphism](http://www.cplusplus.com/doc/tutorial/polymorphism/) for more info.)

Next we need a subclass for each type of object we want to handle. We are specifically interested in the objects (**NOT classes**) `SerialUSB` and `SerialUSB1`. These are global objects of the type `Motate::USBSerial<Motate::USBDevice<Motate::USBCDC, Motate::USBCDC> >`, which isn't very pretty. We can use a typedef to make that more accessible, and then create our specially-crafted base class:

```c++
typedef Motate::USBSerial<
          Motate::USBDevice<
            Motate::USBCDC,
            Motate::USBCDC
          >
        > SerialUSB_t;

struct xioDeviceSerialUSBWrapper : xioDeviceWrapperBase {
    SerialUSB_t* _dev;

    xioDeviceWrapper(SerialUSB_t* dev) {
      _dev = dev;
    };

    int16_t readchar() final {
        return _dev->readByte();
    };
};
```

Here we created a class `xioDeviceSerialUSBWrapper` that is a subclass of `xioDeviceWrapperBase`. It has one _member_, exactly like that of a C structure, that is a pointer to a `SerialUSB_t` called `_dev`.

We then define a _constructor_ for it, which is a convenient way to initilize the contents of the class when it's declared -- IOW, there's no need to create a seperate function that initializes it. This allows us to create a fully-formed and ready-to-use `xioDeviceSerialUSBWrapper` object in one line, at global scope if we wish:

```c++
xioDeviceSerialUSBWrapper serialUSBWrapper { &SerialUSB };
// This is the exact same, with different syntax:
// xioDeviceSerialUSBWrapper serialUSBWrapper = { &SerialUSB };

// This is almost the same (older syntax), with less type safety:
// xioDeviceSerialUSBWrapper serialUSBWrapper ( &SerialUSB );

// Any of those would work for this case, and yield the same result.
```

As you see, we have one parameter in our constructor, `SerialUSB_t* dev`, and we then use that value to assign to the value of `_dev`. So, in that last example, we created an object named `SerialUSBWrapper` that already has it's `_dev` set to a pointer to `SerialUSB`.

Next we defined a function `readchar()` that overrides the virtual function of the same name in the base class. We use the keyword `final` to indicate not only  that it's a virtual function, but also that it needs to override the same function in the base class _and_ that it's not to be overridden.

It would be called like this:

```c++
int16_t c = serialUSBWrapper.readchar();
```

From here there are a few things we can to do make the code more manageable and robust:
* We can make the wrapper class _templated_ so that it will work with any device class, regardless of actual type, as long as it has the calls we use. (Right now we _only_ use `readByte()`.)
* We can make the constructor more efficient by using a _member initialization list_ in the constructor -- for global objects like these the optimizer will often remove the constructor altogether, and initialize the structure like any other static value.

The following is the templated version of the same wrapper class, except it takes the device type by template parameter:

```c++
template<typename Device>
struct xioDeviceWrapper : xioDeviceWrapperBase {
    Device _dev;

    xioDeviceWrapper(Device  dev) : _dev{dev} {};

    int16_t readchar() final {
        return _dev->readByte();
    };
};
```
Starting with the template declaration:

```c++
template<typename Device>
struct …
```

This says that whatever structure is defined after struct will take a template parameter that is a typename and it’ll be called Device. Device can be any valid type, such as char *, int, const int, or even other templated structures. Think of it like a macro, except it’s not a brain-dead copy-and-paste that happens in the preprocessor, but it’s part of the compiler and it can deal with it intelligently. (Obviously, macros still have their place…)

We have one template parameter, `Device`. It is defined as `typename`, so it could be any valid type, from a pointer to a serial usb object to a char. **In this case we are expecting it to be a pointer to the device type, such as `SerialUSB_t*`.** (This'll come together in a second...)

When it comes time to compile, if we used `_dev` in a way that doesn’t make sense for a variable of type `Device` then we’ll get an error.

Now for the struct declaration:

```c++
template<typename Device>
struct xioDeviceWrapper : xioDeviceWrapperBase {
  …
};
```

So far this part is the same, except that our class name has changed from `xioDeviceSerialUSBWrapper` to `xioDeviceWrapper`.

Now for the contents of our class, starting with:
```c++
    Device _dev;
```
We declare a variable (called a member) of type `Device` and name it `_dev`. Remember in our `xioDeviceSerialUSBWrapper` class we had a similar definition:

```c++
SerialUSB_t* _dev;
```

Well, if we construct an object with `Device` set to `SerialUSB_t*`, then it will be result in the _exact same thing_:

```c++
// The specific (non-templated class):
xioDeviceSerialUSBWrapper serialUSBWrapper { &SerialUSB };

// The templated class, which is functionally identical:
xioDeviceWrapper<SerialUSB_t*> anotherSerialUSBWrapper { &SerialUSB };

```



Here is our new constructor, which changes names because the constructor is always the name of the class:

```c++
    xioDeviceWrapper(Device  dev) : _dev{dev} {};
```

Here we use the _member initialization_ special syntax for simple and efficient member initialization. Starting after the : but before the body of the function `{…}`; In this case it’s `_dev{dev}` which basically says `_dev = dev`. _(It could have been written `_dev(dev)` as well, but the `_dev{dev}` construction is a C++11 addition that provides more type safety, among other things.)_


We no longer have anything else to do in this constructor, so we define the body here and the body is empty: `{};`

Now we the `readchar()` function, which stays the exact same as in the non-templated version:

```c++
    int16_t readchar() final {
        return _dev->readByte();
    };

```

However, since we we passed `Device`, the type of `_dev`, we could have a compile time error. For example, if we were passed a string:

```c++
xioDeviceWrapper<char *> anotherSerialUSBWrapper {"blah"}; // ERROR
```

We would get an error, since we cannot call `"blah"->readByte()` -- it doesn't make sense.

####More about Virtual Functions and Inheritance

Inheritance without virtual functions is basically the same as dumping the parent class definitions into the child class definitions. The child class can then use the variables and call the methods of the parent class as if they had been defined in the child class directly.

There’s the added benefit that the compiler knows that they are related, so a pointer to the base class can point to the subclasses as well.

However, that’s not worth much without virtual functions. Since the pointer refers to the base class, calling a method of that pointer will use the base-class function.

For example, if we have a type called Shape and subclasses for various shapes (Circle, Rectangle, etc), we can to this:

```c++
struct Shape {
    void draw() {};
};
struct Circle : Shape {
    void draw() {};
};

Shape *nextShapeToDraw;

void drawNextShape() {
    nextShapeToDraw->draw();
}

void main() {
  Circle c;

  nextShapeToDraw = &c;

  nextShapeToDraw->draw(); // ERROR: draws a Shape!!
}
```

`drawNextShape()` will always call `Shape::draw()`, since `nextShapeToDraw` is a pointer to a `Shape` object, even if it’s actually pointing to a `Circle`.

<That's a bit confusing. We might need to walk through this. So I assume that the pointer to the Shape object set up in drawNextShape point to the base class? It might be clearer to call it ShapeBase or ShapeParent. Is there a C++ naming convention that differentiates the parents from the subclasses?>

Virtual functions fix that:
<pre>
struct Shape {
    virtual void draw() = 0; // Error if called directly
};
struct Circle : Shape {
    void draw() {}; // virtual implied here
};

Shape *nextShapeToDraw;

void drawNextShape() {
    nextShapeToDraw->draw();
}
</pre>
Now if nextShapeToDraw is assigned to the pointer of a Circle object, it will call Circle::draw().
