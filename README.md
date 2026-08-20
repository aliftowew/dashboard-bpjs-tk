# Portal Dashboard — BPJS Ketenagakerjaan

Portal statis (HTML, tanpa backend) berisi **dua aplikasi** di balik satu login:

| Berkas | Aplikasi |
|--------|----------|
| `index.html` | Portal: login + dua kartu pilihan dashboard |
| `rkat.html` | Dashboard RKAT 2027 (perencanaan & pengawasan) |
| `kompendium.html` | Monitoring Kompendium Alkes JKK (pengisian aturan penjaminan) |

Membuka `rkat.html` / `kompendium.html` tanpa login akan dialihkan kembali ke portal.

## Monitoring Kompendium Alkes JKK

- Memuat **776 dari 1.283 item** kompendium (kiriman data terpotong; sisanya tinggal
  ditambahkan ke blok `<script id="DATA">` di `kompendium.html`).
- Empat tampilan: Ringkasan (KPI + progres per wilayah tubuh), Daftar item (cari/filter/status),
  Per bagian tubuh, dan Struktur isian (usulan kamus nilai).
- Klik item membuka panel isian **8 kolom aturan** (Indikasi, ICD-10, Level RS, Approval,
  Batas Penggunaan, Monitoring, Dampak RTW, Keterangan); isian tersimpan di localStorage
  peramban dan bisa diekspor lengkap lewat **Unduh CSV**.

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
| `admin`  | `bpjs2027` |

Sesi disimpan di `sessionStorage`, jadi berakhir otomatis saat tab/browser ditutup. Tombol
**Keluar** ada di pojok kanan atas.

### Mengganti kata sandi / menambah pengguna

Kata sandi tidak disimpan sebagai teks biasa — hanya hash SHA-256-nya, di objek `USERS` pada
bagian bawah `index.html`:

```js
const USERS={
  'admin':'b0f2c4ceee7989fe4c01646b64afb500160dc78408a7455af7908298a942c234'
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
