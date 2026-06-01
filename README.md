
# md-ws
#Nokia Workshop

This page provides the basic step-by-step configuration required to configure the 7750 SR-1 Router.

| Contributors | email |
|---|---|
| Scott Ralph | scott.ralph@nokia.com |


All configurations are in MD-CLI flat format. Reference chassis is 7750 SR-1 and software version is SR OS 26.3.R1. Use `show system info` command to verify your router's chassis model and software version.

# Two Options for Lab Access

1) **Access the Lab Remotely**

Information coming

2) **Create Your Own Container Lab**

If you have a Linux enviroment on your laptop or in a lab such as WSL you can use containerlab to use the same topology used for this lab.

   1) Have a Linux Enviroment- https://learn.microsoft.com/en-us/windows/wsl/install (Optional)
   2) Install Containerlab into your enviroment.
      ```
      curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
   3) Download the 7750-SR1.yml , SR-SIM image, and license.txt file into a folder.
   4) Once downloaded you need to create the container image.
      ```
      docker image load -i srsim.tar.xz
      ```
   6) Within the folder issue the following command:
      ```
      clab deploy -t 7750-SR1.yml

   7) Once the lab is finished being deployed you can issue the following command if it does not already display.
      ```
      clab inspect -a
<img width="544" height="129" alt="clab inspect" src="https://github.com/user-attachments/assets/fe526cb7-8db6-417b-a0c7-297218b2a76e" />

    6) You can now SSH to the container using the following credetials "admin/NokiaSros1!"
    
      ```
      ssh admin@pe20
      
                                                           
   # Workshop Lab

The following Subjects are covered in this lab:

- [Lab 1 BOF File](#lab-1-bof-file)
- [Lab 2 Provisioning the Cards and MDA](#lab-2-provisioning-the-cards-and-mda)
- [Port Configuration](#VPRN)
- [Router Base Interface](#IES)
- [ISIS ](#EVPN-VPWS)
- [BGP](#EVPN-MPLS-with-Multihoming)

A list of [show commands](#show-commands) is also provided in this guide.

A summary of what this guide provides is shown below.



Disclaimer: This is not an exhaustive list of all the features and associated options on SR OS for services. This does not replace official documentation but is a one stop reference guide for basic service configuration. For more details on the features and options, please refer to the documentation links in each section.

# Topology

This is the topology used for the lab exercise.

<img width="1486" height="224" alt="Topology" src="https://github.com/user-attachments/assets/c38d9e5f-9b35-401d-96da-b488d90e3325" />





IPv4 Addressing:

<img width="251" height="106" alt="IPAddress" src="https://github.com/user-attachments/assets/4cf76d44-9d89-4bae-aafe-9df54502f98b" />



# Lab 1 BOF File
1) Log into the Router PE20 with username admin password NokiaSros1!

```
ssh admin@pe20

 ```

2) At the prompt enter bof configuration mode.

```bash
bof private

```


3) Enter the DNS information.

```bash
dns primary-server 8.8.8.8
commit

 ```                      
4) Check to make the BOF file has been updated0

```
info flat

```

5) You should see output similar to below:
<img width="813" height="231" alt="image" src="https://github.com/user-attachments/assets/01d5577c-7bb0-4c8a-9807-ab85ee9cb04a" />

6) Repeat the steps for PE30.
   
# Lab 2 Provisioning the Cards and MDA

1) On PE20 issue the commands to display the state of the cards and MDA.

```
show card
show mda

```
You show get a similar output
<img width="926" height="539" alt="image" src="https://github.com/user-attachments/assets/f821229b-a8cf-406e-bc46-ca93320a07cf" />

* Please note that because this is an SR-SIM the IOM is provisioned automatically.

2)   Configure the IOM ( already configured)
   ```
/configure card 1 card-type iom-1
/configure card 1 level he
commit

```

3) Show that the card is now up.

   <img width="775" height="243" alt="image" src="https://github.com/user-attachments/assets/4fa2568b-85f3-4e62-a257-1e318751b9ce" />

4) View the port status

   ```
   show port
   ```
   You will notice that only the CPM is up.

   <img width="901" height="306" alt="image" src="https://github.com/user-attachments/assets/2c7f81e7-50ad-46b4-a094-9afdbcf900db" />


5) Provision both the 16 port MACSEC with 2 QSFP28 ports and the 6 port QSFP28 MDAs.

