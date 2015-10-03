# masterdns
Step by Step how to setup a DNS Server in CentOS Using Bind

What is DNS Server ?

DNS = Domain Naming Service (or) Domain Name System DNS will resolve the host name for the particular IP address.

Here Im Using CentOS Server to Setup the DNS Server using BIND

Primary DNS Server (or) Master DNS Server:

IP Address :    192.168.0.100
Hostname   :    masterdns.example.com
Secondary DNS Server (or) Slave DNS Server:
IP Address :    192.168.0.101
Hostname   :    slavedns.example.com
Nodes Machines :
IP Address :    192.168.1.1XX  ## Hostname : node1.example.com
IP Address :    192.168.1.1XX  ## Hostname : node2.example.com
IP Address :    192.168.1.1XX  ## Hostname : node3.example.com
IP Address :    192.168.1.1XX  ## Hostname : node4.example.com
Primary DNS Server (or) Master DNS Server :
[root@masterdns ~]# yum install bind* -y
Then Edit the Configuration of name server
[root@masterdns ~]# vim /etc/named.conf 

//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
options {
    listen-on port 53 { 127.0.0.1; 192.168.0.100; }; # Master DNS Servers IP
    listen-on-v6 port 53 { none; }; # Change ::1; to none;
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { localhost; 192.168.1.0/24; }; # IP Range of Hosts
    allow-transfer  { localhost; 192.168.0.101; }; # Slave DNS Servers IP
    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;
    dnssec-lookaside auto;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
    type hint;
    file "named.ca";
};
zone"example.com" IN {
type master;
file "forward.example";
allow-update { none; };
};
zone"0.168.192.in-addr.arpa" IN {
type master;
file "reverse.example";
allow-update { none; };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
Save and Exit the named.conf using wq!

Creat the Forward and Reserve Zone files as mentioned in named.conf
FORWARD ZONE :

a.) Create a Forward Zone file under /var/named in the name of forward.linuxzadmin

There are Sample files under the /var/named/ Directory, Just make a Copy of that file and modify it as our need

b.) Make a Copy of sample file as below

[root@masterdns ~]# cp /var/named/named.localhost /var/named/forward.example
c.) Edit the file forward.example

[root@masterdns ~]# vim /var/named/forward.example


$TTL 86400
@       IN SOA  masterdns.example.com. root.example.com. (
                                2014051001      ; serial
                                        3600    ; refresh
                                        1800    ; retry
                                        604800  ; expire
                                        86400   ; minimum
)
@               IN      NS      masterdns.example.com.
@               IN      NS      slavedns.example.com.
@               IN      A       192.168.1.100
@               IN      A       192.168.1.101
@               IN      A       192.168.1.1XX
@               IN      A       192.168.1.1XX
@               IN      A       192.168.1.1XX
@               IN      A       192.168.1.1XX
masterdns       IN      A       192.168.1.1XX
slavedns        IN      A       192.168.1.1XX
node1           IN      A       192.168.1.1XX
node2           IN      A       192.168.1.1XX
node3           IN      A       192.168.1.1XX
node4           IN      A       192.168.1.1XX
RESERVE ZONE:

a.) Create a Reserver Zone file under /var/named in the name of reverse.example

There are Sample files under the /var/named/ Directory, Just make a Copy of that file and modify it as our need

b.) Make a Copy of sample file as below

[root@masterdns ~]# cp /var/named/named.loopback /var/named/reverse.example

c.) Edit the file reverse.example

[root@masterdns ~]# vim /var/named/reverse.example 


