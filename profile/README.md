# Product Requirements Document (PRD)

> **Document Status:** Approved
> **Last Updated:** August 21, 2026
> **Target Platform:** iOS (iPhone-first)
> **Core Stack:** SwiftUI, VisionKit, Vision, AVFoundation, SwiftData

---

## Executive Summary

**Product Name:** 

**Problem Statement:**

Aktor teater tunanetra mengalami hambatan akses naskah (*Format-Access Failure*) ketika naskah dibagikan dalam format PDF hasil *scan* atau foto dokumen tanpa *text layer*, sehingga dokumen tersebut tidak dapat dibaca oleh *Screen Reader* (VoiceOver) secara mandiri.

**Proposed Solution:**

Aplikasi iOS yang mengekstrak teks dari naskah berbasis gambar/PDF menggunakan Vision OCR, memilah teks ke dalam struktur dialog standar via AI parser, dan menyajikannya dalam antarmuka yang ramah *Screen Reader* (VoiceOver) untuk navigasi latihan mandiri.

**Success Metrics:**

* **OCR Conversion Rate:** $\ge 90\%$ akurasi ekstraksi teks naskah ke format terstruktur (`Scene`, `Tokoh`, `Dialog`, `Cue`).
* **VoiceOver Navigation Independence:** $100\%$ komponen antarmuka dapat diakses mandiri menggunakan gestur rotor VoiceOver.
* **Processing Latency:** Proses OCR dan perapian naskah 1–5 halaman selesai dalam $< 5$ detik.

---

## Goals & Scope

### User Goals

1. Mengubah naskah fisik/PDF scan menjadi format teks digital terstruktur dalam satu alur sederhana.
2. Membaca dan menavigasi baris dialog atau adegan latihan secara mandiri menggunakan VoiceOver.
3. Mengakses kembali naskah aktif secara cepat dari beranda.

### Non-Goals (Out of Scope untuk V1)

* **Pencatatan & Revisi Naskah Dinamis:** Fitur revisi suara/manual di tengah latihan ditunda ke **V2**.
* **Large-Scale Batch Scanning:** Tidak memproses dokumen tebal puluhan/ratusan halaman sekaligus (fokus pada 1–10 lembar naskah per sesi).
* **Multi-Device Real-Time Sync:** Tidak ada sinkronisasi *live* antar perangkat kru/sutradara di versi awal.

---

## User Persona & Use Cases

### Primary Persona: Visually Impaired Theater Actor

* **Peran:** Aktor teater / pegiat seni peran tunanetra (*blind / low-vision*).
* **Konteks:** Mempelajari naskah secara mandiri dan mengikuti sesi *reading* bersama tim produksi.
* **Pain Points:** Menerima naskah dalam format gambar/scan yang tidak memiliki lapisan teks (*no text layer*), sehingga *Screen Reader* membacanya sebagai dokumen kosong.

### Key Use Cases

#### Use Case 1: Import & Parse Script

* **Preconditions:** Pengguna menerima file naskah (PDF scan atau foto naskah fisik).
* **Flow:**
1. Pengguna membuka tombol *“Upload / Scan Naskah”* di beranda.
2. Memilih file PDF atau memotret lembaran naskah via kamera.
3. Sistem menjalankan Vision OCR $\rightarrow$ AI menyusun ke format terstruktur.
4. Halaman naskah terbuka dan siap dinavigasi oleh VoiceOver.



#### Use Case 2: Independent Script Navigation

* **Preconditions:** Naskah yang telah diparsing sudah tersimpan di pustaka lokal.
* **Flow:**
1. Pengguna membuka naskah dari menu *Recent Scripts* atau *Library*.
2. Pengguna memutar *Custom Rotor VoiceOver* untuk melompat antar Adegan, Tokoh Tertentu, atau Arahan Panggung (*Stage Cues*).
3. VoiceOver membacakan teks sesuai elemen yang dipilih tanpa hambatan visual.



---

## Product Requirements

### Functional Requirements

#### Must Have (P0) - Core MVP

1. **File Ingestion & OCR Extraction**
* **Deskripsi:** Mengimpor file PDF gambar atau foto lembaran fisik dan mengekstrak *raw text*.
* **User Story:** *As a visually impaired actor, I want to scan image-based scripts so I have a readable text layer.*
* **Acceptance Criteria:**
* [ ] Mendukung import file PDF dan gambar melalui `DocumentPicker` / `PhotosUI`.
* [ ] Ekstraksi teks *on-device* menggunakan Vision OCR.
* [ ] Memancarkan *Accessibility Announcement* selama proses ekstraksi teks berlangsung.




2. **AI Script Formatting Engine**
* **Deskripsi:** Memformat teks mentah menjadi entitas naskah: `Scene`, `Character`, `Dialogue`, dan `Stage Direction`.
* **User Story:** *As an actor, I want my script parsed into structured dialogue so that my screen reader can read it hierarchically.*
* **Acceptance Criteria:**
* [ ] Memisahkan nama tokoh dan teks dialog secara otomatis.
* [ ] Mengelompokkan arahan panggung/posisi (*cues*) sebagai elemen penjelas terpisah.




