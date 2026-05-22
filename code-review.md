---
name: code-review
description: Melakukan code review secara terstruktur langsung di dalam Claude. Gunakan skill ini ketika user meminta review kode, mengatakan "review kode ini", "cek kode saya", "ada yang salah dengan kode ini?", atau menyebut masalah seperti "LLM review-nya muter-muter", "saran tidak masuk akal", "review tidak real world". Skill ini langsung menjalankan proses review — bukan membuat prompt untuk di-copy ke tempat lain. Selalu gunakan skill ini ketika ada kode yang perlu direview.
---

# Code Review Skill

Jalankan proses review kode ini secara langsung. Ikuti setiap step dengan ketat.

## Prinsip Utama

- **Jangan langsung fix** — scan dulu, diskusi dulu, baru task
- **Jangan ulang temuan yang sudah diputuskan** — selalu cek `.review/summary.md` dulu
- **Setiap temuan dibahas tuntas** — opsi, tradeoff, impact — user yang memutuskan
- **Draft task dibuat saat user bilang FIX** — dikonfirmasi semua di akhir baru difinalisasi
- **Real world only** — jangan flag skenario yang tidak mungkin terjadi di produksi

---

## Struktur Folder `.review/`

```
.review/
  summary.md              ← index ringkas semua temuan dari semua sesi (selalu dibaca di awal)
  tasks.md                ← hanya task yang belum done (master, selalu up to date)
  sessions/
    YYYY-MM-DD/
      findings.md         ← detail temuan sesi ini
      tasks.md            ← task yang lahir dari sesi ini (untuk trace balik)
```

**Aturan file:**
- `summary.md` dan `tasks.md` di root `.review/` harus selalu ringkas — ini yang dibaca LLM di awal sesi
- Task yang sudah done dihapus dari `.review/tasks.md` — detail tetap ada di `sessions/`
- Setiap sesi baru buat folder `sessions/YYYY-MM-DD/`

---

## STEP 0 — Cek Summary

Sebelum apapun, cek apakah `.review/summary.md` ada:

- Kalau **ada** → baca isinya. Semua temuan yang sudah berstatus FIX/SKIP/LATER tidak akan dibahas ulang.
- Kalau **tidak ada** → ini sesi pertama, lanjut ke Step 1.

Tanya user jika perlu:
> "Ada folder `.review/` di project ini? Kalau ada, share isi `summary.md`-nya."

---

## STEP 1 — Gather Context

Kumpulkan informasi berikut. Ekstrak dari percakapan dulu — jangan tanya ulang yang sudah dijawab.

**Wajib:**
- Kode yang akan direview (minta kalau belum ada)
- Nama fitur/module dan tujuan bisnisnya
- Siapa user yang terdampak
- Tech stack dan environment (production, staging, dsb)
- Scope review — pilih satu atau beberapa: `Security` / `Performance` / `Business Logic` / `Error Handling` / `Maintainability`

**Tanya kalau belum disebutkan:**
- Apa yang sudah dijamin upstream? (contoh: "user_id sudah dicek di auth middleware")
- Ada constraint khusus? (tidak bisa ganti library, harus response < 500ms, tim kecil, sistem legacy)

Jangan lanjut ke Step 2 sebelum minimal kode dan scope sudah ada.

---

## STEP 2 — Scan & Triage

Baca seluruh kode. Buat daftar temuan baru — **skip temuan yang sudah ada di `summary.md`**.

Untuk setiap temuan, tentukan:
- Severity: 🔴 Critical / 🟡 Important / 🟢 Minor
- Lokasi spesifik (file/fungsi/baris)
- Skenario nyata kapan ini bisa terjadi di produksi
- Dampak bisnis kalau dibiarkan

**Aturan saat scan:**
- Hanya flag yang bisa terjadi di produksi nyata — bukan skenario teoritis
- Jangan flag yang sudah dijamin upstream
- Jangan flag di luar scope yang disepakati
- Jangan tulis solusi — daftar dulu saja

Tampilkan hasilnya ke user:

```
Saya menemukan X temuan baru:

#1 — 🔴 [Judul singkat]
- Apa: [satu kalimat]
- Di mana: [lokasi]
- Skenario: [kapan ini terjadi di produksi]
- Impact: [apa yang rusak]

#2 — 🟡 [Judul singkat]
...

Mau mulai bahas dari temuan mana?
```

---

## STEP 3 — Deep Dive per Temuan

Ulangi langkah ini untuk setiap temuan sampai semua selesai dibahas.

### 3a. Analisis

Ketika user memilih temuan, bahas secara mendalam:

**Root Cause**
Apa akar masalah sebenarnya — bukan gejalanya.

**Opsi**
List semua opsi realistis termasuk "do nothing" kalau relevan.
Untuk tiap opsi:
- Apa yang dilakukan
- Yang didapat vs yang dikorbankan (tradeoff)
- Effort: Low / Medium / High
- Risiko merusak bagian lain: Low / Medium / High
- Cocok dipakai kalau: [kondisi]

