---
layout: post
title: "Lab 3 - The First Cisco Network Playbooks"
date: 2015-05-22
category: feature
tags: feature
---


So now we have some linux host and Cisco routers built. I have 3 linux 
hosts (1 x ansible, 2 x endpoints) plus 2 Cisco routers. At present 
should be connected to the VM Network port group, so before we can 
move further forward lets create the VMWare networking that will be 
used to plumb everything together. So create the following on your esxi 
host:

Port Goup				VLAN ID		vSwitch			Comment
VM Network				0			vswitch0
Management Network		0			vswitch0

vlan10					10			vlan10			Site 1
vlan20					20			vlan20			Site 2
vlan30					30			vlan30			Site 3
vlan100					100			vlan100			Site 1 WAN uplink
vlan200					200			vlan200			Site 2 WAN uplink
vlan300					300			vlan300			Site 3 WAN uplink
vlan1002				1002		vlan1002		WAN100 <--> WAN200
vlan1003				1003		vlan1003		WAN200 <--> WAN300
vlan2003				2003		vlan2003		WAN200 <--> WAN300
external				150			external		outside interface

So we are only going to need VLAN10 and VLAN100 for this lab, but we at
least have the infrastructure all ready as we grow the lab.

When the VM's are re-configured to use the VLANs in this lab, there 
will be a loss of connectivity to the Internet, but we will re-introduce 
this at a later stage.

So now lets begin by provisioning the VM's into VLAN10 as follows:

RedHat1 172.16.10.1/24 gw 172.16.10.254
Test1 172.16.10.4/24 gw 172.16.10.254
Test10 172.16.10.5/25 gw 172.16.10.254
CSR10 172.16.10.252/24
CSR11 172.16.10.253/24

We will carry out this manually as it requires moving from one IP 
subnet to another so we want to avoid any loss of service.

So no check that you can ping and SSH form the ansible host to the vlan 
10 IP address on each VM.

Go to: https://github.com/craigeowen/my-network-ansible_playbooks/blob/master/7a.%20Site1-setup.txt
for a simple playbook that will test ansible against each of the site 1
routers. Here it is reproduced with a description of the component 
parts (the stuff following the #) -

--- #yaml files begin with 3 dashes
- hosts: er-site1 # the group in the inventroy that contains the Site 1 routers
  gather_facts: no #indent by 2 - DO NOT gather ansible facts
  connection: local # use local connection method (this is being deprecated)
  
  tasks: #The actions to be carried out
  - name: show version # Name of task
    ios_facts: # use built-in module from IoS Network Module
      gather_subset: hardware # only apply this subset
      provider: # the variables needed for login to the router follow
        username: cisco # router login username
        password: P4ssw0rd! # router password associated with username
        authorize: true # use enable
        auth_pass: P4ssw0rd! # enable secret

  - name: print out the IOS version
    debug: # use debug ansible module
      var: ansible_net_version # return ios version
      
Run the playbook at the command prompt (ensure you are in the directory 
containing the playbook!) -
ansible-playbook site1_showVer.yml

#<add screenshots>

playbook tasks runs as follows:

TASK [show ver] # name of task in playbook
#hosts that the playbook is running against

TASK [print out the ios version] #debug task
#IOS versions are output to screen for each host










CSR10 
lo1 1.0.10.1/32
lo10 1.0.10.10/32
lo100 1.0.10.100/32
GE1 172.16.10.252/24
GE2 10.10.10.10/24

CSR11 
lo1 1.0.11.1/32
lo10 1.0.11.10/32
lo100 1.0.11.100/32
GE1 172.16.10.253/24
GE2 10.10.10.11/24

