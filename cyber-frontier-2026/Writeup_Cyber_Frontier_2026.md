# Writeup Cyber Frontier 2026 — Beyond The Attack

Writeup ini disusun oleh tim **CF26-F98637547B16** untuk kompetisi **Cyber Frontier 2026 — Beyond The Attack** (CTF & OSINT), diselenggarakan PT Siber Sentinel Asia bekerja sama dengan Diskominfo Kota Semarang dalam rangka Hari Pahlawan 10 November. Skor akhir tim: **2590 poin**, peringkat **#116**.

Dari 18 tantangan yang dikerjakan, 17 berhasil diselesaikan dan 1 (Soto Semarang) belum terpecahkan saat kompetisi ditutup. Setiap bagian di bawah menjelaskan kategori, pendekatan analisis, langkah teknis kunci, dan flag yang diperoleh.

## Daftar Isi

1. Kota Lama (Digital Forensics)
2. Bandeng Presto (Cryptography)
3. Lumpia (Cryptography)
4. Sam Poo Kong (Digital Forensics)
5. Tugu Muda (Digital Forensics)
6. Simpang Lima (Network Analysis)
7. Stasiun Tawang (Network Analysis)
8. Goa Kreo (OSINT)
9. Kampung Pelangi (OSINT)
10. Kucingan (OSINT)
11. Pantai Marina (OSINT)
12. Tahu Gimbal (OSINT)
13. Gereja Blenduk (Reverse Engineering)
14. Tanjung Mas (Reverse Engineering)
15. Balaikota (Web Security)
16. Lawang Sewu (Web Security)
17. Pasar Johar (Web Security)
18. Soto Semarang (Miscellaneous) — tidak terselesaikan

---

## 1. Kota Lama

**Kategori:** Digital Forensics · **Poin:** 50 · **Flag:** `CF2{kota_lama_0b4f827d6420e7b43df1fa64}`

Soal memberi tiga PNG 96x96 (`capture-101/102/103.png`) dan `chain-of-custody.csv` yang mencatat `input_sha256` dari operasi `camera_ingest`.

1. SHA-256 tiap PNG dihitung dan dicocokkan terhadap `input_sha256` di CSV — hanya `capture-101.png` yang cocok, mengonfirmasinya sebagai salinan kamera **asli** (dua lainnya adalah hasil resize dan scan independen, pengecoh).
2. Pesan diekstrak dengan membaca bit paling rendah (LSB) kanal R, G, B setiap piksel secara berurutan (kiri-ke-kanan, atas-ke-bawah), MSB dulu per byte, berhenti di byte NUL pertama.
3. `capture-101.png` menghasilkan `KL-ORIGINAL-7F2A`; dua file lain menghasilkan pesan pengecoh (`KL-PREVIEW-DECOY`, `KL-SCAN-NO-RECORD`) yang dibuang.
4. Flag = `CF2{kota_lama_<24 hex pertama SHA-256(pesan)>}`.

---

## 2. Bandeng Presto

**Kategori:** Cryptography · **Poin:** 250 · **Flag:** `CF2{bandeng_presto_934e1e05dfaccf58f01482c3}`

Arsip BP-POOL dipecah via Shamir Secret Sharing ambang 2-dari-5 atas GF(2^8) (polinomial `0x11B`), satu share per perangkat (`device-shares.csv`), plus `archive.enc` (JSON berisi `ciphertext_hex` dan `hmac_sha256`).

1. Setiap baris share diverifikasi: `SHA256(x_desimal ":" y_bytes)` harus cocok `provisioning_commitment_sha256`. Satu dari lima kapsul (BP-04) gagal verifikasi dan dibuang.
2. Empat kapsul valid sisanya (BP-01, 02, 03, 05) diinterpolasi berpasangan di GF(2^8), semua pasangan konsisten menghasilkan secret yang sama: `9b70e31c55a2d84f10c6397e2a8b4d61`.
3. Keystream `SHA256(secret || counter32be)` (counter mulai 0, disambung) di-XOR-kan dengan `archive.enc` untuk mendapatkan ciphertext dan HMAC asli.
4. HMAC-SHA256(secret, ciphertext) diverifikasi cocok dengan `hmac_sha256` sebelum ciphertext diterima.
5. Plaintext hasil dekripsi: `BP-POOL|R-73|restore=complete`. Flag = `CF2{bandeng_presto_<24 hex pertama SHA-256(plaintext)>}`.

