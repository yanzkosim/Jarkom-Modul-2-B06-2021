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
