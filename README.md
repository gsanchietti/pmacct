# pmacct

[pmacct](http://www.pmacct.net) RPM with nDPI support for CentOS 7.

This is work is based on RPM from [Lux Repository](https://www.iotti.biz/?page_id=126).

## How to build

### nDPI 3.4

The build must be done on a NethServer 7.

First, compile `ndpi-dev` rRPM because pmacct requires nDPI devel headers.

```
yum install gcc-c++ automake autoconf  libpcap-devel libtool numactl-devel jason-c-devel -y
wget https://github.com/ntop/nDPI/archive/refs/tags/3.4.tar.gz
tar xvzf 3.4.tar.gz
ln -s nDPI-3.4/ nDPI
cd nDPI-3.4/
./autogen.sh && ./configure && make && make install
cd packages/rpm
./configure && make
```

### pmacct

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

## Install

You need to force install to avoid dependency errors:
```
rpm -Uvh --nodeps pmacct-*.rpm
```

## Usage

Create this sqlite file, eg `schema.sql`:
```
CREATE TABLE acct_v5 (
    agent_id INT(8) NOT NULL DEFAULT 0,
    class CHAR(16) NOT NULL DEFAULT ' ',
    class_id CHAR(16) NOT NULL DEFAULT ' ',
    mac_src CHAR(17) NOT NULL DEFAULT '0:0:0:0:0:0',
    mac_dst CHAR(17) NOT NULL DEFAULT '0:0:0:0:0:0',
    vlan INT(4) NOT NULL DEFAULT 0,
    ip_src CHAR(45) NOT NULL DEFAULT '0.0.0.0',
    ip_dst CHAR(45) NOT NULL DEFAULT '0.0.0.0',
    src_port INT(4) NOT NULL DEFAULT 0,
    dst_port INT(4) NOT NULL DEFAULT 0,
    ip_proto CHAR(6) NOT NULL DEFAULT 0, 
    tos INT(4) NOT NULL DEFAULT 0, 
    packets INT NOT NULL,
    bytes BIGINT NOT NULL,
    flows INT NOT NULL DEFAULT 0,
    stamp_inserted DATETIME NOT NULL DEFAULT '0000-00-00 00:00:00',
    stamp_updated DATETIME,
    PRIMARY KEY (agent_id, class, class_id, mac_src, mac_dst, vlan, ip_src, ip_dst, src_port, dst_port, ip_proto, tos, stamp_inserted)
);
```

Then load it:
```
sqlite3 /etc/pmacct/pmacct.db < schema.sql
```

Example of `pmacctd.conf`:
```
! debug: true
daemonize: false
!
! libpcap daemon configs
!
pcap_interface: eth0
!
! Plugins definitions
!
plugins: sqlite3[foo],  memory[all]
!
! 'foo' plugin configuration
!
aggregate[foo]: src_host, dst_host, src_port, dst_port, proto, tos, class
sql_db[foo]: /etc/pmacct/pmacct.db
sql_table_name[foo]: acct
sql_table_version[foo]: 5 
! sql_table_version[foo]: 2 
! sql_table_version[foo]: 3 
sql_refresh_time[foo]: 60
sql_history[foo]: 1m 
sql_history_roundoff[foo]: m

aggregate[all]: dst_host, src_host
!imt_path[all]: /tmp/pipe.memory
```

Execute the daemon:
```
cd /etc/pmacct
pmacctd -f pmacctd.conf
```