---

## 3. Lumpia

**Kategori:** Cryptography · **Poin:** 125 · **Flag:** `CF2{crypto_lumpia_0fb3528b68ef3a1cc589}`

Tiga transaksi terenkripsi XOR dengan mask 16-byte yang diulang per record (offset direset tiap record), tanpa salt. Satu fragmen spool printer menyebutkan prefix plaintext L-001 persis 16 byte: `LUMPIA-ORDER-V3:`.

1. Mask 16-byte dipulihkan dengan meng-XOR-kan 16 byte pertama ciphertext L-001 dengan prefix yang diketahui.
2. Mask yang sama (diulang) digunakan untuk mendekripsi L-002, menghasilkan langsung string flag: `CF2{crypto_lumpia_0fb3528b68ef3a1cc589}`.
3. L-001 dan L-003 mendekripsi menjadi pesan transaksi biasa (`LUMPIA-ORDER-V3:000841;portion=3;table=07`, dst.) — konsisten sebagai validasi bahwa mask yang dipulihkan benar.

---

## 4. Sam Poo Kong

**Kategori:** Digital Forensics · **Poin:** 125 · **Flag:** `CF2{for_sam_poo_kong_da00378693800796e12b}`

Arsip SPK-71 terpecah jadi 5 fragmen (posisi 0–4), masing-masing punya 2–3 kandidat hex di `fragments.csv`. `catalogue.json` menyimpan `compressed_sha256` atas gabungan fragmen yang benar (zlib-compressed).

1. Semua kombinasi kandidat (hasil `itertools.product` tiap posisi, total 243 kombinasi) digabung lalu di-SHA-256 dan dibandingkan dengan `compressed_sha256`.
2. Tepat satu kombinasi cocok.
3. Hasil gabungan di-zlib-decompress, menghasilkan langsung flag `CF2{for_sam_poo_kong_da00378693800796e12b}`.

---

## 5. Tugu Muda

**Kategori:** Digital Forensics · **Poin:** 250 · **Flag:** `CF2{for_tugu_muda_42751a1dca393b5ac5ce}`

Tiga file kamera (`kamera-41/42/43.png`) dengan `camera-events.csv` (mencatat nonce per file) dan `radio-shifts.csv` (frasa sandi radio per interval waktu WIB). `format-ekspor.txt` menjelaskan skema: kunci `SHA256(phrase + "|" + nonce)`, LSB kanal B dibaca mengikuti daftar piksel kandidat `(x+2y)%5==0` yang diacak via `random.Random(seed)` dengan seed dari `SHA256(nonce)`, lalu frame `TM3 + len + ciphertext + HMAC[:16]`, ciphertext = `zlib(payload) XOR SHA256(key||counter)`.

1. Ekspor terjadi 12 Okt 2026 pukul 00.04 WIB = 17:04 UTC → cocok dengan event `export` untuk `kamera-42.png` (nonce `TM-7419`).
2. Frasa radio yang berlaku pada 17:04 UTC (= 00:04 WIB) adalah `lentera-menjaga-arah` (interval 00.00–00.15 WIB).
3. Kunci dihitung, LSB diekstrak sesuai urutan acak, HMAC frame diverifikasi cocok, lalu ciphertext didekripsi dan di-zlib-decompress.
4. Hasil langsung berupa flag. `kamera-41.png` dan `kamera-43.png` tidak punya header `TM3` valid (bukan file target).

---

## 6. Simpang Lima

**Kategori:** Network Analysis · **Poin:** 50 · **Flag:** `CF2{simpang_lima_b5531cbc45bfe17eb13942cb}`

`simpang-lima.pcap` berisi query/response DNS TXT dengan label `pNN.run-ID.archive.lab`, data TXT berformat `INDEX/TOTAL:BASE64`. `dhcp-leases.csv` memetakan `terminal-TW-204` ke `10.26.5.17` pada jendela insiden 17:00–17:10 UTC.

