# OpenCode Skill Agent Config

Panduan konfigurasi **OpenCode CLI Agent** yang bersifat portable, reusable, dan dapat digunakan di berbagai komputer serta project.

Tujuannya sederhana:

> Letakkan `CONFIGURE_SKILL_AGENT.md` di dalam project, jalankan OpenCode, lalu biarkan agent melakukan audit environment dan menyiapkan konfigurasi yang memang diperlukan.

Tidak ada API key yang disimpan.  
Tidak ada path komputer tertentu.  
Tidak ada model yang di-hardcode.  
Tidak ada fallback tambahan di OpenCode.

---

## Apa ini?

`CONFIGURE_SKILL_AGENT.md` adalah panduan setup untuk membantu AI coding agent mengonfigurasi environment OpenCode secara aman dan minimal.

Panduan ini mengatur bagaimana agent:

- Melakukan audit sebelum mengubah konfigurasi
- Mempertahankan konfigurasi OpenCode yang sudah bekerja
- Memasang skill yang diperlukan
- Memasang capability tambahan sesuai kebutuhan project
- Memilih model berdasarkan model yang benar-benar tersedia di komputer saat ini
- Memisahkan role `PLAN`, `BUILD`, dan `EXPLORER`
- Menghindari dependency dan konfigurasi yang tidak diperlukan
- Melindungi credential agar tidak masuk Git
- Melakukan validasi sebelum menyatakan setup selesai

Panduan ini dibuat agar **tidak bergantung pada komputer pembuatnya**.

Setiap komputer melakukan audit terhadap environment-nya sendiri.

---

## Prinsip Utama

### 1. Audit sebelum mengubah

Agent harus memahami environment dan konfigurasi yang sudah ada sebelum melakukan perubahan.

```text
Audit
  ↓
Pahami konfigurasi
  ↓
Tentukan kebutuhan
  ↓
Baru melakukan perubahan