$TTL 86400
@       IN SOA  masterdns.example.com. root.example.com. (
                                2015100301      ; serial
                                        3600    ; refresh
                                        1800    ; retry
                                        604800  ; expire
                                        86400   ; minimum
)
@               IN      NS      masterdns.linuxzadmin.local.
@               IN      NS      slavedns.linuxzadmin.local.
@               IN      PTR     linuxzadmin.local.
masterdns       IN      A       192.168.1.100
slavedns        IN      A       192.168.1.101
node1           IN      A       192.168.1.1XX
node2           IN      A       192.168.1.1XX
node3           IN      A       192.168.1.1XX
node4           IN      A       192.168.1.1XX
200             IN      PTR     masterdns.example.com.
201             IN      PTR     slavedns.example.com.
205             IN      PTR     node1.example.com.
206             IN      PTR     node2.example.com.
207             IN      PTR     node3.example.com.
208             IN      PTR     node4.example.com.
The files we created was in root group We need to change those files to named group
Here we can see the files which have the root group

a.) List the files and see the permissions and group of those created zone files

[root@masterdns ~]# ls -l /var/named/
total 40
drwxr-x---. 6 root  named 4096 May 10 19:33 chroot
drwxrwx---. 2 named named 4096 Nov 16  2011 data
drwxrwx---. 2 named named 4096 Nov 16  2011 dynamic
-rw-r-----. 1 root  root   550 May 10 20:19 forward.linuxzadmin
-rw-r-----. 1 root  named 1892 Feb 18  2008 named.ca
-rw-r-----. 1 root  named  152 Dec 15  2009 named.empty
-rw-r-----. 1 root  named  152 Jun 21  2007 named.localhost
-rw-r-----. 1 root  named  168 Dec 15  2009 named.loopback
-rw-r-----. 1 root  root   676 May 10 20:35 reverse.linuxzadmin
drwxrwx---. 2 named named 4096 Nov 16  2011 slaves
b.) Change the group to named using below Command

[root@masterdns ~]# chgrp named /var/named/forward.example 
[root@masterdns ~]# chgrp named /var/named/reverse.example 
Here we can see the Output now which changed to named group

[root@masterdns ~]# ls -l /var/named/
total 40
drwxr-x---. 6 root  named 4096 May 10 19:33 chroot
drwxrwx---. 2 named named 4096 Nov 16  2011 data
drwxrwx---. 2 named named 4096 Nov 16  2011 dynamic
-rw-r-----. 1 root  named  550 May 10 20:19 forward.linuxzadmin
-rw-r-----. 1 root  named 1892 Feb 18  2008 named.ca
-rw-r-----. 1 root  named  152 Dec 15  2009 named.empty
-rw-r-----. 1 root  named  152 Jun 21  2007 named.localhost
-rw-r-----. 1 root  named  168 Dec 15  2009 named.loopback
-rw-r-----. 1 root  named  676 May 10 20:35 reverse.linuxzadmin
drwxrwx---. 2 named named 4096 Nov 16  2011 slaves
c.) Then we need to check the Context of the files under

[root@masterdns ~]# ls -lZd /etc/named.conf 
-rw-r-----. root named system_u:object_r:named_conf_t:s0 /etc/named.conf

/etc/named.conf
/var/named/forward.example
/var/named/reverse.example
It want to be in the context of named_conf_t

If its Different than this then we need to restore the context using

# restorecon /etc/named.conf 
Now we need to Check for the Error in the conf file and Zone file
[root@masterdns ~]# named-checkconf /etc/named.conf 

[root@masterdns ~]# named-checkzone example.com /var/named/forward.example 
zone example.com/IN: loaded serial 2014051001
OK

[root@masterdns ~]# named-checkzone 1.168.192.in-addr.arpa /var/named/reverse.example 
zone 0.168.192.in-addr.arpa/IN: loaded serial 2015100301
OK
Start the DNS Service
[root@masterdns ~]# service named restart
Stopping named:                                            [  OK  ]
Starting named:                                            [  OK  ]
Make the named Service in runlevels
[root@masterdns ~]# chkconfig named on

[root@masterdns ~]# chkconfig --list named
named           0:off   1:off   2:on    3:on    4:on    5:on    6:off
Deploy iptables Rules to allow DNS service
Add the iptables rules

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 53 -j ACCEPT
iptables -A INPUT -p udp -m state --state NEW -m udp --dport 53 -j ACCEPT
iptables -A INPUT -j DROP
Save the iptables Using