1. PCAP di-parse manual (header pcap classic + UDP/DNS) untuk mengambil semua query dan jawaban TXT.
2. Hanya run `run-7` dari `10.26.5.17` yang lengkap (1/5 s.d. 5/5) dalam jendela insiden; `run-8` dari terminal yang sama terpotong (hanya 1/5, 2/5), dan `run-9` lengkap tapi berasal dari `test-laptop` (10.26.5.99), bukan terminal yang di-lease.
3. Potongan Base64 run-7 digabung sesuai indeks lalu di-decode, menghasilkan `SL-204|ARCHIVE-17|2026-10-11T17:03:12Z`.
4. Flag = hash SHA-256 24 karakter pertama dari pesan tersebut, tanpa newline.

---

## 7. Stasiun Tawang

**Kategori:** Network Analysis · **Poin:** 150 · **Flag:** `CF2{net_stasiun_tawang_3f23688e9ec86ed8f458}`

`tawang.pcap` (linktype RAW IPv4) berisi query DNS massal ke `node*.status.invalid` (noise) dan sesi sinkronisasi `pNN.sesi.archive.invalid` dengan payload `INDEX/TOTAL:chunk:CRC32`. Timestamp pcap sintetis, sehingga sumber ditentukan dari `dhcp.csv`, bukan dari waktu paket.

1. Insiden 12 Okt 2026 09.20 WIB = 02:20 UTC. `dhcp.csv` memetakan terminal **TW-17** ke IP `192.0.2.77` pada hari itu (IP lama `192.0.2.61` berlaku hari sebelumnya).
2. Sesi dari `192.0.2.77` adalah `b7e4`; sesi `a1c3` (dari `.61`, TW-17 hari sebelumnya) dan `d9f2` (dari `.88`, TW-23) adalah pengecoh.
3. Tiap indeks (0–4) punya dua kandidat chunk; CRC32 atas teks ASCII chunk dihitung dan dicocokkan terhadap CRC32 yang tertera — hanya satu kandidat per indeks yang valid.
4. Chunk valid digabung berurutan, di-base32-decode, lalu di-zlib-decompress, menghasilkan langsung flag.

---

## 8. Goa Kreo

**Kategori:** OSINT · **Poin:** 250 · **Flag:** `CF2{osint_goa_kreo_74f570cc8952455187961ebf}`

7 tabel CSV saling terkait (bookings, booking-amendments, archive-alias, visit-assignments, license-changes, basecamp-log, permit-index) dengan interval validitas half-open `[from,until)`, harus diresolusi pada satu titik waktu: ulasan `BK-5819`, mulai tur **2026-10-11T17:20:00Z**, jalur GA, akun `@gerbang_senja`.

1. **Permit:** versi `confirmed` terakhir pada/sebelum waktu mulai di `booking-amendments` adalah pukul 17:10Z (`PR-60728327`); versi 18:10Z terjadi setelah tur sehingga tidak berlaku.
2. **Visit:** token permit tsb (`AP-c4f24f228781`) punya dua assignment; yang efektif pada 17:20Z adalah visit `0741`.
3. **UID pemandu:** alias `@gerbang_senja` pada 17:20Z dipegang `G0721` (alias berpindah ke `G0884` baru mulai 18:00Z).
4. **Lisensi efektif:** snapshot profil `G0721` = `LIC-118`, berubah menjadi `LIC-829` sejak 17:00Z — itulah lisensi saat tur.
5. **Log basecamp:** di antara banyak kunjungan `completed` berlisensi `LIC-829`/route GA/waktu sama, hanya satu bervisit `0741` (tim `TM-KRO`).
6. Jawaban kanonis: `UID|ENTITY|SLOT`... pada pola OSINT lain; untuk Goa Kreo spesifik: `G0721|LIC-829|TM-KRO|0741|2026-10-11T17:20:00Z`, di-hash SHA-256 → 24 hex pertama = flag.

---

## 9. Kampung Pelangi

**Kategori:** OSINT · **Poin:** 150 · **Flag:** `CF2{kampung_pelangi_74c32387760628b6ec0dcdfc}`

Tiga tabel riwayat (`alias-history.csv`, `contact-history.csv`, `registry.csv`) dengan interval half-open `[valid_from, valid_until)`. Postingan `@pelangi_utara` dipublikasikan **2026-10-12 01:30 WIB = 2026-10-11T18:30Z**.

