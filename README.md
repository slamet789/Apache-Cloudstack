# Apache-Cloudstack

## Pengertian ##
Apache CloudStack adalah aplikasi open source yang dirancang untuk membuat, mengelola dan mendeploy infrastruktur jaringan virtual machine. CloudStack digunakan oleh sejumlah penyedia layanan Public Cloud, Private Cloud, dan Hybrid Cloud. CloudStack menjadi solusi kebutuhan organisasi diantaranya ketersediaan compute orchestration, Network-as-a-Service, manajemen pengguna dan akun, native API, akuntansi sumber daya, dan UI tingkat satu.

Saat ini CloudStack mendukung beberapa hypervisors terkenal seperti VMware, KVM, Citrix XenServer, Xen Cloud Platform (XCP), Oracle VM server dan Microsoft Hyper-V. Pengguna dapat mengelola cloud mereka melalui beberapa cara yaitu web, command line tools, dan/atau RESTful API. Sebagai tambahan, CloudStack menyediakan API yang kompatibel dengan AWS EC2 dan S3 untuk organisasi yang ingin mendeploy hybrid cloud.

## Cloud Deployment Model ##
* Zone
Zone merupakan level tertinggi hirarki dari datacenter sebuah organisasi yang mengalokasikan untuk menyediakan isolasi fisik dan redudancy. Umumnya datacenter terdiri dari single zone, namun juga dapat terdiri dari multiple zone. Zone dapat diasosiasikan dengan domain, dapat berupa public atau private. Di dalam zone terdiri dari satu atau beberapa pod.
* Pod
Pod merupakan level kedua hirarki pada pengembangan CloudStack, pod sendiri merupakan partisi dari pod yang dibagi menjadi satu atau beberapa entitas. Pod dapat diasumsikan seperti rak fisik di dalam datacenter dimana seluruh host berada dalam satu subnet. Di dalam pod terdiri dari satu atau beberapa cluster di dalamnya, serta setiap cluster terdiri dari satu atau beberapa hosts.
* Cluster
Cluster merupakan level ketiga hirarki pada pengembangan CloudStack yang berada dalam Pod. Host atau kumpulan host bergabung membentuk grup secara logic yang disebut Cluster. Wujud cluster ini dapat berupa XenServer pool, VMware cluster yang belum dikonfigurasi di VCenter, atau sekumpulan KVM server.

## Cara Instalasi CloudStack ##
Pada dasarnya CloudStack deployment arsitektur terdiri dari dua bagian, yaitu Management Server dan CloudDB. CloudStack dapat melakukan deploy keduanya pada server yang sama maupun server berbeda. CloudStack deployment dapat dilakukan melalui dua cara yaitu :
* Single node installation
* Multinode installation

### Spek Minimum ###
* Komputer yang support hardware virtualization
* CentOS 6.8 x86_64 minimal install CD
* Alamat IP jaringan yang bukan DHCP
### Environment ###
* Komputer yang sudah terinstal OS Centos 6.8 x86_64
* Atur alamat IP nya, sebagai contoh
```
DEVICE=eth0
HWADDR=52:54:00:B9:A6:C0
NM_CONTROLLED=no
ONBOOT=yes
BOOTPROTO=none
IPADDR=172.16.10.2
NETMASK=255.255.255.0
GATEWAY=172.16.10.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```
Jalankan perintah berikut untuk merestart jaringan :
```
chkconfig network on
```
```
service network start
```
* Atur hostname
Coba ketikkan perintah berikut untuk mengetahui hostname :
```
hostname --fqdn
```
Keluarannya, secara default :
```
localhost
```
Edit file /etc/hosts sehingga seperti berikut :
```
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.10.2 srvr1.cloud.priv
```
Restart jaringan kembali :
```
service network restart
```
### SELinux ###
SELinux harus diset permissive atau diperbolehkan agar CloudStack dapat berjalan dengan baik.
```
setenforce 0
```
Atur file /etc/selinux/config :
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
# targeted - Targeted processes are protected,
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

### NTP ###
NTP diatur agar jam pada cloud server dapat sinkron. Namun NTP tidak terinstal secara default sehingga harus diinstal terlebih dahulu dan diatur.
```
yum -y install ntp
```
Nyalakan service NTP sebagai berikut :
```
chkconfig ntpd on
service ntpd start
```
### Configuring the CloudStack Package Repository ###
* Tambahkan repository CloudStack
```
nano /etc/yum.repos.d/cloudstack.repo
```
Tuliskan isinya sebagai berikut :
```
[cloudstack]
name=cloudstack
baseurl=http://cloudstack.apt-get.eu/centos/6/4.9/
enabled=1
gpgcheck=0
```
### NFS ###
Menggunakan NFS pada primary dan secondary storage, maka perlu diinstal NFS :
```
yum -y install nfs-utils
```
Atur pada file /etc/exports :
```
/export/secondary *(rw,async,no_root_squash,no_subtree_check)
/export/primary *(rw,async,no_root_squash,no_subtree_check)
```
Buat folder baru bernama primary dan secondary :
```
mkdir -p /export/primary
mkdir /export/secondary
```
Atur domain pada /etc/idmapd.conf seperti berikut, untuk nama domain misal cloud.coba
```
Domain = cloud.coba
```
Atur juga pada file /etc/sysconfig/nfs seperti berikut :
```
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892
RQUOTAD_PORT=875
STATD_PORT=662
STATD_OUTGOING_PORT=2020
```
Atur firewall dari file /etc/sysconfig/iptables :
```
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 111 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 111 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 2049 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 32803 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 32769 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 892 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 892 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 875 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 875 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p tcp --dport 662 -j ACCEPT
-A INPUT -s 172.16.10.0/24 -m state --state NEW -p udp --dport 662 -j ACCEPT
```
Restart firewall :
```
service iptables restart
```
Jalankan beberapa service NFS berikut :
```
service rpcbind start
service nfs start
chkconfig rpcbind on
chkconfig nfs on
```
