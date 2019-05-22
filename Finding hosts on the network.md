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