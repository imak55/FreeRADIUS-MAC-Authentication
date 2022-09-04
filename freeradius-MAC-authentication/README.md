Types de traductions
Traduction de texte
Texte source
2 654 / 5 000
Résultats de traduction
In this project, we tried to secure access to the internal network of a public institution. For this, we thought of working first with "port security" which just requires some configuration on the switch in order to control access based on a database of MAC addresses, but it was not adequate because it requires a modification each time on the switch. Our second solution was to install a FreeRADIUS server linked to a switch (plays the role of transmitting requests from machines wanting to connect to the server) and the machines. To perform authentication based on the physical addresses of the machines, we will make the following modifications in the FreeRADIUS configuration files under Ubuntu:
1)3.0/mods-available/files: here we added the "files authorized_macs {...} " part at the end of the file to create a new instance of the files module to read a new file of authorized MAC addresses.
2)3.0/authorized_macs: where we added the MAC addresses authorized to access the network.
3)3.0/sites-available/default: we added the following script in the "authorize {...}" part after the term "preprocess":
# If cleaning up the Calling-Station-Id...
        rewrite_calling_station_id

        # Now check against the authorized_macs file
        authorized_macs

        if (!ok) {
                # No match was found, so reject
                reject
        }
        else {
                # The MAC address was found, so update Auth-Type
                # to accept this auth.
                update control {
                        Auth-Type := Accept
                }
        }



Now that we have finalized the configuration of the FreeRADIUS server, we just have to configure the switch (Cisco in our case):
1) basic configuration of the switch:
###ip of the switch ####
interface vlan 1
ip address 172.16.1.253 255.255.255.0
exit
ip default-gateway 172.16.1.254


2) declare the Radius server
radius server "domain-name"
address ipv4 172.16.1.17 auth-port 1812 acct-port 1813
key hello
exit
###define the source interface for radius queries###
ip radius source-interface vlan 1
####to avoid errors in the logs#####
radius-server attribute 6 on-for-login-auth
3)aaa configuration
aaa authentication dot1x default group radius
aaa authorization network default group radius local
4)interface configuration
FastEthernet0/1 interface
switchport mode access
switchport nonegotiate
authentication port control auto
mab
spanning tree portfast
####test #######
debug radius


and There you go!
