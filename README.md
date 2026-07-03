# 🎮 Rijal Multi Device — WhatsApp Game & Sticker Bot

Bot WhatsApp berbasis [Baileys](https://github.com/WhiskeySockets/Baileys) dengan fitur game tebak-tebakan, kuis, cek-cekan random, game saldo/slot, dan pembuat sticker/meme.

Sticker & meme dibuat murni pakai **ffmpeg** (binary sistem) + **node-webpmux** (pure JS) — **tidak** pakai `sharp` atau `wa-sticker-formatter`, karena keduanya native module yang sering gagal di-build di Termux/Android arm64.

---

## ✨ Fitur

### 🏳️ Tebak-Tebakan & Kuis
`.tebakbendera` `.tebakkata` `.kuis` `.kuismath` `.kuisengglish` `.kuisjava`
`.hint_bendera` `.hint_kata` `.hint_english` `.skip_bendera` `.skip_kata` `.skip_kuis` `.skip_math` `.skip_english` `.skip_java`

### 🔮 Cek-Cekan Random
`.cektt` `.cekganteng` `.cekcantik` `.cekjodoh` `.cekhoki` `.cekiq` `.ceknasib` `.cekboty` `.ceksaldo`

### 🎰 Game & Saldo
`.slot [jumlah]` `.saldo` `.suitpvp @user` (khusus grup)

### 🛡️ Admin
`.blacklist @user` `.unblacklist @user` `.tambahsaldo @user <jumlah>` — khusus owner

### 🎨 Sticker & Meme
| Command | Keterangan |
|---|---|
| `.s` / `.sticker` | Bikin sticker dari gambar/video. Opsi: `--crop`, `--resize WxH`, `--circle`, `--rounded`, atau `.s PackName Author` |
| `.brat [teks]` | Sticker brat |
| `.brathd [teks]` | Sticker brat versi HD |
| `.bratvid [teks]` | Sticker brat animasi |
| `.bratsquidward [teks]` | Sticker brat gaya Squidward |
| `.bratpatrick [teks]` | Sticker brat gaya Patrick |
| `.bratbahlil [teks]` | Sticker brat gaya Bahlil |
| `.bratanime` / `.animebrat [teks]` | Sticker brat gaya anime |
| `.smeme teks atas \| teks bawah` | Meme sticker dari gambar/sticker |
| `.smemevid teks atas \| teks bawah` | Meme sticker dari video |
| `.qc <warna> <teks>` | Quote-chat sticker (bisa reply pesan) |

Ketik `.menu` di WhatsApp untuk lihat daftar lengkap + versi terbaru.

---

## 📋 Requirement

- **Node.js** ≥ 18
- **ffmpeg** terinstall di sistem (bukan npm package)
- Nomor WhatsApp aktif untuk pairing

---

## 🚀 Instalasi

### Termux (Android)

```bash
pkg update && pkg upgrade
pkg install nodejs-lts ffmpeg git -y

# masuk ke folder project, lalu:
npm install
```

### Linux / VPS

```bash
sudo apt update
sudo apt install -y ffmpeg
# pastikan Node.js >= 18 sudah terinstall (nodejs.org / nvm)

npm install
```

---

## ⚙️ Konfigurasi

Edit bagian **GLOBAL CONFIG** di paling atas `rijal.js` sesuai kebutuhan:

```js
global.namabot       = "𝕽𝖎𝖏𝖆𝖑 𝕸𝖚𝖑𝖙𝖎 𝕯𝖊𝖛𝖎𝖈𝖊💫✨"; // nama bot
global.ownernumber   = "628";               // nomor owner (628xxx)
global.ownerLid      = "-";              // LID owner (cek via .myid kalau WA pakai sistem LID)
global.ownername     = "rijall💫";                       // nama owner
global.botMode       = true;                            // true = Public, false = Self
global.prefix        = ".";                             // prefix command
global.menuImage     = "https://files.catbox.moe/b8a8ur.jpeg"; // gambar menu
global.stickerAuthor = "punya rijal wlee😝";             // default author sticker
global.stickerPack   = global.namabot;                  // default pack name sticker
global.saldoAwal     = 100000;                           // saldo awal user baru
global.slotWinRate   = 0.38;                             // persentase menang slot
```

> Data saldo/game user disimpan **in-memory** (bukan file/disk) — otomatis reset kalau bot di-restart.

---

## ▶️ Menjalankan Bot

```bash
node rijal.js
```

Saat pertama kali dijalankan (belum ada session), bot akan minta nomor WhatsApp lalu memunculkan **kode pairing** di terminal:

```
📱 Nomor WA (628xxx): 
🔑 KODE: XXXX-XXXX
```

Buka WhatsApp → **Perangkat Tertaut** → **Tautkan dengan nomor** → masukkan kode tersebut.

Untuk menjaga bot tetap jalan di background, disarankan pakai **PM2**:

```bash
npm install -g pm2
pm2 start rijal.js --name rijal-bot
pm2 save
```

---

## 🗂️ Struktur Session

Session login WhatsApp disimpan di folder `./session` (lihat `global.sessionDir`). Jangan hapus folder ini kalau tidak mau login ulang. Kalau muncul `❌ Logged out`, hapus folder `session` lalu jalankan ulang bot untuk pairing baru.

---

## 🧩 Catatan Teknis — Sticker Engine

- Konversi gambar/video → webp animasi/statis dilakukan lewat `ffmpeg` (dipanggil sebagai child process, bukan native binding Node).
- Metadata `sticker-pack-name` & `sticker-pack-publisher` ditulis ke EXIF webp pakai `node-webpmux` (pure JS).
- `.smeme` mengunggah gambar sementara ke layanan hosting gambar (coba **termai.cc** → fallback **catbox.moe** → fallback **telegra.ph**) lalu memanggil API [memegen.link](https://memegen.link) untuk menempel teks di atasnya.
- `.smemevid` menggambar teks meme langsung di frame video pakai filter `drawtext` ffmpeg (bukan `@napi-rs/canvas`), supaya tetap ringan & tanpa native build.
- Video untuk `.s`/`.sticker` dibatasi maksimal **10 detik**; `.smemevid` otomatis dipotong **4 detik**.

---

## ⚠️ Troubleshooting

| Masalah | Solusi |
|---|---|
| `ffmpeg: command not found` | Install ffmpeg dulu (`pkg install ffmpeg` / `apt install ffmpeg`) |
| Sticker gagal / kosong | Cek koneksi internet & pastikan file media tidak corrupt |
| `.smeme` / `.qc` gagal upload | Layanan hosting gambar pihak ketiga sedang down — coba lagi beberapa saat |
| Bot logout terus | Hapus folder `session`, jalankan ulang, pairing dengan kode baru |
| Command owner tidak terbaca | Cek `global.ownernumber` / `global.ownerLid`, atau ketik `.myid` untuk lihat JID yang terdeteksi |

---

## 📄 Lisensi

MIT — bebas dipakai & dimodifikasi untuk keperluan pribadi.
