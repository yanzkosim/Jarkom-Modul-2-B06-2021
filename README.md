# Jarkom-Modul-2-B06-2021

## Anggota Kelompok : 
- 05111940000048 | Bagaskoro Kuncoro Ardi 
- 05111940000096 | Stefanus Albert Kosim 


### Soal 1

EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet.

Jawab :

Pertama dibuat topology sesuai yang terdapat pada soal

![Topology](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/Topology.png)

Kemudian untuk setiap node diatur network configuration-nya seperti berikut,

- Foosha
 ```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.10.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.10.2.1
	netmask 255.255.255.0
```
- Loguetown
```
auto eth0
iface eth0 inet static
	address 10.10.1.2
	netmask 255.255.255.0
	gateway 10.10.1.1
```
- Alabasta
```
auto eth0
iface eth0 inet static
	address 10.10.1.3
	netmask 255.255.255.0
	gateway 10.10.1.1
```
- EniesLobby
```
auto eth0
iface eth0 inet static
	address 10.10.2.2
	netmask 255.255.255.0
	gateway 10.10.2.1
```
- Water7
```
auto eth0
iface eth0 inet static
	address 10.10.2.3
	netmask 255.255.255.0
	gateway 10.10.2.1
```
- Skypie
```
auto eth0
iface eth0 inet static
	address 10.10.2.4
	netmask 255.255.255.0
	gateway 10.10.2.1
```