1. **Pemilik alias** pada 18:30Z: `UID-204` (interval 18:00–20:00Z). `UID-117` sudah berakhir pukul 18:00Z; `UID-311` baru mulai 20:00Z.
2. **Kontak** `UID-204` pada 18:30Z: `CONTACT-41` (interval 17:00–19:00Z); `CONTACT-88` baru berlaku mulai 19:00Z.
3. **Registry** `CONTACT-41` pada 18:30Z: `ENTITY-PELANGI-07` / `SLOT-014` (interval 18:00–19:00Z, revisi `approved`). Baris `ENTITY-PELANGI-70` (revisi `superseded`→ jadi berlaku berikutnya) baru aktif mulai 19:00Z — jebakan "pakai status terbaru" yang harus dihindari sesuai aturan soal.
4. Jawaban kanonis `UID|ENTITY|SLOT` = `UID-204|ENTITY-PELANGI-07|SLOT-014`, di-hash SHA-256 → 24 hex pertama = flag.

---

## 10. Kucingan

**Kategori:** OSINT · **Poin:** 125 · **Flag:** `CF2{osint_kucingan_304acad5fac477f4232a955e}`

1. Ekstrak metadata EXIF dari foto bukti: koordinat GPS (format DMS) dan `DateTimeOriginal`.
2. Konversi DMS → desimal, lalu hitung jarak **haversine** dari titik foto ke setiap gerai pada daftar gerai aktif pada tanggal tersebut (`2026-10-12`).
3. Gerai terdekat yang berstatus aktif pada tanggal foto: `K011`, sektor `A11`.
4. Jawaban kanonis `GERAI|SEKTOR|TANGGAL` = `K011|A11|2026-10-12`, di-hash SHA-256 → 24 hex pertama = flag.

---

## 11. Pantai Marina

**Kategori:** OSINT · **Poin:** 150 · **Flag:** `CF2{osint_pantai_marina_1e6a2bc634e8564f0b4f4549}`

Manifes trip bersifat *append-only* (setiap perubahan status ditambah sebagai baris baru, bukan menimpa baris lama), sehingga harus diambil **versi terakhir** per trip berdasarkan timestamp update.

1. Trip `TR-701`: versi terakhir berstatus `cancelled` — trip ini tidak pernah benar-benar berlayar, jadi dieliminasi sebagai kandidat meski muncul lebih dulu di log.
2. Trip `TR-702`: versi terakhir berstatus `boarded` dengan kode manifes `0926`, cocok dengan jendela waktu kejadian yang ditanyakan.
3. Jawaban kanonis `TRIP|KODE|STATUS` = `TR-702|0926|boarded`, di-hash SHA-256 → 24 hex pertama = flag.

---

## 12. Tahu Gimbal

**Kategori:** OSINT · **Poin:** 50 · **Flag:** `CF2{osint_tahu_gimbal_ae8ce48b3354a033921df804}`

Direktori pedagang difilter bertingkat berdasarkan beberapa kriteria sekaligus yang disebut dalam deskripsi soal: nama pemilik, warna gerobak, sektor penempatan, dan status aktif.

1. Saring direktori per nama pemilik yang disebut → beberapa kandidat dengan nama sama.
2. Saring lanjut per warna gerobak → mempersempit ke beberapa baris.
3. Saring lanjut per sektor dan status `active` (bukan `inactive`/`suspended`) → tersisa satu baris unik: lapak `LP-118`, kode `0427`.
4. Jawaban kanonis `LAPAK|KODE` = `LP-118|0427`, di-hash SHA-256 → 24 hex pertama = flag.

---

## 13. Gereja Blenduk

**Kategori:** Reverse Engineering · **Poin:** 150 · **Flag:** `CF2{rev_gereja_blenduk_de2fe1fb3466b18bfaaa}`

