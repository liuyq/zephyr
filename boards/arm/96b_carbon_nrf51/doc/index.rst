.. _96b_carbon_nrf51_board:

96Boards Carbon nRF51
#####################

Overview
********

This is the secondary nRF51822 chip on the 96Boards Carbon and provides
Bluetooth functionality to the main STM32F401RET chip via SPI.

.. note::

   If you're looking to reprogram the main STMicro part, see
   :ref:`96b_carbon_board`. Users should not use this configuration
   unless they want to reprogram the secondary chip which provides
   Bluetooth connectivity.


Hardware
********

The 96Boards Carbon nRF51 has two external oscillators. The frequency
of the slow clock is 32.768 kHz. The frequency of the main clock is 16
MHz.

See :ref:`96b_carbon_board` for other general information about the
board; that configuration is for the same physical board, just a
different chip.

Supported Features
==================

+-----------+------------+-------------------------------------+
| Interface | Controller | Driver/Component                    |
+===========+============+=====================================+
| NVIC      | on-chip    | nested vector interrupt controller  |
+-----------+------------+-------------------------------------+
| RTC       | on-chip    | system clock                        |
+-----------+------------+-------------------------------------+
| UART      | on-chip    | serial port                         |
+-----------+------------+-------------------------------------+
| GPIO      | on-chip    | gpio                                |
+-----------+------------+-------------------------------------+
| FLASH     | on-chip    | flash                               |
+-----------+------------+-------------------------------------+
| SPIS      | on-chip    | SPI slave                           |
+-----------+------------+-------------------------------------+
| RADIO     | on-chip    | Bluetooth                           |
+-----------+------------+-------------------------------------+

The default configuration can be found in the defconfig file:

        ``boards/arm/96b_carbon_nrf51/96b_carbon_nrf51_defconfig``

Connections and IOs
===================

SPI
---

96Boards Carbon nRF51 has one SPI, which for providing Bluetooth
communication. The default SPI mapping for Zephyr is:

- SPI1_NSS  : P0.25
- SPI1_SCK  : P0.07
- SPI1_MISO : P0.30
- SPI1_MOSI : P0.00

The SWD debug pins are broken out to an external header; all other
connected pins are to the main STM32F401RET chip.

.. _96b_carbon_nrf51_programming:

Programming and Debugging
*************************

Flashing
========

The 96Boards Carbon nRF51 can be flashed using an external SWD
debugger, via the debug header labeled "BLE" on the board's
silkscreen. The header is not populated; 0.1" male header must be
soldered on first.

.. figure:: img/96b_carbon_nrf51.png
     :align: center
     :alt: 96Boards Carbon nRF51 Debug

     96Boards Carbon nRF51 Debug

The following example assumes a Zephyr binary ``zephyr.elf`` will be
flashed to the board.

It uses the `Black Magic Debug Probe`_ as an SWD programmer, which can
be connected to the BLE debug header using flying leads and its 20 Pin
JTAG Adapter Board Kit. When plugged into your host PC, the Black
Magic Debug Probe enumerates as a USB serial device as documented on
its `Getting started page`_.

It also uses the GDB binary provided with the Zephyr SDK,
``arm-zephyr-eabi-gdb``. Other GDB binaries, such as the GDB from GCC
ARM Embedded, can be used as well.

.. code-block:: console

   $ arm-zephyr-eabi-gdb -q zephyr.elf
   (gdb) target extended-remote /dev/ttyACM0
   Remote debugging using /dev/ttyACM0
   (gdb) monitor swdp_scan
   Target voltage: 3.3V
   Available Targets:
   No. Att Driver
    1      nRF51
   (gdb) attach 1
   Attaching to Remote target
   0xabcdef12 in ?? ()
   (gdb) load

A generic ST-Link-v2 programmer could be used to program
the NRF51 chip on the Carbon board as well.

To use a generic ST-Link-V2 programmer, needs to install openocd first,

.. code-block:: console

    # Common dependencies
    $ apt install pkg-config automake libtool

    # openocd upstream repository
    $ git clone git://git.code.sf.net/p/openocd/code openocd-code
    $ cd openocd-code
    $ ./bootstrap
    $ ./configure
    $ make
    $ make instal

and then create the configure file for nRF51 chip,

.. code-block:: console

    $ cat >carbon-nrf51-stlink-v2.cfg <<__EOF__
    source [find interface/stlink-v2.cfg]
    transport select hla_swd

    set WORKAREASIZE 0x4000
    source [find target/nrf51.cfg]
    __EOF__

and flash the generated zephyr.hex at last.

.. code-block:: console

    $ openocd -f carbon-nrf51-stlink-v2.cfg -c "program /pathto/zephyr.hex verify exit"

Or to flash with zephyr west framework:

.. code-block:: console

    $ source zephyr-env.sh
    $ west build -b 96b_carbon_nrf51 samples/bluetooth/hci_spi
    $ west flash

Debugging
=========

After you've flashed the chip, you can keep debugging using the same
GDB instance. To reattach, just follow the same steps above, but don't
run "load". You can then debug as usual with GDB. In particular, type
"run" at the GDB prompt to restart the program you've flashed.

As an aid to debugging, this board configuration directs a console
output to a currently unused pin connected to the STM32F401RET. Users
who are experienced in electronics rework can remove a resistor (R22)
on the board and attach a wire to the nRF51822's UART output.

.. _96b_carbon_nrf51_bluetooth:

Providing Bluetooth to 96b_carbon
*********************************

This 96b_carbon_nrf51 Zephyr configuration can be used to provide
Bluetooth functionality from the secondary nRF51822 chip to the
primary STM32F401RE chip on the :ref:`96b_carbon_board`.

To do this, build the ``samples/bluetooth/hci_spi/`` application
provided with Zephyr with ``BOARD=96b_carbon_nrf51``, then flash it to
the nRF51822 chip using the instructions :ref:`above
<96b_carbon_nrf51_programming>`. (For instructions on how to build a
Zephyr application, see :ref:`build_an_application`.)

.. warning::

   Be sure to flash the hci_spi application to the nRF51822 chip and
   not to the main STM32F401RET chip.  While both chips are supported
   by Zephyr, the hci_spi application providing Bluetooth support will
   only run on the nRF51822 chip.

References
**********

- `Board documentation from 96Boards`_
- `nRF51822 information from Nordic Semiconductor`_

.. _Black Magic Debug Probe:
   https://github.com/blacksphere/blackmagic/wiki

.. _Getting started page:
   https://github.com/blacksphere/blackmagic/wiki/Getting-Started

.. _Board documentation from 96Boards:
   http://www.96boards.org/product/carbon/

.. _nRF51822 information from Nordic Semiconductor:
   https://www.nordicsemi.com/eng/Products/Bluetooth-low-energy/nRF51822