[root@masterdns ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
Restart the iptables Service Using

[root@masterdns ~]# service iptables restart
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
Make it to run in multi run levels

[root@masterdns ~]# chkconfig iptables on

[root@masterdns ~]# chkconfig --list iptables 
iptables        0:off   1:off   2:on    3:on    4:on    5:on    6:off
Check the DNS server using Dig Command
[root@masterdns ~]# dig masterdns.example.local

; <<>> DiG 9.7.3-P3-RedHat-9.7.3-8.P3.el6 <<>> masterdns.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41316
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;masterdns.example.com.   IN  A

;; ANSWER SECTION:
masterdns.example.com. 86400 IN   A   192.168.1.100

;; AUTHORITY SECTION:
linuxzadmin.local.  86400   IN  NS  masterdns.example.com.
linuxzadmin.local.  86400   IN  NS  slavedns.example.com.

;; ADDITIONAL SECTION:
slavedns.example.com. 86400 IN    A   192.168.1.101

;; Query time: 0 msec
;; SERVER: 192.168.1.100#53(192.168.1.100)
;; WHEN: Sat Oct 10 11:07:10 2015
;; MSG SIZE  rcvd: 114
Check for the Available Hosts in DNS
[root@masterdns ~]# nslookup example.com
Server:     192.168.1.100
Address:    192.168.1.100#53

Name:   example.com
Address: 192.168.1.108
Name:   example.com
Address: 192.168.1.108
Name:   example.com
Address: 192.168.1.100
Name:   example.com
Address: 192.168.1.101
Name:   example.com
Address: 192.168.1.XXX
Name:    example.com
Address: 192.168.1.XXX
Now we Need to Setup the Slave DNS server

Secondary DNS server (or) Slave DNS Server

Host Deployed with RHEL Server
[root@slavedns ~]# lsb_release -a
LSB Version:    :core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID: RedHatEnterpriseServer
Description:    Red Hat Enterprise Linux Server release 6.2 (Santiago)
Release:    6.2
Codename:   Santiago
Insatall the BIND package in Server
[root@slavedns ~]# yum install bind* -y
Edit the named.conf to add the configuration
[root@slavedns ~]# vim /etc/named.conf 


//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
    listen-on port 53 { 127.0.0.1; 192.168.0.201; }; # Slave DNS server's IP
    listen-on-v6 port 53 { none; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { localhost; 192.168.0.0/24; };
    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;
    dnssec-lookaside auto;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
    type hint;
    file "named.ca";
};
zone"example.com" IN {
type slave;
file "slaves/example.fwd";
masters { 192.168.1.100; };
};
zone"0.168.192.in-addr.arpa" IN {
type slave;
file "slaves/example.rev";
masters { 192.168.1.100; };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
Start the named Service and make it to Run in Multi Runlevels
[root@slavedns ~]# service named start 
Starting named:                                            [  OK  ]

[root@slavedns ~]# chkconfig named on

[root@slavedns ~]# chkconfig --list named
named           0:off   1:off   2:on    3:on    4:on    5:on    6:off
We Don't need to Create the Zone file here, If will be resolved from Master Server While we Start the Named Service
[root@slavedns ~]# ls -l /var/named/slaves/
total 8
-rw-r--r--. 1 named named 634 May 10 23:35 linuxzadmin.fwd
-rw-r--r--. 1 named named 773 May 10 23:35 linuxzadmin.rev
Here we can Check the Both File's
[root@slavedns ~]# cat /var/named/slaves/linuxzadmin.fwd 
$ORIGIN .
$TTL 86400  ; 1 day
linuxzadmin.local   IN SOA  masterdns.linuxzadmin.local. root.linuxzadmin.local. (
                2014051001 ; serial
                3600       ; refresh (1 hour)
                1800       ; retry (30 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
            NS  slavedns.linuxzadmin.local.
            NS  masterdns.linuxzadmin.local.
            A   192.168.0.200
            A   192.168.0.201
            A   192.168.0.205
            A   192.168.0.206
            A   192.168.0.207
            A   192.168.0.208
$ORIGIN linuxzadmin.local.
masterdns       A   192.168.0.200
node1           A   192.168.0.205
node2           A   192.168.0.206
node3           A   192.168.0.207
node4           A   192.168.0.208
slavedns        A   192.168.0.201
This is the Out put of linuxzadmin.rev

[root@slavedns ~]# cat /var/named/slaves/linuxzadmin.rev 
$ORIGIN .
$TTL 86400  ; 1 day
0.168.192.in-addr.arpa  IN SOA  masterdns.linuxzadmin.local. root.linuxzadmin.local. (
                2014051001 ; serial
                3600       ; refresh (1 hour)
                1800       ; retry (30 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
            NS  slavedns.linuxzadmin.local.
            NS  masterdns.linuxzadmin.local.
            PTR linuxzadmin.local.
$ORIGIN 0.168.192.in-addr.arpa.
100         PTR masterdns.linuxzadmin.local.
101         PTR slavedns.linuxzadmin.local.
1XX         PTR node1.linuxzadmin.local.
1XX         PTR node3.linuxzadmin.local.
1XX         PTR node4.linuxzadmin.local.
masterdns       A   192.168.1.200
node1           A   192.168.1.XXX
node2           A   192.168.1.XXX
node3           A   192.168.1.XXX
node4           A   192.168.1.XXX
slavedns        A   192.168.1.XXX
Check the DNS Server using dig from Slave Server
[root@slavedns ~]# dig masterdns.linuxzadmin.local

; <<>> DiG 9.7.3-P3-RedHat-9.7.3-8.P3.el6 <<>> masterdns.linuxzadmin.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11178
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;masterdns.example.com.   IN  A

;; ANSWER SECTION:
masterdns.example.com. 86400 IN   A   192.168.0.200

;; AUTHORITY SECTION:
example.com.     86400   IN  NS  masterdns.example.com.
example.com.     86400   IN  NS  slavedns.example.com.

;; ADDITIONAL SECTION:
slavedns.example.com. 86400 IN    A   192.168.1.101

;; Query time: 2 msec
;; SERVER: 192.168.0.200#53(192.168.0.200)
;; WHEN: Sat May 10 23:42:03 2014
;; MSG SIZE  rcvd: 114
Client Side :

Now we Need to Assign the Name Server for the Node's in our network to get assigned a host name from DNS server.
Use the Setup Command and assign the Primary and Secondary DNS server's We Don't need to Assing the hostname

a.) Just Assign the IP, Subnet, Gateway, PDNS, SDNS

b.) Restart the Network and Check the hostname

c.) Here i have not changed the hostname

[root@node1 ~]# cat /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=localhost.localdomain
d.) Here we can see the hostname Assigned from the DNS server

[root@node1 ~]# hostname
node1.linuxzadmin.local
e.) If we need to check the DNS just do a Dig

[root@node1 ~]# dig masterdns.example.com

; <<>> DiG 9.7.3-P3-RedHat-9.7.3-8.P3.el6 <<>> masterdns.example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51788
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;masterdns.linuxzadmin.local.   IN  A

;; ANSWER SECTION:
masterdns.example.com. 86400 IN   A   192.168.1.100

;; AUTHORITY SECTION:
example.com.  86400   IN  NS  slavedns.example.com.
example.com.  86400   IN  NS  masterdns.example.com.

;; ADDITIONAL SECTION:
slavedns.example.com.  86400 IN    A   192.168.0.201

;; Query time: 1 msec
;; SERVER: 192.168.1.100#53(192.168.1.100)
;; WHEN: Sat Oct 03 11:58:32 2014
;; MSG SIZE  rcvd: 114
If we need to flush the DNS Server Caches Use Below Command
# yum install nscd
# nscd -i hosts
All done 
