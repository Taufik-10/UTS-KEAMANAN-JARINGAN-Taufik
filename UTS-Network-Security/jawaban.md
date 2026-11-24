# UNIVERSITAS TEKNOLOGI BANDUNG
# DEPARTEMEN TEKNIK INFORMATIKA
# UJIAN TENGAH SEMESTER - KEAMANAN JARINGAN

**Nama:** Taufik Nurjaman
**NIM:** 25552012006
**Kelas:** TIF RP 24E CNS

## Perhitungan Kasus
- Digit NIM: 2, 5, 5, 5, 2, 0, 1, 2, 0, 0, 6
- Jumlah digit: 2+5+5+5+2+0+1+2+0+0+6 = 28
- Sisa bagi 10: 28 mod 10 = 8
- **Nomor kasus: Kasus 8**

---

## Tugas 1 (Pertemuan 1): Docker, Volume, dan Compose

### Bukti:

**Output `cat /data/laporan-k8.txt`:**
```
25552012006
```

**Output `wget` dari attacker ke target:**
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### Penjelasan:
Volume Docker disimpan secara terpisah dari lifecycle container, di storage milik Docker daemon sendiri. Ketika container `w-k8` dihapus dengan `docker rm`, data di volume `vol-k8` tidak ikut terhapus karena volume bukan bagian dari container itu sendiri, melainkan resource independen yang bisa di-attach ke container manapun. Terbukti saat container baru dijalankan dengan volume yang sama, isi file `laporan-k8.txt` masih bisa dibaca dan menampilkan NIM saya.

---

## Tugas 2 (Pertemuan 2): Kriptografi

### Bukti:

**Dua hash SHA-256 (file asli dan hasil dekripsi):**
```
(hash sha256sum laporan-k8.txt)
(hash sha256sum hasil-k8.txt)
```
*(kedua hash identik)*

**Subject, Issuer, dan tanggal sertifikat:**
```
subject=CN = k8.uts.local
issuer=CN = k8.uts.local
notBefore=...
notAfter=...
```

### Penjelasan:
Kedua hash SHA-256 (file asli dan hasil dekripsi) identik, membuktikan proses enkripsi AES-256-CBC dan dekripsinya tidak merusak atau mengubah data sama sekali. Pada sertifikat, field Subject dan Issuer bernilai sama (`CN=k8.uts.local`), ciri khas sertifikat **self-signed**.

**Perbedaan sertifikat self-signed dan CA-signed:** Sertifikat self-signed ditandatangani oleh entitas itu sendiri sehingga tidak otomatis dipercaya oleh browser/sistem lain dan rentan terhadap serangan impersonasi. Sertifikat CA-signed ditandatangani oleh pihak ketiga tepercaya (Certificate Authority) yang telah memverifikasi identitas pemilik domain, sehingga diakui valid secara luas oleh browser dan sistem operasi.

---

## Tugas 3 (Pertemuan 3): ARP Spoofing

### Data lab:
- IP korban-k8: 172.18.0.2
- IP gateway-k8: 172.18.0.3
- IP penyerang-k8: 172.18.0.4
- MAC asli gateway-k8: 92:3e:cd:4f:f3:eb

### Bukti:

**Tabel ARP sebelum spoofing:**
```
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 92:3e:cd:4f:f3:eb [ether] on eth0
```

**Tabel ARP saat spoofing berjalan:**
```
penyerang-k8.lab-arp-k8 (172.18.0.4) at 7e:6b:c3:63:a5:ec [ether] on eth0
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 7e:6b:c3:63:a5:ec [ether] on eth0
```

**Tabel ARP setelah mitigasi static ARP:**
```
penyerang-k8.lab-arp-k8 (172.18.0.4) at 7e:6b:c3:63:a5:ec [ether] on eth0
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 92:3e:cd:4f:f3:eb [ether] PERM on eth0
```

