# Beaglebone IEEE 802.15.4
 Tinkering with the MRF24J40 module on BeagleBone Black

Using the latest Debian image (*debian-9.9-iot-armhf-2019-08-03*) at the time of writing.

## Device tree

`spi-max-frequency = <20000000>` had to be changed to `spi-max-frequency = <10000000>` in order to work with the latest image.

Compile the device tree.

```
dtc -O dtb -o BB-BONE-MRF24J40-00A0.dtbo -b o -@ cape-bone-mrf24j40-00A0.dts
```

then copy the resulting .dtbo file to */lib/firmware*

```
sudo cp BB-BONE-MRF24J40-00A0.dtbo /lib/firmware
```

Reboot.

## uBoot configuration

As bonecape_mgr seems to be in a state of obsolescence, we have to load the device tree with uBoot with the following line in *uEnv.txt*

```
uboot_overlay_addr4=/lib/firmware/BB-BONE-MRF24J40-00A0.dtbo
```

Replace the *uEnv.txt* file in /boot/uEnv.txt by the one present in this repository.

## wpan-tools

Install wpan-tools.

```
sudo apt-get update
sudo apt-get install wpan-tools
```

Check that the device is indeed present with `ip addr`.

```
debian@beaglebone:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: wpan0: <BROADCAST,NOARP> mtu 123 qdisc noop state DOWN group default qlen 300
    link/ieee802.15.4 0a:8e:e6:5c:04:1b:d2:28 brd ff:ff:ff:ff:ff:ff:ff:ff
```

Configure the device according to your desired parameters

```
sudo ip link set wpan0 down
sudo iwpan phy0 set channel 0 13
sudo iwpan dev wpan0 set pan_id 0xcafe
sudo ip link add link wpan0 name lowpan0 type lowpan
sudo ip link set wpan0 up
sudo ip link set lowpan0 up
```

You should now be able to ping your device from another node (*riot-os*, for instance).

## riot-os

Using the nanocoap_server example: 

https://github.com/RIOT-OS/RIOT/tree/master/examples/nanocoap_server

you can share and access data using the CoAP protocol over 6LoWPAN.

```
debian@beaglebone:~$ coap-client -a fe80::5488:fc06:424f:84c6%lowpan0  -m get coap://[fe80::7b7c:d27:84f3:fcba]/riot/ver
v:1 t:CON c:GET i:e2ae {} [ ]
This is RIOT (Vev:1 t:CON c:GET i:e2af {} [ ]
rsion: 2019.07) v:1 t:CON c:GET i:e2b0 {} [ ]
running on a samv:1 t:CON c:GET i:e2b1 {} [ ]
r21-xpro board wv:1 t:CON c:GET i:e2b2 {} [ ]
ith a samd21 MCUv:1 t:CON c:GET i:e2b3 {} [ ]
.
```