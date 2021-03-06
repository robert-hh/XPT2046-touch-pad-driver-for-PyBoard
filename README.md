#touch pad Class for the XPT2046 controller

**Description**

A Python class for using a resistive touch pad with a XPT2046 controller. This port uses a software SPI for communication to the TFT, whic uses the following GPIO ports:

- X12 for Clock
- X11 for Data Out (from Pyboard to XPT2046)
- Y2 for Data In  (from XPT2046 to Pyboard)

CS of the touch pad must be tied to GND. The touch pad will typically uses in combination with a TFT. In my case, it was glued to a 4.3" TFT with an SSD1963 controller. The class itself does not rely on an TFT, but for calibration a TFT is used.

At the moment, the code is a basic package. It will deliver touch events. But the mapping to a TFT's coordinates is tested with that single TFT in Landscape mode only.

The touch pad is used in 12 bit mode, returning values in the range of 0..4095 for the coordinates. A single raw sampling takes about 60µs. The result is somewhat noisy, and the touch pad has the habit of creating false touches in the transition of press and release. The function get_touch caters for that.


**Functions**
```
Create instance:

mytouch = TOUCH(controller, asyn=False, *, confidence=5, margin=50,
          delay=10, calibration=None, spi=None)
    controller: String with the controller model. At the moment, it is ignored
    asyn: Set True if asynchronous operation intended. In this instance the
        uasyncio library must be available.
    confidence: confidence level - number of consecutive touches with a
        margin smaller than the given level which the function will sample
        until it accepts it as a valid touch
    margin: Difference from mean centre at which touches are considered
        at the same position
    delay: Delay between samples in ms. (n/a if asynchronous)
    calibration: Tuple of 8 numbers, which transpose touch pad coordinates
        into TFT  coordinates.
        You can determine these with the tool calibraty.py (see below) of the
        package. A vector of (0, 1, 0, 1, 0, 1, 0, 1) will deliver the raw
        values.  The calibration is performed typically once. Once determined,
        you may also code these values into the sources.
    spi: A spi object which is used for communiation. If None is supplied, the
        driver creates this object with the pins X12, X11 and Y2

Methods:

touch_parameter(confidence=5, margin=10, delay=10, calibration=None)
    # Set the operational parameters of the touch pad. All parameters are optional
    confidence: confidence level - number of consecutive touches with a
        margin smaller than the given level which the function will sample
        until it accepts it as a valid touch
    margin: Difference from mean centre at which touches are considered
        at the same position
    delay: Delay between samples in ms. (n/a if asynchronous)
    calibration: Tuple of 8 numbers, which transpose touch pad coordinates
        into TFT  coordinates.

get_touch(initial=True, wait=True, raw=False, timeout=None)
    # This is the major data entry function. Parameters:
    initial: if True, wait for a non-touch state of the touch pad before getting
        the touch coordinates. This is the natural behavior. If False, get the next touch
        immediately and do not what for the stylus to be released.
    wait: If True, wait for a valid touch event. If False, return immediately if no
        touch is made.
    raw: Setting whether raw touch coordinates (True) or normalized ones
        (False) are returned setting the calibration vector to
        (0, 1, 0, 1, 0, 1, 0, 1) result in a identity mapping
    timeout: Timeout for the function, unit ms, for all situations where the function is
        told to wait, e.g. initial = True or wait = True.
        A value of None is considered as a timeout of an hour.

    The function returns a two value tuple (x,y) of the touch coordinates,
    or 'None', if either no touch is pressed or the timeout triggers.

do_normalize(touch)
    # Transpose touch coordinates into TFT coordinates. The function requires
      the calibration values to be set to a reasonable value. It is called
      within get_touch too. Parameter:
    touch: a touch pad value tuple returned by get_touch() in raw mode
        or raw_touch()
----- lower level functions ---

raw_touch()
    # determine the raw touch value and return immediately. The return value is a pair of
      touch pad coordinates, is a touch is present, or 'None'

touch_talk(command, bits)
    # send commands to the touch pad controller and retrieves 'bits' data from it.
      It will always perform and return. No checking is done for the command value
      and the returned information.
```

**Files:**
- touch.py: Source file with comments.
- calibration.py: Code to determine the calibration of the touch pad, which
allows to map between touch pad and screen coordinates. You will be asked
to touch four points at the screen indicated by a cross-hair.
The confidence level is set high, so keep your hand steady and use a stylus.
If it fails at a certain point, release and touch again.
The determined values are printed on the screen and at the USB interface.
So you can copy them from there. Once the values are know, they are set
temporarily, and you may try them. Just touch the screen. At the point of
touching, a small green circle should light up. If the match is bad,
repeat the calibration.
- touchtest.py: Another sample test program, which creates a small four button
keypad, which is defined by a table.
- README.md: this one
- LICENSE: The MIT license file

**To Do**
- consider ISR mode
- test portrait mode

**Short Version History**

**0.1**
Initial release with the basic functions

**1.0**
Added an asynchronous mode implemented by Peter Hinch

**1.1**
Asynchronous mode adapted to use uasyncio

**1.2**
Replace bit-bang communication by SPI built-in methods, making it less hardware
dependent.
