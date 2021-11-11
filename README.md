# Jarkom-Modul-3-C06-2021

## Anggota Kelompok : 
- 05111940000004 | Nizar Mayraldo         | No: 1-7
- 05111940000087 | William Handi Wijaya   | No: 8-13
- 05111940000202 | Muhammad Naufaldillah  | No: 8-10

## Jawaban Praktikum
### No.1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.

### Jawaban
#### Topologi GNS
Pada kelompok kami setting network configurationnya menyesuaikan dengan prefix IP dari kelompok kami yaitu `10.17`.  

![Topologi GNS](https://user-images.githubusercontent.com/52129348/141282783-909a55be-2966-4d36-ab7b-94c37daea770.png)  

#### EniesLobby GNS Interfaces (sebagai DNS Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.2
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Jipangu GNS Interfaces (sebagai DHCP Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.4
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Water7 GNS Interfaces (sebagai Proxy Server)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.17.2.3
	netmask 255.255.255.0
	gateway 10.17.2.1
```

#### Loguetown, Alabasta, TottoLand, Skypie GNS Interfaces (sebagai DHCP Client)
```
auto eth0
iface eth0 inet dhcp
```

#### Konfigurasi node 
1. Pada **Foosha**, jalankan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.17.0.0/16`..
2. Pada node **Jipangu**, 
   1. Install DHCP Server dengan perintah `apt-get install isc-dhcp-server`.
   2. Jalankan DHCP Server dengan perintah `service isc-dhcp-server start`.
   3. Install **apache2-utils** dengan perintah `apt-get install apache2-utils`.
3. Pada node **Water7**, 
   1. Install Squid dengan perintah `apt-get install squid`.
   2. Jalankan Squid dengan perintah `service squid start`.
4. Pada node **LogueTown**, **Alabasta**, **TottoLand**, dan **Skypie**,
    1. Install **lynx** dengan perintah `apt-get install lynx`.
    2. Install **dnsutils** dengan perintah `apt-get install dnsutils`.
5. Pada **EniesLobby**,
    1. Pada `/etc/resolv.conf`, tuliskan `nameserver 192.168.122.1`
    2. Install dan aktifkan **bind9** dengan perintah:
        ```
        apt-get install bind9 -y
        service bind9 start
        ```
Setelah itu, konfigurasi ini akan disimpan pada `~/.bashrc`.

Keterangan :
* `iptables`: iptables merupakan suatu tools dalam sistem operasi Linux yang berfungsi sebagai filter terhadap lalu lintas data.
* `NAT` (Network Address Translation): Suatu metode penafsiran alamat jaringan yang digunakan untuk menghubungkan lebih dari satu komputer ke jaringan internet dengan menggunakan satu alamat IP.
* `Masquerade`: Digunakan untuk menyamarkan paket, misal mengganti alamat pengirim dengan alamat router. -s (Source Address): Spesifikasi pada source. Address bisa berupa nama jaringan, nama host, atau alamat IP.


### No.2
dan Foosha sebagai DHCP Relay.

### Jawaban
#### Foosha GNS Interfaces (sebagai DHCP Relay)
```
# DHCP config for eth0
auto eth0
iface eth0 inet dhcp

# Static config for eth1
auto eth1
iface eth1 inet static
	address 10.17.1.1
	netmask 255.255.255.0

# Static config for eth2
auto eth2
iface eth2 inet static
	address 10.17.2.1
	netmask 255.255.255.0

# Static config for eth3
auto eth3
iface eth3 inet static
	address 10.17.3.1
	netmask 255.255.255.0
```

Pada **Foosha** (DHCP Relay):
1. Instal DHCP Relay dengan perintah `apt-get install isc-dhcp-relay`.
2. Tuliskan script berikut pada `~/2.sh`:
    ```
   # Setup DHCP Relay
   echo "
   SERVERS=\"10.17.2.4\"
   INTERFACES=\"eth1 eth2 eth3\"
   OPTIONS=\"\"
   " > "/etc/default/isc-dhcp-relay"

   service isc-dhcp-relay start
    ```
3. Tuliskan `bash ~/2.sh` di `~/.bashrc`.

Pada **Jipangu** (DHCP Server):
1. Tuliskan script berikut pada `~/2.sh`:
   ```
   # Setup DHCP Server
   echo "
   INTERFACES="eth0"
   " > "/etc/default/isc-dhcp-server"

   service isc-dhcp-server start
   ```
2. Tuliskan `bash ~/2.sh` di `~/.bashrc`.


### No.3 & No.5-6 (Switch 1)
#### No.3
Luffy dan Zoro menyusun peta tersebut dengan hati-hati dan teliti.

Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
1. Semua client yang ada **HARUS** menggunakan konfigurasi IP dari DHCP Server.
2. Client yang melalui Switch1 mendapatkan range IP dari `10.17.1.20` - `10.17.1.99` dan `10.17.1.150` - `10.17.1.169` 

#### No.5
Client mendapatkan DNS dari **EniesLobby** dan client dapat terhubung dengan 
internet melalui DNS tersebut.

#### No.6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### Jawaban
Pada **EniesLobby** (DNS Server):
1. Tuliskan script berikut pada `~356.sh`:
   ```
   echo "
   options {
      directory \"/var/cache/bind\";

      forwarders {
         192.168.122.1;
      };
      // dnssec-validation auto;
      allow-query{ any; };

      auth-nxdomain no;    # conform to RFC1035
      listen-on-v6 { any; };
   };" > /etc/bind/named.conf.options
   ```
2. Restart bind9.
3. Tuliskan `bash ~/356.sh` di `~/.bashrc`.

Pada **Jipangu** (DHCP Server):
1. Tuliskan script berikut pada `~/356.sh`:
   ```
   echo "
   # Setup IP for nodes in switch1 
   subnet 10.17.1.0 netmask 255.255.255.0 {
      range 10.17.1.20 10.17.1.99;
      range 10.17.1.150 10.17.1.169;
      option routers 10.17.1.1;
      option broadcast-address 10.17.1.255;
      option domain-name-servers 10.17.2.2;   # Number 5
      default-lease-time 360;                 # Number 6
      max-lease-time 7200;                    # Number 6
   }" >> /etc/dhcp/dhcpd.conf
   ```
2. Tuliskan `bash ~/356.sh` di `~/.bashrc`.
3. Jalankan script di atas dan restart DHCP server dengan perintah `service isc-dhcp-server restart`.


### No.4 & No.5-6 (Switch 2)
#### No.4
Client yang melalui Switch3 mendapatkan range IP dari `10.17.3.30` - `10.17.3.50`.

#### No.5
Client mendapatkan DNS dari **EniesLobby** dan client dapat terhubung dengan 
internet melalui DNS tersebut.

#### No.6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui 
Switch2 selama 12 menit dengan waktu maksimal yang dialokasikan untuk 
peminjaman alamat IP selama 120 menit.

### Jawaban
Pada **Jipangu** (DHCP Server):
1. Tuliskan script berikut pada `~/456.sh`:
   ```
   echo "
   # Setup IP for nodes in switch3
   subnet 10.17.3.0 netmask 255.255.255.0 {
      range 10.17.3.30 10.17.3.50;
      option routers 10.17.3.1;
      option broadcast-address 10.17.3.255;
      option domain-name-servers 10.17.2.2;   # Number 5
      default-lease-time 720;                 # Number 6
      max-lease-time 7200;                    # Number 6
   }" >> /etc/dhcp/dhcpd.conf
   ```
2. Tuliskan `bash ~/456.sh` di `~/.bashrc`.
3. Jalankan script di atas dan restart DHCP server dengan perintah `service isc-dhcp-server restart`.


### No.7
Luffy dan Zoro berencana menjadikan **Skypie** sebagai server untuk jual beli 
kapal yang dimilikinya dengan alamat IP yang tetap dengan IP `10.17.3.69`.

### Jawaban
Pada **Jipangu** (DHCP Server):
1. Dapatkan hwaddress milik **Skypie** dengan mengeksekusi perintah `ip a` di **Skypie**.
2. Tuliskan script berikut pada `~/7.sh`:
    ```
   echo "
   host Skypie {
      hardware ethernet ca:25:dd:4b:6b:17;
      fixed-address 10.17.3.69;
   }" >> "/etc/dhcp/dhcpd.conf"
    ```
3. Tuliskan `bash ~/7.sh` di `~/.bashrc`.
4. Jalankan script di atas dan restart DHCP server dengan perintah `service isc-dhcp-server restart`.  

Pada **Skypie** (DHCP Client):
1. Restart node.
2. Tambahkan `hwaddress ether ca:25:dd:4b:6b:17` ke interfaces.
3. Pastikan ip **Skypie** sama dengan `10.17.3.69` melalui perintah `ip -a`.


### No.8
**Loguetown** digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.

Pada **Loguetown**, proxy harus bisa diakses dengan nama `jualbelikapal.C06.com` dengan port yang digunakan adalah 5000.

### Jawaban
Pada **Water7** (Proxy Server):
1. Pastikan **Squid** sudah terinstall.
2. Tulisan script berikut pada `~/8-10.sh`:
   ```
   # Backup Squid Configuration
   mv /etc/squid/squid.conf /etc/squid/squid.conf.bak

   # Setup Squid Configuration
   echo "
   http_port 5000
   visible_hostname jualbelikapal.C06.com
   dns_nameservers 10.17.2.2
   " > /etc/squid/squid.conf
   ```
2. Tuliskan script berikut pada `~/8.sh`:
   ```
   # Setup Squid Configuration
   echo "
   http_access allow all
   " >> /etc/squid/squid.conf
   ```
4. Tuliskan `bash ~/8-10.sh` dan `bash ~/8.sh` di `~/.bashrc`.
5. Comment script `9.sh` dan `10.sh` di `~/.bashrc`.
6. Jangan lupa untuk restart **Squid** dengan perintah `service squid restart`.

Pada **Loguetown** (Proxy Client):
1. Jalankan perintah tersebut pada terminal untuk mengkatifkan / megnonaktifkan http_proxy;
   ```
    # Enable http_proxy
    export http_proxy="http://10.17.2.3:5000"
    env | grep -i proxy

    # Disable http_proxy
    #unset http_proxy
   ```
### Hasil
![Hasil no.8](https://user-images.githubusercontent.com/52129348/141282738-398ebc08-e746-403f-b7de-dfe011c53781.png)  


### No.9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu `luffybelikapalC06` dengan password `luffy_C06` dan `zorobelikapalC06` dengan password `zoro_C06`.

### Jawaban
Pada **Water7** (Proxy Server):
1. Pastikan **Apache2-utils** sudah terinstall dengan perintah `apt-get install apache2-utils`.
2. Pastikan `http_access allow` tidak ada pada `/etc/squid/squid.conf`,
3. Tuliskan script berikut pada `~/9.sh`:
   ```
   # Buat akun dengan username "luffybelikapalC06" and password "luffy_C06" dengan enkripsi MD5
   htpasswd -b -c -m /etc/squid/passwd luffybelikapalC06 luffy_C06

   # Buat akun dengan username "zorobelikapalC06" and password "zoro_C06" dengan enkripsi MD5
   htpasswd -b -m /etc/squid/passwd zorobelikapalC06 zoro_C06

   # Setup Squid Auth Configuration
   echo "
   auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
   auth_param basic children 5
   auth_param basic realm Login
   auth_param basic credentialsttl 2 hours
   auth_param basic casesensitive on
   acl USERS proxy_auth REQUIRED
   http_access allow USERS
   " >> "/etc/squid/squid.conf"
   ```
4. Restart **Squid** dengan perintah `service squid restart`.
5. Tuliskan `bash ~/9.sh` di `~/.bashrc`.
6. Comment script `8.sh` di `~/.bashrc`.

### Hasil
![Hasil no.9](https://user-images.githubusercontent.com/52129348/141283380-e77def08-b5ed-4a94-8a71-bf3505455b3a.gif)  


### No.10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00) 

### Jawaban
Pada **Water7** (Proxy Server):
1. Pastikan `http_access allow all` tidak ada pada `/etc/squid/squid.conf`.
2. Pastikan `http_access allow USERS` tidak ada pada `/etc/squid/squid.conf`.
3. Tuliskan script berikut pada `~/10.sh`:
   ```
   # Setup acl time
   echo "
   acl AVAILABLE_1 time MTWH 07:00-11:00
   acl AVAILABLE_2 time TWHF 17:00-23:59
   acl AVAILABLE_3 time WHFA 00:00-03:00
   " > /etc/squid/acl.conf

   # Setup Squid.conf
   echo "
   include /etc/squid/acl.conf

   http_access allow USERS AVAILABLE_1
   http_access allow USERS AVAILABLE_2
   http_access allow USERS AVAILABLE_3
   http_access deny all
   " >> /etc/squid/squid.conf
   ```
4. Tuliskan `bash ~/10.sh` di `~/.bashrc`.
5. Comment script `8.sh` di `~/.bashrc`.
6. Ubah waktu di Proxy Server untuk melakukan testing. Contoh perintah: `date -s "9 NOV 2021 13:00:00"`.

### Hasil
#### Jika bisa
![Hasil no.10a](https://user-images.githubusercontent.com/52129348/141283692-f9d12948-03f1-4c5f-8ba1-6cb13297714e.png)

#### Jika tidak bisa
![Hasil no.10b](https://user-images.githubusercontent.com/52129348/141288863-8853fc42-8aa7-4539-bfbd-fbf04df297d8.gif)


### No.11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses `google.com`, akan diredirect menuju `super.franky.C06.com` dengan website yang sama pada soal shift modul 2. Web server `super.franky.C06.com` berada pada node Skypie.

### Jawaban
Pada **Enieslobby** (DNS Server):
1. Tuliskan script berikut pada `~/11.sh`:
   ```
   # Mendaftarkan domain name
   echo "
   zone \"super.franky.C06.com\" {
         type master;
         file \"/etc/bind/super.franky.C06.com\";
   };
   " >> /etc/bind/named.conf.local

   # Buat file yang menyimpan konfigurasi DNS
   cp /etc/bind/db.local /etc/bind/super.franky.C06.com

   # Buat konfigurasi DNS
   echo "
   ;
   ; BIND data file for local loopback interface
   ;
   \$TTL    604800
   @       IN      SOA     super.franky.C06.com. root.super.franky.C06.com. (
                       2021100401         ; Serial
                           604800         ; Refresh
                           86400          ; Retry
                           2419200        ; Expire
                           604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      super.franky.C06.com.
   @       IN      A       10.17.3.69      ; IP Skypie
   " > /etc/bind/super.franky.C06.com

   # Restart bind9
   service bind9 restart
   ```
2. Tuliskan `bash ~/11.sh` di `~/.bashrc`.

Pada **Skypie** (Web Server):
1. Tuliskan script berikut pada `~/11.sh`:
   ```
   # Install requirements
   apt-get update
   yes '' | apt-get install wget
   yes '' | apt-get install unzip
   yes '' | apt-get install apache2
   service apache2 start
   yes '' | apt-get install php
   yes '' | apt-get install libapache2-mod-php7.0

   # Download super.franky.zip dan unzip file tersebut
   [ ! -f "~/super.franky.zip" ] && `wget -O ~/super.franky.zip https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip`
   [ ! -d "~/super.franky" ] && `unzip -o ~/super.franky.zip`

   # Buat folder yang menjadi document root dan dapatkan konten web dari super.franky.zip
   cp -r ~/super.franky/. /var/www/super.franky.C06.com

   # Buat konfigurasi server apache2
   echo "
   <VirtualHost *:80>
      ServerAdmin webmaster@localhost

      DocumentRoot /var/www/super.franky.C06.com
      ServerName super.franky.C06.com

      ErrorLog ${APACHE_LOG_DIR}/error.log
   </VirtualHost>
   " > /etc/apache2/sites-available/super.franky.C06.com.conf

   # Aktifkan document root tersebut
   a2ensite super.franky.C06.com
   service apache2 restart
   ```
2. Tuliskan `bash ~/11.sh` di `~/.bashrc`.

Pada **Water7** (Proxy Server):
1. Tuliskan script berikut pada `~/11.sh`:
   ```
   # Redirect google.com to super.franky.C06.com
   echo "
   acl banned dstdomain google.com
   deny_info http://super.franky.C06.com/ banned
   http_reply_access deny banned
   " >> /etc/squid/squid.conf
   ```
2. Tuliskan `bash ~/11.sh` di `~/.bashrc`.

### Hasil
#### Membuka `http://super.franky.C06.com/`
![Hasil no.11a](https://user-images.githubusercontent.com/52129348/141285876-57067f98-d4c5-4cd6-a835-a54c6615944e.gif)

#### Redirect `google.com` menuju `http://super.franky.C06.com/`
![Hasil no.11b](https://user-images.githubusercontent.com/52129348/141286191-30442ebd-ccc5-428a-88f3-b84ade387f18.gif)


### No.12-13
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di `super.franky.C06.com`. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps. Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya.

### Jawaban
Pada **Water7** (Proxy Server):
1. Tuliskan script berikut pada `~/12-13.sh`:
   ```
   # Add Squid bandwidth config
   echo "
   acl download url_regex -i  \.png$ \.jpg$

   acl luffy proxy_auth luffybelikapalC06
   acl zoro proxy_auth zorobelikapalC06

   delay_pools 2

   delay_class 1 1
   delay_access 1 deny zoro
   delay_access 1 allow luffy
   delay_access 1 allow download
   delay_access 1 deny all
   delay_parameters 1 10000/64000

   delay_class 2 1
   delay_access 2 allow zoro
   delay_access 2 deny luffy
   delay_access 2 deny all
   delay_parameters 2 -1/-1
   " > /etc/squid/acl-bandwidth.conf

   # Setup Squid bandwidth config
   echo "
   include /etc/squid/acl-bandwidth.conf
   " >> "/etc/squid/squid.conf"

    # Restart Squid
    service squid restart
   ```
2. Tuliskan `bash ~/12-13.sh` di `~/.bashrc`.

### Hasil
#### Akun Luffy
![Hasil no.12](https://user-images.githubusercontent.com/52129348/141288085-30c026aa-f669-4ef2-90fa-9f5b78f00210.gif)

#### Akun Zoro
![Hasil no.13](https://user-images.githubusercontent.com/52129348/141287865-e6caeb7e-dc68-43bd-bb84-ba01efaca1f3.gif)


## Kendala
* GNS project dari Ubuntu mengalami error `Project ID xxx not found` saat diimport ke GNS di Windows.