1. Disassembly biner (`objdump -d -M intel`) menunjukkan validator berupa **sistem persamaan linear mod 257** yang diterapkan pada byte-byte serial masukan.
2. Matriks koefisien diselesaikan dengan **eliminasi Gauss** memakai invers modular mod 257 (257 prima sehingga setiap elemen non-nol punya invers), menghasilkan kandidat byte serial.
3. Pemeriksaan tambahan di jalur validasi menggunakan **LCG** (`v = (s*197 + 31) & 255`) untuk mengacak/memvalidasi byte-byte tertentu; LCG ini dibalik dengan tabel pencarian brute-force atas 256 kemungkinan nilai.
4. Serial yang lolos kedua pemeriksaan: **`YLpaZdJg`**.
5. Serial dipakai sebagai kunci untuk membangkitkan keystream `SHA-256(key || counter32be)`, di-XOR-kan ke blob terenkripsi → plaintext berisi flag.

---

## 14. Tanjung Mas

**Kategori:** Reverse Engineering · **Poin:** 150 · **Flag:** `CF2{rev_tanjung_mas_7d666448bc5641697c86}`

1. Biner ELF x86-64 dibedah statis dengan `objdump -d -M intel` (tanpa eksekusi). Fungsi validasi serial ternyata adalah **Feistel cipher 16 ronde**.
2. Fungsi ronde (round function) diidentifikasi dari disassembly, lalu dijalankan **mundur** (invers) dari nilai output yang diketahui untuk merekonstruksi blok input — menghasilkan kandidat serial.
3. Serial yang memvalidasi: **`SKcnbmGE`**.
4. Serial ini menjadi seed untuk kombinasi **FNV-1a 64-bit hash** dan **PRNG splitmix64**, yang membangkitkan keystream byte demi byte.
5. Keystream di-XOR-kan ke payload terenkripsi dalam biner → plaintext berisi flag.

---

## 15. Balaikota

**Kategori:** Web Security · **Poin:** 275 · **Flag:** `CF2{web_balaikota_f33ebf1ecd0c30002800}`

Target `http://103.196.155.224:8102`. Soal menyebut satu sesi staf diduga memperoleh akses ke ekspor arsip yang seharusnya terbatas, dan meminta rekonstruksi jalur akses serta pengambilan bukti ekspor terakhir. Halaman utama menyediakan kotak pencarian (`/cari?q=`) dan link "Dokumen API" yang menjelaskan endpoint `GET /archive/export`: header `Authorization: Bearer <token>`, dengan `token = base64url(key_id + "." + audience + "." + expiry) + "." + hex(HMAC-SHA256(service_secret, key_id + "." + audience + "." + expiry))`, dan keempat nilai tersebut "diperoleh dari record audit dan ekspor arsip yang sah" — tidak diberikan langsung.

1. Fitur pencarian rentan **SQL injection** (SQLite). Payload `q=a'` memicu error ("Pencarian gagal."); `q=a' OR '1'='1` mengubah hasil, mengonfirmasi query bisa dimanipulasi.
2. Jumlah kolom ditentukan via `ORDER BY` bertahap (gagal di 4 → 3 kolom), lalu dikonfirmasi dengan `UNION SELECT 1,2,3`. Jenis database dipastikan SQLite 3.46.1 lewat `sqlite_version()` (uji MySQL `information_schema` gagal).
3. Enumerasi `sqlite_master` menemukan empat tabel: `audit`, `dokumen`, `ekspor`, `pengguna`. Dump tabel `audit` membocorkan `key_id = balai-arsip-01`, `audience = arsip-balaikota`, `expiry = 2026-11-01`, serta petunjuk bahwa kata sandi staf dipilih dari kamus internal kota di `/static/kamus-kota.txt`.
4. Dump tabel `pengguna` mengungkap `password_hash` berupa SHA-256 polos tanpa salt. Hash akun `staf.arsip` dicocokkan secara offline terhadap kombinasi kata dari kamus (nama tempat/makanan khas Semarang plus varian angka/tahun), ditemukan kata sandi `tahugimbal13`.
5. Login sebagai `staf.arsip` membuka Dasbor staf dengan link "Ekspor bulanan staf" beralamat `/ekspor?id=1`. Dump tabel `ekspor` via SQLi menunjukkan baris id 2 dan 5 milik akun berbeda (`akun-arsip`). Mengganti parameter menjadi `/ekspor?id=2` (**IDOR**, broken access control) membuka ekspor milik akun lain, berisi keempat bahan lengkap termasuk `service_secret`.
6. Token HMAC dirakit dari keempat nilai tersebut dan dikirim sebagai header `Authorization` ke `/archive/export`, yang langsung mengembalikan flag.

