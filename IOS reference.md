# Cisco IOS reference #

A number of the switches in the department use IOS rather than the older CatOS for configuration. There is more publically available IOS documentaiton, and for anything complex, you should refer to cisco's documentation. This guide is intended as a quick reference for common tasks in the department.



## Adding a module to a Catalyst (IOS) ##

After installing the module, and waiting to see that it is live and configured and accepted as enabled by the supervisor module (use the command `sho mod`) there are 3 steps you need to take to use the module.

(here i assume you use a gigabit ethernet switching module, other types will not be covered in this guide)

* make sure the module is enabled and passed initial checks with `sho mod` . under the  `Online Diag Status`  heading, make sure everything has `pass` and not `minor error` or another status.
* enter enabled mode (the command is `enable`)
* enter configuration mode (`conf t`) [the t is the default value for config via the terminal]
* your prompt should now show the `#` for enabled mode, and `(config)` should be the prompt.
* for this example we will configure the whole module the same way, but you can use the commands below to different smaller ranges.
* from config mode, we need to specify which interfaces we want to configure. Unlike on CatOS, where each port on the module is refered to by the command `port`, in IOS, each port on the module is a different interface. we will select a range of interfaces (or ports) on the module to configure all at once.
* issue the command ` int range gi2/1-48` where int says we will configure an interface, range specifies that we want to configure more than one at once, and the module is referred to by the 2 in this example. for a module in a different slot, use the module number from sho mod, or look at the bay number on the physical device.  The range I am using here starts at 1 and goes to 48, effectively spanning the entire module. For example, to configure module 6 we would use the command `int range gi6/1-48` to configure every port. To select a single interface, omit the `range` argument and give only a sigle interface like `int gi4/33`, which will configure port 33 on module 4.
* once the range is selected, your prompt will show `(config-if-range)` (if you are doing only one port/interface, the prompt should show `(config-if)`)
* from this point, the commands needed to configure the module will change, esp if you have added a new module, rahter than replaced the old one. Refering to the cisco docs, or a knowledgeable person is the best reference from this point. 

