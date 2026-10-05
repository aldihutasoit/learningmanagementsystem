# learningmanagementsystem
Design sistem Learning Management System

## Flowchart LMS Universitas Kristen Maranatha

Status keputusan: **biru** = sudah diputuskan, **kuning putus-putus** = perlu konfirmasi / usulan sementara, **abu-abu** = tahap pengembangan berikutnya.

Render di GitHub, VS Code (ekstensi *Markdown Preview Mermaid Support*), atau tempel tiap blok ke https://mermaid.live.

### 0. Gambaran Besar

```mermaid
%% LMS Universitas Kristen Maranatha — 0. Gambaran Besar
%% Status keputusan per Oktober 2026: biru = sudah diputuskan, kuning putus-putus = perlu konfirmasi, abu-abu = tahap berikutnya
flowchart TB

  subgraph LEG["Keterangan"]
    direction LR
    LG1["Sudah diputuskan"]:::decided
    LG2["Perlu konfirmasi / usulan sementara"]:::pending
    LG3["Tahap pengembangan berikutnya"]:::next
  end

  subgraph AKTOR["Pengguna"]
    direction LR
    U1["Mahasiswa internal<br/>aktif / cuti"]
    U2["Dosen internal"]
    U6["Reviewer akademik<br/>ACADEMICHEAD / SUPERUSERAKADEMIK"]
    U3["Admin LMS"]
    U4["Peserta eksternal<br/>termasuk alumni dan mantan mahasiswa"]
    U5["Pengajar eksternal"]
  end

  OM{{"One Maranatha<br/>akun, status mahasiswa, mata kuliah,<br/>KRS, CPL / CPMK / Sub-CPMK"}}

  subgraph MLOGIN["1. Login"]
    direction TB
    L1["Login internal<br/>akun yang sama dengan One Maranatha"]:::decided
    L1b["Lulus / keluar / DO<br/>akun otomatis menjadi eksternal"]:::pending
    L2["Daftar eksternal dengan Gmail<br/>verifikasi lewat email, tanpa persetujuan<br/>profil: nama, email, alamat, pendidikan"]:::decided
  end

  subgraph MKURSUS["Kursus"]
    direction TB
    K1["Kelas akademik<br/>dari One Maranatha, ikut kalender semester"]:::decided
    K1b["Dibuka untuk umum<br/>disetujui akademik, tanpa kuota"]:::decided
    K2["Kursus terbuka<br/>dibuat dosen, prodi, atau pengajar eksternal<br/>disetujui akademik, tidak ikut kalender semester"]:::decided
    K3["Pengajar eksternal di kelas akademik<br/>wajib SK dan penugasan resmi"]:::decided
  end

  subgraph MBELAJAR["2. Pembelajaran"]
    direction TB
    P1["Kelas akademik terjadwal"]:::decided
    P1b["Peserta eksternal di kelas campuran:<br/>ikut jadwal atau mandiri"]:::pending
    P2["Kursus terbuka mandiri"]:::decided
    P3["Asesmen boleh sama atau berbeda<br/>sesuai arahan pengajar"]:::decided
  end

  subgraph MABSEN["3. Absensi"]
    direction TB
    A1["Internal: check-in per pertemuan"]:::decided
    A1b["Sumber resmi absensi internal:<br/>LMS atau One Maranatha"]:::pending
    A2["Eksternal: progres aktivitas wajib"]:::decided
    A3["Batas minimum diatur dosen"]:::decided
  end

  subgraph MNILAI["4. Penilaian"]
    direction TB
    N0["Nilai diinput per item asesmen"]:::decided
    N1["Internal: nilai angka + huruf<br/>finalisasi + persetujuan akademik, lalu terkunci"]:::pending
    N1b["Capaian OBE / CPMK<br/>khusus internal, data dari One Maranatha"]:::next
    N2["Eksternal: bobot komponen + nilai minimum lulus"]:::pending
    N3["Skala huruf eksternal, remedial, banding"]:::next
  end

  subgraph MHASIL["5. Hasil"]
    direction TB
    H1["Internal: nilai final"]:::decided
    H1b["Kirim nilai dan absensi ke One Maranatha"]:::next
    H2["Eksternal: status lulus + sertifikat"]:::decided
  end

  subgraph INTEG["Integrasi One Maranatha"]
    direction TB
    I1["Daftar API yang tersedia"]:::next
    I2["Usulan: One Maranatha benar untuk data induk,<br/>LMS benar untuk aktivitas belajar"]:::pending
    I3["Usulan: add/drop mengikuti One Maranatha,<br/>data belajar tidak dihapus"]:::pending
  end

  U1 --> L1
  U2 --> L1
  U3 --> L1
  U6 --> L1
  U4 --> L2
  U5 --> L2
  L1 --> L1b
  L1b -.-> L2
  L1 -. "verifikasi akun" .-> OM

  OM -. "mata kuliah, KRS" .-> K1
  K1 --> K1b
  U6 -. "menyetujui" .-> K1b
  U6 -. "menyetujui" .-> K2
  K1 --> P1
  K1b --> P1b
  K2 --> P2
  K3 --> P1

  P1 --> A1
  P1b --> A2
  P2 --> A2
  A1 --- A1b
  P1 --> N0
  P2 --> N0
  A1 -. "syarat ikut UAS" .-> N1
  A2 -. "syarat selesai" .-> N2
  N0 --> N1
  N0 --> N2
  N1 --> N1b
  N2 --> N3
  N1 --> H1
  H1 --> H1b
  N2 --> H2
  H1b -.-> OM
  OM -. "CPL / CPMK" .-> N1b
  INTEG -.- OM

  classDef decided fill:#E3EEFA,stroke:#1D4E89,color:#0F2540
  classDef pending fill:#FFF4D6,stroke:#B7791F,stroke-dasharray:5 3,color:#3D2A00
  classDef next fill:#EFEFEF,stroke:#8A8A8A,stroke-dasharray:2 3,color:#444444
```

