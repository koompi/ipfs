# ipfs

#Description

Th InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices. Wikipedia

  

#Install IPFS on CLI linux

* Download file from dist.ipfs.io for Raspberry Pi

```
wget https://dist.ipfs.io/go-ipfs/v0.7.0/go-ipfs_v0.7.0_linux-arm64.tar.gz
```
  

* Attach file for tar.gz
```
tar -xvzf go-ipfs_v0.7.0_linux-arm64.tar.gz

cd go-ipfs/

sudo ./install
```

 result: Moved ipfs to /usr/local/bin

  

Initialize ipfs:
```
ipfs init
```
Start ipfs service
```
ipfs daemon
```
  

Add file to IPFS server

```
ipfs add  "directory of file
```
  
Add folder to IPFS
```
ipfs add -r  "directory of folder"
```
  

Print out the content from IPFS

```
ipfs cat  "Hash"
```
  
#DNSlink IPFS

You need to assign static IP address with netplan
```
    sudo nano /etc/netplan/50-cloud-init.yaml
```

```
network:
    ethernets:
    wls2:
         dhcp4: true
    eno1:
         dhcp4: no
         addresses: [192.168.10.1/24]
         nameservers:
              addresses: [192.168.10.1, 8.8.8.8]
    version: 2
```
```
sudo netplan apply
```

```$sudo apt update 
   $sudo apt install isc-dhcp-server
```
You will probably need to change the default configuration by editing ```/etc/dhcp/dhcpd.conf``` to suit your needs and particular configuration.

```
# minimal sample /etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.10.0 netmask 255.255.255.0 {
 range 192.168.10.150 192.168.10.200;
 option routers 192.168.10.1;
 option domain-name-servers 192.168.10.1, 8.8.8.8;
 option domain-name "mydomain.example";
}
```