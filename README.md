LWR4 Realtime Communication with Xenomai + RTNet
==============

Adaptation of Kuka-provided FRI communication interface to Xenomai+RTNet. 
First and Second example are re-written. Example for pure receiving/sending of packages (no commands) provided as well as simultanenous (asynchronous) communication with two arms. 

### Kuka Network Setup
On the Kuka side, you have to make sure that the network card is correctly set up. It needs to know its own and the remote side's IP address. On the Kuka controller (KRC) open the file *C:\Windows\vxwin.ini* and enter it IP address (e.g. 192.168.0.1)
```
[Boot]
Bootline=elPci(0,1)pc:vxworks h=192.0.1.2 b=192.0.1.1 e=192.168.0.1 u=target pw=vxworks
```
In *C:\KRC\Roboter\INIT\dlrrc.ini* set a static IP for the remote computer that is connected to the controller. (e.g. FRIHOST=192.168.0.10). 
```
[DLRRC]
FRIHOST=192.168.0.10
FRISOCK=49938,0
```
Reboot the KRC.

###System Setup

To install Xenomai follow this guide: http://jbohren.com/articles/xenomai-precise/
By default the Comedi driver (for NI PCI6221 and ATI F/T 6DOF sensor) are enabled in xenomai configuration. Don't disable them.

We are running xenomai 2.6.4 and linux kernel 3.8.13 using Ubuntu 12.04 LTS. 
The ethernet card is a Realtek 8139. Output of lspci is 
```
03:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8139/8139C/8139C+ (rev 10)
```
###RTNet Installation
For RTNet we are using the master branch of **git://rtnet.git.sourceforge.net/gitroot/rtnet/rtnet**.
Currently 

```
commit 7c8ba10513fe7b63873f753ab22d340bf44119a2
Author: Leopold Palomo-Avellaneda <leopold.palomo@upc.edu>
Date:   Mon Dec 2 15:58:14 2013 +0100
```

After cloning RTNet, configure and compile rtnet for your system

```
	git clone git://rtnet.git.sourceforge.net/gitroot/rtnet/rtnet

    cd rtnet

    make menuconfig 

    make 

    make install 

    mknod /dev/rtnet c 10 240 
```

After that, the binaries and testscripts are installed in /usr/local/rtnet and a new device node is created.

###Network Card Interface Setup
Change the mapping of the hw-adresses your network ports to names (may not be necessary.) in */etc/udev/rules.d/70-persistent-net.rules*. 
The RTnet ports should start at eth0 and the normal ports start with whatever id is available. E.g. on our machine, we have one RT port (eth0) and 1 non-realtime port (eth1). 
Uninstall NetworkManager so that it is not managing the network devices automatically.
```
apt-get remove network-manager 
```
Instead, setup your */etc/network/interfaces* as in the "interfaces" example in configs folder.

###Bringing up RTNet Interfaces
After rebooting the pc ifconfig should show both ethernet device. For starting the realtime ethernet ports or reverting back to non-real time drivers, run the script accordingly:
```
sudo ./configs/host_rtnet start|stop
```
On start it will first unload all other network drivers, and then set up one rtnet port with a static ip 192.168.0.10, set a route to the kuka arm via IP 192.168.0.1 and create the device rteth0. Then, it sets up the non-realtime port by loading the driver 8139too and bringing up eth1. Check with ifconfig, if the ethernet ports rteth0 exist now.

For running this rtnet script at startup, copy it to /etc/init.d/ and create symbolic links in all the runlevels to this script.

An example script for starting up the FRI with 1ms on the kuka arms and starting the axis-specific stiffness controller strategy is also contained in directory configs (friBare_RT.src).
