---
name: ux-review
description: Melakukan UX review kode frontend secara terstruktur langsung di dalam Claude. Gunakan skill ini ketika user meminta review UX, mengatakan "review UX kode ini", "cek user experience", "ada masalah UX?", "review frontend ini dari sisi UX", atau menyebut masalah seperti "UX-nya tidak konsisten", "user flow membingungkan", "accessibility bermasalah". Skill ini langsung menjalankan proses review — bukan membuat prompt untuk di-copy ke tempat lain. Selalu gunakan skill ini ketika ada kode frontend yang perlu direview dari sisi UX.
---

# UX Review Skill

Jalankan proses UX review kode frontend ini secara langsung. Ikuti setiap step dengan ketat.

## Prinsip Utama

- **Jangan langsung fix** — scan dulu, diskusi dulu, baru task
- **Jangan ulang temuan yang sudah diputuskan** — selalu cek `.ux-review/summary.md` dulu
- **Setiap temuan dibahas tuntas** — opsi, tradeoff, impact — user yang memutuskan
- **Draft task dibuat saat user bilang FIX** — dikonfirmasi semua di akhir baru difinalisasi
- **Real user only** — jangan flag skenario yang tidak mungkin dialami user nyata di produksi

---

## Checklist UX saat Scan

Gunakan ini sebagai panduan saat membaca kode frontend:

**Accessibility**
- Semantic HTML (heading hierarchy, landmark elements)
- ARIA labels pada elemen interaktif
- Keyboard navigability (tab order, focus visible)
- Color contrast (teks vs background)
- Alt text pada gambar

**User Flow**
- Loading state — apakah user tahu sistem sedang bekerja?
- Empty state — apakah ada feedback kalau data kosong?
- Error state — apakah pesan error jelas dan actionable?
- Success state — apakah user tahu aksi berhasil?
- Dead end — apakah ada jalan buntu di flow?

**Visual Consistency**
- Spacing dan layout konsisten antar komponen
- Typography hierarchy jelas
- Warna dan style komponen sejenis konsisten
- Icon usage konsisten

**Responsiveness**
- Layout tidak rusak di mobile
- Touch target minimal 44x44px
- Tidak ada horizontal scroll yang tidak disengaja
- Font size readable di small screen

**Micro-interactions**
- Hover/focus state ada di elemen interaktif
- Transisi tidak terlalu lambat atau terlalu cepat
- Feedback visual saat user melakukan aksi

**Performance UX**
- Tidak ada layout shift yang mengganggu (CLS)
- Konten penting tidak tertunda di balik lazy load
- Skeleton/placeholder ada saat konten loading

---

## Struktur Folder `.ux-review/`

```
.ux-review/
  summary.md              ← index ringkas semua temuan dari semua sesi (selalu dibaca di awal)
  tasks.md                ← hanya task yang belum done (master, selalu up to date)
  sessions/
    YYYY-MM-DD/
      findings.md         ← detail temuan sesi ini
      tasks.md            ← task yang lahir dari sesi ini (untuk trace balik)
```

**Aturan file:**
- `summary.md` dan `tasks.md` di root `.ux-review/` harus selalu ringkas — ini yang dibaca LLM di awal sesi
- Task yang sudah done dihapus dari `.ux-review/tasks.md` — detail tetap ada di `sessions/`
- Setiap sesi baru buat folder `sessions/YYYY-MM-DD/`

---

## STEP 0 — Cek Summary

Sebelum apapun, cek apakah `.ux-review/summary.md` ada:

- Kalau **ada** → baca isinya. Semua temuan yang sudah berstatus FIX/SKIP/LATER tidak akan dibahas ulang.
- Kalau **tidak ada** → ini sesi pertama, lanjut ke Step 1.

Tanya user jika perlu:
> "Ada folder `.ux-review/` di project ini? Kalau ada, share isi `summary.md`-nya."

---

## STEP 1 — Gather Context

Kumpulkan informasi berikut. Ekstrak dari percakapan dulu — jangan tanya ulang yang sudah dijawab.

**Wajib:**
- Kode frontend yang akan direview (minta kalau belum ada)
- Nama fitur/halaman dan tujuannya
- Siapa target user (persona, konteks penggunaan)
- Framework/stack (React, Vue, dsb) dan environment
- Scope review — pilih satu atau beberapa: `Accessibility` / `User Flow` / `Visual Consistency` / `Responsiveness` / `Micro-interactions` / `Performance UX`

