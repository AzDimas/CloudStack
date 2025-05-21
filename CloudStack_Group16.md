# Dokumentasi Instalasi Apache Cloudstack
## Group 16:
- Sihombing Giovano Geraldo - 2206059566
- Azriel Dimas Ash-Shidiqi - 2206059414
- Yasmin Devina Sinuraya - 2206817244
- Fabsesya Muhammad - 2206829433


## Persiapan 

### Spesifikasi Hardware

```
CPU : Intel Core i5 Gen 8
RAM : 24 GB
Storage : 200GB
Operating System : Ubuntu Server 24.04
```


**Penjelasan:**  
Spesifikasi perangkat keras ini digunakan sebagai dasar instalasi Apache CloudStack. Prosesor dan RAM yang cukup tinggi memastikan performa sistem tetap stabil. Ubuntu Server dipilih karena ringan, aman, dan mendukung banyak tools virtualisasi.

---

### Alamat Jaringan

```
Network Address : 192.168.0.0/24
Host IP address : 192.168.0.10/24
Gateway : 192.168.0.1
Public IP : 125.165.153.204
```

**Penjelasan:**  
Konfigurasi ini menetapkan jaringan lokal dan akses ke jaringan luar:
- `192.168.0.0/24` adalah rentang alamat jaringan lokal (subnet).
- `192.168.0.10` adalah IP statis dari server dalam jaringan lokal.
- `192.168.0.1` sebagai gateway utama untuk mengakses internet.
- `125.165.153.204` adalah IP publik yang digunakan untuk akses dari luar jaringan lokal.


## Konfigurasi Jaringan

### Mengedit File Konfigurasi Jaringan di Direktori /netplan

```
sudo -i          # Sudo untuk mendapatkan hak akses sebagai root, -i agar command sudo tidak perlu diketik ulang ketika ingin menggunakan command khusus root. 
cd /etc/netplan  # Pindah ke direktori etc/netplan
nano ./*.yaml    # Mengedit file YAML
``` 
 
Langkah ini dilakukan untuk masuk sebagai superuser (`sudo -i`), lalu berpindah ke direktori `/etc/netplan` tempat konfigurasi jaringan disimpan. File YAML di dalam folder ini akan diedit untuk menyesuaikan pengaturan jaringan secara manual.

---

Kemudian edit isi file YAML seperti berikut:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: false
      dhcp6: false
      optional: true
  bridges:
    cloudbr0:
      addresses: [192.168.0.10/24]  
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp1s0]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```

Bagian ini membuat sebuah *bridge interface* bernama `cloudbr0` yang menghubungkan adapter `enp1s0` ke jaringan.  
- `dhcp4` dan `dhcp6` disetel ke `false` agar menggunakan IP statis.  
- `addresses` diisi dengan IP lokal statis untuk host.  
- `routes` untuk mengatur jalur/route traffic, routes yang digunakan adalah default (0.0.0.0) yang melalui alamat default gateway (192,168.0.1)
-`nameservers` untuk mengatur DNS yang digunakan, yaitu 1.1.1.1 (Cloudfare) dan 8.8.8.8 (Google).  
- Penggunaan *bridge* penting untuk kebutuhan virtualisasi, karena memungkinkan VM berbagi koneksi jaringan secara langsung.

<img src="https://i.imgur.com/zsy6mBc.jpeg" width="600"/>
---


### Menerapkan Konfigurasi Jaringan

```
netplan generate        # Membuat file config
netplan apply           # Menerapkan konfigurasi jaringan ke sistem
reboot                  # Reboot sistem
```

Setelah konfigurasi disimpan, perintah `netplan generate` dan `netplan apply` digunakan untuk menerapkan pengaturan jaringan baru. Sistem kemudian di-*reboot* agar semua perubahan benar-benar aktif dan stabil.


### Menguji Koneksi Jaringan

```
ip address        # Melihat IP address dan interface jaringan yang aktif
ping 8.8.8.8      # Mengirim ping ke google untuk memastikan konfigurasi jaringan sudah benar dan VM bisa menggunakan jaringan internet dengan baik
```
<img src="https://i.imgur.com/xhqBzSp.jpeg" width="600"/>

<img src="https://i.imgur.com/q8LV8gk.jpeg" width="600"/>

Langkah ini bertujuan untuk memastikan bahwa konfigurasi jaringan yang telah dibuat sebelumnya sudah berhasil diterapkan dan sistem bisa mengakses jaringan eksternal (internet).

---

### Masuk ke Sistem sebagai Pengguna root

```
sudo -i                  # Sudo untuk mendapatkan hak akses sebagai root, -i agar command sudo tidak perlu diketik ulang ketika ingin menggunakan command khusus root. 
passwd root              # Mengubah password milik root
#change it to Pa$$w0rd   # Mengubah password root menjadi Pa$$w0rd
```

### Mengaktifkan Login SSH untuk Pengguna root

```
# Mengizinkan login sebagai root melalui SSH
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config  