Rantai kerentanannya berlapis tiga: SQL injection membocorkan kredensial dan rahasia layanan, hash kata sandi lemah bisa dipecahkan dengan kamus kontekstual, dan IDOR pada endpoint ekspor tetap membocorkan data akun lain meski autentikasi sudah berhasil dilewati secara sah.

---

## 16. Lawang Sewu

**Kategori:** Web Security · **Poin:** 50 · **Flag:** `CF2{web_lawang_sewu_e4710b37cd0f26958ead}`

Target `http://103.196.155.224:8101`. Portal arsip menampilkan dokumen lewat parameter `page`, dan soal meminta ditemukannya dokumen internal yang semestinya tidak bisa diakses publik.

1. Halaman utama menampilkan empat dokumen publik dengan pola URL seragam `?page=<nama>.txt`, ditambah satu link kelima "Konfigurasi arsip" yang memakai `?page=../config.ini` — membocorkan bahwa parameter `page` membaca file relatif terhadap sebuah folder dan menerima notasi `../`.
2. Isi `config.ini` menyebutkan dua direktori sejajar, `public` dan `private`, dan secara eksplisit mencatat bahwa dokumen internal bernama `receipt.txt` tersimpan di direktori `private`.
3. Dengan pola traversal yang sama, direktori dinaikkan satu level lalu diarahkan ke `private`: `?page=../private/receipt.txt`. Permintaan ini langsung mengembalikan isi file, yang memuat flag.

Kerentanannya adalah **path traversal / local file inclusion** sederhana, di mana file konfigurasi yang seharusnya tersembunyi justru terekspos lewat jalur yang sama dan menjadi petunjuk sintaks traversal yang valid serta nama file target.

---

## 17. Pasar Johar

**Kategori:** Web Security · **Poin:** 250 · **Flag:** `CF2{web_pasar_johar_4013c0f749b871c9635a}`

Target `http://103.196.155.224:8103`. Soal menyebut sebuah akun pembeli mampu membuka fungsi pedagang dan mengekspor laporan internal, dan meminta direkonstruksinya rangkaian kelemahan yang memungkinkan hal itu.

1. Akun demo pembeli (`pembeli01`–`pembeli30`, sandi `johar-demo`) menghasilkan JWT HS256 dengan klaim `{"sub":"pembeli01","role":"buyer"}` dan header menyebut `kid: johar-current-01`. Pencarian endpoint fungsi pedagang secara brute force path menemukan `/merchant` merespons 403 "Akses merchant ditolak" (route ada, tapi perlu role tertentu).
2. Source map JavaScript (`/static/app.js.map`), yang seharusnya tidak ikut ter-deploy ke produksi, membocorkan isi file sumber `config/legacy.js` berisi komentar bahwa kunci lama dengan `kid: johar-legacy-01` "masih diterima verifikator" beserta nilai secret-nya secara penuh.
3. Dengan secret lama tersebut, token baru dipalsukan memakai `kid: johar-legacy-01` dan klaim `role` diubah menjadi `merchant` (nama role ditebak dari teks pesan penolakan). Token palsu ini diterima oleh `/merchant` dan membuka Dasbor Merchant — sebuah **privilege escalation** murni dari sisi klien karena server memercayai klaim `role` di dalam JWT tanpa verifikasi tambahan terhadap akun aslinya.
4. Dasbor Merchant menyebut lokasi file laporan (`/srv/johar/internal/laporan-final.txt`) dan menyediakan fitur "pratinjau template" di `/merchant/preview?template=`. Payload `{{7*7}}` dievaluasi menjadi `49`, mengonfirmasi **Server-Side Template Injection** (Jinja2).
5. Payload SSTI standar ala Flask (`self`, `config`, `request`, bahkan fungsi global `lipsum`) semuanya gagal — environment Jinja2 dikonfigurasi tanpa globals bawaan. Namun akses atribut dasar seperti `__class__` dan `__mro__` tidak diblokir, sehingga rantai `''.__class__.__mro__[1].__subclasses__()` tetap bisa dipakai untuk mendaftar seluruh kelas Python yang sudah dimuat di proses tersebut, termasuk `subprocess.Popen`.
6. Payload akhir memanggil kelas `Popen` tersebut langsung dari indeksnya di daftar subclass untuk menjalankan `cat` atas file laporan dengan `shell=True`, membaca hasilnya lewat `.communicate()`. Respons memuat isi file laporan sekaligus flag.

