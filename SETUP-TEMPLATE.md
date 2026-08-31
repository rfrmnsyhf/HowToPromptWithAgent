# OpenCode Global Config — SETUP-TEMPLATE

> Template share global `opencode` biar agent langsung paham. Copy ke `~/.config/opencode/`. Beda cuma `apiKey` & `model` per provider (opencode / openrouter / poolside).

## 0. Cara Pakai 1 Menit
1. Copy block `opencode.json` di bawah ke `~/.config/opencode/opencode.json`
2. Set env var (jangan hardcode key):
   ```powershell
   setx POOLSIDE_API_KEY "sky_xxx"
   setx OPENROUTER_API_KEY "sk-or-xxx"
   setx OPENCODE_API_KEY "xxx"
   ```
3. Copy `AGENTS.md` ke `~/.config/opencode/AGENTS.md`
4. Buat file `~/.config/opencode/.ponytail-active` isi `ultra`
5. Restart opencode → `opencode agent list`

---

## 1. `opencode.json` — Template (anonymized, cross-OS)

> Ganti `REPLACE_USER` tidak perlu — sudah pakai `~`. Ganti `apiKey` pakai `{env:...}` sesuai provider lu.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "poolside": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "https://inference.poolside.ai/v1",
        "apiKey": "{env:POOLSIDE_API_KEY}"
      },
      "models": {
        "laguna-s-2.1": {
          "name": "Poolside Laguna S 2.1"
        }
      }
    },
    "openrouter": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "https://openrouter.ai/api/v1",
        "apiKey": "{env:OPENROUTER_API_KEY}"
      }
    },
    "opencode": {
      "options": {
        "apiKey": "{env:OPENCODE_API_KEY}"
      }
    }
  },
  "default_agent": "build",
  "enabled_providers": ["opencode", "openrouter", "poolside"],
  "agent": {
    "plan": {
      "model": "opencode/muse-spark-1.2-contributor-free",
      "description": "Gunakan untuk perencanaan arsitektur, reasoning berat, dan long-horizon tasks"
    },
    "build": {
      "model": "opencode/nemotron-3-ultra-free",
      "description": "Default. Cocok untuk coding, multi-file edit, dan agentic execution sehari-hari"
    },
    "code": {
      "model": "openrouter/z-ai/glm-5.2:free",
      "description": "Khusus coding agent intensif (terminal-heavy, multi-file refactor, SWE-style). Fallback manual via /model poolside/laguna-s-2.1 bila GLM 5.2 free kena rate-limit."
    },
    "automation": {
      "mode": "primary",
      "model": "opencode/nemotron-3-ultra-free",
      "description": "Auto-agent tanpa henti: bekerja otomatis tanpa konfirmasi tiap langkah (auto-approve aksi). Untuk task panjang yang sudah jelas tujuannya.",
      "permission": {
        "edit": "allow",
        "bash": "allow"
      }
    },
    "explorer": {
      "description": "Fast explorer subagent for codebase exploration",
      "mode": "subagent",
      "model": "opencode/claude-haiku-4-5"
    }
  },
  "lsp": true,
  "plugin": ["@dietrichgebert/ponytail"],
  "skills": {
    "paths": ["~/.config/opencode/skills"]
  },
  "model": "opencode/nemotron-3-ultra-free"
}
```

**Yang diganti teman:**
- `provider.*.options.apiKey` → `{env:XXX}` sesuai key masing-masing
- `provider.*.options.baseURL` kalau pakai proxy/custom
- `agent.*.model` → bebas, contoh: `poolside/laguna-s-2.1`, `openrouter/anthropic/claude-sonnet-4`, `opencode/*`

> Catatan: block `mcp.open-design` sengaja tidak di-include (local-only). Teman yang butuh OpenDesign tinggal tambah manual.

---

## 2. `AGENTS.md` — Wajib Copy

Simpan sebagai `~/.config/opencode/AGENTS.md`:

```markdown
# Aturan Hemat Token (Global)

Aturan ini otomatis berlaku di setiap sesi, sebagai pelengkap ponytail ultra.

## Cara merespon
- Jawab langsung dan ringkas: tanpa pembukaan/penutup basi, tanpa basa-basi.
- Output minimal yang cukup. Kalau bisa jawab dalam 1-3 kalimat, lakukan.

## Sebelum menulis kode
- Baca dulu kode/file yang akan disentuh. Pahami alurnya sebelum mengubah.
- Jangan tulis ulang yang sudah ada: gunakan kembali stdlib, util, dan pola yang sudah ada di codebase.
- Hanya edit/baca file yang relevan dengan task. Jangan buka file yang tidak berkaitan.

## Saat menulis
- Utamakan solusi paling sederhana yang bekerja.
- Kurangi token: hindari boilerplate, komentar yang tidak diminta, dan duplikasi.
- Ponytail ultra tetap sumber utama perilaku lazy/sederhana.

## Keamanan
- Jangan pernah mengorbankan validasi input, keamanan, atau pencegahan kehilangan data demi menghemat token.
```

---

## 3. `.ponytail-active`

File `~/.config/opencode/.ponytail-active` isi 1 baris:

```
ultra
```

---

## 4. Skills yang Kebawa

Semua skill di `~/.config/opencode/skills/` ikut kebawa (path generic di atas). Daftar saat ini (24):

- `code-review-and-quality`, `code-simplification`, `debugging-and-error-recovery`
- `git-workflow-and-versioning`, `planning-and-task-breakdown`, `incremental-implementation`, `test-driven-development`, `documentation-and-adrs`, `security-and-hardening`
- `seo-audit`, `seo-coach`, `seo-project-setup`, `competitor-analysis`, `competitive-landscape`, `keyword-research`, `keyword-clustering`, `link-prospecting`, `local-seo`
- `no-ai-slop`, `i-have-adhd`, `customize-opencode`
- ponytail suite: `ponytail`, `ponytail-audit`, `ponytail-debt`, `ponytail-gain`, `ponytail-help`, `ponytail-review`

> Teman tinggal copy folder `skills/` apa adanya.

---

## 5. Verifikasi

```powershell
opencode --version
opencode agent list
opencode plugin list
```

Jika `agent.code` kena rate-limit `openrouter/z-ai/glm-5.2:free`, ganti manual:
```
/model poolside/laguna-s-2.1
```