#restart ssh service 
service ssh restart

#or
systemctl restart sshd.service
```

Langkah ini memungkinkan pengguna `root` untuk login ke sistem menggunakan SSH. File konfigurasi `sshd_config` dimodifikasi agar baris `PermitRootLogin yes` ditambahkan, lalu layanan SSH di-restart agar perubahan diterapkan.

## Instalasi CloudStack (Controller dan Compute Node di Host yang Sama)


### Mengimpor Kunci Repositori CloudStack

```
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.20 / > /etc/apt/sources.list.d/cloudstack.list
```

- Baris pertama membuat direktori untuk menyimpan kunci publik CloudStack.  
- Baris kedua mengunduh file kunci dan langsung meneruskannya ke `gpg --dearmor`, yang mengubah format ASCII menjadi biner.  Hasilnya disimpan sebagai file `.gpg` melalui `sudo tee`.  
- Baris ketiga menambahkan alamat repositori CloudStack ke daftar sumber paket sistem.

### Instalasi CloudStack dan MySQL Server

```
apt update -y                                  # Mengizinkan update packet pada Ubuntu
apt install cloudstack-management mysql-server # Meng-install Cloudstack management dan server MySQL
```

### Konfigurasi MySQL

#### Membuka File Konfigurasi MySQL

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Langkah ini untuk mengedit konfigurasi utama MySQL. File `mysqld.cnf` akan disesuaikan agar dapat digunakan oleh CloudStack, misalnya dengan mengubah bind address dan parameter performa lainnya.


#### Menyalin Baris ke Dalam Bagian [mysqld]

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

Pengaturan ini digunakan untuk mengonfigurasi database agar kompatibel dengan CloudStack.  
- `sql-mode` memastikan validasi data yang ketat.  
- `log-bin` dan `binlog-format` diaktifkan untuk mendukung replikasi dan pelacakan perubahan data.

<img src="https://i.imgur.com/SrEHwrO.jpeg" width="600"/> ping

#### Restart Layanan MySQL
```
systemctl restart mysql
```

Diperlukan untuk menerapkan perubahan konfigurasi pada database server.

### Deploy Database sebagai root, lalu Buat User "cloud"

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.0.10
```

Perintah ini akan:
- Menginisialisasi database CloudStack.
- Membuat user `cloud` dengan password `cloud`.
- Mengatur IP host (dalam hal ini 192.168.0.10) sebagai pengakses utama database.

### Konfigurasi Penyimpanan Primer dan Sekunder

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

- Menginstal layanan NFS untuk membagikan direktori ke jaringan.  
- Direktori `/export/primary` dan `/export/secondary` dibuat sebagai lokasi penyimpanan utama dan sekunder untuk CloudStack.  
- File `/etc/exports` diisi dengan konfigurasi akses, lalu diterapkan dengan `exportfs -a`.


### Konfigurasi NFS Server

