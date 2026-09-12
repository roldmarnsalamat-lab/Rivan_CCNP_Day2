
<!-- Your monitor number = 21 -->


## â›… Warm Up for Day 2.

<br>

### ðŸ”§ Physically Connect the following:
| Device   | Port       |  -  | Port   | Switch   |
| ---      | ---        | --- | ---    | ---      |
| PC       | TunayNaLAN |  -  | fa0/1  | CoreBABA |
| WLC      | PoE        |  -  | fa0/2  | CoreBABA |
| AP       | PoE        |  -  | fa0/4  | CoreBABA |
| CUCM     | fe0/0      |  -  | fa0/3  | CoreBABA |
| ePhone 1 | Network    |  -  | fa0/5  | CoreBABA |
| ePhone 2 | Network    |  -  | fa0/7  | CoreBABA |
| Cam6     | Eth        |  -  | fa0/6  | CoreBABA |
| Cam8     | Eth        |  -  | fa0/8  | CoreBABA |
| CoreTAAS | fa0/10     |  -  | fa0/10 | CoreBABA |
| CoreTAAS | fa0/11     |  -  | fa0/11 | CoreBABA |
| CoreTAAS | fa0/12     |  -  | fa0/12 | CoreBABA |


<br>
<br>

---
&nbsp;

## Day 1 via Ansible

![Ansible](img/Ansible.png)

<br>
<br>

### Principles
1. __Agent-less Architecture__ - Low maintenance overhead by avoiding the installation of additional software across IT infrastructure.

<br>

2. __Simplicity__ - Automation playbooks use straightforward YAML syntax for code that reads like documentation. Ansible is also decentralized, using SSH with existing OS credentials to access to remote machines.

<br>

3. __Scalability and Flexibility__ - Easily and quickly scale the systems you automate through a modular design that supports a large range of operating systems, cloud platforms, and network devices.

<br>

4. __Idempotence and predictability__ - When the system is in the state your playbook describes, Ansible does not change anything, even if the playbook runs multiple times.


&nbsp;
---
&nbsp;


### Step 1 - Add routing to PC
~~~
!@cmd
route add 10.0.0.0 mask 255.0.0.0 10.21.1.4
route add 200.0.0.0 mask 255.255.255.0 10.21.1.4
~~~

<br>

### Step 2 - Add ip address and routing to device
~~~
!@coreTaas
conf t
 int vlan 1
  no shut
  ip add 10.21.1.2 255.255.255.0
  desc mgmtData-configuredManually
  exit
  enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@coreBaba
conf t
 int vlan 1
  no shut
  ip add 10.21.1.4 255.255.255.0
  desc mgmtData-configuredManually
  exit
 vlan 100
  name VOICEVLAN
  exit
 int vlan 100
  no shut
  ip add 10.21.100.4 255.255.255.0
  desc vlanMgmtVoice-configuredManually
  exit
 int fa 0/3
  sw mo ac
  sw ac vlan 100
  exit
 int gi0/1
  no switchport
  no shut 
  ip add 10.21.21.4 255.255.255.0
  exit
 ip routing
 ip route 0.0.0.0 0.0.0.0 10.21.21.1 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@cucm
conf t
 int fa0/0
  no shut
  ip add 10.21.100.8 255.255.255.0
  exit
 ip routing
 ip route 0.0.0.0 0.0.0.0 10.21.100.4 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

<br>

~~~
!@edge
conf t
 int gi 0/0/0
  ip add 10.21.21.1 255.255.255.0
  no shut
  exit
 ip routing
 ip route 10.21.0.0 255.255.0.0 10.21.21.4 120
 enable secret pass
 line vty 0 14
  password pass
  transport input all
  login
  exec-timeout 0 0
  end
~~~

&nbsp;
---
&nbsp;

### Step 3 - Telnet to each device
On SecureCRT:
| IP                 | Device   |
| ---                | ---      |
| 10.21.1.2	     | CoreTaas |
| 10.21.1.4      | CoreBaba |
| 10.21.100.8    | CUCM     |
| 10.21.21.1 | EDGE     |

&nbsp;
---
&nbsp;

### Step 4 - Note MAC Addresses
~~~
@CoreBaba
sh mac address-table
~~~

<br>


| Device               | Port  | MAC         |
| ---                  | ---   | ---         |
| Camera 6 MAC Address | fa0/6 | ___.___.___ |
| Camera 8 MAC Address | fa0/8 | ___.___.___ |
| Ephone 1 MAC Address | fa0/5 | ___.___.___ |
| Ephone 2 MAC Address | fa0/7 | ___.___.___ |

&nbsp;
---
&nbsp;

### Step 4 - Enable SSH
For an SSH connection to be established, the device must have:  
| Description                      | Command                                  |
| ---                              | ---                                      |
| a non-default hostname           | hostname coreBaba21                  |
| a domain name                    | ip domain name day1lab.com               |
| a local user account             | username admin privilege 15 secret pass  |
| generated crypto keys            | crypto key generate rsa modulus 2048     |
| SSH enabled                      | ip ssh version 2                         |
| enable remote access             | transport input all                      |
| enable remote user/pass login    | login local                              |

<br>

Extra lines  
| Description             | Command                      |
| ---                     | ---                          |
| password encryption     | service password-encryption  |
| no logs                 | no logging console           |
| no domain lookups       | no ip domain-lookup          |
| no timeout              | exec-timeout 0 0             |

<br>

~~~
!@coreTaas
conf t
 hostname coreTaas-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
  exit
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@coreBaba
conf t
 hostname coreBaba-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
  exit
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@cucm
conf t
 hostname cucm-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

<br>

~~~
!@edge
conf t
 hostname edge-21
 service password-encryption
 no logging console
 no ip domain-lookup
 ip domain name autoday1.com
 username admin privilege 15 secret pass
 line vty 0 14
  transport input all
  login local
  exec-timeout 0 0
 crypto key generate rsa modulus 2048 label devs
 ip ssh rsa keypair-name devs
 ip ssh version 2
 end
~~~

&nbsp;
---
&nbsp;

### Step 5 - Run Virtual Machines
NetOps:  
  Name: NetOps-PH  
  
  | NetAdapter   |                    |
  | ---          | ---                |
  | NetAdapter   | NAT                |
  | NetAdapter 2 | VMNet2             |
  | NetAdapter 3 | VMNet3             |
  | NetAdapter 4 | Bridge (Replicate) |

<br>

CSR1000v:  
  Name: UTM-PH  
  
  | NetAdapter   |        |
  | ---          | ---    |
  | NetAdapter   | NAT    |
  | NetAdapter 2 | VMNet2 |
  | NetAdapter 3 | VMNet3 |

&nbsp;
---
&nbsp;


### Step 6 - Set IP address and Routing
~~~
!@UTM-PH
conf t
 hostname UTM-PH
 enable secret pass
 service password-encryption
 no logging cons
 no ip domain lookup
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int g1
  ip add 208.8.8.11 255.255.255.0
  no shut
 int g2
  ip add 192.168.102.11 255.255.255.0
  no shut
 int g3
  ip add 10.11.11.113 255.255.255.224
  no shut
 !
 username admin privilege 15 secret pass
 ip http server
 ip http secure-server
 ip http authentication local
 ip route 0.0.0.0 0.0.0.0 208.8.8.2
 end
wr
!
~~~


<br>


__NetOps-PH Setup__
> Login: root
> Pass: C1sc0123

<br>

1. Get the MAC Address for the Bridge connection  
VMWare > NetOps-PH Settings > NetAdapter (2, 3, & 4) > Advance > MAC Address  

| NetAdapter   | MAC Address      | VM Interface | ENS     |
| ---          | ---              | ---          | ---     |
| NetAdapter 2 | ___.___.___.___  | ens___       |  ens192 |
| NetAdapter 3 | ___.___.___.___  | ens___       |  ens224 |
| NetAdapter 4 | ___.___.___.___  | ens___       |  ens256 |

<br>

2. Get Network-VM Mapping
~~~
!@NetOps-PH
ip -br link
~~~

<br>

3. Modify Interface IP  
VMNet2:  192.168.102.6/24  
VMNet3:  10.11.11.100/27  
Bridged: 10.21.1.6/24  

<br>

~~~
!@NetOps-PH
ifconfig ens192 192.168.102.6 netmask 255.255.255.0 up
ifconfig ens224 10.11.11.100 netmask 255.255.255.224 up
ifconfig ens256 10.21.1.6 netmask 255.255.255.0 up
~~~

<br>

Verify:
~~~
!@NetOps-PH
ip -4 addr

nmcli connection show
netstat -rn
~~~

<br>

or  

<br>

__Using Network Management CLI for persistent IP.__

<br>

VMNet2:
~~~
!@NetOps-PH
nmcli connection add \
type ethernet \
con-name VMNET2 \
ifname ens192 \
ipv4.method manual \
ipv4.addresses 192.168.102.6/24 \
autoconnect yes

nmcli connection up VMNET2

nmcli connection add \
type ethernet \
con-name VMNET3 \
ifname ens224 \
ipv4.method manual \
ipv4.addresses 10.11.11.100/27 \
autoconnect yes

