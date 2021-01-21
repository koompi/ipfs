# ipfs

#Description
The InterPlanetary File System is a protocol and peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices. Wikipedia

#Install IPFS on CLI linux
* Download file from dist.ipfs.io for Raspberry Pi
'''wget https://dist.ipfs.io/go-ipfs/v0.7.0/go-ipfs_v0.7.0_linux-arm64.tar.gz'''
* Attach file for tar.gz
'''tar -xvzf go-ipfs_v0.7.0_linux-arm64.tar.gz
cd go-ipfs/
sudo ./install'''
** result: '''Moved ipfs to /usr/local/bin'''

Initialize ipfs:
'''ipfs init'''