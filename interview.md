---
name: interview
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Outputs a spec with user stories.
---

**Bahasa**: Semua pertanyaan wajib disampaikan dalam Bahasa Indonesia. Output spec boleh dalam Bahasa Indonesia atau Inggris sesuai preferensi user.

Use the AskUserQuestionTool to ask about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. — but make sure the questions are not obvious.

If a question can be answered by exploring the codebase, explore the codebase instead.

## STEP 0 — Cek Resume

Sebelum mulai interview, cek apakah `.dev/summary.md` ada di project root:

- Kalau **ada** → baca isinya. Cari spec dengan status `draft`. Jika ada, tanya user:
  > "Ada spec draft yang belum selesai: `{nama-feature}` (dibuat {tanggal}). Lanjutkan interview ini atau mulai yang baru?"
- Kalau **tidak ada** atau user pilih baru → lanjut ke Step 1.

---

## STEP 1 — Interview

Interview relentlessly tentang setiap aspek fitur. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

### Mandatory interview topics

Beyond the feature's core mechanics, you MUST explicitly probe these areas:

- **Entry points**: Where does the user navigate FROM to reach this feature? What existing page/nav/button needs to change?
- **User journey start-to-finish**: Walk through the complete flow from "user is on an existing page" → "user discovers feature" → "user completes the feature" → "user returns to where they were"
- **Roles**: Which user roles interact with this feature? Check `references/project.md` for the project's role list; otherwise infer from the codebase.
- **Edge cases**: What happens on error, concurrent access, mobile vs desktop?
- **State transitions across roles**: For each state change, which roles see it and how (real-time — SSE/websockets, polling, manual refresh)? What does each role's screen show before and after?
- **Terminal states & dead ends**: For every end-state, what's the forward action? What URL params, cookies, or tokens does the entry point URL need?
- **Out-of-scope**: Apa yang TIDAK akan diimplementasikan di iterasi ini?

**Cross-cutting invariants** (MANDATORY): Check `references/project.md` for the project's invariants document. If one exists, read it and for each invariant ask the **"Spec-time question"** section during the interview. Every invariant covers a bug class the team has shipped before. The spec must capture the decision for each one — do not leave them implicit.

Selama interview berlangsung, buat folder dan file draft awal agar bisa di-resume jika sesi terputus:

```
.dev/specs/YYYY-MM-DD-HH-mm_<feature-name>/spec.md
```

Tulis dengan frontmatter status `draft` dan isi spec yang sudah terkumpul sejauh ini. Update file ini setiap beberapa pertanyaan selesai dijawab.

---

## STEP 2 — Draft Review

Setelah semua topik selesai dibahas, susun spec lengkap di memori lalu **tampilkan ke user untuk review** sebelum finalisasi:

```
Interview selesai. Ini draft spec yang akan ditulis:

## {Nama Fitur}
**Deskripsi**: ...

## Acceptance Criteria
- [ ] ...
- [ ] ...

## Out of Scope
- ...

## User Stories
- US-1: As a [role], I [action] so that [outcome].
- US-2: ...

Ada yang mau diubah sebelum spec ditulis?
```

Tunggu konfirmasi user. Jangan finalisasi file sebelum user setuju.

---

## STEP 3 — Tulis Spec & Update Summary

Setelah user konfirmasi, finalisasi spec:

### 3a. Tulis spec file

Path: `.dev/specs/YYYY-MM-DD-HH-mm_<feature-name>/spec.md`

Frontmatter wajib:
```markdown
---
status: ready
feature: <feature-name>
created_at: YYYY-MM-DD HH:mm
---
```

### 3b. Struktur spec wajib

```markdown
---
status: ready
feature: <feature-name>
created_at: YYYY-MM-DD HH:mm
---

# {Nama Fitur}

## Deskripsi
<Penjelasan singkat fitur dan tujuan bisnisnya>

## Roles
<Daftar user role yang terlibat dan permission masing-masing>

## User Journey
<Flow lengkap dari user membuka fitur sampai selesai>

## Acceptance Criteria
- [ ] <kriteria 1 — spesifik dan verifiable>
- [ ] <kriteria 2>
...

## Edge Cases
- <kondisi error, concurrent access, dll>

## Out of Scope
- <apa yang tidak dikerjakan di iterasi ini>

## Technical Notes
<Catatan teknis dari interview — arsitektur, constraint, preferensi implementasi>

## User Stories

- **US-1**: As a [role], I [action] so that [outcome].
- **US-2**: As a [role], I [action] so that [outcome].
...
```

Rules for user stories:
- Every user role that interacts with the feature must have at least one story
- Navigation/entry point stories come FIRST
- Cover happy path, key error states, and edge cases
- Stories must be specific enough to verify — "I can use the feature" is too vague
- Number them (US-1, US-2, ...) so implement-spec can reference them

### 3c. Buat/update `.dev/summary.md`

Jika file belum ada, buat baru. Jika sudah ada, append entry baru.

```markdown
# Dev Summary

| Spec | Feature | Status | Created | Completed |
|------|---------|--------|---------|-----------|
| YYYY-MM-DD-HH-mm_feature | {feature-name} | ready | YYYY-MM-DD | - |
```

Status yang valid: `draft` / `ready` / `in-progress` / `done` / `pending`

### 3d. Informasikan ke user

```
Spec selesai ditulis:
  .dev/specs/YYYY-MM-DD-HH-mm_{feature-name}/spec.md

Jalankan skill implement-spec untuk mulai implementasi.
```

---

## Project Customization

Read `references/project.md` in this skill's directory if it exists. It provides project-specific context:
- User roles and their permissions
- Path to a cross-cutting invariants document (if any)
- Real-time transport preference (SSE, websockets, polling)
- Any other project-specific patterns
