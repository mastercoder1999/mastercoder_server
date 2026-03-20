# Réseautique

## Tables d'informations
- UBUNTU SERVER 24.04.4 LTS
### Ip
- 192.168.1.169
- IP EXTERNE NON DIVULGÉ
### Ports
- 5335 (Unbound)
- 53 (PiHole)
- 1346 (SSH)
- 51820 (Wireguard)

## UFW
### Install
```
sudo apt install ufw
sudo ufw reset
```
### Config
```
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 1346/tcp
sudo ufw allow 80/tcp
sudo ufw allow 51820/udp
sudo ufw allow 53
sudo ufw allow in on wg0 to any port 53

sudo ufw route allow in on wg0 out on enp7s0
sudo ufw route allow in on enp7s0 out on wg0

sudo ufw enable
```
### Résultat
```
To                         Action      From
--                         ------      ----
1346/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
22/tcp                     DENY        Anywhere
51820/udp                  ALLOW       Anywhere
OpenSSH                    ALLOW       Anywhere
53                         ALLOW       Anywhere
53 on wg0                  ALLOW       Anywhere
1346/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
22/tcp (v6)                DENY        Anywhere (v6)
51820/udp (v6)             ALLOW       Anywhere (v6)
OpenSSH (v6)               ALLOW       Anywhere (v6)
53 (v6)                    ALLOW       Anywhere (v6)
53 (v6) on wg0             ALLOW       Anywhere (v6)

127.0.0.1 5335             ALLOW OUT   Anywhere

Anywhere on eth0           ALLOW FWD   Anywhere on wg0
Anywhere (v6) on eth0      ALLOW FWD   Anywhere (v6) on wg0
```

## SSH
Je ne vais pas expliquer comment créer une passkey vu que cela est en dehors du context de ce projet, mais les fichiers de configuration que j'utilises sont dans le conf file du repo git. Utilisez ceux-ci si vous en avez de besoin.

## UNBOUND
### Install
```
sudo apt install unbound
```
### Config
```
sudo nano /etc/unbound/unbound.conf.d/pihole.conf
```
```
server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335

    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    root-hints: "/var/lib/unbound/root.hints"

    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no

    edns-buffer-size: 1232
    prefetch: yes

    num-threads: 2

    rrset-cache-size: 100m
    msg-cache-size: 50m

    cache-max-ttl: 86400
    cache-min-ttl: 3600

    hide-identity: yes
    hide-version: yes

```
```
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
sudo systemctl restart unbound
dig @127.0.0.1 -p 5335 google.com
```
La requette passe sans problème

## PIHOLE
### Intall
```
curl -sSL https://install.pi-hole.net | bash
```
Interface → ton interface réseau (ex: eth0)
DNS upstream → 127.0.0.1#5335 (Unbound)
Web UI → YES
### Config
http://192.168.1.169:80/admin/network

- PIHOLE DNS

- PIHOLE LISTS

## WIREGUARD
### Install
```
sudo apt install wireguard
wg genkey | tee privatekey | wg pubkey > publickey
```
Utiliser les valeurs de ces clés dans les prochaines étapes. Pour moi elles étaient dans mon ~
### Config
#### Serveur
```
sudo nano /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.6.0.1/24
ListenPort = 51820
PrivateKey = YOUR_PRIVATE_KEY_HERE

PostUp = ufw route allow in on wg0 out on eth0
PostUp = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = ufw route delete allow in on wg0 out on eth0
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Client
[Peer]
PublicKey = YOUR_PUBLIC_KEY_HERE
AllowedIPs = 10.6.0.2/32
```
```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```
#### Client
```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.6.0.2/24
DNS = 10.6.0.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = TON_IP_PUBLIC:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Adaptez pour vos informations.
### Fowarding
```
sudo nano /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```
## TELUS 
- Désactiver l'ipv6. (cause des problèmes avec de la confusion de dns)
image ipv6

- Changer le dns local
image dns

- Ouvrir le port WireGuard
image wireguard
