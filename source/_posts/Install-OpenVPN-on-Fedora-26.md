---
title: Install OpenVPN on Fedora 26
date: 2017-08-02 10:48:13
tags: [OpenVPN, linux, Fedora]
---


There are serveral tutorials in the internet ([this](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04) and [this](https://fedoraproject.org/wiki/Openvpn#Setting_up_an_OpenVPN_server)). Which describes 
installation process OpenVPN on linux.

They both not so applicable for Fedora 26 because from moment they was written it has been several years and there are many discrepancies. 

And when I ended them up and faced several issues:
  
 - Why I can ping but cannot access internet/local network behind the vpn server?
 - Where keys should be placed? 
 - how to use Easy-RSA v3 instead v2?
 - how to omit password on service start up?

------
1) First of all install necessary dependencies

```bash
sudo dnf install openvpn easy-rsa
```

2) Copy rsa scripts to the home folder

```bash
cp -ai /usr/share/easy-rsa/3/* ~/openvpn-ca
cd ~/rsa
```


3) According to [this](https://community.openvpn.net/openvpn/wiki/EasyRSA3-OpenVPN-Howto) start a new PKI and build a CA keypair/cert

```
./easyrsa init-pki
./easyrsa build-ca nopass
```

4) Build Server certificate, key

```
./easyrsa build-server-full server nopass
```

5) Build Client certificate, key

```
./easyrsa build-client-full client1 nopass
```
_you can omitt `nopass` on steps 3,4,5 if you need to_


6) Generate a strong Diffie-Hellman keys

```
./easyrsa gen-dh
```

7) Generate HMAC signature to strengthen the server's TLS integrity verification capabilities

```
openvpn --genkey --secret pki/ta.key
```

8) Before openvpn server setting up we need to put appropriate keys `ca.crt ca.key server.crt server.key ta.key dh.pem` into `/etc/openvpn/server/keys` folder


```bash
sudo cp ~/openvpn-ca/pki/issued/server.crt /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/private/server.key /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/private/ca.key /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/ca.crt /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/dh.pem /etc/openvpn/server
sudo cp ~/openvpn-ca/pki/ta.key /etc/openvpn/server
```

9) Now we need to set up the server itself, firstly copy configurations

```
sudo cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/server
```

10) Modify several lines in that configuration file

_add this lines at the end of the file:_
 
```
key-direction 0
auth SHA256
```
_remove `;` symbol to uncomment following lines_

```
user nobody
group nogroup
```

10')**[optional]** In order to to Redirect All Traffic Through the VPN, remove `;` from following lines

```
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

10'')**[optional]** Adjust port and protocol if you don't wish to use default 

```
port 443
proto tcp 
```

and if you have `server.crt` and `server.key` with the different name point to them here 

```
cert myservername.crt
key myservername.key
```

11) Allow IP Forwarding. This is fairly essential to the functionality we want our VPN server to provide.

```
sudo nano /etc/sysctl.conf
```

and drop a line there

```
net.ipv4.ip_forward=1
```

activate that:

```
sudo sysctl -p
```
    
12) Set up `firewalld` to work with `OpenVPN`

```
sudo firewall-cmd --permanent --add-service openvpn
sudo firewall-cmd --permanent --add-masquerade
```

13) Now we are going to set up our `systemd` service.

```
sudo ln -s /lib/systemd/system/openvpn-server\@.service /etc/systemd/system/multi-user.target.wants/openvpn-server\@server.service
                                                                                                                    ^^^^^^
```

#### NB! `server` corresponts with the configuration file name in /etc/openvpn/server such as server.conf. So if you have `myserver.conf` you have to replace `server` with `myserver`

14) Now we are ready to start `OpenVPN` service

```
sudo systemctl -f enable openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
```

Done! We successfully deployed our OpenVPN server, and we are ready to move on and set up the client

------
Client setup
------
As you remember we have already generated `client1.crt` and `client1.key` at the __step 5__. And now we need combine them with our general Certificates of Authority in order to build client config file.

1) First of all we need generate Client Configurations. Lets create `client-configs` directory and prepare with the keys

```bash
mkdir ~/client-configs
cd ~/client-configs
mkdir ~/keys
cp ~/openvpn-ca/pki/ca.crt ~/client-configs/keys
cp ~/openvpn-ca/pki/ta.key ~/client-configs/keys
cp ~/openvpn-ca/pki/private/client1.key ~/client-configs/keys
cp ~/openvpn-ca/pki/private/client1.crt ~/client-configs/keys
```
2) Next we need to copy base configuration from examples

```
cp /usr/share/doc/openvpn/sample/sample-config-files/client.conf ~/client-configs/base.conf
```

3) Open this file in your text editor
```
nano ~/client-configs/base.conf
```

4) and modify as following

```bash
remote server_IP_address 1194 #place your server address here
proto udp # update with specified protocol
```
next uncomment (by removing `;`)
```bash
user nobody
group nogroup
```
##### NB: If you are using CentOS, change the `group` from `nogroup` to `nobody` to match the distribution's available groups

and comment out the lines because we place them directly in client's config

```bash
#ca ca.crt
#cert client.crt
#key client.ke
```
add this lines at the end of the file
```
auth SHA256
key-direction 1
```

5) Next, we will create a simple script to compile our base configuration with the relevant certificate, key, and encryption files. This will place the generated configuration in the `~/client-configs/` files directory.

```
nano ~/client-configs/make_config.sh
```

```bash
BASE_CONFIG=~/client-configs/base.conf
KEY_DIR=~/client-configs/keys
OUTPUT_DIR=~/client-configs

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

make that file executable 

```
chmod 700 ~/client-configs/make_config.sh
```

6) Execute that file with `client1` input parameter

```
./make_config.sh client1
```

If everything went well, we should have a client1.ovpn file in our `~/client-configs/` directory

7) Now that file can be used on the client machine 

```
sudo dnf install openvpn
sudo openvpn --config client1.ovpn
```

## That's it!