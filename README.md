# 2020-04-15-lab-vlan-de-base

https://cisco.goffinet.org/ccna/vlans/lab-vlan-base-cisco-ios/

# 1.Configuration des paramètres globaux

  Sur SW0 et SW1 :

        Switch>enable
        Switch#conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        Switch(config)#hostname SW0
        SW0(config)#ip domain-name entreprise.lan
        SW0(config)#enable secret cisco
        SW0(config)#username admin secret cisco
        SW0(config)#crypto key generate rsa
        The name for the keys will be: SW0.entreprise.lan
        Choose the size of the key modulus in the range of 360 to 4096 for your
          General Purpose Keys. Choosing a key modulus greater than 512 may take
          a few minutes.

        How many bits in the modulus [512]: 2048
        % Generating 2048 bit RSA keys, keys will be non-exportable...
        [OK] (elapsed time was 1 seconds)

        SW0(config)#
        *Apr 15 09:43:07.093: %SSH-5-ENABLED: SSH 1.99 has been enabled
        SW0(config)#ip ssh version 2
        SW0(config)#line vty 0 15
        SW0(config-line)#login local
        SW0(config-line)#transport input ?
          all     All protocols
          lat     DEC LAT protocol
          nasi    NASI protocol
          none    No protocols
          pad     X.3 PAD
          rlogin  Unix rlogin protocol
          ssh     TCP/IP SSH protocol
          telnet  TCP/IP Telnet protocol
          udptn   UDPTN async via UDP protocol

        SW0(config-line)#transport input ssh
        SW0(config-line)#logging synchronous
        SW0(config-line)#exit
        SW0(config)#exit

  Configuration statique de l’interface de gestion :

        SW0(config)#interface vlan 99
        SW0(config-if)#ip address 192.168.1.1 255.255.255.0
        SW0(config-if)#no shutdown
        SW0(config-if)#exit
        SW0(config)#ip default-gateway 192.168.1.254
        SW0(config)#exit

        SW1(config)#interface vlan 99
        SW1(config-if)#ip address 192.168.1.2 255.255.255.0
        SW1(config-if)#no shutdown
        SW1(config-if)#exit
        SW1(config)#ip default-gateway 192.168.1.254
        SW1(config)#exit

# 2.Création des VLANs sur les commutateurs

        SW0(config)#vlan 10
        SW0(config-vlan)#name DATA
        SW0(config-vlan)#exit
        SW0(config)#vlan 20
        SW0(config-vlan)#name VOICE
        SW0(config-vlan)#exit
        SW0(config)#vlan 99
        SW0(config-vlan)#name MANAGEMENT
        SW0(config-vlan)#exit
        SW0(config)#vlan 100
        SW0(config-vlan)#name NATIVE
        SW0(config-vlan)#exit

  Vérification des VLANs :

        SW0#show vlan

        VLAN Name                             Status    Ports
        ---- -------------------------------- --------- -------------------------------
        1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                        Gi1/0, Gi1/1, Gi1/2, Gi1/3
                                                        Gi2/0, Gi2/1, Gi2/2, Gi2/3
                                                        Gi3/0, Gi3/1, Gi3/2, Gi3/3
        10   DATA                             active    
        20   VOICE                            active    
        99   MANAGEMENT                       active    
        100  NATIVE                           active    
        1002 fddi-default                     act/unsup 
        1003 token-ring-default               act/unsup 
        1004 fddinet-default                  act/unsup 
        1005 trnet-default                    act/unsup



 Désactivation de VTP sur SW0 et SW1 :

        SW0(config)#vtp mode off
        Setting device to VTP Off mode for VLANS.

        SW1(config)#vtp mode off
        Setting device to VTP Off mode for VLANS.


# 3.Configuration des ports « Access » et « Trunk »

          SW0(config)#interface range g1/0-3
          SW0(config-if-range)#switchport mode access
          SW0(config-if-range)#switchport access vlan 10
          SW0(config-if-range)#spanning-tree portfast
          SW0(config-if-range)#exit
          SW0(config)#interface range g2/0-3
          SW0(config-if-range)#switchport mode access
          SW0(config-if-range)#switchport access vlan 20
          SW0(config-if-range)#spanning-tree portfast
          SW0(config-if-range)#end
          SW0(config)#interface range g0/1-2
          SW0(config-if-range)#switchport trunk encapsulation dot1q
          SW0(config-if-range)#switchport trunk native vlan 100
          SW0(config-if-range)#switchport mode trunk
          SW0(config-if-range)#end

          SW1(config)#interface range g1/0-3
          SW1(config-if-range)#switchport mode access
          SW1(config-if-range)#switchport access vlan 10
          SW1(config-if-range)#spanning-tree portfast
          SW1(config-if-range)#exit
          SW1(config)#interface range g2/0-3
          SW1(config-if-range)#switchport mode access
          SW1(config-if-range)#switchport access vlan 20
          SW1(config-if-range)#spanning-tree portfast
          SW1(config-if-range)#end
          SW1#conf t
          SW1(config)#interface g0/1
          SW1(config-if)#switchport trunk encapsulation dot1q
          SW1(config-if)#switchport trunk native vlan 100
          SW1(config-if)#switchport mode trunk
          SW1(config-if)#end

