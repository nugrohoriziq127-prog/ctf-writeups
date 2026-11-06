# Log Solve HI CyberCon CTF 2026

Solo, divisi Agentic AI. Kickoff: Jumat 30 Okt 2026 21:00 WIB. Berakhir: Minggu 1 Nov 2026 21:00 WIB.
Platform: ctf.hiddeninvestigations.net. Hadiah total USD 2.500 (top 5), 70+ soal, 12 kategori.

Catat LANGSUNG saat menyelesaikan soal. Log ini dokumen kerja pribadi selama lomba.
Versi publik untuk GitHub (paket writeup November) baru disusun SETELAH lomba berakhir 1 Nov 21:00 WIB,
bukan saat lomba masih berjalan.

## Checklist sebelum kickoff 21:00 WIB

- [ ] Login platform berhasil
- [ ] Divisi Agentic AI terpilih di dalam platform (satu-satunya sisa registrasi yang belum terkonfirmasi)
- [ ] Folder bukti disiapkan (misal: `hi-cybercon/bukti/soal-01/`)

## Cara kerja tim Agentic AI selama 48 jam

- Delegasikan soal ke Muse dengan: nama soal, kategori, deskripsi, dan file/URL yang relevan.
- Verifikasi output Muse sebelum submit flag (aturan lomba soal scope dan fair play tetap berlaku
  meski bantuan AI tidak dibatasi).
- Tulis di blok soal: apa yang didelegasikan, apa yang diverifikasi, dan apa yang diubah dari saran awal.

## Per soal (salin blok ini untuk tiap soal yang diselesaikan)

### Soal #__: [nama soal]

- Kategori:
- Poin:
- Waktu mulai:            Waktu selesai:
- Ringkasan masalah (1-2 kalimat):
- Pendekatan:
- Yang didelegasikan ke Muse:
- Hasil verifikasi sebelum submit (apa yang dicek, apa yang diubah):
- Langkah solve:
  1.
  2.
  3.
- Tools/commands kunci:
- Flag: `FLAG{...}`
- Bukti: [daftar screenshot/file di folder bukti]
- Kesulitan utama dan cara mengatasinya:
- Pelajaran buat soal berikutnya:

---

## Checklist penutup 1 Nov 21:00 WIB

- [ ] Semua blok soal terisi lengkap
- [ ] Semua flag tercatat dan bisa diverifikasi ulang
- [ ] Semua bukti terkumpul dan terlabel per soal
- [ ] Bahan writeup November dipilih dari log ini (versi publik dirapikan setelah lomba berakhir)

---

*Disiapkan 30 Okt 2026, siang hari kickoff, sebagai pendamping log writeup Cyber Frontier yang sudah ada sejak 23 Okt 2026.*

### Soal #1: Cold Beacon

- Kategori: Reverse Engineering
- Poin: 85 pts (dinamis; 137 pts saat pertama dilihat, terpotong solves yang masuk selama analisis; CakraWatch solver ke-94)
- Waktu mulai: 21:10 WIB  Waktu selesai: ±21:17 WIB (30 Okt 2026)
- Ringkasan masalah: Binary ELF x86-64 (ternyata hasil kompilasi Nim, stripped) menerima satu argumen kunci 24 karakter; kunci benar membuat binary mencetak config yang dibawanya.
- Yang didelegasikan ke Muse: analisis statis penuh (strings, disassembly, identifikasi pemeriksa kunci), ekstraksi tabel persamaan dari .rodata, dan penyelesaian sistem persamaannya.
- Yang diverifikasi Nugroho: menjalankan binary dengan kunci hasil analisis dan membaca output-nya sebelum submit.
- Langkah penyelesaian:
  1. SHA-256 zip cocok dengan deskripsi soal. Binary dicek: ELF 64-bit, stripped; string output tidak tersimpan plaintext (dibangun saat runtime).
  2. Ditemukan anti-debug: binary membaca /proc/self/status dan memeriksa TracerPid, jadi analisis dilakukan statis, bukan dengan debugger.
  3. Fungsi pemeriksa kunci memakai tabel di .rodata: untuk tiap posisi kunci ada operasi subset (XOR / ADD / MUL pasangan) yang harus sama dengan byte target; pemeriksa kedua menambah 3 relasi antar byte kunci.
  4. Sistem persamaan diselesaikan (rantai ADD antar posisi berurutan, XOR, dan MUL); solusi parsialnya terbaca sebagai pola leetspeak "C?l?B??c0n::unl0ck::7a3f" sehingga kunci lengkap tertebak: C0ldB34c0n::unl0ck::7a3f.
  5. Seluruh persamaan terverifikasi cocok, lalu binary dijalankan dengan kunci itu dan mencetak flag.
- Tools: objdump, Python 3 (penyelesai sistem persamaan), strings.
- Flag: hiCTF{c0ld_b34c0n_cfg_uns34l3d_fr0m_th3_s4mpl3}
- Folder bukti: file soal cold-beacon.zip (SHA-256 a01791f8259c7bcf1eb07f578abc686b2a4ab78d4fe410f565c74624e52610d8)

### Soal #2: Epsilon Drift

- Kategori: Reverse Engineering
- Poin: dinamis (106 pts saat diambil)
- Waktu mulai: 21:19 WIB  Waktu selesai: ±21:24 WIB (30 Okt 2026)
- Ringkasan masalah: Validator ELF x86-64 statis (C/glibc) menerima satu argumen hex 64 karakter = vektor kalibrasi 32 byte; vektor yang tepat membuat binary mencetak run config.
- Yang didelegasikan ke Muse: analisis statis penuh (alur parse, pemeriksa vektor, fungsi dekripsi), penurunan sistem persamaan bit, dan perhitungan vektor.
- Yang diverifikasi Nugroho: menjalankan binary dengan vektor hasil analisis dan membaca output-nya sebelum submit.
- Langkah penyelesaian:
  1. SHA-256 zip cocok dengan deskripsi soal. Semua string output terenkode (pesan gagal "calibration residual out of band" = XOR 0x6b), jadi analisis murni dari disassembly.
  2. Argumen di-parse sebagai 64 karakter hex menjadi 32 byte, lalu ditafsirkan sebagai 4 double IEEE 754.
  3. Pemeriksa membedah bit v0 dan v1 menjadi limb mantissa 13-bit (bit 0-12, 13-25, 26-38, 39-51) plus field eksponen 11-bit, dan memaksakan untuk tiap limb pasangan persamaan (A+B = konstanta) dan (A+2B = konstanta); tiap pasangan berdeterminan 1 sehingga solusinya unik per limb. Hasilnya: mantissa v0 = mantissa v1, eksponen berbeda 1.
  4. Syarat lain: v2 harus sama persis bit-nya dengan hasil penjumlahan floating point v0+v1, dan v3 harus -0.0 (nol bertanda; pemeriksaan kesetaraan biasa tidak membedakan +0.0 dan -0.0, tetapi uji bit tanda di akhir memisahkannya).
  5. Vektor tersusun: v0 = 1.1, v1 = 2.2, v2 = 3.3000000000000003, v3 = -0.0, hex = 9a9999999999f13f9a999999999901406766666666660a400000000000000080. Binary dijalankan dengan vektor itu dan mencetak flag (dekripsi memakai kunci turunan SHA-256 dari vektor, plaintext-nya diverifikasi binary terhadap digest tersimpan).
- Tools: objdump, Python 3 (struct IEEE 754 + aljabar limb).
- Flag: hiCTF{1eee754_s1gn3d_z3r0_1u1p_dr1ft_eps!}
- Folder bukti: file soal epsilon-drift.zip (SHA-256 993a5ed3ced717af48dedf7b18fd54396ba55e314ea4519613a96a9c6b05284f)

### Soal #3: Night Batch (BELUM SELESAI — flag kandidat DITOLAK platform)

- Kategori: Reverse Engineering
- Poin: 106 pts (61 solves saat dibuka)
- Waktu mulai: 21:29 WIB (30 Okt 2026), masih terbuka
- Ringkasan masalah: Load module hasil kompilasi COBOL (GnuCOBOL) memvalidasi account key dari argumen baris perintah; kunci yang benar "clears the step", sisanya ditolak dengan pesan "account rejected".
- Status temuan:
  1. SHA-256 zip cocok dengan deskripsi soal. Binary = program GnuCOBOL (string EC-I-O-INVALID-KEY, EC-IMP-ACCEPT, runtime libcob); pesan keluaran terenkode XOR (pesan gagal "account rejected" = blob di 0x44d0a0 XOR 0x5a, dicetak fungsi 0x402780).
  2. Fungsi dekripsi C di 0x402f40 (pola sama dengan Epsilon Drift): gerbang argumen penanda (esi, dihitung logika COBOL); bila lolos, keystream diturunkan dari SHA-256 atas 18 byte pertama field kunci (field argumen mentah di 0x460140) lewat KDF berantai, mendekripsi 42 byte ciphertext di 0x44d0c0, dan mencetak plaintext hanya bila SHA-256-nya cocok digest tersimpan (bd3f6ab64668142c88e427a5ef541618dbea3a32f68fbc6d8fa0f6aeef80a6b0).
  3. Uji patch (gerbang di-NOP pada salinan binary): dekripsi selalu berjalan tetapi SEMUA input uji gagal digest, jadi kunci benar memang menentukan hasil dekripsi; tidak ada jalan pintas paksa-gerbang.
  4. KOREKSI: teks "hiCTF{n16h7_b47ch_k3y_r3c0v3r3d}" yang sempat terkirim sebagai flag berasal dari sesi ptrace yang cacat (panggilan yang dikira SETREGS ternyata PTRACE_DETACH, proses lepas di tengah instruksi) dan DITOLAK platform. Bukan flag resmi; kemungkinan teks umpan/artefak. Jangan dipakai.
  5. Sisa pekerjaan: ekstrak logika pemeriksaan COBOL yang menghitung penanda lolos (esi) dari argumen, turunkan kunci akun 18 byte-nya, verifikasi dengan menjalankan binary sampai mencetak flag 42 byte yang lolos digest.
- Tools: objdump, Python 3 (ctypes ptrace), LD_PRELOAD memory dump untuk memetakan field.
- Folder bukti: file soal night-batch.zip (SHA-256 64ad99c6d690a7fac13b7d7cb56e2cc7c8553feceda2b503305dcb3ff6afcc9c)

### Soal #4: Shadow Identity

- Kategori: OSINT
- Waktu selesai analisis: ±21:42 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Satu foto dari perangkat tersangka; identifikasi orang di foto; flag adalah nama lengkap huruf kecil.
- Langkah penyelesaian:
  1. Zip (SHA-256 cocok) hanya berisi miss_grand.jpeg; tidak ada metadata EXIF/GPS sama sekali, jadi identifikasi murni dari konten visual.
  2. Foto adalah poster promosi resmi bertuliskan "MISS GRAND TANZANIA 2026", dengan bendera Tanzania, latar Gunung Kilimanjaro, dan pemenang bermahkota bergaun hijau.
  3. Penelusuran web: Miss Grand Tanzania 2026 digelar 8 Agustus 2026 di The Super Dome, Masaki, Dar es Salaam; pemenangnya, perwakilan Mwanza, adalah Jihan Dimack (tabel hasil Wikipedia "Miss Grand Tanzania 2026"; The Citizen, 9 Agustus 2026).
  4. Korroborasi independen: orang yang sama kemudian memenangkan Miss Grand International 2026 di Bangkok pada 10 Oktober 2026, jadi identitas perempuan di poster tidak ambigu.
- Catatan: nama legal penuhnya Jihan Mohin Dimachk (nama panggung/kompetisi: Jihan Dimack). Bila checker menolak varian pertama, coba varian kedua.
- Flag kandidat: hiCTF{jihan_dimack} (cadangan: hiCTF{jihan_mohin_dimachk})
- Folder bukti: file soal shadow-identity.zip (SHA-256 d5ad9fe2781bd6f933b6c7d1e9480c1ef80f3dab1c55819016be9271c521afae)

### Soal #5: Obfuscated Flow

- Kategori: Reverse Engineering
- Poin: 111 pts saat dibuka (58-61 solves)
- Waktu mulai: 21:26 WIB  Waktu selesai: ±21:44 WIB (30 Okt 2026)
- Ringkasan masalah: Validator "Sentinel Gateway" (ELF x86-64) diobfuscate control-flow flattening + MBA; passkey yang diminta adalah flag itu sendiri.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Format passkey dipastikan dari main: tepat 47 karakter, awalan hiCTF{ dan akhiran }, isi 40 karakter dibaca sebagai 10 kata u32 little-endian.
  2. Dispatcher terflatten berjalan 7 state; tiap blok state berakhir pada indeks yang harus 0 (tabel state berikutnya berisi satu state sah dan sampah 0x??badc0d di slot lain). Blok-blok juga meng-XOR material pasangan kata ke state bergulir berseed IV MD5; blok final mendekripsi pesan dengan state itu dan mensyaratkan hash djb2 == 0xb4f079c8 sebelum mencetak sukses.
  3. Gerbang per pasangan kata diekstrak statis lalu diselesaikan: (w6,w7) dua persamaan linear mod 2^32-5; (w8,w9) jumlah dan hasil kali mod 2^31-1 (akar kuadratik); (w0,w1) sistem menghasilkan kuartik dengan solusi printable unik; (w2,w3) persamaan kurva eliptik mod 1e9+7 plus gerbang pangkat 128-bit (eksponen 1337), disaring dari kandidat kata terbaca lalu diuji oracle; (w4,w5) dua logaritma diskret mod 2^31-1 dipecahkan Pohlig-Hellman (2^31-2 berfaktor halus).
  4. Hasil kata per kata: m4th _c0n tr0l _fl0 w_3l l1pt 1c_d log_ 4f7b 2c9e, membentuk frasa "math control flow elliptic dlog" + sufiks hex.
  5. Verifikasi: binary asli diberi passkey ini lewat stdin, mencetak "[+] ACCESS GRANTED! Passkey verified." dan flag-nya, exit 0. Urutan w8/w9 terbalik ditolak (Access Denied), mengonfirmasi urutan byte persis.
- Tools: objdump, Python 3 + z3 (aljabar modular, Pohlig-Hellman), harness Unicorn sebagai oracle penelusur blok; percobaan angr ditinggalkan karena tidak konvergen.
- Flag: hiCTF{m4th_c0ntr0l_fl0w_3ll1pt1c_dlog_4f7b2c9e}
- Folder bukti: file soal rev-obs-flow.zip (SHA-256 ddaca1f93d3f748482982814ed307b58f9fba73e6298bb4551594a2b344a2d30)