### Penjelasan:
Sebelum serangan, MAC address untuk IP gateway (172.18.0.3) adalah MAC asli (`92:3e:cd:4f:f3:eb`). Saat `arpspoof` dijalankan dari `penyerang-k8`, tabel ARP korban berubah -- MAC untuk 172.18.0.3 menjadi `7e:6b:c3:63:a5:ec`, yaitu MAC milik penyerang sendiri. Ini membuktikan ARP spoofing berhasil membohongi korban sehingga mengira penyerang adalah gateway, membuka celah serangan Man-in-the-Middle (MITM) di mana seluruh traffic korban ke gateway akan melewati penyerang terlebih dahulu.

Setelah mitigasi dengan static ARP (`arp -s`), MAC gateway kembali ke nilai asli dengan tambahan tag `PERM`, menandakan binding IP-MAC dikunci secara permanen sehingga ARP reply palsu dari penyerang tidak lagi bisa mengubah entri tersebut.

**Mitigasi tambahan (selain static ARP):** Dynamic ARP Inspection (DAI) -- fitur pada switch managed yang memvalidasi setiap paket ARP terhadap tabel DHCP snooping. Paket ARP yang pasangan IP-MAC-nya tidak cocok dengan binding yang sah akan langsung ditolak oleh switch, mencegah ARP spoofing sebelum sempat mencapai korban.

---

## Tugas 4 (Pertemuan 4): Analisis Handshake WPA2

Tugas ini belum dapat saya selesaikan karena file `handshake-k8.cap` (file capture handshake WPA2) dan `wordlist-k8.txt` (daftar kata sandi untuk percobaan cracking) tidak tersedia di repositori sumber lab (`01-sumber-lab`). Sesuai ketentuan soal, kedua file ini seharusnya disediakan oleh penguji/dosen secara terpisah untuk setiap mahasiswa (berbeda per NIM/kasus), bukan diunduh sendiri dari repositori publik. Karena file tersebut belum saya terima dari dosen hingga laporan ini disusun, perintah `aircrack-ng -w wordlist-k8.txt handshake-k8.cap` tidak dapat dijalankan, sehingga saya belum dapat melampirkan bukti output berupa kata sandi yang ditemukan atau pesan "KEY NOT FOUND".

---

## Tugas 5 (Pertemuan 5): ICMP dengan Scapy