### 1. Modul Login

```mermaid
%% LMS Universitas Kristen Maranatha — 1. Modul Login
flowchart TD

  subgraph LEG["Keterangan"]
    direction LR
    LG1["Sudah diputuskan"]:::decided
    LG2["Perlu konfirmasi / usulan sementara"]:::pending
    LG3["Tahap pengembangan berikutnya"]:::next
  end

  START(["Pengguna membuka halaman login LMS"]):::start --> TAB{"Jenis akun"}

  %% ───────── INTERNAL ─────────
  TAB -- "Civitas Maranatha" --> I1["Login dengan akun One Maranatha<br/>username dan password yang sama"]:::decided
  subgraph INT["Login internal"]
    direction TB
    I1 --> I2{"Akun valid di One Maranatha?"}
    I2 -- Tidak --> I2x["Tampilkan: username / password salah"]
    I2 -- Ya --> I3{"Status di One Maranatha"}:::pending
    I3 -- "Mahasiswa aktif / cuti,<br/>dosen, reviewer akademik, admin" --> I4["Masuk sebagai internal<br/>nama, email, foto diperbarui dari One Maranatha"]:::decided
    I3 -- "Lulus / keluar / DO" --> I5["Akun dikonversi menjadi eksternal<br/>riwayat belajar tetap tersimpan<br/>kursus terbuka tetap bisa diakses<br/>kelas akademik lama hanya bisa dilihat"]:::pending
    I3 -- "Tidak punya akses LMS" --> I6["Akses ditolak"]
    I5 --> I7["Login berikutnya lewat jalur eksternal"]
  end
  NOTE1["Status mahasiswa butuh API data mahasiswa<br/>dari One Maranatha"]:::next
  I3 -.- NOTE1

  %% ───────── EKSTERNAL ─────────
  TAB -- "Eksternal" --> E0{"Sudah punya akun?"}
  subgraph EXT["Login dan pendaftaran eksternal"]
    direction TB
    E0 -- Belum --> R1["Daftar dengan akun Gmail"]:::decided
    R1 --> R2{"Gmail sudah dipakai<br/>mahasiswa aktif?"}:::pending
    R2 -- Ya --> R2x["Arahkan ke login Civitas Maranatha<br/>satu orang satu akun"]
    R2 -- Tidak --> R3["Isi profil:<br/>nama, email, alamat, tingkat pendidikan"]:::decided
    R3 --> R4["Link verifikasi dikirim ke email"]:::decided
    R4 --> R5["Klik link: akun aktif<br/>tanpa persetujuan akademik"]:::decided
    R5 --> E1
    E0 -- Sudah --> E1["Login dengan Gmail"]
    E1 --> E2{"Akun valid dan<br/>email sudah terverifikasi?"}
    E2 -- Tidak --> E2x["Tampilkan alasan<br/>kirim ulang link verifikasi"]
  end
  NOTE2["Cara daftar Gmail: tombol Masuk dengan Google<br/>atau email + password, perlu konfirmasi"]:::pending
  R1 -.- NOTE2

  %% ───────── MASUK ─────────
  I4 --> S1
  E2 -- Ya --> S1["Masuk ke LMS"]:::good
  S1 --> S2{"Peran pengguna"}
  S2 -- "Admin" --> P1["Portal admin"]
  S2 -- "Reviewer akademik" --> P2["Persetujuan kursus dan nilai"]
  S2 -- "Dosen / pengajar" --> P3["Portal pengajar"]
  S2 -- "Mahasiswa / peserta" --> P4["Portal belajar"]
  S2 -- "Lebih dari satu peran" --> P5["Pilih portal"]

  classDef decided fill:#E3EEFA,stroke:#1D4E89,color:#0F2540
  classDef pending fill:#FFF4D6,stroke:#B7791F,stroke-dasharray:5 3,color:#3D2A00
  classDef next fill:#EFEFEF,stroke:#8A8A8A,stroke-dasharray:2 3,color:#444444
  classDef start fill:#1D4E89,color:#ffffff,stroke:#1D4E89
  classDef good fill:#16A34A,color:#ffffff,stroke:#16A34A
```

