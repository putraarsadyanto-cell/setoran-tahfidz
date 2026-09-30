# Setoran Tahfidz

**Setoran Tahfidz** adalah aplikasi pencatat setoran hafalan Al-Qur'an untuk
santri, yang berjalan sepenuhnya **offline** — semua data disimpan di
`localStorage` perangkat, tidak ada backend/server, dan tidak ada koneksi
internet yang dibutuhkan.

Repo ini membungkus aplikasi web single-page tersebut (`www/index.html`)
menjadi aplikasi Android native menggunakan [Capacitor](https://capacitorjs.com/),
siap dibangun sebagai APK (untuk testing) maupun App Bundle/AAB (untuk rilis
ke Google Play Store).

- **Nama tampilan:** Setoran Tahfidz
- **Application ID:** `id.tahfidz.setoran`
- **Web dir:** `www/`
- **Native project:** `android/`

## 1. Menjalankan di Browser (tanpa Android)

Karena aplikasi ini adalah satu file HTML statis, Anda bisa langsung
membukanya di browser tanpa build tool apa pun:

```powershell
# Cukup buka file-nya langsung
start www\index.html
```

Atau jalankan lewat server statis sederhana (opsional, untuk menghindari
batasan file:// di beberapa browser):

```powershell
npx serve www
```

## 2. Struktur Proyek

```
www/index.html          # Aplikasi web (HTML+CSS+JS inline, self-contained)
capacitor.config.json    # Konfigurasi Capacitor (appId, appName, webDir)
android/                 # Proyek Android native (dibuat oleh `npx cap add android`)
assets-src/              # Source gambar 1024x1024 untuk ikon & splash (generate_icon.py)
resources/               # Input untuk @capacitor/assets (icon.png, splash.png, dst.)
release-signing/         # Panduan membuat upload keystore & signing config (README.md)
PRIVACY_POLICY.md        # Kebijakan privasi (dibutuhkan untuk Play Store)
```

## 3. Setup Awal (sekali saja, sudah dilakukan di repo ini)

Langkah-langkah ini sudah dijalankan dan hasilnya sudah ada di repo, dicatat
di sini untuk referensi/jika ingin mengulang dari nol:

```powershell
npm install
npx cap init "Setoran Tahfidz" "id.tahfidz.setoran" --web-dir www
npx cap add android
npx capacitor-assets generate --android   # dari resources/icon.png & resources/splash.png
```

## 4. Membuka Proyek di Android Studio

1. Install [Android Studio](https://developer.android.com/studio) (sudah
   termasuk Android SDK dan JDK yang kompatibel).
2. Buka folder `android/` di Android Studio (**File → Open**, pilih folder
   `android`, bukan folder root repo).
3. Tunggu proses Gradle sync selesai.
4. Klik **Run ▶** untuk menjalankan di emulator/perangkat, atau gunakan menu
   **Build** untuk membuat APK/AAB (lihat bagian 5 & 6).

Jika Anda mengubah `www/index.html`, jalankan `npx cap sync android` agar
perubahan tersalin ke `android/app/src/main/assets/public`.

## 5. Build Debug APK (baris perintah)

Dibutuhkan: JDK 21 dan Android SDK (build-tools + platform sesuai
`android/variables.gradle`, saat ini `compileSdk`/`targetSdk` 36, `minSdk` 24).

```powershell
cd android
$env:JAVA_HOME = "<path ke JDK 21>"
.\gradlew.bat assembleDebug
```

Output: `android/app/build/outputs/apk/debug/app-debug.apk`

Build ini **sudah diverifikasi berhasil** di lingkungan pengembangan proyek
ini (JDK Temurin 21, Android SDK build-tools 35/36).

## 6. Build Release AAB (untuk Play Store)

Play Store mewajibkan **Android App Bundle (.aab)** yang ditandatangani
dengan upload key Anda sendiri. Lihat **`release-signing/README.md`** untuk
langkah lengkap membuat keystore dan `android/keystore.properties`. Setelah
itu:

```powershell
cd android
.\gradlew.bat bundleRelease
```

Output: `android/app/build/outputs/bundle/release/app-release.aab`

> Tidak ada keystore asli yang disertakan di repo ini (dan memang tidak boleh
> di-commit). Anda wajib membuat keystore sendiri secara pribadi.

## 7. Checklist Submit ke Google Play Store

Ikuti langkah-langkah berikut secara berurutan:

1. **Buat akun Google Play Console** — daftar sebagai developer di
   [play.google.com/console](https://play.google.com/console/), bayar biaya
   pendaftaran satu kali sebesar **USD 25**, verifikasi identitas.
2. **Buat aplikasi baru** di Play Console → **Create app**, isi nama
   "Setoran Tahfidz", bahasa default Indonesia, kategori (misal:
   Pendidikan/Education), pernyataan gratis (Free).
3. **Lengkapi Store Listing:**
   - Deskripsi singkat (short description, maks 80 karakter) dalam Bahasa
     Indonesia.
   - Deskripsi lengkap (full description, maks 4000 karakter) dalam Bahasa
     Indonesia — jelaskan fitur pencatatan setoran hafalan, penyimpanan
     lokal/offline, tanpa iklan, tanpa akun.
   - Minimal **2 screenshot ponsel** (rasio 16:9 atau portrait, PNG/JPEG,
     min. 320px, maks. 3840px sisi terpanjang) — ambil dari aplikasi yang
     berjalan di emulator/perangkat.
   - **Feature graphic** 1024x500 px (banner promosi di halaman listing).
   - Ikon aplikasi 512x512 (untuk listing; ikon adaptif Android sudah
     tersedia otomatis dari build, tapi Play Console meminta upload
     terpisah 512x512 — export dari `assets-src/icon-1024.png`, resize ke
     512x512).
4. **Content Rating Questionnaire** — isi kuesioner rating konten (aplikasi
   edukasi tanpa konten kekerasan/dewasa, jawab sesuai kondisi sebenarnya)
   untuk mendapatkan rating usia.
5. **Data Safety form** — isi bahwa aplikasi **tidak mengumpulkan dan tidak
   membagikan data pengguna apa pun**; semua data hanya disimpan lokal di
   perangkat (`localStorage`), tidak ada transmisi data ke server manapun.
   Gunakan `PRIVACY_POLICY.md` di repo ini sebagai referensi jawaban.
6. **Privacy Policy URL** — publikasikan isi `PRIVACY_POLICY.md` di suatu
   halaman web publik (misalnya GitHub Pages dari repo ini, atau hosting
   statis lain), lalu masukkan URL tersebut di Play Console → App content →
   Privacy Policy.
7. **Pilih release track** — mulai dari **Internal testing** (Play Console →
   Release → Testing → Internal testing → Create new release), tambahkan
   penguji (email Anda sendiri/tim), ini memungkinkan pengujian cepat tanpa
   review penuh.
8. **Upload AAB** — unggah `app-release.aab` hasil `bundleRelease` yang sudah
   ditandatangani ke release track yang dipilih.
9. **Submit for review** — lengkapi rollout, isi release notes, lalu kirim
   untuk ditinjau oleh Google. Setelah lolos internal testing dan Anda puas
   dengan hasilnya, promosikan release ke **Production** track (juga
   melalui proses review Google).

### Yang TIDAK bisa dilakukan agen ini
Pembuatan akun Play Console, verifikasi identitas developer, pembayaran
biaya pendaftaran, pengambilan screenshot dari perangkat/emulator asli, dan
persetujuan akhir submission adalah langkah yang **membutuhkan aksi manual
dari Anda (pemilik akun)** dan tidak dapat diotomatisasi oleh agen coding.

## 8. Ikon & Splash Screen

Ikon dan splash screen adalah placeholder yang dibuat secara programatik
(latar ungu plum `#5b2a6b` dengan motif buku terbuka + tanda centang emas,
dan monogram "ST") menggunakan `assets-src/generate_icon.py` (Python + Pillow)
dan diproses lewat `@capacitor/assets` menjadi seluruh resolusi mipmap
Android (ldpi–xxxhdpi) plus adaptive icon (`ic_launcher_foreground` /
`ic_launcher_background`) dan splash screen semua densitas/orientasi.
Anda bebas mengganti `resources/icon.png`, `resources/icon-foreground.png`,
`resources/icon-background.png`, dan `resources/splash.png` dengan desain
final, lalu jalankan ulang:

```powershell
npx capacitor-assets generate --android
```

## 9. Izin Aplikasi

Aplikasi ini **tidak meminta izin khusus apa pun** — sepenuhnya offline,
tidak ada `INTERNET` atau permission lain di `AndroidManifest.xml`.
