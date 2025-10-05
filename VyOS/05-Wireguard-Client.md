# Wireguard
Wireguard is a modern VPN protocol with high speed and low latency. It is generally preffered over openvpn thanks to its minimal overhead.

## Generate Client Key Pair
A key pair is used for encryption in wireguard. Two sets of keys are generated, one for the server and one for the client. 
1. SSH into your VyOS router
2. Type `config`
3. Type `generate pki wireguard key-pair`
4. Save the output somewhere

## Create Wireguard interface on VyOS
We need a wireguard interface on vyos for the client to connect. The interface will have its own key pair and will take up one port on your WAN interface. A subnet for the wireguard interface is needed. Stick with something outside of the designated contellation subnet (10.72.0.0/14) and choose a range of /30. We used 10.15.21.0/30 as an example.
1. Type `generate pki wireguard key-pair install interface wg10`
2. Save the public key somewhere
3. Type `set interfaces wireguard wg10 address 10.15.21.1/30`
4. Type `set interfaces wireguard wg10 description 'client'`
5. Type `set interfaces wireguard wg10 port 30000`<br>Replace 30000 with your desired port number
6. Type `set interfaces wireguard wg10 peer 'client' public-key '[client public key]'`
7. Type `set interfaces wireguard wg10 peer '[peer name]' allowed-ips 10.15.21.0/30`
8. Type `set interfaces wireguard wg10 peer '[peer name]' allowed-ips 10.72.0.0/14`
9. Type `set interfaces wireguard wg10 peer '[peer name]' persistent-keepalive 25`

## Open Firewall Port
A firewall rule is needed to allow udp traffic into the wireguard port.<br>
<br>
This example will use `WIREGUARD_IN` as the sub ruleset name and port `30000` for the inbound port. You do not have to use a sub ruleset.

## Sub Ruleset
1. Type `set firewall ipv4 name WIREGUARD_IN rule 10 action accept`
2. Type `set firewall ipv4 name WIREGUARD_IN rule 10 description 'Allow established/related'`
3. Type `set firewall ipv4 name WIREGUARD_IN rule 10 state established`
4. Type `set firewall ipv4 name WIREGUARD_IN rule 10 state related`
5. Type `set firewall ipv4 name WIREGUARD_IN rule 20 action accept`
6. Type `set firewall ipv4 name WIREGUARD_IN rule 20 description 'WireGuard port'`
7. Type `set firewall ipv4 name WIREGUARD_IN rule 20 destination port 30000`
8. Type `set firewall ipv4 name WIREGUARD_IN rule 20 log`
9. Type `set firewall ipv4 name WIREGUARD_IN rule 20 protocol 'udp'`
10. Type `set firewall ipv4 name WIREGUARD_IN rule 20 source`
11. Type `set firewall ipv4 name WIREGUARD_IN default-action return`

## Jump Point
A jump point is needed to make sure the sub ruleset is used.
1. Type `set firewall ipv4 input filter rule 90 action jump`<br>
Use an available rule number on your system
2. Type `set firewall ipv4 input filter rule 90 jump-target WIREGUARD_IN`

## Create NAT
A nat source rule is needed to allow the client to reach the network. A translation address is used as a middleman for the client. In this example we used 10.72.21.1, please use your routers ip address. Also change 10.15.21.0/30 to your wireguard subnet.
1. Type `set nat source rule 200 source address 10.15.21.0/30`
2. Type `set nat source rule 200 destination address 10.72.0.0/14`
3. Type `set nat source rule 200 translation address 10.72.21.1`

## Create client config
The client uses information from the config file to connect to the router.<br>
<br>
1. Fill out the following template with information written down from earlier, don't include the {} curly braces:<br>
```
[Interface]
PrivateKey = {Client private key from Generate Client Key Pair}
Address = {Client IP address, EX: 10.69.21.2/30}
DNS = 10.72.69.208, 10.72.69.209
[Peer]
PublicKey = {Router public key from Create Wireguard interface on VyOS}
Endpoint = {PUBLIC IP ADDRESS of your router and the port, EX: 128.210.6.74:40005}
AllowedIPs = {Wireguard subnet, EX: 10.15.21.0/30}, 10.72.0.0/14
PersistentKeepalive = 25
```
<br>
2. Save it as a .conf file and import it in your wireguard client of choice.