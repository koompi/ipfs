# ipfs
#Description

Th InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices. [Wikipedia](https://en.wikipedia.org/wiki/InterPlanetary_File_System)

* Install IPFS on CLI linux

* Download file from [dist.ipfs.io](https://dist.ipfs.io/) for Raspberry Pi

	`$wget https://dist.ipfs.io/go-ipfs/v0.7.0/go-ipfs_v0.7.0_linux-arm64.tar.gz`
* Attach file for tar.gz
	```
	$tar -xvzf go-ipfs_v0.7.0_linux-arm64.tar.gz
	$cd go-ipfs/
	$sudo ./install
	```
* result: Moved ipfs to /usr/local/bin
- Initialize ipfs:
	`$ipfs init`
- Start ipfs service
	`$ipfs daemong`
## Adding the Service

**systemd**  is a software suite that comes with most newer Linux distributions, that allows the user to create and manage background services. These services are started automatically when the server boots, restarted if they fail, and have their output logs persisted to disk. Now that IPFS is installed, we want to create a service for it so that we get all these benefits.

To do this, we create a  _unit file_  at  `/etc/systemd/system/ipfs.service`  with the contents:

```
[Unit]
Description=IPFS Daemon

[Service]
ExecStart=/usr/local/bin/ipfs daemon 
User=root
Restart=always
LimitNOFILE=10240
[Install]
WantedBy=multi-user.target
```

Change the line "User=root" if you're not running the daemon as root, and then tell systemd about the new service:
	```
	$sudo systemctl daemon-reload
	$sudo systemctl enable ipfs
	$sudo systemctl start ipfs
	```
Notes on systemd

-   See high-level information on how the IPFS daemon is doing:  
    `$systemctl status ipfs`
-   Stop the daemon:  
    `$systemctl stop ipfs`
-   Start the daemon:  
    `$systemctl start ipfs`
-   See all logs from the daemon:  
    `$journalctl -u ipfs`
-   See only most recent logs, and show new logs as they're written:  
    `$journalctl -f -u ipfs`
* Add file to IPFS serve 
    `$ipfs add "directory of file`
* Add folder to IPFS
    `$ipfs add -r "directory of folder"`
* Print out the content from IPFS
	`$ipfs cat <CID>`
## DNSlink IPFS
You need to assign static IP address with netplan
	`$sudo nano /etc/netplan/50-cloud-init.yaml`
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
	`$sudo netplan apply`
* You need to assign static IP address **networking.service**
	`$sudo nano /etc/network/interfaces`
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
 * Restart ```service networing```
`
$sudo systemctl restart networking.service
`
* You will need DHCP server ```isc-dhcp-server```
`
$sudo apt update
$sudo apt install isc-dhcp-server
`
* You will probably need to change the default configuration by editing `/etc/dhcp/dhcpd.conf` to suit your needs and particular configuration.

	```
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
Go to edit file `/etc/bind/named.conf.options`
	```
		forwarders {
		192.168.10.1;
		8.8.8.8;
	};
	```
`$sudo systemctl restart bind9.service`
Add configure on file `named.conf.local` to link.
   `sudo nano /etc/bind/named.conf.local:`
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

	`$sudo cp /etc/bind/db.local /etc/bind/db.ipfs.kmp`

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
	@ 	IN 	NS 		ns.ipfs.kmp.
	@ 	IN 	A 		192.168.10.1
	@ 	IN 	AAAA 	::1
	ns 	IN	A	 	192.168.10.1
```
`$sudo cp /etc/bind/db.127 /etc/bind/db.192`
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
		@ 	IN	 NS		ns.
		1 	IN	 PTR	ns.ipfs.kmp.
```
`$sudo systemctl restart bind9.service`
- Add content to IPFS
`$ipfs add -r scr/`
	result: 
	```
	added QmSUWgkEeXTm5R4mSUbnA7xGJaWZWfzcUint9ZWwBGD223 scr/index.html
	added QmV4AtWqk3JakPR6578g9mXJxp6c5tMRAcdvvv8N6AQNJQ scr
	129 B / 129 B [================================================] 100.00%
	```
- Pin ipfs content to local storage
	`$ipfs pin add -r /ipfs/QmV4AtWqk3JakPR6578g9mXJxp6c5tMRAcdvvv8N6AQNJQ`
- Add IPFS content with DNSlink
	`$sudo nano /etc/bind/db.ipfs.kmp`
	```
	;
	; BIND data file for local loopback interface
	;
	$TTL    86400
	@       IN      SOA     ipfs.kmp. root.ipfs.kmp. (
	                              2         ; Serial
	                         604800         ; Refresh
	                          86400         ; Retry
	                        2419200         ; Expire
	                         604800 )       ; Negative Cache TTL
	;
	@       IN      NS      ipfs.kmp.
	@       IN      A       192.168.10.1
	ipfs.kmp.       IN      A       192.168.10.1
	ipfs.kmp.       IN      TXT     "dnslink=/ipfs/QmV4AtWqk3JakPR6578g9mXJxp6c5tMRAcdvvv8N6AQNJQ
	@       IN      AAAA    ::1
	ns      IN      A       192.168.10.1
	www     IN      CNAME   ipfs.kmp.
	```
	```
	$sudo systemctl restart bind9.service
	$sudo systemctl status bind9.service
	```
- Check results DNSlink on client node.
`$dig +noall +answer TXT ipfs.kmp`
**Troubleshooting**
	```
	$dig -x 192.168.10.1
	$dig ipfs.kmp
	$sudo rm -rf /etc/resolv.conf
	$ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
	```