**Impact Analysis**
Kalau dibiarkan:
- Jangka pendek (hari/minggu)
- Jangka panjang (seiring scale)
- Worst case realistis di produksi

**Rekomendasi**
Sebutkan opsi yang direkomendasikan dan alasannya. Langsung — jangan terlalu banyak hedging.

**Jangan sentuh**
Bagian kode apa yang harus tetap utuh saat menangani temuan ini.

### 3b. Minta Keputusan

Setelah analisis selesai, tanya:

> "Keputusannya apa?
> - **FIX** — pilih opsi mana?
> - **SKIP** — kenapa?
> - **LATER** — kapan akan dibahas ulang?"

### 3c. Catat Draft Task (kalau FIX)

Kalau user bilang FIX, buat draft task dan simpan di memori — belum ditulis ke file:

```
### [ ] TASK-N — {judul}
Priority: Critical / Important / Minor
Finding: #N
Session: YYYY-MM-DD
What: Perubahan spesifik yang perlu dilakukan
Why: Alasan + impact kalau dibiarkan
Approach: Opsi yang dipilih
Do NOT touch: Bagian yang tidak boleh diubah
Effort: Low / Medium / High
```

Lanjut tanya: *"Lanjut ke temuan mana?"*

---

## STEP 4 — Review Draft Tasks

Setelah semua temuan selesai dibahas, tampilkan semua draft task sekaligus:

```
Semua temuan sudah dibahas. Ini draft tasks yang terkumpul:

[ ] TASK-1 — ...
[ ] TASK-2 — ...
[ ] TASK-3 — ...

Yang di-skip:
- #N [judul] — [alasan]

Yang ditunda:
- #N [judul] — revisit kalau [kondisi]

Ada yang mau diubah, di-reorder, atau dibatalkan sebelum difinalisasi?
```

Tunggu konfirmasi user. Jangan tulis file apapun sebelum user setuju.

---

## STEP 5 — Finalisasi

Setelah user konfirmasi, buat semua file berikut:

### `.review/summary.md`
Append temuan sesi ini ke file yang sudah ada, atau buat baru. Hanya status ringkas — tidak ada detail panjang.

```markdown
# Review Summary

## {feature_name} — sesi {YYYY-MM-DD}
Scope: {scope} | Stack: {stack}

| # | Judul | Severity | Status | Keputusan |
|---|-------|----------|--------|-----------|
| 1 | {judul} | 🔴 Critical | FIX | Opsi B — exponential backoff |
| 2 | {judul} | 🟡 Important | SKIP | Tidak mungkin terjadi di flow bisnis ini |
| 3 | {judul} | 🟢 Minor | LATER | Revisit setelah migrasi microservice |
```

---

### `.review/tasks.md`
Append task baru dari sesi ini. Task yang sudah done dihapus dari file ini oleh user secara manual.

```markdown
# Open Tasks

### [ ] TASK-N — {judul}
**Priority:** Critical / Important / Minor
**Finding:** #{N} — {nama fitur} (sesi {YYYY-MM-DD})
**What:** {perubahan spesifik}
**Why:** {alasan + impact}
**Approach:** {opsi yang dipilih}
**Do NOT touch:** {constraint}
**Effort:** Low / Medium / High
```

---

### `.review/sessions/YYYY-MM-DD/findings.md`
Detail lengkap temuan sesi ini untuk keperluan trace balik.

```markdown
# Findings — {feature_name}
Tanggal: {YYYY-MM-DD}
Code reviewed: {file/module}

## Context
- Stack: {stack}
- Scope: {scope}
- Guaranteed upstream: {apa yang sudah dijamin}
- Constraints: {constraint khusus}

## Findings

### #1 — 🔴 {judul}
- **What:** {deskripsi}
- **Where:** {lokasi}
- **Real scenario:** {kapan terjadi di produksi}
- **Business impact:** {dampak}
- **Status:** FIX
- **Decision:** {opsi yang dipilih + alasan}
- **Do NOT touch:** {bagian yang tidak boleh diubah}

### #2 — 🟡 {judul}
- ...
- **Status:** SKIP
- **Decision:** {alasan skip}

### #3 — 🟢 {judul}
- ...
- **Status:** LATER
- **Decision:** {kondisi kapan dibahas ulang}
```

---

### `.review/sessions/YYYY-MM-DD/tasks.md`
Salinan task yang lahir dari sesi ini — untuk trace balik saja, tidak perlu diupdate.

```markdown
# Tasks — {feature_name} ({YYYY-MM-DD})

### [ ] TASK-N — {judul}
(format sama dengan .review/tasks.md)
```

---

Setelah semua file dibuat, ingatkan user:

> "Selesai. File yang dibuat:
> - `.review/summary.md` — diupdate, dibaca di awal sesi berikutnya
> - `.review/tasks.md` — task baru ditambahkan, hapus manual kalau sudah done
> - `.review/sessions/{YYYY-MM-DD}/` — detail sesi ini untuk trace balik"
