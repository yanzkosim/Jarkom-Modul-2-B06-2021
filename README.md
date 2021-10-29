# Jarkom-Modul-2-B06-2021

## Anggota Kelompok : 
- 05111940000048 | Bagaskoro Kuncoro Ardi 
- 05111940000096 | Stefanus Albert Kosim 


### Soal 1

EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet.

Jawab

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

![No1](https://github.com/yanzkosim/Jarkom-Modul-2-B06-2021/blob/main/Screenshot/No1Ping.png)
