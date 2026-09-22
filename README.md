# Monitoring Energi Terbarukan

Dashboard web statis (satu file `index.html`) untuk memantau stasiun cuaca/energi terbarukan berbasis ESP32 + Firebase Realtime Database. Didesain untuk di-host gratis lewat **GitHub Pages** — tidak perlu server backend.

Dashboard ini terhubung langsung ke Firebase yang sama dengan firmware ESP32, dan membaca ulang data dari `/history` untuk empat halaman:

- **Dashboard** — nilai sensor terkini (diambil dari entri `/history` terbaru) dan status jadwal lampu
- **Grafik** — tren 50 pembacaan terakhir untuk tiap parameter
- **Histori** — tabel 50 entri terbaru
- **Jadwal Sistem** — mengubah jam nyala/mati lampu, ditulis langsung ke path yang dibaca firmware

## Kenapa dashboard versi lama tidak menampilkan data

Firmware ESP32 mengirim data dengan nama field **Bahasa Indonesia** (`suhu`, `kelembapan`, `curah_hujan`, `kecepatan_angin`, `cahaya_lux`, `timestamp`) ke path `/history`. Dashboard ini sudah disesuaikan agar membaca nama field yang sama persis — itu penyebab utama kartu dan grafik sebelumnya selalu kosong (`--`), meski data sebenarnya sudah masuk ke Firebase.

| Path Firebase | Ditulis oleh | Dibaca oleh |
|---|---|---|
| `/history/{push_id}` | Firmware ESP32 (tiap 60 detik) | Dashboard, Grafik, Histori |
| `/JadwalLampu/waktu_nyala` | Dashboard (halaman Jadwal Sistem) | Firmware ESP32 |
| `/JadwalLampu/waktu_mati` | Dashboard (halaman Jadwal Sistem) | Firmware ESP32 |

Firmware **tidak** menulis nilai sensor "live" ke path terpisah — jadi kartu realtime di halaman Dashboard sengaja diambil dari entri `/history` paling baru, bukan dari path `/sensor/current` yang tidak pernah ditulis firmware.

## Cara pakai

1. Fork atau download file `index.html` ini ke dalam repo GitHub kamu.
2. Buka `index.html`, cari blok `firebaseConfig` di bagian bawah file, dan pastikan nilainya sama dengan **Project settings → SDK setup and configuration** di Firebase Console proyekmu.
3. Commit dan push ke branch `main`.
4. Di repo GitHub: **Settings → Pages → Source**, pilih branch `main` dan folder `/ (root)`, lalu Save.
5. Tunggu 1–2 menit, dashboard akan aktif di `https://<username>.github.io/<nama-repo>/`.

## Firebase Realtime Database Rules

Supaya dashboard bisa membaca `/history` dan menulis `/JadwalLampu`, pastikan rules Firebase (Realtime Database → Rules) tidak dalam mode "deny all" default. Contoh rules minimal untuk pengujian (perketat lagi untuk produksi, misalnya dengan Firebase Auth):

```json
{
  "rules": {
    "history": {
      ".read": true,
      ".write": true
    },
    "JadwalLampu": {
      ".read": true,
      ".write": true
    }
  }
}
```

> Rules `true`/`true` di atas membuka akses publik — cukup untuk demo/skripsi, tapi untuk deployment nyata sebaiknya batasi `.write` hanya untuk user terautentikasi (sesuai `USER_EMAIL`/`USER_PASSWORD` yang dipakai firmware).

## Struktur data yang diharapkan

Setiap entri di `/history` berbentuk:

```json
{
  "suhu": 28.4,
  "kelembapan": 76.2,
  "curah_hujan": 0.7,
  "kecepatan_angin": 2.15,
  "cahaya_lux": 340,
  "timestamp": "2026-09-22 14:28:21"
}
```

## Teknologi

- HTML/CSS/JS murni (tanpa build step)
- [Firebase JS SDK (compat, v10)](https://firebase.google.com/docs/web/setup) via CDN — realtime listener ke `/history` dan `/JadwalLampu`
- [Chart.js](https://www.chartjs.org/) via CDN — grafik tren sensor
- Google Fonts: Space Grotesk (judul) + Inter (isi)

## Kustomisasi lanjutan

- **Field baru**: tambahkan kartu di `#page-dashboard`, kolom di tabel histori, dan chart baru mengikuti pola yang sudah ada di `<script>` bagian bawah `index.html`.
- **Otentikasi**: kalau ingin membatasi akses dashboard, tambahkan Firebase Authentication (email/password atau anonymous) sebelum memanggil `db.ref(...)`.
- **Jumlah data grafik/histori**: ubah angka `limitToLast(50)` di bagian JavaScript sesuai kebutuhan.
