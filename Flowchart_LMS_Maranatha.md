# Flowchart LMS Universitas Kristen Maranatha

Render di GitHub, VS Code (ekstensi *Markdown Preview Mermaid Support*), atau tempel tiap blok ke https://mermaid.live.

## 0. Gambaran Besar Seluruh Fitur

```mermaid
%% Maranatha LMS — 0. Gambaran besar seluruh fitur
%% Prinsip: satu kelas bisa campuran; jalur internal / eksternal ditentukan per enrollment
flowchart TB

  subgraph AKTOR["Pengguna"]
    direction LR
    U1["Mahasiswa internal"]
    U2["Dosen internal"]
    U3["Admin LMS<br/>permission LMSADMIN"]
    U6["Reviewer akademik<br/>ACADEMICHEAD / SUPERUSERAKADEMIK"]
    U4["Peserta eksternal"]
    U5["Pengajar tamu eksternal"]
  end

  OM{{"One Maranatha / SAT<br/>akun, mata kuliah, KRS"}}

  subgraph MLOGIN["1. Modul Login"]
    direction LR
    L1["Login internal<br/>kredensial SAT"]
    L2["Login eksternal<br/>email + password<br/>daftar, verifikasi email, reset"]
    L3["Satu jenis sesi LMS<br/>JWT access + refresh token<br/>roles: ADMIN / LECTURER / USER"]
  end

  subgraph MKURSUS["Kursus dan Enrollment"]
    direction LR
    K1["Kursus akademik<br/>dari mata kuliah One Maranatha<br/>dibuka untuk umum: disetujui akademik"]
    K2["Kursus terbuka<br/>dibuat dosen, prodi, atau pengajar eksternal<br/>wajib disetujui akademik"]
    K3["Enrollment<br/>internal: dari KRS / add-drop<br/>eksternal: daftar mandiri<br/>menentukan mode absensi dan skema nilai"]
  end

  subgraph MBELAJAR["2. Modul Pembelajaran"]
    direction LR
    P1["Kelas akademik: terjadwal<br/>internal + eksternal, jadwal sama"]
    P2["Kelas terbuka: kohort atau mandiri<br/>konten dari template yang sama"]
  end

  subgraph MABSEN["3. Modul Absensi"]
    direction LR
    A1["Internal: check-in per pertemuan<br/>PRESENT / LATE / EXCUSED / ABSENT"]
    A2["Eksternal: progres<br/>aktivitas wajib yang selesai"]
  end

  subgraph MNILAI["4. Modul Penilaian"]
    direction LR
    N0["Input nilai per item asesmen<br/>sama untuk semua peserta"]
    N1["Internal: OBE<br/>item, Sub-CPMK, CPMK, CPL<br/>+ nilai huruf"]
    N2["Eksternal: konvensional<br/>bobot komponen, nilai akhir"]
  end

  subgraph MHASIL["5. Hasil"]
    direction LR
    H1["Internal: nilai final dikunci dosen pengampu<br/>+ laporan capaian OBE"]
    H2["Eksternal: status COMPLETED / NOT PASSED<br/>+ sertifikat"]
  end

  subgraph MDASH["6. Dashboard dan Laporan"]
    direction LR
    D1["Dosen / pengajar"]
    D2["Mahasiswa / peserta"]
    D3["Admin: pengguna, master data,<br/>audit-log, laporan OBE"]
  end

  U1 --> L1
  U2 --> L1
  U3 --> L1
  U6 --> L1
  U6 -. "setujui kursus" .-> MKURSUS
  U4 --> L2
  U5 --> L2
  L1 -. "verifikasi akun" .-> OM
  L1 --> L3
  L2 --> L3

  OM -. "mata kuliah, KRS" .-> K1
  L3 --> K3
  K1 --> K3
  K2 --> K3

  K3 -- "kelas akademik" --> P1
  K3 -- "kelas terbuka" --> P2
  MBELAJAR -- "internal" --> A1
  MBELAJAR -- "eksternal" --> A2
  P1 --> N0
  P2 --> N0
  A1 -. "syarat ikut UAS" .-> N1
  N0 -- "internal" --> N1
  N0 -- "eksternal" --> N2
  A2 -. "syarat selesai" .-> N2
  N1 --> H1
  N2 --> H2
  H1 -. "kirim nilai, tahap berikutnya" .-> OM

  H1 --> MDASH
  H2 --> MDASH
  A1 --> MDASH
  A2 --> MDASH
```