nmcli connection up VMNET3

nmcli connection add \
type ethernet \
con-name BRIDGED \
ifname ens256 \
ipv4.method manual \
ipv4.addresses 10.21.1.6/24 \
autoconnect yes

nmcli connection up BRIDGED

ip route add 10.0.0.0/8 via 10.21.1.4 dev ens256
ip route add 200.0.0.0/24 via 10.21.1.4 dev ens256
ip route add 0.0.0.0/0 via 10.11.11.113 dev ens224
~~~

&nbsp;
---
&nbsp;

### Remote Access
Connect to Management Interfaces of Devices  
NetOps-PH: 192.168.102.6  
UTM-PH: 192.168.102.11  

&nbsp;
---
&nbsp;

### Step 7 - Exchange SSH Keys
Delete existing SSH Keys  
~~~
!@NetOps
rm -rf /root/.ssh/known_hosts
~~~

<br>

SSH to the ff devices:
~~~
!@NetOps
ssh admin@10.21.1.2
~~~

<br>

| IP                 | Device   |
| ---                | ---      |
| 10.21.1.2      | CoreTaas |
| 10.21.1.4      | CoreBaba |
| 10.21.100.8    | CUCM     |
| 10.21.21.1 | EDGE     |

<br>

- Accept the keys  
- End the SSH session  

&nbsp;
---
&nbsp;

### Step 8 - Download the Configs  
~~~
!@NetOps
git clone https://github.com/4rthurcyber08/SSHAUTOMATE
cd SSHAUTOMATE/_Ansible/Ex\ 02\ -\ Day1/__day1_project/
~~~

&nbsp;
---
&nbsp;

### Step 9 - Specify the MAC address of End devices
Camera MAC Addresses:  
~~~
!@NetOps
nano host_vars/cbaba_21.yml
~~~

<br>

EPhone MAC Addresses:
~~~
!@NetOps
nano host_vars/cucm_21.yml
~~~

&nbsp;
---
&nbsp;

### Step 10 - Run the Playbook
~~~
!@NetOps
ansible-playbook -i rivan_mkt.ini playbooks/deploy_21.yml --skip-tags ivrs
~~~


<br>
<br>

---
&nbsp;


### ðŸŽ¯ Exercise 01: Add Loopback via Ansible
__Public Key Authentication__
~~~
!@NetOps
mkdir /etc/ansible/keys
ssh-keygen -t rsa -b 2048 -f /etc/ansible/keys/adminph.key
~~~


<br>


__Output at a minimum 46 chars__
~~~
!@NetOps
fold -w 46 /etc/ansible/keys/adminph.key.pub
~~~


<br>


__Decrypt the Key__
~~~
!@NetOps
ssh-keygen -p -f /etc/ansible/keys/adminph.key
~~~


<br>


__Make Cisco use Pubkey__
~~~
!@UTM-PH
conf t
 ip ssh pubkey-chain
  user admin
   key-string
 
 <Paste PUBLIC KEY>
~~~


<br>


__Verify Hosts__
~~~
!@NetOps
ssh -i /etc/ansible/keys/adminph.key admin@192.168.102.11
~~~


<br>


Install if needed
~~~
!@NetOps
pip install ansible-pylibssh
~~~


<br>


__Hosts File__
~~~
!@NetOps
cd /etc/ansible ; nano hosts
~~~


<br>


~~~
[utmph]
192.168.102.11

[utmph:vars]
ansible_user=admin
ansible_ssh_private_key_file=/etc/ansible/keys/adminph.key
ansible_port=22
ansible_network_os=ios
ansible_connection=network_cli
# ansible_become=yes
# ansible_become_method=enable
# ansible_become_password=pass
~~~


<br>


__Playbook (addloop.yml)__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes
  tasks:
    - name: "Create Loopbacks"
      ios_command:
        commands:
          - conf t
          - int lo100
          - ip add 100.100.100.100 255.255.255.255
          - exit
          - int lo101
          - ip add 101.101.101.101 255.255.255.255
          - exit
          - int lo102
          - ip add 102.102.102.102 255.255.255.255
      vars:
        ansible_network_os: ios
~~~


<br>


__or__


<br>


__Using ios_config__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes

  tasks:
    - name: Create Loopbacks
      cisco.ios.ios_config:
        lines:
          - ip address 100.100.100.100 255.255.255.255
        parents: interface Loopback100

    - name: Create Loopback101
      cisco.ios.ios_config:
        lines:
          - ip address 101.101.101.101 255.255.255.255
        parents: interface Loopback101

    - name: Create Loopback102
      cisco.ios.ios_config:
        lines:
          - ip address 102.102.102.102 255.255.255.255
        parents: interface Loopback102
~~~


<br>


__or__


<br>


__Using Variables (Best Practice)__
~~~
---
- name: addloop
  hosts: utmph
  gather_facts: yes
  become: yes

  tasks:
    - name: Create loopbacks
      cisco.ios.ios_config:
        parents: "interface Loopback{{ item.id }}"
        lines:
          - ip address {{ item.ip }} 255.255.255.255
      loop:
        - { id: 100, ip: 100.100.100.100 }
        - { id: 101, ip: 101.101.101.101 }
        - { id: 102, ip: 102.102.102.102 }
~~~


<br>


__Run the Playbook__
~~~
!@NetOps
ansible-playbook -i hosts addloop.yml 
~~~


<br>
<br>

---
&nbsp;

## Terraform
~~~
!@Cisco
conf t
 username admin privilege 15 secret pass
 ip http authentication local
 ip http secure-server
 netconf-yang
 restconf
 line vty 0 14
  password pass
  login local
  transport input all
  exec-timeout 0 0
  end
~~~


<br>


~~~
!@NetOps
mkdir /etc/terraform ; cd /etc/terraform ; nano addloop.tf
~~~


<br>


__addloop.tf__
~~~
terraform {
  required_providers {
    iosxe = {
      source = "CiscoDevNet/iosxe"
    }
  }
}

provider "iosxe" {
  username = "admin"
  password = "pass"
  host     = "192.168.102.11"
}

resource "iosxe_interface_loopback" "example" {
  name               = 22
  description        = "Configured via Terraform"
  shutdown           = false
  ipv4_address       = "22.22.22.22"
  ipv4_address_mask  = "255.255.255.255"

}
~~~


<br>
<br>

---
&nbsp;

## SDN (Software Defined Networking)
*https://account.meraki.com/login/*

<br>

__Control Plane (vSmart)__ - The part of the device that makes routing and network decisions. builds and maintains the network topology and make decisions on the traffic flows. The vSmart controller disseminates control plane information between WAN Edge devices, implements control plane policies and distributes data plane policies to network devices for enforcement.

~~~
!@Cisco
show processes cpu sorted
show ip route
show ip ospf database
show ip bgp
show ip eigrp topology
show spanning-tree
show policy-map control-plane
~~~


<br>
<br>


__Data Plane (vEdge)__ - The part that forwards packets according to rules calculated by the control plane. WAN Edge devices are responsible for establishing secure connections for traffic forwarding, for security, encryption, Quality of Service (QoS) enforcement and more.

~~~
!@Cisco
show ip nat translations
show interfaces counters protocol status 
show access-lists
show policy-map interface
ping / traceroute
~~~


<br>
<br>


__Management Plane (vManage)__ - the instance where the user interacts with the device. It is responsible for central configuration and monitoring. The vManage controller is the centralized network management system that provides a single pane of glass GUI interface to easily deploy, configure, monitor and troubleshoot all Cisco SD-WAN components in the network.

- SSH/Telnet Access
- SNMP
- Syslog, NetFlow
- NTP, Logging


<br>
<br>


__Orchestration Plane (vBond)__ - Policy distribution and Overlay control. Centralized Control. Assists in securely onboarding the SD-WAN WAN Edge routers into the SD-WAN overlay. The vBond controller, or orchestrator, authenticates and authorizes the SD-WAN components onto the network.
- Control Node
- SDN Controller


&nbsp;
---
&nbsp;


### SDN Elements
~~~
User
 V

Northbound Int/API
 V - RESTCONF, NETCONF, ONOS (Open Network Operating System)

SDN Controller - Cisco APIC (Application Policy Infra Controller) or Cisco DNA (Digital Network Architecture) Center
 V

Southbound Int/API
 V - RESTCONF,  NETCONF, OpenFlow, OVSDB (Open vSwitch DB), gNMI (gRPC Network Management Interface)

Network Devices
~~~


<br>
<br>

---
&nbsp;

## SD-WAN
*https://www.cisco.com/c/dam/en/us/td/docs/solutions/CVD/SDWAN/sdwan-wan-edge-onboarding-deploy-guide-2020nov.pdf*  

![SDWAN](img/SDWAN.JPG)

<br>
<br>


Each component authenticates each other and if successful, a Datagram Transport Layer Security (DTLS) tunnel is established.  

Separating the control plane from the data and management plane,  
the protocol that vSmart uses to communicate all the WAN Edge routing is called Overlay Management Protocol (OMP)  


<br>
<br>

---
&nbsp;

