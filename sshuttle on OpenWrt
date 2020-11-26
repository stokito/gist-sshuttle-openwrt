# sshuttle on OpenWrt

## extroot

Before you can install `sshuttle`, you'll need to add some space. Pop a USB drive in the back and follow [these directions](https://openwrt.org/docs/guide-user/additional-software/extroot_configuration).

Once you have the space, install sshuttle: 

```bash
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install python3 python3-pip iptables-mod-extra iptables-mod-nat-extra iptables-mod-ipopt
root@OpenWrt:~# python3 /usr/bin/pip3 install sshuttle
```

## Create your wifi access point

I really wanted a wireless access point that tunneled everything on it through sshuttle. To do that, you'll need to add a `Static IP` interface, and give it a unique block of DHCP addresses to give clients. I used `192.168.2.0/24`. The interface should be in the `lan` firewall group and bridge to the `wan` port. 

Next, create a wifi access point that uses your new interface. 

## ssh key

Generate an ssh key to add to authorized keys on the remote server:

```bash
root@OpenWrt:~# dropbearkey -t rsa -f /root/.ssh/id_rsa
```

## sshuttle.conf

Create a file called `sshuttle.conf` that looks something like this: 

```bash
-D
-v
0/0
-l
0.0.0.0:12345
--ns-hosts
192.168.2.1
-e
ssh -i /root/.ssh/id_rsa
-r
you@remote-host
-x
192.168.0.0/16
```

# Start sshuttle

You should be set. To start `sshuttle`, run:

```bash
root@OpenWrt:~# sshuttle @sshuttle.conf
```

If everything is working, then everything passing through the router should be tunneled. To restrict tunneling to just the `192.168.2.1/24` subnet, you'll need to add an `iptables` rule:

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