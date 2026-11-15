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

> Tempel screenshot tabel VLSM dan tabel CIDR dari file Excel.

![Tabel VLSM](images/tabel-vlsm.png)

![Pohon penggabungan CIDR](images/pohon-cidr.png)

![Tabel CIDR](images/tabel-cidr.png)

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

### Soal 1 - Blok alamat dan kebutuhan LAN

> Screenshot tabel kebutuhan: jumlah host, kebutuhan, prefix, ukuran blok, subnet mask.

![Soal 1](images/soal1.png)

### Soal 2 - Rancangan kasar topologi

> Screenshot topologi (sama dengan bagian 1, atau gambar rancangan kasar).

![Soal 2](images/soal2.png)

### Soal 3 - VLSM

**a. Tabel alokasi VLSM**

![Soal 3 tabel](images/soal3-tabel.png)

**b. Status interface** (`show ip interface brief`), tempel satu screenshot per router:

![R-PUSAT](images/vlsm-int-pusat.png) ![R-DC](images/vlsm-int-dc.png)
![R-UTARA](images/vlsm-int-utara.png) ![R-BARAT](images/vlsm-int-barat.png)
![R-TIMUR](images/vlsm-int-timur.png) ![R-SELATAN](images/vlsm-int-selatan.png)
![R-PELABUHAN](images/vlsm-int-pelabuhan.png)

**c. Tabel routing** (`show ip route` di R-PUSAT)

![VLSM route](images/vlsm-route.png)

**d. Ping antar LAN**

| Dari         | Ke           | Perintah            | Hasil          |
| ------------ | ------------ | ------------------- | -------------- |
| PC-WiFiBarat | PC-SCADA     | `ping 10.120.160.2` | (isi: 0% loss) |
| PC-CCTV      | PC-Pegawai   | `ping 10.120.224.2` | (isi)          |
| PC-PJU       | PC-WiFiTimur | `ping 10.120.128.2` | (isi)          |
| PC-KantorPel | PC-WiFiUtara | `ping 10.120.0.2`   | (isi)          |

![Ping VLSM 1](images/vlsm-ping1.png) ![Ping VLSM 2](images/vlsm-ping2.png)

### Soal 4 - CIDR / Supernetting

**a. Pohon penggabungan**

![Soal 4a](images/soal4a.png)

**b. Tabel alokasi CIDR**

![Soal 4b](images/soal4b.png)

**c. Status interface dan tabel routing**

![CIDR interface](images/cidr-int.png) ![CIDR route](images/cidr-route.png)

**d. Ping antar LAN**

| Dari         | Ke           | Perintah            | Hasil |
| ------------ | ------------ | ------------------- | ----- |
| PC-WiFiUtara | PC-SCADA     | `ping 10.121.128.2` | (isi) |
| PC-CCTV      | PC-PJU       | `ping 10.122.96.2`  | (isi) |
| PC-WiFiBarat | PC-WiFiTimur | `ping 10.122.0.2`   | (isi) |

![Ping CIDR](images/cidr-ping.png)

### Soal 5 - Perbandingan efisiensi

**a. Prefix terkecil, persentase pemakaian, dan alamat terbuang**

| Metode | Prefix | Total alamat | Terpakai | Pemakaian | Terbuang |
| ------ | ------ | ------------ | -------- | --------- | -------- |
| VLSM   | /16    | 65.536       | 61.752   | 94,23%    | 3.784    |
| CIDR   | /13    | 524.288      | 61.752   | 11,78%    | 462.536  |

**b. Kesimpulan:** VLSM lebih efisien karena setiap subnet mendapat blok sesuai kebutuhan, sedangkan
CIDR membulatkan ukuran ke pangkat 2 di tiap tingkat penggabungan sehingga banyak alamat kosong.

---
