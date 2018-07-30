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
### Management Server Installation ###
* Instal terlebih dahulu MySQL Server
```
yum -y install mysql-server
```
Atur file /etc/my.cnf pada [mysqld] section :
```
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```
Restart mysqld :
```
service mysqld start
chkconfig mysqld on
```
* MySQL connector Installation
Buat file /etc/yum.repos.d/mysql.repo dengan aturan :
```
[mysql-connectors-community]
name=MySQL Community connectors
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/$releasever/$basearch/
enabled=1
gpgcheck=1
```
* Import GPG public key dari MySQL:
```
rpm --import http://repo.mysql.com/RPM-GPG-KEY-mysql
```
* Install mysql-connector
```
yum install mysql-connector-python
```
* Instalasi management server
```
yum -y install cloudstack-management
cloudstack-setup-databases cloud:password@localhost --deploy-as=root
```
Jika berhasil pesan seperti berikut akan muncul :
```
“CloudStack has successfully initialized the database.”
```
```
cloudstack-setup-management
```
Jika servlet container adalah Tomcat7 maka argumen -tomcat7 harus digunakan.

* System Template Setup
CloudStack menggunakan angka sistem VM yang menyediakan fungsionalitas untuk mengakses console VM, menyediakan berbagai layanan jaringan, dan mengelola berbagai aspek storage. Selanjutnya perlu mendownload system VM Template dan mendeploynya. Management server termasuk script yang digunakan untuk memanipulasi system VMs images.
```
/usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
-m /export/secondary \
-u http://cloudstack.apt-get.eu/systemvm/4.6/systemvm64template-4.6.0-kvm.qcow2.bz2 \
-h kvm -F
```
Managemen server selesai, selanjutnya menginstal hypervisor.

# Instalasi dan Konfigurasi KVM

## Instalasi

Instalasi KVM bisa kita gunakan dengan perintah :

```yum -y install cloudstack-agent```

## Konfigurasi

Ada 2 KVM yang dikonfigurasi, yaitu libvirt dan QEMU.

**Konfigurasi QEMU**

Kita hanya perlu mengedit QEMU VNC dengan cara mengedit ```/etc/libvirt/qemu.conf``` dan pastikan tiap line ada dan tidak ada commend

```vnc_listen=0.0.0.0```

**Konfigurasi Libvirt**

CloudStack menggunakan libvirt untuk memanage  VM, sehingga libvirt harus terkonfigurasi secara benar. Libvirt adalah clount-agent dependen dan harus sudah di install.

* Untuk migrasi secara live libvirt harus menerima unsecure TCP Connection. Kita perlu menonaktifkan libvirt untuk melakukan Multicast DNS advertising. Kedua setting settinga ini ada ```/etc/libvirt/libvirtd.conf```

Set parameter menjadi :

```
listen_tls = 0
listen_tcp = 1
tcp_port = "16059"
auth_tcp = "none"
mdns_adv = 0
```

* Hidupkan  “listen_tcp” di libvirtd.conf tidak cukup, keta harus merubah parameters dengan mengedit ```/etc/sysconfig/libvirtd```

Hilangkan comment pada perintah :

```LIBVIRTD_ARGS="--listen"```

* Kemudian restart libvirt

```service libvirtd restart```

* Untuk memastikan, kita cek apakah KVM sudah running di PC kita 

```
lsmod | grep kvm
```

____

# Konfigurasi

Seperti yang kami sebutkan sebelumnya, kami akan menggunakan grup keamanan untuk memberikan isolasi dan secara default yang menyiratkan bahwa kami akan menggunakan jaringan lapisan-2 datar. Ini juga berarti bahwa kesederhanaan pengaturan kami berarti bahwa kita dapat menggunakan penginstal cepat.

### Akses UI

