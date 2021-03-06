====
 MLX90614 sensor utility for Raspberry PI.
 Supports reading of the object and ambient temperature, reading and writing of the useful eeprom params 
 Currently supported only Axx sensors with one IR module

 Tested on Raspbian with Linux kernel 4.4.15+

 Kutkov Oleg <kutkov.o@yandex.ru>

---
 Requirments:
   libi2c-dev

 Building:
	make

 * * *

 Important note before usage!!!
  This device requires combined write/read request,
  which is disabled in i2c_bcm2708 driver by default (tested on Raspberry PI model B+)
  To activate combined mode please run this command as root (not sudo):
    echo -n 1 > /sys/module/i2c_bcm2708/parameters/combined

 * * *

---
 Usage:
 * Read IR temp: 
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a -i
    or
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --get_ir_temp

    where bus - number of the actual i2c bus in system
    i2c_addr - address of the slave device (0x5e is factory default)
    get_ir_temp - what we want to read

    Normal answer:
      Tobj = XX.XX
    Where XX.XX is object temperature in C


 * Read ambient temp:
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a -a
    or
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --get_ambient_temp

    arguments are the same

    Normal answer:
      Tamb = XX.XX
    Where XX.XX is ambient temperature in C


 * Get emissivity correction coefficient:
    Factory default is 1.0 = 0xFFFF = 65535
    Emissivity = dec2hex[ round( 65535 x ) ]
    Where dec2hex[ round( X ) ] represents decimal to hexadecimal conversion with round-off to nearest value (not truncation). 
    In this case the physical emissivity values are = 0.1...1.0.

    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a -e
    or
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --emissivity_coefficient


 * Change emissivity correction coefficient.
    In normal cases this value should not be changed!

    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2A --emissivity_coefficient=65534 -w

    Where 65534 is new value
    Argument 'w' means 'write', supported commands with this argument will be used for writing to device


 * Change i2c address:
    This command may be used to resolve address conflicts of the multiple devices on the same I2C bus

    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a -w --new_addr=0x2f

    Where 0x2f is new i2c address. after this command you should power off and power on device to applly changes. 
    Now i2cdetect should show a new address of your device


 * PWM mode:
    MLX devices supports two formats of the output signal: SMBus and simple PWM. See datasheet for details about both modes.
    Factory setting is SMBus. Switching between modes can be done via pulling down SCL pin for >1.2 ms.
    For permanent mode selection EEPROM configuration must be changed.

    Check current mode:

    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a -p
    or
    $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --pwm_mode

    Output:
     PWM mode - disabled|enabled

 * Change PWM mode:
   $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --pwm_mode=1 -w

   pwm_mode=1 - enable PWM mode
   pwm_mode=0 - disable PWM mode

   Power down and power on device to apply changes


 * Debug features:
   Additional argument --debug may be used to see some details of the operations
   Example usage and output:

   $ ./read_mlx90614 --bus 1 --i2c_addr 0x2a --pwm_mode=1 -w --debug
   Opening i2c interface /dev/i2c-1
   Setting up slave address 0x2A
   Perfoming I2C_SMBUS_READ request to device, command = 0x22
   Ok, got answer from device
   EEPROM cell = 0x22 current value = 0x0201
   Erasing EEPROM cell = 0x22
   Trying to store value = 0x0203 to the EEPROM cell = 0x22
   PWM mode is now enabled


---
 TODO
   - Implement extended configuration for PWM
   - Add support of the Bxx devices with two onboard sensors
   - Implement ranges reading and setting