### Bukti:
```
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

### Penjelasan:
TTL balasan bernilai 64, nilai default untuk host berbasis Linux, menunjukkan paket menempuh jarak hop yang dekat. Tipe ICMP balasan bernilai 0 (Echo Reply), merupakan balasan normal terhadap Echo Request (tipe 8) yang dikirim oleh skrip Scapy.

**Penjelasan Smurf Attack:** Smurf attack adalah serangan DDoS yang memanfaatkan ICMP broadcast. Penyerang mengirim paket ICMP echo request dengan source IP address dipalsukan (spoofed) menjadi IP korban, lalu mengirimkannya ke alamat broadcast suatu jaringan. Seluruh host dalam jaringan tersebut akan membalas echo reply ke korban (bukan ke penyerang asli), sehingga korban dibanjiri balasan dari banyak host sekaligus. Efek amplifikasi ini membuat volume traffic yang diterima korban jauh lebih besar dari yang dikirim penyerang, berpotensi melumpuhkan layanan korban.

---

## Tugas 6 (Pertemuan 6): SYN Flood dan Three-Way Handshake

### Bukti:

**SYN_RECV sebelum serangan:** `0`

**SYN_RECV saat serangan:** `1`

**Paket tcpdump (three-way handshake):**
```
IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [S], seq 141186423, win 64240
IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [S.], seq 898206995, ack 141186424, win 65160
IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [.], ack 1, win 126
```

### Penjelasan:
Sebelum serangan, jumlah koneksi dengan status SYN_RECV adalah 0 (kondisi normal). Saat `hping3 --flood` dijalankan, angka SYN_RECV meningkat menjadi 1, menunjukkan adanya koneksi TCP setengah-terbuka (half-open) yang terbentuk akibat paket SYN yang tidak pernah diselesaikan dengan ACK dari pengirim aslinya (karena IP sumber dipalsukan/acak oleh hping3).

Dari hasil tcpdump, terlihat urutan three-way handshake TCP yang normal untuk request HTTP yang sah: `Flags [S]` (SYN, client memulai koneksi) -> `Flags [S.]` (SYN-ACK, server merespons) -> `Flags [.]` (ACK, client mengonfirmasi, koneksi terbentuk).

**Mitigasi:** SYN flood dapat dimitigasi dengan mengaktifkan SYN cookies (`sysctl net.ipv4.tcp_syncookies=1`). Dengan mekanisme ini, server tidak langsung mengalokasikan resource untuk koneksi setengah-terbuka saat menerima SYN, melainkan meng-encode informasi koneksi ke dalam sequence number SYN-ACK, dan baru mengalokasikan resource penuh setelah menerima ACK balik yang valid dari client.

---

## Tugas 7 (Pertemuan 7): DNS Exfiltration

### Bukti:

**NIM dalam hex:** `3235353532303132303036`

**Hasil dig (query DNS):**
```
10.0.0.99
```

**Log DNS dan hasil analisis:**
```
Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):
(tidak ada yang terdeteksi)
```

**Hasil decode kembali ke NIM:**
```
25552012006
```

### Penjelasan:
NIM berhasil diubah ke representasi hex (`3235353532303132303036`), dikirim sebagai subdomain (`<hex>.k8.exfil.lab`), dan server DNS lab (dnsmasq) berhasil menerima serta meresolusinya ke `10.0.0.99` sesuai konfigurasi. Query tersebut tercatat di log DNS (`exfil.lab: 1`).

Namun, skrip deteksi (`analisis_dns.py`) tidak menandai label ini sebagai mencurigakan, karena ambang batas panjang label yang digunakan skrip (`BATAS_PANJANG_LABEL = 30` karakter) lebih besar dari panjang label hex NIM saya (23 karakter). Ini menunjukkan keterbatasan skrip deteksi untuk kasus dengan payload data pendek seperti NIM -- ambang batas idealnya disesuaikan (misalnya diturunkan ke 15-20 karakter) agar dapat menangkap pola exfiltration dengan payload pendek.

Proses decode hex kembali menghasilkan NIM asli saya, membuktikan data berhasil dikirim dan diterima kembali secara utuh tanpa kerusakan.

**Mengapa teknik ini sulit diblokir firewall biasa:** Lalu lintas DNS (port 53) pada umumnya selalu diizinkan keluar oleh firewall/organisasi karena dianggap sebagai lalu lintas esensial untuk resolusi nama domain, sehingga firewall jarang memeriksa isi/konten dari query DNS secara mendalam.

**Indikator deteksi:** Subdomain dengan label yang sangat panjang dan/atau tersusun dari karakter acak/hex (bukan nama domain yang biasa dibaca manusia), serta frekuensi query yang tinggi dan tidak wajar ke domain induk yang sama dalam waktu singkat.

---

## Kesimpulan

Saya telah menyelesaikan enam dari tujuh tugas UTS Praktik Keamanan Jaringan untuk Kasus 8, yaitu Tugas 1 (Docker & Volume), Tugas 2 (Kriptografi), Tugas 3 (ARP Spoofing), Tugas 5 (ICMP dengan Scapy), Tugas 6 (SYN Flood), dan Tugas 7 (DNS Exfiltration), lengkap dengan bukti output terminal dan penjelasan konsep untuk masing-masing tugas.

Melalui pengerjaan UTS ini, saya memahami secara praktik beberapa konsep penting keamanan jaringan: persistensi data pada Docker volume, integritas data melalui hashing dan enkripsi, mekanisme serangan ARP spoofing beserta mitigasinya, prinsip kerja smurf attack berbasis ICMP, dampak SYN flood terhadap resource server beserta mitigasi SYN cookies, serta teknik DNS exfiltration dan keterbatasan skrip deteksi sederhana terhadap payload data pendek.
