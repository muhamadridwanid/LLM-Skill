---
name: implement-spec
description: |
  Skill ini mengimplementasikan fitur berdasarkan spec file yang ada di
  .dev/specs/: membaca spec, melakukan deep-dive task bersama user, mengimplementasikan
  kode, menjalankan unit test hingga lulus, lalu menambahkan report implementasi
  ke dalam spec. Status di-track via .dev/summary.md dan spec frontmatter.
---

## Trigger Intents

Gunakan skill ini jika user meminta hal seperti:
- implementasikan spec
- kerjakan spec
- implement the spec
- implement spec file
- implement feature from spec
- eksekusi spec
- jalankan spec
- kerjakan dari spec
- implement dan test sesuai spec
- buat implementasi berdasarkan spec

## Purpose / Description

Skill ini mengotomatisasi siklus **spec → plan → implement → test → report** pada
project Java Maven:

1. **Resolusi spec**: Baca `.dev/summary.md`, temukan spec yang akan dikerjakan.
   Jika tidak ada spec, buat spec minimal dari deskripsi user.
2. **Resume check**: Jika spec sudah `in-progress`, baca `tasks.md` yang ada —
   lanjut dari task yang masih `[ ]`.
3. **Analisis spec**: Baca spec secara menyeluruh — identifikasi semua task
   berdasarkan acceptance criteria dan user stories.
4. **Deep dive per task**: Bahas setiap task satu per satu dengan user —
   approach, referensi codebase, keputusan LANJUT/UBAH.
5. **Konfirmasi tasks**: Tampilkan semua draft tasks → user konfirmasi → tulis `tasks.md`.
6. **Implementasi**: Kerjakan task satu per satu, update `[x]` di `tasks.md` tiap selesai.
7. **Unit test**: Delegasikan ke skill `run-and-fix-unit-test` (mode `test+fix`, scope `all`).
8. **Report**: Tulis `implementation-report.md` ke folder spec.
9. **Update summary**: Update status di `.dev/summary.md` menjadi `done` atau `pending`.

Gunakan skill ini untuk:
- Mengimplementasikan fitur baru berdasarkan spec yang telah ditulis
- Melanjutkan implementasi yang terputus (resume via `tasks.md`)
- Memastikan implementasi sesuai user stories dan acceptance criteria

Jangan gunakan skill ini untuk:
- Menulis spec baru (gunakan skill `interview`)
- Hanya menjalankan test tanpa ada spec (gunakan skill `run-and-fix-unit-test`)
- Deploy ke production

## Preconditions

- **OS**: Windows 10/11 dengan PowerShell 5.1+
- **Java**: Project Java Maven dengan `pom.xml` valid
- **Maven**: Dikonfigurasi via `maven-auto-resolver` skill
- **Spec file**: Ada di `.dev/specs/` atau user menyebutkan deskripsi fitur secara eksplisit

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| Spec File | No | (auto-detect dari .dev/summary.md) | Path atau nama feature spec. Jika tidak diberikan, list dari summary dan tanya user. |
| Project Path | No | (auto-detect dari konteks workspace) | Path absolut ke root project Java Maven (tempat pom.xml berada). |
| Test Scope | No | all | Scope unit test: `all`, `specific-class`. Default: jalankan semua test. |

## Non-Negotiable Rules

1. **WAJIB** baca `.dev/summary.md` sebelum apapun — ini sumber kebenaran status semua spec.
2. **WAJIB** membaca seluruh spec sebelum mulai koding — jangan skip section apapun.
3. **WAJIB** deep dive setiap task dengan user (Step 3) dan tunggu konfirmasi (Step 4) sebelum mulai coding.
4. **DILARANG** mulai coding sebelum `tasks.md` ditulis dan user konfirmasi.
5. **WAJIB** update `[x]` di `tasks.md` setiap task coding selesai — ini yang memungkinkan resume.
6. **WAJIB** mendelegasikan setup Java/Maven ke skill `maven-auto-resolver` —
   **DILARANG** setup `JAVA_HOME` atau Maven path secara manual.
7. **WAJIB** mendelegasikan eksekusi dan perbaikan unit test ke skill
   `run-and-fix-unit-test` dengan mode `test+fix` dan scope `all` —
   **DILARANG** memanggil `mvn test` langsung tanpa melalui skill tersebut.
