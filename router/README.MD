# Setting up the Teltonika RUTX08

## Synopsis
The Teltonika RUTX08 router runs on a unified operating system based on [OpenWrt](https://openwrt.org/) used by all Teltonika-Networks devices.


## Setup
### Setting up ssh access with pub keys
1. Boot up your router and connect your computer via an RJ45 cable to it.
2. To add your pub keys to the router run the following command:
```bash
ssh root@192.168.1.1 "tee -a /etc/dropbear/authorized_keys" < ~/.ssh/id_rsa.pub
```
The default password for root is `admin01`. Refer to this [page](https://wiki.teltonika-networks.com/view/RUTX08_Device_Recovery_Options) for further information.
3. Check that your keys were successfully added by sshing into the router as follows:
```
ssh -o PasswordAuthentication=no root@192.168.1.1 
```
4. If all ran well, we are going to disable password login for ssh as follows: 
```
uci set dropbear.@dropbear[0].PasswordAuth="0"
uci set dropbear.@dropbear[0].RootPasswordAuth="0"
uci commit dropbear
/etc/init.d/dropbear restart
```
5. If you feel like testing the configuration, logout of the router and try logging in again with password as follows:
```
ssh PubkeyAuthentication=no root@192.168.1.1
```
TODO: Add how to change the default password (working for the webui) over CLI.


### Setting up Wireguard VPN
1. Login to the router via ssh.
2. Setup the `wireguard` device and assigng an ip to it:
```
ip link add dev wg0 type wireguard
ip address add dev wg0 10.0.0.2/24
```
3. Create the wireguard config doc:
```
cat << EOF > wg0.conf
[Interface]
# Note that the pub key of the below private key (produced with 'wg pubkey < path_to_private_key_file_produced_with_wg_genkey) must be added to the wireguard peer on the other end
PrivateKey = <...> # the private key that you produced with 'wg genkey'
ListenPort = 32971

[Peer]
PublicKey = <...> # the public key from your peer
# Note that the below IP address and port must be publicly accessible and static.
Endpoint = ip_address:port_number # of the peer
AllowedIPs = 10.0.0.1/32
# Since the router is behind a NAT and there is also no guarantee its Public address won't change, this is vital, so that the peer with the static public IP can discover the routers endpoint.
PersistentKeepalive = 25
EOF
```
4. Apply the configuration the wireguard device:
```
wg setconf wg0 wg0.conf
```
5. Bring up the wireguard device
```
ip link set up dev wg0
```
6. Make sure the wireguard device is configuard and comes up on each boot automatically:

```
TODO: Add how to set firewall settings over CLI for ssh access over the internet, etc. Had a roadblock with OpenWRT firewall settings via CLI and opted for using the web ui at the moment to get things going.

To enable SSH access from the internet on OpenWrt through the command line, you can use the `uci` command to modify the necessary configuration files. Here are the steps to follow:

1. Connect to your OpenWrt router via SSH using a terminal emulator such as PuTTY.
2. Once you've logged in, use the following command to enable SSH access from the internet:

```
uci set dropbear.@dropbear[0].Interface='wan'
```

This command tells OpenWrt to listen for SSH connections on the WAN interface.

3. Next, you'll need to add a firewall rule to allow incoming SSH traffic from the internet. Use the following command:

```
uci add firewall rule
uci set firewall.@rule[-1].name='SSH from Internet'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].dest_port='22'
uci commit firewall
```

This command creates a new firewall rule that allows incoming SSH traffic on port 22 from the WAN interface.

4. Finally, restart the necessary services to apply the changes:

```
/etc/init.d/dropbear restart
/etc/init.d/firewall restart
```

After completing these steps, SSH access from the internet should be enabled on your OpenWrt router. Remember to use strong passwords and consider using SSH keys for authentication to ensure the security of your router.






To make the above WireGuard setup boot-safe and ensure that it automatically comes up on reboot, you'll need to follow these steps:

1. Create a configuration file for the WireGuard interface. Use the following command to create a new configuration file for the WireGuard interface:

```
uci set network.wg0=interface
uci set network.wg0.proto=wireguard
uci set network.wg0.private_key=$(cat privatekey)
uci add_list network.wg0.addresses='10.0.0.1/24'
```

2. Add the new peer to the WireGuard configuration. Use the following commands to add the peer to the configuration:

```
uci add network wireguard_wg0
uci set network.@wireguard_wg0[-1].public_key=$(cat peer_publickey)
uci set network.@wireguard_wg0[-1].allowed_ips='10.0.0.2/32'
uci set network.@wireguard_wg0[-1].private_key=$(cat peer_privatekey)
```

3. Set up the WireGuard firewall rules. Use the following commands to set up the firewall rules:

```
uci add firewall zone
uci set firewall.@zone[-1].name='wg'
uci set firewall.@zone[-1].input='ACCEPT'
uci set firewall.@zone[-1].forward='ACCEPT'
uci set firewall.@zone[-1].output='ACCEPT'
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='wg'
uci set firewall.@forwarding[-1].dest='lan'
```

4. Save and apply the changes. Use the following commands to save and apply the changes:

```
uci commit
/etc/init.d/network reload
/etc/init.d/firewall reload
```

5. Make the configuration boot-safe. Use the following command to create a startup script that will start the WireGuard interface on boot:

```
echo -e '#!/bin/sh\n/usr/bin/wg-quick up wg0' > /etc/rc.local
chmod +x /etc/rc.local
```

This script will start the `wg0` interface using the `wg-quick` command on boot.

After completing these steps, your WireGuard setup should be boot-safe and will automatically come up on reboot.