# poubelles
shitpo

## Client

```

```

## Server

```
su
apt install wireguard -y
sudo sed -i 's/^#net\.ipv4\.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.d/99-sysctl.conf
grep '^net.ipv4.ip_forward=1' /etc/sysctl.d/99-sysctl.conf
systemctl reboot
```

```
su
umask 077 
privatekey=$(wg genkey)
cat <<EOF > /etc/wireguard/server.key
    $privatekey
EOF
echo "The privatekey is $privatekey"
pubkey=$(cat /etc/wireguard/server.key | wg pubkey)
echo "The pubkey is $pubkey"
cat <<EOF > /etc/wireguard/server.pub
    $pubkey
EOF

cat <<EOF > /etc/wireguard/wg0.conf 
[Interface]
Address = 192.168.1.55/24
ListenPort = 51820
PrivateKey = $privatekey
# Firewall rules
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client #1 details
PublicKey = clientkey
# Traffic to route to this client
AllowedIPs = 172.16.0.2/32
EOF

systemctl enable --now wg-quick@wg0.service
```