### Resource Requirements
__EVENG VM__
- 32GB RAM 
- 12 Core 
- Network Adapter 1 : NAT (208.8.8.0/24)
- Network Adapter 2 : VMNet15 (10.255.10.0/24)
- Network Adapter 3 : VMNet16 (10.69.255.0/29)


&nbsp;
---
&nbsp;


### SDWAN Procedure

__SDWAN Appliances Login__
> Login: admin
> Pass: C1sc0123

<br>

| Devices        | Ports |
| ---            | ---   |
| CLOUD          | 32902 |
| PKI SERVER     | 32913 |
|                |       |
| vManage        | 32897 |
| vSmart         | 32898 |
| vBond          | 32899 |
|                |       |
| vEdge-LUZON    | 32900 |
| vEdge-VISAYAS  | 32903 |
| vEdge-MINDANAO | 32904 |
|                |       |
| CSW-LUZON      | 32905 |
| CSW-VISAYAS    | 32906 |
| CSW-MINDANAO   | 32907 |

<br>

### STEP 1 - Set a host route for vManage GUI
~~~
!@cmd 
route add 10.69.255.13 mask 255.255.255.255 10.69.255.6
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Bootstrap Configuration
| Device  | Port  |
| ---     | ---   |
| vManage | 32897 |
| CLOUD   | 32902 |

~~~
!@vManage
conf t
 system
  host-name Rivan-vManage
  site-id 10
  system-ip 10.1.10.2
  organization-name RIVANCORP
  vbond 172.16.10.3
  admin-tech-on-failure
 vpn 0
  int eth0
   ip add 172.16.10.2/29
   no shut
   tunnel-interface
    allow-service all
  ip route 0.0.0.0/0 172.16.10.6
 vpn 512
  int eth1
   ip add 10.69.255.13/30
   no shut
  ip route 0.0.0.0/0 10.69.255.14
  commit
  end
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Access vManage GUI
> [!NOTE]
> It usually takes 10 MINUTES before vManage GUI is accessible

<br>

URL: https://10.69.255.13:8443  
- User: admin  
- C1sc0123  

<br>

Verification
~~~
!@vManage
request nms all status
show system status
~~~


&nbsp;
---
&nbsp;


### STEP 4 - vBond & vSmart
| Device  | Port  |
| ---     | ---   |
| vBond   | 32899 |
| vSmart  | 32898 |

<br>

~~~
!@vBond
conf t
 system
  host-name Rivan-vBond
  site-id 10
  system-ip 10.1.10.3
  organization-name RIVANCORP
  vbond 172.16.10.3 local vbond
  admin-tech-on-failure
 vpn 0
  int ge0/0
   ip add 172.16.10.3/29
   no shut
   tunnel-interface
    encapsulation ipsec
    allow-service all
  ip route 0.0.0.0/0 172.16.10.6
  commit
  end
~~~

<br>

~~~
!@vSmart
conf t
 system
  host-name Rivan-vSmart
  site-id 10
  system-ip 10.1.10.1
  organization-name RIVANCORP
  vbond 172.16.10.3
  admin-tech-on-failure
  vpn 0
   int eth0
    ip add 172.16.10.1/29
	no shut
	tunnel-interface
	 allow-service all
   ip route 0.0.0.0/0 172.16.10.6
   commit
   end
~~~

<br>

> [!IMPORTANT]
> Wait until the SDWAN Controllers are stable before turning on the vEdges

<br>

| Device         | Port  |
| ---            | ---   |
| vEdge-LUZON    | 32900 |
| vEdge-VISAYAS  | 32903 |


&nbsp;
---
&nbsp;


### STEP 5 - Configure BGP on CLOUD to establish connection with controllers
~~~
!@Cloud (BGP Config)
conf t
 int lo8
  ip add 8.8.8.8 255.255.255.255
 router bgp 1
  bgp log-neighbor-changes
  neighbor 192.168.20.1 remote-as 100
  neighbor 192.168.20.5 remote-as 100
  neighbor 192.168.20.9 remote-as 100
  address-family ipv4
   neighbor 192.168.20.1 activate
   neighbor 192.168.20.5 activate
   neighbor 192.168.20.9 activate
   neighbor 192.168.20.1 as-override
   neighbor 192.168.20.5 as-override
   neighbor 192.168.20.9 as-override
   network 8.8.8.8 mask 255.255.255.255
   network 192.168.20.0 mask 255.255.255.0
   network 172.16.10.0 mask 255.255.255.248
   network 10.69.255.0 mask 255.255.255.248
   end
~~~

<br>

~~~
!@vEdge-LUZON
show bgp routes
show bgp neighbors
~~~


&nbsp;
---
&nbsp;


### STEP 6 - Return to the vManage GUI then send the vEDGE list to all controllers

`Configuration` > `Certificates` > `Send to Controllers`


&nbsp;
---
&nbsp;


### STEP 7 - Configure Templates for SYSTEM

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `System`

~~~
Name: VE-SYSTEM
Desc: VE-SYSTEM

Console Baud Rate(bps): 9600

GPS - Longitude & Latitude
~~~


&nbsp;
---
&nbsp;


### STEP 8 - Configure Templates for BANNER

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `Banner`

~~~
Name: VE-BANNER
Desc: VE-BANNER

Login Banner: Welcome to RIVANCORP
MOTD Banner: Property owned by RIVANCORP
~~~


&nbsp;
---
&nbsp;


### STEP 9 - Configure Templates for VPN0

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN0
Desc: VE-VPN0

VPN: 0
Name: TRANSPORT VPN

Ipv4 Route:
  Prefix: 0.0.0.0/0
  Gateway: NextHop
  Add Next Hop:
    Address: Device Specific
~~~


&nbsp;
---
&nbsp;


### STEP 10 - Configure Templates for VPN512

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN512
Desc: VE-VPN512

VPN: 512
Name: MANAGEMENT VPN
~~~


&nbsp;
---
&nbsp;


### STEP 11 - Configure Templates for Interface IP address (VPN512)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN512-ETH0
Desc: VE-VPNINT-VPN512-ETH0

Shutdown: No
Interface Name: eth0
Description: MANAGEMENT INTERFACE

IPv4Add: Default
~~~


&nbsp;
---
&nbsp;


### STEP 12 - Configure Templates for Interface IP address (VPN0-G0/1)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN0-GIG01
Desc: VE-VPNINT-VPN0-GIG01

Shutdown: No
Interface Name: ge0/1
Description: TRANSPORT INTERFACE

IPv4Add: Device Specific

Tunnel Interface: On
Color: BIZ-INTERNET

Allow Service: All, NETCONF, SSH, BGP
NAT: On
~~~


&nbsp;
---
&nbsp;