Lalu pada foosha dijalankan command
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.10.0.0/16
```
dan
```
cat /etc/resolv.conf
```
Akan diperoleh sebuah IP Adress yang akan digunakan untuk command selanjutnya.

![No1](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No1.png)

Di node-node lainnya, agar dapat terhubung ke internet, dijalankan command berikut dan digunakan IP yang diperoleh tadi.
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

Selanjutnya dilakukan ping google.com untuk memastikan bahwa semua node dapat mengakses internet.

![No1Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No1Ping.png)

### Soal 2

Luffy ingin menghubungi Franky yang berada di EniesLobby dengan denden mushi. Kalian diminta Luffy untuk membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku.

Jawab :

Pada file /etc/bind/named.conf.local di EniesLobby ditambahkan konfigurasi berikut untuk untuk membuat domain
```
zone "franky.B06.com" {
        type master;
        file "/etc/bind/kaizoku/franky.B06.com";
};
```
Dibuat folder kaizoku
```
mkdir /etc/bind/kaizoku
```
Dan pada file /etc/bind/kaizoku/franky.B06.com, dibuat konfigurasi seperti berikut

![No2Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No2Config.png)

Line berikut digunakan untuk membuat alias
```
www             IN      CNAME   franky.B06.com.
```
Dilakukan restart bind9 dan dilakukan setting nameserver pada client Loguetown dan Alabasta
```
nano /etc/resolv.conf
```

![No2Nameserver](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No2Nameserver.png)

setelah itu pada client Loguetown atau Alabasta dilakukan ping franky.B06.com dan www.franky.B06.com untuk memastikan domain berhasil dibuat.

![No2Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No2Ping.png)

### Soal 3

Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

Jawab : 

Di EniesLobby ditambahkan konfigurasi berikut pada file /etc/bind/kaizoku/franky.B06.com.

![No3Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No3Config.png)

Dilakukan restart bind9 dan setelah itu pada client Loguetown atau Alabasta dilakukan ping super.franky.B06.com dan www.super.franky.B06.com untuk memastikan domain berhasil dibuat. (setting nameserver telah dilakukan di soal sebelumnya)

![No3Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No3Ping.png)

### Soal 4

Buat juga reverse domain untuk domain utama.

Jawab :

Di EniesLobby ditambahkan konfigurasi berikut pada file /etc/bind/named.config.local.
```
zone "2.10.10.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.10.10.in-addr.arpa";
};
```
Di-copy file db.local pada /etc/bind ke folder kaizoku yang dibuat sebelumnya.
```
cp /etc/bind/db.local /etc/bind/kaizoku/2.10.10.in-addr.arpa
```
Dan pada file /etc/bind/kaizoku/2.10.10.in-addr.arpa ditambahkan konfigurasi berikut,

![No4Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No4Config.png)

Pada client, dilakukan command berikut untuk pengecekan apakah reverse domain sudah dibuat dengan benar.
```
host -t PTR 10.10.2.4
```
![No4Cek](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No4Cek.png)

### Soal 5

Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama.

Jawab :

Di Water7 ditambahkan konfigurasi berikut pada file /etc/bind/named.config.local.
```
zone "franky.B06.com" {
    type slave;
    masters { 10.10.2.2; };
    file "/var/lib/bind/franky.B06.com";
};
```
Di EniesLobby ditambahkan konfigurasi berikut pada file /etc/bind/named.config.local pada bagian zone "franky.B06.com".
```
notify yes;
also-notify { 10.10.2.3; };
allow-transfer { 10.10.2.3; };
```
Kemudian dilakukan restart bind9 pada Water7 dan EniesLobby.

Lalu dilakukan pengecekan apakah DNS Slave telah berhasil dibuat dengan cara,

dilakukan stop bind pada Enies Lobby, dan dilakukan ping franky.B06.com pada client.

![No5BindStop](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No5BindStop.png)

![No5Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No5Ping.png)

### Soal 6

Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

Jawab :

Di EniesLobby ditambahkan konfigurasi berikut pada file /etc/bind/kaizoku/franky.B06.com.

![No6ConfigEnies](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No6ConfigEnies.png)

Kemudian ditambahkan baris berikut pada file /etc/bind/named.conf.options pada EniesLobby.
```
allow-query{any;};
```
Di Water7 ditambahkan konfigurasi berikut pada file /etc/bind/named.config.local.
```
zone "mecha.franky.B06.com" {
        type master;
        file "/etc/bind/sunnygo/mecha.franky.B06.com";
};
```
Kemudian ditambahkan baris berikut pada file /etc/bind/named.conf.options pada EniesLobby.
```
allow-query{any;};
```
Dibuat folder sunnygo.
```
mkdir /etc/bind/sunnygo
```
Dan pada file /etc/bind/sunnygo/mecha.franky.B06.com, dibuat konfigurasi seperti berikut.

![No6ConfigWater7](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No6ConfigWater7.png)

Dilakukan restart bind pada EniesLobby dan Water7 dan dilakukan testing dengan cara ping ke domain mecha.franky.B06.com dan www.mecha.franky.B06.com pada client.

![No6Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No6Ping.png)

### Soal 7

Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Water7 dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

Jawab :

Pada file /etc/bind/sunnygo/mecha.franky.B06.com, ditambahkan konfigurasi seperti berikut.

![No7Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No7Config.png)

Dilakukan restart bind dan setelah itu pada client Loguetown atau Alabasta dilakukan ping general.mecha.franky.B06.com dan www.general.mecha.franky.B06.com untuk memastikan domain berhasil dibuat. (setting nameserver telah dilakukan di soal sebelumnya)

![No7Ping](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No7Ping.png)

### Soal 8

Dibuat konfigurasi Webserver untuk `www.franky.B06.com` dimana lokasi DocumentRoot nya adalah `/var/www/franky.B06.com`. 

### Jawab

Seperti penjelasan yang ada pada [Soal nomor 1](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021#soal-1), konfigurasi Webserver ini berada pada `Skypie` sehingga semua konfigurasi mulai dari nomor ini hingga nomor terakhir akan berada pada `Skypie`. Sebelum melakukan konfigurasi, maka *utility* yang dibutuhkan akan diunduh terlebih dahulu, yaitu `lynx` pada client dan `apache2`, `php`, `libapache2`, `unzip` dan `wget` pada webserver.

Setelah *utility* berhasil diunduh, copy file `000-default.conf` di direktori `/etc/apache2/sites-available` ke `/etc/apache2/sites-available/franky.B06.com.conf` dengan command

```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/franky.B06.com.conf
```

Lalu file tersebut diedit dengan mengubah DocumentRoot menjadi `/var/www/franky.B06.com` dan menambahkan ServerName dan ServerAlias

```
DocumentRoot /var/www/franky.B06.com
ServerName franky.B06.com
ServerAlias www.franky.B06.com
```

![No8Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No8Config.png)

Dalam proses ini, direktori `/var/www/franky.B06.com` belum ada, maka diunduh terlebih dahulu dari link yang telah disediakan, direname, lalu dipindah ke `/var/www/`

```
wget -P /root/ https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/franky.zip

