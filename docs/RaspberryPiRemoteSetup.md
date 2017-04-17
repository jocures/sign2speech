Raspberry Pi Remote Setup
=========================

Here's the setup instructions if you want to run the Leap Motion on a Raspberry Pi. The steps are different depending on whether the "receiving" computer is running Windows or Linux.

Windows instructions (USB-Over-Network by FabulaTech)
-----------------------------------------------------

Fabulatech's USB-Over-Network product is proprietary. The server side (which you install on the RPi) has a 15-day/1-device-at-a-time free trial. The client side (which you install on Windows) is free.

### On the RPi

Install `ftusbnet` from their website and unpack it into `/opt`:

    wget http://www.fabulatech.com/usb-over-network-linux-packages/ftusbnet-5.2.3-armhf.tar.gz
    sudo tar -zxvf ftusbnet-*.tar.gz -C /opt

Start the `ftusbnetd` daemon in daemon-mode so it runs in the background (you'll need to do this every time you reboot or make an init.d service to automate it).

    sudo /opt/ftusbnet/sbin/ftusbnetd -d

To see what devices are available for sharing:

    /opt/ftusbnet/bin/ftusbnetctl list 

**Note:** Occasionally, no devices (not even internal hubs) will show up. Try rebooting, then reinstalling, then re-image your Pi if nothing works. I have no idea why this happens, but it's uncommon, and re-imaging solves it.

Now you can share the Leap motion. Whichever ID it is on the list (let's go with 104 for example), you can share that device like this (substituting your device's ID):

    sudo /opt/ftusbnet/bin/ftusbnetctl share 104

Use `ifconfig` and copy down your Pi's local ip on the Ethernet network.

### On Windows

Download the client side of USB-Over-Network from [here](http://www.fabulatech.com/usb-over-network-64bit.msi). Use the Wizard to install it. 

When it opens, click "Add Server" and type in the IP address you copied down. The Leap Motion Controller should appear as an available device. Click "Connect". At this point, if you've installed the Leap drivers and software, the indicator in the task panel should go from black to green to indicate a connection.

Linux Instructions (usbip)
--------------------------

The `usbip` package can be downloaded from `apt`, but the version that comes packaged in the Linux kernel should do just fine. On both sides, you want to make sure that you're using version 2.0 (try `usbip version`). Some package managers stock version 0.2, which isn't compatible and has slightly different command line options from the ones you'll see here.

### On the RPi

Run the following commands to set up the server (you'll need to do this after every reboot):

    sudo modprobe usbip-core
    sudo modprobe usbip-host
    sudo usbipd -D

Use this command to view the available USB devices: 

    sudo usbip list -l

Make note of the Leap's bus ID (it'll proabably be something like `1-1` or `1-1.2`). Substitute that bus ID into this command:

    sudo usbip bind -b 1-1.2

Use `ifconfig` and copy down your Pi's local ip on the Ethernet network.

### On your Ubuntu Box (or VirtualBox if you're nasty)

Run the following command to set up the client (you'll need to do this after every reboot):

    sudo modprobe vhci-hcd

Use the following command to view the devices shared by the Pi:

    usbip list -r <pi IP address>

Use the following command to attach that device:

    sudo usbip attach --host <pi ip> --busid <busid>

The Leap Motion should now show up when you run `lsusb` and if you've installed the Leap drivers and software, the indicator in the task panel should go from black to green to indicate a connection.

Troubleshooting
---------------

If things aren't working, here are some hurdles I've come up against in the past:

- This pretty much only works over a wired Ethernet connection. If you want to try WiFi, Bluetooth or anything else, bear in mind that the Leap needs to send about 80Mbps of data over the connection.
- The Windows Task manager (Ctrl+Alt+Esc) and the Linux `nethogs` tool (`sudo apt-get install -y nethogs`, make sure to run the tool itself with `sudo` too) are good for seeing how much bandwidth you're using.
- This will not work if the receiving computer is running macOS. Nobody has developed a mature solution for forwarding USB to macOS; I think it has to do with the Darwin kernel dealing with USB in an opaque way, but I don't know for sure.
- Running the Leap in a VirtualBox VM is discouraged, as the 64-bit Virtualbox has a messed up USB emulator according to [Leap's Linux FAQ](https://support.leapmotion.com/hc/en-us/articles/223782608-Linux-Installation). The Leap *may* still work, but it will have connection issues.