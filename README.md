# Antarmuka Web — Penyiraman Otomatis

Halaman web untuk mengendalikan sistem penyiraman langsung dari browser, tanpa
memasang aplikasi. Berisi tombol **Hidupkan Pompa**, **Matikan Pompa**, dan
**Mode Otomatis**, ditambah panel kondisi sensor dan pengaturan ambang batas.

Komunikasinya memakai **Web Bluetooth**: halaman web berbicara langsung ke modul
BLE pada Arduino. Tidak ada server, tidak ada internet, tidak ada akun.

---

## 1. Prasyarat

### Browser

| Sistem | Browser | Dukungan |
|---|---|---|
| macOS | Chrome, Edge | **ya** |
| Windows | Chrome, Edge | **ya** |
| Android | Chrome, Edge, Samsung Internet | **ya** |
| macOS | Safari | tidak |
| iPhone / iPad | semua browser | tidak |
| semua | Firefox | tidak |

Safari belum mendukung Web Bluetooth, dan seluruh browser di iOS wajib memakai
mesin Safari. Jadi halaman ini **tidak dapat dipakai di iPhone maupun iPad** —
gunakan aplikasi BLE Serial HM-10 di sana.

### Perangkat keras

Modul Bluetooth harus bertipe **BLE**, bukan Bluetooth Classic. Cirinya memiliki
service `FFE0`. Modul HM-10, HC-08, AT-09, JDY-08, dan CC41-A termasuk BLE.

Sebagian modul bersablon "HC-06" sebenarnya juga BLE. Cara memastikannya ada di
bab 12 `DOKUMENTASI.md`.

### Sketch Arduino

Pastikan `PAKAI_BLUETOOTH` bernilai `1` dan sketch sudah ter-upload. Tanpa itu
halaman akan tersambung tetapi panelnya tetap kosong.

---

## 2. Menjalankan di laptop

Web Bluetooth menuntut *secure context*, sehingga berkas ini **tidak dapat**
dibuka langsung dengan klik ganda — alamat `file://` akan ditolak. Jalankan
server lokal, karena `localhost` dihitung aman.

Buka Terminal di folder `web`, lalu:

```bash
cd "/Users/macbook/Documents/Arduino/SMART Penyiraman/Project_Penyiraman/web"
python3 -m http.server 8000
```

Setelah itu buka di Chrome:

```
http://localhost:8000
```

Hentikan server dengan `Ctrl-C` bila sudah selesai.

Alternatif bila Node.js tersedia:

```bash
npx serve -l 8000
```

---

## 3. Menjalankan di HP Android

`localhost` di laptop tidak dapat dijangkau dari HP. Dan membuka lewat alamat IP
laptop seperti `http://192.168.1.5:8000` **tidak akan bekerja**, karena HTTP
biasa bukan secure context sehingga Web Bluetooth tetap diblokir.

Jadi berkas ini perlu diletakkan di layanan hosting berbasis HTTPS. Dua cara
yang gratis:

### GitHub Pages

1. Buat repositori baru di GitHub
2. Unggah `index.html`
3. Buka Settings → Pages, pilih branch `main` dan folder root
4. Tunggu beberapa menit, lalu buka URL `https://namamu.github.io/namarepo/`

### Netlify Drop

1. Buka `app.netlify.com/drop`
2. Jatuhkan folder `web` ke halaman itu
3. URL HTTPS langsung diberikan

Setelah terbuka lewat HTTPS, halaman dapat dipasang ke homescreen Android agar
terasa seperti aplikasi.

---

## 4. Cara pakai

1. Nyalakan Arduino, pastikan LED modul Bluetooth berkedip
2. Buka halaman, tekan **Sambungkan Perangkat**
3. Pilih modulmu dari daftar yang muncul — namanya bisa `HC-06`, `BT05`,
   `HMSoft`, atau `CC41-A`
4. Titik indikator berubah hijau, dan data mulai mengalir tiap 3 detik

Setelah tersambung, tombol kendali menjadi aktif:

| Tombol | Perintah | Akibat |
|---|---|---|
| Hidupkan Pompa | `N` | Pompa menyala, berhenti sendiri setelah 60 detik |
| Matikan Pompa | `F` | Pompa dipaksa mati |
| Mode Otomatis | `A` | Kembali ke keputusan sensor |
| Kirim Ambang | `B` lalu `K` | Mengubah ambang basah dan kering |
| Minta Status | `S` | Memaksa pembaruan panel |

Daftar perangkat hanya muncul setelah kamu menekan tombol. Ini aturan Web
Bluetooth: pemilihan perangkat wajib dipicu tindakan pengguna, tidak boleh
otomatis.

---

## 5. Penelusuran masalah

| Gejala | Penyebab | Tindakan |
|---|---|---|
| Muncul peringatan "browser tidak mendukung" | Safari atau Firefox | Pakai Chrome atau Edge |
| Muncul peringatan "harus HTTPS atau localhost" | Dibuka sebagai `file://` | Jalankan server lokal, lihat bagian 2 |
| Tombol Sambungkan tidak bereaksi | Prasyarat belum terpenuhi | Baca peringatan di bagian atas halaman |
| Daftar perangkat kosong | Modul tidak mendapat daya | Periksa LED modul, lalu VCC dan GND |
| `service FFE0 tidak ada pada perangkat ini` | Salah perangkat dipilih, atau modul bertipe Classic | Pilih ulang. Bila tetap gagal, modulmu bukan BLE |
| Tersambung tetapi panel kosong | Arduino tidak mengirim | Periksa sketch sudah ter-upload, dan kabel `TXD` ke D2 |
| Data masuk tetapi tombol tidak berpengaruh | Jalur D3 ke `RXD` bermasalah | Periksa pembagi tegangan 1k/2k |
| Koneksi terputus sendiri | Daya modul kurang | Pasang kapasitor 100 µF di VCC dan GND modul |

Log komunikasi di bagian bawah halaman menampilkan seluruh lalu lintas data.
Tanda `>>` berarti dikirim ke Arduino, `<<` berarti diterima dari Arduino.

Bila hanya ada `>>` tanpa `<<`, perintahmu terkirim tetapi Arduino tidak
menjawab. Bila hanya ada `<<` tanpa pengaruh saat tombol ditekan, berarti arah
sebaliknya yang bermasalah.

---

## 6. Catatan keamanan

Halaman ini berjalan sepenuhnya di perangkatmu dan hanya berbicara ke modul
lewat radio Bluetooth. Tidak ada data yang dikirim ke internet, dan tidak ada
server yang menyimpan apa pun.

Meski begitu, jangkauan BLE sekitar 10 meter dan tidak ada autentikasi pada
modul. Siapa pun dalam jangkauan yang memakai halaman atau aplikasi serupa dapat
menyalakan pompa. Untuk pemakaian di tempat ramai, ubah nama dan PIN modul lewat
perintah AT.

Pengaman yang sudah tertanam di Arduino: perintah manual selalu berakhir sendiri
setelah 60 detik dan sistem kembali ke mode otomatis, sehingga perintah yang
terlupakan tidak membuat pompa menyala tanpa henti.