Untuk mendapatkan akses ke antarmuka web CloudStack, arahkan browser Anda ke http://172.16.10.2:8080/client Nama pengguna default adalah ‘admin’, dan kata sandi defaultnya adalah ‘kata sandi’. Anda akan melihat layar splash yang memungkinkan Anda memilih beberapa opsi untuk mengatur CloudStack. Anda harus memilih opsi Lanjutkan dengan Pengaturan Dasar.

Anda sekarang harus melihat prompt yang meminta Anda mengubah kata sandi untuk pengguna admin. Silakan lakukan.

### Menyiapkan Zona

Zona adalah entitas organisasi terbesar di CloudStack - dan kami akan membuatnya, ini seharusnya adalah layar yang Anda lihat di depan Anda sekarang. Dan bagi kami ada 5 buah informasi yang kami butuhkan.

	Name - we will set this to the ever-descriptive ‘Zone1’ for our cloud.
	Public DNS 1 - we will set this to 8.8.8.8 for our cloud.
	Public DNS 2 - we will set this to 8.8.4.4 for our cloud.
	Internal DNS1 - we will also set this to 8.8.8.8 for our cloud.
	Internal DNS2 - we will also set this to 8.8.4.4 for our cloud.
	
### Catatan

CloudStack membedakan antara DNS internal dan publik. DNS internal diasumsikan mampu menyelesaikan hostname internal saja, seperti nama DNS server NFS Anda. DNS publik disediakan untuk VM tamu untuk menyelesaikan alamat IP publik. Anda dapat memasukkan server DNS yang sama untuk kedua jenis, tetapi jika Anda melakukannya, Anda harus memastikan bahwa alamat IP internal dan publik dapat rute ke server DNS. Dalam kasus spesifik kami, kami tidak akan menggunakan nama apa pun untuk sumber daya secara internal, dan kami memang telah menetapkan untuk mencari sumber daya eksternal yang sama sehingga tidak menambahkan pengaturan namerserver ke daftar persyaratan kami.

### Konfigurasi Pod

Sekarang setelah kami menambahkan Zona, langkah berikutnya yang muncul adalah prompt untuk informasi memasukkan pod. Yang mencari beberapa barang.

    Name - We’ll use Pod1 for our cloud.
    Gateway - We’ll use 172.16.10.1 as our gateway
    Netmask - We’ll use 255.255.255.0
    Start/end reserved system IPs - we will use 172.16.10.10-172.16.10.20
    Guest gateway - We’ll use 172.16.10.1
    Guest netmask - We’ll use 255.255.255.0
    Guest start/end IP - We’ll use 172.16.10.30-172.16.10.200

### Cluster
Sekarang setelah kami menambahkan Zona, kami hanya perlu menambahkan beberapa item lagi untuk mengonfigurasi gugus.

    Name - We’ll use Cluster1
    Hypervisor - Choose KVM
    
Anda harus diminta untuk menambahkan host pertama ke kluster Anda pada titik ini. Hanya beberapa bit informasi yang dibutuhkan.

    Hostname - we’ll use the IP address 172.16.10.2 since we didn’t set up a DNS server.
    Username - we’ll use root
    Password - enter the operating system password for the root user

### Primary Storage
Dengan kluster Anda sekarang setup - Anda akan diminta untuk informasi penyimpanan primer. Pilih NFS sebagai tipe penyimpanan dan kemudian masukkan nilai berikut di bidang:

    Name - We’ll use Primary1
    Server - We’ll be using the IP address 172.16.10.2
    Path - Well define /export/primary as the path we are using
    

### Secondary Storage
Jika ini adalah zona baru, Anda akan dimintai informasi penyimpanan sekunder - isi sebagai berikut:
    
    NFS server - We’ll use the IP address 172.16.10.2
    Path - We’ll use /export/secondary

Sekarang, klik Luncurkan dan cloud Anda harus memulai penyetelan - mungkin perlu waktu beberapa menit tergantung pada kecepatan koneksi internet Anda untuk penyetelan yang akan diselesaikan.

Itu saja, Anda selesai dengan pemasangan cloud CloudStack Apache Anda.