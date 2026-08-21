# Product Requirements Document (PRD)

> **Document Status:** Approved
> **Last Updated:** August 21, 2026
> **Target Platform:** iOS (iPhone-first)
> **Core Stack:** SwiftUI, VisionKit, Vision, AVFoundation, Speech, SwiftData

---

## Executive Summary

**Product Name:** 

**Problem Statement:**

Aktor teater tunanetra mengalami hambatan akses naskah (*Format-Access Failure*) ketika naskah dibagikan dalam bentuk PDF gambar atau foto fisik tanpa *text layer*, serta kesulitan melacak catatan dan revisi mendadak di tengah sesi latihan secara mandiri.

**Proposed Solution:**

Aplikasi iOS yang mengekstrak naskah berbasis gambar menggunakan Vision OCR, menyusunnya ke dalam format dialog terstruktur lewat AI parser, dan menyajikannya dalam antarmuka yang ramah *Screen Reader* (VoiceOver) dengan dukungan revisi naskah berbasis suara.

**Success Metrics:**

* **OCR Conversion Rate:** $\ge 90\%$ akurasi ekstraksi teks naskah ke format terstruktur (Scene, Tokoh, Dialog).
* **VoiceOver Navigation Independence:** $100\%$ komponen antarmuka dapat diakses mandiri menggunakan gestur rotor VoiceOver.
* **Revision Latency:** Proses revisi baris dialog via suara selesai dalam $< 3$ detik.

---

## Goals & Scope

### User Goals

1. Mengubah naskah fisik/PDF gambar menjadi teks digital terstruktur dalam satu ketukan.
2. Membaca dan menavigasi baris dialog atau adegan latihan secara mandiri via VoiceOver.
3. Mencatat revisi arahan sutradara di tengah latihan tanpa mengetik manual.

### Non-Goals (Out of Scope untuk V1)

* Pemindaian buku tebal ratusan halaman sekaligus (fokus pada 1–10 lembar adegan per sesi).
* Simulasi suara aktor berbeda berbasis multi-voice AI generatif.
* Sinkronisasi *multi-device* real-time antar perangkat kru.

---

## User Persona & Use Cases

### Primary Persona: Visually Impaired Theater Actor

* **Peran:** Aktor teater / pegiat seni peran tunanetra (*blind / low-vision*).
* **Konteks:** Menghafal naskah mandiri dan mengikuti sesi *reading* / latihan fisik panggung.
* **Pain Points:** Menerima file PDF gambar yang dibaca sebagai "gambar kosong" oleh *Screen Reader*, serta tertinggal saat ada revisi dialog mendadak dari sutradara.

### Key Use Cases

#### Use Case 1: Import & Read Script

* **Preconditions:** Pengguna memiliki file naskah (PDF/foto).
* **Flow:**
1. Pengguna membuka tombol *“Upload Naskah”* di beranda.
2. Memilih file PDF atau memotret lembaran naskah.
3. Sistem menjalankan Vision OCR $\rightarrow$ AI menyusun ke format naskah.
4. Halaman naskah terbuka dan VoiceOver membacakan naskah per adegan/tokoh.



#### Use Case 2: Voice Revision During Rehearsal

* **Preconditions:** Naskah aktif sedang dibuka di ruang latihan.
* **Flow:**
1. Pengguna menekan tombol aksesibel *“Rekam Revisi”*.
2. Pengguna mendiktekan revisi (misal: *"Revisi Scene 1, dialog Budi diganti: Siapa di sana?"*).
3. Sistem mentranskripsi suara $\rightarrow$ AI memperbarui baris terkait di SwiftData.
4. Haptic & VoiceOver mengonfirmasi pembaruan baris naskah.



---

## Product Requirements

### Functional Requirements

#### Must Have (P0) - Core MVP

1. **File Ingestion & OCR Extraction**
* **Deskripsi:** Mengimpor file PDF gambar atau foto fisik dan mengekstrak *raw text*.
* **User Story:** *As an actor, I want to scan image-based scripts so I have readable text for my screen reader.*
* **Acceptance Criteria:**
* [ ] Mendukung import PDF dan foto kamera via `DocumentPicker` / `PhotosUI`.
* [ ] Vision OCR mengekstrak teks *on-device* dengan akurasi tinggi.
* [ ] Terdapat umpan balik audio (*Accessibility Announcement*) saat proses *scanning*.