unzip /root/franky.zip -d /root/
mv /root/franky /root/franky.B06.com
mv /root/franky.B06.com /var/www/
```

Setelah konfigurasi telah selesai, enable site tersebut dengan command

```
a2ensite franky.B06.com
```

![No8Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No8Hasil.png)

### Soal 9

Url `www.franky.B06.com/index.php/home` dapat diakses dengan menggunakan url yang lebih pendek yaitu `www.franky.B06.com/home`.

### Jawab

Digunakan module rewrite untuk menulis ulang atau memanipulasi URL website secara dinamik pada server. Pertama jalankan command untuk menyalakan module rewrite pada apache2

```
a2enmod rewrite
```

Setelah itu buat file `.htaccess` pada `/var/www/franky.B06.com/`, lalu isi file `.htaccess` tersebut sebagai berikut

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([^\.]+)$ index.php/$1 [NC,L]
```

![No9htaccess](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No9htaccess.png)

Agar file tersebut dapat dieksekusi, ditambahkan konfigurasi ke file `/etc/apache2/sites-available/franky.B06.com.conf` sebagai berikut

```
<Directory /var/www/franky.B06.com>
	Options +FollowSymLinks -Multiviews
	AllowOverride All
</Directory>
```

![No9Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No9Config.png)

![No9Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No9Hasil.gif)

### Soal 10

Membuat konfigurasi webserver untuk subdomain `www.super.franky.B06.com` dengan DocumentRoot yang berada pada `/var/www/super.franky.B06.com`

### Jawab

Hal yang perlu dilakukan pertama kali adalah mengunduh file pada link yang telah diberikan, direname, lalu dipindah ke `/var/www/`

```
wget -P /root/ https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/raw/main/super.franky.zip

unzip /root/franky.zip -d /root/
mv /root/super.franky /root/super.franky.B06.com
mv /root/franky.B06.com /var/www/
```

Setelah diunduh dan diipindah, copy file `000-default.conf` ke `super.franky.B06.com.conf`

```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.B06.com.conf
```

Ubah DocumentRoot pada file tersebut lalu tambahkan ServerName dan ServerAlias

```
DocumentRoot /var/www/super.franky.B06.com
ServerName super.franky.B06.com
ServerAlias www.super.franky.B06.com
```

![No10Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No10Config.png)

Agar dapat diakses, enable site tersebut

```
a2ensite super.franky.B06.com
```

![No10Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No10Hasil.png)

### Soal 11

Folder /public diatur agar hanya dapat melakukan directory listing.

### Jawab

Pada file `/etc/apache2/sites-available/super.franky.B06.com.conf`, ditambahi konfigurasi sebagai berikut agar direktori `/public` dapat melakukan directory listing

```
<Directory /var/www/super.franky.B06.com/public>
	Options +Indexes
</Directory>
```

Lalu agar direktori tersebut hanya bisa melakukan directory listing saja, ditambahi konfigurasi agar muncul Error Forbidden ketika dicoba untuk mengakses folder yang ada di dalam direktori `/public` tersebut

```
<Directory /var/www/super.franky.B06.com/public/*>
	Options -Indexes
</Directory>
```

![No11Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No11Config.png)

![No11Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No11Hasil.gif)

### Soal 12

Mengubah tampilan Error 404 default dari apache2 menjadi custom Error 404 yang terdapat pada direktori `/var/www/super.franky.B06.com/error/`

### Jawab

Untuk mengubah tampilan error, ditambahkan konfigurasi *ErrorDocument* pada file konfigurasi `/etc/apache2/sites-available/super.franky.B06.com.conf`

![No12Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No12Config.png)

```
ErrorDocument 404 /error/404.html
```

![No12Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No12Hasil.gif)

### Soal 13

Dibuatkan konfigurasi agar dapat mengakses `www.super.franky.B06.com/public/js` dengan membuka `www.super.franky.B06.com/js`

### Jawab

Digunakan Directory Alias untuk mempersimple URL tersebut.

```
Alias "/js" "/var/www/super.franky.B06.com/public/js"
```

Karena sebelumnya direktori js tidak dapat diakses, perlu diatur lagi dengan menambahkan