### Soal #6: Last Execution

- Kategori: Reverse Engineering
- Poin: 195 pts saat pertama dilihat (23 solves); menyusut ke 95 pts / 74 solves pada 21:49 WIB
- Waktu selesai analisis: ±21:50 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Satu image disk EXE9CORE bertahan dari eksekusi terakhir EXE-9; berisi kode rusak, tabel terpuntir, dan segel yang tidak seharusnya ditemukan. Image dipenuhi flag decoy hiCTF{f4k3_fl4g}.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Struktur image dipetakan: header 0x100 byte (EXEC_SEED 0x9E3779B9F00DCAFE, TABLE_SEED 0xBADC0FFEE0DDF00D, ENTRY=1) + tabel 24 segmen (record 64 byte: id, tipe, offset, size, flags, AUX, CRC32, nama). Semua CRC segmen cocok.
  2. Segmen PRIMER adalah manual VM, tetapi berisi 3 kesalahan cetak yang disengaja; ketiganya dikoreksi dari trace encode-time di dalam PRIMER sendiri: rantai rotasi halaman ROL awal 17 (bukan 19) dengan update ROL(((m^w^X)*K),31); rantai rotasi baris tabel memakai i*13 (bukan i*11); dispatch idx = (op + (pc>>3) + AUX) mod 256 (bukan pc>>4).
  3. Emulator VM EXE-9 dibangun (fetch terotasi per halaman, dispatch terotasi + fallback PSH, LDRR tabel terotasi, rantai HALT) dan divalidasi: menjalankan EXEC_SELFTEST menghasilkan SHA-256(R1..R4) yang persis sama dengan commitment di PRIMER.
  4. Rantai eksekusi EXEC_MAIN → FOLD_A → FOLD_B → LIMIT → FINAL → LIMIT → FINAL dijalankan; EXEC_LIMIT adalah loop raksasa (penghitung 0x123456789ABC, ±2×10^13 iterasi) yang di-fast-forward secara analitik (siklus LCG 18-bit + siklus akumulator), dengan model loop divalidasi identik melawan simulasi langsung 500 ribu iterasi.
  5. Jebakan TELEMETRY dikenali: register file-nya hampir benar tetapi R2 meleset satu nibble; kunci dari state versi telemetry membuka TEST_SEAL_2 menjadi decoy hiCTF{f4k3_fl4g}. State hasil emulasi terverifikasi dipakai sebagai gantinya.
  6. Kunci segel = SHA-256("LASTEXEC" || be64(R1..R4) || "LASTEXEC") dari state final; SBOX diturunkan dari baris logis DLOOK (Fisher-Yates); WILL_SEAL di-undo rotasi kolomnya, didekripsi SEAL-256, dan di-strip PKCS#7 hingga menghasilkan flag.
- Tools: Python 3 (parser image, emulator VM, solver segel) di folder kerja pekerja latar.
- Flag: hiCTF{3x3c_9_r3m3mb3rs_4ll_but_th3_l4st_h4lt_l13s}
- Folder bukti: file soal last-execution.zip (SHA-256 3b82c94be14529c9daacd9709e718e2c0337c11537e1889ef26f8ab16a01bc85)

### Soal #3 (lanjutan): Night Batch — TUNTAS

- Status akhir: SOLVED ±21:52 WIB (30 Okt 2026), menunggu submit Nugroho
- Kunci akun: 481702596318740925
- Flag final: hiCTF{c0b0l_p4ck3d_d3c1mal_n1ghtly_b4tch!}
- Jalur penyelesaian pekerja latar (menindaklanjuti analisis statis awal):
  1. Argumen wajib tepat 18 karakter dan lulus uji kelas IS-NUMERIC, jadi kuncinya 18 digit; loop program memindahkan tiap digit ke 18 field satu digit COMP-3 di 0x460100 (tiap byte = digit<<4 | 0xF, dikonfirmasi dinamis via ptrace).
  2. Validasi terdiri dari 23 blok aritmetika desimal (helper aritmetika field libcob): tiap blok mengevaluasi ekspresi atas digit-digit, mengambil REM 10, dan mensyaratkan sama dengan digit cek tetap. Pola bobot (3,1,2,3,1) bergeser melintasi jendela 5 digit yang tumpang tindih, ditambah cek titik seperti (d1*d10)%10==8.
  3. Blok terakhir lebih kuat: (4*d3 + 5*d6 + 5*d10 + 2*d15 + 9*d16) mod 100 == 94.
  4. Sistem 23 persamaan atas d0..d17 (masing-masing 0..9) memiliki solusi unik; ditemukan dengan z3 menghasilkan kunci di atas. Lolos semua blok menyalakan byte gerbang untuk pemanggilan fungsi unlock C, yang pemeriksaan digest SHA-256 internalnya kemudian lolos.
  5. Verifikasi: binary ASLI dijalankan dengan argumen kunci tersebut mencetak flag (bukan "account rejected"). Teks hiCTF{n16h7_b47ch_k3y_r3c0v3r3d} dipastikan BUKAN flag (artefak bug ptrace lama).
- Tools: objdump, Python 3 + z3, ptrace untuk konfirmasi format field COMP-3.
- Folder bukti: file soal night-batch.zip (SHA-256 64ad99c6d690a7fac13b7d7cb56e2cc7c8553feceda2b503305dcb3ff6afcc9c)

### Soal #7: VaultKeeper

- Kategori: Android
- Poin: 114 pts saat dibuka
- Waktu selesai analisis: ±22:00 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: APK "vault" paling aman; saldo di dalam; master key diklaim satu ketukan jauhnya, tetapi hanya untuk "seseorang". Aplikasi dipenuhi flag umpan berlapis.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Semua flag yang terlihat terbukti umpan: fungsi native getFlag (tanpa caller), FlagRepository.getMasterKey (XOR enam array, hanya teks vault id UI), dan lima string dari resources.arsc termasuk beta_key_fragment berprefix salah.
  2. UI tidak pernah menampilkan flag. Gerbangnya ritual: ketuk shield 3x (jeda <=700 ms), lalu tekan-tahan subtitle >=600 ms dalam jendela 1-1500 ms setelah ketukan ketiga, menghasilkan token SecureRandom 16 byte dan jendela armed 3000 ms untuk ClaimOrchestrator.
  3. Pipeline native libvaultnative.so ter-reverse penuh: nv_seed menurunkan 128 sel dari SHA-256 atas assets/vault.bin; CellMixer.k1 menjalankan 7 ronde pencampuran; nv_unseal membuka program 0x800 byte; CellMixer.k2 meng-XOR program; nv_exec menjalankan VM; nv_final merakit material 51 byte dan memverifikasinya dengan SHA-256 terhadap commitment tertanam.
  4. Wawasan penentu: token hanya menggerbangi rantai state dan tidak pernah masuk jalur flag, jadi flag adalah fungsi murni vault.bin dan dapat dipulihkan 100% offline dengan pembuktian mandiri lewat cek digest nv_final.
  5. Eksekusi: .so Android dimuat di Linux lewat stub libc/libdl; satu-satunya simbol yang gagal adalah __strrchr_chk (tidak diekspor glibc), diatasi dengan implementasi lokal semantik bionic di dalam stub. Pipeline asli lalu berjalan utuh (stage 1→2→3, fault flag 0).
  6. Verifikasi: SHA-256("VK-TAG-2026" || material)[:16] cocok persis dengan commitment di .so (DIGEST MATCH: True), nv_final mengembalikan 0, dan proses segar dengan token berbeda menghasilkan material byte-identik.
- Tools: analisis statis dex + disassembly native, Python + ctypes dengan stub library bionic.
- Flag: hiCTF{!!I_10v3_4ndr0!d_m4n_m4k!ng_th3m_f331_g0Od!!}
- Folder bukti: file soal VaultKeeper.zip (SHA-256 7c0ccbd218561af27d1f51dcb18cf3ef5ab566e538788d88e285de1fb8d7aa10)

### Soal #8: Forgotten Secrets

