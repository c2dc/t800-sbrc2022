# esp32_research

## How-to use
Before running the actual experiment, you must have `iperf v2` and `nmap` installed
on your machine. The python script will use TCP ports `6767`, `6768` and `5001`, so
in case a socket error occurs check if these are in use by other programs.

Also, you'll need to run any program that connects to wi-fi in the ESP32 so you can
get the IP of the device in your network. Then, build and flash the experiment
onto your ESP32:
1. Change `ATTACKER_ADDRESS` in line 30 of `/tcp_server/main/tcp_server.c` to your IP
2. Change `ESP32_ADDRESS` in line 31 of `/tcp_server/main/tcp_server.c` to esp32 IP
3. Run `idf.py menuconfig` > `Example Connect Configuration` > Set your `wifi ssid` and `password`

```bash
get_idf
cp -r ./firewall $IDF_PATH/components/
cp -r ./tflite-lib $IDF_PATH/components/
cp -r ./esp-nn $IDF_PATH/components/
cp lwip/CMakeLists.txt $IDF_PATH/components/lwip/CMakeLists.txt
cp lwip/lwip/src/core/ipv4/ip4.c $IDF_PATH/components/lwip/lwip/src/core/ipv4/ip4.c
cp lwip/lwip/src/include/lwip/ip4.h $IDF_PATH/components/lwip/lwip/src/include/lwip/ip4.h
cd ./tcp_server
idf.py -p <YOUR_ESP32_PORT> flash
```

Now, we can execute the experiment:
1. run `sudo python attacker.py` in one terminal
2. run `idf.py -p <YOUR_ESP32_PORT> monitor` on another terminal, in parallel right after the python script

During the experiment, the pin `D5` will be 0 when the experiment isn't running and
1 during it's duration.

After the experiment, a `data.csv` file will be generated in this folder with the
data collected during the experiment.
