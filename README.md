# pmacct

[pmacct](http://www.pmacct.net) RPM with nDPI support for CentOS 7.

This is work is based on RPM from [Lux Repository](https://www.iotti.biz/?page_id=126).

# How to build

The build must be done on a NethServer 7.

First, compile and install nDPI 3.4 because pmacct requires nDPI devel headers.
```
yum install gcc-c++ automake autoconf  libpcap-devel libtool numactl-devel jason-c-devel -y
wget https://github.com/ntop/nDPI/archive/refs/tags/3.4.tar.gz
tar xvzf 3.4.tar.gz
cd nDPI-3.4/
./autogen.sh && ./configure && make && make install
```

Move to root directory, then prepare the evironment to build rpms:
```
yum install podman git
mkdir ~/bin
curl https://raw.githubusercontent.com/NethServer/nethserver-makerpms/master/install.sh | bash
```

Build pmacct:
```
export PATH=$PATH:~/bin
git clone https://github.com/gsanchietti/pmacct.git
cd pmacct
YUM_ARGS="--enablerepo=nethserver-testing" ~/bin/makerpms pmacct.spec
```
