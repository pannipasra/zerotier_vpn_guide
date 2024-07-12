# This guide rely on official guide ➡ [**VPN Exit Node: Full Tunnel Mode or, Overriding Default Route**](https://docs.zerotier.com/exitnode/)

***Disclaimer***: This guide intends to use as *guide* of newbie linux, network. so don't mind me if it just rewriten from official guide with my understanding 💨


## Perequites:
- Create Zerotier Network


## I am going to talk only 
- Setup an `exit node`.
- Join the `exit node`.
- How to using `exit node` with zerotier app.



### Setup an `exit node`.
- **Join the network**

join + authorized(if needed do authorization in ZerotierCentral)
```
export zerotier_nwid=xxxxxxxxxxxxxxxxx
sudo zerotier-cli join $zerotier_nwid
```
- **Enable IPv4 Forwarding on the exit node**

actually, in this section we can use `vim`.
```
sudo nano /etc/sysctl.conf
```
uncomment the line
```
net.ipv4.ip_forward = 1
```
*Note: ctrl+o (save + hit the `Enter` to confirm), ctrl+x (exit)*

reload settings
```
sudo sysctl -p
```

- **Config Interfaces**

in this section I love to use `route` instead of `ip link show`, since I am confused which one is my actual **WAN**

```
route
```
output:
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         xxxxxxx.setup   0.0.0.0         UG    600    0        0 wlan0
192.168.11.0    0.0.0.0         255.255.255.0   U     600    0        0 wlan0
buffalo.setup   0.0.0.0         255.255.255.255 UH    600    0        0 wlan0
192.168.196.0   0.0.0.0         255.255.255.0   U     0      0        0 ztxxxxxxxx
```
if using: `ip link show`

output: 
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DORMANT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
4: ztxxxxxxxx: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 2800 qdisc pfifo_fast state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
```

- **Set environtment variables**

in my case `WAN_IF=wlan0` and `ZT_IF=ztxxxxxxxx`
```
export ZT_IF=ztxxxxxxxx

export WAN_IF=wlan0
```

- **Config Iptables**

Enable NAT and IP masquerading:
```
sudo iptables -t nat -A POSTROUTING -o $WAN_IF -j MASQUERADE
```
Allow traffic forwarding:
```
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```
Allow traffic forwarding from the ZeroTier interface to the WAN interface:
```
sudo iptables -A FORWARD -i $ZT_IF -o $WAN_IF -j ACCEPT
```
Make `iptables` rules persist after reboot:
```
sudo apt-get install iptables-persistent
```
Save your new iptables rules:
```
sudo netfilter-persistent save
```

- **Restart**
```
sudo reboot
```
After rebooted,
```
sudo iptables-save
```
output:
```
...

-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i ztxxxxxxxx -o wlan0 -j ACCEPT

...

-A POSTROUTING -o wlan0 -j MASQUERADE

...
```

- **Script for tunnel or notunnel**

in this section we have to use `nano`
```
sudo nano ~/.bashrc
```
add this to the bottom of file:
```
# Custom aliases for tunnel control
tunnel() {
    sudo zerotier-cli set $zerotier_nwid allowDefault=1
}

notunnel() {
    sudo zerotier-cli set $zerotier_nwid allowDefault=0
}
```
apply the changes immediately:
```
source ~/.bashrc
```
then after this we can simple use command `tunnel`, `notunnel`;


### Join the `exit node`.

This will setting in **ZerotierCentral** page

Central > Network > Settings > Managed Routes

click **Add Routes** `0.0.0.0/0` via `local-zerotier-ip-of-node-exit` 

*sample* my `exit node` has zerotier lan ip = `192.168.196.99`:
```
0.0.0.0 via 192.168.196.99
```

### How to using `exit node` with zerotier app.
Use app (e.g. zerotier ios), in the under each network.

If you want to use "Full Tunnel" select 'Enable Default Route' or else deselect it.


