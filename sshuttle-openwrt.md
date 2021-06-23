# Install and configure sshuttle on OpenWrt

Note that ANY flash update will wreck the extroot configuration. I didn't realize that and had to:

* Take the USB out and reboot so overlay would detach
* Repartition/format the USB
* [Set up extroot from scratch](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration).

## Configure extroot

Before you can install `sshuttle`, you'll need to add some space. Pop a USB drive in the back of the router and follow [these directions](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration).

Once you have the space, install `sshuttle`: 

```bash
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install python3 python3-pip iptables-mod-extra iptables-mod-nat-extra iptables-mod-ipopt
root@OpenWrt:~# python3 /usr/bin/pip3 install sshuttle
```

## Create a wifi access point

I really wanted a wireless access point that tunneled everything on it through sshuttle. To do that, you'll need to add a `Static IP` interface, and give it a unique block of DHCP addresses to give clients. I used `192.168.2.0/24`. The interface should be in the `lan` firewall group and bridge to the `wan` port. Next, configure a wifi access point to use your new interface. 

## Generate an ssh key

Generate an ssh key to add to authorized keys on the remote server:

```bash
root@OpenWrt:~# dropbearkey -t rsa -f /root/.ssh/id_rsa
```

## Create sshuttle.conf

Create a file called `sshuttle.conf` that looks something like this: 

```bash
0/0
-v
-l
0.0.0.0:12345
-e
ssh -i /root/.ssh/id_rsa
-r
kyle.king@jump.eecs.ninja
--ns-host
192.168.2.1
```

# Start sshuttle

You should be set. To start `sshuttle`, run:

```bash
root@OpenWrt:~# sshuttle @sshuttle.conf
```

Everything passing through the router should now be tunneled. To restrict tunneling to just the `192.168.2.1/24` subnet, you'll need to add an `iptables` rule:

```bash
root@OpenWrt:~# iptables -t nat -I sshuttle-12345 -j RETURN \! --src 192.168.2.0/24
```

You can see the `iptables` rules for `sshuttle` with the following command:

```bash
root@OpenWrt:~# iptables -t nat -L sshuttle-12345
Chain sshuttle-12345 (2 references)
target     prot opt source               destination         
RETURN     all  -- !192.168.2.0/24       anywhere            
RETURN    !udp  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL
RETURN     udp  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL udp dpt:!domain
RETURN     tcp  --  anywhere             192.168.0.0/16      
REDIRECT   tcp  --  anywhere             anywhere             TTL match TTL != 63 redir ports 12345
REDIRECT   udp  --  anywhere             OpenWrt.lan          udp dpt:domain TTL match TTL != 63 redir ports 12299
```