## 1. Modul Login (Internal & Eksternal)

```mermaid
%% Maranatha LMS — 1. Modul Login: internal (SAT / One Maranatha) dan eksternal (email)
%% Render: https://mermaid.live
flowchart TD

  START(["Pengguna membuka /login"]) --> TAB{"Pilih tab"}

  %% ───────────── INTERNAL (SAT) ─────────────
  TAB -- "Civitas Maranatha" --> I1["Browser: POST SAT /login<br/>username, password, version: latest"]
  I1 --> I2{"SAT menerima?"}
  I2 -- tidak --> IERR["Tampilkan: username / password SAT salah"]
  I2 -- ya --> I3["Browser: POST /api/auth/sat/session<br/>body: bearer SAT"]
  I3 --> I4["Backend: GET SAT /data dengan bearer<br/>bearer tidak disimpan"]
  I4 --> I5{"isblocked = 0 dan permissions memuat<br/>MAHASISWA / DOSEN / LMSADMIN?"}
  I5 -- tidak --> I403["403: akun tidak punya akses LMS"]
  I5 -- ya --> I6["Upsert users + internal_accounts by om_user_id<br/>timpa nama, email, permissions, foto, synced_at<br/>users.roles = petakan permissions"]
  I6 --> ST

  %% ───────────── EKSTERNAL (EMAIL) ─────────────
  TAB -- "Akun Eksternal" --> E1["Browser: POST /api/auth/login<br/>email, password"]
  E1 --> E2{"Email ada di external_accounts?"}
  E2 -- tidak --> EERR["401: email atau password salah<br/>pesan sama untuk semua kasus"]
  E2 -- ya --> E3{"locked_until > sekarang?"}
  E3 -- ya --> ELOCK["423: akun dikunci sementara, coba lagi nanti"]
  E3 -- tidak --> E4{"Password cocok? bcrypt.compare"}
  E4 -- tidak --> E5["failed_login_count + 1<br/>jika 5 kali atau lebih: locked_until = sekarang + 15 menit"]
  E5 --> EERR
  E4 -- ya --> E6{"email_verified_at terisi?"}
  E6 -- tidak --> EVER["403: verifikasi email dulu<br/>tombol kirim ulang link"]
  E6 -- ya --> E7["failed_login_count = 0, locked_until = NULL"]
  E7 --> ST

  %% ───────────── SESI (SAMA UNTUK KEDUANYA) ─────────────
  ST{"users.status = active<br/>dan dflag = false?"}
  ST -- tidak --> SUS["403: akun dinonaktifkan admin LMS"]
  ST -- ya --> T1["TokenService<br/>access JWT 15 menit: sub, roles, user_type<br/>refresh token acak, simpan hash di refresh_tokens"]
  T1 --> T2["Response: accessToken di body<br/>Set-Cookie refresh_token httpOnly"]
  T2 --> T3["Update last_login_at<br/>catat audit-log LOGIN"]
  T3 --> R{"Jumlah roles"}
  R -- "1 role" --> PR["Langsung ke portal:<br/>ADMIN ke /admin<br/>LECTURER ke /teacher<br/>USER ke /student"]
  R -- "lebih dari 1" --> PICK["/ portal picker<br/>hanya portal yang diizinkan"]

  %% ───────────── PENDAFTARAN EKSTERNAL ─────────────
  subgraph REGF["Pendaftaran eksternal"]
    direction TB
    G1["/register: nama, email, password"] --> G2{"Cek email"}
    G2 -- "domain maranatha.edu" --> G3["Tolak: gunakan tab Civitas Maranatha"]
    G2 -- "sudah terdaftar" --> G4["Respons generik:<br/>silakan cek email Anda"]
    G2 -- "baru" --> G5["Buat users: external, roles USER<br/>+ external_accounts, password di-hash<br/>+ auth_tokens verify_email, 24 jam"]
    G5 --> G6["Kirim email berisi link verifikasi"]
    G6 --> G7["/verify-email?token=..."]
    G7 --> G8{"Hash cocok, belum dipakai,<br/>belum kedaluwarsa?"}
    G8 -- tidak --> G9["Link tidak valid, minta kirim ulang"]
    G8 -- ya --> G10["Set email_verified_at dan used_at"]
  end
  G10 --> START

  %% ───────────── LUPA PASSWORD ─────────────
  subgraph FORGOT["Lupa password - eksternal"]
    direction TB
    P1["/forgot-password: email"] --> P2["Selalu respons generik<br/>jika email ada: auth_tokens reset_password, 1 jam"]
    P2 --> P3["/reset-password?token=...<br/>password baru"]
    P3 --> P4["Validasi token, simpan hash baru<br/>tandai used_at, cabut semua refresh_tokens user"]
  end
  P4 --> START

  %% ───────────── REFRESH & LOGOUT ─────────────
  subgraph REF["Refresh dan logout - internal dan eksternal"]
    direction TB
    F1["POST /api/auth/refresh<br/>cookie refresh_token"] --> F2{"Status token"}
    F2 -- "valid" --> F3["Cabut token lama, terbitkan pasangan baru<br/>rotasi"]
    F2 -- "kedaluwarsa / tidak ada" --> F4["401: arahkan ke /login"]
    F2 -- "sudah dicabut tapi dipakai lagi" --> F5["Indikasi token dicuri:<br/>cabut semua refresh_tokens user"]
    L1["POST /api/auth/logout"] --> L2["revoked_at = sekarang<br/>hapus cookie"]
  end
```