```
<Directory /var/www/super.franky.B06.com/public/js>
    Options +Indexes
</Directory>
```

![No13Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No13Config.png)

![No13Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No13Hasil.gif)

### Soal 14

Diatur konfigurasi untuk website `www.general.mecha.franky.B06.com` agar hanya dapat diakses pada port 15000 dan 15500

### Jawab

Untuk mengakses port 15000 dan 15500, kedua port tersebut ditambahkan ke dalam `ports.conf`

```
Listen 15000
Listen 15500
```

![No14Ports](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No14Ports.png)

Setelah itu copy file `000-default.conf` ke `general.mecha.franky.B06.com.conf` dan ubah port VirtualHost menjadi 15000 dan 15500

```
<VirtualHost *:80>
```

menjadi 

```
<VirtualHost *:15000 *:15500>
```
![No14Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No14Config.png)

Kemudian enable site tersebut 

```
a2ensite general.mecha.franky.B06.com
```

![No14Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No14Hasil.gif)

### Soal 15

Tambahkan authentikasi untuk mengakses web tersebut, dan atur DocumentRoot nya pada `/var/www/general.mecha.franky.B06`

### Jawab

Untuk menambahkan authentikasi, digunakan *htpasswd*. Pertama buat file `.htpasswd` pada direktori `/etc/apache2/`, lalu tuliskan hasil htpasswd dengan username **luffy** dan password **onepiece** ke dalam file `.htpasswd` tersebut.

```
touch /etc/apache2/.htpasswd
htpasswd -nb luffy onepiece > /etc/apache2/.htpasswd
```

Ubah DocumentRoot dan tambahkan konfigurasi pada file `/etc/apache2/sites-available/general.mecha.franky.B06.com.conf`

```
DocumentRoot /var/www/general.mecha.franky.B06
ServerName general.mecha.franky.B06.com
ServerAlias www.general.mecha.franky.B06.com

<Directory /var/www/general.mecha.franky.B06>
	AuthType Basic
	AuthName "Restricted Content"
	AuthUserFile /etc/apache2/.htpasswd
	Require valid-user
</Directory>
```

![No15Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No15Config.png)

![No15Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No15Hasil.gif)

### Soal 16

Jika mengakses IP Skypie, maka akan diarahkan ke `www.franky.B06.com`

### Jawab

Agar diarahkan hanya jika mengakses IP Skypie saja, maka digunakan modul rewrite, namun jika ingin menjadikan `www.franky.B06.com` menjadi site default, digunakan *Redirect*. 

Pertama tambahkan konfigurasi agar `.htacces` dapat digunakan pada file `000-default.conf`

```
<Directory /var/www/html>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```

![No16Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No16Config.png)

Lalu buat file `.htaccess` pada direktori default yaitu `/var/www/html` dan diisi seperti ini

```
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^10\.10\.2\.4$
RewriteRule ^(.*)$ http://franky.b06.com/$1 [L,R=301]
```

![No16htaccess](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No16htaccess.png)

Keterangan :

Jika HTTP Host yang diterima adalah IP Skypie (10.10.2.4), maka url nya ditulis ulang menjadi `http://franky.b06.com`

![No16Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No16Hasil.gif)

### Soal 17

Setiap file dengan format gambar akan dialihkan ke `franky.png` jika nama file terebut memiliki substring ***franky***.

### Jawab

Untuk mengalihkan ke `franky.png` digunakan module rewrite yaitu dengan membuat dan mengisi file `/var/www/super.franky.B06.com/public/images/.htaccess` sebagai berikut

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*)franky(.*)(png|jpg) http://super.franky.b06.com/public/images/franky.png
```

![No17htaccess](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No17htaccess.png)

Lalu tambahkan konfigurasi pada `/etc/apache2/sites-available/super.franky.B06.com.conf` agar mod rewritenya dapat digunakan dan direktori imagesnya dapat diakses

```
<Directory /var/www/super.franky.B06.com/public/images>
    Options +FollowSymLinks -Multiviews +Indexes
    AllowOverride All
</Directory>
```

![No17Config](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No17Config.png)

![No17Hasil](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No17Hasil.gif)

## Catatan

Selalu jalankan command berikut mulai dari nomor 8 hingga nomor terakhir apabila telah dilakukan konfigurasi

```
service apache2 restart
```