[![Logo](https://github.com/user-attachments/assets/30735ef8-5268-4c95-b172-f1b826ecc745)](https://ee.ui.ac.id/)


# Proyek Apache CloudStack | Cloud Computing


![Logo_CloudStack](https://hackmd.io/_uploads/H16l590Mgl.png)

## Group CloudStack 16:
* Sihombing Giovano Geraldo - 2206059566
* Azriel Dimas Ash-Shidiqi - 2206059414
* Yasmin Devina Sinuraya - 2206817244
* Fabsesya Muhammad - 2206829433


## Apache CloudStack
Apache Cloudstack adalah sebuah platform Cloud Computing yang dimiliki oleh Apache. Bersifat Open-Source dan digunakan untuk membangun layanan IaaS seperti AWS. Beberapa Karakteristiknya:
● Dapat digunakan untuk cloud pribadi, publik, atau hybrid.
● Mendukung berbagai hypervisor seperti KVM, XenServer, VMware.
● Cocok untuk organisasi yang ingin mengelola cloudnya sendiri

## Langkah-langkah Implementasi CloudStack
1. Mempersiapkan sistem yang akan digunakan.
2. Melakukan konfigurasi jaringan seperti IP statis, gateway, DNS, dan bridge. Hal ini untuk mengatur koneksi VM ke network secara fisik.
3. Meng-install MySQL sebagai database untuk CloudStack
4. Meng-install CloudStack Management Server, dan melakukan setup database dan akses ke interface web. 
5. Melakukan konfigurasi NFS sebagai tempat penyimpanan (primary dan secondary).
6. Melakukan instalasi hypervisor KVM dan melakukan konfigurasi pada libvirtd, port, dan generate UUID. Ini bertujuan untuk bisa menjalankan VM di atas hypervisor.
7. Melakukan konfigurasi firewall untuk keamanan dan mengatur port-port yang akan digunakan untuk CloudStack.
8. Melakukan konfigurasi VPN dan SSH untuk akses remote dengan menggunakan OpenSSH dan WireGuard VPN. 


## Ilustrasi Topologi Implementasi CloudStack
![Untitled Diagram.drawio(9)](https://hackmd.io/_uploads/rJBfmjCMxl.png)


## Referensi
1. Ahmad Rifqi. (2023). CloudStack Install and Configure. GitHub Repository.
    https://github.com/AhmadRifqi86/cloudstack-install-and-configure
    
2. Bakshi, K. (2018). A Tutorial on CloudStack. In Cloud Computing Technologies for Green Enterprise Architectures. IGI Global.
    https://www.igi-global.com/chapter/a-tutorial-on-cloudstack/188131
