# Portal Dashboard — BPJS Ketenagakerjaan

Portal statis (HTML, tanpa backend) berisi **dua aplikasi** di balik satu login:

| Berkas | Aplikasi |
|--------|----------|
| `index.html` | Portal: login + dua kartu pilihan dashboard |
| `rkat.html` | Dashboard RKAT 2027 (perencanaan & pengawasan) |
| `kompendium.html` | Monitoring Kompendium Alkes JKK (pengisian aturan penjaminan) |
| `operasional.html` | Operasional Alkes JKK (e-purchasing, SPH, vendor, tarif & aktuaria) |
| `referensi.html` | Formularium & Kompendium JKK (register rujukan penjaminan) |
| `alkes-data.js` | Data 776 item kompendium (dipakai bersama kompendium.html & referensi.html) |

Membuka `rkat.html` / `kompendium.html` tanpa login akan dialihkan kembali ke portal.

## Monitoring Kompendium Alkes JKK

- Memuat **776 dari 1.283 item** kompendium (kiriman data terpotong; sisanya tinggal
  ditambahkan ke blok `<script id="DATA">` di `kompendium.html`).
- Empat tampilan: Ringkasan (KPI + progres per wilayah tubuh), Daftar item (cari/filter/status),
  Per bagian tubuh, dan Struktur isian (usulan kamus nilai).
- Klik item membuka panel isian **8 kolom aturan** (Indikasi, ICD-10, Level RS, Approval,
  Batas Penggunaan, Monitoring, Dampak RTW, Keterangan); isian tersimpan di localStorage
  peramban dan bisa diekspor lengkap lewat **Unduh CSV**.

## Operasional Alkes JKK

- **Katalog & e-Purchasing** — 13 item kompendium (ASSA + GOENMED SUPREME) dengan spesifikasi,
  kemasan, nomor izin AKD, dan tautan e-katalog INAPROC.
- **Penawaran Harga** — SPH 026/SPM-P/PLG/08.26 PT Sinergi Persada Medica (18 item BHP); tabel
  disiapkan untuk banding harga saat SPH vendor lain masuk.
- **Vendor** — direktori 39 vendor alkes dengan tautan telepon dan WhatsApp.
- **Tarif & Aktuaria** — tabulasi tarif kelas 1 RS pemerintah tipe A untuk prosedur kecelakaan
  kerja (ortopedi/trauma/cedera). Kolom tarif **sengaja kosong** — diisi dari sumber resmi
  (tersimpan di localStorage, bisa diekspor CSV). Kalkulator aktuaria menghitung beban skenario
  (kasus × tarif × inflasi medis) dan dampaknya ke ketahanan aset neto dana JKK
  (basis RKAT 2027: iuran Rp10,78 T, klaim Rp6,58 T, ketahanan 174,57 bulan).

## Formularium & Kompendium JKK

Register rujukan "apa yang dijamin JKK", berdiri sendiri sesuai permintaan:

- **Kompendium Alkes** — 776 item terdaftar + 13 usulan baru (ASSA/GOENMED SUPREME) dengan status,
  pencarian, dan filter wilayah.
- **Formularium Obat** — register obat generik (sediaan, kelas terapi, restriksi, status penjaminan).
  Kosong secara bawaan karena daftar obat resmi belum tersedia; bisa diisi manual atau **impor CSV**
  (format: `Nama generik; Sediaan & kekuatan; Kelas terapi; Restriksi; Status`), tersimpan di
  localStorage, dan diekspor kembali sebagai CSV.

## Dashboard RKAT 2027

Rangkuman Buku RKAT 2027 diuji silang laporan keuangan Semester I 2026, dilengkapi:

- **Seksi "Capaian Target"** — input realisasi terkini untuk menghitung berapa persen target 2027
  sudah tercapai, dibandingkan dengan laju tahun berjalan. Data tersimpan di browser (localStorage).
- **Identitas visual BPJS Ketenagakerjaan** — logo digambar ulang sebagai SVG inline (tanpa file
  gambar terpisah) dan seluruh tema warna mengikuti palet logo: hijau `#3AAA35`, biru `#29ABE2`,
  lime `#D7DF23`. Semua warna terpusat di variabel CSS pada blok `:root` di `index.html`, jadi
  mudah disetel ulang dari satu tempat.
- **Penjelajah anggaran** — pencarian, filter unit, urutkan, dan muat bertahap atas baris kegiatan
  lampiran RKAT (saat ini memuat 482 kegiatan teratas; data lengkap 2.092 baris menyusul).
- **Seksi LK S1-2026** — uji silang laporan keuangan resmi Semester I 2026 terhadap prognosa RKAT
  (solvabilitas JHT, unrealized loss, kontraksi kepesertaan, verifikasi silang indikator).
- **Catatan Pengawasan** — 21 temuan tiga tingkat (Substansi / Risiko & Indikator / Editorial)
  dengan centang pilihan (tersimpan di localStorage) dan penyusun draft SNP siap salin.

## Menjalankan

Buka `index.html` langsung di browser, atau sajikan lewat server statis:

```bash
python3 -m http.server 8000
# lalu buka http://localhost:8000
```

## Login

Kredensial bawaan:

| Pengguna | Kata sandi |
|----------|------------|
| `admin`  | `bpjslif2027` |

Sesi disimpan di `sessionStorage`, jadi berakhir otomatis saat tab/browser ditutup. Tombol
**Keluar** ada di pojok kanan atas.

### Mengganti kata sandi / menambah pengguna

Kata sandi tidak disimpan sebagai teks biasa — hanya hash SHA-256-nya, di objek `USERS` pada
bagian bawah `index.html`:

```js
const USERS={
  'admin':'441bee83dfc9956add76ca02ebde07a083c77e2c5fee7d36b1ff7bacb818b004'
};
```

Untuk membuat hash kata sandi baru, jalankan di terminal:

```bash
printf '%s' 'kata-sandi-baru' | sha256sum
```

atau di konsol browser (F12):

```js
crypto.subtle.digest('SHA-256', new TextEncoder().encode('kata-sandi-baru'))
  .then(b => console.log([...new Uint8Array(b)].map(x => x.toString(16).padStart(2,'0')).join('')));
```

Lalu tambahkan/ganti entri di objek `USERS`.

> **Catatan keamanan:** karena ini situs statis tanpa server, login ini hanyalah *pagar* di sisi
> browser — siapa pun yang bisa membaca berkas HTML-nya tetap bisa melihat datanya. Untuk
> perlindungan sungguhan, sajikan di balik autentikasi server (mis. basic auth di reverse proxy,
> Cloudflare Access, atau SSO internal).

## Seksi Capaian Target

- Isi kolom input dengan angka realisasi terkini (satuan tertera di tiap baris: juta TK,
  Rp triliun, atau Rp miliar).
- Atur **"Data per tanggal"** sesuai tanggal data realisasi — garis putus-putus pada batang
  menunjukkan laju wajar tahun berjalan 2027, dan status tiap indikator (sesuai laju / sedikit
  tertinggal / tertinggal / tercapai) dihitung terhadap laju itu.
- Pos belanja (pembayaran jaminan, beban usaha, belanja modal, SKP) dibaca sebagai **penyerapan**,
  bukan prestasi.
- Data tersimpan otomatis di localStorage browser — tidak terkirim ke mana pun. Tombol
  **Reset data** menghapusnya.