```
  /configure card 1 mda 1 mda-type me16-25gb-sfp28+2-100gb-qsfp-b
  /configure card 1 mda 2 mda-type me6-100gb-qsfp28
   commit
   
```
6) You should now see the MDA up.
   <img width="807" height="194" alt="image" src="https://github.com/user-attachments/assets/dcd53444-13ae-40ca-871a-acc8738a42a1" />

 7) The ports can now be viewed.
      <img width="733" height="667" alt="image" src="https://github.com/user-attachments/assets/64314dad-34d5-447a-bb99-6d4c22dffc8a" />

 8) Configure the connectors to be 10G ports. Only 1/1/c1 and 1/1/c2 are connected in this lab.
   ```
    /configure port 1/1/c1 connector breakout c1-10g admin-state enable
    /configure port 1/1/c2 connector breakout c1-10g admin-state enable
   commit
   ```
9) Bring the ports administratively up.

    ```
    /configure port 1/1/c1 admin-state enable
    /configure port 1/1/c1/1 admin-state enable
    /configure port 1/1/c2 admin-state enable
    /configure port 1/1/c2/1 admin-state enable
    commit
    
    ```
    
10) The show ports command will now show 2 x 10G interfaces. Please note there will not be a link until you complete the same task on PE30.
    ```
    show port
    ```
<img width="759" height="335" alt="image" src="https://github.com/user-attachments/assets/04e2fe80-7d80-4c82-896e-2432147a15cc" />

  12) Please repeat the same task on PE30

# Lab 3 Create LAG Interfaces

1) The ports 1/1/c1/1 and 1/1/c2/1 are both going be using VLAN so the need to be configured to accept VLANS.
   ```
   /configure port 1/1/c1/1 ethernet encap-type dot1q
    /configure port 1/1/c2/1 ethernet encap-type dot1q
   commit


   ```

2) You will now see the ports are dot1q.
      <img width="781" height="268" alt="image" src="https://github.com/user-attachments/assets/ef253a5d-954f-498f-bc6c-97aabc0e50c8" />

3) Create a LAG and apply the configured interfaces to the LAG
   ```
   /configure lag "lag-1" admin-state enable
    /configure lag "lag-1" encap-type dot1q
    /configure lag "lag-1" lacp-xmit-interval fast
    /configure lag "lag-1" lacp mode active
    /configure lag "lag-1" lacp administrative-key 3
    /configure lag "lag-1" port 1/1/c1/1 { }
    /configure lag "lag-1" port 1/1/c2/1 { }
   commit
   ```

   4) Show LAG to see that are are operational. Do a "show lag detail" to see more information.
  ```
show lag detail
```
   <img width="843" height="278" alt="image" src="https://github.com/user-attachments/assets/9a02f544-2b23-45b5-b25e-f6cb745cc946" />

5) Perform the same tasks on PE30

   # Lab4- Create Router "Base" Interfaces

1) Create a router interface in the Base instance on PE20
```
/configure router "Base" interface "To-Pe30" admin-state enable
/configure router "Base" interface "To-Pe30" port lag-1:200
/configure router "Base" interface "To-Pe30" ipv4 primary address 10.10.1.1
/configure router "Base" interface "To-Pe30" ipv4 primary prefix-length 30
commit

```
2) Create a router interface on PE30
```
    /configure router "Base" interface "To-Pe20" admin-state enable
    /configure router "Base" interface "To-Pe20" port lag-1:200
    /configure router "Base" interface "To-Pe20" ipv4 primary address 10.10.1.2
    /configure router "Base" interface "To-Pe20" ipv4 primary prefix-length 30
    commit
```

3) Show router interface. This will show the router interface is up and the lag port is attached.
```
show router interface
```
<img width="804" height="308" alt="image" src="https://github.com/user-attachments/assets/553ebe94-926d-465d-ab73-d0bb4f47e5ed" />

4) Configure the System Interface on PE20
```
configure router "Base" interface "system" ipv4 primary address 100.100.100.2
/configure router "Base" interface "system" ipv4 primary prefix-length 32
commit

```
5) Configure the System Interface on PE30
   ```
    /configure router "Base" interface "system" ipv4 primary address 100.100.100.2
    /configure router "Base" interface "system" ipv4 primary prefix-length 32
   commit

   ``

6) Issue the ping command to verify connectivity. Please note the system interfaces will not be reachable until a routing protocol is configured.
```
ping 10.10.1.2
```

   <img width="576" height="173" alt="image" src="https://github.com/user-attachments/assets/0e3a8830-8d15-4b6f-9a39-87795366fb79" />


# Lab 5- Configure ISIS

1) Enter into the "router" context. This is where all routing protocols are configured including ISIS.

   ```
    /configure router "Base" isis 0 { }
   ```
2) In the router isis context we want to set the ISIS area and interfaces that you want to advertise.
```
/configure router "Base" isis 0 admin-state enable
/configure router "Base" isis 0 area-address [49.0001]
/configure router "Base" isis 0 interface "To-Pe30" interface-type point-to-point
/configure router "Base" isis 0 interface "system" { }
   commit

