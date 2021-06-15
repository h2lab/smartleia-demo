
## Quickstarting LEIA-Solo

Congratulation ! You have received your LEIA-Solo board(s) ! Let's start with the overall tooling and board description.


Here you will get the essentials to know in order to get your  first steps with your LEIA Solo Board.
Details about the hardware, software and firmware are documented [here](https://h2lab.github.io/smartleia.github.io/quickstart.html).


### Firmware

The firmware API is fully specified [here](https://h2lab.github.io/smartleia.github.io/c/test.html) and can be accessed using the python library of the smartleia library.

This library is packaged for Debian (see https://h2lab.github.io/smartleia.github.io/installation.html) and can be installed using pip.

### Software

The smartleia library expose an efficient smartcard oriented interface in order to communicate with the target.
This interface is fully documented [here](https://h2lab.github.io/smartleia.github.io/api.html).

Don't hesitate to take a look to the overall [LEIA-Solo documentation and tooling informations](https://h2lab.github.io/smartleia.github.io/index.html).

### Hardware

Smart card interface and signal acquisition Side-channel attacks are divided into two phases: signal acquisition and signal statistical analysis. The quality of the acquisition of side-channel
signal is essential to get proper results during the analysis phase. 
This is why the acquisition circuitry has been designed with great care.

On LEIA the leakage signal used for the side channel signals
capture is the power consumption in the form of current sensing on the
power lines.

## Measurements

### Board pinout

{{< image-resize "http://www.ha2lab.org/images/devices/leia_pinout.png" >}}


### Testing points

{{< image-resize "http://www.ha2lab.org/images/devices/leia_test_points_top.png" >}}

{{< image-resize "http://www.ha2lab.org/images/devices/leia_test_points_bot.png" >}}

### Current sense
When sensing current, the designer can choose to place
the sensors (usually a serial resistor on the power line) either between the
supply voltage (VCC) and load, or between the load and ground. The
former is called high-side sensing whereas the latter is called low-side
sensing.

* **High-side sensing** has the advantage that the load is directly connected
to the ground GND. In other words, there is no change on the load side
except a small power drop due to the current sensor on the VCC line.
Nevertheless the main disadvantage is that we have to use a differential
probe to measure the current. 
* In **low-side sensing** the current is sensed in
the ground return path (GND) of the power line to the monitored load.
This has the advantage to produce a ground referenced measured signal
but the load is no more directly connected to GND.

In our design the target, the LEIA boards and the controling device like the Chipwhisperer can have **separate power domains**, since we do not want the noise produced by the power switching supply the controling device to be visible in the measurements. **We choose to use a low-side measurement** as it minimizes the amount of hardware to support all the power domains.


### Shunt Resistor
The shunt resistor is the element that is used in a
circuit to redirect currents around the measuring device. The addition of
a shunt resistor induces a voltage drop at the maximum current rating.
This is why the value of the Shunt Resistor must be selected carefully.
Important parameters include the resistance tolerance, the power rating
and the temperature coefficient:

 * The power rating indicates the amount of electrical power that
the resistor can dissipate at a given ambient temperature without
being damaged nor changing the resistor parameters.
 * The temperature coefficient describes the relative change of resistor
value according to the temperature.
 * Resistance tolerance is the accuracy the constructor guarantees on
the component’s characteristics.

ISO7816 Class A devices, which are the most power-consuming devices
among smart cards, can draw at most 160 mA for 400 ns and continuously
draw at most 60 mA. We want the voltage drop at maximum current to be
at most 50 mV for not disturbing nominal working of the TOE whatever
class tho TOE belongs to. 
Thus we decided to use a **0.1 Ω resistor** with
a tolerance of 1%, a temperature coefficient of 300 PPM/C and a power
rating of 100 mW. It is a widespread, easily available component that
meets the needs. This resistor, as it induces a maximum voltage drop far
from the limit, allows us to get clean measurements.
Connectors and measurement. 

{{< image-resize "/images/devices/leia_shunt_circuit.png" >}}


### Measurements bandwidth
In order to provide high quality measurements, we use SMA End Launch Connectors since they offer reliable broadband performance from DC to 18 GHz with low reflection and constant 50 Ω impedance.

Because we want you to be able to measure the leaking signal from the target without any bandwidth loss we have chosen to not embed a low noise-amplifier. This allow you to caracterise the leaking signal of your target and decide which amplifier to use to get the best result.

### Setting up measure Mode

In order to setup LEIA in the measure mode: 

 * Move the PRG1, PRG2, PRG3 (LEIA Solo < v1.4) and PRG4 to the LEIA position (1-2).
 * Remove the shunt bypass jumber if it is set.
 * Move the tearing jumper to the OFF position.
 * Setup the power source for the smartcard. We would advise an external "clean" power source for clean measurements. However, we are able to get proper traces with the USB-C power supply on the funcard. 

{{< image-resize "/images/devices/leia_traces_cheap_scope.jpeg" >}}

Testing setup : 

 * Trigger is set on EXT and connected to C8 which is configured in trigger mode 2 (Rising Slope 1V).
 * C4 configured in trigger mode 1 is set on CH1. The signal is toggling at every AES round (1V/DIV).
 * The Shunt signal is connected on CH2 (20mV/DIV). 
 * The sampling rate is set to 500MS/s with a 500us/div scale. 

### Setting up Funcard programming mode

In order to setup LEIA in the Funcard programming mode: 

* Move the PRG1, PRG2, PRG3 (LEIA Solo < v1.4) and PRG4 to the ISP position (2-3).
* Setup the shunt bypass jumber if not set.
* Move the tearing jumper to the OFF position.
* Setup the power source for the smartcard to enable the USB-C power supply on the funcard. 

### Regulatory compliance & handling

The LEA board is intended for use as a development measure platform. The board is an open system design, which does not include a
shielded enclosure. This may cause interference to other electrical or
electronic devices in close proximity. In a domestic environment, this product
may cause radio interference in which case the user may be required to take
adequate measures. In addition, this board should not be used near any medical
equipment or RF devices.

The board is sensitive to ESD. Hold the board only by its edges. After removing the board from its box, place it on a grounded, static-free surface.

The board can become hot, like any fan-less design, during continuous high CPU loads, mind its temperature while handling it.


## Validating your LEIA Solo with the Funcard target

The LEIA validating target is based on a Funcard. 

The [smartleia-target-funcard repository contains](https://github.com/h2lab/smartleia-target-funcard/) an AVR implementation of an unprotected 128-bit key AES as well as a dummy PIN verification algorithm for playing with the LEIA boards.

Please be sure to update the funcard using the "sh flash_funcard.sh" (this will compile the latest funcard code and upload it through LEIA), and then you are good to go to test the ``script-AES128-enc.py`` for testing AES encryptions.

The implementations have been tested on a 
[WB Electronics 64 Kbit ATMega chipcard](http://www.infinityusb.com/default.asp?show=store&ProductGrp=8).

You can use the ``pin_timing_attack.py`` script for testing a timing attack on a dummy "verify PIN" on the funcard (please note that when testing raw access to LEIA as you do, you are in the ``USE_LEIA=True`` case of the README).

### Building the Funcard firmware

Go to the [src/](src/) folder and run ``make``. You must have `gcc-avr`, e.g. from [avr-gcc](https://gcc.gnu.org/wiki/avr-gcc), and the `avr-libc` installed on your PC: these are usually
packaged with popular distros such as Debian or Ubuntu.
Make should create ``aes-<DDMMYY>-<HHMMSS>.hex`` and ``eedata-<DDMMYY>-<HHMMSS>.hex`` in the [src/build/](src/build/) folder. 

### Funcard firmware loading

Load the files ``eedata.hex`` (in EEPROM int.) and ``aes-<DDMMYY>-<HHMMSS>.hex`` (in FLASH) in the ATMega8515 component. You can for instance use the [**Infinity USB Unlimited**](http://www.infinityusb.com/default.asp?show=store&ProductID=11) Reader and IDE from WB Electronics for this step.

If you bought a **recent LEIA board (PCB version >= 1.4) with the flashing mode feature**, you can simply execute the local flashing script:

```
sh flash_funcard.sh
```
This will compile and push the firmware on the funcard inserted in your LEIA board.

### Testing scripts

The testing scripts are mainly Python based, and have been tested with Python3. The **dependency requirements** for these scripts are:

  * The `smartleia` package in its **version v1.0.1-1 at least** (this contains a small fix for funcards usage through PCSC relay), available [here](https://github.com/h2lab/smartleia).
  * The `pyscard`, `numpy` and `crypto` packages, all available with `pip`.

Two test scripts are provided: `script-AES128-enc.py` and `pin_timing_attacks.py`. Please be sure to run this script using LEIA's
direct access through `/dev/ttyACMx` with the toggle `USE_LEIA=True` as an environment variable. 


#### AES-128 encryption and decryption

The`script-AES128-enc.py` tests AES-128 encryption and decryption APDUs: this can be a basis to mount some side-channel attacks on an unprotected
AES (NOTE: although some APDUs setting masks are present, these are not used and are here for future evolutions).

```shell
$ USE_LEIA=True python3 script-AES128-enc.py 
[+] Using LEIA raw access
```

#### Pin timing attack

The `pin_timing_attacks.py` extracts a secret PIN from the programmed funcard using a **timing attack** that exploits the dummy algorithm
used to check the PIN. In order for this attack to succeed, a timing oracle is needed. Since such a timing oracle exploits variations
of less than milliseconds, a proper time measurement for APDUs is necessary. This script shows that LEIA's [timing feature](https://h2lab.github.io/smartleia.github.io/c/test.html#timers)
can be of use here.

```shell
$ USE_LEIA=True python3 pin_timing_attacks.py 
[+] Using LEIA raw access
```


### Handling the Funcard internal triggers

For additional measurement precision, two dedicated triggers have been implemented in the ATMega8515 firmware on the ISO7816-2 pins C4 and C8 (see the figure below). Beware that the C8 pin is shared with LEIA's onboard own
trigger.

These two pins are unused by the ISO7816-3 layer, and since they are connected to internal
pins of the ATMega8515 (PB5 and PB7), we can use them without perturbing the APDU communication with a reader (either
LEIA or any reader).

Two modes are proposed. In the first mode, the pins C4 and C8 are set high just before executing the AES, and set low after its execution.
In the second mode, the pins C4 and C8 are toggled at each AES round in order to isolate on the scope each round for a better
focus on the points of interest. You can play around with the ``trig_high_c4/8()``, ``trig_low_c4/8()`` and ``trig_inv_c4/8()`` functions
calls inside the AES (it is safe to call them from C and assembly).

```shell
      -------------+------------- 
     |   C1        |         C5  | 
     |             |             | 
     +-------\     |     /-------+ 
     |   C2   +----+    +    C6  | 
     |        |         |        | 
     +--------|         |--------+ 
     |   C3   |         |    C7  | 
     |        +----+----+        | 
     +-------/     |     \-------+ 
     |   C4=TRIG1  |    C8=TRIG2 | 
     |             |             | 
      -------------+------------- 
```

By default, the internal triggers are not active. You can specifically activate each one of them using the `00 20 00 00 02 XX YY` APDU
(class `0x00` and instruction `0x20`, with P1 and P2 set to `0x00` and two bytes data). `XX=0x01` will activate the
first trigger mode on C4 (trig C4 high when AES begins, low after). `XX=0x02` will activate the second trigger mode on C4
(toggle C4 at each AES round). `XX=0x00` will deactivate the trigger on C4. The same logic holds independently for the second
pin C8 using the valye `YY`.

You can get the actual current values of the triggers modes with the `00 21 00 00 02` APDU, getting two bytes from the card
representing the current mode on C4 and C8 respectively.

**WARNING:** using the internal trigger **C8** can perturb LEIA's own trigger set through the dedicated
trigger strategies. So use this internal trigger **with care** and if you know what you are doing!


## LEIA Scripting

### Getting the target ATR

```python
# We import the leia package
import smartleia as sl

# Connect to the reader
reader = sl.LEIA()

# Verify the card is detected
result = reader.is_card_inserted()

print(result)

try:
    reader.configure_smartcard()
except:
    try:
        reader.configure_smartcard(negotiate_pts=False)
    except:
        print("Error: are you sure that a smartcard is inserted in the LEIA board?")
        sys.exit(42)

# Initialize a connection to a card
#reader.configure_smartcard(protocol_to_use=1,  # Use T=1
#                           ETU_to_use=None,    # Let the reader determine the ETU to use
#                           freq_to_use=None,   # Let the reader determine the freq to use
#                           negotiate_pts=True, # Let the reader negotiate the PTS
#                           negotiate_baudrate=True
#)

ATR = reader.get_ATR()
print(ATR)
```

### Using LEIA Solo with your Lab Scope 

H2LAB provides two python modules in order to drive your LEIA Solo experimentation to success.

* The [smartleia-target](https://github.com/cw-leia/smartleia-target) repository holds the source of the python package used to drive the LEIA smart card reader target. With it, you will be able to drive both the LEIA-target-applet and the Funcard-target.

* The [scopy](https://github.com.cnpmjs.org/h2lab/scopy) python module provides oscilloscope controls and allow you to quickly build automated tests, capture traces. 
This module is under developpment but already allows you to automate traces capture with Lecroy and Owon scopes but also using the Chipwhisperer.

### Funcard Test campaign example

The [smartleia-target-funcard](https://github.com/h2lab/smartleia-target-funcard) repository contains an AVR implementation of an unprotected 128-bit key AES as well as a dummy PIN verification algorithm for playing with the LEIA boards.
Please be sure to update the funcard using the "sh flash_funcard.sh" (this will compile the latest funcard code and upload it through LEIA), and then you are good to go.

In this example we will use the [smartleia-target](https://github.com/cw-leia/smartleia-target) python module to control the Funcard target. For additional measurement precision, two dedicated triggers have been implemented in the ATMega8515 firmware on the ISO7816-2 pins C4 and C8. Beware that the C8 pin is shared with LEIA's onboard own trigger.

The following code snippet does not include the traces saving function. The implementation of the saving feature is left to the reader. Nevertheless a more complete script is available on [smartleia-scripting](https://github.com/h2lab/smartleia-scripting)

You can find a demontration notebook [here](leia-solo-scripting.ipynb) 

```python
# We import the scopy package
from scopy import *

# We import TriggerPoints and APDU from smartleia module
from smartleia import APDU, TriggerPoints

# We import the TargetController abstraction layer from the martleia_target module
from smartleia_target import TargetController

import secrets # Used for generating random encryption keys and plaintexts
import itertools, sys # Used by reader_wait_for_card

spinner = itertools.cycle(['-', '/', '|', '\\'])

LEIA_FUNCARD_APPLET = [0x45, 0x75, 0x74, 0x77, 0x74, 0x75, 0x36, 0x41, 0x70, 0x80]

def reader_wait_for_card(target):
    """
        Wait for the smartcard to be inserted
    """
    print('Waiting for card to be inserted...\t',end='')
    while not(target.is_card_inserted()):
        sys.stdout.write(next(spinner))
        time.sleep(0.5)
        sys.stdout.flush()            
        sys.stdout.write('\b')
    print('OK')


def toStr(l):    
    return ''.join(format(x, '02x') for x in l)


def funcard_init():
    """
        Initialize the LEIA reader, wait for the smartcard to be inserted and request the ATR.
        By default, the internal triggers are not active. 
        We specifically activate each one of them using the 00 20 00 00 02 XX YY APDU
        	XX=0x01 will activate the first trigger mode on C4 (trig C4 high when AES begins, low after). 
        	XX=0x02 will activate the second trigger mode on C4 (toggle C4 at each AES round). 

        Returns the target object.

    """
    target = TargetController()
    reader_wait_for_card(target)
    target.configure_smartcard(
        protocol_to_use = SL.T.T0,
        negotiate_pts = False,
    )
            
    # Select the LEIA Funcard applet
    target.select_applet(applet=LEIA_FUNCARD_APPLET)
    
    # Enable funcard triggers on C4 & C8
    target.sendMyAPDU(0x00, 0x20, 0x00, 0x00, tx_data=[0x02,0x01], recv=0, send_le=0)

    return target


def generate_trace(key, plaintext, target = None, scope = None, timeout = 50, save_trigs = False):
    """
        LEIA Test Case:
            AES encryption using Random key, text pair 

        Parameters:
            index: index of trace, not used here.
            scope: scope used to record the traces
                    CH1 = data
                    CH2 = trigger
            target: LeiaTarget

        Return:  Trace with leakage from oscilloscope, 
                 Value from the target plaintext/ciphertext,
                 Computing time
    """
    
    # Select the target applet
    target_init(applet=LEIA_FUNCARD_APPLET)
    
    # Load encyption key to the target
    target.loadEncryptionKey(key)
   	 
   	 # Setup random plaintext
    plaintext = secrets.token_hex(16) 
    
    # Load the plaintext to the target
    target.loadInput(plaintext)
    
    # sync the input data with the target (some target are truncating the inputs)
    key = toStr(target.checkEncryptionKey(key).data)
    plaintext = toStr(target.checkPlaintext(plaintext).data)
    
    # Configure Trigger strategy here if needed
    # Since the Funcard can generate synchronous triggers on C4 and or C8. 
    # We do not use any LEIA Trigger strategy for this exemple
    # reader_set_triggers(target)

    # arm the scope if any
    scope.arm()

    # launch computation on target and get timming delta
    timing = target.go().delta_t

    # wait for target to finish
    while target.isDone() is False and timeout:
        timeout -= -1
        time.sleep(0.01)

    try:
        ret = scope.capture()
        if ret:
            print("Timeout happened during acquisition")
    except IOError as e:
        print(f"IOError: {e}")

	trace = None
	
    # run aux stuff that shoud happen after trace from here
    
    # Read computation results
    res = target.readOutput()
    ciphertext = toStr(res.data)
    print("ciphertext:",ciphertext,"res",res)
    
   	# Read trace on scope
    channels = scope.get_last_trace()
    trace = channels[0] # CH1 = Power Trace
    
    if (len(channels) >=2) and (save_trigs == True):
        try:
            if len(trig) >= 0:
                trig = channels[1]
        except:
            # NOTE: some scope such as CW do not provide a trig trace but a timing
            # Do note save it
            trig = []
    else:
        trig = []
    return (trace, trig, key, plaintext, ciphertext, timing)

import sys
if __name__ == "__main__":
	
	# Setup the scoppy dummy scope
	#
	# Warning: 
	#   For now you must setup your scope parameters manualy only the trigger arming and
	#   traces capture features are fully operational. 	
	scope = dummy(
		"net_tcp": {"ip": "127.0.0.1") , "port": 42 },
		"usb_bulk": {"vid": 0xDEAD, "pid": 0xBEEF },
	))
	
	# Setup the funcard target
	funcard_init()
	
	# Test loop
	nb_traces = 50
   for i in range(nb_traces):
   		# Set the key and the plaintext  
       key = secrets.token_hex(16)
       plaintext = secrets.token_hex(16)
       res = generate_trace(key=key,
       					plaintext=plaintext, 
       						target=target, 
       						 scope=scope, 
       					   timeout=50, 
       				    save_trigs=True)
       				    
       trace, trigtrace, key, plaintext, ciphertext, timing = res
        
       # TODO print("Saving trace... %d/%d" % (i+1 , args.nb_traces))
      	# save(trace, key, plaintext, ciphertext, timing, trigtrace)


```

### LEIA Solo with the Chipwhisperer

Our pull request on the official Chipwhisperer repository as not been merged yet. This is why
you will have to clone the [H2Lab Chipwhisperer repository](https://github.com/h2lab/Chipwhisperer) and deploy the smartleia-target python module on your controlling plateform.
This will enable to use any LEIA target as an standard Chipwhisperer target.

You can find a demontration notebook [here](leia-cw-scripting.ipynb) 

## Contacting H2Lab for LEIA-Solo

   * Technical question: leia-faq.technical [at] h2lab.org
   * Other question: leia-faq.general [at] h2lab.org


## FAQ, Problems, incompatibilities

During our tests, we have found some various problems that you may encounter:

### I can't communicate with LEIA-Solo board!

**1. Do not use ModemManager on GNU/Linux**

ModemManager may be installed by your distribution. This service tries to communicate with any ttyACM-based device, making interactions with LEIA-Solo unstable to other tools. You may uninstall or deactivate it temporarily.

**2. Check the ttyACM numbers in your dmesg**

There are some cases where the udev mechanisms upgrade the ttyACM numbering dynamically for already plugged devices. You might have another ttyACM device connected. This may impact the python layer, even if we try to handle this use case and check successive devices.

**3. Are you using direct access or VPCD-based access ?**

If the communication is made using VPCD, the vsmartcard VPCD relay must be installed and the smartleia main loop must be executed first and keeped started in background:

```shell
$ python3 -m smartleia


        The connection with LEIA is opened and is connected to pcscd through virtualsmartcard.

        You can change the link with the smartcard with the following commands :

            configure( protocol_to_use=0,
                       ETU_to_use=...,
                       freq_to_use=...,
                       negotiate_pts=True,
                       negotiate_baudate=True)

            t0()    Equivalent to configure(protocol_to_use=0)
            t1()    Equivalent to configure(protocol_to_use=1)
            dfu()

        You have access to leia through the `leia` variable.

        Type exit() or CTRL-D to exit.


>>> Starting LEIA PCSC relay for host 127.0.0.1:35963
```


### I didn't manage to use LEIA-Solo as a smartcard reader

Be sure that you have installed the `vsmartcard VPCD backend` to your PC/SC installation. The relay is packaged under Debian Bullseye and higher:

```shell
$ sudo apt install vsmartcard-vpcd
```

On other distros, it can be compiled from sources. The upstream project can be found here: https://github.com/frankmorgner/vsmartcard

To check that the VPCD library is load by PC/SC, you can stop the `pcscd` service and run it in foreground using:

```shell
# pcscd -fad
```
You should get, in the pcscd logs, the following.

```
...
valuatetoken() Add reader: Virtual PCD
...
```

