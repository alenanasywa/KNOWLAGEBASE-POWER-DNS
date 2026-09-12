

# Instalasi dan Konfigurasi DNS Server Menggunakan PowerDNS pada Ubuntu 24.04 LTS

Pelajari cara instalasi dan konfigurasi DNS Server menggunakan BIND di Kilat VM 2.0 berbasis Ubuntu 18.04. Panduan lengkap mulai dari persiapan, setup Glue Record, hingga konfigurasi zona domain.
***

Halo, Kawan Belajar!

DNS atau Domain Name System merupakan sistem yang digunakan untuk menerjemahkan nama domain menjadi IP Address. Dengan adanya DNS, pengguna dapat mengakses layanan menggunakan nama domain tanpa harus mengingat IP Address dari server.

Salah satu aplikasi yang dapat digunakan sebagai DNS Server adalah **PowerDNS**. PowerDNS merupakan DNS Server yang digunakan untuk menangani permintaan DNS dan mengelola informasi domain melalui DNS Zone dan DNS Record.

Pada panduan kali ini, akan dilakukan instalasi dan konfigurasi **PowerDNS pada Kilat VM 2.0** dengan sistem operasi Ubuntu. Konfigurasi dilakukan menggunakan domain sendiri dengan nameserver ns1 dan ns2 sehingga diperlukan Glue Record.

# 1. Pengenalan


**PowerDNS** adalah perangkat lunak DNS Server yang digunakan untuk mengelola dan melayani permintaan DNS pada sebuah domain. PowerDNS bertugas menerjemahkan nama domain menjadi informasi yang dibutuhkan, seperti alamat IP, sehingga domain dapat diakses oleh pengguna.

**Cara kerja PowerDNS** secara sederhana adalah ketika pengguna mengakses suatu domain, permintaan DNS akan diteruskan ke **Nameserver** yang menggunakan PowerDNS. PowerDNS kemudian mencari informasi domain pada **DNS Zone** dan **DNS Record** yang telah dikonfigurasi, lalu mengembalikan hasilnya kepada pengguna.

**Fungsi PowerDNS** antara lain:
* Menjadi DNS Server untuk sebuah domain.
* Mengelola DNS Zone dan DNS Record.
* Menjawab permintaan DNS dari client.
* Mendukung berbagai jenis DNS Record seperti A, AAAA, CNAME, MX, NS, dan TXT.
* Dapat menggunakan database sebagai tempat penyimpanan data DNS, tergantung backend yang digunakan.

# 2. Persiapan

Untuk melakukan instalasi dan konfigurasi DNS Server menggunakan PowerDNS, beberapa kebutuhan yang perlu disiapkan antara lain:
* Domain dan Kilat VM 2.0 aktif.
* Sistem operasi Ubuntu 24.04.
* Setup Glue Record.

Jika belum memiliki Kilat VM 2.0, pengguna dapat melakukan pemesanan layanan Kilat VM 2.0 melalui CloudKilat.
# 3. Instalasi dan Konfigurasi
### a. Setup Glue Record
Sebelum melakukan konfigurasi PowerDNS, pastikan domain telah memiliki **Glue Record** apabila menggunakan nameserver sendiri.

Glue Record merupakan informasi IP Address yang digunakan oleh suatu nameserver dan didaftarkan pada registrar domain.


Adapun panduan cara setup Glue Record seperti berikut ini:

