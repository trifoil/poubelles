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
$privatekey > /etc/wireguard/server.key
echo "The privatekey is $privatekey"
wg pubkey < /etc/wireguard/server.key > /etc/wireguard/server.pub
cat /etc/wireguard/server.pub 
pubkey=$(cat /etc/wireguard/server.pub)
echo "The pubkey is $pubkey"


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

```
#!/bin/bash

# Ensure the script is run as root
if [ "$(id -u)" -ne 0 ]; then
  echo "This script must be run as root"
  exit 1
fi

# Install WireGuard tools if not installed
dnf install -y wireguard-tools

# Step 1: Generate client keys
echo "Generating client keys..."
umask 077
wg genkey > /etc/wireguard/client.key
wg pubkey < /etc/wireguard/client.key > /etc/wireguard/client.pub

# Output the client private key to the console (for client to use)
echo "Client private key:"
cat /etc/wireguard/client.key

# Step 2: Generate server keys
echo "Generating server keys..."
umask 077
wg genkey > /etc/wireguard/server.key
wg pubkey < /etc/wireguard/server.key > /etc/wireguard/server.pub

# Output the server public key to the console (for server configuration)
echo "Server public key:"
cat /etc/wireguard/server.pub

# Step 3: Configure server WireGuard
echo "Configuring server WireGuard..."

# Create the configuration directory if it doesn't exist
mkdir -p /etc/wireguard

# Copy the server private key and create wg0.conf
cp /etc/wireguard/server.key /etc/wireguard/wg0.conf

# Edit the configuration file using a here document
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
Address = 172.16.0.1/24
ListenPort = 51820
PrivateKey = $(cat /etc/wireguard/server.key)

# Firewall rules
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client #1 details
PublicKey = (client's public key goes here)

# Traffic to route to this client
AllowedIPs = 172.16.0.2/32
EOF

echo "Server WireGuard configuration has been created."

# Step 4: Instructions for client setup
echo "To complete the setup, add the client's public key to the server config in /etc/wireguard/wg0.conf under the [Peer] section."
echo "The client's public key is located in /etc/wireguard/client.pub"

```