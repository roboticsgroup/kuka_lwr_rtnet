LWR4 Realtime Communication with Xenomai + RTNet
==============

Adaptation of Kuka-provided FRI communication interface to Xenomai+RTNet. 
First and Second example are re-written. Example for pure receiving/sending of packages (no commands) provided as well as simultanenous (asynchronous) communication with two arms. 

###System Setup
We are running xenomai 2.5.6 and linux kernel 2.6.35.9 using Ubuntu 12.04 LTS. 
The ethernet card is a Intel Pro/1000 PT Quad Port Low Profile. Output of lspci is 
```
24:00.0 Ethernet controller: Intel Corporation 82571EB Gigabit Ethernet Controller (Copper) (rev 06)
```
###RTNet Installation
For RTNet we are using the master branch of **git://rtnet.git.sourceforge.net/gitroot/rtnet/rtnet**.
Currently 

```
commit 2c718c680bbda7d1abed918a2c4226b1713791c0
Author: Jan Kiszka <jan.kiszka@siemens.com>
Date:   Wed Sep 12 19:51:35 2012 +0200
```

After cloning RTNet, configure and compile rtnet for your system

```
    ./configure --enable-e1000 --enable-rtcap 

    make 

    make install 

    mknod /dev/rtnet c 10 240 
```

After that, the binaries and testscripts are installed in /usr/local/rtnet and a new device node is created.

###Network Card Interface Setup
Change the mapping of the hw-adresses your network ports to names (may not be necessary.) in */etc/udev/rules.d/70-persistent-net.rules*. 
The RTnet ports should start at eth0 and the normal ports start with whatever id is available. E.g. on our machine, we have four rtnet capable ports (eth0-3) and 2 non-realtime ports (eth4,5). 
Uninstall NetworkManager so that it is not managing the network devices automatically.
```
apt-get remove network-manager 
```
Instead, setup your */etc/network/interfaces*. E.g. on our machine, we only need to bring up one non-realtime port for normal ethernet and dhcp. For an **example**, look at interfaces in directory configs.

### Kuka Network Setup
On the Kuka side, you have to make sure that the network card is correctly set up. It needs to know its own and the remote side's IP address. On the Kuka controller (KRC) open the file *C:\Windows\vxwin.ini* and enter it IP address
```
[Boot]
Bootline=elPci(0,1)pc:vxworks h=192.0.1.2 b=192.0.1.1 e=192.168.0.20 u=target pw=vxworks
```
In *C:\KRC\Roboter\INIT\dlrrc.ini* set a static IP for the remote computer that is connected to the controller. (e.g. FRIHOST=192.168.0.100). 
```
[DLRRC]
FRIHOST=192.168.0.100
FRISOCK=49938,0
```
After rebooting the KRC and having an ethernet cable connected directly from the KRC realtime ethernet card to the remote machine (with IP as set in FRIHOST, e.g. 192.168.0.100), then you should be able to ping the KRC from the remote machine. This should be possible even from a non-realtime ethernet connection.


###Bringing up RTNet Interfaces
After rebooting, ifconfig should show one ethernet device. In our case this would be eth4. For starting the realtime ethernet ports, run the script **host_rtnet** (in directory configs). It will first unload all other network drivers, and then set up one rtnet port with a static ip 192.168.0.100, set a route to the kuka arm via IP 192.168.0.20 and create the device rteth0. Then, it sets up the non-realtime port by loading the driver tg3 and bringing up eth4. Check with ifconfig, if the ethernet ports rteth0 exist now.

For running this rtnet script at startup, copy it to /etc/init.d/ and create symbolic links in all the runlevels to this script.

An example script for starting up the FRI with 1ms on the kuka arms and starting the axis-specific stiffness controller strategy is also contained in directory configs (friBare_RT.src).