3. **Accessible Script Reader Page**
* **Deskripsi:** Tampilan pembaca naskah yang optimal untuk VoiceOver bawaan iOS.
* **User Story:** *As an actor, I want to jump across dialogue lines and scenes using VoiceOver rotors.*
* **Acceptance Criteria:**
* [ ] Mendukung *Custom Accessibility Rotor* (Lompat per Adegan, Tokoh, atau Aksi).
* [ ] Lolos uji *Grayscale Accessibility Audit* (kontras tinggi, tidak bergantung pada warna semata).
* [ ] Target area sentuh seluruh tombol minimal $\ge 44 \times 44\text{ pt}$.




4. **Recent Scripts ("Continue Reading")**
* **Deskripsi:** Akses cepat di halaman utama untuk langsung membuka naskah aktif terakhir.
* **Acceptance Criteria:**
* [ ] Menampilkan 1 naskah aktif terakhir beserta metadata waktu pembacaan terakhir.




5. **Library & Search**
* **Deskripsi:** Pustaka penyimpanan lokal untuk seluruh naskah yang pernah dipindai.
* **Acceptance Criteria:**
* [ ] Menampilkan daftar seluruh dokumen naskah yang tersimpan di `SwiftData`.
* [ ] Fitur *Search Bar* yang sepenuhnya kompatibel dengan VoiceOver.





---

#### V2 Roadmap (Out of Scope V1)

1. **Voice & Manual Script Revision:** Perekaman instruksi suara dari sutradara untuk memperbarui baris dialog/posisi di naskah secara otomatis.
2. **Export & Share Script:** Ekspor naskah hasil pembacaan/perapian ke format file eksternal via `ShareLink`.

---

### Non-Functional Requirements

* **Accessibility Compliance:** Mematuhi Apple HIG Accessibility (*Dynamic Type*, VoiceOver traits/labels/hints, dan zero color-dependency).
* **Offline-First:** Seluruh fitur pembacaan, navigasi VoiceOver, dan penyimpanan database lokal wajib berfungsi $100\%$ tanpa koneksi internet.
* **Performance:** Waktu pemrosesan OCR dan *parsing* naskah (1–5 halaman) tidak melebihi 5 detik pada perangkat target.

---

## Technical Specifications

### Data Model Schema (`SwiftData`)

```swift
import Foundation
import SwiftData

@Model
final class ScriptDocument {
    var id: UUID
    var title: String
    var createdAt: Date
    var lastReadAt: Date?
    var rawText: String
    @Relationship(deleteRule: .cascade) var scenes: [ScriptScene]

    init(title: String, rawText: String) {
        self.id = UUID()
        self.title = title
        self.createdAt = Date()
        self.rawText = rawText
        self.scenes = []
    }
}

@Model
final class ScriptScene {
    var sceneNumber: Int
    var sceneTitle: String
    @Relationship(deleteRule: .cascade) var lines: [ScriptLine]

    init(sceneNumber: Int, sceneTitle: String) {
        self.sceneNumber = sceneNumber
        self.sceneTitle = sceneTitle
        self.lines = []
    }
}

@Model
final class ScriptLine {
    var orderIndex: Int
    var characterName: String
    var dialogueText: String
    var stageDirection: String?

    init(orderIndex: Int, characterName: String, dialogueText: String, stageDirection: String? = nil) {
        self.orderIndex = orderIndex
        self.characterName = characterName
        self.dialogueText = dialogueText
        self.stageDirection = stageDirection
    }
}

```

### Apple Framework Mapping

| Modul | Framework Apple | Tanggung Jawab Teknis |
| --- | --- | --- |
| **Ingestion** | `VisionKit`, `Vision` | OCR pemindaian teks dari PDF scan atau foto naskah. |
| **Data Persistence** | `SwiftData`, `Foundation` | Penyimpanan lokal entitas naskah (`Script`, `Scene`, `Line`). |
| **UI & Accessibility** | `SwiftUI`, `Accessibility` | VoiceOver Custom Rotor, Accessibility Traits, dan kontras visual. |
| **Audio Playback** | `AVFoundation` | Komponen audio/speech pendukung jika dibutuhkan di luar VoiceOver. |

---

## Timeline & Milestones

| Sprint | Fokus Deliverable | Status |
| --- | --- | --- |
| **Sprint 1** | Setup Arsitektur MVVM, SwiftData Schema, & Vision OCR PoC | Ready |
| **Sprint 2** | AI Formatting Engine, VoiceOver Rotor, & Accessible Script Reader | Backlog |
| **Sprint 3** | Recent Scripts, Library & Search, UI Grayscale Audit, & Polish MVP | Backlog |