**Tanya kalau belum disebutkan:**
- Device utama user — desktop, mobile, atau keduanya?
- Ada design system atau style guide yang jadi acuan?
- Ada constraint khusus? (tidak bisa ganti library, harus support IE, dsb)

Jangan lanjut ke Step 2 sebelum minimal kode dan scope sudah ada.

---

## STEP 2 — Scan & Triage

Baca seluruh kode frontend. Buat daftar temuan baru — **skip temuan yang sudah ada di `summary.md`**.

Gunakan checklist UX di atas sebagai panduan scan, sesuaikan dengan scope yang disepakati.

Untuk setiap temuan, tentukan:
- Severity: 🔴 Critical / 🟡 Important / 🟢 Minor
- Lokasi spesifik (file/komponen/baris)
- Skenario nyata kapan user akan terdampak
- Dampak ke user experience kalau dibiarkan

**Aturan saat scan:**
- Hanya flag yang akan dialami user nyata di produksi — bukan skenario teoritis
- Jangan flag di luar scope yang disepakati
- Jangan flag yang sudah dijamin oleh design system atau framework
- Jangan tulis solusi — daftar dulu saja

Tampilkan hasilnya ke user:

```
Saya menemukan X temuan UX baru:

#1 — 🔴 [Judul singkat]
- Apa: [satu kalimat]
- Di mana: [lokasi komponen/file]
- Skenario: [kapan user terdampak]
- Impact: [apa yang dirasakan user]

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
Apa akar masalah UX sebenarnya — bukan gejalanya.

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
- Dampak ke user (frustrasi, kebingungan, accessibility barrier)
- Dampak ke bisnis (drop-off, conversion, retention)
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
Why: Alasan + dampak ke user kalau dibiarkan
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

### `.ux-review/summary.md`
Append temuan sesi ini ke file yang sudah ada, atau buat baru.

```markdown
# UX Review Summary

## {feature_name} — sesi {YYYY-MM-DD}
Scope: {scope} | Stack: {stack} | Target device: {device}

| # | Judul | Kategori | Severity | Status | Keputusan |
|---|-------|----------|----------|--------|-----------|
| 1 | {judul} | Accessibility | 🔴 Critical | FIX | Opsi A — tambah aria-label |
| 2 | {judul} | User Flow | 🟡 Important | SKIP | Sudah di-handle design system |
| 3 | {judul} | Responsiveness | 🟢 Minor | LATER | Revisit setelah redesign mobile |
```

---

### `.ux-review/tasks.md`
Append task baru dari sesi ini. Task yang sudah done dihapus dari file ini oleh user secara manual.

```markdown
# Open UX Tasks

### [ ] TASK-N — {judul}
**Priority:** Critical / Important / Minor
**Category:** Accessibility / User Flow / Visual Consistency / Responsiveness / Micro-interactions / Performance UX
**Finding:** #{N} — {nama fitur} (sesi {YYYY-MM-DD})
**What:** {perubahan spesifik}
**Why:** {alasan + dampak ke user}
**Approach:** {opsi yang dipilih}
**Do NOT touch:** {constraint}
**Effort:** Low / Medium / High
```

---

### `.ux-review/sessions/YYYY-MM-DD/findings.md`
Detail lengkap temuan sesi ini untuk keperluan trace balik.

```markdown
# UX Findings — {feature_name}
Tanggal: {YYYY-MM-DD}
Code reviewed: {file/komponen}

## Context
- Stack: {stack}
- Scope: {scope}
- Target user: {persona}
- Target device: {device}
- Design system: {design system kalau ada}
- Constraints: {constraint khusus}

## Findings

### #1 — 🔴 {judul}
- **Category:** {kategori UX}
- **What:** {deskripsi}
- **Where:** {lokasi komponen/file}
- **Real scenario:** {kapan user terdampak}
- **User impact:** {apa yang dirasakan user}
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

### `.ux-review/sessions/YYYY-MM-DD/tasks.md`
Salinan task yang lahir dari sesi ini — untuk trace balik saja.

```markdown
# UX Tasks — {feature_name} ({YYYY-MM-DD})

### [ ] TASK-N — {judul}
(format sama dengan .ux-review/tasks.md)
```

---

Setelah semua file dibuat, ingatkan user:

> "Selesai. File yang dibuat:
> - `.ux-review/summary.md` — diupdate, dibaca di awal sesi berikutnya
> - `.ux-review/tasks.md` — task baru ditambahkan, hapus manual kalau sudah done
> - `.ux-review/sessions/{YYYY-MM-DD}/` — detail sesi ini untuk trace balik"
