# Wireguard
Wireguard is a modern VPN protocol with high speed and low latency. It is generally preffered over openvpn thanks to its minimal overhead.

## Generate Client Key Pair
A key pair is used for encryption in wireguard. Two sets of keys are generated, one for the server and one for the client. 
1. SSH into your VyOS router
2. Generate a client key pair <br> `generate pki wireguard key-pair`
3. Save the output somewhere

## Create Wireguard interface on VyOS
We need a wireguard interface on vyos for the client to connect. The interface will take up one port on your WAN interface.
1. Create a new wireguard interface with new keys<br>`generate pki wireguard key-pair install interface wg10`
2. Save the public key somewhere
3. Set the interface address<br>`set interfaces wireguard wg10 address 10.15.21.1/30`<br>
You may choose to use a different ip address but stuck with something within 10.0.0.0 and with /30
4. Add a description<br>`set interfaces wireguard wg10 description 'client'`
5. Set the port to be used<br>`set interfaces wireguard wg10 port 30000`<br>Replace 30000 with your desired port number
6. Set the client public key<br>`set interfaces wireguard wg10 peer 'client' public-key '[client public key]'`
7. Set the routes<br>`set interfaces wireguard wg10 peer 'LAPTOP' allowed-ips 10.15.21.0/30`<br>`set interfaces wireguard wg10 peer 'LAPTOP' allowed-ips 10.72.0.0/14`<br>Replace 'LAPTOP' with your desired peer name (can be anything)
8. Set the keepalive so wireguard retries in case of a dropped connection<br>`set interfaces wireguard wg10 peer LAPTOP persistent-keepalive 25`

## Open Firewall Port
A firewall rule is needed to allow udp traffic into the wireguard port.<br>
<br>
This example will use `WIREGUARD_IN` as the sub ruleset name and port `30000` for the inbound port.

## Sub Ruleset
1. Create a new subruleset<br>`set firewall ipv4 name WIREGUARD_IN rule 10 action accept`
2. Allow established and related connections<br>`set firewall ipv4 name WIREGUARD_IN rule 10 description 'Allow established/related'`<br>
`set firewall ipv4 name WIREGUARD_IN rule 10 state established`<br>
`set firewall ipv4 name WIREGUARD_IN rule 10 state related`
3. Allow new connections<br>`set firewall ipv4 name WIREGUARD_IN rule 20 action accept`
4. Set description<br>`set firewall ipv4 name WIREGUARD_IN rule 20 description 'WireGuard port'`
5. Set port<br>`set firewall ipv4 name WIREGUARD_IN rule 20 destination port 30000`
6. Enable logs<br>`set firewall ipv4 name WIREGUARD_IN rule 20 log`
7. UDP only because wireguard is udp only<br>`set firewall ipv4 name WIREGUARD_IN rule 20 protocol 'udp'`
8. Set port as source<br>`set firewall ipv4 name WIREGUARD_IN rule 20 source`
9. Return to main ruleset if not wireguard related traffic<br>`set firewall ipv4 name WIREGUARD_IN default-action return`

## Jump Point
A jump point is needed to make sure the sub ruleset is used.
1. Create a new jump rule <br>`set firewall ipv4 input filter rule 90 action jump`<br>
Use an available rule number on your system
2. Set the jump target<br>`set firewall ipv4 input filter rule 90 jump-target WIREGUARD_IN`

## Create NAT
A nat source rule is needed to allow other devices to reach the client.
1. Set the nat source<br>`set nat source rule 200 source address 10.15.21.0/30`<br>Replace 200 and 10.15.21.0/30 with your available rule number and the wgireguard subnet you set earlier
2. Set the destination<br>`set nat source rule 200 destination address 10.72.0.0/14`<br>This makes sure only traffic destined for constellation is translated
3. Set the translation address<br>`set nat source rule 200 translation address 10.72.21.1`<br>Replace 10.72.21.1 with your routers ip

## Create client config
The client uses information from the config file to connect to the router.<br>
<br>
Fill out the following template with information written down from earlier, don't include the {} curly braces:<br>
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
Save it as a .conf file and import it in your wireguard client of choice.