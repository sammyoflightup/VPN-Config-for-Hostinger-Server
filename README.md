# VPN-Config-for-Hostinger-Server
# OpenVPN Server Setup on Hostinger Ubuntu VPS

This guide will walk you through the process of setting up an OpenVPN server on your Hostinger Ubuntu VPS for secure remote access.

## Prerequisites

- A Hostinger Ubuntu VPS
- Root access or sudo privileges
- Basic understanding of the Linux command line

## Steps

### 1. Update the Server

Start by updating your server's packages:

```bash
sudo apt update && sudo apt upgrade -y
2. Install OpenVPN and Easy-RSA
Next, install OpenVPN and Easy-RSA for certificate management:

bash
Copy
Edit
sudo apt install openvpn easy-rsa -y
3. Set Up the Certificate Authority (CA)
Create a directory for Easy-RSA and set up the Certificate Authority (CA):

bash
Copy
Edit
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
Copy Easy-RSA files into the current directory:

bash
Copy
Edit
cp -r /usr/share/easy-rsa/* .
Edit the vars file to configure the details for your certificate authority:

bash
Copy
Edit
nano ~/openvpn-ca/vars
Modify the variables in vars to reflect your organization's details:

bash
Copy
Edit
export KEY_COUNTRY="US"
export KEY_PROVINCE="California"
export KEY_CITY="SanFrancisco"
export KEY_ORG="MyCompany"
export KEY_EMAIL="admin@mycompany.com"
export KEY_OU="MyOrgUnit"
4. Build the Certificate Authority (CA)
Source the vars file and clean any previous settings:

bash
Copy
Edit
source ./vars
./clean-all
Now, build the Certificate Authority:

bash
Copy
Edit
./build-ca
5. Generate the Server and Client Certificates
Generate the server certificate:

bash
Copy
Edit
./build-key-server server
Generate the Diffie-Hellman parameters for better security:

bash
Copy
Edit
./build-dh
Generate the HMAC key:

bash
Copy
Edit
openvpn --genkey --secret keys/ta.key
For each client, generate a certificate:

bash
Copy
Edit
./build-key client1
6. Configure OpenVPN Server
Copy the necessary certificates and keys to OpenVPN's configuration directory:

bash
Copy
Edit
sudo cp ~/openvpn-ca/keys/{server.crt,server.key,ca.crt,dh.pem,ta.key} /etc/openvpn
Next, copy the sample OpenVPN server configuration file:

bash
Copy
Edit
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gunzip /etc/openvpn/server.conf.gz
Edit the OpenVPN server configuration file:

bash
Copy
Edit
sudo nano /etc/openvpn/server.conf
Make sure the following lines are uncommented and configured:

bash
Copy
Edit
tls-auth ta.key 0
cipher AES-256-CBC
auth SHA256
user nobody
group nogroup
ca ca.crt
cert server.crt
key server.key
dh dh.pem
7. Enable IP Forwarding
Allow the server to forward IP packets, which is required for routing traffic from the VPN:

bash
Copy
Edit
sudo nano /etc/sysctl.conf
Uncomment the following line to enable IP forwarding:

bash
Copy
Edit
net.ipv4.ip_forward=1
Apply the changes:

bash
Copy
Edit
sudo sysctl -p
8. Set Up Firewall Rules
Allow VPN traffic through the server’s firewall:

bash
Copy
Edit
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
sudo ufw enable
9. Start the OpenVPN Service
Start the OpenVPN service:

bash
Copy
Edit
sudo systemctl start openvpn@server
Enable OpenVPN to start automatically on boot:

bash
Copy
Edit
sudo systemctl enable openvpn@server
10. Create Client Configuration
Create a directory to store the client configurations:

bash
Copy
Edit
mkdir -p ~/client-configs/keys
cp ~/openvpn-ca/keys/{client1.crt,client1.key,ca.crt,ta.key} ~/client-configs/keys
Create a base configuration file for clients:

bash
Copy
Edit
nano ~/client-configs/base.conf
Add the following content to the base.conf file:

bash
Copy
Edit
client
dev tun
proto udp
remote YOUR_SERVER_IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
auth SHA256
key-direction 1
tls-auth ta.key 1
ca ca.crt
cert client1.crt
key client1.key
Replace YOUR_SERVER_IP with your VPS’s public IP address.

11. Bundle Client Configuration
Now bundle the client configuration into a zip file for distribution:

bash
Copy
Edit
cd ~/client-configs
zip -r client1.zip client-configs/keys client-configs/base.conf
Send the client1.zip file to your client so they can configure their VPN client.

12. Testing and Troubleshooting
If there are any issues, you can view OpenVPN logs for troubleshooting:

bash
sudo journalctl -u openvpn@server
Optional: Configure Automatic Restart
To ensure that OpenVPN restarts after a server reboot, add the service to system startup:

bash
sudo systemctl enable openvpn@server
For more details on OpenVPN setup, visit OpenVPN's Official Documentation.

vbnet
### Key Features:
- **Step-by-step guide** for setting up OpenVPN on an Ubuntu VPS from Hostinger.
- Clear instructions for **building CA**, generating **server and client certificates**, and configuring OpenVPN.
- A bundled **client configuration file** to distribute to users for secure VPN access.
- A troubleshooting section to check OpenVPN logs.

This markdown snippet is perfect for sharing on GitHub and guides users through each step with clear and organized instructions.







