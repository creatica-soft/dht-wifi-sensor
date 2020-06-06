# dht-wifi-sensor
esp-01s with DHT11 sensor

This is another example of Expressif Non-OS-SDK 3.0 programming for $3 wireless DHT (Temperature/Humidity) sensor from Ali Express https://www.aliexpress.com/item/32880487687.html

The ESP8266EX module will connect to a WiFi access point, optionally read the timer paramters from a web server and periodically submit temperature, humidity data obtained from GPIO02 pin, to a configured URL using GET request; for example, to a time series database such as RRD using PHP binding.

http://example.com/dht.php?t=20.5&h=60.0

