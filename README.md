# ipfs
#Description

Th InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices. [Wikipedia](https://en.wikipedia.org/wiki/InterPlanetary_File_System)

* Install IPFS on CLI linux

* Download file from [dist.ipfs.io](https://dist.ipfs.io/) for Raspberry Pi
```
wget https://dist.ipfs.io/go-ipfs/v0.7.0/go-ipfs_v0.7.0_linux-arm64.tar.gz
```
* Attach file for tar.gz
```
tar -xvzf go-ipfs_v0.7.0_linux-arm64.tar.gz
cd go-ipfs/
sudo ./install
```
* result: Moved ipfs to /usr/local/bin
* Initialize ipfs:
```
ipfs init
```
* Start ipfs service
```
ipfs daemon
```
* Add file to IPFS server
```
ipfs add "directory of file
```
* Add folder to IPFS
```
ipfs add -r "directory of folder"
```
* Print out the content from IPFS
```
ipfs cat <CID>
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
* Apply  netplan
```
sudo netplan apply
```

* You need to assign static IP address **networking.service**
```
sudo nano /etc/network/interfaces
```
```
auto lo
iface lo inet loopback

auto wls2
iface wls2 inet dhcp

auto eno1
iface eno1 inet static
address 192.168.10.1
netmask 255.255.255.0
dns-nameservers 192.168.10.1 8.8.8.8
```
 * Restart **service networing**
```
sudo systemctl restart networking.service
```
* You will need DHCP server ```isc-dhcp-server```
```
$sudo apt update
$sudo apt install isc-dhcp-server
```
* You will probably need to change the default configuration by editing ```/etc/dhcp/dhcpd.conf``` to suit your needs and particular configuration.

```
# minimal sample /etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;
 
subnet 192.168.10.0 netmask 255.255.255.0 {
range 192.168.10.150 192.168.10.200;
option routers 192.168.10.1;
option domain-name-servers 192.168.10.1, 8.8.8.8;
option domain-name "ipfs.kmp";
}

```
```
$sudo apt update
$sudo apt install bind9
```
Go to edit file ```/etc/bind/named.conf.options```
```
forwarders {
	192.168.10.1;
	8.8.8.8;
};
```
```
sudo systemctl restart bind9.service
```
Add configure on file ```named.conf.local``` to link.
```
sudo nano /etc/bind/named.conf.local:
```
```
zone "ipfs.kmp" {
	type master;
	file "/etc/bind/db.ipfs.kmp";
};
zone "1.168.192.in-addr.arpa" {
	type master;
	file "/etc/bind/db.192";
};
```

```
sudo cp /etc/bind/db.local /etc/bind/db.ipfs.kmp
```

```
;
; BIND data file for ipfs.kmp
;
$TTL 604800
@ IN SOA ipfs.kmp. root.ipfs.kmp. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
@ IN NS ns.ipfs.kmp.
@ IN A 192.168.10.1
@ IN AAAA ::1
ns IN A 192.168.10.1
```
```
sudo cp /etc/bind/db.127 /etc/bind/db.192
```
```
;
; BIND reverse data file for local 192.168.10.XXX net
;
$TTL 604800
@ IN SOA ns.ipfs.kmp. root.ipfs.kmp. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
;
@ IN NS ns.
1 IN PTR ns.ipfs.kmp.
```
```
sudo systemctl restart bind9.service
```
**Troubleshooting**
```
dig -x 192.168.10.1
dig ipfs.kmp
```