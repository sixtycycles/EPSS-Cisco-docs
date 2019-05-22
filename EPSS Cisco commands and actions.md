# Common commands and actions for Cisco Network hardware in EPSS #

## Network infrastructure in EPSS ##

We have 5 Cisco catalyst 6500's of varying ages.

There are two in Slichter in the 5th floor closet, 3 in Geology (basement, 3rd floor, 5th floor library). The Slichter switches take care of most of the former IGPP offices ect, whereas the Geology switches cover all floors of the geology building.

There are also 3 Cisco 4948 switches, 2 in the basement (telecom room and server room) and one in the 3rd floor closet. The important note about these is that the 4948's speak Cisco IOS whereas the older catalysts speak CatOS, the predecessor to IOS. IOS actions are significantly more google-able than CatOS, so this guide will focus on the CatOS commands initially. The Core router in Slichter also speaks IOS

## Getting Information ##
The management IP's and passwords for these devices are stored in lastpass as secure notes. Ask Rod for any credentials you need.

### If you are in the closet ###

If you are in the closet and want to use the console cable to administer the switch, you will need to run a command to pipe the serial output to your terminal emulator. On a mac, use GNU `screen` followed by the location of your usb port, and 9600 (the baud rate you will be reading).

To find your usb ports on a mac, see [this article](https://stackoverflow.com/questions/12254378/how-to-find-the-serial-port-number-on-mac-os-x#12260359)
Your final command should be something like this:
`screen /dev/tty.usbserial-A6004byf 9600`
and hit enter a few times.  You should be asked for a password.

### If you are remote ###

If you are on campus, but don't have time/don't feel like/don't want to go to a network closet, you can telnet into most of the switches Via their management IP. For security reasons this guide will not offer more information on this topic, but Rod can provide the credentials and IP's.


## What is going on with X? ##

Once you have logged into/connected to a catalyst switch, you will probably want to ensure that you know some things about the device.

Here are some commands in no particular order:

* `show module` or just `sho mod`
  * This will give you a list of all the modules installed and a top level view of their status (hopefully its OK all the way down!)
* `show port {module}/{port_number}`
  * This command will show you information about the ports on the switch. if you have traced a line form the panel to the switch this is the command to start diagnosing the issue.
* `sho vlan`
  * Prints the Vlans which are in use on the switch. It's important to note that you may have Vlans available that are just not in use on this particular switch.
* `enable` and `disable`
  * Enter and exit (respectively) from Enabled mode.
  * Enabled mode is for editing configurations. You can do no harm unless you enter enabled mode, so be sure you want to change things before being in enabled mode, and exit once you are done with changes! think of it like a cisco `sudo`
* `show cam xxx.xxx.xxx`
  * Shows information about mac address if it was seen on the switch
* `?`
  * The `?` is you friend! Typing `?` after a command like show, gives a list of available commands. This is different between enabled mode and non enabled mode, but can help if you forget some syntax or command.