```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

Baris-baris ini mengatur port tertentu untuk layanan NFS agar kompatibel dan stabil:
- Mengatur port statis untuk `mountd`, `statd`, dan `rquotad`.
- Menambahkan baris `NEED_STATD=yes` agar statd dijalankan.
- Setelah semuanya dikonfigurasi, layanan NFS di-restart.

Penjelasan command:

* `sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server`
Command ini akan mencari line RPCMOUNTDOPTS="--manage-gids" pada /etc/default/nfs-kernel-server, lalu menggantinya dengan RPCMOUNTDOPTS="-p 892 --manage-gids". Maksud dari -i adalah melakukan modifikasi ini langsung di file asli dan menyimpannya.
==> Tujuan command ini yaitu untuk mengonfigurasikan NFS server agar rpc.mountd mendengarkan (listen) ke port 892 dan menggunakan manage-gids.


* `sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common`
Command ini akan mencari line yang berisi STATDOPTS= di file /etc/default/nfs-common dan menggantinya dengan STATDOPTS="--port 662 --outgoing-port 2020". Maksud dari -i adalah melakukan modifikasi ini langsung di file asli dan menyimpannya. 
==> Tujuan command ini untuk menetapkan port 662 untuk statd dan port 2020 untuk outgoing port atau koneksi keluar.

* `echo "NEED_STATD=yes" >> /etc/default/nfs-common`
Command ini akan menambahkan line baru yang berisi NEED_STATD=yes ke bagian paling akhir (>>) file /etc/default/nfs-common. 
==> Tujuan command ini untuk mengaktifkan statd pada konfigurasi NFS. 

* `sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota`
Command ini akan mencari line RPCRQUOTADOPTS=  di file /etc/default/quota dan menggantinya dengan RPCRQUOTADOPTS="-p 875".
==> Tujuan command ini untuk menetapkan port 875 untuk rpc.rquotad. 


* `service nfs-kernel-server restart`
Melakukan restart pada NFS service

## Konfigurasi Host CloudStack dengan Hypervisor KVM

### Instalasi KVM dan CloudStack Agent

```
apt install qemu-kvm cloudstack-agent -y
```

Menginstal hypervisor KVM (untuk menjalankan VM) dan agen CloudStack (untuk menghubungkan host ke manajer CloudStack).

### Konfigurasi Manajemen Virtualisasi KVM

#### Ubah Beberapa Baris Konfigurasi

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# On Ubuntu 22.04, add LIBVIRTD_ARGS="--listen" to /etc/default/libvirtd instead.

sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

- `vnc_listen = "0.0.0.0"` memungkinkan akses VNC dari jaringan manapun.  
- `LIBVIRTD_ARGS="--listen"` membuat daemon `libvirtd` bisa menerima koneksi dari host lain, yang dibutuhkan CloudStack agar dapat mengontrol VM secara remote.

Penjelasan command:

* `sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf`
Command ini akan mencari line #vnc_listen di file /etc/libvirt/qemu.conf dan menggantinya dengan vnc_listen = "0.0.0.0". IP address 0.0.0.0 adalah IP address yang menyatakan keseluruhan IP address pada sistem lokal. 
==> Tujuan command ini untuk mengonfigurasikan agar VNC dapat mendengar (listen) ke seluruh IP adddress pada sistem lokal.  

* `sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd`
Command ini akan mencari line LIBVIRTD_ARGS= di file /etc/default/libvirtd dan menggantinya dengan LIBVIRTD_ARGS="--listen". Maksud -i.bak adalah untuk modifikasi langsung file aslinya dan membuat file backup-nya (.bak)
==> Tujuan command ini untuk mengonfigurasikan agar libvirtd aktif dan dapat mendegarkan (listen) terhadap request pembentukan koneksi jaringan.


#### Command untuk Konfigurasi libvirtd

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

- `listen_tls=0`
Menonaktifkan koneksi TLS (secure) pada daemon libvirtd.
- `listen_tcp=1`
Mengaktifkan koneksi TCP standar.
- `tcp_port = "16509"`
Menentukan port TCP yang akan digunakan oleh libvirtd untuk menerima koneksi.
- `mdns_adv = 0`
Menonaktifkan iklan layanan via multicast DNS (mDNS), agar tidak ditemukan secara otomatis di jaringan.
- `auth_tcp = "none"`
Menonaktifkan autentikasi pada koneksi TCP ke libvirtd.


#### Restart layanan libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

*Penjelasan command:*
* `systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket`
Menonaktifkan semua soket default yang digunakan libvirtd untuk komunikasi internal dan eksternal. Masking ini mencegah sistem menjalankan soket tersebut secara otomatis atau manual.
- `systemctl restart libvirtd`
Me-restart daemon libvirtd agar semua konfigurasi baru yang ditambahkan ke libvirtd.conf diterapkan.
- Sedikit penjelasan mengenai libvirtd, libvirtd adalah daemoin yang akan mengelola virtual machine, virtual network, dan storage untuk melakukan virtualiasi. 


### Konfigurasi Tambahan untuk Mendukung Docker dan Layanan Lain

```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

Perintah di atas akan memastikan bahwa paket ARP dan IP tidak lagi diproses oleh arptables maupun iptables. Hal ini penting agar Docker dan container lainnya dapat berjalan dengan baik tanpa konflik jaringan.


### Membuat UUID Unik untuk Host