8. **DILARANG** menganggap implementasi selesai jika masih ada test gagal.
9. **WAJIB** menulis `implementation-report.md` ke folder spec setelah implementasi selesai.
10. **DILARANG** mengubah konten asli `spec.md` (user stories, acceptance criteria). Hanya boleh update frontmatter status.
11. **WAJIB** update `.dev/summary.md` saat status berubah (in-progress → done/pending).
12. **WAJIB** update status spec di frontmatter `spec.md` saat status berubah.
13. **DILARANG** melakukan deploy, push ke remote, atau modifikasi konfigurasi
    production tanpa explicit approval dari user.
14. **WAJIB** melaporkan setiap langkah ke user secara progresif agar user
    bisa memantau proses.

## Execution Flow

### Step 0 — Resolusi Spec File

**Baca `.dev/summary.md` terlebih dahulu.**

Jika file ada → tampilkan daftar spec dengan statusnya ke user.

**Jika user menyebutkan nama/feature secara eksplisit:**
- Cari di `.dev/summary.md` berdasarkan nama. Gunakan langsung.

**Jika user tidak menyebutkan spec:**

Tampilkan daftar spec yang ada:
```
Ada beberapa spec di .dev/. Pilih yang ingin dikerjakan:

  [1] YYYY-MM-DD-HH-mm_feature-a   status: ready    (dibuat: tanggal)
  [2] YYYY-MM-DD-HH-mm_feature-b   status: in-progress (dibuat: tanggal)
  ...

→ Ketik nomor, nama feature, atau instruksi lain:
```

Prioritaskan spec dengan status `in-progress` — tawarkan untuk dilanjutkan.

**Jika `.dev/summary.md` tidak ada atau kosong:**

> ⚠️ Tidak ada spec di `.dev/`. Disarankan jalankan skill `interview` dulu untuk membuat spec yang matang.
>
> Atau, deskripsikan fitur yang ingin diimplementasikan dan saya akan membuatkan spec minimal.

Jika user memilih lanjut tanpa spec → buat spec minimal:
- Tanya nama fitur dan deskripsi singkat
- Tanya acceptance criteria utama
- Buat `.dev/specs/YYYY-MM-DD-HH-mm_<feature>/spec.md` dengan status `ready`
- Tambahkan ke `.dev/summary.md`

### Step 1 — Resume Check

Cek apakah spec yang dipilih sudah punya `tasks.md`:

**Jika `tasks.md` ada** (sesi sebelumnya terputus):
- Baca `tasks.md`
- Tampilkan ke user progress yang sudah ada:
  ```
  Ditemukan progress implementasi sebelumnya:
  ✅ Buat UserRepository
  ✅ Buat UserService
  [ ] Buat UserController    ← belum selesai
  [ ] Buat unit test

  Lanjutkan dari task yang belum selesai?
  ```
- Jika user setuju → skip Step 2–4, langsung ke Step 5 (eksplorasi) untuk task yang belum done
- Jika user minta mulai ulang → hapus `tasks.md`, lanjut ke Step 2

**Jika `tasks.md` tidak ada** → lanjut ke Step 2.

### Step 2 — Update Status ke in-progress

Update frontmatter `spec.md`:
```markdown
---
status: in-progress
started_at: YYYY-MM-DD HH:mm
---
```

Update `.dev/summary.md` — ubah status spec menjadi `in-progress`.

### Step 3 — Baca dan Analisis Spec

Baca seluruh isi `spec.md`. Identifikasi semua task yang perlu dibuat berdasarkan:
- Acceptance criteria
- User stories
- Technical notes

Untuk setiap acceptance criteria / user story, derivasikan task teknis konkret:
- Class/interface apa yang perlu dibuat atau dimodifikasi
- Method apa yang perlu ditambahkan
- Konfigurasi apa yang perlu diubah
- Test apa yang perlu ditulis

Pisahkan juga **human tasks** yang tidak bisa dikerjakan agent:
- Deploy ke environment production/staging
- Approval dari stakeholder
- Konfigurasi secrets di CI/CD pipeline
- Integrasi dengan sistem eksternal yang memerlukan akses kredensial production

### Step 4 — Eksplorasi Codebase

Sebelum deep dive task, lakukan eksplorasi untuk memahami pola yang ada:

1. **Struktur project**: Baca `pom.xml` untuk memahami dependency.
2. **Pola yang ada**: Cari class yang mirip dengan yang akan dibuat
   (Controller, Service, Repository dengan fitur serupa).
3. **Konvensi package**: Identifikasi package naming convention.
4. **Test patterns**: Cari contoh unit test yang sudah ada untuk memahami
   pola mocking, assertion, dan setup.

Gunakan hasil eksplorasi ini sebagai referensi saat deep dive per task.

### Step 5 — Deep Dive per Task

Ulangi langkah ini untuk setiap task yang diidentifikasi di Step 3.

#### 5a. Presentasi Task

Untuk setiap task, tampilkan ke user:

```
### Task #N — {judul task}

Berdasarkan: {acceptance criteria / user story yang memicu task ini}

Approach:
  {deskripsi konkret apa yang akan dibuat/diubah}
  Package: {package}
  File: {nama file}
  Method/field: {detail}

Referensi pola:
  → {file existing yang dijadikan referensi dari hasil eksplorasi}

Keputusannya?
  - LANJUT — setuju approach ini
  - UBAH   — approach lain?
```

#### 5b. Tangani Keputusan

- **LANJUT** → catat task di memori, lanjut ke task berikutnya
- **UBAH** → diskusikan approach alternatif sampai user setuju, catat versi final

Jangan tulis `tasks.md` dulu — kumpulkan semua di memori.

### Step 6 — Konfirmasi Semua Tasks

Setelah semua task selesai dibahas, tampilkan ringkasan ke user:

```
Semua task sudah dibahas. Ini yang akan diimplementasikan:

Coding Tasks:
  [ ] TASK-1 — {judul}   ({approach singkat})
  [ ] TASK-2 — {judul}   ({approach singkat})
  [ ] TASK-3 — {judul}   ({approach singkat})

Human Tasks (tidak bisa dikerjakan agent):
  [ ] {deskripsi task manusia}   — {alasan}

Ada yang mau diubah, di-reorder, atau ditambah sebelum coding mulai?
```

Tunggu konfirmasi user. **Jangan tulis file apapun sebelum user setuju.**

Setelah konfirmasi → tulis `tasks.md` ke `.dev/specs/YYYY-MM-DD-HH-mm_<feature>/tasks.md`:

```markdown
# Tasks — {feature-name}
Spec: .dev/specs/{folder}/spec.md
Started: YYYY-MM-DD HH:mm

## Coding Tasks
- [ ] TASK-1 — {judul}
  Approach: {detail approach}
  File: {path file}
  Ref: {referensi pola}

- [ ] TASK-2 — {judul}
  ...

## Human Tasks
- [ ] {deskripsi}
  Reason: {mengapa harus manusia}
```

### Step 7 — Implementasi Kode

Kerjakan coding tasks satu per satu sesuai urutan di `tasks.md`.

Urutan implementasi yang direkomendasikan:
1. **Model/Entity**: Buat atau modifikasi class model/entity.
2. **Repository**: Buat atau modifikasi repository interface.
3. **Service**: Buat atau modifikasi service class/interface + implementation.
4. **Controller/Handler**: Buat atau modifikasi controller/endpoint.
5. **Konfigurasi**: Tambah/ubah konfigurasi yang diperlukan.
6. **Test class**: Buat atau modifikasi unit test.

Aturan implementasi:
- Ikuti pola yang ada di codebase (naming, package, annotation).
- Jangan tambah dependency baru di `pom.xml` kecuali spec secara eksplisit memintanya.
- Buat unit test untuk setiap public method yang baru ditambahkan.
- Test HARUS berada di `src/test/java/` dengan naming convention `<ClassName>Test.java`.

**Setelah setiap task selesai**, update `tasks.md` — ubah `[ ]` menjadi `[x]`:
```markdown
- [x] TASK-1 — {judul}
```

### Step 8 — Setup Java/Maven via Skill maven-auto-resolver [WAJIB]

> ⚠️ **DILARANG** setup Java/Maven secara manual di sini. Agent WAJIB mendelegasikan
> seluruh setup ke skill `maven-auto-resolver`. Jangan copy-paste logic Java/Maven
> ke dalam skill ini.

Baca file skill berikut dan ikuti seluruh instruksinya:
```
C:\Users\ridwamu\.copilot\skills\maven-auto-resolver\SKILL.md
```

Input yang WAJIB diberikan ke skill tersebut:
- **Project Path**: path absolut ke folder yang berisi `pom.xml`.

