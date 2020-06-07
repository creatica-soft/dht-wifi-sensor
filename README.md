# dht-wifi-sensor
## esp-01s with DHT11 sensor

This is another example of Expressif Non-OS-SDK 3.0 programming for $3 wireless DHT (Temperature/Humidity) sensor from Ali Express https://www.aliexpress.com/item/32880487687.html

The ESP8266EX module will connect to a WiFi access point, optionally read the timer paramters from a web server and periodically submit temperature, humidity data obtained from GPIO02 pin, to a configured URL using GET request; for example, to a time series database such as RRD using PHP binding.

http://example.com/dht.php?t=20.5&h=60.0

```
rrdtool create /var/env.rrd \
          --step 1h \
          DS:temperature:GAUGE:1d:-50:50 \
          RRA:AVERAGE:0.5:1h:1d \
          RRA:AVERAGE:0.5:1h:1M \
          RRA:AVERAGE:0.5:1d:1y \
          RRA:AVERAGE:0.5:1d:10y \
          RRA:MIN:0.5:1h:1d \
          RRA:MIN:0.5:1h:1M \
          RRA:MIN:0.5:1d:1y \
          RRA:MIN:0.5:1d:10y \
          RRA:MAX:0.5:1h:1d \
          RRA:MAX:0.5:1h:1M \
          RRA:MAX:0.5:1d:1y \
          RRA:MAX:0.5:1d:10y \
          DS:humidity:GAUGE:1d:0:100 \
          RRA:AVERAGE:0.5:1h:1d \
          RRA:AVERAGE:0.5:1h:1M \
          RRA:AVERAGE:0.5:1d:1y \
          RRA:AVERAGE:0.5:1d:10y \
          RRA:MIN:0.5:1h:1d \
          RRA:MIN:0.5:1h:1M \
          RRA:MIN:0.5:1d:1y \
          RRA:MIN:0.5:1d:10y \
          RRA:MAX:0.5:1h:1d \
          RRA:MAX:0.5:1h:1M \
          RRA:MAX:0.5:1d:1y \
          RRA:MAX:0.5:1d:10y
```

(install php-rrd)

### dht.php 

```
<?php
$t = htmlspecialchars($_REQUEST['t']);
$h = htmlspecialchars($_REQUEST['h']);
$values = array(sprintf("N:%f:%f", $t, $h));
if (!rrd_update("/var/env.rrd", $values)) {
  header('HTTP/1.1 500 Internal Server Error');
  exit;
}
?>
```

### dht-timer-parameters.txt

```
* 0 * * * *
55000
0
```

## Compiling and Flashing

1. Review and update defines

```
wget https://github.com/espressif/ESP8266_NONOS_SDK/archive/v3.0.3.tar.gz
tar -zxf ESP8266_NONOS_SDK-v3.0.3.tar.gz
mkdir -p ESP8266_NONOS_SDK-3.0.3/dht/user
cp user_main.c ESP8266_NONOS_SDK-3.0.3/dht/user
cp ESP8266_NONOS_SDK-3.0.3/examples/IoT_Demo/Makefile ESP8266_NONOS_SDK-3.0.3/dht
```

2. Comment out "driver" lines in Makefile above

```
cp -r ESP8266_NONOS_SDK-3.0.3/examples/IoT_Demo/include ESP8266_NONOS_SDK-3.0.3/dht/
cp SP8266_NONOS_SDK-3.0.3/examples/IoT_Demo/user/Makefile ESP8266_NONOS_SDK-3.0.3/dht/user
```

3. Install esp-open-sdk first for crosscompiling: https://github.com/pfalcon/esp-open-sdk

```
export PATH=~/esp-open-sdk/xtensa-lx106-elf/bin:$PATH
```

4. Compile user1.bin for esp-01s with `make COMPILE=gcc BOOT=new APP=1 SPI_SPEED=40 SPI_MODE=0 SPI_SIZE_MAP=2`

5. Compile user2.bin for esp-01s with `make clean; make COMPILE=gcc BOOT=new APP=2 SPI_SPEED=40 SPI_MODE=0 SPI_SIZE_MAP=2`

6. Install esptool from https://github.com/espressif/esptool

7. Power off, ground GPIO0 before flashing, power on

Power cycle after each flash!

8. Flash boot.bin first:

```
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0x00000 ../ESP8266_NONOS_SDK-3.0.3/bin/boot_v1.7.bin
```

9. Flash user1.bin:

```
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0x01000 ../ESP8266_NONOS_SDK-3.0.3/bin/upgrade/user1.1024.new.2.bin
```

10. Flash redundant image user2.bin:

```
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0x81000 ../ESP8266_NONOS_SDK-3.0.3/bin/upgrade/user2.1024.new.2.bin
```

11. Flash blank.bin and esp_init_data_default.bin:

```
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0xfb000 ../ESP8266_NONOS_SDK-3.0.3/bin/blank.bin
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0xfc000 ../ESP8266_NONOS_SDK-3.0.3/bin/esp_init_data_default_v08.bin
./esptool.py -p /dev/ttyAMA0 -b 115200 write_flash 0xfe000 ../ESP8266_NONOS_SDK-3.0.3/bin/blank.bin
```

12. Power off, unground GPIO0 for normal operation, power on

13. Run terminal emulator to see the os_printf() messages:

```
miniterm --raw /dev/ttyAMA0 74880
```