## 2. Modul Pembelajaran (Persetujuan Kursus, Kelas Akademik & Kelas Terbuka)

```mermaid
%% Maranatha LMS — 2. Modul Pembelajaran (v3: + persetujuan kursus oleh akademik)
%% Prinsip: pacing ditetapkan PER KELAS, bukan per orang.
%%   Kelas akademik = terjadwal, boleh campuran internal + eksternal.
%%   Belajar mandiri = kelas terbuka terpisah yang memakai ulang konten dari template.
flowchart TD

  %% ───────── PEMBUATAN & PERSETUJUAN KELAS ─────────
  OMK["Mata kuliah dari One Maranatha"] --> KA{"Dibuka untuk umum?"}
  KA -- "Tidak, internal saja" --> K1["Kelas akademik<br/>pacing = SCHEDULED<br/>tanpa persetujuan tambahan"]
  KA -- "Ya, diajukan dosen pengampu / prodi" --> C1
  NEW(["Buat kursus terbuka baru"]) --> C0

  subgraph BUAT["Pembuatan dan persetujuan - reviewer: ACADEMICHEAD / SUPERUSERAKADEMIK"]
    direction TB
    C0{"Siapa pembuat kursus terbuka?"}
    C0 -- "Dosen internal / prodi" --> C1
    C0 -- "Pengajar eksternal" --> CV{"Akun sudah diverifikasi<br/>sebagai pengajar eksternal?"}
    CV -- Tidak --> CV1["Belum bisa membuat kursus<br/>ajukan verifikasi pengajar ke akademik"]
    CV -- Ya --> C1
    C1["Susun draft dari template<br/>modul, materi, tugas, bank soal<br/>audience tiap aktivitas<br/>status DRAFT"] --> C3["Ajukan persetujuan<br/>status MENUNGGU_PERSETUJUAN"]
    C3 --> C4{"Reviewer akademik<br/>tidak boleh menyetujui kursus buatannya sendiri"}
    C4 -- Tolak --> C5["DITOLAK + catatan<br/>kembali ke DRAFT"]
    C5 --> C1
    C4 -- Setujui --> C6["DIPUBLIKASI"]
    C6 --> C7{"Perubahan setelah publikasi"}
    C7 -- "Perbaikan materi" --> C8["Langsung berlaku<br/>tercatat di audit-log"]
    C7 -- "Judul, deskripsi,<br/>syarat kelulusan, sertifikat" --> C9["Change request<br/>versi lama tetap tayang"]
    C9 --> C10{"Reviewer menyetujui perubahan?"}
    C10 -- Ya --> C11["Perubahan diterapkan"]
    C10 -- Tidak --> C12["Ditolak + catatan<br/>versi lama tetap berlaku"]
  end

  C6 --> KT{"Jenis kelas yang dipublikasi"}
  KT -- "Mata kuliah dibuka untuk umum" --> K1b["Kelas akademik campuran<br/>pacing = SCHEDULED<br/>keputusan BAA: eksternal ikut jadwal atau mandiri"]
  KT -- "Kursus terbuka" --> K2{"Pacing kursus terbuka"}
  K2 -- "Kohort" --> K3["Kelas terbuka terjadwal<br/>tanggal mulai dan selesai sendiri"]
  K2 -- "Mandiri" --> K4["Kelas terbuka mandiri<br/>pacing = SELF_PACED<br/>access_days per peserta<br/>kuis pakai soal acak dari bank"]

  K1 --> START
  K1b --> START
  K3 --> START
  K4 --> START

  %% ───────── MASUK KELAS ─────────
  START(["Peserta membuka kelas"]) --> B{"Terdaftar di kelas?"}
  B -- Tidak --> B1([Access Denied])
  B -- Ya --> FIL["Server memfilter aktivitas<br/>sesuai participant_type vs audience<br/>aktivitas yang tidak sesuai tidak dikirim API"]
  FIL --> PACE{"Pacing kelas"}

  %% ───────── TERJADWAL (AKADEMIK / KOHORT) ─────────
  PACE -- "SCHEDULED" --> S1
  subgraph SCH["Terjadwal - semua peserta mengikuti jadwal yang sama, termasuk kelas campuran"]
    direction TB
    S1{"Modul sudah dibuka<br/>sesuai jadwal pertemuan?"} -- Belum --> S2["Tampilkan jadwal buka modul"]
    S1 -- Ya --> S3["Pelajari materi dan video<br/>ikut diskusi forum<br/>forum hanya menampilkan nama"]
    S3 --> S4{"Ada tugas / kuis?"}
    S4 -- Tidak --> S11
    S4 -- Ya --> S5{"Dikumpulkan sebelum deadline?"}
    S5 -- Ya --> S6["Tersimpan<br/>kuis: dinilai otomatis<br/>tugas: menunggu penilaian"]
    S5 -- Tidak --> S7{"Kebijakan keterlambatan"}
    S7 -- "Diterima" --> S8["Tersimpan, ditandai TERLAMBAT<br/>penalti sesuai kebijakan"]
    S7 -- "Ditolak" --> S9["Tidak bisa mengumpulkan<br/>nilai item = 0"]
    S6 --> S10["Setelah deadline:<br/>pembahasan kuis dibuka untuk semua"]
    S8 --> S10
    S9 --> S10
    S10 --> S11{"Masih ada pertemuan?"}
    S11 -- Ya --> S1
    S11 -- Tidak --> S12["Akhir periode kelas<br/>kelas diarsipkan, read-only<br/>untuk internal dan eksternal"]
  end

  %% ───────── MANDIRI (KELAS TERBUKA) ─────────
  PACE -- "SELF_PACED" --> E1
  subgraph SELF["Mandiri - hanya di kelas terbuka"]
    direction TB
    E1{"Akses masih berlaku?<br/>sekarang &lt;= access_ends_at"} -- Tidak --> E2(["Akses berakhir<br/>hasil tetap bisa dilihat"])
    E1 -- Ya --> E3["Pilih aktivitas<br/>sistem menyarankan aktivitas wajib berikutnya"]
    E3 --> E4{"Modul terbuka?<br/>prasyarat modul sebelumnya terpenuhi"}
    E4 -- Tidak --> E5["Modul terkunci, tampilkan alasan"]
    E5 --> E3
    E4 -- Ya --> E6{"Jenis aktivitas"}
    E6 -- "Materi / Video" --> E7{"Aturan selesai terpenuhi?"}
    E7 -- Belum --> E8["Simpan posisi terakhir"]
    E8 --> E3
    E7 -- Ya --> E9["Catat activity_completions"]
    E6 -- "Kuis" --> E10{"Skor &gt;= passing score?<br/>soal acak per percobaan"}
    E10 -- Ya --> E9
    E10 -- Tidak --> E11{"Sisa percobaan?"}
    E11 -- Ya --> E3
    E11 -- Tidak --> E12["Aktivitas wajib GAGAL permanen"]
    E6 -- "Tugas" --> E13["Submit, status MENUNGGU REVIEW<br/>peserta boleh lanjut aktivitas lain"]
    E13 -. "asinkron" .-> E14{"Pengajar: memenuhi kriteria?"}
    E14 -- Ya --> E9
    E14 -- Tidak --> E15{"Sisa revisi?"}
    E15 -- Ya --> E13
    E15 -- Tidak --> E12
    E9 --> E16{"Semua aktivitas wajib selesai?"}
    E16 -- Belum --> E3
  end

  %% ───────── KE MODUL LAIN ─────────
  S3 -. "event belajar" .-> ABS["Ke Modul Absensi<br/>internal: check-in pertemuan<br/>eksternal: progres"]
  E9 -. "event belajar" .-> ABS
  S6 --> NIL["Ke Modul Penilaian<br/>internal: OBE<br/>eksternal: konvensional"]
  S8 --> NIL
  S9 --> NIL
  S12 --> NIL
  E16 -- Ya --> NIL
  E12 --> NIL

  NOTE["Kelas campuran = kelas akademik dengan peserta eksternal.<br/>Jadwal sama untuk semua; yang berbeda hanya<br/>absensi dan skema nilai per enrollment.<br/>Laporan OBE hanya menghitung peserta internal."]
  K1b -.- NOTE

  style START fill:#4F46E5,color:#fff
  style C6 fill:#16A34A,color:#fff
  style C5 fill:#DC2626,color:#fff
  style FIL fill:#E5E7EB,color:#111
  style E12 fill:#DC2626,color:#fff
  style S9 fill:#DC2626,color:#fff
  style NIL fill:#7C3AED,color:#fff
  style ABS fill:#0891B2,color:#fff
  style NOTE fill:#FEF3C7,color:#111
```