Jika `maven-auto-resolver` gagal atau mengembalikan error, **STOP** dan laporkan
error tersebut ke user — jangan coba lanjut ke build atau test.

### Step 9 — Jalankan Unit Test via Skill run-and-fix-unit-test [WAJIB]

> ⚠️ **DILARANG** menjalankan `mvn` secara langsung di sini untuk keperluan test.
> Agent WAJIB mendelegasikan seluruh eksekusi dan perbaikan test ke skill
> `run-and-fix-unit-test`. Jangan inline-kan logic run/fix test di skill ini.

Baca file skill berikut dan ikuti seluruh instruksinya:
```
C:\Users\ridwamu\.copilot\skills\run-and-fix-unit-test\SKILL.md
```

Input yang WAJIB diberikan ke skill tersebut:
- **Project Path**: path absolut ke project (sama dengan yang digunakan di Step 8).
- **Mode**: `test+fix` — karena implementasi HARUS selesai dengan semua test hijau.
- **Scope**: `all` — jalankan semua test untuk memastikan tidak ada regresi.

> Karena `Mode` dan `Scope` sudah ditentukan di atas, skill `run-and-fix-unit-test`
> **TIDAK PERLU** tanya user lagi untuk kedua parameter itu — sampaikan langsung
> sebagai input eksplisit.

**Jika skill `run-and-fix-unit-test` melaporkan masih ada test gagal setelah
maks retry:**
- Catat semua failure detail di `implementation-report.md` (Step 10).
- Tandai sebagai pending task manusia.
- Jangan anggap implementasi selesai — update status ke `pending`.

### Step 10 — Tulis implementation-report.md

Tulis file `implementation-report.md` ke folder spec:
`.dev/specs/YYYY-MM-DD-HH-mm_<feature>/implementation-report.md`

```markdown
# Implementation Report — {feature-name}

**Date**: YYYY-MM-DD HH:mm
**Implemented by**: GitHub Copilot Agent

## Summary
{Ringkasan singkat apa yang diimplementasikan}

## Files Created
| File | Description |
|---|---|
| `{path/ke/file}` | {deskripsi} |

## Files Modified
| File | Change |
|---|---|
| `{path/ke/file}` | {deskripsi perubahan} |

## Completed User Stories
| Story | Status | Notes |
|---|---|---|
| US-1 | ✅ Done | {catatan} |
| US-2 | ✅ Done | {catatan} |

## Unit Test Report

**Test Command**: `mvn clean test -s <settings-path>`

### Result
- **Total Tests**: {angka}
- **Passed**: {angka}
- **Failed**: 0
- **Skipped**: {angka}
- **Status**: ✅ ALL PASSED

### Test Classes Executed
| Test Class | Tests | Status |
|---|---|---|
| `{TestClassName}` | {jumlah} | ✅ Pass |

## Pending Tasks (Human Required)
| Task | Reason | Status |
|---|---|---|
| {deskripsi task} | {mengapa harus manusia} | 🔲 Pending |

_(Kosong jika tidak ada task yang memerlukan campur tangan manusia)_
```

### Step 11 — Update Status Final

**Jika TIDAK ada pending task manusia:**

Update frontmatter `spec.md`:
```markdown
status: done
completed_at: YYYY-MM-DD HH:mm
```

Update `.dev/summary.md` — ubah status menjadi `done` dan isi kolom `Completed`.

**Jika ADA pending task manusia:**

Update frontmatter `spec.md`:
```markdown
status: pending
pending_since: YYYY-MM-DD HH:mm
```

Update `.dev/summary.md` — ubah status menjadi `pending`.

### Step 12 — Laporan Akhir ke User

```
=== IMPLEMENT-SPEC REPORT ===

Spec       : {nama feature}
Status     : {done | pending}
Folder     : .dev/specs/{folder}/

Implementasi:
  - Files created  : {jumlah}
  - Files modified : {jumlah}
  - User stories   : {jumlah selesai} / {total}

Unit Test:
  - Tests passed   : {jumlah}
  - Tests failed   : 0
  - Status         : ✅ ALL GREEN

Pending Tasks (harus dikerjakan manusia):
  - {task 1} (jika ada)
  (Kosong jika semua selesai)

Files:
  - .dev/specs/{folder}/spec.md
  - .dev/specs/{folder}/tasks.md
  - .dev/specs/{folder}/implementation-report.md
```