```
3) Repeat the same steps on router PE-30.
```
   /configure router "Base" isis 0 admin-state enable
    /configure router "Base" isis 0 area-address [49.0001]
    /configure router "Base" isis 0 interface "To-Pe20" admin-state enable
    /configure router "Base" isis 0 interface "To-Pe20" interface-type point-to-point
    /configure router "Base" isis 0 interface "system" { }
     commit

```

4) To verify the ISIS adjanency is up and routes are being advertied enter " show router isis adjacency" .
```
show router isis adjacency
```
<img width="674" height="188" alt="image" src="https://github.com/user-attachments/assets/7d09bbe2-e215-46a8-83b9-783670fd8cb7" />

5) To verify that ISIS is advertising the system interface addresses.
```
show router route-table
```
<img width="764" height="338" alt="image" src="https://github.com/user-attachments/assets/6fc86b4f-a8d6-49d0-b57f-9d187a0fa930" />

7) To add traffic engineering capabilities that will be required for RSVP-TE lab. Add the command on both PE20 and PE30

```
/configure router "Base" isis 0 traffic-engineering true
commit

```

# Lab 6- Configuring BGP

1) Using the router context we will enter into BGP router context. A BGP group is mandatory and then attached to a "neighbor" statement. In this lab we will be iBGP and will use the system address to peer with. The autonomous number is added under the "router" context.

   ```
    /configure router "Base" autonomous-system 65001
     commit
   
   ```

   
2) BGP configuration on PE20.
```

    /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp group "iBGP" family ipv4 true
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp neighbor "100.100.100.3" group "iBGP"
    /configure router "Base" bgp neighbor "100.100.100.3" peer-as 65001
     commit
```
3) BGP Configuration on PE30.
   ```
   /configure router "Base" bgp admin-state enable
    /configure router "Base" bgp group "iBGP" family ipv4 true
    /configure router "Base" bgp group "iBGP" family vpn-ipv4 true
    /configure router "Base" bgp neighbor "100.100.100.3" group "iBGP"
    /configure router "Base" bgp neighbor "100.100.100.3" peer-as 65001
   commit
   
   ```
4) Verify that the BGP adjaceny is up. We see it up and both address families are not sending or receiving routes.
```
show router bgp summary all
```

<img width="718" height="252" alt="image" src="https://github.com/user-attachments/assets/b125cd22-4248-450c-af0d-50264e9ef027" />

6) The show router bgp neighbor <address> detail can show the router is Established. 

<img width="657" height="543" alt="image" src="https://github.com/user-attachments/assets/386afcc8-d7da-4a78-a5e0-981e5b382108" />

7) Adding three Loopback interfaces to the routers configuration.
   PE-20
```
    /configure router "Base" interface "Loopback1" loopback
    /configure router "Base" interface "Loopback1" ipv4 primary address 20.20.20.1
    /configure router "Base" interface "Loopback1" ipv4 primary prefix-length 32
    /configure router "Base" interface "Loopback2" loopback
    /configure router "Base" interface "Loopback2" ipv4 primary address 20.20.30.1
    /configure router "Base" interface "Loopback2" ipv4 primary prefix-length 32
    /configure router "Base" interface "Loopback3" loopback
    /configure router "Base" interface "Loopback3" ipv4 primary address 10.10.40.1
    /configure router "Base" interface "Loopback3" ipv4 primary prefix-length 32
    commit

```
PE-30
```
   /configure router "Base" interface "Loopback1" loopback
    /configure router "Base" interface "Loopback1" ipv4 primary address 20.20.20.1
    /configure router "Base" interface "Loopback1" ipv4 primary prefix-length 32
    /configure router "Base" interface "Loopback2" loopback
    /configure router "Base" interface "Loopback2" ipv4 primary address 20.20.30.1
    /configure router "Base" interface "Loopback2" ipv4 primary prefix-length 32
    /configure router "Base" interface "Loopback3" loopback
    /configure router "Base" interface "Loopback3" ipv4 primary address 10.10.40.1
    /configure router "Base" interface "Loopback3" ipv4 primary prefix-length 32
     commit