```
apt install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

UUID digunakan untuk mengidentifikasi host secara unik oleh CloudStack.

### Konfigurasi Firewall dengan Iptables dan Persistensinya

```
NETWORK=192.168.0.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT    # NFS server port
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT   # NFS mountd (dynamic port)
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT   # NFS nlockmgr
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT     # NFS rpc.mountd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT     # 	NFS rquotad (quota)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT     # 	NFS statd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT    # CloudStack Agent
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT    # 	CloudStack Console Proxy (VM console)	
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT   # libvirt (KVM communication)

apt install iptables-persistent
#just answer yes yes
```

- `-A INPUT`. Menambahkan aturan pada chain INPUT.
- `-s $NETWORK`. Menentukan sumber IP yaitu subnet internal.
- `-m state --state NEW`. Memastikan rules atau aturan tersebut hanya berlaku pada paket dari koneksi baru saja.
- `-p udp/tcp --dport [PORT]`. Menargetkan protokol dan port tertentu.
- `-j ACCEPT`. Mengizinkan koneksi yang sesuai aturan.



### Instalasi Ulang CloudStack Agent (Jika Belum Ada)

```
systemctl unmask cloudstack-agent
apt update -y
apt install cloudstack-agent -y
```


### Menonaktifkan AppArmor untuk libvirtd

```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

- `ln -s`. Membuat symbolic link (referensi atau rujukan ke suatu file atau direktori). 
- `apparmor_parser -R`. Menghapus profil AppArmor dari kernel.

Penjelasan command:


* `ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/`
Command ini bertujuan untuk membuat symbolic link dari /etc/apparmor.d/usr.sbin.libvirtd ke /etc/apparmor.d/disable/. Ini bertujuan untuk menonaktifkan profil AppArmor untuk libvirtd. 

* `ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/`
Command ini bertujuan untuk membuat symbolic link dari /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper ke /etc/apparmor.d/disable/. Ini bertujuan untuk menonaktifkan profil AppArmor untuk virt-aa-helper. 

* `apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd`
Command ini bertujuan untuk menghapus profil libvirtd dari sistem AppArmor. 

* `apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper`
Command ini bertujuan untuk menghapus profil virt-aa-helper dari sistem AppArmor.



### Jalankan Management Server CloudStack

```
cloudstack-setup-management
systemctl status cloudstack-management
```
<img src="https://i.imgur.com/GL6rk65.jpeg" width="600"/> 

- `cloudstack-setup-management`. Men-setup manajemen server CloudStack termasuk IP dan koneksi database.
- `systemctl status`. Memeriksa apakah layanan berhasil berjalan.



### Akses Web UI CloudStack

```
http://<IP_ADDRESS>:8080
```

Contoh:

```
http://192.168.0.10:8080/client
```
<img src="https://i.imgur.com/CZd7l7A.jpeg" width="600"/> 

### Menginstal paket WireGuard dan resolvconf. resolvconf membantu mengelola DNS ketika VPN aktif.

```
sudo apt install wireguard resolvconf
```


### Membuka atau membuat file konfigurasi WireGuard dengan nama SNetff7ac99e.conf. Nama file ini akan menjadi nama interface WireGuard.

```
sudo nano /etc/wireguard/SNetff7ac99e.conf
```

<img src="https://i.imgur.com/WXQIsIz.png" width="600"/> 


### Mengaktifkan interface VPN dengan konfigurasi SNetff7ac99e.conf. VPN akan mulai menerima koneksi.


```
sudo wg-quick up SNetff7ac99e
```

### Membuat layanan VPN ini otomatis aktif setiap kali server boot.

```
sudo systemctl enable wg-quick@SNetff7ac99e
```

### Menampilkan status WireGuard saat ini: termasuk interface aktif, peer yang terhubung, transfer data, dan durasi koneksi.

```
sudo wg show
```

<img src="https://i.imgur.com/MK8S3IB.jpeg" width="600"/> 


### Akses Dashboard Apache Cloudstack lewat VPN sudah bisa dilakukan. Begitu juga dengan SSH

```
http://columba.s-net.id:9611/client/
```

<img src="https://i.imgur.com/vWNhR1o.png" width="600"/> 

### Akses Ubuntu Server lewat SSH juga telah bisa dilakukan

<img src="https://i.imgur.com/nqAUdWR.png" width="600"/> 

<img src="https://i.imgur.com/afeiI9X.png" width="600"/> 
