# Subnetting VLSM dan CIDR - Smart City Pemerintah Kota Surabaya Project UTS KOMDATJARKOM Kelas A

**Mata kuliah:** Komunikasi Data dan Jaringan Komputer (UTS Gasal 2026/2027) **Kelompok:** 15
**Anggota:**

| Nama                 | NRP        | Bagian |
| -------------------- | ---------- | ------ |
| Hasheemi Rafsanjani  | 5027251015 | CPT    |
| Sebastian Elroi H.P. | 5027251040 | CIDR   |
| Razana Aulia         | 5027251127 | VLSM   |

**Blok IP private:** oktet ke-2 = 8 x 15 = 120, sehingga blok dasar `10.120.0.0/16` (VLSM) dan
supernet `10.120.0.0/13` (CIDR).

---

## 1. Gambar Topologi

<img width="945" height="546" alt="image" src="https://github.com/user-attachments/assets/0d5e36db-61f0-4b5e-ace5-f7edd49931f0" />

**Ringkasan topologi:** R-PUSAT (Balai Kota) menjadi pusat dan terhubung langsung ke R-DC, R-BARAT,
R-UTARA, R-TIMUR, dan R-SELATAN. R-PELABUHAN terhubung ke R-UTARA. Setiap router memiliki dua LAN
(sesuai tabel soal). Satu PC mewakili satu LAN.

---

## 2. Alasan Pemilihan Skema

Skema yang dipakai untuk pengalamatan adalah **VLSM**, sedangkan **CIDR (supernetting)** dikerjakan
sebagai pembanding.

| Metode                    | Prefix | Total alamat | Alamat terpakai | Pemakaian |
| ------------------------- | ------ | ------------ | --------------- | --------- |
| VLSM                      | /16    | 65.536       | 61.752          | 94,23%    |
| CIDR (pohon penggabungan) | /13    | 524.288      | 61.752          | 11,78%    |

**Alasan:**

- VLSM memberi setiap subnet blok sesuai kebutuhan hostnya, sehingga muat dalam satu blok /16 dan
  alamat yang terbuang sedikit.
- CIDR menggabungkan blok secara berpasangan, dan ukuran dibulatkan ke pangkat 2 di setiap tingkat,
  sehingga muncul banyak ruang kosong.
- Kelebihan CIDR ada di tabel routing, karena beberapa subnet dapat diringkas menjadi satu rute.

---

## 3. Penjelasan Desain Topologi

### 3.1 Kebutuhan host per LAN

| Router      | LAN                         | Host   |
| ----------- | --------------------------- | ------ |
| R-PUSAT     | Pegawai Balai Kota          | 1.800  |
| R-PUSAT     | Server Balai Kota           | 20     |
| R-DC        | CCTV Kota                   | 6.000  |
| R-DC        | Server dan Storage DC       | 180    |
| R-UTARA     | WiFi Publik Utara           | 11.000 |
| R-UTARA     | Sensor Lalu Lintas Utara    | 450    |
| R-PELABUHAN | SCADA Pelabuhan             | 5.000  |
| R-PELABUHAN | Kantor Pelabuhan            | 120    |
| R-SELATAN   | WiFi Publik Selatan         | 6.500  |
| R-SELATAN   | PJU Pintar Selatan          | 90     |
| R-TIMUR     | WiFi Publik Timur           | 5.500  |
| R-TIMUR     | Sensor Banjir Timur         | 700    |
| R-BARAT     | WiFi Publik Barat           | 4.500  |
| R-BARAT     | Puskesmas dan Sekolah Barat | 220    |

### 3.2 Cara menentukan prefix

- Bit host = pembulatan ke atas dari log2(host + 2).
- Prefix = 32 - bit host.
- Subnet VLSM diurutkan dari host terbesar ke terkecil agar blok selalu rata.

### 3.3 Link antar router

Setiap link memakai /30 (2 host). Link tidak ada di tabel soal, tetapi dibutuhkan agar router saling
terhubung.

| Link | Antara                  | Port sisi 1 | Port sisi 2 |
| ---- | ----------------------- | ----------- | ----------- |
| L1   | R-PUSAT dan R-DC        | gi7/0       | fa0/0       |
| L2   | R-PUSAT dan R-UTARA     | fa0/0       | fa0/0       |
| L3   | R-PUSAT dan R-SELATAN   | gi8/0       | fa0/0       |
| L4   | R-PUSAT dan R-TIMUR     | gi6/0       | fa0/0       |
| L5   | R-PUSAT dan R-BARAT     | fa1/0       | fa0/0       |
| L6   | R-UTARA dan R-PELABUHAN | fa1/0       | fa0/0       |

### 3.4 Tabel alokasi

Tabel Pembagian IP VLSM

<img width="1755" height="568" alt="image" src="https://github.com/user-attachments/assets/4a145533-abf4-402b-a5a4-8c147e15a4e0" />


Tabel Pohon Penggabungan CIDR

<img width="700" height="525" alt="image" src="https://github.com/user-attachments/assets/c108f53c-ee50-494e-a984-c11a3d074db0" />

Tabel Pembagian IP CIDR

<img width="1359" height="415" alt="image" src="https://github.com/user-attachments/assets/dca4519f-2f21-498f-81f2-b593b726b86d" />


---

## 4. Penjelasan Konfigurasi

File konfigurasi lengkap tiap router:

- [`K15_VLSM_OSPF.txt`](K15_VLSM_OSPF.txt)
- [`K15_CIDR_OSPF.txt`](K15_VLSM_CIDR.txt)

### 4.1 Contoh konfigurasi (R-PUSAT, VLSM)

