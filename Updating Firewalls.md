# How to do stuff with ACL's on the core router #

Each Vlan has 2 ACLS, an one inbound and one outbound.

This refers to the direction of traffic from the router's perspective.  So, the outbound rules apply to traffic leaving the router and entering the Vlan and inbound rules apply to traffic entering (going IN) the router from the Vlan.

So for my Vlan 65 (epss main vlan), my ACL's are 102 (outbound) and 103 (inbound)

You can find this information by using the command: `show ip interface | include line protocol | access list` which will show you what ACL's are on what VLANs.
`show ip int` is the shorthand for the first part of this command, and will dump a LOT of information on you, so i use the following pipes to limit the output

My result is:
```
...
Vlan65 is up, line protocol is up
  Outgoing access list is 102
  Inbound  access list is 103
...
  ```

*** This command is also very useful to see if your attempt to re-apply the ACL after an update the ACL has been successful. ***

### To obtain a copy of the ACL as it exists on the interface: ###
* Enter enabled mode
* Then: `sho run | begin list 102` to see the running configuration starting at the ACL 102. 
  * Probably, you should copy this to a text editor before you do anything else. 
  * Another useful tool is to say ` sho access-list 102` which will show you how many times a particular rule is "hit" on the ACL. The caveat, is that this command does not display remarks, whereas ` sho run` does. 


```
telnet x.x.x.x

Trying xxx.xx.xxx.xxx...
Connected to REDACTED.
Escape character is '^]'.


User Access Verification

Password: Kerberos:	No default realm defined for Kerberos!

Password: <entered - access password>
gs-cr>
```
then `enable` to entered enabled mode and then `show run | begin list 102` to see the rules for list 102.

I copy these to a text file for a backup, and also to edit rules somewhere that is not the router.

after making the updates and saving a copy of the original, (in case something strange goes very wrong...) I'm ready to replace the old ACL with the new one.

from enabled mode enter:

`conf t`

now you are in config mode, using the default value 'terminal' (the t is short for this)

Once you have done that the prompt should change to `(config)` 

Now you can select interface to edit, in this case, vlan 65

`int vlan65`

which should change the prompt to `(config-int)` letting you know you are editing a specific interface. 

These commands remove the applied ACL from the interface, and then delete the ACL from the router memory.
#### DO NOT DO THIS WITHOUT FIRST HAVING A LOCAL COPY OF THE ACL SAVED ####
```
no ip access-group 102 out
no access-list 102
```
Now the prompt should say `(config)` instead of `(config-if)`

Now you can paste your ACL rules from the text file to the terminal, watching for errors. if all goes well, re enter the interface by typing:
```
int vlan 65
ip access-group 102 out
```
Then type
```
exit
```
To leave the interface, and then 
```
exit
```
Again to exit config mode. 
Now you need to write changes to memory with:
```
wr
```
And now you can exit enabled mode with
```
disable
```
Now make sure everything looks good with:
``` show access-list 102 ```
where `102` can be the number of whatever list you are editing. 
If it all looks good: You DID IT! Awesome! 