- Kategori: Miscellaneous
- Poin: 63 pts (224 solves)
- Waktu selesai analisis: ±22:08 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Dump biner 256 KB dari memori server; flag terobfuscate dan tidak muncul di pencarian string biasa.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Pencarian string langsung hanya menemukan umpan plaintext ("This is not the flag you are looking for", "flag_not_here_keep_looking").
  2. Brute-force XOR satu byte untuk pola hiCTF{ menghasilkan tepat satu kunci: 0x42, pertama di offset 143838.
  3. Dekode seluruh dump dengan XOR 0x42 menghasilkan flag yang sama persis di 22 offset berbeda, mengonfirmasi bukan kebetulan.
- Tools: Python 3 (analisis byte + brute-force XOR).
- Flag: hiCTF{m3m0ry_dump_d1g_d33p}
- Folder bukti: file soal forgotten-secrets.zip (SHA-256 bfde7ee94bc2841f3c9fc6275e6f005256df9d1a77363f291ec8adc90929840a)

### Soal #4 (lanjutan): Shadow Identity — kandidat putaran kedua

- Status: kandidat baru siap submit per 22:08 WIB (30 Okt 2026)
- Yang sudah gugur di halaman yang benar: jihan_dimack, jihan_dimachk, jihan_mohin_dimachk, beatrice_alex_akyoo, beatrice_alex.
- Temuan putaran kedua: poster ini adalah bagian dari seri poster blog pageant "Mayor of Pageants" (@mayor_of_pageants); slide pengumuman pemenang Miss Grand Tanzania 2026 mereka menggambarkan potret yang sama persis (mahkota perak berpermata hijau, anting hijau senada, gaun hijau tua berpayet, bendera Tanzania) dengan teks nama Jihan. Jadi orangnya memang Jihan, tetapi bentuk namanya belum tepat.
- Bentuk yang belum pernah dicoba: nama lengkap "Jihan Mohin Dimack" (nama tengah Mohin dengan ejaan Dimack), terbukti dipakai laporan elysplanet.ph tentang Miss Grand International 2026. Bentuk tiga token ini juga cocok dengan contoh format di deskripsi soal.
- Flag kandidat: hiCTF{jihan_mohin_dimack}
- Cadangan (spekulatif, tanpa bukti poster): hiCTF{prisca_noel_lyimo} (Prisca Lyimo, runner-up 1 / Miss Cosmo Tanzania 2026).
- Catatan: pencarian gambar terbalik langsung tidak dapat dijalankan pekerja karena unggahan penyimpanan file terkena rate limit; pencocokan dilakukan lewat ringkasan pencarian sosial atas grafis Mayor of Pageants.

### Soal #9: Abandoned Tracker

- Kategori: Miscellaneous
- Poin: 64 pts (204 solves)
- Waktu selesai analisis: ±22:10 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Forensik offline murni; rekonstruksi aturan skor "tempat yang paling sering didatangi" dari catatan, config bobot AP tepercaya, satu ekspor scan, dan log satu-satunya run yang berhasil.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Dari run.log: baris dengan step_delta 40 dibuang (stationary) dan 41 disimpan (moving), konsisten di dua AP, jadi cutoff-nya bergerak bila step_delta > 40.
  2. Rumus skor divalidasi dari dua subtotal log: CafeDelMundo (bobot 2) baris bergerak bersinyal 58+60+64 = 182, x2 = 364 (persis subtotal log); HomePlex_5G (bobot 2) hanya sinyal 87 yang lolos, x2 = 174 (persis). Jadi skor AP = bobot x jumlah sinyal pada baris bergerak, hanya untuk AP di aps.conf.
  3. Diterapkan ke seluruh ekspor: FoxgloveCoffee_Guest (bobot 3) 5 baris bergerak, sinyal 348, skor 1044; CafeDelMundo 364; NETGEAR-8841 240; HomePlex_5G 174.
  4. Jebakan terhindari: xfinitywifi terkeras (sinyal 96) tetapi tidak ada di daftar tepercaya; HomePlex_5G paling sering muncul (10 baris) tetapi hampir selalu stasioner.
- Tools: Python 3 (CSV scoring).
- Flag: hiCTF{FoxgloveCoffee_Guest}
- Folder bukti: file soal abandoned-tracker.zip (SHA-256 e76d9c898c069fa96e85d929009f3bd16e94a747940e90f7a23df2424ef3e5e4)

### Soal #10: Ghost in the Wire

- Kategori: Forensics
- Poin: 59 pts (316 solves)
- Waktu selesai analisis: ±22:44 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Lalu lintas ICMP anomali dalam sebuah packet capture; data dieksfiltrasi menyatu dengan noise jaringan normal.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Capture berisi 30 paket, semuanya ICMP echo request (type 8), terbagi dalam dua aliran.
  2. Aliran noise 192.168.1.100 → 10.0.0.5 membawa payload ping yang wajar ("ping from server1", "health check ok", deretan huruf A).
  3. Aliran eksfiltrasi 10.13.37.1 → 10.13.37.2 berisi 26 paket (paket #2-#27) yang payload-nya masing-masing hanya satu byte, sangat tidak standar untuk ping sungguhan.
  4. Merangkai payload satu byte itu menurut urutan sequence ICMP (1-26) mengeja flag secara verbatim tanpa dekode tambahan.
- Flag: hiCTF{1cmp_3xf1l_d3t3ct3d}
- Folder bukti: file soal ghost-in-the-wire.zip (SHA-256 392084661bebf2a86f4194056436ddb2b78917577bc259cc07e1210f007c563c)

### Soal #11: Broken Signals

- Kategori: Forensics
- Poin: 58 pts (338 solves)
- Waktu selesai analisis: ±22:44 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: File intersepsi tidak bisa dibuka aplikasi standar mana pun; ada yang salah dengan file itu sendiri.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. File ternyata PNG 400x300 (16-bit RGBA) yang 4 byte pertama signature-nya (89 50 4E 47) di-nol-kan, menyisakan 00 00 00 00 0D 0A 1A 0A, cukup untuk menggagalkan deteksi tipe file dan semua pembuka standar.
  2. Seluruh chunk lain (IHDR, cHRM, bKGD, tIME, tEXt, IDAT, IEND) ternyata utuh dan lolos verifikasi CRC; tidak ada data ekor.
  3. Memulihkan 4 byte signature membuat PNG valid kembali; pesan tersembunyi tergambar di dalam image itu sendiri ("SIGNAL INTERCEPTED" dengan flag tercetak merah di bawahnya).
- Flag: hiCTF{h34d3r_r3p41r_m4st3r}
- Folder bukti: file soal broken-signals.zip (SHA-256 47517fa29db01823c9416f0b285f069e2022d63c956d87fb995c1b97a9ea34f5); PNG hasil perbaikan disimpan pekerja di workspace.

### Soal #12: Memory Liar

- Kategori: Forensics
- Poin: 61 pts (262 solves)
- Waktu selesai analisis: ±22:46 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Capture memori fisik workstation yang diduga dikompromikan APT; audit proses otomatis melaporkan semuanya bersih.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. Image adalah dump bergaya Windows rakitan 16 MB (Volatility tidak bisa memindainya), jadi blok-blok EPROCESS diurai manual: pool block berjarak tetap 0x8000, ImageFileName di +0x174, PID di +0xBC, tautan Flink/Blink di +0xC0/+0xC4.
  2. Satu-satunya flag plaintext di image adalah umpan yang mengaku sendiri palsu, tertanam di "incident log" dalam region notepad.exe yang mengklaim integritas sistem tervalidasi.
  3. Menelusuri rantai daftar proses (smss sampai notepad) tidak pernah mencapai winxnet.exe (PID 1337): blok EPROCESS-nya ada di pool, tetapi Flink/Blink-nya menunjuk ke dirinya sendiri, pola klasik DKOM unlink yang tak terlihat enumerasi pslist dan hanya ketahuan lewat pool scan (cross-check psscan).
  4. Region winxnet.exe memuat artefak konfigurasi APT (C2 c2.darkshadow-apt.org:8443, penanda OP_SHADOW_LIAR_ACTIVE). Di blob konfigurasinya (offset file 0x35F010, header ROOT9, kunci dword tersimpan di sampingnya) flag terenkode XOR satu byte 0x5D; dekode menghasilkan satu-satunya hit hiCTF{...} yang sah di seluruh image.
- Tools: Python 3 (parsing manual struktur EPROCESS dan pool scan).
- Flag: hiCTF{dk0m_unl1nk_3pr0c3ss_l1st_cr0ss_ch3ck_7c3a9f}
- Folder bukti: file soal memory-liar.zip (SHA-256 26a5e19d3e6a3c591e008abf1d644ecab735197d06c5c1e2b4a42e64183c1414)

### Soal #13: Cipher Cascade

- Kategori: Cryptography
- Poin: 57 pts (394 solves)
- Waktu selesai analisis: ±22:51 WIB (30 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Transmisi terenkripsi berlapis-lapis; kupas setiap lapisan untuk membuka flag.
- Langkah penyelesaian:
  1. SHA-256 zip cocok. File cipher_cascade.txt menyatakan urutan lapisannya sendiri: Reverse → Base64 → Hex → ???.
  2. String dibalik, lalu di-decode Base64 menghasilkan string hex; decode hex menghasilkan uvPGS{y4l3e_ol_y4l3e_haj1aq}.
  3. Lapisan terakhir yang misterius adalah ROT13; menerapkannya menghasilkan flag.
- Tools: Python 3 (base64, hex, ROT13).
- Flag: hiCTF{l4y3r_by_l4y3r_unw1nd}
- Folder bukti: file soal cipher-cascade.zip (SHA-256 d7245a4bbf50857739833d75890ae001a9ed5cbee1f7eb5873171f1234c02631)

### Soal #14: C3 Vault

- Kategori: Hardware
- Poin: 90 pts (84 solves)
- Waktu selesai analisis offline: ±23:10 WIB (30 Okt 2026), flag live menunggu host:port instans dari user
- Ringkasan masalah: Gateway ESP32-C3 pensiun; pulihkan kredensial enrollment dari citra flash SPI dan jendela register eFuse, lalu yakinkan layanan live bahwa kita adalah gateway tersebut. Flag hanya ada di instans live.
- Langkah penyelesaian:
  1. Pemetaan blok: jendela register eFuse mengikuti tata letak ESP32-C3; hanya KEY0 (BLOCK4) yang terisi. Kunci XTS-AES-128 flash adalah blok itu dengan byte terbalik; tabel partisi di offset 0x8000 terdekripsi bersih dengan espsecure mode aes_xts.
  2. Partisi: nvs @0x9000 (tidak terenkripsi), nvs_keys @0xF000 (terenkripsi), factory @0x10000. Bootloader dan app (c3_vault_device, ESP-IDF v5.3.1) terdekripsi menjadi image ESP valid.
  3. Kunci NVS dari partisi nvs_keys terdekripsi dipakai membuka entri NVS (XTS per sektor, tweak = alamat byte relatif). Kredensial pulih: gateway/device_id (blob 16 byte), generation 7, hostname c3-vault-gw, dan seed namespace vault_v2 (yang dipakai app saat ini; vault_v1 adalah versi lama).
  4. Disassembly fungsi enrollment pada app terdekripsi memberi rumus persis: DK = HMAC-SHA256(seed_v2, label "c3-vault/device-key/v2" + NUL + device_id + BE32(generation)); respons tantangan = HMAC-SHA256(DK, "C3VAULT-AUTH-V2" + NUL + challenge 32 byte + device_id). DK terhitung dan skrip auth terverifikasi end-to-end terhadap verifier tiruan lokal.
- Tools: Python, espsecure (XTS-AES), parser NVS kustom, disassembly image ESP32-C3.
- Skrip siap tembak: files/auth_c3vault.py, pemakaian python3 auth_c3vault.py <host> <port>.
- Flag: MENUNGGU endpoint instans dari panel soal (tidak ada di artefak unduhan).
- Folder bukti: file soal c3_vault.zip (SHA-256 kartu ef94f82b12ebfb1640f9e52d6a74c6f514c4ea718bc52f7c7c05aaea3aeb4a59)

### Soal #15: Apex Engine

- Kategori: Binary Exploitation
- Poin: 67 pts (175 solves)
- Flag tertangkap dari instans live: 31 Okt 2026 ±00:00 WIB, menunggu submit Nugroho
- Endpoint: apex-engine-b541372b203c.chall.hiddeninvestigations.net:1337 (TLS, perintah platform ncat --ssl)
- Ringkasan masalah: Dua celah race condition pada layanan vault multithread: double-spend pipeline dan TOCTOU jalur file enclave.
- Langkah penyelesaian:
  1. Delapan transfer escrow 100 kredit dikirim dalam satu penulisan; semua worker settlement lolos cek terhadap saldo awal 100 karena pengurangan terjadi setelah tidur 25 ms. Escrow menjadi 800, cash underflow ke 2^64 - 700, melewati gerbang VIP 10.000.
  2. Di enclave, stream slot jinak dimulai; di dalam jendela tidur worker, path slot diedit menjadi /flag sehingga worker membaca /flag dengan clearance root.
  3. Eksploit yang terbukti lokal dijalankan ke instans live via TLS; tiga kali percobaan memberi flag identik.
- Tools: Python + pwntools (remote ssl=True, sni=host).
- Skrip: files/exploit_apex_engine.py (mendukung argumen --tls).
- Flag: hiCTF{t0ct0u_mult1thr34d_r4c3_c0nd1t10n_w1n_8d3f_6ded46a2bf}
- Folder bukti: file soal (kartu Apex Engine, oleh otolk1)

### Soal #16: Warden's Maze

- Kategori: Miscellaneous
- Poin: 84 pts (97 solves)
- Flag tertangkap dari instans live: 31 Okt 2026 ±00:00 WIB, menunggu submit Nugroho
- Endpoint: maze-breach-91569f7af21e.chall.hiddeninvestigations.net:1337 (TLS)
- Ringkasan masalah: Labirin dengan bug indeks visibilitas: offset tulis terpotong ke signed byte sehingga sel jauh bisa ditimpa.
- Langkah penyelesaian: Berdiri di (4,14) dan menulis 0x40 ke sel (12,9) mengubah dinding vault menjadi lantai; rute (12,9) ke (12,10) lalu prize (11,11); total 45 gerakan, persis seperti pembuktian lokal.
- Tools: Python (socket TLS).
- Skrip: files/exploit_wardens_maze.py (argumen --tls).
- Flag: hiCTF{maze-warden-vault-seed-2026_65f6cc212f}
- Folder bukti: file soal (kartu Warden's Maze)

### Soal #14 (pembaruan): C3 Vault

- Flag tertangkap dari instans live: 31 Okt 2026 ±00:02 WIB, menunggu submit Nugroho
- Endpoint: c3-vault-3c7b741448be.chall.hiddeninvestigations.net:1337 (TLS)
- Koreksi rumus: generation pada pesan derivasi DK berformat LITTLE-endian (bacaan awal disassembly menyangka big-endian); perintah live adalah "auth <hex>" (bukan RESPONSE seperti konsol perangkat). Dengan dua koreksi itu server menjawab VAULT OPENED.
- Skrip files/auth_c3vault.py sudah diperbaiki (TLS default, perintah auth, generation little-endian).
- Flag: hiCTF{efus3_cl34r_jt4g_0p3n_nv5_d46f9c7da9}

### Soal #17: Whispers in the Margin

- Kategori: Steganography
- Poin: 61 pts (246 solves)
- Waktu selesai: ±00:21 WIB (31 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Sepasang berkas, DOCX dan foto besar; detail penting bersembunyi di tempat yang tampak kosong.
- Langkah penyelesaian:
  1. Di ekor word/document.xml di dalam DOCX ada paragraf-paragraf berisi hanya run spasi dan tab. Dibaca sebagai biner (spasi = 0, tab = 1, tiap paragraf data = 1 byte, paragraf satu tab = pemisah) terdekode pesan 25 karakter: "haha! u got me, now what?"
  2. Pesan itu adalah kata sandi steghide untuk chall.jpg (persis, termasuk spasi dan tanda tanya). Ekstraksi steghide mengeluarkan flag. Angka-angka process counts di badan dokumen hanyalah umpan.
- Tools: parser XML DOCX, steghide.
- Flag: hiCTF{573g4n0gr4phy_!5_fun}
- Folder bukti: file soal whispers-in-the-margin.zip (SHA-256 kartu e7633b86df93f2ab68eb769c978130b07aadbeff9e51e685f090867a8739f55e)

### Soal #18: Roof of Africa

- Kategori: Steganography
- Poin: 57 pts (356 solves)
- Waktu selesai: ±00:28 WIB (31 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Foto Kilimanjaro; makna sebenarnya terkunci oleh "angka paling terkenal" gunung itu.
- Langkah penyelesaian: Payload steghide di roof_of_africa.jpg; kata sandinya 5895, ketinggian Kilimanjaro dalam meter, persis petunjuk kartunya. Ekstraksi mengeluarkan flag verbatim 38 byte. (Kata sandi kosong, "kilimanjaro", "5895m", dan "19341" semuanya ditolak.)
- Tools: steghide 0.5.1 (dipasang manual dari arsip Ubuntu karena tidak tersedia di repo VM).
- Flag: hiCTF{h4kun4_m4t4t4_m34ns_n0_w0rr13s}
- Folder bukti: file soal roof-of-africa.zip (SHA-256 kartu 137a5a7ec2a314e1027a7a279d33e45cd0a95256f63b35cca2d3fb233c3209ed)

### Soal #19: Zanzibar Whisper

- Kategori: Steganography
- Poin: 58 pts (334 solves)
- Waktu selesai: ±00:28 WIB (31 Okt 2026), menunggu submit Nugroho
- Ringkasan masalah: Foto pantai Zanzibar membawa pesan tersembunyi, "bisikan dari Pulau Rempah".
- Langkah penyelesaian: Payload steghide di zanzibar_whisper.jpg dengan kata sandi KOSONG. Ekstraksi mengeluarkan flag verbatim 28 byte. LSB piksel dan JSTEG dicek lebih dulu dan negatif.
- Tools: steghide.
- Flag: hiCTF{sp1c3_1sl4nd_s3cr3ts}
- Folder bukti: file soal zanzibar-whisper.zip (SHA-256 kartu 48aedaaf1a363d859ceaece068354394edd6a8ff1ea7245af3fde395aa749a67)

### Soal #20: Tasmota Vault

- Kategori: Hardware
- Poin: 64 pts (202 solves per kartu soal)
- Waktu selesai: ±02:09 WIB (31 Okt 2026), submit terkonfirmasi Nugroho ±09:41 WIB (progres menjadi 20/28)
- Ringkasan masalah: Dump flash ESP32 Tasmota (4 MB) + tangkapan MQTT kecil; recovery.txt memberi asumsi akuisisi. Soal sepenuhnya offline.
- Langkah penyelesaian: Hash arsip cocok persis dengan kartu (74e01ec6…). Tabel partisi @0x8000: partisi berlabel spiffs @0x3B0000 (320 KB) ternyata berisi LittleFS v2.0; parser tulisan tangan mengekstrak autoexec.be (memuat vault.bec), vault/device.id (16 byte), vault/keyring.bin (3 record epoch 6/7/8 berisi kunci 32 byte), dan vault/sealed.bin (nonce GCM 12B + ciphertext 46B + tag 16B). vault.bec (bytecode Berry terkompilasi) didisassembly dengan parser khusus dari format be_bytecode.py source Tasmota v15.6.0 yang disertakan: fungsi unseal() memvalidasi device + epoch, lalu unwrap_key = HMAC-SHA256(keyring[epoch], "tasmota-vault/unwrap/v2" || 0x00 || device_id || epoch LE32 || nonce 32B dari pesan MQTT). mqtt.pcap berisi dua PUBLISH VaultRotate: epoch 7 (retained, basi) dan epoch 8 (sah). Dekripsi AES-256-GCM sealed.bin dengan kunci epoch 8 menghasilkan flag dan TAG TERVERIFIKASI; tiga kombinasi salah (epoch 7 dua endianness, epoch 8 big-endian) semuanya gagal tag, persis jalur "VAULT REJECTED" agennya. Bukti kerja: ~/workspace/scratch/tasmota/ (device.id, keyring.bin, sealed.bin, vault.bec, disasm.py, littlefs.img).
- Tools: parser LittleFS tulisan tangan, disassembler bytecode Berry khusus, Python (hmac/hashlib/AES-GCM).
- Flag: hiCTF{b3rry_r0t4t10n_c94934d76344ab1151e03afd}
- Folder bukti: file soal tasmota_vault.zip (SHA-256 kartu 74e01ec638452b8674b70301a8270fe8607950737ca1c7a7dc3f5f62c03906ad)

### Soal #21: RESURRECTION (Operation Eclipse Void)

- Kategori: Forensics
- Poin: 82 pts (102 solves per kartu soal)
- Waktu selesai: segmen #1 dan #4 ditemukan pekerja ronde 2 pada 30 Okt malam (laporan FINDINGS_seg1_seg4.md); rakitan diserahkan ke Nugroho 31 Okt ±09:41 WIB, menunggu submit
- Ringkasan masalah: Image NTFS 1 GB; flag utuh dirakit dari 5 segmen yang masing-masing disembunyikan lewat kanal berbeda, berpola <ordinal-leet>--<deskripsi leet kanal>.
- Langkah penyelesaian: Geometri partisi dibangun ulang dari backup GPT (0x3FFFFE00) dan backup boot sector (0x3FDFFE00); parser NTFS tulisan tangan (~/resur/ntfs.py). Segmen #2: 34 panjang run runlist $DATA file Confidential_Q3 (MFT 111) terbaca sebagai ASCII. Segmen #3: selisih FILETIME low-dword entri jurnal transaksi 80-byte dikurangi konstanta 9984 (pembacaan ordinal yang diyakini benar "th1rd"). Segmen #5: celah antar cluster yatim $Bitmap mulai cluster 180000 sebagai ASCII, termasuk gap penutup yang membentuk "}". Segmen #1: blob overlay 42 byte di offset 0x400 kbdusx.dll (MFT 179) ternyata XOR kunci berulang 4 byte = TimeDateStamp PE itu sendiri dalam little-endian (80 85 ec 65), ditemukan lewat pemecahan statistik periode-4; plaintext-nya sendiri mengandung "hiCTF{" bersarang. Segmen #4: array nilai BinXML di event record Security_Audit.evtx (MFT 193): 33 nilai u32 yang masing-masing adalah code point satu karakter. Temuan lengkap: ~/resur/FINDINGS_seg1_seg4.md (termasuk varian cadangan bila rakitan utama ditolak: seg1 tanpa "hiCTF{" dalam, seg1 "!" menjadi "1", seg4 ternormalisasi "f0ur7h").
- Tools: parser NTFS tulisan tangan, analisis statistik XOR, parser BinXML.
- Flag (rakitan utama, semua segmen mentah): hiCTF{f!r5t--hiCTF{p3_0v3rl4y_71m3d473574mp_k3y_53c0nd--n7f5_runl157_cl5_g30m37ry_th1rd--7r4ns4c710n_n4n0_f1l371m3_f01_1rth--3v7x_b1n4ry_xml_70k3n5_f1f7h--b17m4p_orph4n_v01d_r35urr3c710n}
- Folder bukti: ~/resur/ (ntfs.py, FINDINGS_seg1_seg4.md)

- KOREKSI 31 Okt ±09:49 WIB (verifikasi ulang mata segar dari image, laporan ~/workspace/scratch/resur-verify/VERIFY_REPORT.md): segmen #1, #2, #4 terkonfirmasi byte-exact. Segmen #3 yang benar menurut byte mentah adalah "th1jd" (konstanta 9984 cocok di 32/33 posisi; posisi 3 menghasilkan selisih 10090 = 'j'; 'r' butuh konstanta berbeda khusus satu posisi). Segmen #5 yang benar adalah "51f7h" (gap pertama 180053-180000 = 53 = '5'). Rakitan utama di atas gagal karena membawa KEDUA bacaan ternormalisasi itu sekaligus; kombinasi serba-mentah belum pernah disubmit. Kandidat prioritas baru: hiCTF{f!r5t--hiCTF{p3_0v3rl4y_71m3d473574mp_k3y_53c0nd--n7f5_runl157_cl5_g30m37ry_th1jd--7r4ns4c710n_n4n0_f1l371m3_f01_1rth--3v7x_b1n4ry_xml_70k3n5_51f7h--b17m4p_orph4n_v01d_r35urr3c710n}
- RESOLUSI 31 Okt 10:05 WIB — SOLVED: kandidat serba-mentah di atas TETAP DITOLAK, begitu juga semua varian segmen-penuh (total 12+ pengiriman). Yang DITERIMA platform adalah rakitan PAYLOAD SAJA: setiap segmen terdiri dari tag ordinal (f!r5t--, 53c0nd--, th1jd--, f01_1rth--, 51f7h--) yang TIDAK ikut ke flag, disusul payload; kelima payload digabung di dalam satu hiCTF{...}. Catatan kartu "Each recovered phase reveals an ordinal identifier tag" ternyata harfiah: tag hanyalah penanda urutan fase. Seluruh karakter yang dipersengketakan (!/1, j/r, 5/f, f01_1rth) berada di dalam tag, sehingga tidak pernah memengaruhi flag. Flag final yang diterima: hiCTF{p3_0v3rl4y_71m3d473574mp_k3y_n7f5_runl157_cl5_g30m37ry_7r4ns4c710n_n4n0_f1l371m3_3v7x_b1n4ry_xml_70k3n5_b17m4p_orph4n_v01d_r35urr3c710n} (Forensics naik ke 4/6; progres tim 22/42 saat itu).

### Soal #22: Signal Lock

- Kategori: Reverse Engineering
- Poin: 89 pts (86 solves per kartu soal sensus 31 Okt 10:03 WIB)
- Waktu selesai: 31 Okt ±10:09 WIB oleh pekerja Muse; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Modul anti-cheat BEAM (Erlang) terkompilasi dan stripped; satu-satunya token yang diterima validator membuka flag.
- Langkah penyelesaian: signal_lock.beam di dalam zip ternyata terkompresi gzip; didekompresi lalu didisassembly dengan beam_disasm di Erlang/OTP 27. verify/1 menuntut binary 8 byte, membacanya sebagai integer 64-bit X, dan menerima bila transform(X) == 12675316763487955908; transform = ((X XOR 11936128517019425370) * 11400714819323198485) rotl 29 + 81985529216486895 (mod 2^64). Invers analitik (invers modular pengali ganjil, rotr 29, kurangi, XOR) memberi X = 0x67346d3330763372 = ASCII "g4m30v3r". unlock/1 meng-XOR literal cipher 48 byte dengan keystream blok SHA-256 counter SHA256(<<X:64, Counter:32>>). Diverifikasi LIVE di runtime: signal_lock:verify(<<"g4m30v3r">>) mengembalikan {ok, flag}; token salah memberi {error, locked}/{error, bad_token}.
- Tools: beam_disasm (OTP 27), Python (invers modular).
- Flag: hiCTF{b34m_byt3c0d3_n3v3r_l13s_s3ss10n_unl0ck3d}
- Folder bukti: ~/workspace/scratch/signal_lock/ (signal_lock_real.beam, work/disasm.txt, token.bin)

### Soal #23: GhostBus

- Kategori: Hardware
- Poin: 95 pts (75 solves per kartu soal sensus 31 Okt 10:03 WIB)
- Waktu selesai: 31 Okt ±10:10 WIB oleh pekerja Muse; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Hub smart-lock dan secure element; nonce untuk record tersegel "tidak pernah meninggalkan capture", jadi bacaan tersembunyi harus ditemukan di dalam tangkapan logika itu sendiri, lalu derivasi kunci dari firmware dibalik untuk membuka region tersegel.
- Langkah penyelesaian: capture.csv dan capture.vcd dibaca dengan decoder UART 115200 8N1 tulisan sendiri; semua frame berformat AA 55 | LEN | CMD | PAYLOAD | CRC16-Modbus LE dan CRC-nya valid. Frame CMD 0x10 dari secure element memuat nonce 16 byte: 9e3b7a44c1d05f82a6e1b4907cd3f5a2 (blok 48 byte berlabel "SEALED v1 rotated out" di flash 0x68000 adalah decoy rotasi lama). Dari disassembly firmware.elf (ARM, .text 2,9 KB): kunci = SHA-256(SALT24 || nonce)[0:16] dengan SALT24 = 04325a9c117f3da0c0ffee5aa55a00ff1234567890abcdef dari .rodata; kunci AES-128 = 664ecd32a0a21d02a90afb1efb6919b1. Region tersegel 48 byte di flash 0x6A000: setiap kata 4 byte dibalik urutan byte-nya, lalu AES-128-CBC dengan IV = nonce. Plaintext lolos validasi firmware sendiri persis: diawali hiCTF{, kurung tutup tepat sebelum padding PKCS#7 valid (7 x 0x07).
- Tools: decoder UART/VCD tulisan sendiri, disassembler ARM, Python (hashlib, AES-CBC).
- Flag: hiCTF{gh0st_bu5_uart_n0nc3_r3v3al_9c3f12}
- Folder bukti: ~/workspace/scratch/ghostbus/ (solve.py reproduksi penuh, disasm.txt, disasm2.txt)

### Soal #24: Dormant Signer

- Kategori: Blockchain
- Poin: 110 pts (57 solves per kartu soal sensus 31 Okt 10:03 WIB)
- Waktu selesai: 31 Okt ±10:14 WIB oleh pekerja Muse; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Desk kustodian memakai signer off-chain otomatis; kunci privat tidak pernah dicatat dan artefak publik yang sama dipakai untuk semua akun. Plaintext escrow milik production authorizer yang sudah dihancurkan harus dipulihkan hanya dari artefak publik.
- Langkah penyelesaian: Zip berisi mini-chain yang bisa direplay (genesis anvil + 380 tx legacy terpresign, chainId 7411) plus sumber Vyper Authorizer/VoucherDesk/Notary. Semua 470 tanda tangan ECDSA dikumpulkan (level transaksi + (v,r,s) di calldata authorize/redeem/stamp) dan di-ecrecover. Kelemahannya: perangkat lunak signer ceroboh soal nonce, sering menandatangani dengan nonce RFC 6979 yang dinegasi (k -> n-k, r sama) dan artefak dipakai lintas akun. Enam nilai r berulang; dua pasangan bisa dieksploitasi: signer 0x740a... (reuse k klasik) dan signer produksi 0x4e4b... (pasangan k dan -k, terbukti dari k1+k2 = n, dipulihkan lewat varian s1+s2). Kedua kunci privat pulih dan terverifikasi alamat. Escrow menurut Authorizer.vy = blob XOR escrow_mask(sig liveness) XOR running_digest: blob dari argumen destroy() tx #352; running_digest direplay dengan semantik revert persis (tx #118 dan #268 revert, tidak ikut fold); sig liveness dibuat dengan kunci pulihan (RFC 6979 atas keccak256 "custody-desk/authorizer/liveness/v1"). Hasil XOR: "layout_undead_borrowed_betrays" + 2 byte nol. Validasi silang: pipeline yang sama membuka escrow authorizer kedua menjadi ASCII bersih ("packed_frozen_shared_exposes", rahasia decoy-nya), membuktikan seluruh konstruksi.
- Tools: Python (RLP decode, ecrecover, keccak), replay state tulisan sendiri.
- Flag: hiCTF{layout_undead_borrowed_betrays}
- Folder bukti: ~/workspace/scratch/dormant-signer/ (parse.py, analyze.py, solve.py, simulate.py, recovered_keys.json)

### Soal #25: The Inner Engine

- Kategori: Reverse Engineering
- Poin: 94 pts (77 solves per kartu soal sensus 31 Okt 10:03 WIB)
- Waktu selesai: 31 Okt ±10:18 WIB oleh pekerja Muse; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Checker lisensi node-locked di ELF Linux x86-64; kunci aktivasi yang valid membuka record. Binary adalah program Nim stripped dengan pesan gagal terobfuskasi XOR 0x3C.
- Langkah penyelesaian: Di dalam main ternyata ada VM bytecode ("inner engine"): instruksi 5 byte (opcode + operand u32) di VA 0x405E00, tabel handler 256 entri, stack machine u32 dengan memori data kecil, opcode muat byte kunci, aritmetika/logika/rotate, lompatan kondisional, satu opcode anti-debug (membaca TracerPid; di bawah ptrace langsung halt tersegel), dan opcode SHA-256 native. Tracer ptrace Python ctypes khusus (setelah memperbaiki pemotongan argumen 64-bit dan memaksa nilai kembali handler anti-debug) mencatat alur instruksi asli; program 234 instruksi didisassembly statis penuh. VM menyimpan key[0..3] ke memori lalu memberlakukan ±23 kendala byte (jumlah berpasangan mod 256, produk mod 256, tripel XOR, 4 persamaan besar) yang diselesaikan z3: tepat 1024 solusi. Cek native-nya: d1 = SHA256(K), keystream = SHA256(d1||BE32(0)) || SHA256(d1||BE32(1)), P = blob 40 byte di .rodata XOR keystream, valid bila SHA256(P) sama dengan digest tersimpan. Menguji 1024 kandidat terhadap konstruksi ini menyisakan satu kunci: L1c#N0d3_7mQ9f3a2b8c6d1e. Diverifikasi end-to-end: binary dijalankan normal dengan kunci itu mencetak flag; kunci salah mencetak "license sealed".
- Tools: tracer ptrace ctypes tulisan sendiri, disassembler VM statis, z3, Python hashlib.
- Flag: hiCTF{vm_byt3c0d3_1nn3r_3ng1n3_unw0und!}
- Folder bukti: ~/workspace/scratch/inner-engine/ (full.asm, trace3.py, solve.py, solutions.pkl)

### Soal #26: Hollow Reserve

- Kategori: Blockchain
- Poin: dinamis ±190-235 pts selama lomba (24 solves per kartu soal sensus 31 Okt 10:03 WIB)
- Waktu selesai: 31 Okt ±10:33 WIB oleh pekerja Muse melawan instance live; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Bank sentral fiktif menyimpan seluruh cadangannya (10.000.000 RSV) di vault on-chain; fasilitas kredit resmi boleh meminjam dengan agunan yang dihargai memakai pasar on-chain bank itu sendiri. Kurangkan cadangan sampai vault memegang kurang dari 10% saldo awal.
- Langkah penyelesaian: Instance menyajikan POST / sebagai JSON-RPC (chainId 31337, anvil automine), GET /info (kunci pemain terdanai: 10 ETH, 100 COL, 1.000 RSV), GET /flag (terkunci di Setup.isSolved()). Kontrak dari getter Setup: Vault (cadangan 10 juta RSV, hanya meminjamkan ke Facility), CreditFacility (borrow/repay, LTV 75%, membaca feed), PriceFeed yang getPrice()-nya hanya STATICCALL source().spotPrice() tanpa pengaman, dan source pool-nya adalah AMM TIPIS (1.000 COL / 100.000 RSV, fee 0,3%). GuardedPool besar berpagar deviasi ternyata decoy (tidak dibaca jalur pinjam). FlashLender ERC-3156 memegang 5.000.000 RSV dengan flashFee = 0. Serangan satu transaksi via kontrak helper: flash-borrow 5 juta RSV, swap seluruhnya ke COL di pool tipis sampai harga spot melonjak 100 -> ±260.000 RSV/COL, jadikan semua COL agunan di Facility, pinjam 9.600.000 RSV langsung dari Vault, kembalikan flash loan. Hasil: cadangan vault 10.000.000 -> 400.000 RSV (4%), isSolved() = 1, GET /flag mengeluarkan flag.
- Tools: web3.py, kontrak helper Solidity (Attack.sol), skrip deploy+serang run_attack.py.
- Flag: hiCTF{th1n_p00l_was_the_r3al_0racle}
- Folder bukti: ~/workspace/scratch/hollow/ (Attack.sol, run_attack.py, probe*.py, dump bytecode kontrak)

### Soal #27: Whisper

- Kategori: Web
- Poin: 244 pts saat soal ditangkap (16 solves; poin dinamis)
- Waktu selesai: 31 Okt ±10:39 WIB oleh pekerja Muse melawan instance live; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Gateway API partner: klien menukar kredensial dengan token, klien istimewa menandatangani request ke endpoint internal. Hanya report-signer yang boleh mengekspor laporan, dan secret-nya tidak diberikan.
- Langkah penyelesaian: /clients mendaftar public-web (read) dan report-signer (read + report:export); /docs menyebut POST /internal/export diautentikasi tanda tangan HMAC-SHA256 (X-Client-Id, X-Timestamp, X-Signature), dan pesan error endpoint membocorkan format string-to-sign persis: METHOD \n PATH \n CANONICAL_QUERY \n HEX_SHA256(body) \n X-Timestamp. (1) Injeksi NoSQL di POST /api/token: body JSON diteruskan mentah ke query Mongo; {"client_id":"report-signer","client_secret":{"$ne":null}} mengembalikan token full-scope (token bearer saja tidak cukup untuk export, tetap ditagih X-Signature). (2) Ekstraksi buta: oracle boolean client_secret {"$regex":"^<prefix><char>"} di endpoint yang sama (token terbit = karakter benar) membocorkan secret report-signer 32 karakter: a4a16db86902198ae555e57c4555b150. (3) Tanda tangan dipalsukan dengan secret asli; proteksi replay mengharuskan nilai header X-Nonce ditambahkan sebagai baris terakhir string-to-sign — request final (timestamp segar + nonce UUID, body {}) dijawab 200 berisi flag oleh gateway sendiri.
- Tools: curl/python requests, skrip ekstraksi regex buta, HMAC-SHA256.
- Flag: hiCTF{bl1nd_regex_3xfil_then_f0rge_the_cust0m_s1gner_bf66cc261e}
- Folder bukti: ~/workspace/scratch/whisper/ (TRANSCRIPT.md urutan request pemenang, extract_secret.py, export.py, flag_response.txt)

### Soal #28: LetMeOut

- Kategori: Pwn
- Poin: ±158 pts saat ditangkap (dinamis)
- Waktu selesai: 31 Okt ±11:08 WIB oleh pekerja Muse melawan instance live; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Kalkulator SSTI Jinja dalam sandbox; setiap koneksi mengundi filter huruf (payload maks 170 karakter, <=3 huruf berbeda dari himpunan yang diundi); direktori /flags berisi 4 berkas, hanya satu yang asli.
- Langkah penyelesaian: Celahnya ketidakcocokan normalisasi NFKC di jail.py: fallback getitem menormalisasi Tag menjadi str untuk getattr, tetapi is_safe_attribute memeriksa argumen MENTAH. Tag yang diawali underscore full-width U+FF3F tidak diawali "_" pada bentuk mentahnya sehingga lolos pemeriksaan, sementara getattr memakai nama ternormalisasi: rantai (9|RICH) -> __str__ -> __globals__ -> G["random"] -> _os -> listdir() menampilkan 4 nama berkas; os.open + os.read membaca berkas asli 11052df4 (tiga lainnya umpan). Trik anggaran karakter: escape oktal menulis kunci/atribut tanpa menghabiskan huruf filter (\162 = r dst.), dan karakter Tag sesudah char pertama boleh oktal juga; solver reconnect sampai undian filter membuat tiap baris muat (7/10 undian muat untuk baris listdir).
- Tools: python pwntools, solver.py/gen.py/remote_solve.py.
- Flag: hiCTF{__^_^__n0rm4l1z3_b3f0r3_y0u_tru5t__>_<___a1e9862416}
- Folder bukti: ~/workspace/scratch/letmeout/ (solver.py, gen.py, remote_solve.py)

### Soal #29: TESSERA

- Kategori: Forensics
- Poin: 185 pts saat ditangkap (dinamis); format khusus hiCTF{part1part2part3}
- Waktu selesai: 31 Okt ±11:27 WIB oleh pekerja Muse dari image resmi; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Image disk Windows ±993 MB dari sebuah insiden; flag terbagi tiga bagian yang disembunyikan lewat tiga mekanisme Windows berbeda. Petunjuk kartu berbentuk teka-teki: "local interstice/unmapped portico" (port ALPC), "ephemeral colloquy where disparate entities conjoined" (sesi/trace), "redirects reality itself" (reparse point).
- Langkah penyelesaian: (0) Batu Rosetta: log PowerShell Operational/Admin EVTX dibersihkan di tengah sesi (stub 69.632 B), tetapi slack chunk-nya menyimpan blok skrip skenario penulis sendiri (residu 4104) yang mendokumentasikan ketiga tahap. (1) Berkas AuditDiagTrace.etl (LogFiles/WMI) memuat satu event pesan tertanam dari provider Microsoft-Windows-Diagnostics-LpcEndpoint berisi PORT=\RPC Control\AepLocalEndpoint_Q8 dan HEX 65 byte; XOR byte tunggal 0x5A membukanya menjadi bagian 1 + penunjuk layanan tahap 2 (DiagTrackSvcAux), setelah membuang tag ordinal "1ist-----". (2) Vault DPAPI LocalMachine cpinforev.dat (662 B, header blob DPAPI asli) di path systemprofile yang persis tertulis di skrip; JSON tahap 2 memberi bagian 2 setelah membuang tag "--s32nd--" (tombol layanan dummy dihapus saat cleanup, namanya masih muncul di hive SYSTEM mentah). (3) Rekor MFT 101281 direktori ProgramData\Microsoft\Diagnosis\ETLLocalStore membawa atribut $REPARSE_POINT residen tag 0xA0000028 berisi payload 48 byte; dekripsi AES-256-CBC (kunci dari skrip skenario, IV 0x10..0x1F) memberi padding PKCS7 valid dan bagian 3 setelah membuang tag "iiiiiiiiiii-3hird---". Pola tag ordinal penyesat sama seperti soal RESURRECTION. Dikecualikan selama analisis: telemetri OneDrive, arsip WER, Recycle Bin, indeks pencarian, hive registry tanpa tulisan in-window, pagefile/hiberfil yang tidak terkumpul.
- Tools: python (parsing ETL/EVTX slack, MFT, kriptografi AES), analisis timeline berkas sesi.
- Flag: hiCTF{alpc_p0rt_c0lloquy_s35s10n_z3r0_syst3npr0f1l3_r3p4r53_buff3r_vort3x}
- Folder bukti: ~/workspace/scratch/tessera/ (AuditDiagTrace.etl, ps_slack.txt berisi skrip skenario hasil carve, stage3_payload.bin, pf_full.txt, mft_paths.txt)

### Soal #30: Tyco

- Kategori: Cryptography
- Poin: 102 pts saat ditangkap (dinamis); teks kartu hanya "🫠🤖"
- Waktu selesai: 31 Okt ±11:39 WIB oleh pekerja Muse dari berkas resmi; submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: RSA related-message: N = p*q (dua prima 256-bit), eksponen prima e = 56263, selisih acak d DIKETAHUI; diterbitkan t1 = r^e mod N dan t2 = (r+d)^e mod N dengan r adalah flag sebagai integer.
- Langkah penyelesaian: Serangan Franklin-Reiter: gcd(x^e - t1, (x+d)^e - t2) atas Z/N menghasilkan faktor linear x - r. Rintangan perkakas: gcd polinomial FLINT menolak modulus komposit, Euclid Python/FLINT terlalu lambat di derajat 56.263, dan jalan pintas CRT atas prima kecil tidak sah secara matematis di sini. Yang bekerja: program C tulisan tangan memakai GMP (algoritma Euclid atas Z/N dengan reduksi malas, satu lintasan mod per langkah bagi), divalidasi ujung-ke-ujung pada instance uji buatan sendiri sebelum run asli ±4.170 detik (algoritmanya inheren O(e^2)). GCD mencapai derajat 1 setelah 56.262 langkah; akar r terverifikasi memenuhi kedua persamaan terbit sebelum flag dibaca.
- Tools: C + GMP (tyco.c), python untuk validasi.
- Flag: hiCTF{your_ai_slop_bores_me}
- Folder bukti: ~/workspace/scratch/tyco/ (tyco.c, biner tyco, euclid.py, crt_solve.py, log)

### Soal #31: Entitlement

- Kategori: Reverse Engineering
- Poin: 165 pts saat ditangkap (dinamis)
- Waktu selesai: 31 Okt ±13:35 WIB oleh pekerja Muse (grind binary selesai; flag dicetak ke file output); submit via browser Muse (menunggu konfirmasi platform)
- Ringkasan masalah: Checker lisensi dua tahap (serial + activation code) yang masing-masing harus lolos terhadap binary asli, lalu grind KDF internal yang disengaja sebelum flag dicetak.
- Langkah penyelesaian:
  1. Tahap 1: VM kustom S^13 di GF(2^64) diinvers secara analitik; serial hasil inversi lolos cek tahap 1 binary asli.
  2. Tahap 2: perkalian skalar kurva eliptik — kurva ternyata anomalous, #E(F_p) = p persis, sehingga dipecahkan dengan serangan Smart; activation code hasil serangan lolos cek tahap 2 binary asli. Koreksi selama analisis: "paradoks T off-curve" yang sempat dilaporkan ternyata salah salin satu nibble oleh pekerja sendiri, bukan sifat kurva.
  3. Setelah kedua cek lolos, binary menjalankan grind KDF: 50.000 blok x 10.000 iterasi fungsi mixing bignum, lalu mencetak flag ke output.
  4. Perjalanan grind: proses pertama jalan 31 Okt 11:31 WIB, mati ±12:35 WIB (keluar dengan kode 137; penyebab pasti tidak terkonfirmasi). Relaunch pertama pekerja (±12:35 WIB) ikut mati dalam hitungan menit. Relaunch kedua (±12:38 WIB) memakai setsid terlepas dari sesi, terverifikasi berjalan penuh (14 detik CPU per 20 detik wall) dan bertahan sampai selesai ±13:35 WIB — lebih cepat dari estimasi awal ±15:50 WIB (laju CPU awal ternyata pesimistis).
  5. Flag diverifikasi langsung dari file output binary (baris "Correct! The flag is ...") sebelum dikirim ke browser untuk submit; proses binary sudah berhenti sendiri karena selesai normal.
- Tools: harness binary kustom, analisis kripto (inversi GF(2^64), serangan Smart pada kurva anomalous).
- Flag: hiCTF{>o<_C0ngrats_Her3_1s_Y0ur_HI4_Subscript1on_^_^}
- Folder bukti: ~/workspace/scratch/entitlement/ (binary Entitlement, native_out2.txt)
- Catatan hitungan: entri ini = solve platform ke-32 (31 solved sebelum Entitlement). Dua solve platform yang belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #32: Log Jam

- Kategori: Cryptography
- Poin: 65 pts saat ditangkap (dinamis); 199 solves sebelum submit, CakraWatch solver ke-200
- Waktu selesai: 31 Okt 16:09-16:16 WIB (laporan pekerja selesai 16:10 WIB; platform menjawab "Flag correct!" dan "Challenge solved!")
- Ringkasan masalah: params.json (satu-satunya berkas di zip) berisi parameter DH grup safe-prime 71-bit (p, g, h), nonce 12 byte, ciphertext 50 byte, tag AES-GCM 16 byte; kunci AES digerbangi eksponen privat x = log_g(h), tanpa ephemeral yang diberikan.
- Langkah penyelesaian:
  1. Skema teridentifikasi: p = 0x7958085483c2736f23, p-1 = 2q (q prima ~2^69,9), g dan h di subgrup orde q; kunci = SHA-256(string desimal x) sebagai AES-256, dekripsi AES-256-GCM dengan nonce yang diberikan.
  2. Pencarian murah tanpa DLP gagal semua (turunan SHA-256 dari p/g/h, h^nonce, dll., semua gagal verifikasi tag GCM).
  3. Solver Pollard-kangaroo C tulisan tangan (kangaroo2.c, checkpoint/resume tiap 2^27 lompatan, ~8-12 juta lompatan/detik), divalidasi memulihkan kunci tanam persis di skala 2^36, 2^44, dan 2^56. Pencarian terbatas mengecualikan x di [0, 2^64) dan di (q-2^48, q], jadi x berukuran nyaris penuh.
  4. Run final rentang penuh [0, 2^70): tame walk 68,72 miliar lompatan (523.586 distinguished point), lalu wild walk bertabrakan setelah ~1,1x10^10 lompatan. Kunci pulih x = 471774559909450042061 (0x19932feeab3d034acd), terverifikasi pow(g, x, p) == h.
  5. Verifikasi independen oleh Muse dari params.json saja (proses Python segar): SHA-256(str(x)) membuka ciphertext dan tag GCM lolos, plaintext-nya flag.
- Tools: kangaroo2.c (C), Python (hashlib, AES-GCM).
- Flag: hiCTF{W0W1_y0u_4ctu4a11y_s01v3d_1t_ju$t_L1k3_tH4t}
- Folder bukti: ~/workspace/scratch/logjam/ (kangaroo2.c, ck_b70.bin, run_b70.log, trykeys2.py, SOLUTION_NOTES.md)
- Catatan hitungan: entri ini = solve platform ke-33 (32 solved sebelum Log Jam). Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #33: Snapfile

- Kategori: Reverse Engineering (soal gelombang baru, platform bertambah 44 -> 54 soal pada 1 Nov pagi)
- Poin: 61 pts; 257 solves sebelum submit, CakraWatch solver ke-258
- Waktu selesai: 1 Nov ±08:01 WIB oleh pekerja Muse; platform menjawab "Flag correct!" dan "Challenge solved!" ±08:02-08:03 WIB (submit via browser Muse). Total tim sesudahnya: 1.929 pts, 34/54 solved, peringkat 229/1.065; kategori Reverse Engineering kembali sapu bersih 9/9.
- Ringkasan masalah: `snaptool` (binary Crystal, stripped) memvalidasi berkas berformat kustom "snapfile"; setiap kegagalan mencetak `invalid snapfile`.
- Langkah penyelesaian:
  1. Analisis statik `__crystal_main` menemukan format persis 104 byte: magic "SNAP", u16 BE versi 2, byte 0x03 0x00, marker 8 byte "SNAP-LNG", lalu enam field u32 big-endian 40, 16, 56, 32, 88, 8 (tabel chunk: tiga chunk di offset 40/56/88, panjang 16/32/8, berakhir di 96).
  2. Delapan byte terakhir = checksum: mixing 64-bit gaya splitmix (seed 0x1357bdf02468ace1, pengali 0xd1b54a32d192ed03, xor-shift 29) atas byte 8..96; u64 BE tersimpan harus memenuhi stored ^ mix == 0x58.
  3. Berkas valid dibuat dengan craft.py (payload bebas, 'A'x56 lolos); binary mencetak flag dan exit 0 pada dua run berurutan. Kontrol negatif: berkas identik versi little-endian ditolak (invalid snapfile, rc=1).
- Tools: analisis statik binary Crystal, Python (craft.py).
- Flag: hiCTF{r3c0nstruct3d_th3_sn4pf1l3_by73_by_by73}
- Folder bukti: ~/workspace/scratch/new-snapfile/ (craft.py, craft_be.bin, craft_le.bin)
- Catatan hitungan: entri ini = solve platform ke-34 (33 solved sebelum Snapfile). Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #34: Pruned Ledger

- Kategori: Blockchain (soal gelombang baru, platform bertambah 44 -> 54 soal pada 1 Nov pagi)
- Poin: 64 pts; 203 solves sebelum submit, CakraWatch solver ke-204; kategori Blockchain naik menjadi 3/4
- Waktu selesai: 1 Nov ±08:21-08:22 WIB (platform menjawab "Flag correct!" dan "Challenge solved!" pada kandidat kedua; submit via browser Muse). Total tim sesudahnya: 1.993 pts, 35/54 solved, peringkat 216/1.066.
- Ringkasan masalah: Sistem ledger penyelesaian dinonaktifkan; operator membersihkan field record dari storage kontrak dengan prune(). Deskripsi soal: "Reconstruct the pruned case file for case 0xca5ef11e." Tidak ada instance; seluruh chain archive ada di handout (400 tx, 26 deployment).
- Langkah penyelesaian:
  1. Identifikasi kasus: tag bytes4 0xca5ef11e hanya muncul di 4 deployment Relay; mengikuti pointer `next`, semua trail bermuara ke satu Ledger 0x68fb8760b1353f7349c431442d6e97ab61685ee7 (caseReference = pruned_storage). Arus dana 4,75 ETH bertag sama menguatkan.
  2. Record saat seal: configure tx#123 = "calldata" (configure sesudah seal tidak relevan; prune tx#325 mengosongkannya dari storage, tetapi calldata history tidak bisa dihapus).
  3. Entri arsip: seal tx#136, maskedEntry dibuka dengan XOR keccak256(keccak256("hictf.pruned-ledger.seal") || ledger || record) = "never"; slot arsip = uint256(keccak256("hictf.pruned-ledger.archive" || record)) - 1.
  4. Note: maskedNote dibuka unik untuk caller investigator 0x1233...060f = "forgets" (terkonfirmasi dari revert unlock tx#291 sesuai Ledger.sol).
  5. Flag = caseReference + kalimat berkas kasus: pruned_storage + calldata never forgets. Kandidat pertama tanpa caseReference (hiCTF{calldata_never_forgets}) DITOLAK platform; kandidat kedua diterima.
- Tools: dekoder RLP + simulator EVM Python (ronde 1), recover_case.py (ronde 2), kontrak Ledger.sol dari handout.
- Flag: hiCTF{pruned_storage_calldata_never_forgets}
- Folder bukti: ~/workspace/scratch/new-pruned/ dan ~/workspace/scratch/new-pruned2/ (FINDINGS.md, recover_case.py)
- Catatan hitungan: entri ini = solve platform ke-35 (34 solved sebelum Pruned Ledger). Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #35: Quiet Upgrade

- Kategori: Blockchain (soal gelombang baru) — kategori Blockchain SAPU BERSIH 4/4 dengan solve ini
- Poin: 67 pts
- Waktu selesai: eksploitasi instance live selesai 1 Nov ±08:23 WIB oleh pekerja Muse (Setup.isSolved() = true, flag diambil dari GET /flag instance); platform menjawab "Flag correct!" dan "Challenge solved!" ±08:23-08:24 WIB (submit via browser Muse). Total tim sesudahnya: 2.060 pts, 36/54 solved, peringkat 204/1.066.
- Ringkasan masalah: Proxy upgradeable SettlementProxy menjalankan SettlementV1 via delegatecall. Keduanya memakai slot storage berurutan polos (tanpa hashing EIP-1967) dengan makna bertabrakan: slot 0 = admin (proxy) vs branchOperator (V1), slot 1 = implementation (proxy) vs branchRegistrar (V1).
- Langkah penyelesaian:
  1. registerBranch(operator) di V1 tidak tergating (registrar tidak pernah dihubungkan) dan menulis slot 0, sehingga siapa pun bisa menimpa admin proxy lewat fallback delegatecall.
  2. Sebagai admin, upgradeTo() ke implementasi jahat Attack yang proxiableSlot()-nya mengembalikan nilai terikat proxy keccak256(abi.encodePacked(keccak256("quiet-upgrade.proxiable"), proxy)).
  3. drain() dipanggil lewat proxy: delegatecall membuat address(this) == proxy, sehingga Treasury.release() melihat msg.sender sebagai sistem settlement resminya dan melepas seluruh cadangan 5.000.000 STL ke pemain.
  4. Bukti on-chain: treasury 5.000.000 -> 0 STL, pemain 5.000.200 STL, Setup.isSolved() = true, lalu GET /flag instance mengembalikan flag.
  5. Catatan ketelitian: kunci pemain dari brief sempat kurang satu digit heks (63 karakter); pekerja mengambil nilai resmi dari GET /info instance dan memverifikasi kunci menurunkan alamat pemain sebelum mengirim transaksi apa pun.
- Tools: solc 0.8.24 statis, driver Python tanpa dependensi (Keccak pycryptodome, signing secp256k1 RFC6979 tulisan tangan, RLP, ABI) — fast_exploit.py; versi web3 umum di exploit.py + Attack.sol.
- Flag: hiCTF{sl0t0_c0llision_then_a_qu1et_upgrade_b37c872cce}
- Folder bukti: ~/workspace/scratch/new-quiet/ (fast_exploit.py, exploit.py, contracts/Attack.sol)
- Catatan hitungan: entri ini = solve platform ke-36 (35 solved sebelum Quiet Upgrade). Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #36: Dead Air

- Kategori: Miscellaneous (soal gelombang baru)
- Poin: 63 pts; 215 solves sebelum submit, CakraWatch solver ke-216
- Waktu selesai: rantai dekode tuntas 1 Nov ±08:23 WIB (ronde 2); flag diterima platform ±08:32-08:33 WIB (submit via browser Muse). Total tim sesudahnya: 2.123 pts, 37/54 solved, peringkat 193/1.066.
- Ringkasan masalah: Rekaman radio "dead air" (WAV stereo 47,2 dtk) + arsip 7z terkunci berisi codepad substitusi glif dan halaman logbook berglif.
- Langkah penyelesaian (rantai 4 lapis):
  1. Spektrogram kanal R: teks KSTR-7742-VXQ di pita 5-13,5 kHz = kata sandi 7z.
  2. Glif logbook diterjemahkan dengan codepad (37 simbol A-Z 0-9 _) secara deterministik (dua matcher IoU independen + verifikasi visual): ciphertext 59R_6L3Q9BYGBQ3QZAYZMP3QIZ92.
  3. Kanal L: nada beacon 700 Hz ternyata Morse pada resolusi envelope 20 ms (ronde 1 keliru menyimpulkan kontinu karena resolusi terlalu kasar): "KESTREL STATION OFF AIR KEY IS NIGHTJAR LISTEN WITH YOUR EYES" — kunci = NIGHTJAR.
  4. Vigenere atas alfabet 37 simbol codepad, kunci NIGHTJAR, arah C-K: plaintext S1L3NC3_W4S_TH3_M3SS4G3_6R3V (terverifikasi roundtrip byte per byte).
  5. KUNCI PENYELESAIAN: alfabet sandi hanya huruf besar sehingga buta huruf kecil; flag terdaftar di platform dalam huruf kecil. Empat kandidat ditolak lebih dulu (ciphertext mentah, plaintext huruf besar dengan/ekor, kalimat de-leet huruf besar) sebelum bentuk huruf kecil penuh diterima.
- Tools: analisis spektrogram + envelope Hilbert (Python), py7zr, matcher glif IoU kustom; catatan ROUND2_NOTES.md dan ROUND3_NOTES.md.
- Flag: hiCTF{s1l3nc3_w4s_th3_m3ss4g3_6r3v}
- Folder bukti: ~/workspace/scratch/new-deadair/ (spec_full.png, logbook/, contact_sheet.png, ROUND2_NOTES.md, ROUND3_NOTES.md)
- Catatan hitungan: entri ini = solve platform ke-37 (36 solved sebelum Dead Air). Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #37: Titan Relay

- Kategori: Binary Exploitation (soal gelombang baru)
- Poin: 68 pts; 164 solves sebelum submit, CakraWatch solver ke-165; kategori Binary Exploitation naik menjadi 3/4
- Waktu selesai: flag dibaca dari layanan live 1 Nov ±09:10 WIB oleh pekerja Muse; platform menjawab "Flag correct!" dan "Challenge solved!" ±09:10-09:11 WIB (submit via browser Muse). Total tim sesudahnya: 2.190 pts, peringkat 188/1.068. CATATAN PAPAN: platform kini menampilkan 62 tantangan total (38/62), bukan 54 — gelombang soal tambahan tampaknya terbit lagi; enumerasi ulang dijalankan.
- Ringkasan masalah: Layanan "route consensus VM" (binary Windows, Rust) menerima kapsul bytecode VM kustom lewat koneksi TCP+SSL; kapsul yang lolos seluruh gerbang membuat layanan membuka dan membacakan sebuah path berkas.
- Langkah penyelesaian:
  1. ISA VM dipetakan penuh (8 register u64, tabel seed, 14 opcode); kapsul didekode XOR keystream (0x71 + 0x3d*i) & 0xff.
  2. Program exploit dibangun lolos berurutan: gerbang FINALIZE (state+8 == seed, state+0x10 == alamat validator, reg6 terpasang), lima cek validator turunan seed, cek byte payload 12 byte, dan validasi UTF-8 — setiap tahap dibuktikan lewat diskriminasi binary-patch pada oracle Wine lokal.
  3. Fase akhir: payload terdekripsi 11 byte harus berupa path relatif yang dibuka layanan dan dibacakan dalam potongan 0x2000 byte. CWD layanan /app (Dockerfile handout), flag di /flag.txt, jadi kandidat utama ../flag.txt tepat 11 byte.
  4. Instance pertama kedaluwarsa sebelum tembakan dinilai; pada instance baru, ../flag.txt diterima pada tembakan pertama ("[+] capsule accepted") dan layanan mencetak flag; kandidat cadangan ..\flag.txt mengembalikan flag yang sama sebagai konfirmasi.
- Tools: analisis statik + oracle Wine, exploit.py (pembangun kapsul), fire_live.py (penembak SSL).
- Flag: hiCTF{ru5t_vm_g4te_and_h4ndler_ch41n_70e4d8d89f}
- Folder bukti: ~/workspace/scratch/new-titan/ (STATE.md, exploit.py, fire_live.py)
- Catatan hitungan: entri ini = solve platform ke-38. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #38: Ghost in RAM

- Kategori: Forensics (soal gelombang baru)
- Poin: 67 pts; platform menjawab "Flag correct!" dan "Challenge solved!" ±09:24 WIB 1 Nov (submit via browser Muse). Total tim sesudahnya: 2.256 pts, 39 soal selesai, peringkat 187/1.069. Catatan papan: total tantangan kini tampil 61 (sebelumnya 62) — panitia masih aktif mengubah papan; penyebut berfluktuasi.
- Ringkasan masalah: Dump RAM Linux 2,16 GB (mem.elf); sebuah proses "ghost" (daemon syncfsd dan beberapa binary umpan) menyimpan struktur region kustom "GVM1" berisi bytecode VM dan flag terenkripsi; dump juga memuat dua flag plaintext umpan.
- Langkah penyelesaian:
  1. Ronde 1 mengekstrak dump (strings 152 MB) dan meng-carve binary ghostbin.elf; ronde 2 melanjutkan dari artefak itu dan menemukan struktur region GVM1 pada offset 35471416 di mem.elf: header memuat magic GVM1, seed 0x6f2e1a53, code_off 128, code_len 91, flag_off 448, flag_len 30, slot string berisi umpan hiCTF{stop_here}, dan 30 byte cipher pada flag_off.
  2. Code stream VM menyematkan konstanta LCG 0x41c64e6d dan 0x3039 (1103515245 dan 12345, LCG ANSI klasik). Keystream: state = state*1103515245 + 12345 (mod 2^32) mulai dari seed, ambil (state >> 24) & 0xff tiap langkah; XOR dengan 30 byte cipher menghasilkan flag. Diverifikasi dua kali dengan skrip independen langsung dari mem.elf; semua konvensi keystream lain menghasilkan non-printable, jadi identifikasi tegas.
  3. Jalan buntu ditutup dengan bukti: kedua flag plaintext di dump adalah umpan (hiCTF{stop_here} di slot GVM dan hiCTF{d3bug_m3} di config decoy2); blok base64 "SESSION KEY" terbukti noise daemon murni (rantai state LCG 53.077 baris log mereproduksinya 900/900 karakter); ghostbin/syncfsd adalah daemon noise glibc statik; berkas flag_obf.bin/mblob.bin ronde 1 ternyata potongan .symtab binary lain.
- Tools: ekstraksi strings + carving ELF, analisis struktur region GVM1, Python (dekripsi LCG-XOR).
- Flag: hiCTF{gh0st_pr0c3ss_rng_pwn3d}
- Folder bukti: ~/workspace/scratch/new-ghostram/ (FLAG.txt, region_full.bin, region_candidate.bin, loglines.json)
- Catatan hitungan: entri ini = solve platform ke-39. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #39: File Viewer

- Kategori: Web (soal gelombang baru)
- Poin: nilai dinamis (181 pts saat enumerasi 09:15 adalah Collector; File Viewer 149 pts saat enumerasi; poin gabungan dua submit berurutan +229, lihat catatan di entri Collector). Platform menjawab "Flag correct!" / "Challenge solved!" ±09:29-09:30 WIB 1 Nov (submit via browser Muse, bersamaan dengan entri berikutnya).
- Ringkasan masalah: Aplikasi Flask "Zip Checker": POST /analyze menerima zip, mengekstraknya ke uploads/<hash>/, lalu menyajikan isinya di /<hash>/<path>; berkas hasil ekstrak hanya hidup beberapa detik sebelum dihapus janitor, jadi seluruh rantai harus berjalan dalam satu ledakan terskrip.
- Langkah penyelesaian:
  1. Ekstraktor mempertahankan symlink Unix dari dalam zip (ekstraksi kustom, bukan zipfile polos) DAN view penyaji berkas mengikuti symlink tanpa pemeriksaan realpath containment.
  2. Zip dibuat berisi satu berkas kontrol biasa + satu entri symlink (create_system=3, external_attr 0o120777<<16, isi entri = path target) yang menunjuk /app/flag.txt.
  3. Upload, parse hash dari halaman hasil, segera GET /<hash>/lk → server mengikuti symlink dan mengembalikan isi /app/flag.txt (HTTP 200).
  4. Bukti mekanisme: zip identik dengan symlink ke /etc/passwd mengembalikan isi passwd asli; symlink relatif ke berkas saudara juga terbaca. Kontrol negatif: /flag, /flag.txt, /app/flag dan traversal ../ semuanya 404; traversal URL langsung pada viewer terblokir rapi; zip-slip tersanitasi. Keunikan: symlink menggantung (target tidak ada) membatalkan seluruh ekstrak, jadi satu symlink per zip dengan target yang ada.
- Tools: Python requests (probe + exploit terskrip), exploit1.py.
- Flag: hiCTF{E45Y_5yml1Nk_ab4f8ebc64}
- Folder bukti: ~/workspace/scratch/new-fileviewer/ (exploit1.py, probe_fast.py s/d probe5.py, sampel HTML respons)
- Catatan hitungan: entri ini = solve platform ke-40 (lihat catatan entri berikutnya soal penghitungan gabungan).

### Soal #40: The Collector's Trail

- Kategori: Miscellaneous (soal gelombang baru)
- Poin: 113 pts saat diterima (181 pts saat enumerasi 09:15; dynamic scoring). Platform menjawab "Flag correct!" dan "Challenge solved!" ±09:29-09:30 WIB 1 Nov (submit via browser Muse). Total tim sesudah dua submit berurutan (File Viewer + soal ini): 2.484 pts, 41/61 soal, peringkat 170/1.070; gabungan kenaikan +229 pts untuk dua soal (113 + ±116).
- Ringkasan masalah: Dua arsip berisi enam reproduksi lukisan terkenal; setiap karya asli menyembunyikan satu fragmen berkunci segel; fragmen diurutkan menurut tahun lahir pelukisnya untuk menurunkan kunci master (HKDF), digest katalog museum menurunkan kata sandi kabinet zip AES, dan payload terakhir menunjuk berkas flag terenkripsi AES-GCM di rilis GitHub.
- Langkah penyelesaian:
  1. Enam karya teridentifikasi dari berkas gambar: I Girl with a Pearl Earring (Vermeer, 1632), II The Third of May 1808 (Goya, 1746), III The Starry Night (Van Gogh, 1853), IV The Kiss (Klimt, 1862), V The Scream versi 1893 (Munch, 1863), VI Death and the Maiden (Schiele, 1890, wafat 1918). Lokator kanonik diverifikasi ke rekor resmi museum: mauritshuis.nl/670, museodelprado.es/P000749, moma.org/472.1941, belvedere.at/912, nasjonalmuseet.no/NG.M.00939, belvedere.at/3171.
  2. Fragmen diekstrak dari enam persembunyian berbeda: chunk PNG kustom coLl; base64 sesudah IEND; teks pada piksel alpha=0 di tepi atas; steghide dengan passphrase = lokator karya; LSB setiap byte biru ke-7; bit byte filter scanline PNG. Keenam fragmen lolos cek segel (2 byte pertama SHA-256 dari 8 byte mentah = nilai cek).
  3. Kunci master = HKDF-SHA256(IKM = 48 byte fragmen terurut tahun lahir, salt "collector-v2:ossuary", info "master", L=32). Digest katalog = SHA-256 enam lokator digabung "\n". Kata sandi kabinet = HKDF-SHA256(master, salt=digest katalog, info "zip", L=16) sebagai heks.
  4. sealed_cabinet.zip (WinZip AES-256 metode 99) didekripsi dengan implementasi tangan PBKDF2-HMAC-SHA1 + AES-CTR + HMAC (PV dan HMAC terverifikasi), menghasilkan readme + stego.png. Skema readme: kunci citra HKDF info "stego"; posisi dipilih HMAC-SHA256(kunci, counter BE 8 byte) mod 6.220.800 posisi RGB, LSB per posisi; frame panjang BE 4 byte (451) + payload + CRC-32 (cocok).
  5. Payload menunjuk repo GitHub broski-creator/velarium-ossuary-ledger-7f3c2a rilis v1.0 aset flag.enc (SHA-256 cocok dengan yang dinyatakan payload); serial ada di pesan tag anotasi v1.0. flag_key = HKDF-SHA256(master, salt=byte serial, info "flag", L=32); flag.enc = nonce(12)‖ct‖tag(16) didekripsi AES-256-GCM dengan AAD "collector-v2" — tag GCM terverifikasi, plaintext adalah flag.
- Tools: Python (PIL, pycryptodome, implementasi WinZip-AES manual), steghide, API GitHub read-only.
- Flag: hiCTF{marble_sparrow_9eb44602eedb764aac082a3816b7d7cb}
- Folder bukti: ~/workspace/scratch/new-collector/ (artefak rantai lengkap: fragmen, kabinet terekstrak, flag.enc, payload)
- Catatan hitungan: entri ini = solve platform ke-41. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #41: Cold Chain

- Kategori: Hardware (soal gelombang baru)
- Poin: 115 pts saat diterima (kemudian tercatat 111 pts dynamic, solver ke-56); platform menjawab "Flag correct!" dan "Challenge solved!" ±09:32-09:35 WIB 1 Nov (submit via browser Muse). Total tim sesudahnya: ±2.582 pts, 42/61 soal, peringkat ±138/1.070.
- Ringkasan masalah: Controller ColdLink CaseManager CM-900 memaksa keenam case pendingin supermarket ke defrost terus-menerus. Autentikasi ke controller, temukan pemaksa defrost, kembalikan semua case di bawah ceiling kepatuhan 5 °C DAN buat bertahan (perbaikan naif tidak bertahan), maka status kepatuhan melepas flag.
- Langkah penyelesaian:
  1. Fase luring: handout (toolkit coldlink_tech.py, referensi protokol CLMP, sampel historian) dianalisis penuh. Auth = challenge/response per koneksi: greeting memberi SERIAL + NONCE; token = sha256("CLNK-FIELD-MASTER-6E1D-DO-NOT-DISTRIBUTE|<serial>|<nonce>").hexdigest()[:16] — field master key tercetak di toolkit handout, berlaku fleet-wide.
  2. Akar masalah terpetakan dari handout dan terbukti di live: schedule S9 WINTER_DEFROST_ALL action=force_defrost enabled=1 diterapkan tiap cycle; macro M2 AUTO_WINTERIZE action=ensure_schedule period 90 dtk menarget S9 dan menghidupkannya lagi walau sudah di-disable atau di-delete (sampel historian membuktikan siklus delete → re-enabled).
  3. exploit_prep.py ditulis dan di-smoke-test end-to-end melawan mock CM-900 lokal (skenario S9+M2 persis) sebelum instance live ada.
  4. Eksekusi live (±81 detik): AUTH CLX-900-SN-4417 diterima (role=technician); diagnosis live cocok persis dengan prep (6 case temp=12.00 mode=defrost; S9 enabled; M2 enabled); urutan fix: MACRO DISABLE M2 → SCHED SET S9 enabled=0 → MODE cool keenam case; dalam 5 detik 6/6 case di bawah 5 °C; hold naik mulus 4→61 detik tanpa satu pun re-forcing; pada held=61s STATUS membalas DATA compliant=1 + flag, lalu QUIT bersih.
  5. Catatan: tap historian live gagal handshake TLS (timeout ±10 dtk) — tidak berpengaruh, jalur control cukup untuk diagnosis dan fix; transkrip control lengkap tersimpan.
- Tools: Python stdlib (TLS+SNI socket), exploit_prep.py, mock_cm900.py.
- Flag: hiCTF{cold_chain_restored_defrost_cleared_707eefa27b}
- Folder bukti: ~/workspace/scratch/new-coldchain/ (ANALYSIS.md, exploit_prep.py, flag.txt, transcript.log)
- Catatan hitungan: entri ini = solve platform ke-42. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #42: Ground Station

- Kategori: Hardware (soal gelombang baru)
- Poin: 113 pts; platform menjawab "Flag correct!" dan "Challenge solved!" ±09:36-09:37 WIB 1 Nov (submit via browser Muse); CakraWatch solver ke-56. Total tim sesudahnya: ±2.688 pts, 43/61 soal, peringkat 120/1.070.
- Ringkasan masalah: Controller GSC-7 stasiun pendukung satelit menjalankan pengisian propelan dan pendinginan; interlock keselamatan urutan pengisian telah diubah engineer sebelumnya. Pulihkan cara controller mengautentikasi engineer, temukan dan perbaiki logika keselamatan yang diubah, buat perbaikan bertahan, dan selesaikan uji fueling/cooling dengan bersih agar controller menerbitkan token validasi (flag).
- Langkah penyelesaian:
  1. Fase luring: firmware gsc-fw.bin (ELF kecil, stripped) didisassembly. Rutin akses dipulihkan penuh: keystream 16 byte dari LCG state=0x1F2E3D4C, state=state*0x19660D+0x3C6EF35F, byte i = ((state>>17) ^ (i*0x5B)) & 0xFF → 50e5d52349171740d3823ef092cdb7bd; device key 8 byte di .rodata "GSC-7"\0\x01\x23 (4753432d37000123); mixing 12 ronde atas kata LE u32 dengan rol 7/11/17/19/13 dan konstanta 0x9E3779B9.
  2. Bukti kebenaran auth: reimplementasi Python mereproduksi jangkar boot firmware persis, access(keystream, key) = de8c1297e603570f cocok dengan image-sig=de8c1297 yang dicetak binary. Respons live = access(keystream, nonce) dengan nonce 8 byte sebagai argumen B (kandidat primer langsung diterima; enam fallback tidak diperlukan).
  3. gsc_driver.py dibangun dan divalidasi end-to-end melawan mock controller lokal (auth pulihan + interlock teralterasi + logika restore watchdog): driver mengautentikasi, memperbaiki hanya rung yang menyimpang, menyimpan backup, menjalankan test, menangkap flag mock, exit 0.
  4. Eksekusi live (±10 detik): AUTH → OK access=engineer. PROG LIST menemukan tepat satu rung menyimpang: INTERLOCK_OK live = (TANK_TEMP <= 145) AND (TANK_PSI <= 100) — ambang suhu dinaikkan dari nominal 45 menjadi 145; tiga rung lain sesuai nominal. PROG SET INTERLOCK_OK ke "TANK_TEMP <= 45 AND TANK_PSI <= 100" → live_checksum fa6a6baaa7d0. BACKUP SAVE menyelaraskan trusted_checksum; verifikasi BACKUP STATUS live == backup == trusted dan arm_will_restore=0 (watchdog tidak perlu dimatikan). RUNTEST → DATA result=SUCCESS reason=fill_target_reached_scan_51 + flag; STATUS mengonfirmasi solved=1.
- Tools: objdump (disassembly firmware), Python (reimplementasi auth + driver GSP/1.0), mock_server.py.
- Flag: hiCTF{ground_station_fueling_test_nominal_e853ea7687}
- Folder bukti: ~/workspace/scratch/new-groundstation/ (ANALYSIS.md, gsc_driver.py, disasm.txt, transkrip sesi live)
- Catatan hitungan: entri ini = solve platform ke-43. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

### Soal #43: Labyrinth

- Kategori: Miscellaneous (soal gelombang baru)
- Poin: 133 pts saat diterima (131 pts tercatat dynamic; CakraWatch solver ke-43); platform menjawab "Flag correct!" dan "Challenge solved!" ±10:02-10:04 WIB 1 Nov (submit via browser Muse). Total tim sesudahnya: ±2.768 pts, 44 soal selesai, peringkat 110/1.072. Penyebut total papan berfluktuasi 61-62 antar pemeriksaan (panitia masih mengubah papan).
- Ringkasan masalah: Server pada port instance berbicara dalam protokol biner yang sama sekali tidak terdokumentasi; setiap jawaban mengajarkan sedikit bahasanya. Pelajari bahasanya, navigasi labirin di belakangnya, dan jawab pertanyaan gerbang terakhir.
- Langkah penyelesaian:
  1. Ronde 1 memecahkan protokol penuh secara live: frame = magic LBY\x7f (4B) | opcode (1B) | panjang u16 BE | body; server menyapa op 0x80. Tahap HELLO (op 0x01) memberi nonce 8 byte; PROBE (op 0x10, body = nonce persis) memicu frame tantangan; ANSWER (op 0x30) memakai transformasi T(w) = balik 4 byte lalu XOR kunci per-sesi K (diturunkan dari pasangan contoh 00000000; diverifikasi terhadap ketiga pasangan contoh tiap sesi sebelum menjawab).
  2. Navigasi op 0x20 MOVE (arah 1B + token ruang 2B; arah 0..3 = (0,-1),(+1,0),(0,+1),(-1,0)); frame ruang op 0xa0 memuat [x int8][y int8][exit-mask][tipe][token][teks]. Token BEROTASI setiap masuk ruang — selalu pakai token frame terakhir; token basi mereset sesi ke tahap 0. Tata letak labirin acak per sesi (5x5-ish, ±22-24 ruang); penjelajah DFS memetakannya dalam ±30-60 detik.
  3. Jebakan: ruang tipe 0x02 bertuliskan "a light!" adalah pintu bercahaya umpan (ruang daun); memasukinya mengantrekan satu frame 0xf0 ekstra berisi umpan hiCTF{not_the_exit} — harus dikonsumsi dengan satu read ekstra sebelum mundur, sesi selamat. Umpan itu BUKAN flag.
  4. Gerbang: ruang daun tipe 0x01 via op 0xa1, teks "a sealed gate. it asks how you came, by the shortest way", dengan tokennya sendiri dan tanpa frame antrean. Jawaban yang benar (ronde 2, sesi pertama): hitung jalur terpendek BFS dari pintu masuk (0,0) ke gerbang atas graf yang sudah dipetakan (17 langkah pada sesi pemenang; gerbang di (1,0), token bff4), lalu kirim op 0x30 dengan body = byte arah mentah jalur itu. Server membalas frame 0xf0 berisi flag. Bentuk yang sudah terbakar di ronde 1 dan tidak diulang: op 0x40 + token + path (umpan + koneksi ditutup; jawaban gerbang yang salah membakar sesi itu saja, bukan instance).
- Tools: Python socket/TLS (transport sess2.py, penjelajah solve.py, driver gate2.py), BFS atas graf ruang.
- Flag: hiCTF{l34rn3d_th3_t0ngu3_w4lk3d_th3_l4byr1nth_a1de74ff77}

## 44. Rocinate (Cryptography) — SOLVED, terkonfirmasi platform 1 Nov ±11:33 WIB
- Poin tercatat: 62 pts (CakraWatch solver ke-244)
- Pendekatan: konstruksi N = p*q dengan 104 draw randint berurutan dari CPython random (MT19937); ternyata draw berasal dari DUA stream random yang terinterleave (generator platform bergaya task-queue): 25 draw di stream X, 79 draw (termasuk a dan r) di stream Y. State Y dipulihkan secara eksak lewat pemodelan GF(2): filter propagasi relasi-twist atas 84,6 juta penempatan menyisakan 63.216 penyintas, ekstensi DFS + eliminasi Gauss GF(2) menentukan window 624 kata state secara eksak. Stream Y dijalankan mundur 87 blok hingga r terverifikasi; s diperoleh dari kombinasi Bezout atas eksponen C_i (gcd = 1), lalu flag = s dalam byte. Verifikasi independen atas seluruh 100 pasangan (Z_i, C_i) dari handout mentah: PASS.
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: verifikasi kriptografis independen lulus 100/100 pasangan; submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{lmaoooo_w0t???y0u_d1d_17_??435p37++}

## 45. Standing Order (Blockchain) — SOLVED, terkonfirmasi platform 1 Nov ±11:40 WIB
- Poin tercatat: 144 pts saat submit (dinamis, turun ke 139 setelahnya); kategori Blockchain kini 5/5
- Pendekatan: dua jalur penebusan (onboarding credit kecil dan approval komite besar) berbagi pemeriksaan tanda tangan yang sama, dan digest voucher keccak256(abi.encodePacked(amount, nonce)) tidak mengikat penerima maupun fungsi. Celah utamanya reentrancy: Disbursement.disburse() mengirim ETH SEBELUM menandai voucherUsed[digest] = true. Kontrak penyerang sebagai recipient masuk kembali ke disburse() dari hook receive() dengan voucher bertanda tangan komite yang sama (1 ETH, nonce 0); voucher yang sama terbayar 20 kali dalam satu transaksi, reserve 20 ETH terkuras ke 0 (ambang menang < 0,2 ETH), Setup.isSolved() = true, lalu GET /flag.
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: SHA-256 handout cocok; digest voucher dihitung ulang lokal cocok dengan kontrak; isSolved() on-chain true dan flag diambil dari endpoint instance; submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{one_sig_two_paths_then_the_l00p_15a5e94e87}

## 46. Cipher (android) — SOLVED, terkonfirmasi platform 1 Nov ±12:07 WIB
- Poin tercatat: 62 pts (CakraWatch solver ke-234); kategori android kini 2/2
- Pendekatan: APK com.hictf26.cipher menyimpan vault terenkripsi (format V1LT v3, 64 record) di raw resource p7.bin. Kode LegacyPassphrase.fromSeed() yang tertinggal di APK membangun passphrase dari leksikon 256 kata (m4.bin) memakai java.util.Random(7640891576956012809 ^ seed), enam kata + sufiks dua digit; jendela seed yang dapat dicari berdasar 764149760 rentang 262144. Brute force menemukan seed 764408707. Kunci root = PBKDF2-HmacSHA256(passphrase, salt store, 32768 iterasi, 256-bit); verifier HMAC-SHA256(root, "cipher-verifier-v3")[:16] cocok persis dengan anchor n3.bin (f3840a2b902c711f0ac8f924c295e815). Tiap record didekripsi AES-GCM dengan kunci turunan HKDF-SHA256 per indeks; 64/64 record terbuka, record #47 "Archived security memo" berisi flag (ejaan "implemnetd" memang dari penulis soal).
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: verifier kriptografis cocok persis dengan anchor dan seluruh vault terdekripsi tanpa gagal; submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{cryptography_should_be_implemnetd_properly}
- Folder bukti: ~/workspace/scratch/new-labyrinth/ (PROTOCOL.md, gate2.py, gate2.out, transcript.log, map.json)
- Catatan hitungan: entri ini = solve platform ke-44. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

## 47. Cold Circuit (Reverse Engineering) — SOLVED, terkonfirmasi platform 1 Nov ±15:21 WIB
- Poin tercatat: 98 pts saat submit (CakraWatch solver ke-71); soal BARU yang muncul di papan ±15:05 (papan membengkak 65 → 68)
- Pendekatan: handout = satu ELF x86-64 PIE ter-strip bernama `check` (SHA-256 cocok dengan kartu). Checker menegakkan panjang input 39, awalan hiCTF{, penutup }, lalu hex-decode 32 karakter tengah menjadi 16 byte. Inti = SPN 20 ronde atas 16 byte itu: tiap ronde tambah round key 16 byte (tabel 320 byte di .rodata 0x2140), S-box tetap 256 byte (0x2020), permutasi byte tetap (0x2120); hasil dibandingkan dengan state harapan 16 byte di 0x2280. Karena S-box dan permutasi bijektif, seluruh ronde dibalik dari state harapan (inverse permutasi → inverse S-box → kurang round key) memakai solve.py; 16 byte input pulih dan payload-nya terbaca ASCII "n0_jtag_needed!!". Anti-debug ptrace hanya membalik satu byte round key bila di-trace (tidak berpengaruh saat jalan normal); dua predikat pembantu adalah tautologi gerbang jalur tak terjangkau.
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: checker asli menerima flag ("Access granted.", exit 0, via argv maupun stdin) dan menolak kontrol meleset satu byte ("Incorrect.", exit 1); submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{6e305f6a7461675f6e65656465642121}
- Folder bukti: ~/workspace/scratch/new-coldcircuit/ (solve.py)
- Catatan hitungan: entri ini = solve platform ke-48. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

## 48. The Quiet Divide (Forensics) — SOLVED, terkonfirmasi platform 1 Nov ±15:34 WIB
- Poin tercatat: 95 pts saat dibuka, 94 pts saat submit (CakraWatch solver ke-76); soal BARU dari papan 68; kategori Forensics kini 7/9
- Pendekatan: pcap 146 MB. Rantai chain-of-custody mb-custody/2 untuk INC-3142/ARCH-14 ada dua versi: /v8/audit 5 event berakhir di epoch 734 COMMIT b34dc728…, log replika 7 event dengan dua event ekstra 735 dari ctrl-west-2 (fork dr-ops tak berwenang; policy case_terminal_epoch=734 + checkpoint W0 beku di 734). Escrow slot A+B di-XOR menjadi TLS keylog (sha256 cocok dengan keylog_sha256). Sesi QUIC .14↔.37:4433 didekripsi (Initial keys dari DCID + keylog) → observasi bertanda tangan ARCH-14 yang sha256 kanoniknya cocok dengan /v3/status. Firmware ELF 7.7.0 dirakit dari HTTP 206 (sha256 cocok manifest); frame PLS2 didekode memakai implementasi referensi /v10/sdk dari trafik itu sendiri. Objek mb-final-seal/10 di /v8/audit + bukti SDK (chain‖HMAC(chain,ELF)‖sig atestasi‖sig observasi‖codec) → HKDF archive key membuka 3 arsip AVLT10 (SQLite); hanya tx yang ter-anchor ke checkpoint asli (5e4f78ad…) yang sah, dua lainnya decoy. 4 fragmen tx pemenang dibuka maskernya → material 32 byte yang commit-nya cocok receipts.fragment_commit; final_key = HKDF(material‖evidence‖sig checkpoint W0, seal salt, "MIRRORBREAK/10/PRODUCTION-CASE") membuka segel.
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: tag AES-256-GCM segel final VALID (bukan tebakan); submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{c8c46fdbd6e5e4eacabfba53a156ff9ab10567f2}
- Folder bukti: ~/workspace/scratch/new-quietdivide/ (SOLUTION.md, DB terdekripsi, ELF, keylog, skrip)
- Catatan hitungan: entri ini = solve platform ke-49. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

## 49. Cairn (Cryptography) — SOLVED, terkonfirmasi platform 1 Nov ±18:20 WIB
- Poin tercatat: 74 pts saat submit (CakraWatch solver ke-132); nilai papan 124 pts pada enumerasi 11:37 (47 solve) meluruh dinamis seiring solve bertambah (131 solve sesaat sebelum submit); kategori Cryptography kini 6/8
- Pendekatan: walk sekuensial 2^20 = 1.048.576 langkah di atas labirin isogeni supersingular. Solver C kustom (aritmetika Fp radix-64 dengan reduksi lipat secp256k1, root-finding jalur trace X + X^p ke charpoly di Fp lalu Cantor-Zassenhaus, gcd pseudo-division bebas inversi, perkalian polinomial Karatsuba) dengan checkpoint tiap 4.096 langkah; checkpoint inilah yang menyelamatkan progres melewati dua reboot VM. Nilai akhir walk j_T = (0686a0dcfa679fe2b32c0dc0fd57aca5ceae0df3919e10943d19467cac99d0a9, a8aeb8489a9d1f8884de7f0b47587c3fa1af7335b349c0e533d4cad5c846bcb4) tercapai ±18:15 WIB; kunci = SHA256(ser(j_T) || "IsogenyLabyrinth") membuka AES-256-GCM berisi flag.
- Delegasi: dikerjakan Muse (divisi Agentic AI)
- Verifikasi sebelum submit: tag AES-256-GCM VALID (satu langkah walk saja salah pasti menggagalkan tag); agen utama menjalankan ulang decrypt.py secara independen dari hex j_T dan memperoleh flag identik; tiga implementasi independen (PARI/cypari2, python-flint, solver C) sepakat bit-exact di langkah 300 dan 4.096; submit terkonfirmasi platform ("Flag correct!" / "Challenge solved!")
- Flag: hiCTF{w41k_th3_sup3rs1n9u14r_l46yr1ntH}
- Folder bukti: ~/workspace/scratch/new-cairn/ (decrypt.py, run1/ckpt.txt)
- Catatan hitungan: entri ini = solve platform ke-50. Dua solve platform yang masih belum masuk log ini: ChronoGrind (teks flag masih diminta ke user) dan The Foxglove Trace (teks flag belum tercatat).

## 50. Stillwell Estates (Web) — SOLVED 1 Nov ±20:10 WIB
- Flag (byte-persis, DIAWALI SATU SPASI): ` hiCTF{s1l3nt_l4n73rn_3573735_c2a1878cdf}`
- Metode: impor JSON terkompresi Brotli ke /api/agency/import/preview membaca berkas server tanpa batas; kode /opt/stillwell/server.js menunjukkan loadFlag() membaca process.env.RCTF_FLAG; dump environ membuktikan token diawali 0x20. Formulir platform memangkas spasi sehingga dua submit normal ditolak; Nugroho mengirim ulang via DevTools Copy as fetch dari laptopnya ±20:10 WIB dengan nilai flag persis termasuk spasi, platform menerima.
- Status setelah solve (TERVERIFIKASI platform oleh Muse 20:12 WIB): "Challenge solved!", CakraWatch solver ke-161 pada soal ini, kategori Web naik 3/7. Total tim: **51/69 soal, 3.008 poin, peringkat #150** (scoreboard menampilkan 1.118 tim). Nilai Stillwell saat verifikasi 69 pts.


## Klasemen final — recheck 2 Nov 2026 ±09:07-09:10 WIB (pasca-tutup, hanya-baca)
- Platform ctf.hiddeninvestigations.net, sesi CakraWatch aktif, header "CTF ended". Scoreboard dimuat ulang 3x dan profil tim dicek: angka SAMA persis di semua pemuatan (STABIL): **3.009 pts, peringkat #151 dari 1.119 tim, 51/69 solve**; peringkat divisi Agentic AI #121; profil menampilkan "3,009 / 5,115 pts".
- Selisih vs snapshot stabil terakhir saat lomba berjalan (20:12 WIB 1 Nov: 3.008 pts, #150/1.118, 51 solve): skor +1 poin, peringkat turun 1 tingkat, total tim +1, solve tetap 51.
- Selisih vs snapshot pasca-tutup 1 Nov 21:03 WIB (3.002 pts/#155/51): skor +7 poin, peringkat membaik 4 tingkat; fluktuasi malam penutupan tidak terulang.
- Batasan: tidak ada label "final resmi terkunci" di platform dan tidak ditemukan pengumuman final terpisah di dalam platform; aturan Rules memungkinkan penyesuaian skor di kemudian hari, jadi angka ini adalah angka live yang stabil pada pengecekan ini, bukan jaminan final yang dikunci penyelenggara.