```
enable
configure terminal
hostname R-PUSAT
interface gi7/0
 description L1 ke R-DC
 ip address 10.120.241.33 255.255.255.252
 no shutdown
interface gi9/0
 description LAN Pegawai
 ip address 10.120.224.1 255.255.248.0
 no shutdown
exit
router rip
 version 2
 network 10.0.0.0
 no auto-summary
end
write memory
```

### 4.2 Arti tiap perintah

| Perintah                 | Fungsi                                                     |
| ------------------------ | ---------------------------------------------------------- |
| `hostname R-PUSAT`       | Memberi nama router                                        |
| `interface gi7/0`        | Masuk ke konfigurasi port                                  |
| `description ...`        | Catatan tujuan port                                        |
| `ip address <IP> <mask>` | Memberi IP pada port; untuk LAN, IP ini menjadi gateway    |
| `no shutdown`            | Menyalakan port                                            |
| `router rip`             | Mengaktifkan routing dinamis RIP                           |
| `version 2`              | RIPv2, mendukung VLSM dan CIDR karena mengirim subnet mask |
| `network 10.0.0.0`       | Mengiklankan semua jaringan di blok 10.x.x.x               |
| `no auto-summary`        | Mematikan ringkasan otomatis, wajib untuk VLSM dan CIDR    |
| `write memory`           | Menyimpan konfigurasi                                      |

### 4.3 Konfigurasi PC

Setiap PC memakai IP statis. IP PC adalah alamat usable kedua (`.2`), gateway adalah IP router pada
LAN tersebut (`.1`). Contoh konfigurasi node pc untuk VLSM

| PC              | IP (VLSM)      | Mask            | Gateway        |
| --------------- | -------------- | --------------- | -------------- |
| PC-Pegawai      | 10.120.224.2   | 255.255.248.0   | 10.120.224.1   |
| PC-ServerBK     | 10.120.241.2   | 255.255.255.224 | 10.120.241.1   |
| PC-CCTV         | 10.120.96.2    | 255.255.224.0   | 10.120.96.1    |
| PC-StorageDC    | 10.120.239.2   | 255.255.255.0   | 10.120.239.1   |
| PC-WiFiUtara    | 10.120.0.2     | 255.255.192.0   | 10.120.0.1     |
| PC-SensorLalin  | 10.120.236.2   | 255.255.254.0   | 10.120.236.1   |
| PC-SCADA        | 10.120.160.2   | 255.255.224.0   | 10.120.160.1   |
| PC-KantorPel    | 10.120.240.2   | 255.255.255.128 | 10.120.240.1   |
| PC-WiFiSelatan  | 10.120.64.2    | 255.255.224.0   | 10.120.64.1    |
| PC-PJU          | 10.120.240.130 | 255.255.255.128 | 10.120.240.129 |
| PC-WiFiTimur    | 10.120.128.2   | 255.255.224.0   | 10.120.128.1   |
| PC-SensorBanjir | 10.120.232.2   | 255.255.252.0   | 10.120.232.1   |
| PC-WiFiBarat    | 10.120.192.2   | 255.255.224.0   | 10.120.192.1   |
| PC-Puskesmas    | 10.120.238.2   | 255.255.255.0   | 10.120.238.1   |

---

## 5. Bukti Pengujian Mandiri
Uji Ping VLSM

<img width="756" height="186" alt="image" src="https://github.com/user-attachments/assets/9fadf17c-4917-4647-aa3f-d9a5448f7e37" />

Uji PDU

<img width="775" height="156" alt="image" src="https://github.com/user-attachments/assets/cd72ba77-f57b-4682-878d-6e2c7f54d40f" />


## Soal 8

Layanan aplikasi Smart City memakai domain internal **k15.smartcity.lab** dan berjalan di dua Server-PT. **Server-DC** (LAN Server dan Storage DC) menjalankan DNS, FTP, dan email: DNS memiliki record A untuk `www`, `ftp`, dan `mail` yang dapat diakses dari seluruh wilayah, FTP dengan akun login `cctv` dipakai PC-CCTV untuk mengunggah rekaman, dan layanan email (SMTP dan POP3) menampung akun `pegawai1` dan `pelabuhan1` agar pengguna Balai Kota dan Pelabuhan dapat saling mengirim dan menerima email. **Server-BK** (LAN Server Balai Kota) menjalankan layanan HTTP berisi halaman "Portal Smart City Surabaya" yang mencantumkan nomor kelompok dan dapat dibuka dengan `www.k15.smartcity.lab` dari semua wilayah. Seluruh PC memakai DNS Server milik Server-DC, dan pengujian dilakukan dengan `ping` dan `nslookup` nama domain, membuka web dari tiap wilayah, mengunggah file lewat FTP, serta mengirim email dua arah.

<img width="893" height="354" alt="Screenshot 2026-11-15 212056" src="https://github.com/user-attachments/assets/1f87647f-47e8-4977-905a-777e0877b56c" />
<img width="926" height="356" alt="Screenshot 2026-11-15 212255" src="https://github.com/user-attachments/assets/841eb19f-7db4-4fc2-8f64-101c57c7948a" />
<img width="530" height="298" alt="Screenshot 2026-11-15 212351" src="https://github.com/user-attachments/assets/c49b1ac2-bd71-4a8a-8451-de6efbb588db" />
<img width="851" height="305" alt="Screenshot 2026-11-15 212513" src="https://github.com/user-attachments/assets/f7180e89-22fc-443d-a5ac-7711ffa57938" />

<img width="919" height="379" alt="Screenshot 2026-11-15 212748" src="https://github.com/user-attachments/assets/2da97604-b075-4258-947c-edad76fca2f3" />