### 2. Modul Pembelajaran

```mermaid
%% LMS Universitas Kristen Maranatha — 2. Modul Pembelajaran
flowchart TD

  subgraph LEG["Keterangan"]
    direction LR
    LG1["Sudah diputuskan"]:::decided
    LG2["Perlu konfirmasi / usulan sementara"]:::pending
    LG3["Tahap pengembangan berikutnya"]:::next
  end

  %% ───────── PEMBUATAN KELAS ─────────
  subgraph BUAT["A. Pembuatan dan persetujuan kelas"]
    direction TB
    OMK["Mata kuliah dari One Maranatha"] --> KA["Kelas akademik<br/>mengikuti kalender semester"]:::decided
    KA --> KB{"Dibuka untuk umum?"}
    KB -- "Ya" --> AJU
    KB -- "Tidak" --> SIAP

    NEW(["Kursus terbuka baru"]) --> PEMBUAT["Dibuat oleh dosen internal,<br/>prodi, atau pengajar eksternal"]:::decided
    PEMBUAT --> DRAFT["Susun materi, tugas, kuis<br/>tandai: untuk semua / khusus internal / khusus eksternal"]:::decided
    DRAFT --> AJU["Ajukan persetujuan"]
    AJU --> REV{"Reviewer akademik<br/>ACADEMICHEAD / SUPERUSERAKADEMIK<br/>tidak menyetujui ajuan sendiri"}:::decided
    REV -- "Tolak + catatan" --> DRAFT
    REV -- "Setujui" --> PUB["Dipublikasikan<br/>tanpa batas kuota"]:::decided
    PUB --> SIAP["Kelas siap digunakan"]

    PUB --> UBAH{"Perubahan setelah dipublikasikan"}
    UBAH -- "Perbaikan materi" --> UB1["Langsung berlaku"]:::decided
    UBAH -- "Judul, deskripsi,<br/>syarat kelulusan, sertifikat" --> UB2["Persetujuan ulang akademik<br/>versi lama tetap tayang"]:::decided

    SK["Pengajar eksternal di kelas akademik<br/>wajib SK dan penugasan resmi,<br/>diperiksa akademik"]:::decided
    SK -.-> KA
  end

  %% ───────── BELAJAR ─────────
  SIAP --> START(["Peserta membuka kelas"]):::start
  START --> B{"Terdaftar di kelas?"}
  B -- Tidak --> B1["Akses ditolak"]
  B -- Ya --> FIL["Tampilkan aktivitas sesuai jenis peserta<br/>asesmen boleh sama atau berbeda"]:::decided
  FIL --> JK{"Jenis kelas"}

  subgraph AKAD["B. Kelas akademik"]
    direction TB
    PT{"Jenis peserta"}
    PT -- "Mahasiswa internal" --> M1["Ikut kalender semester<br/>modul dibuka per pertemuan, ada deadline"]:::decided
    PT -- "Peserta eksternal" --> M2["Ikut jadwal semester (usulan)<br/>atau belajar mandiri"]:::pending
    M1 --> M3["Pelajari materi, kerjakan tugas dan kuis"]
    M2 --> M3
    M3 --> M4["Akhir semester: kelas diarsipkan"]
  end

  subgraph TERB["C. Kursus terbuka"]
    direction TB
    T1["Belajar mandiri<br/>tidak mengikuti kalender semester"]:::decided
    T1 --> T2["Pilih aktivitas: materi, kuis, tugas<br/>progres tersimpan, lanjut kapan saja"]
    T2 --> T3{"Semua aktivitas wajib selesai?"}
    T3 -- Belum --> T2
    T3 -- Ya --> T4["Lanjut ke penilaian akhir"]
    T5["Mahasiswa cuti, keluar, atau DO<br/>tetap bisa mengakses kursus terbuka"]:::decided
    T5 -.-> T1
  end

  JK -- "Kelas akademik" --> PT
  JK -- "Kursus terbuka" --> T1

  PRIV["Pengajar eksternal hanya melihat peserta di kursusnya:<br/>nama, NRP, prodi, serta nilai dan aktivitas di kursus itu"]:::decided
  START -.- PRIV

  M3 -. "kehadiran / progres" .-> ABS["Ke Modul Absensi"]:::start
  T2 -. "progres" .-> ABS
  M4 --> NIL["Ke Modul Penilaian"]:::start
  T4 --> NIL

  classDef decided fill:#E3EEFA,stroke:#1D4E89,color:#0F2540
  classDef pending fill:#FFF4D6,stroke:#B7791F,stroke-dasharray:5 3,color:#3D2A00
  classDef next fill:#EFEFEF,stroke:#8A8A8A,stroke-dasharray:2 3,color:#444444
  classDef start fill:#1D4E89,color:#ffffff,stroke:#1D4E89
```