### STEP 13 - Configure Templates for Interface IP address (VPN0-G0/0)

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN Interface Ethernet`

~~~
Name: VE-VPNINT-VPN0-GIG00
Desc: VE-VPNINT-VPN0-GIG00

Shutdown: No
Interface Name: ge0/0
Description: LAN INTERFACE

IPv4Add: Device Specific

Tunnel Interface: On
Color: Private1
Restrict: On
Allow Service: All, NETCONF, SSH, OSPF
~~~


&nbsp;
---
&nbsp;


### STEP 14 - Configure Templates for BGP

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `BGP`

~~~
Name: VE-BGP-VPN0
Desc: VE-BGP-VPN0

Shutdown: No
AS Number: Global 100

Neigbor:
  Address: 192.168.20.6
  Remote AS: 1
  Address Family: On
    Address Family: IPv4 Unicast
	Shutdown: No
~~~


&nbsp;
---
&nbsp;


### STEP 15 - Configure Device Templates

~~~
Device Model: vEdge Cloud
Device Role: SDWAN Edge
Template Name: VE-TEMP
Desc: VE-TEMP

Basic Info:
  System: VE-SYSTEM
 
Transport & Management VPN:
  VPN0: VE-VPN0
  Add: 
    BGP: VE-BGP-VPN0
	VPN Interface: VE-VPNINT-VPN0-GIG00
	VPN Interface: VE-VPNINT-VPN0-GIG01
  
  VPN512: VE-VPN512
  VPN Interface: VE-VPNINT-VPN512-ETH1
~~~


&nbsp;
---
&nbsp;


### STEP 16 - Attatch Device Tempates to vEdge-LUZON & VISAYAS


&nbsp;
---
&nbsp;


### STEP 17 - Assign Correct Values Per device

PerDevice Edit:
  - VISAYAS
  - IPv4 (g0/1): 192.168.20.1/24
  - IPv4 (g0/0): 172.16.1.1/30
  - Hostname: vEDGE-VISAYAS
  - Latitude/Longitude: 3 deci
  - System IP: 10.1.25.1
  - Site ID: 25


&nbsp;
---
&nbsp;


### ðŸŽ¯ Exercise 02: Add Template for a user account

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `AAA`

~~~
Name: VE-ACC
Desc: VE-ACC

Auth Order: Local Only
Add User: admin C1sc0123
          rivan C1sc0123
~~~


<br>
<br>

---
&nbsp;

## Establish Connection between LUZON & VISAYAS
| Device      | Port  |
| ---         | ---   |
| CSW-LUZON   | 32905 |
| CSW-VISAYAS | 32906 |


~~~
!@CSW-LUZON
conf t
 hostname CSW-LUZON
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 1.1.1.1 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.1.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.1.2 255.255.255.252
  no shut
 router ospf 1
  router-id 1.1.1.1
  network 172.16.1.0 0.0.0.3 area 0
  network 1.1.1.1 0.0.0.0 area 0
  network 10.1.1.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~

<br>

~~~
!@CSW-VISAYAS
conf t
 hostname CSW-VISAYAS
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 2.2.2.2 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.5.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.2.2 255.255.255.252
  no shut
 router ospf 1
  router-id 2.2.2.2
  network 172.16.5.0 0.0.0.3 area 0
  network 2.2.2.2 0.0.0.0 area 0
  network 10.1.2.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~


&nbsp;
---
&nbsp;


### STEP 1 - Configure Templates for VPN1

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `VPN`

~~~
Name: VE-VPN1
Desc: VE-VPN1

VPN: 1
NAME: DATA VPN

IPv4 Route: 
  Prefix: 0.0.0.0/0
  Gateway: VPN
  Enable VPN: On
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Configure Templates VPN1 Interface [MODIFY]  - OPTIONAL

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > MODIFY `VE-VPNINT-VPN0GIG00`

~~~
Name: Retain
Desc: Retain

Shutdown: No
Name: ge0/0
Description: LAN INTERFACE

Ipv4 Address: Device Specific
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Configure Templates for OSPF 

`Configuration` > `Templates` > `Feature Templates` > `vEdge Cloud` > `OSPF`

~~~
Name: VE-OSPF-VPN1
Desc: VE-OSPF-VPN1

Redistribute:
  Protocol: omp

Area:
  Area Num: 0
  Interface:
    Interface Name: ge0/0
	
	ADD x2

Advance:
  Originate: On
  Always: On
~~~


&nbsp;
---
&nbsp;


### STEP 4 - Update VE-TEMP (Device Template)

~~~
Service VPN:
  Add VPN: VE-VPN1
    OSPF: VE-OSPF-VPN1
	VPN Interface: VE-VPNINT-VPN0-GIG00

Remove GIG00 From VPN 0

Assign G0/0 With the IP based on the topology
~~~


<br>
<br>

---
&nbsp;

## vEDGE Onboarding
| Device       | Port  |
| ---          | ---   |
| CSW-MINDANAO | 32907 |
| PKI-Server   | 32913 |

~~~
!@CSW-MINDANAO
conf t
 hostname CSW-MINDANAO
 enable secret pass
 service password-encryption
 no logging console
 no ip domain lookup
 username admin priv 15 secret pass
 line vty 0 14
  transport input all
  password pass
  login local
  exec-timeout 0 0
 int lo0
  ip add 3.3.3.3 255.255.255.255
  exit
 int g0/0
  no sw
  ip add 172.16.9.2 255.255.255.252
  no shut
 int g0/1
  no sw
  ip add 10.1.3.2 255.255.255.252
  no shut
 router ospf 1
  router-id 3.3.3.3
  network 172.16.9.0 0.0.0.3 area 0
  network 3.3.3.3 0.0.0.0 area 0
  network 10.1.3.0 0.0.0.3 area 0
  passive-interface lo0
  end
~~~

<br>

~~~
!@vEdge-MINDANAO (Optional)
conf t
 system
  host-name vEdge-MINDANAO
  system-ip 10.1.29.1
  site-id 29
  organization-name RIVANCORP
  admin-tech-on-failure
  vbond 172.16.10.3
 vpn 0
  name "Transport VPN"
   router
    no bgp 100
    bgp 100
	 address-family ipv4-unicast
	  network 192.168.20.0/24
	  exit
	 neighbor 192.168.20.6 remote-as 1
	 exit
	exit
   exit
  int ge0/0
   description LAN-Traffic
   ip address 172.16.9.1/30
   tunnel-interface
    encapsulation ipsec
	color private1 restrict
    allow-service all
    no shut
	exit
  int ge0/1
   description INTERNET-Traffic
   ip add 192.168.20.9/24
   tunnel-interface
    encapsulation ipsec
    color biz-internet
    allow-service all
	no shut
	exit
   exit
  int ge0/2
   no description
   no ip add
   shutdown
   no tunnel-interface
   exit
  vpn 512
   name "Management VPN"
   interface eth0
    description eth0
    ip dhcp-client
    no shutdown
	commit
	end
~~~


<br>


### STEP 1 - Export Root CA
~~~
!@PKI-Server
conf t
 crypto pki export rivanpki pem terminal


~~~


&nbsp;
---
&nbsp;


### STEP 2 - Setup Chain-of-trust on vEDGE
~~~
!@vEdge-MINDANAO
vshell

mkdir /home/admin/pkicerts
cd /home/admin/pkicerts
vim rivanpki.ca

     HOW TO USE VIM
		-> press ENTER
		-> press "i" for insert mode
		-> then, paste the root ca
		-> press "esc" button
		-> enter ":wq" which means write quit"
~~~


&nbsp;
---
&nbsp;


### STEP 3 - Generate Certificate Signing Requests
~~~
!@vEdge-MINDANAO
request root-cert-chain install /home/admin/pkicerts/rivanpki.ca
request csr upload /home/admin/pkicerts/mindanao.csr

vshell

cat /home/admin/pkicerts/mindanao.csr

~~~


&nbsp;
---
&nbsp;


### STEP 4 - Sign the CSR 
~~~
!@PKI-Server
crypto pki server rivanpki request pkcs10 terminal

~~~


&nbsp;
---
&nbsp;


### STEP 5 - Import Signed CSR
~~~
!@vEdge-MINDANAO
vshell

vim /home/admin/pkicerts/grant.ca

     HOW TO USE VIM
		-> press ENTER
		-> press "i" for insert mode
		-> then, paste the root ca
		-> press "esc" button
		-> enter ":wq" which means write quit"
~~~


&nbsp;
---
&nbsp;


### STEP 6 - Install Certificates
~~~
!@vEdge-MINDANAO
request certificate install /home/admin/pkicerts/grant.ca
show certificate serial
~~~


&nbsp;
---
&nbsp;


### STEP 7 - Onboard to vManage & vBond
~~~
!@vManage, vBond
request vedge add chassis-num xxxxxx-xxxx-xxxx-xxxx-35b26f5d569a serial-num xx
~~~


<br>
<br>

---
&nbsp;


### ðŸŽ¯ Exercise 03: Setup Trunking, Etherchannel, & MST on CoreTAAS & CoreBABA
~~~
!@CoreTAAS
conf t
 hostname coreTaas-21
 enable secret pass
 service password-encryption
 no logging console
 no ip domain-lookup
 line cons 0
  password pass
  login
  exec-timeout 0 0
 line vty 0 14
  password pass
  login
  exec-timeout 0 0
 int vlan 1
  no shut
  ip add 10.21.1.2 255.255.255.0
  desc DEFAULT-VLAN
 int vlan 10
  no shut
  ip add 10.21.10.2 255.255.255.0
  desc WIFI-VLAN
 int vlan 50
  no shut
  ip add 10.21.50.2 255.255.255.0
  desc CCTV-VLAN
 int vlan 100
  no shut
  ip add 10.21.100.2 255.255.255.0
  desc VOICE-VLAN
 end
~~~


<br>


~~~
!@CoreBABA
conf t
 hostname coreBaba-21
 enable secret pass
 service password-encryption
 no logging console
 no ip domain-lookup
 line cons 0
  password pass
  login
  exec-timeout 0 0
 line vty 0 14
  password pass
  login
  exec-timeout 0 0
 int gi 0/1
  no shut
  no switchport
  ip add 10.21.21.4 255.255.255.0
 int vlan 1
  no shut
  ip add 10.21.1.4 255.255.255.0
  desc DEFAULT-VLAN
 int vlan 10
  no shut
  ip add 10.21.10.4 255.255.255.0
  desc WIFI-VLAN
 int vlan 50
  no shut
  ip add 10.21.50.4 255.255.255.0
  desc CCTV-VLAN
 int vlan 100
  no shut
  ip add 10.21.100.4 255.255.255.0
  desc VOICE-VLAN
 end
~~~


<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>


&nbsp;
---
&nbsp;

### ANSWER
<details>
<summary>Show Answer</summary>

~~~
!@CoreTAAS, CoreBABA
conf t
 int range fa0/10-12
  channel-group 1 mode active
  channel-protocol lacp
  exit
 int po1
  switchport trunk encaps dot1q
  switchport mode trunk
  switchport trunk allowed vlan all
  switchport trunk native vlan 1
  exit
 !
 vtp domain ccnp
 vtp password pass
 vtp mode server
 vtp version 2
 !
 vlan 10
  name WIFIVLAN
 vlan 50
  name CCTVVLAN
 vlan 100
  name VOICEVLAN
  exit
 !
 spanning-tree mode mst
 spanning-tree mst configuration
  name SUPERMAN-STP
  revision 1
   instance 1 vlan 1,10
   instance 2 vlan 50,100
   end
show int trunk
~~~

<br>

~~~
!@CoreTAAS
conf t
 spanning-tree mst 0 root primary
 spanning-tree mst 1 root secondary
 spanning-tree mst 2 root primary
 end
~~~

<br>

~~~
!@CoreBABA
conf t
 spanning-tree mst 0 root secondary
 spanning-tree mst 1 root primary
 spanning-tree mst 2 root secondary
 end
~~~

<br>

__STP Features__
~~~
!@CoreBABA
conf t
 int range fa0/1-9
  spanning-tree portfast
  spanning-tree bpduguard enable
  exit
 spanning-tree uplinkfast
 end
~~~

<br>

~~~
!@CoreTAAS
conf t
 spanning-tree backbonefast
~~~

</details>


<br>
<br>

---
&nbsp;


### Interactive Voice Response System
~~~
!@CUCM
config t
dial-peer voice 69 voip
 service rivanaa out-bound
 destination-pattern 2169
 session target ipv4:10.21.100.1
 incoming called-number 2169
 dtmf-relay h245-alphanumeric
 codec g711ulaw
 no vad
!
telephony-service
 moh "flash:/en_bacd_music_on_hold.au"
!
application
 service rivanaa flash:app-b-acd-aa-3.0.0.2.tcl
  paramspace english index 1        
  param number-of-hunt-grps 2
  param dial-by-extension-option 8
  param handoff-string rivanaa
  param welcome-prompt flash:en_bacd_welcome.au
  paramspace english language en
  param call-retry-timer 15
  param service-name rivanqueue
  paramspace english location flash:
  param second-greeting-time 60
  param max-time-vm-retry 2
  param voice-mail 1234
  param max-time-call-retry 700
  param aa-pilot 2169
 service rivanqueue flash:app-b-acd-3.0.0.2.tcl
  param queue-len 15
  param aa-hunt1 2100
  param aa-hunt2 2177
  param aa-hunt3 2101
  param aa-hunt4 2133
  param queue-manager-debugs 1
  param number-of-hunt-grps 4
  end
~~~


<br>
<br>

---
&nbsp;


## Private & Public WAN
~~~
!@EDGE
conf t
 no router ospf 1
 router ospf 1
  router-id 21.0.0.1
  network 10.21.21.0 0.0.0.255 area 0
  default-information originate always
  exit
 ip domain lookup
 ip name-server 8.8.8.8
 ip domain lookup source-int g0/0/0
 ip route 0.0.0.0 0.0.0.0 200.0.0.1
 end
~~~

<br>

~~~
!@EDGE
conf t
 int g0/0/0 
  ip nat inside
  exit
 int g0/0/1
  ip nat outside
  exit
 !
 ip access-list extended NAT-POLICY
  permit ip any any
  exit
 !
 ip nat inside source list NAT-POLICY int g0/0/1 overload
 end
~~~


&nbsp;
---
&nbsp;


### Multi-Point GRE Tunnel
~~~
!@EDGE
conf t
 int tun1
  ip add 172.16.1.21 255.255.255.0
  ! no ip redirects
  tunnel source g0/0/1
  tunnel mode gre multipoint
  no shut
  tun key 123
  ip nhrp authentication C1sc0123
  ip nhrp map multicast dynamic             !hubs
! ip nhrp redirect                          !hubs
! ip nhrp shortcut                          !spoke
! ip nhrp nhs <hub tunnel ip>               !spoke
! ip nhrp map multicast <hub pubip>         !spoke
! ip nhrp map <hub tunnel ip>  <hub pubip>  !spoke
  ip nhrp network-id 1337
  ip nhrp map 172.16.1.11 200.0.0.11
  ip nhrp map 172.16.1.12 200.0.0.12
  ip nhrp map 172.16.1.21 200.0.0.21
  ip nhrp map 172.16.1.22 200.0.0.22
  ip nhrp map 172.16.1.31 200.0.0.31
  ip nhrp map 172.16.1.32 200.0.0.32
  ip nhrp map 172.16.1.41 200.0.0.41
  ip nhrp map 172.16.1.42 200.0.0.42
  ip nhrp map 172.16.1.51 200.0.0.51
  ip nhrp map 172.16.1.52 200.0.0.52
  ip nhrp map 172.16.1.61 200.0.0.61
  ip nhrp map 172.16.1.62 200.0.0.62
  ip nhrp map 172.16.1.71 200.0.0.71
  ip nhrp map 172.16.1.72 200.0.0.72
  ip nhrp map 172.16.1.81 200.0.0.81
  ip nhrp map 172.16.1.82 200.0.0.82
  ip nhrp map 172.16.1.91 200.0.0.91
  ip nhrp map 172.16.1.92 200.0.0.92
  no ip nhrp map 172.16.1.21 200.0.0.21
  exit
 !
 ip route 10.11.0.0 255.255.0.0 172.16.1.11 252
 ip route 10.12.0.0 255.255.0.0 172.16.1.12 252
 ip route 10.21.0.0 255.255.0.0 172.16.1.21 252
 ip route 10.22.0.0 255.255.0.0 172.16.1.22 252
 ip route 10.31.0.0 255.255.0.0 172.16.1.31 252
 ip route 10.32.0.0 255.255.0.0 172.16.1.32 252
 ip route 10.41.0.0 255.255.0.0 172.16.1.41 252
 ip route 10.42.0.0 255.255.0.0 172.16.1.42 252
 ip route 10.51.0.0 255.255.0.0 172.16.1.51 252
 ip route 10.52.0.0 255.255.0.0 172.16.1.52 252
 ip route 10.61.0.0 255.255.0.0 172.16.1.61 252
 ip route 10.62.0.0 255.255.0.0 172.16.1.62 252
 ip route 10.71.0.0 255.255.0.0 172.16.1.71 252
 ip route 10.72.0.0 255.255.0.0 172.16.1.72 252
 ip route 10.81.0.0 255.255.0.0 172.16.1.81 252
 ip route 10.82.0.0 255.255.0.0 172.16.1.82 252
 ip route 10.91.0.0 255.255.0.0 172.16.1.91 252
 ip route 10.92.0.0 255.255.0.0 172.16.1.92 252
 !
 no ip route 10.21.0.0 255.255.0.0 172.16.1.21 252
~~~

<br>

__Modify NAT Policy__
~~~
conf t
 no ip access-list extended NAT-POLICY
 ip access-list extended NAT-POLICY
  deny ip 10.21.0.0 0.0.255.255 10.11.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.12.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.21.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.22.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.31.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.32.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.41.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.42.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.51.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.52.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.61.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.62.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.71.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.72.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.81.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.82.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.91.0.0 0.0.255.255
  deny ip 10.21.0.0 0.0.255.255 10.92.0.0 0.0.255.255
  no deny ip 10.21.0.0 0.0.255.255 10.21.0.0 0.0.255.255
  permit ip any any
  end
~~~


<br>
<br>

---
&nbsp;


# Dynamic Multipoint VPN

![DMVPN](img/DMVPN.png)


&nbsp;
---
&nbsp;


## Setup
`VMWare` > `Edit` > `Virtual Network Editor`  

<br>

Add/Edit the following VMNets:  

| VMNet 2    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.102.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 3    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.103.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 4    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.104.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 5    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.105.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 6    |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 192.168.106.0 |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 

<br>

| VMNet 8    |               |
| ---        | ---           |
| VMNet Info | NAT           |
| IP address | 208.8.8.0     |
| Net Mask   | 255.255.255.0 |
| DHCP       | Unchecked     | 	

<br>

| VMNet 15   |               |
| ---        | ---           |
| VMNet Info | Host-only     |
| IP address | 10.255.10.0   |
| Net Mask   | 255.255.255.0 |
| DHCP       | Checked       | 

<br>

| VMNet 16   |                 |
| ---        | ---             |
| VMNet Info | Host-only       |
| IP address | 10.69.255.0     |
| Net Mask   | 255.255.255.248 |
| DHCP       | Checked         | 


&nbsp;
---
&nbsp;


## PRECONFIGS
- Windows Server
- VPNSec Lab

~~~
!@WinVM-cmd
route add 172.16.29.0 mask 255.255.255.0 10.69.255.6
~~~


&nbsp;
---
&nbsp;


### BGP

| ISP | ASN   |
| --- | ---   |
| I1  | 1     |
| I2  | 2     |
| I3  | 3     |
| Ex  | 12345 |


~~~
!@I1
conf t
 no router bgp 1
 router bgp 1
  bgp log-neighbor-changes
  neighbor 12.1.2.2 remote-as 2
  neighbor 13.1.3.3 remote-as 3
  neighbor 1.10.1.10 remote-as 12345
  neighbor 3.30.3.30 remote-as 12345
  address-family ipv4
   neighbor 12.1.2.2 activate
   neighbor 13.1.3.3 activate
   neighbor 1.10.1.10 activate
   neighbor 3.30.3.30 activate
   neighbor 1.10.1.10 as-override
   neighbor 3.30.3.30 as-override
   network 12.1.2.0 mask 255.255.255.0
   network 13.1.3.0 mask 255.255.255.0
   network 1.10.1.0 mask 255.255.255.0
   network 3.30.3.0 mask 255.255.255.0
   network 8.8.8.8 mask 255.255.255.255
   end
~~~

<br>

~~~
!@I2
conf t
 no router bgp 2
 router bgp 2
  bgp log-neighbor-changes
  neighbor 12.1.2.1 remote-as 1
  neighbor 2.20.2.20 remote-as 12345
  neighbor 5.50.5.50 remote-as 12345
  address-family ipv4
   neighbor 12.1.2.1 activate
   neighbor 2.20.2.20 activate
   neighbor 2.20.2.20 as-override
   neighbor 5.50.5.50 activate
   neighbor 5.50.5.50 as-override
   network 12.1.2.0 mask 255.255.255.0
   network 2.20.2.0 mask 255.255.255.0
   network 5.50.5.0 mask 255.255.255.0
   end   
~~~

<br>

~~~
!@I3
conf t
 no router bgp 3
 router bgp 3
  bgp log-neighbor-changes
  neighbor 13.1.3.1 remote-as 1
  neighbor 4.40.4.40 remote-as 12345
  address-family ipv4
   neighbor 13.1.3.1 activate
   neighbor 4.40.4.40 activate
   neighbor 4.40.4.40 as-override
   network 13.1.3.0 mask 255.255.255.0 
   network 4.40.4.0 mask 255.255.255.0
   end
~~~

<br>

~~~
!@E1
conf t
 int lo0
  ip add 1.1.1.1 255.255.255.255
 no router bgp 12345
 router bgp 12345
  bgp log-neighbor-changes
  neighbor 1.10.1.1 remote-as 1
  address-family ipv4
   neighbor 1.10.1.1 activate
   network 1.10.1.0 mask 255.255.255.0
   network 1.1.1.1 mask 255.255.255.255
 !
 ip access-list extended NAT
  permit ip 10.1.1.0 0.0.0.3 any
 !
 int e0/1
  ip nat outside
 int e0/0
  ip nat inside
 !
 ip nat inside source list NAT int e0/1 overload
 end
~~~

<br>

~~~
!@E2
conf t
 no router bgp 12345
 router bgp 12345
  bgp log-neighbor-changes
  neighbor 2.20.2.2 remote-as 2
  address-family ipv4
   neighbor 2.20.2.2 activate
   network 2.20.2.0 mask 255.255.255.0
 !
 ip access-list extended NAT
  permit ip 10.2.2.0 0.0.0.3 any
 !
 int e0/1
  ip nat outside
 int e0/0
  ip nat inside
 !
 ip nat inside source list NAT int e0/1 overload
 end
~~~

<br>

~~~
!@E3
conf t
 no router bgp 12345
 router bgp 12345
  bgp log-neighbor-changes
  neighbor 3.30.3.1 remote-as 1
  address-family ipv4
   neighbor 3.30.3.1 activate
   network 3.30.3.0 mask 255.255.255.0
 !
 ip access-list extended NAT
  permit ip 192.168.3.0 0.0.0.255 any
 !
 int e0/1
  ip nat outside
 int e0/0
  ip nat inside
 !
 ip nat inside source list NAT int e0/1 overload
 end
~~~

<br>

~~~
!@E4
conf t
 no router bgp 12345
 router bgp 12345
  bgp log-neighbor-changes
  neighbor 4.40.4.3 remote-as 3
  address-family ipv4
   neighbor 4.40.4.3 activate
   network 4.40.4.0 mask 255.255.255.0
 !
 ip access-list extended NAT
  permit ip 192.168.4.0 0.0.0.255 any
 !
 int e0/1
  ip nat outside
 int e0/0
  ip nat inside
 !
 ip nat inside source list NAT int e0/1 overload
 end
~~~

<br>

~~~
!@E5
conf t
 no router bgp 12345
 router bgp 12345
  bgp log-neighbor-changes
  neighbor 5.50.5.2 remote-as 2
  address-family ipv4
   neighbor 5.50.5.2 activate
   network 5.50.5.0 mask 255.255.255.0
 !
 ip access-list extended NAT
  permit ip 192.168.5.0 0.0.0.255 any
 !
 int e0/1
  ip nat outside
 int e0/0
  ip nat inside
 !
 ip nat inside source list NAT int e0/1 overload
 end
~~~


&nbsp;
---
&nbsp;

__Phase 1 (IKEv2) & Phase 2 (IPSec)__
- Encryption
- Integrity
- DH

<br>

__Tunnel Properties__
- IP
- Source Interface
- Destination Peer IP
- Remote Subnets


&nbsp;
---
&nbsp;


### PHASE 1 - IKEv2

1. PHASE 1 PROPOSAL
2. PHASE 1 POLICY
3. PHASE 1 KEYRING
4. PHASE 1 PROFILE

__HUB__
~~~
!@E1,E2
conf t
 crypto ikev2 proposal PROP1
  encr aes-cbc-256
  integrity sha256
  group 14
 crypto ikev2 proposal PROP2
  encr aes-gcm-256
  prf sha256
  group 14
 !
 crypto ikev2 policy IKEV2-POL1
  proposal PROP1
  proposal PROP2
 !
 crypto ikev2 keyring KEY1
  peer DMVPN-EDGES
   address 0.0.0.0  0.0.0.0
   pre-shared-key C1sc0123
 !
 crypto ikev2 profile IKEV2-PROF1
  match identity remote address 0.0.0.0  0.0.0.0
  authentication remote pre-share
  authentication local pre-share
  keyring local KEY1
  lifetime 86400
  end
~~~

<br>

__SPOKES__
~~~
!@E3,E4,E5
conf t
 crypto ikev2 proposal PROP1
  encr aes-cbc-256
  integrity sha256
  group 14
 crypto ikev2 proposal PROP2
  encr aes-gcm-256
  prf sha256
  group 14
 !
 crypto ikev2 policy IKEV2-POL1
  proposal PROP1
  proposal PROP2
 !
 crypto ikev2 keyring KEY1
  peer HUB1
   address 1.10.1.10 255.255.255.255
   pre-shared-key C1sc0123
  peer HUB2
   address 2.20.2.20 255.255.255.255
   pre-shared-key C1sc0123
 !
 crypto ikev2 profile IKEV2-PROF1
  match identity remote address 1.10.1.10
  match identity remote address 2.20.2.20
  authentication remote pre-share
  authentication local pre-share
  keyring local KEY1
  lifetime 86400
  end
~~~


&nbsp;
---
&nbsp;


### PHASE 2 - IPSec

1. PHASE 2 TRANSFORM SET
2. PHASE 2 PROFILE

~~~
!@E1,E2,E3,E4,E5
conf t
 crypto ipsec transform-set TS1 esp-aes 256 esp-sha256-hmac
  mode tunnel
 crypto ipsec transform-set TS2 esp-gcm 256
  mode tunnel
 !
 crypto ipsec profile IPSEC-PROF1
  set transform-set TS1 TS2
  set ikev2-profile IKEV2-PROF1
  set pfs group14
  end
~~~

<br>

~~~
!@E1
conf t
 int tunnel0
  ip add 172.16.1.254 255.255.255.0
  tunnel mode gre multipoint
  tunnel source e0/1
  tunnel protection ipsec profile IPSEC-PROF1
  ip nhrp network-id 12345
  ip nhrp authentication C1sc0123
  ip nhrp map multicast dynamic
  ip nhrp redirect
 !
 no ip access-list extended NAT
 ip access-list extended NAT
  deny ip host 10.1.1.1 host 10.2.2.1
  deny ip host 10.1.1.1 192.168.3.0 0.0.0.255
  deny ip host 10.1.1.1 192.168.4.0 0.0.0.255
  deny ip host 10.1.1.1 192.168.5.0 0.0.0.255
  permit ip 10.1.1.0 0.0.0.3 any
  end
~~~

<br>

~~~
!@E2
conf t
 int tunnel0
  ip add 172.16.1.253 255.255.255.0
  tunnel mode gre multipoint
  tunnel source e0/1
  tunnel protection ipsec profile IPSEC-PROF1
  ip nhrp network-id 12345
  ip nhrp authentication C1sc0123
  ip nhrp map multicast dynamic
  ip nhrp redirect
 !
 no ip access-list extended NAT
 ip access-list extended NAT
  deny ip host 10.2.2.1 host 10.1.1.1
  deny ip host 10.2.2.1 192.168.3.0 0.0.0.255
  deny ip host 10.2.2.1 192.168.4.0 0.0.0.255
  deny ip host 10.2.2.1 192.168.5.0 0.0.0.255
  permit ip 10.2.2.0 0.0.0.3 any
  end
~~~

<br>

~~~
!@E3
conf t
 int tunnel0
  ip add 172.16.1.3 255.255.255.0
  tunnel mode gre multipoint
  tunnel source e0/1
  tunnel protection ipsec profile IPSEC-PROF1
  ip nhrp network-id 12345
  ip nhrp authentication C1sc0123
  ip nhrp map 172.16.1.254  1.10.1.10
  ip nhrp map 172.16.1.253  2.20.2.20
  ip nhrp map multicast 1.10.1.10
  ip nhrp map multicast 2.20.2.20
  ip nhrp nhs 172.16.1.254
  ip nhrp nhs 172.16.1.253
  ip nhrp shortcut
!
 no ip access-list extended NAT
 ip access-list extended NAT
  deny ip host 192.168.3.101 host 10.1.1.1
  deny ip host 192.168.3.101 host 10.2.2.1
  deny ip host 192.168.3.101 192.168.4.0 0.0.0.255
  deny ip host 192.168.3.101 192.168.5.0 0.0.0.255
  permit ip 192.168.3.0 0.0.0.255 any
  end
~~~

<br>

~~~
!@E4
conf t
 int tunnel0
  ip add 172.16.1.4 255.255.255.0
  tunnel mode gre multipoint
  tunnel source e0/1
  tunnel protection ipsec profile IPSEC-PROF1
  ip nhrp network-id 12345
  ip nhrp authentication C1sc0123
  ip nhrp map 172.16.1.254  1.10.1.10
  ip nhrp map 172.16.1.253  2.20.2.20
  ip nhrp map multicast 1.10.1.10
  ip nhrp map multicast 2.20.2.20
  ip nhrp nhs 172.16.1.254
  ip nhrp nhs 172.16.1.253
  ip nhrp shortcut
 !
 no ip access-list extended NAT
 ip access-list extended NAT
  deny ip host 192.168.4.101 host 10.1.1.1
  deny ip host 192.168.4.101 host 10.2.2.1
  deny ip host 192.168.4.101 192.168.3.0 0.0.0.255
  deny ip host 192.168.4.101 192.168.5.0 0.0.0.255
  permit ip 192.168.4.0 0.0.0.255 any
  end
~~~

<br>

~~~
!@E5
conf t
 int tunnel0
  ip add 172.16.1.5 255.255.255.0
  tunnel mode gre multipoint
  tunnel source e0/1
  tunnel protection ipsec profile IPSEC-PROF1
  ip nhrp network-id 12345
  ip nhrp authentication C1sc0123
  ip nhrp map 172.16.1.254  1.10.1.10
  ip nhrp map 172.16.1.253  2.20.2.20
  ip nhrp map multicast 1.10.1.10
  ip nhrp map multicast 2.20.2.20
  ip nhrp nhs 172.16.1.254
  ip nhrp nhs 172.16.1.253
  ip nhrp shortcut
 !
 no ip access-list extended NAT
 ip access-list extended NAT
  deny ip host 192.168.5.101 host 10.1.1.1
  deny ip host 192.168.5.101 host 10.2.2.1
  deny ip host 192.168.5.101 192.168.3.0 0.0.0.255
  deny ip host 192.168.5.101 192.168.4.0 0.0.0.255
  permit ip 192.168.5.0 0.0.0.255 any
  end
~~~


&nbsp;
---
&nbsp;


### Routing : EIGRP

~~~
!@E1
conf t
 router eigrp 100
  network 10.1.1.0 0.0.0.3
  network 172.16.1.0 0.0.0.255
  end
~~~

<br>

~~~
!@S1
conf t
 ip route 0.0.0.0 0.0.0.0 10.1.1.2
 end
~~~

<br>

~~~
!@E2
conf t
 router eigrp 100
  network 10.2.2.0 0.0.0.3
  network 172.16.1.0 0.0.0.255
  end
~~~

<br>

~~~
!@S2
conf t
 ip route 0.0.0.0 0.0.0.0 10.2.2.2
 end
~~~

<br>

~~~
!@E3
conf t
 router eigrp 100
  network 192.168.3.0 0.0.0.255
  network 172.16.1.0 0.0.0.255
  end
~~~

<br>

~~~
!@P1
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.3.3
 end
~~~

<br>

~~~
!@E4
conf t
 router eigrp 100
  network 192.168.4.0 0.0.0.255
  network 172.16.1.0 0.0.0.255
  end
~~~

<br>

~~~
!@P2
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.4.4
 end
~~~

<br>

~~~
!@E5
conf t
 router eigrp 100
  network 192.168.5.0 0.0.0.255
  network 172.16.1.0 0.0.0.255
  end
~~~

<br>

~~~
!@P3
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.5.5
 end
~~~


<br>
<br>


__SPLIT-HORIZON & NEXT-HOP-SELF__
~~~
!@E1,E2
conf t
 int tunnel0
  no ip split-horizon eigrp 100
  no ip next-hop-self eigrp 100
  end
~~~


&nbsp;
---
&nbsp;


### Routing: OSPF

~~~
!@E1
conf t
 router ospf 1
  network 10.1.1.0 0.0.0.3 area 0
  network 172.16.1.0 0.0.0.255 area 0
 !
 int tunnel0
  ip ospf network point-to-multipoint
  end
~~~

<br>

~~~
!@S1
conf t
 ip route 0.0.0.0 0.0.0.0 10.1.1.2
 end
~~~

<br>

~~~
!@E2
conf t
 router ospf 1 
  network 10.2.2.0 0.0.0.3 area 0
  network 172.16.1.0 0.0.0.255 area 0
 !
 int tunnel0
  ip ospf network point-to-multipoint
  end
~~~

<br>

~~~
!@S2
conf t
 ip route 0.0.0.0 0.0.0.0 10.2.2.2
 end
~~~

<br>

~~~
!@E3
conf t
 router ospf 1
  network 192.168.3.0 0.0.0.255 area 0
  network 172.16.1.0 0.0.0.255 area 0
 int tunnel0
  ip ospf network point-to-multipoint
  end
~~~

<br>

~~~
!@P1
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.3.3
 end
~~~

<br>

~~~
!@E4
conf t
 router ospf 1
  network 192.168.4.0 0.0.0.255 area 0
  network 172.16.1.0 0.0.0.255 area 0
 int tunnel0
  ip ospf network point-to-multipoint
  end
~~~

<br>

~~~
!@P2
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.4.4
 end
~~~

<br>

~~~
!@E5
conf t
 router ospf 1
  network 192.168.5.0 0.0.0.255 area 0
  network 172.16.1.0 0.0.0.255 area 0
 int tunnel0
  ip ospf network point-to-multipoint
  end
~~~

<br>

~~~
!@P3
conf t
 ip route 0.0.0.0 0.0.0.0 192.168.5.5
 end
~~~


<br>
<br>

---
&nbsp;


### Exercise: Verify and TShoot the DMVPN Configurations
~~~
!@P1
traceroute 192.168.5.103
~~~

<br>

~~~
!@E3,E4,E5
conf t
 crypto ikev2 keyring KEY1
  peer E3
   address 3.30.3.30 255.255.255.255
   pre-shared-key C1sc0123
  peer E4
   address 4.40.4.40 255.255.255.255
   pre-shared-key C1sc0123
  peer E5
   address 5.50.5.50 255.255.255.255
   pre-shared-key C1sc0123
 !
 crypto ikev2 profile IKEV2-PROF1
  match identity remote address 1.10.1.10
  match identity remote address 2.20.2.20
  match identity remote address 3.30.3.30
  match identity remote address 4.40.4.40
  match identity remote address 5.50.5.50
  authentication remote pre-share
  authentication local pre-share
  keyring local KEY1
  end
~~~


<br>
<br>

---
&nbsp;


### Tunnel Settings

1. MTU Adjustment
- MTU defines the absolute largest packet size an interface can forward. If a packet exceeds this size, the router must either fragment it or drop it (if the Don't Fragment bit is set).

*Lower the MTU on the tunnel interface to account for the ~50-70 bytes of IPsec overhead.*

<br>

2. TCP MSS Clamping
- TCP MSS (Maximum Segment Size) clamping is a proactive fix. When a user establishes a TCP connection (like HTTPS or SSH), the router intercepts the handshake and artificially lowers the requested packet size.

*The router edits the TCP SYN packet. It tells both computers, "Do not send packets larger than X bytes because this path has a VPN."*

~~~
!@E1,E2,E3,E4,E5
conf t
 int tun0
  ip mtu 1400
  ip tcp adjust-mss 1360
  bandwidth 100000
  delay 1000
  end
~~~


&nbsp;
---
&nbsp;


3. IPSec VPN Prefragmentation
- Pre-fragmentation for IPsec VPNs operates in IPsec tunnel mode and IPsec tunnel mode with GRE, but not with IPsec transport mode.   

*Pre-fragmentation is a reactive fix for packets that are already too big for the VPN tunnel, especially non-TCP traffic that MSS clamping cannot touch.*  

When a large UDP packet arrives, the router splits it into smaller cleartext pieces before wrapping them in IPsec. The receiving router can decrypt each piece immediately in hardware without waiting to reassemble them.  
 
~~~
!@E1,E2,E3,E4,E5
conf t
 crypto ipsec transform-set TS1 esp-aes 256 esp-sha256-hmac
  mode tunnel
 crypto ipsec transform-set TS2 esp-gcm 256
  mode tunnel
 crypto ipsec fragmentation before-encryption
 end
~~~


<br>
<br>

---
&nbsp;


# DMVPN : Certificate Auth

### STEP 1 - Setup WinServer

| NetAdapter  | VMNet   | IP Address        | 
| ---         | ---     | ---               |
| 1           | NAT     | 208.8.8.8     /24 |
| 2           | VMNet2  | 192.168.102.8 /24 |
| 3           | VMNet16 | 10.69.255.4   /29 |

<br>

~~~
!@Powershell
set-netfirewallprofile -name private,public,domain -enabled false
rename-computer ccnp21
ncpa.cpl
~~~

<br>

~~~
!@WinVM-cmd
route add 172.16.29.0 mask 255.255.255.0 10.69.255.6 -p
~~~


&nbsp;
---
&nbsp;


### STEP 2 - Install Active Directory Domains and Services
Create a Service Account:
- Active Directory and Users and Computers

| New User    |              |
| ---         | ---          |
| User Name   | ca           |
| Full Name   | CERTAUTH     |
| Password    | C1sc0123     |
| Pass Policy | Never Expire |
| Member of   | IIS_IUSRS    |


&nbsp;
---
&nbsp;


### STEP 3 - Install Active Directory Certificate Services

Afterwards, install ADCS Add-Ons:
- Certificate Enrollment Policy Web Service  
- Certificate Enrollment Web Service         
- Certificate Authority Web Enrollment
- Network Device Enrollment Service


&nbsp;
---
&nbsp;


### STEP 4 - Configure Certificates on Cisco (.3 ako sa lab, .4 original)
__TRUSTPOINTS__
~~~
!@E1
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://10.69.255.3/certsrv/mscep/mscep.dll
  serial-number
  fqdn e1.ccnp21.com
  ip-address 1.10.1.10
  subject-name CN=E1,OU=HQ,O=RIVANCORP,L=MANILA,ST=NCR,C=PH
  subject-alt-name e1.ccnp21.com
  revocation-check none
  source interface e3/0
  rsakeypair CERTKEY
  vrf MANAGEMENT
  exit
 !
 crypto pki authenticate CCNPTRUST !after nito, yes then #crypto pki enroll CCNPTRUST  , then verify HQ-Edge-1#show crypto pki trustpoints status
~~~

<br>

~~~
!@E2
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://10.69.255.3/certsrv/mscep/mscep.dll
  serial-number
  fqdn e2.ccnp21.com
  ip-address 2.20.2.20
  subject-name CN=E2,OU=HQ,O=RIVANCORP,L=MARIKINA,ST=NCR,C=PH
  subject-alt-name e2.ccnp21.com
  revocation-check none
  source interface e3/0
  rsakeypair CERTKEY
  vrf MANAGEMENT
  exit
 !
 crypto pki authenticate CCNPTRUST
~~~

<br>

~~~
!@E3
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://10.69.255.3/certsrv/mscep/mscep.dll
  serial-number
  fqdn e3.ccnp21.com
  ip-address 3.30.3.30
  subject-name CN=E3,OU=BRANCH3,O=RIVANCORP,L=MAKATI,ST=NCR,C=PH
  subject-alt-name e3.ccnp21.com
  revocation-check none
  source interface e3/0
  rsakeypair CERTKEY
  vrf MANAGEMENT
  exit
 !
 crypto pki authenticate CCNPTRUST
~~~

<br>

~~~
!@E4
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://10.69.255.3/certsrv/mscep/mscep.dll
  serial-number
  fqdn e4.ccnp21.com
  ip-address 4.40.4.40
  subject-name CN=E4,OU=BRANCH4,O=RIVANCORP,L=TOKYO,ST=KANTO,C=JP
  subject-alt-name e4.ccnp21.com
  revocation-check none
  source interface e3/0
  rsakeypair CERTKEY
  vrf MANAGEMENT
  exit
 !
 crypto pki authenticate CCNPTRUST
~~~

<br>

~~~
!@E5
conf t
 crypto key generate rsa modulus 2048 label CERTKEY
 !
 crypto pki trustpoint CCNPTRUST
  enrollment url http://10.69.255.3/certsrv/mscep/mscep.dll
  serial-number
  fqdn e5.ccnp21.com
  ip-address 5.50.5.50
  subject-name CN=E5,OU=BRANCH5,O=RIVANCORP,L=WS,ST=WASHINGTON,C=US
  subject-alt-name e5.ccnp21.com
  revocation-check none
  source interface e3/0
  rsakeypair CERTKEY
  vrf MANAGEMENT
  exit
 !
 crypto pki authenticate CCNPTRUST
~~~


&nbsp;
---
&nbsp;


### STEP 6 - Access CA Web Enrollment

http://10.69.255.4/certsrv/mscep/mscep.dll  


<br>
<br>

Grab the Hash & Challenge Password    
- Hash: ___________    
- Pass: ___________  


&nbsp;
---
&nbsp;


### STEP 7 - Enroll Network Devices

~~~
!@E1,E2,E3,E4,E5
conf t
 crypto pki enroll CCNPTRUST

~~~


<br>
<br>

---
&nbsp;


### VERIFICATION - Modify DMVPN Tunnel to use RSA-SIGNATURE
~~~
!@E1,E2,E3,E4,E5
conf t
 crypto ikev2 profile IKEV2-PROF1
  no authentication remote pre-share
  authentication remote rsa-sig
  authentication local rsa-sig
  pki trustpoint CCNPTRUST
  no keyring local KEY1
  end
~~~

<br>

__Verify__
~~~
!@E1,E2,E3,E4,E5
clear crypto ikev2 sa fast 
sh crypto ikev2 sa
~~~


<br>
<br>

---
&nbsp;


# AAA Radius

~~~
!@All Devices
conf t
 int e3/0
  no shut
  end
ping vrf MANAGEMENT 10.69.255.4
~~~


<br>


~~~
!@RST Devices
conf t
 aaa new-model
 radius server WINRAD
  address ipv4 10.69.255.4 auth-port 1812 acct-port 1813
  key C1sc0123
 !
 aaa group server radius RADGROUP
  server name WINRAD
  ip vrf forwarding MANAGEMENT
 ip radius source-int e3/0
 !
 aaa authentication login default group RADGROUP local
 aaa authorization exec default group RADGROUP local
 !
 line vty 0 4
  login authentication default
  end
~~~


<br>
<br>

---
&nbsp;


# LISP

- EID : Endpoint Identifier
- Map Resolver/Server
- Route Locator
- ITR : Points IP address for Map Resolver   ! ITR + ETR = XTR
- ETR : Register EID to Map Server

| EID            | RLOC      | 
| ---            | ---       |
| 192.168.3.0/24 | 3.30.3.30 |
| 192.168.4.0/24 | 4.40.4.40 |
| 192.168.5.0/24 | 5.50.5.50 |



~~~
!@E1,E2,E3,E4,E5
conf t
 no int tunnel0
 end
~~~

<br>

__Map Resolver /Map Server__
~~~
!@E1-MS/MR
conf t
 router lisp
  ipv4 map-server
  ipv4 map-resolver
 !
  site E2
   eid-prefix 10.2.2.0/30
   authentication-key C1sc0123
 !
  site E3
   eid-prefix 192.168.3.0/24
   authentication-key C1sc0123
 !
  site E4
   eid-prefix 192.168.4.0/24
   authentication-key C1sc0123
 !
  site E5
   eid-prefix 192.168.5.0/24
   authentication-key C1sc0123
 !
 end
show lisp site summary
show ip lisp 
~~~

<br>

~~~
!@E2-XTR
conf t
 router lisp
  ipv4 itr
  ipv4 etr
  ipv4 itr map-resolver 1.10.1.10
  ipv4 etr map-server 1.10.1.10 key C1sc0123
  locator-set RLOCS
   ipv4-interface e0/1 priority 100 weight 100
  !
  database-mapping 10.2.2.0/30 locator-set RLOCS
  end
sh ip lisp map-cache 
~~~

<br>

~~~
!@E3-XTR
conf t
 router lisp
  ipv4 itr
  ipv4 etr
  ipv4 itr map-resolver 1.10.1.10
  ipv4 etr map-server 1.10.1.10 key C1sc0123
  locator-set RLOCS
   ipv4-interface e0/1 priority 100 weight 100
  !
  database-mapping 192.168.3.0/24 locator-set RLOCS
  end
sh ip lisp map-cache 
~~~

<br>

~~~
!@E4-XTR
conf t
 router lisp
  ipv4 itr
  ipv4 etr
  ipv4 itr map-resolver 1.10.1.10
  ipv4 etr map-server 1.10.1.10 key C1sc0123
  locator-set RLOCS
   ipv4-interface e0/1 priority 100 weight 100
  !
  database-mapping 192.168.4.0/24 locator-set RLOCS
  end
sh ip lisp map-cache 
~~~

<br>

~~~
!@E5-XTR
conf t
 router lisp
  ipv4 itr
  ipv4 etr
  ipv4 itr map-resolver 1.10.1.10
  ipv4 etr map-server 1.10.1.10 key C1sc0123
  locator-set RLOCS
   ipv4-interface e0/1 priority 100 weight 100
  !
  database-mapping 192.168.5.0/24 locator-set RLOCS
  end
sh ip lisp map-cache 
~~~



