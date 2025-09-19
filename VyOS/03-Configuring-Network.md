# Configuring Networking on VyOS
its a router you need something to route

## Static IP
For this example we will use the following range `192.168.75.0/24`. For the purposes of the tutorial ill also use VLANs

1. Typed `conf`
2. Typed `set interfaces ethernet eth0 vif 75 address 192.168.75.2/24`
3. Typed `set interfaces ethernet eth0 vif 75 description "IXP Interface`

## Dynamic IP

**ADD SOMETHING HERE**