### 3. Modul Absensi

```mermaid
%% LMS Universitas Kristen Maranatha — 3. Modul Absensi
flowchart TD

  subgraph LEG["Keterangan"]
    direction LR
    LG1["Sudah diputuskan"]:::decided
    LG2["Perlu konfirmasi / usulan sementara"]:::pending
    LG3["Tahap pengembangan berikutnya"]:::next
  end

  START(["Peserta terdaftar di kelas"]):::start --> JP{"Jenis peserta"}

  %% ───────── INTERNAL ─────────
  subgraph INT["A. Mahasiswa internal: absensi per pertemuan"]
    direction TB
    D1["Dosen membuat pertemuan<br/>tatap muka / daring / asinkron"]:::decided
    D1 --> D2{"Jenis pertemuan"}
    D2 -- "Tatap muka" --> C1["Check-in dengan kode / QR<br/>yang berganti-ganti"]:::decided
    D2 -- "Daring" --> C2["Check-in dengan tombol<br/>di sesi live"]:::decided
    D2 -- "Asinkron" --> C3["Hadir jika aktivitas wajib<br/>selesai sebelum batas waktu"]:::decided
    C1 --> ST{"Tepat waktu?"}
    C2 --> ST
    ST -- Ya --> H["HADIR"]:::good
    ST -- Tidak --> L["TERLAMBAT"]:::warn
    C3 --> H
    DM["Dosen juga bisa mengisi manual"]:::decided
    DM -.-> H

    H --> TUTUP["Sesi ditutup"]
    L --> TUTUP
    TUTUP --> ABSEN["Yang belum tercatat otomatis TIDAK HADIR"]:::bad
    ABSEN --> IZIN["Izin / sakit / dispensasi<br/>usulan: ajukan dengan bukti, dosen mengubah status"]:::pending
    IZIN --> REKAP["Rekap persentase kehadiran"]
    REKAP --> MIN{"Memenuhi batas minimum kehadiran?"}
    MIN -- Ya --> UAS["Boleh ikut UAS"]:::good
    MIN -- Tidak --> UAS2["Tidak memenuhi syarat UAS"]:::bad
  end
  MINNOTE["Batas minimum diatur dosen;<br/>untuk internal: ikut peraturan kampus 75 persen?"]:::pending
  MIN -.- MINNOTE
  SRC["Sumber resmi absensi internal:<br/>LMS atau One Maranatha"]:::pending
  SYNC["Sinkron absensi ke One Maranatha"]:::next
  REKAP -.- SRC
  SRC -.- SYNC

  %% ───────── EKSTERNAL ─────────
  subgraph EXT["B. Peserta eksternal: progres, bukan absensi per pertemuan"]
    direction TB
    P1["Setiap aktivitas wajib yang selesai<br/>tercatat sebagai progres"]:::decided
    P1 --> P2["Progres = aktivitas wajib selesai<br/>dibagi total aktivitas wajib"]
    P2 --> P3{"Memenuhi batas minimum progres<br/>yang diatur dosen?"}:::decided
    P3 -- Belum --> P4["Tampilkan sisa aktivitas"]
    P3 -- Ya --> P5["Syarat penyelesaian terpenuhi"]:::good
    P6["Perpanjangan deadline / masa akses<br/>oleh pengajar"]:::pending
    P6 -.-> P1
  end

  JP -- "Mahasiswa internal" --> D1
  JP -- "Peserta eksternal" --> P1
  UAS --> NIL["Ke Modul Penilaian"]:::start
  UAS2 --> NIL
  P5 --> NIL

  classDef decided fill:#E3EEFA,stroke:#1D4E89,color:#0F2540
  classDef pending fill:#FFF4D6,stroke:#B7791F,stroke-dasharray:5 3,color:#3D2A00
  classDef next fill:#EFEFEF,stroke:#8A8A8A,stroke-dasharray:2 3,color:#444444
  classDef start fill:#1D4E89,color:#ffffff,stroke:#1D4E89
  classDef good fill:#16A34A,color:#ffffff,stroke:#16A34A
  classDef warn fill:#D97706,color:#ffffff,stroke:#D97706
  classDef bad fill:#DC2626,color:#ffffff,stroke:#DC2626
```