## 3. Modul Absensi (Internal & Eksternal)

```mermaid
%% Maranatha LMS — 3. Modul Absensi: internal (per pertemuan) dan eksternal (progres)
%% Prinsip: kehadiran = check-in + jenis sesi; activity tracking hanya untuk progres dan analitik
flowchart TD
  START(["Peserta terdaftar di course"]) --> MODE{"attendance_mode<br/>pada enrollment"}

  %% ───────── INTERNAL ─────────
  MODE -- "MEETING - internal" --> M0
  subgraph INT["Internal - per pertemuan, dibuat dosen pengampu"]
    direction TB
    M0["Dosen membuat pertemuan ke-n<br/>jenis: TATAP_MUKA / DARING_SINKRON / ASINKRON<br/>jadwal, modul terkait, toleransi terlambat"] --> M1{"Jenis sesi"}
    M1 -- "Tatap muka / Daring sinkron" --> M2["Dosen membuka check-in<br/>kode / QR dinamis atau tombol check-in"]
    M2 --> M3{"Mahasiswa check-in<br/>dalam jendela waktu?"}
    M3 -- Tidak --> M4["Check-in ditutup<br/>ajukan izin / koreksi ke dosen"]
    M3 -- Ya --> M5{"checked_in_at &lt;= mulai + toleransi?"}
    M5 -- Ya --> PRESENT(["PRESENT"])
    M5 -- Tidak --> LATE(["LATE"])
    M1 -- "Asinkron" --> M6{"Aktivitas wajib modul sesi ini<br/>selesai sebelum batas sesi?"}
    M6 -- Ya --> PRESENT
  end

  subgraph TUTUP["Penutupan sesi - dijalankan server"]
    direction TB
    Z1["Sesi mencapai batas waktu<br/>atau dosen menutup sesi"] --> Z2["Peserta tanpa status<br/>diberi ABSENT"]
    Z2 --> Z3["Dosen koreksi bila perlu<br/>EXCUSED: izin / sakit + catatan<br/>tercatat di audit-log"]
    Z3 --> Z4["Rekap: persentase kehadiran =<br/>PRESENT + LATE dibagi pertemuan selesai"]
    Z4 --> Z5{"Memenuhi minimum kehadiran?<br/>misal 75 persen"}
    Z5 -- Ya --> Z6["Boleh ikut UAS"]
    Z5 -- Tidak --> Z7["Peringatan ke mahasiswa dan dosen<br/>status UAS sesuai kebijakan"]
  end

  PRESENT --> Z1
  LATE --> Z1
  M4 --> Z3

  %% ───────── EKSTERNAL ─────────
  MODE -- "PROGRESS - eksternal" --> P0
  subgraph EXT["Eksternal - progres aktivitas, bukan check-in"]
    direction TB
    P0["Event belajar dari Modul Pembelajaran<br/>lesson_completed, quiz_passed,<br/>assignment_approved"] --> P1["Catat activity_completions<br/>sekali per aktivitas"]
    P1 --> P2["Progres = aktivitas wajib selesai<br/>dibagi total aktivitas wajib<br/>dihitung saat dibaca"]
    P2 --> P3{"Mencapai minimum progres?"}
    P3 -- Belum --> P4["Tampilkan sisa aktivitas wajib"]
    P3 -- Ya --> P5["Syarat penyelesaian terpenuhi"]
    P6["Opsional: ikut sesi live kelas campuran<br/>tercatat, tidak wajib"] -.-> P0
  end

  Z6 --> NIL["Ke Modul Penilaian"]
  Z7 --> NIL
  P5 --> NIL

  Z4 --> DASH["Dashboard<br/>dosen: rekap kehadiran dan progres<br/>peserta: status absensi / progres"]
  P2 --> DASH

  style START fill:#4F46E5,color:#fff
  style PRESENT fill:#16A34A,color:#fff
  style LATE fill:#F59E0B,color:#fff
  style Z2 fill:#DC2626,color:#fff
  style NIL fill:#7C3AED,color:#fff
  style DASH fill:#0891B2,color:#fff
```