```
7) Verify that the interfaces are up on both PE-20 and PE-30.
```
show router interface
```

   <img width="694" height="367" alt="image" src="https://github.com/user-attachments/assets/e9f2ba53-2c20-486c-a24c-d1718bd34fbf" />

   
9) Create the following prefix-lists and policies to advertise the Loop back interfaces into BGP.
   PE-20
```
/configure policy-options prefix-list "Loopbacks" { prefix 20.20.20.1/32 type exact }
    /configure policy-options prefix-list "Loopbacks" { prefix 20.20.30.1/32 type exact }
    /configure policy-options prefix-list "Loopbacks" { prefix 20.20.40.1/32 type exact }
    /configure policy-options policy-statement "Export" entry-type named
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopbacks" from prefix-list ["Loopbacks"]
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopbacks" from protocol name [direct]
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopbacks" action action-type accept
      commit

```
PE-30
```
    /configure policy-options prefix-list "Loopbacks" { prefix 30.30.30.1/32 type exact }
    /configure policy-options prefix-list "Loopbacks" { prefix 30.30.40.1/32 type exact }
    /configure policy-options prefix-list "Loopbacks" { prefix 30.30.50.1/32 type exact }
    /configure policy-options policy-statement "Export" entry-type named
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopback" from prefix-list ["Loopbacks"]
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopback" from protocol name [direct]
    /configure policy-options policy-statement "Export" named-entry "BGP-Loopback" action action-type accept
      commit

```
9) Apply the policy to the neighbor statement. Context is the same for both PE-20 and PE-30.

   ```
   /configure router "Base" bgp neighbor "100.100.100.2" export policy ["Export"]
   ```

10) Verify that the router is receiving routes and the are active in the routing table.
       <img width="781" height="251" alt="image" src="https://github.com/user-attachments/assets/83a70df7-73a5-4dcf-bd77-41bcf140b641" />
       <img width="763" height="534" alt="image" src="https://github.com/user-attachments/assets/6f6d5776-71ad-412f-98d4-38e50b44d1e3" />
       <img width="799" height="473" alt="image" src="https://github.com/user-attachments/assets/b4e73775-c908-4a57-8776-63b18024aeee" />
       <img width="711" height="470" alt="image" src="https://github.com/user-attachments/assets/ba036e96-55fb-4ebe-b663-f5fbc6b21bf3" />

# RSVP-TE Configuration

1) The interface to both PE routers needed to be added to the MPLS and RSVP command contexts.
  PE-30
   ```
   /configure router "Base" mpls admin-state enable
    /configure router "Base" mpls interface "To-Pe20" admin-state enable
    /configure router "Base" rsvp admin-state enable
    /configure router "Base" rsvp interface "To-Pe20" admin-state enable
   commit

   
   ```
   PE-20
   ```
    /configure router "Base" mpls admin-state enable
    /configure router "Base" mpls interface "To-Pe30" admin-state enable
    /configure router "Base" rsvp admin-state enable
    /configure router "Base" rsvp interface "To-Pe30" admin-state enable
   commit

   ```
3)  Configure the RSVP-TE LSP that will be used as the transport protocol.

   PE-20
```
    /configure router "Base" mpls admin-state enable
    /configure router "Base" mpls path "loose" admin-state enable
    /configure router "Base" mpls lsp "To-PE30" admin-state enable
    /configure router "Base" mpls lsp "To-PE30" type p2p-rsvp
    /configure router "Base" mpls lsp "To-PE30" to 100.100.100.3
    /configure router "Base" mpls lsp "To-PE30" path-computation-method local-cspf
    /configure router "Base" mpls lsp "To-PE30" { primary "loose" }
     commit


```
PE-30
```
   /configure router "Base" mpls admin-state enable
    /configure router "Base" mpls path "loose" admin-state enable
    /configure router "Base" mpls lsp "To-PE20" admin-state enable
    /configure router "Base" mpls lsp "To-PE20" type p2p-rsvp
    /configure router "Base" mpls lsp "To-PE20" to 100.100.100.2
    /configure router "Base" mpls lsp "To-PE20" path-computation-method local-cspf
    /configure router "Base" mpls lsp "To-PE20" { primary "loose" }
     commit