2. **AI Script Formatting Engine**
* **Deskripsi:** Memilah teks mentah menjadi entitas terstruktur: `Scene`, `Character`, `Dialogue`, dan `Stage Direction`.
* **User Story:** *As an actor, I want my script parsed into scenes and characters so that I can navigate it logically.*
* **Acceptance Criteria:**
* [ ] Memisahkan nama tokoh dan isi dialog secara otomatis.
* [ ] Menandai arahan gerak/panggung (*cues*) sebagai elemen terpisah dari dialog.




3. **Accessible Script Reader Page**
* **Deskripsi:** Halaman pembaca naskah yang optimal untuk VoiceOver bawaan iOS.
* **User Story:** *As an actor, I want to jump between dialogue lines and scenes using VoiceOver gestures.*
* **Acceptance Criteria:**
* [ ] Mendukung *Custom Rotor* VoiceOver (Pilihan: Lompat per Adegan, Tokoh, atau Aksi).
* [ ] Kontras warna dan tata letak lolos uji *Grayscale Accessibility*.
* [ ] Target area sentuh tombol minimal $\ge 44 \times 44\text{ pt}$.




4. **Recent Scripts (Continue Reading)**
* **Deskripsi:** Kartu akses cepat di beranda untuk membuka naskah yang terakhir dibaca.
* **Acceptance Criteria:**
* [ ] Menampilkan 1 naskah aktif terakhir dengan label status adegan terakhir.




5. **Library & Search**
* **Deskripsi:** Daftar seluruh naskah tersimpan beserta pencarian judul.
* **Acceptance Criteria:**
* [ ] Menampilkan daftar naskah lokal via SwiftData.
* [ ] Fitur *Search Bar* yang sepenuhnya dapat diakses via VoiceOver.





---

#### Should Have (P1) - Next Iteration

1. **Voice & Manual Revision**
* **Deskripsi:** Fitur perekaman suara menggunakan framework `Speech` untuk mengubah baris naskah secara *non-destructive*.
* **Acceptance Criteria:**
* [ ] Merekam instruksi vokal dan memetakan pembaruan ke baris naskah yang ditargetkan.
* [ ] Opsi penyuntingan manual melalui *TextEditor* aksesibel.




2. **Export & Share Script**
* **Deskripsi:** Mengekspor naskah yang telah direvisi ke format PDF / Teks via `ShareLink`.



---

### Non-Functional Requirements

* **Accessibility Compliance:** Wajib mendukung *Dynamic Type*, VoiceOver label/hints/traits, dan lolos uji kontras *Grayscale*.
* **Offline-First:** Seluruh fitur pembacaan, navigasi, dan penyimpanan data lokal wajib berjalan $100\%$ tanpa internet.
* **Performance:** Waktu proses OCR dan pemodelan naskah (1–5 halaman) selesai dalam $< 5$ detik.

---

## Technical Specifications

### Data Model Schema (`SwiftData`)

```swift
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
    var isRevised: Bool

    init(orderIndex: Int, characterName: String, dialogueText: String, stageDirection: String? = nil) {
        self.orderIndex = orderIndex
        self.characterName = characterName
        self.dialogueText = dialogueText
        self.stageDirection = stageDirection
        self.isRevised = false
    }
}

```

### Apple Framework Mapping

| Modul | Framework Apple | Tanggung Jawab |
| --- | --- | --- |
| **Ingestion** | `VisionKit`, `Vision` | OCR pemindaian teks dari PDF / Gambar. |
| **Parsing & Model** | `SwiftData`, `Foundation` | Penyimpanan lokal & struktur entitas naskah. |
| **Reader & UI** | `SwiftUI`, `Accessibility` | VoiceOver Rotor, accessibility traits, high-contrast UI. |
| **Revision** | `Speech`, `AVFoundation` | Transkripsi audio instruksi revisi panggung. |

---

## Timeline & Milestones

| Sprint | Fokus Deliverable | Status |
| --- | --- | --- |
| **Sprint 1** | Setup Arsitektur MVVM, SwiftData Models, & Vision OCR PoC | Ready |
| **Sprint 2** | AI Formatting Engine, VoiceOver Rotor, & Script Reader View | Backlog |
| **Sprint 3** | Library, Search, Grayscale UI Audit, & Usability Testing | Backlog |
| **Sprint 4** | Voice-Driven Revision (`Speech`), ShareLink Export, & Polish | Backlog |