### 4. Modul Penilaian

```mermaid
%% LMS Universitas Kristen Maranatha — 4. Modul Penilaian
flowchart TD

  subgraph LEG["Keterangan"]
    direction LR
    LG1["Sudah diputuskan"]:::decided
    LG2["Perlu konfirmasi / usulan sementara"]:::pending
    LG3["Tahap pengembangan berikutnya"]:::next
  end

  %% ───────── PERSIAPAN & INPUT ─────────
  subgraph PREP["A. Persiapan dan input nilai"]
    direction TB
    S1["Pengajar membuat tugas, kuis, UTS, UAS"]:::decided
    S1 --> S2["Tandai tiap asesmen:<br/>untuk semua / khusus internal / khusus eksternal"]:::decided
    S2 --> S3["Atur bobot nilai<br/>internal dan eksternal boleh berbeda"]:::decided
    S3 --> S4["Nilai diinput per asesmen<br/>kuis otomatis, tugas dan ujian oleh pengajar"]:::decided
  end

  ABS["Dari Modul Absensi:<br/>syarat UAS / syarat penyelesaian"] -.-> S4
  S4 --> JP{"Jenis peserta"}

  %% ───────── INTERNAL ─────────
  subgraph INT["B. Mahasiswa internal"]
    direction TB
    N1["Nilai akhir angka<br/>lalu nilai huruf skala kampus"]:::decided
    N1 --> F0{"Siapa yang memfinalisasi?"}
    F0 -- "Dosen pengampu internal" --> F1["Finalisasi nilai"]
    F0 -- "Pengajar eksternal ber-SK" --> F2["Finalisasi nilai"]:::decided
    F1 --> AP["Persetujuan akademik"]:::pending
    F2 --> AP2["Persetujuan akademik"]:::decided
    AP --> LOCK
    AP2 --> LOCK["Nilai final dan terkunci<br/>usulan: perubahan hanya oleh akademik<br/>dengan alasan tercatat"]:::pending
    LOCK --> KIRIM["Kirim nilai ke One Maranatha"]:::next
    MP["Beberapa pengajar dalam satu kelas:<br/>usulan sementara, satu pengajar utama<br/>yang memfinalisasi"]:::pending
    MP -.-> F0
    OBE["Capaian OBE: Sub-CPMK, CPMK, CPL<br/>data kurikulum dari One Maranatha<br/>pemetaan asesmen ke Sub-CPMK"]:::next
    N1 -.-> OBE
  end

  %% ───────── EKSTERNAL ─────────
  subgraph EXT["C. Peserta eksternal"]
    direction TB
    E1["Usulan sementara:<br/>nilai akhir = bobot komponen<br/>yang diatur pengajar"]:::pending
    E1 --> E2{"Aktivitas wajib selesai dan<br/>nilai akhir mencapai minimum?"}:::pending
    E2 -- Ya --> E3["LULUS"]:::good
    E2 -- Tidak --> E4["TIDAK LULUS"]:::bad
    E3 --> E5["Sertifikat"]:::decided
    E6["Tanpa capaian CPMK<br/>CPMK khusus mahasiswa internal"]:::decided
    E6 -.-> E1
  end

  JP -- "Mahasiswa internal" --> N1
  JP -- "Peserta eksternal" --> E1

  NEXT["Ditunda ke tahap berikutnya:<br/>skala huruf eksternal, remedial,<br/>perbaikan dan banding nilai,<br/>keberatan atas rumus yang berbeda"]:::next
  E4 -.-> NEXT
  LOCK -.-> NEXT

  classDef decided fill:#E3EEFA,stroke:#1D4E89,color:#0F2540
  classDef pending fill:#FFF4D6,stroke:#B7791F,stroke-dasharray:5 3,color:#3D2A00
  classDef next fill:#EFEFEF,stroke:#8A8A8A,stroke-dasharray:2 3,color:#444444
  classDef good fill:#16A34A,color:#ffffff,stroke:#16A34A
  classDef bad fill:#DC2626,color:#ffffff,stroke:#DC2626
```

### Lampiran Teknis: Detail Login

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
