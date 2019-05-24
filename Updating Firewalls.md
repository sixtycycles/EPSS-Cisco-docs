# How to do stuff with ACL's on the core router #


First off, each Vlan has 2 ACLS, an inbound and an outbound.

The best explanation I have is that the outbound rules apply to traffic leaving the router and entering the Vlan and inbound rules apply to traffic entering (going IN) the router from the Vlan.

So for my Vlan 65 (epss main vlan), my ACL's are 102 (outbound) and 103 (inbound)

You can find this information by using the command: `show ip interface | include line protocol | access list` which will show you what ACL's are on what VLANs.

My result is:
```
...
Vlan65 is up, line protocol is up
  Outgoing access list is 102
  Inbound  access list is 103
...
  ```

*** This command is also very useful to see if your attempt to update the ACL has been successful. ***

To obtain a copy of the ACL as it exists on the interface:
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

from enabled mode:
```
conf t

# select interface to edit, in this case, vlan 65

int vlan65

# this removes the existing ACL.

no ip access-group 102 out
no access-list 102

 exit
```
now the prompt should say `(config)` instead of `(config-if)`

now you can paste your ACL rules from the text file to the terminal, watching for errors. if all goes well, re enter the interface by typing:
```
int vlan 65
ip access-group 102 out
exit
# now back in (config)
exit
#write changes:
wr
#leave!
qu
```
You DID IT!




also:
show ip interface | include line protocol|access list
