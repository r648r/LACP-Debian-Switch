# Wireguard Cheat Sheet

## Serveur

### Installation 

```bash	
sudo apt update && sudo apt install wireguard resolvconf -y
sudo mkdir /etc/wireguard/clients
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```

### Restart VPN serveur

```bash
sudo systemctl restart wg-quick@vpn
sudo systemctl status wg-quick@vpn
```


### Modifier le fichier de configuration 

```bash
export vpn=""
wg-quick down $vpn ; nano /etc/wireguard/$vpn.conf ; wg-quick up $vpn
```

### Voire les hôtes et VPNs actif

`wg show`

```
$ wg show
interface: vpn
public key: 7YA1[...]NVVB=
private key: (hidden)
listening port: 51194

peer: yqx3[...]Vrbd00=
endpoint: 185.XX.XX.90:53542
allowed ips: 10.99.99.2/32
latest handshake: 1 minute, 49 seconds ago
transfer: 20.26 MiB received, 333.57 MiB sent
persistent keepalive: every 20 seconds
```

### Voir les logs

`
journalctl -u wg-quick@wg99
`


### Configuration serveur

`sudo systemctl enable wg-quick@vpn`

`sudo nano /etc/wireguard/vpn.conf`
```init
[Interface]
Address = 10.99.99.1/24
ListenPort = 53
SaveConfig = true
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PostUp = ip6tables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PreDown = ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PrivateKey = <Clef privé du serveur>

[Peer]
PublicKey = <Clef publique du client n°1>
AllowedIPs = 10.99.99.2/32

[Peer]
PublicKey = <Clef publique du client n°1>
AllowedIPs = 10.99.99.3/32

[Peer]
PublicKey = <Clef publique du client n°1>
AllowedIPs = 10.99.99.4/32
```

## Clients

### Générer un couple de clef 

```bash
export name=""
sudo wg genkey | sudo tee ./$name.key
sudo chmod 600 ./$name.key
sudo cat ./$name.key | wg pubkey | sudo tee $name.pub
```

### Configuration client via un QR-Code

```bash
for conf_file in /etc/wireguard/clients/*.conf; do
    echo "Creating ${conf_file}.png QR code file ..."
    qrencode -t png -o "${conf_file}.png" -r "${conf_file}" &&
    mkdir -p /etc/wireguard/clients/QR &&
    mv "${conf_file}.png" /etc/wireguard/clients/QR
done
```

### Configuration client

```
[Interface]
PrivateKey = <Clef privé du client>
Address = 10.99.99.4/32, fd42:420:420::4/128

[Peer]
PublicKey = <Clef publique du serveur>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 84.247.130.86:53
PersistentKeepalive = 25
```
