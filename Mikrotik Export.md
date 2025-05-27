# Mikrotik Setup
Ideally you want an ARM of ARM64 device, but a similar device would work.
n/ For our demonstration, we are using a hAP ac lite TC.

## IP Setup
SIEM: 192.168.100.200
Mikrotik: 192.168.100.2

# RSC Export
- Some of this is cosmetic only such as naming schemes and the NetFlow list.

/interface bridge
add name="Firewall Bridge"
/interface wireless
set [ find default-name=wlan1 ] country=australia installation=indoor mode=\
    ap-bridge name="2GHz Wireless" ssid="MikroTik 2.4ghz"
set [ find default-name=wlan2 ] band=5ghz-a/n/ac country=australia disabled=\
    no installation=indoor mode=ap-bridge name="5GHz Wireless" ssid=\
    "MikroTik 5ghz"
/interface ethernet
set [ find default-name=ether1 ] name="E1 - Firewall Uplink"
set [ find default-name=ether2 ] name="E2 - **"
set [ find default-name=ether3 ] name="E3 - ***"
set [ find default-name=ether4 ] advertise=10M-baseT-half,10M-baseT-full \
    name="E4 - ***"
set [ find default-name=ether5 ] name="E5 - ***"
/interface list
add name=WAN
add name=LAN
add name=NetFlow_interfaces
/interface wireless security-profiles
set [ find default=yes ] authentication-types=wpa2-psk mode=dynamic-keys \
    supplicant-identity=MikroTik wpa2-pre-shared-key=Mikrotik
/interface bridge port
add bridge="Firewall Bridge" interface="E1 - Firewall Uplink"
add bridge="Firewall Bridge" interface="E2 - **"
add bridge="Firewall Bridge" interface="E3 - ***"
add bridge="Firewall Bridge" interface="E4 - ***"
add bridge="Firewall Bridge" interface="E5 - ***"
add bridge="Firewall Bridge" interface="5GHz Wireless"
add bridge="Firewall Bridge" interface="2GHz Wireless"
/ip neighbor discovery-settings
set discover-interface-list=!dynamic
/interface list member
add interface="5GHz Wireless" list=WAN
add interface="Firewall Bridge" list=LAN
add interface="E2 - **" list=NetFlow_interfaces
add interface="E3 - ***" list=NetFlow_interfaces
add interface="E4 - ***" list=NetFlow_interfaces
add interface="E5 - ***" list=NetFlow_interfaces
add interface="2GHz Wireless" list=NetFlow_interfaces
add interface="5GHz Wireless" list=NetFlow_interfaces
add interface="E2 - **" list=LAN
add interface="E3 - ***" list=LAN
add interface="E4 - ***" list=LAN
add interface="E5 - ***" list=LAN
add interface="E1 - Firewall Uplink" list=LAN
/interface wireless access-list
add interface="5GHz Wireless" mac-address=B6:31:B7:2B:8E:24 vlan-mode=no-tag
/ip address
add address=192.168.100.2/24 disabled=yes interface="5GHz Wireless" network=\
    192.168.100.0
add address=192.168.100.2/24 interface="Firewall Bridge" network=\
    192.168.100.0
/ip dhcp-client
add default-route-tables=main disabled=yes interface="E1 - Firewall Uplink"
/ip traffic-flow
set enabled=yes interfaces=NetFlow_interfaces
/ip traffic-flow target
add dst-address=192.168.100.200
/system clock
set time-zone-name=Australia/Sydney
/system identity
set name="VU23220 "
/system logging
add action=remote topics=info
add action=remote topics=error
add action=remote topics=critical
add action=remote topics=warning
add action=remote topics=bridge,stp
/system note
set note="Unauthorised Access is Prohibited!\r\
    \nThis device is in use for the VU23220 for the Cert IV of Cybersecurity!!\
    !\r\
    \n\r\
    \nPlease do not wipe. Get another device from E2.23."
/system routerboard settings
set auto-upgrade=yes
/tool romon
set enabled=yes