## 4. Modul Penilaian (OBE Internal & Konvensional Eksternal)

```mermaid
%% Maranatha LMS — 4. Modul Penilaian: internal OBE dan eksternal konvensional (tanpa OBE)
%% Prinsip: nilai diinput SEKALI per item asesmen; skema enrollment menentukan cara menghitungnya
flowchart TD

  subgraph SETUP["Persiapan - dosen pengampu, sebelum kelas berjalan"]
    direction TB
    S1["Buat item asesmen<br/>kuis, tugas, UTS, UAS, proyek"] --> S2["Tentukan bobot item<br/>validasi: total = 100 persen"]
    S2 --> S3{"Kursus akademik?"}
    S3 -- Ya --> S4["Ambil CPL prodi, CPMK, Sub-CPMK<br/>sesuai RPS mata kuliah"]
    S4 --> S5["Petakan tiap item ke Sub-CPMK + bobot<br/>validasi: setiap Sub-CPMK punya item"]
    S5 --> S6["Kunci pemetaan<br/>hanya dosen pengampu / prodi yang bisa ubah"]
    S3 -- Tidak --> S7["Tentukan nilai minimum lulus<br/>dan syarat sertifikat"]
  end

  subgraph INPUT["Input nilai - sama untuk semua peserta"]
    direction TB
    N1["Nilai per item<br/>kuis: otomatis<br/>tugas / UTS / UAS: dosen atau pengajar tamu, pakai rubrik"] --> N2["Simpan scores<br/>enrollment x item"]
  end

  S6 --> N1
  S7 --> N1
  ABS["Dari Modul Absensi<br/>kehadiran di bawah minimum: status UAS sesuai kebijakan"] -.-> N1
  N2 --> MODE{"Skema penilaian enrollment"}

  %% ───────── INTERNAL: OBE ─────────
  MODE -- "OBE - internal" --> O1
  subgraph OBE["Internal - OBE"]
    direction TB
    O1["Nilai akhir angka<br/>= jumlah nilai item x bobot item"] --> O2["Konversi ke nilai huruf<br/>skala resmi kampus"]
    O1 --> O3["Capaian Sub-CPMK<br/>rata-rata berbobot item yang dipetakan"]
    O3 --> O4["Agregasi ke CPMK"]
    O4 --> O5["Agregasi ke CPL"]
    O5 --> O6{"Ada CPMK di bawah ambang?"}
    O6 -- Ya --> O7["Tandai CPMK belum tercapai<br/>remedial / evaluasi pembelajaran"]
    O7 --> O8
    O6 -- Tidak --> O8
    O2 --> O8["Review dosen pengampu<br/>banding nilai dan remedial<br/>diproses sebelum finalisasi"]
    O8 --> O9["Finalisasi nilai<br/>HANYA dosen pengampu internal<br/>nilai dikunci, perubahan lewat audit-log"]
    O9 --> O10["Laporan capaian OBE<br/>per mahasiswa, kelas, prodi<br/>akses: REPORTOBE"]
    O9 -.-> O11["Kirim nilai ke One Maranatha<br/>tahap berikutnya"]
  end

  %% ───────── EKSTERNAL: KONVENSIONAL ─────────
  MODE -- "Konvensional - eksternal" --> K1
  subgraph KONV["Eksternal - konvensional, tanpa OBE"]
    direction TB
    K1["Nilai akhir<br/>= jumlah nilai item x bobot item<br/>item dan bobot sama dengan internal"] --> K2{"Semua aktivitas wajib selesai<br/>dan nilai akhir &gt;= minimum lulus?"}
    K2 -- Ya --> K3(["COMPLETED"])
    K2 -- Tidak --> K4{"Masih ada aktivitas<br/>atau percobaan tersisa?"}
    K4 -- Ya --> K5["Lanjut belajar<br/>kembali ke Modul Pembelajaran"]
    K4 -- Tidak --> K6(["NOT PASSED<br/>opsi daftar ulang / ajukan ke pengajar"])
    K3 --> K7{"Syarat sertifikat terpenuhi?"}
    K7 -- Ya --> K8["Sertifikat<br/>kode verifikasi unik + QR<br/>opsional: cantumkan kompetensi"]
    K7 -- Tidak --> K9["Completed tanpa sertifikat<br/>tampilkan syarat yang kurang"]
  end

  O10 --> DASH["Dashboard<br/>dosen: gradebook + capaian CPMK<br/>mahasiswa: nilai + capaian<br/>peserta eksternal: nilai + sertifikat"]
  K8 --> DASH
  K9 --> DASH
  K6 --> DASH

  style O9 fill:#7C3AED,color:#fff
  style K3 fill:#16A34A,color:#fff
  style K6 fill:#DC2626,color:#fff
  style K8 fill:#CA8A04,color:#fff
  style DASH fill:#0891B2,color:#fff
  style S6 fill:#E5E7EB,color:#111
```