```

4) Verify the LSP is up and operational.
<img width="672" height="217" alt="image" src="https://github.com/user-attachments/assets/0f2a6629-033c-423d-8fda-4abc038a045b" />
<img width="662" height="544" alt="image" src="https://github.com/user-attachments/assets/90fa8772-8c61-4944-8f1a-955da43cd57a" />


# Lab 7-VPRN-Configuration 

1) Configure a VPRN instance named "OPEN" with a service-id of 200 with 3 x Loopback Interfaces in the VPRN instance. The vrf-targe will be 100:10 with unique route-distinguishers.

PE-30

```
     /configure service vprn "OPEN" admin-state enable
    /configure service vprn "OPEN" service-id 200
    /configure service vprn "OPEN" customer "1"
    /configure service vprn "OPEN" bgp-ipvpn mpls admin-state enable
    /configure service vprn "OPEN" bgp-ipvpn mpls route-distinguisher "172.16.1.1:10"
    /configure service vprn "OPEN" bgp-ipvpn mpls vrf-target community "target:100:1"
    /configure service vprn "OPEN" bgp-ipvpn mpls auto-bind-tunnel resolution filter
    /configure service vprn "OPEN" bgp-ipvpn mpls auto-bind-tunnel resolution-filter rsvp true
    /configure service vprn "OPEN" interface "Loopback1" loopback true
    /configure service vprn "OPEN" interface "Loopback1" ipv4 primary address 200.200.200.1
    /configure service vprn "OPEN" interface "Loopback1" ipv4 primary prefix-length 32
    /configure service vprn "OPEN" interface "Loopback2" loopback true
    /configure service vprn "OPEN" interface "Loopback2" ipv4 primary address 200.200.200.2
    /configure service vprn "OPEN" interface "Loopback2" ipv4 primary prefix-length 32
    /configure service vprn "OPEN" interface "Loopback3" admin-state enable
    /configure service vprn "OPEN" interface "Loopback3" loopback true
    /configure service vprn "OPEN" interface "Loopback3" ipv4 primary address 200.200.200.3
    /configure service vprn "OPEN" interface "Loopback3" ipv4 primary prefix-length 32
      commit
```

PE-20
```
/configure service vprn "OPEN" admin-state enable
    /configure service vprn "OPEN" service-id 200
    /configure service vprn "OPEN" customer "1"
    /configure service vprn "OPEN" bgp-ipvpn mpls admin-state enable
    /configure service vprn "OPEN" bgp-ipvpn mpls route-distinguisher "172.16.1.1:11"
    /configure service vprn "OPEN" bgp-ipvpn mpls vrf-target community "target:100:1"
    /configure service vprn "OPEN" bgp-ipvpn mpls auto-bind-tunnel resolution filter
    /configure service vprn "OPEN" bgp-ipvpn mpls auto-bind-tunnel resolution-filter rsvp true
    /configure service vprn "OPEN" interface "Loopback1" loopback true
    /configure service vprn "OPEN" interface "Loopback1" ipv4 primary address 200.200.100.1
    /configure service vprn "OPEN" interface "Loopback1" ipv4 primary prefix-length 32
    /configure service vprn "OPEN" interface "Loopback2" admin-state enable
    /configure service vprn "OPEN" interface "Loopback2" loopback true
    /configure service vprn "OPEN" interface "Loopback2" ipv4 primary address 200.200.100.2
    /configure service vprn "OPEN" interface "Loopback2" ipv4 primary prefix-length 32
    /configure service vprn "OPEN" interface "Loopback3" loopback true
    /configure service vprn "OPEN" interface "Loopback3" ipv4 primary address 200.200.100.3
    /configure service vprn "OPEN" interface "Loopback3" ipv4 primary prefix-length 32
      commit
```

2) Verify that you are receiving the routes.
   The BGP is now receiving VPN-IPV4 routes
   
   <img width="693" height="276" alt="image" src="https://github.com/user-attachments/assets/90683676-4149-4457-b152-5883db64dda2" />
   
   show BGP neighbor show the receiving routes.
   
   <img width="714" height="475" alt="image" src="https://github.com/user-attachments/assets/2460be5c-e614-48d6-a347-fbe57d469bf0" />
   
   The VRF route table for the instance OPEN has the routes in the route-table
   
   <img width="697" height="454" alt="image" src="https://github.com/user-attachments/assets/ae2391a9-fbeb-4f26-b80e-03f283ef9f14" />

   
# End of Labs

   

# MD-CLI Command Reference

Below is a reference table with some commonly used commands for CLI navigation.

| Action | Command |
| --- | --- |
| Enter candidate mode | `configure private` |
| Commit configuration changes | `commit` |
| Delete configuration elements | `delete {command path}` |
| Discard configuration changes | `discard` |
| Compare candidate to running | `compare /` |
| View configuration in current context | `info` |
| View configuration in flat format | `info full-context` |
| View full configuration of router | `admin show config` |
| Search for keyword in output | `<command> \| match {keyword}` |
| Find a command | `tree flat detail \| match <keyword>` |
| Exit candidate mode | `exit` |
| Exit from router | `logout` |

