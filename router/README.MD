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
