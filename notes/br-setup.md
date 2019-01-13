## Operating system
- debian 9.6 (vm)
- raspbian (RPi3)

Install packages:
```console
# apt update
# apt install sudo vim git
```
Add user to `sudo` group.
```console
# usermod -a -G sudo <username>
```

## OpenThread Border Router
```console
$ git clone --branch thread-br-certified-20180819 https://github.com/openthread/borderrouter.git
```
Install dependencies:
```console
$ cd borderrouter
$ ./script/bootstrap
```

Compile and install OTBR and `wpantund`. Without setting up the machine as a Wi-Fi access point.
```console
$ ./script/setup NETWORK_MANAGER=0
```

Edit `/etc/wpantund.conf`
```
Config:NCP:SocketPath "/dev/ttyACM0"
```

## NCP
TODO

## Auto commissioning
```console
$ sudo wpanctl setprop Network:PANID 0xDEAD
$ sudo wpanctl setprop Network:XPANID DEAD1111DEAD2222
$ sudo wpanctl setprop Network:Key 11112233445566778899DEAD1111DEAD

$ sudo wpanctl config-gateway -d "fd11:22::"
```
```console
$ cd ~/borderrouter/tools
$ ./pskc D0M001 DEAD1111DEAD2222 domotica
370b5a599bbf0ac63e733b1929bbbef9

$ sudo wpanctl setprop Network:PSKc --data 370b5a599bbf0ac63e733b1929bbbef9
```
```console
$ sudo wpanctl form "domotica"

$ sudo wpanctl commissioner start
$ sudo wpanctl commissioner joiner-add "*" 600 D0M001
```

On the joiner node:
```console
> ifconfig up
Done
> joiner start D0M001
Done
> Join success
> thread start
Done
> udp open
Done
> udp bind :: 1212
Done
```
Get the node's __fd11:22__ ip address.
```console
> ipaddr
fd11:22:0:0:4e82:c63e:1d26:922e
fdde:ad11:11de:0:0:ff:fe00:7401
fe80:0:0:0:6c8e:ba4f:eeff:f413
fdde:ad11:11de:0:4bcf:6d38:49f9:63e0
Done
```
Send a test message from the BR to the node
```console
$ echo "hello" > /dev/udp/fd11:22:0:0:4e82:c63e:1d26:922e/1212
```

Multicast a test message to the mesh network
```console
$ sudo apt install smcroute
$ sudo mcsender -iwpan0 -t3 ff03::1:1212
```
