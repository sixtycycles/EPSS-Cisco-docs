# Locating a problem host on the network. #

Many times, we are asked by IT services to look into a problem host, either for a virus, or some other problem, like a needed update, or exposed ports. To find these machines, we are given an IP and a time of detection. However, if the IP is in the DHCP range its both good and bad. We may need to look at the log server to get a copy of the logs from that time to get at the mac address of the host, because the host on the IP at this time may be completely different. 

Because IP's move around in our subnets thanks to DHCP, we need to get the mac address of the problem host. This action is done on the * core router * 

## Get the mac address: ##

Once we are remoted into the router, we will use the ARP tables to get the hardware address of the problem host. We can print the entire arp table with `show arp` but that is likely not useful for us, because it has EVERYTHING In there (very chatty!).

Instead, we can pipe the output to another command `include` (kinda like a primitive `grep`). Let's say the problem ip was `192.168.31.133`.

`show arp | include 133` will work, but will show all ips with 133 ANYWHERE in them, so to narrow it down, use:
`show arp | include 31\.133` where the `\` is to escape the period in order to be properly parsed.

Another example would be for 192.168.144.244:
`show arp | include 144\.244` which should return something like this:
```
  gs-cr>show arp | include 144\.244
  Internet  192.168.144.111        1   xxxx.bxxs.3s23  ARPA   Vlan05
```
If you want to be really fancy, you can use abbrevs like:
`sho arp | i 144\.244`

## Once you have the mac address: ##

Ok! Now we have the Mac address (copy it to clipboard or something) we can exit the router and start trying to chase the host down across the switches. Since we don't know what switch the host is on, we have to pick a place to start. 

We should start with the basement switch, as it is the closest to the divisional backbone.

After logging in, type ` show cam  xxxx.xxxx.xxxx` where xxxx.xxxx.xxxx is the copied mac address. This will give us some output like this:
```
  GEO_C65_B> sho cam xxxx.xxxx.xxxx
  * = Static Entry. + = Permanent Entry. # = System Entry. R = Router Entry.
  X = Port Security Entry $ = Dot1x Security Entry

  VLAN  Dest MAC/Route Des    [CoS]  Destination Ports or VCs / [Protocol Type]
  ----  ------------------    -----  -------------------------------------------
  05    xx-xx-xx-xx-xx-xx             4/2 [ALL]
  Total Matching CAM Entries Displayed  =1
```

From here we can see what Vlan this IP is on (usually 65, but we have others sometimes particularly with IGPP hosts) under "Destination Ports" you can see that this device is connected to Module 4, port 2. We can type `sho port 4/2` to see more information about this device.

```
SwtichA> sho port 4/2
* = Configured MAC Address

Port  Name                 Status     Vlan       Duplex Speed Type
----- -------------------- ---------- ---------- ------ ----- ------------
 4/2  some_other_switch   connected  trunk        full  1000 1000-LX/LH

... (more stuff we don't need below) ...
```


The output of show port, lets us know that this address is actually on a port that trunks to a different switch. In this case we should continue by exiting the remote session for this switch and connecting to that switch to complete our trace.

Once we're logged into the other switch, run `sho cam xxxx.xxxx.xxxx` again, and we should get a different output, this time with a non trunked port.
```
VLAN  Dest MAC/Route Des    [CoS]  Destination Ports or VCs / [Protocol Type]
----  ------------------    -----  -------------------------------------------
05    xx-xx-xx-xx-xx-xx             5/9 [ALL]
Total Matching CAM Entries Displayed = 1
```

and now, `sho port 5/9` give s a more useable report.

```
SwitchB> sho port 5/9
* = Configured MAC Address

# = 802.1X Authenticated Port Name.

Port  Name                 Status     Vlan       Duplex Speed       Type
----- -------------------- ---------- ---------- ------ ----------- ------------
 5/9                       connected  05         a-full       a-1Gb 10/100/1000
```

Now we know what port your problem host is on! You can either read the name value, if we have set one, or you will have to trace that cable back to the patch-bay in the closet to find the location of the problem host.

If you do have to trace it back ** please add a name to the port ** in question, something like `bldg_roomnum_port_03` following the format: building_name/room_number/port_ID

* You can use the command (from enabled mode) `set port name {module_number}/{port_number} {name_value}`
* For example: `set port name 5/9 buildinga_298_c03` for port c/03 in buildingA 289 (my office)


## Additional info ##
Ports can shut themselves down if they error out.(or if we shut them down) The command to re-enable them is:

`set port enable 5/12`

Sometimes we need to cut someone off:

`set port disable 5/12`

Also, if you have an empty port, or port unused port on another vlan and want to put it on vlan65 so you can use it, the command is:

`set vlan 05 5/12`

where the syntax is `set vlan {vlan_number} {module_number}/{port_number}`
