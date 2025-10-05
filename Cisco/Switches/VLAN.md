# Using VLANs on a Cisco Switch
VLANs are a helpful way of splitting up your switch so you can have more security.

## Configuring a VLAN
1. Typed `enable`
2. Typed `configure terminal` or `conf t` (1)
3. Typed `vlan 75` - Change 75 to any VLAN number
4. Typed `name IXP` - Change IXP to any ASCII name
5. Typed `exit`

## Making an access port
Confirm that you are in configure mode (1)

1. Typed `interface Gi0/2`
2. Typed `switchport mode access`
3. Typed `switchport access vlan 75`
4. Typed `exit`

## Making a trunk port
Confirm that you are in configure mode (1)

1. Typed `interface range Gi0/0-1`
2. Typed `switchport trunk encapsulation dot1q`
3. Typed `switchport mode trunk`
4. Typed `switchport mode trunk allowed vlan add 75` - This limits the VLANs the port can access (optional)
5. Typed `switchport mode trunk native vlan {number}` - Dont put the vlan you are using but this is what the device will use untagged. Use this for access points etc
6. Typed `exit`

## Write config
Ensure you just see `Switch#` by typing `exit`

1. Typed `wr`