Rantainya: source map bocor ke produksi → secret JWT lama yang sengaja masih berlaku demi kompatibilitas mundur → server memercayai klaim role di token tanpa verifikasi lanjutan → endpoint pratinjau template ternyata me-render input pengguna langsung sebagai Jinja2 (SSTI) → upaya hardening (globals dikosongkan) masih bisa dilewati lewat model objek Python murni untuk mencapai eksekusi perintah.

---

## 18. Soto Semarang (Tidak Terselesaikan)

**Kategori:** Miscellaneous · **Poin:** 125 (tidak diperoleh)

Soal memberi arsip `misc-soto-semarang-lampiran.zip` berisi `migration.txt` dan `receipts.csv` (113 baris, kolom `receipt,position,label,value,state`), dengan narasi: "Tampilan kasir memperlihatkan dua label yang nyaris sama. Audit menemukan satu kanal data tersembunyi pada ekspor struk. Pulihkan pesan dari struk aktif." Petunjuk berbayar (−10 poin) mengonfirmasi bahwa perbedaan kedua label terletak pada *code point* Unicode.

**Yang berhasil dipastikan:**

1. Kanal tersembunyi adalah dua varian label "kuah hangat" yang identik secara visual: satu memakai spasi biasa (U+0020), satu memakai *zero-width space* (U+200B) di antara kedua kata.
2. Baris "aktif" didefinisikan sebagai `state == approved` dengan label ber-U+200B. Tepat ada 65 baris seperti ini, satu untuk setiap `position` 1 sampai 65 tanpa celah maupun duplikat — cocok dengan panjang pesan 65 karakter.
3. Seluruh 12 baris `state == void` memakai label ber-U+200B juga, dan `migration.txt` menyatakan struk *void* tidak diekspor ke pembukuan, sehingga baris-baris ini disimpulkan sebagai pengecoh. 35 baris `approved` berlabel spasi biasa, yang bertumpuk di posisi yang sama dengan baris aktif, juga disimpulkan sebagai pengecoh.
4. Analisis statistik pada kolom `value` dari 65 baris aktif (diurutkan menurut `position`) menemukan bahwa meng-XOR-kan nilainya dengan *keystream* linear `(13 × position + b) mod 256` menghasilkan 65 byte yang **seluruhnya** berada pada rentang ASCII tercetak (32–126) untuk tepat dua nilai `b` (250 dan 251) — dan, krusialnya, **slope 13 adalah satu-satunya** dari 256 kemungkinan yang punya solusi sama sekali pada `b` berapa pun. Temuan ini secara statistik praktis mustahil kebetulan, sehingga slope 13 diyakini memang bagian dari skema enkripsi aslinya.

**Yang belum terpecahkan:** Hasil XOR dengan `b=250` maupun `b=251` tetap berupa untaian karakter tercetak tapi tidak terbaca sebagai teks maupun format flag `CF2{...}`. Berbagai transformasi lanjutan dicoba tanpa hasil: operasi XOR/penjumlahan/pengurangan terhadap `position`, ID struk, indeks baris, dan kunci berulang; skema affine penuh pada berbagai modulus; known-plaintext dengan asumsi awalan `CF26{`; dekode Base64/hex/zlib; serta pencarian lapis kedua di atas hasil XOR slope-13 tersebut. Pendekatan berbasis bit label (U+200B sebagai 1, spasi sebagai 0) dengan berbagai urutan pengelompokan dan arah baca juga tidak menghasilkan teks bermakna.

Tiket bantuan ke panitia (tiket #32, berisi ringkasan temuan slope-13 di atas) telah dikirim namun belum mendapat balasan hingga kompetisi ditutup. Soal ini dicatat sebagai satu-satunya tantangan yang tidak terselesaikan dari 18 soal yang dikerjakan tim.