1. [Login Portal Client Area CloudKilat](https://portal.cloudkilat.com/clientarea) terlebih dahulu.
2. Untuk langkah-langkah lengkapnya, Anda dapat mengikuti panduan resmi melalui tautan [Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat).
### b. Install dan Konfigurasi PowerDNS
Selanjutya, untuk melakukan instalasi dan konfigurasi DNS Server, silakan mengikuti langkah-langkah berikut ini:

Masuk ke Kilat VM 2.0 Anda terlebih dahulu, atau Anda juga bisa melakukan *remote* menggunakan SSH. Jika Anda masih belum mengetahui cara *remote* menggunakan SSH, silakan membaca panduannya melalui tautan [Cara Akses Kilat VM Melalui SSH](https://kb.cloudkilat.id/akses-kilat-vm/cara-akses-kilat-vm-melalui-ssh).

Setelah berhasil masuk ke dalam Kilat VM 2.0, perbarui paket sistem ke versi terbaru dengan menjalankan perintah berikut seperti pada Gambar 1:

```
apt update -y
```

<p align="center">
  <img width="1352" height="555" alt="apt update" src="https://github.com/user-attachments/assets/5fa107d5-5933-4f65-b352-50bca91f8a1e" />
  <br>
  <em>Gambar 1: Update Paket Ubuntu Server</em>
</p>

### Instalasi PowerDNS

Tunggu proses update hingga benar-benar selesai, dan selanjutnya install paket PowerDNS menggunakan perintah : 

```
apt install pdns-server
```

tekan **Y** apabila diminta untuk melanjutkan proses instalasi seperti pada Gambar 2:
<p align="center">
  <img width="1202" height="241" alt="install powerdns (y)" src="https://github.com/user-attachments/assets/222dbb7f-2e98-4efb-ae31-1da58e6d8bd5" />
  <br>
  <em>Gambar 2: Instalasi PowerDNS</em>
</p>

Setelah proses instalasi selesai, Anda dapat memeriksa versi PowerDNS yang terinstal menggunakan perintah sebagai berikut:

```
pdns_server --version
```
<p align="center">
  <img width="1360" height="347" alt="versi powerdns" src="https://github.com/user-attachments/assets/8f2ffb58-6ae8-4c6c-9759-8e52315b8220" />
  <br>
  <em>Gambar 3: Versi PowerDNS</em>
</p>

Kemudian, aktifkan dan pastikan service PowerDNS berjalan dengan baik menggunakan perintah status berikut, dan pastikan statusnya bernilai active (running):

```
systemctl status pdns
```
### Instalasi Database Backend

PowerDNS dapat menggunakan berbagai jenis *backend* untuk menyimpan data DNS. Pada praktik ini, digunakan MariaDB sebagai *database backend*.

Instal MariaDB dengan menjalankan perintah berikut:
```
apt install mariadb-server
```
tekan **Y** apabila diminta untuk melanjutkan proses instalasi seperti pada Gambar 4:

Setelah instalasi selesai, cek service MariaDB untuk memastikan berjalan dengan normal:
```
systemctl status mariadb
```

Selanjutnya, install backend MariaDB untuk PowerDNS menggunakan perintah berikut:
```
apt install pdns-backend-mysql
```

### Membuat Database PowerDNS

Masuk atau *login* ke MariaDB sebagai pengguna *root* dengan menjalankan perintah berikut:
```
mysql -u root -p
```

Kemudian, buat sebuah database baru untuk PowerDNS menggunakan perintah:
```
CREATE DATABASE powerdns;
```

Buat user baru untuk mengakses database tersebut (pastikan mengganti 'PASSWORD' dengan kata sandi yang Anda inginkan):
```
CREATE USER 'powerdns'@'localhost' IDENTIFIED BY 'PASSWORD';
```

Berikan hak akses penuh kepada user tersebut pada database PowerDNS:
```
GRANT ALL PRIVILEGES ON powerdns.* TO 'powerdns'@'localhost';
```

Terakhir, perbarui hak istimewa (privileges) dengan menjalankan perintah:
```
FLUSH PRIVILEGES;
```

### Import Database Schema PowerDNS

Setelah *backend* MariaDB PowerDNS berhasil diinstal, cari file *schema* yang tersedia pada sistem dengan menjalankan perintah berikut:
```
ls /usr/share/doc/pdns-backend-mysql/
```

Kalau ingin mencari file SQL secara lebih spesifik, Anda bisa menggunakan perintah:
```
find /usr/share -type f -iname "*.sql" | grep -i pdns
```

Maka lakukan import menggunakan path tersebut
```
mysql -u powerdns -p powerdns < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql
```
(Saat diminta password, masukkan password user powerdns yang sudah Anda buat sebelumnya).

Setelah proses selesai, silakan login kembali ke database untuk memastikan tabel-tabelnya sudah terbuat:
```
mysql -u powerdns -p powerdns
```

Kemudian cek tabel untuk memastikan schema berhasil di-import dan tabel-tabel yang dibutuhkan PowerDNS muncul:
```
SHOW TABLES;
```

### Konfigurasi Backend PowerDNS

Nah, setelah *schema* masuk ke *database*, baru kita beri tahu PowerDNS bahwa data DNS disimpan di dalam *database* MariaDB.

Edit file konfigurasi utama PowerDNS dengan menggunakan editor teks `nano`:
```
nano /etc/powerdns/pdns.conf
```

Tambahkan atau sesuaikan konfigurasi backend database di dalam file tersebut seperti berikut:
```
launch=gmysql
gmysql-host=127.0.0.1
gmysql-user=powerdns
gmysql-password=password
gmysql-dbname=powerdns
```

Setelah konfigurasi disimpan, restart service PowerDNS untuk menerapkan perubahan:
```
systemctl restart pdns
```

Kemudian cek kembali status service PowerDNS untuk memastikan semuanya berjalan dengan normal:
```
systemctl status pdns
```
Pastikan statusnya menunjukkan keterangan active (running):

### Membuat DNS Zone

Setelah *backend* berhasil dikonfigurasi, langkah selanjutnya adalah membuat DNS *Zone* untuk domain yang akan digunakan.

*Login* kembali ke *database* PowerDNS menggunakan perintah berikut:
```
mysql -u powerdns -p powerdns
```

Kemudian buat zone domain baru dengan memasukkan query SQL berikut
```
INSERT INTO domains (name, type) VALUES ('domainkamu.id', 'NATIVE');
```

Setelah itu, cek apakah zone tersebut sudah berhasil tersimpan dengan menjalankan perintah:
```
SELECT * FROM domains;
```

### Menambahkan DNS Record

Setelah DNS *Zone* berhasil dibuat, langkah berikutnya adalah menambahkan berbagai macam DNS *Record* yang diperlukan ke dalam *database*.
#### A Record
*A Record* digunakan untuk mengarahkan domain utama ke alamat IP *server* Anda:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES (1, 'domainkamu.id', 'A', 'IP_SERVER', 3600);
```

#### NS Record
Tambahkan nameserver yang akan digunakan oleh domain Anda:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'domainkamu.id', 'NS', 'ns1.domainkamu.id', 3600),
(1, 'domainkamu.id', 'NS', 'ns2.domainkamu.id', 3600);
```

#### A Record untuk Nameserver
Karena nameserver yang digunakan merupakan child nameserver, tambahkan A Record untuk masing-masing nameserver tersebut:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'ns1.domainkamu.id', 'A', 'IP_SERVER', 3600),
(1, 'ns2.domainkamu.id', 'A', 'IP_SERVER', 3600);
```

#### CNAME Record
Jika Anda ingin mengarahkan subdomain www ke domain utama, tambahkan CNAME Record berikut:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'www.domainkamu.id', 'CNAME', 'domainkamu.id', 3600);
```
### Mengecek DNS Zone dan Record
Setelah seluruh *record* ditambahkan ke dalam *database*, langkah terakhir adalah memeriksa kembali seluruh data yang telah dibuat untuk memastikan semuanya sudah terkonfigurasi dengan benar.

Jalankan *query* SQL berikut di dalam MariaDB:
```
SELECT name, type, content, ttl FROM records;
```

Pastikan record yang muncul sudah sesuai dengan konfigurasi.
```
domainkamu.id          A       IP_SERVER
domainkamu.id          NS      ns1.domainkamu.id
domainkamu.id          NS      ns2.domainkamu.id
ns1.domainkamu.id      A       IP_SERVER
ns2.domainkamu.id      A       IP_SERVER
www.domainkamu.id      CNAME   domainkamu.id
```

### Restart PowerDNS
Setelah seluruh konfigurasi dan penambahan record selesai, lakukan restart terakhir pada service PowerDNS untuk menerapkan semua pembaruan secara sempurna, lalu pastikan kembali bahwa service telah berjalan dengan normal dan stabil:
```
#systemctl restart pdns
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-12 21:36:42 WIB; 7s ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 329879 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 47.8M (peak: 48.0M)
        CPU: 171ms
     CGroup: /system.slice/pdns.service
```

# 4. Verifikasi
### Mengecek Port DNS
Layanan DNS menggunakan port `53` (baik protokol UDP maupun TCP) untuk menerima setiap *query* atau permintaan DNS yang masuk dari klien.

Untuk mengecek apakah port tersebut sudah digunakan dan aktif oleh PowerDNS, jalankan perintah berikut di terminal:
```
#ss -lntup | grep :53
udp   UNCONN 0      0            0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=329879,fd=5))
udp   UNCONN 0      0               [::]:53           [::]:*    users:(("pdns_server",pid=329879,fd=6))
tcp   LISTEN 0      128          0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=329879,fd=7))
tcp   LISTEN 0      128             [::]:53           [::]:*    users:(("pdns_server",pid=329879,fd=8))
```

### Pengujian DNS Server Menggunakan dig

Setelah layanan PowerDNS aktif dan seluruh konfigurasi selesai, lakukan pengujian fungsionalitas DNS menggunakan utilitas `dig`.

Pertama, lakukan pengujian *query* terhadap DNS yang telah dibuat dengan perintah:
#### Cek DNS Zone
```
#dig @IP_SERVER domainkamu.id
```
Jika berhasil, akan muncul bagian **ANSWER SECTION** yang berisi IP Address domain.

#### Cek NS Record
```
#dig @IP_SERVER domainkamu.id NS
domainkamu.id.    NS    ns1.domainkamu.id.
domainkamu.id.    NS    ns2.domainkamu.id.
```
#### Pengujian Nameserver dari Domain
```
#dig domainkamu.id NS
ns1.domainkamu.id
ns2.domainkamu.id
```

#### Pengujian Domain
```
#dig domainkamu.id +short
IP_SERVER
```

### Verifikasi Menggunakan DNS Checker
Apabila hasil *output* IP Address sudah mengarah ke IP Address *server* yang digunakan, maka hasil *pointing* domain sudah *resolved*.

Selain itu, Anda juga dapat memeriksa hasil *pointing* lebih lanjut menggunakan *tools* berbasis web seperti [DNS Checker](https://dnschecker.org/). Jika sudah resolved semua, maka akan ditandai dengan centang warna hijau secara keseluruhan pada tool DNS Checker seperti pada Gambar 26.

GAMBAR

Apabila dari hasil verifikasi, hasil pointing domain masih belum mengarah ke IP Address server yang digunakan atau masih belum terdapat tanda centang hijau secara keseluruhan pada tool DNS Checker, biasanya hal tersebut masih berada dalam proses propagasi.

> _Propagasi adalah waktu yang dibutuhkan oleh internet atau ISP untuk mengenali record-record DNS yang baru pada sebuah domain (biasanya dibutuhkan ketika terjadi perubahan pada record-record DNS). Pada saat proses propagasi berlangsung, domain terkadang akan mengalami anomali ketika diakses._
>
>> _Proses propagasi ini dipengaruhi oleh beberapa faktor, yaitu pengaturan TTL (Time to Live), jaringan ISP, serta pihak Registry domain. Waktu yang dibutuhkan untuk proses propagasi ini biasanya memakan waktu kurang lebih hingga 48 jam._

# Kesimpulan
Dengan memahami konsep dan cara kerja PowerDNS, kamu bisa membangun layanan DNS Authoritative pada VPS secara lebih fleksibel dan terkelola. Dengan dukungan MariaDB sebagai backend, konfigurasi zone dan record DNS dapat disimpan serta dikelola dengan lebih terstruktur sesuai kebutuhan.

Setelah PowerDNS berhasil dikonfigurasi, kamu dapat mengelola domain, nameserver, serta DNS record melalui database dan melakukan pengecekan untuk memastikan layanan DNS berjalan dengan baik.

Sekian, dan semoga bermanfaat.


## Inline code

This web site is using `markedjs/marked`.
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/1439c605-b827-48d5-9f25-b1ba3a784a03" />


