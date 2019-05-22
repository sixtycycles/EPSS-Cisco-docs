should# Common commands and actions for Cisco Network hardware in EPSS #

The first order of business is to outline the infrastructure in EPSS.

We have 5 Cisco catalyst 6500's of varying ages.

There are two in Slichter in the 5th floor closet, 3 in Geology(basement, 3rd floor, 5th floor library). The Slichter switches take care of most of the former IGPP offices ect, whereas the Geology switches cover all floors of the geology building.

There are also 3 Cisco 4948 switches, 2 in the basement (telecom room and server room) and one in the 3rd floor closet.

The important note about these is that the 4948's understand Cisco IOS whereas the older catalysts speak CatOS, the predecessor to ios.

IOS is significantly more google-able than CatOS, so this guide will focus on the CatOS commands initially. the Core router in Slichter also speaks IOS


## Getting Information ##
first, the management IP's and passwords for these devices are stored in lastpass as secure notes. ask Rod for any credentials you need.

if you are in the closet and want to use the console cable to administer the switch, you will need to run a command to pipe the serial output to your terminal emulator. on a mac, use GNU `screen` followed by the location of your usb port, and 9600 (the baud rate you will be reading).

to find your usb ports on a mac, see [this article](https://stackoverflow.com/questions/12254378/how-to-find-the-serial-port-number-on-mac-os-x#12260359)
your final command should be something like this:
`screen /dev/tty.usbserial-A6004byf 9600`
and hit enter a few times.  You should be asked for a password.

### What is going on  in there? ###

once you have logged into a catalyst, you will probably want to ensure that you know some things about the device.

here are some commands in no particular order:

* `show module` or just `sho mod`
  * this will give you a list of all the modules installed and a top level view of their status (hopefully its OK all the way down!)
* `show port {module}/{port_number}`
  * this command will show you information about the ports on the switch. if you have traced a line form the panel to the switch this is the command to start diagnosing the issue.
* `sho vlan`
  * prints the Vlans which are in use on the switch. Its important to note that you may have vlans available that are just not in use on this particular switch.
* `enable` and `disable`
  * enter and exit (respectively) from Enabled mode.
  * enabled mode is for editing configurations. you can do no harm unless you enter enabled mode, so be sure you want to change things before being in enabled mode, and exit once you are done with changes!



### Locating a problem host on the network. ###

** NOTE: This should be less important once we label everything this summer! **

Many times, we are asked by IT services to look into a problem host, either for a virus, or some other problem, like a needed update, or exposed ports. To find these machines, we are given an IP and a time of detection. however, if the IP is in the DHCP range, we may need to talk to Jason to get a copy of the logs from that time. This should be more straightforward after we move DHCP back to our department.

So, because IP's move around in our subnets, we need to get the mac address of the problem host. This is done on the * core router *

Once you are remoted into the router, you will use the ARP tables to get the hardware address of the problem host.
you can print the entire arp table with `show arp` btu that is likely not useful for you, because it has EVERYTHING In there.
so, we can pipe the output to another command `include` (kinda like a primitive `grep`). so lets say the problem ip was `128.97.31.133`

`show arp | include 133` will work, but will show all ips with 133 ANYWHERE in them, so to narrow it down, use:
`show arp | include 31\.133`

where the period needs to be 'escaped' with the slash to be properly read.

another example would be for 169.232.144.244:
`show arp | include 144\.244` which should return something like this:
```
  gs-cr>show arp | include 144\.244
  Internet  169.232.144.244         1   xxxx.bxxs.3s23  ARPA   Vlan65
```
if you want to be really fancy, you can super abbreviate it like:
`sho arp | i 144\.244`

ok! so, now we have the Mac address (copy it to clipboard or something) we can exit the router and start trying to chase it across the switches. this is largely a trail and error, but a procedural approach is the best way.

I start with the basement catalyst.

after logging in, type ` show cam  xxxx.xxxx.xxxx` where xxxx.xxxx.xxxx is the copied mac address. this should give you some output like this:
```
  GEO_C65_B> sho cam xxxx.xxxx.xxxx
  * = Static Entry. + = Permanent Entry. # = System Entry. R = Router Entry.
  X = Port Security Entry $ = Dot1x Security Entry

  VLAN  Dest MAC/Route Des    [CoS]  Destination Ports or VCs / [Protocol Type]
  ----  ------------------    -----  -------------------------------------------
  65    xx-xx-xx-xx-xx-xx             4/2 [ALL]
  Total Matching CAM Entries Displayed  =1
```

From here we can see what Vlan this IP is on (usually 65, but we have others sometimes particularly with IGPP hosts) under "Destination Ports" you can see that this device is connected to Module 4, port 2. so we can type `sho port 4/2` to see more information about this device.

```
GEO_C65_B> sho port 4/2
* = Configured MAC Address

Port  Name                 Status     Vlan       Duplex Speed Type
----- -------------------- ---------- ---------- ------ ----- ------------
 4/2  geology_5th_Switch   connected  trunk        full  1000 1000-LX/LH

... (more stuff we don't need below) ...
```


The output of show port, lets me know that this address is actually on a port that trunks to the geology 5th floor switch. So, in this case we should continue by exiting the remote session for this switch and connecting to that switch to complete our trace.

Once we're in the 5th floor switch, run `sho cam xxxx.xxxx.xxxx` again, and we should get a different output, this time with a non trunked port.
```
VLAN  Dest MAC/Route Des    [CoS]  Destination Ports or VCs / [Protocol Type]
----  ------------------    -----  -------------------------------------------
65    xx-xx-xx-xx-xx-xx             5/9 [ALL]
Total Matching CAM Entries Displayed = 1
```

and now, `sho port 5/9` give s a more useable report.

```
Geo_65_5> sho port 5/9
* = Configured MAC Address

# = 802.1X Authenticated Port Name.

Port  Name                 Status     Vlan       Duplex Speed       Type
----- -------------------- ---------- ---------- ------ ----------- ------------
 5/9                       connected  65         a-full       a-1Gb 10/100/1000
```

so now you know what port your problem host is on! you can either read the name value, if we have set one, or you will have to trace that cable back to the patch-bay in the closet to find the location of the problem host.

    If you do have to trace it back ** please add a name to the port ** in question, something like `geology_4687_c_03` following the format: building_name/room_number/patch_panel/port_number
* You can use the command (from enabled mode) `set port name {module_number}/{port_number} {name_value}`
* For example: `set port name 5/9 geology_4687_c_03` for port c/03 in geology 4687 (my office)


### Additional info ###
Ports can shut themselves down if they error out. The command to re-enable them is:

`set port enable 5/12`

or sometimes we need to cut someone off:

`set port disable 5/12`

Also, if you have an empty port, or port unused port on another vlan and want to put it on vlan65 so you can use it, the command is:

`set vlan 65 5/12`

where the syntax is `set vlan {vlan_number} {module_number}/{port_number}`