# 4.Activation du routage

   Configuration du Trunk :

          Router(config)#interface g0/0
          Router(config-if)#no ip address
          Router(config-if)#no shutdown
          Router(config-if)#interface g0/0.1
          Router(config-subif)#encapsulation dot1q 10
          Router(config-subif)#ip address 192.168.10.254 255.255.255.0
          Router(config-subif)#exit
          Router(config)#interface g0/0.2
          Router(config-subif)#encapsulation dot1q 20
          Router(config-subif)#ip address 192.168.20.254 255.255.255.0
          Router(config-subif)#exit
          Router(config)#interface g0/0.3
          Router(config-subif)#encapsulation dot1q 99
          Router(config-subif)#ip address 192.168.1.254 255.255.255.0
          Router(config-subif)#exit
          Router(config)#interface g0/0.4
          Router(config-subif)#encapsulation dot1q 100 native
          Router(config-subif)#end

   Vérification de la configuration du routeur :

          Router#show ip int b
          Interface                  IP-Address      OK? Method Status                Protocol
          GigabitEthernet0/0         unassigned      YES NVRAM  up                    up      
          GigabitEthernet0/0.1       192.168.10.254  YES manual up                    up      
          GigabitEthernet0/0.2       192.168.20.254  YES manual up                    up      
          GigabitEthernet0/0.3       192.168.1.254   YES manual up                    up      
          GigabitEthernet0/0.4       unassigned      YES unset  up                    up      
          GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down    
          GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
          GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down    


          Router#show ip route
                192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.1.0/24 is directly connected, GigabitEthernet0/0.3
          L        192.168.1.254/32 is directly connected, GigabitEthernet0/0.3
                192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.10.0/24 is directly connected, GigabitEthernet0/0.1
          L        192.168.10.254/32 is directly connected, GigabitEthernet0/0.1
                192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.20.0/24 is directly connected, GigabitEthernet0/0.2
          L        192.168.20.254/32 is directly connected, GigabitEthernet0/0.2


   Configuration du service DHCP sur le routeur :

          Router(config)#ip dhcp pool VLAN10 
          Router(dhcp-config)#network 192.168.10.0 255.255.255.0
          Router(dhcp-config)#default-router 192.168.10.254
          Router(dhcp-config)#dns-server 1.1.1.1
          Router(dhcp-config)#exit
          Router(config)#ip dhcp pool VLAN20
          Router(dhcp-config)#default-router 192.168.20.254
          Router(dhcp-config)#network 192.168.20.0 255.255.255.0
          Router(dhcp-config)#dns-server 1.1.1.1
          Router(dhcp-config)#exit
          Router(config)#exit

   Configuration d’un commutateur multicouche :

   Dans la topologie, on remplace le router par un switch de couche 3. Dans GNS3, on choisit le switch Cisco IOSvL2 15.2. On connecte l’interface g0/2 de SW0 à l’interface g0/1 du nouveau switch. On renomme ce switch : « DS0 ».

          DS0(config)#int g0/1
          DS0(config-if)#switchport trunk encapsulation dot1q
          DS0(config-if)#switchport trunk native vlan 100
          DS0(config-if)#switchport mode trunk
          DS0(config-if)#end

          DS0(config)#vlan 10
          DS0(config-vlan)#vlan 20  
          DS0(config-vlan)#vlan 99
          DS0(config-vlan)#vlan 100
          DS0(config-vlan)#end

          DS0(config)#int vlan 10
          DS0(config-if)#ip add 192.168.10.254 255.255.255.0
          DS0(config-if)#no shutdown
          DS0(config-if)#exit
          DS0(config)#int vlan 20
          DS0(config-if)#ip add 192.168.20.254 255.255.255.0
          DS0(config-if)#no shutdown
          DS0(config-if)#exit
          DS0(config)#int vlan 99
          DS0(config-if)#no shutdown
          DS0(config-if)#ip add 192.168.1.254 255.255.255.0
          DS0(config-if)#exit
          DS0(config)#exit

          DS0(config)#ip dhcp pool VLAN10
          DS0(dhcp-config)#network 192.168.10.0 /24
          DS0(dhcp-config)#default-router 192.168.10.254
          DS0(dhcp-config)#dns-server 192.168.10.254
          DS0(dhcp-config)#exit
          DS0(config)#ip dhcp pool VLAN20
          DS0(dhcp-config)#network 192.168.20.0 /24
          DS0(dhcp-config)#default-router 192.168.20.254
          DS0(dhcp-config)#dns-server 192.168.20.254
          DS0(dhcp-config)#end
          DS0#wr

   Penser à activer le routage IPv4 :

          DS0(config)#ip routing

   Vérification sur le commutateur de couche 3 :

          DS0#show ip route
          Gateway of last resort is not set

                192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.1.0/24 is directly connected, Vlan99
          L        192.168.1.254/32 is directly connected, Vlan99
                192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.10.0/24 is directly connected, Vlan10
          L        192.168.10.254/32 is directly connected, Vlan10
                192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
          C        192.168.20.0/24 is directly connected, Vlan20
          L        192.168.20.254/32 is directly connected, Vlan20