## Terminal Execution Rule For Agent

- Jalankan semua PowerShell command satu per satu menggunakan `run_in_terminal`.
- JANGAN menambahkan `exit` agar sesi terminal tetap hidup.
- Set working directory ke `project-root` sebelum menjalankan `mvn`.
- Tangkap `$LASTEXITCODE` setelah setiap perintah mvn untuk cek sukses/gagal.
- JANGAN jalankan `mvn` dalam sesi PowerShell terpisah; gunakan sesi yang sama
  agar environment variable (JAVA_HOME, PATH) tetap aktif.

## Outputs

| Output | Location | Description |
|---|---|---|
| `spec.md` | `.dev/specs/{folder}/` | Spec asli dengan frontmatter status terupdate |
| `tasks.md` | `.dev/specs/{folder}/` | Task list dengan progress `[x]`/`[ ]` |
| `implementation-report.md` | `.dev/specs/{folder}/` | Report implementasi + test result |
| `.dev/summary.md` | `.dev/` | Index semua spec + status terupdate |
| Files di codebase | `src/` | File Java, test, konfigurasi yang dibuat/dimodifikasi |

## Failure Handling

- **Jika `.dev/summary.md` tidak ada dan user tidak ada deskripsi**: Stop, minta user jalankan `interview` dulu atau deskripsikan fitur.
- **Jika pom.xml tidak ditemukan**: Stop, minta user konfirmasi project root.
- **Jika build gagal (bukan test gagal)**: Cek dan perbaiki compile error dulu. Jika tidak bisa diselesaikan agent, tambahkan sebagai pending task manusia.
- **Jika test gagal setelah max retry**: Stop autofix, update status ke `pending`, catat detail failure di `implementation-report.md`.
- **Jika ada konflik file (file sudah ada)**: Baca file yang ada, merge dengan hati-hati. JANGAN overwrite tanpa membaca isinya dulu.

## Related Skills

- `interview` — Membuat spec file yang menjadi input untuk skill ini
- `maven-auto-resolver` — **[WAJIB]** Setup Java/Maven environment
- `run-and-fix-unit-test` — **[WAJIB]** Eksekusi dan autofix unit test. SELALU gunakan mode `test+fix` + scope `all`
- `code-quality` — Opsional: review kode hasil implementasi sebelum finalisasi

## Agent Guidance

- **Baca spec sampai tuntas** sebelum mulai koding. Jangan terburu-buru.
- **Update `tasks.md` setiap task selesai** — ini yang memungkinkan resume jika sesi terputus.
- **Cek pola existing** di codebase sebelum membuat file baru — selalu lebih
  baik mengikuti pola yang sudah ada daripada menciptakan pola baru.
- **Lapor progres secara berkala** kepada user, terutama saat pindah antar fase
  (spec → implement → test → report).
- **Jangan asumsi** bahwa task yang tidak disebutkan di spec tidak penting.
  Baca acceptance criteria secara literal.
- **Jika spec ambigu**, jangan asumsikan — tanya user sebelum implementasi.
- **Jika menemukan bug saat implementasi** yang tidak ada di spec, catat di
  Implementation Report tapi hanya fix jika langsung berdampak pada task yang
  sedang dikerjakan.
- **Urutan file yang dibuat**: Entity → Repository → Service → Controller →
  Test. Urutan ini mengurangi compile error karena dependency antar class.

## Known Limitations

- Skill ini hanya bisa mengimplementasikan backend Java Maven. Frontend (React,
  Vue, Angular) TIDAK termasuk dalam scope skill ini.
- Skill ini TIDAK melakukan deploy atau push ke remote repository.
- Skill ini TIDAK bisa mengakses sistem eksternal yang memerlukan VPN atau
  kredensial khusus.
- Skill ini TIDAK bisa melakukan perubahan database production.
- Jika spec sangat kompleks (>20 user stories), pertimbangkan untuk memecahnya
  menjadi beberapa spec yang lebih kecil.
- Integration test yang memerlukan service eksternal (message broker, third-party
  API) di luar jangkauan skill ini — tandai sebagai pending task manusia.

## Skill Intent Summary

Implementasi otomatis dari spec file ke kode Java yang berjalan dan teruji, dengan laporan lengkap di dalam spec.
