# PLAN PERBAIKAN — hasil validasi `AUDIT_DAHOST.md` terhadap repo `mzkkajsbd/DA`

Tanggal validasi: 2026-09-19 (sesi lanjutan). Metode: setiap temuan dibuka berkasnya di `/app/backend` & `/app/frontend`
(grep + baca kode), bukan mempercayai dokumen audit. Angka audit dibuat pada repo `pandeyoga/DAHOST@df6fb5c`; repo ini
adalah turunannya, jadi sebagian angka berbeda sedikit tetapi **polanya sama**.

## A. HASIL VALIDASI

| ID | Putusan | Bukti di repo ini |
|---|---|---|
| T-01 | **VALID** | `require_auth` hanya memuat `_permissions`, tidak menegakkan. Contoh persis ada: `dewi_rnd_hpp.py:779 delete_hpp`, `dewi_hris_performance.py:456 submit_review` (hanya `Depends(require_auth)`), `marketing_scope_guard.py:80` melewatkan peran non-toko. Hitung kasar saya (pola sederhana, 12 baris pertama): **580 dari 999** endpoint tulis tanpa kata kunci peran, **79 DELETE**. Urutan besaran sama dengan audit (651/1.323). |
| T-02 | **VALID** | `auth.py:182,193` seed `admin@garment.com / Admin@123`; `deploy/README_DEPLOY.md:10` mencetaknya. Tidak ada `BOOTSTRAP_ADMIN_*`. Repo publik. |
| T-03 | **VALID** | Penulis `rahaza_work_orders` (insert/update/delete) di luar tests/scripts: **0**. Pembaca: **23 berkas** di routes/services/core (lebih banyak dari 17 di audit). `ARCHITECTURE.md` menetapkan `production_jobs` sebagai SSOT internal. |
| T-04 | **VALID** | `product_costing.py:660` `computable = bool(bom_id) and unvalued_count == 0`; status `unlinked` (baris 323) hanya menambah gap, tidak menaikkan penghitung; `apply_model_cost` (baris 851) hanya menyaring `not bom_id or hpp_unit <= 0`. |
| T-05 | **VALID** | Dua berkas ada (4.626 vs 3.964 byte). Salinan `utils/` tidak memuat `po_accessories`/`production_variances`/`rahaza_ar_invoices` (0 hit vs 3). `master_data.py:21` masih mengimpor salinan lama; `production_pos.py:18` & `maklon_seed.py:20` memakai salinan baru. |
| T-06 | **VALID** | `dewi_maklon_pos.py:557 PUT /pos/{po_id}` tidak memeriksa `mirror_of` (mirror ditulis di baris 451); `production_maklon_bridge.py:182` `$set` ulang `mirror_fields`. Pola penjaga sudah ada di `dewi_maklon_billing.py:319`. |
| T-07 | **VALID** | `production_execution.py:717 delete_job` hanya `delete_many` 3 koleksi; tidak memanggil `_void_je_by_source` (ada di `rahaza_posting.py:254`), tidak membersihkan `fg_cost_layers`/`rahaza_hpp_snapshots` — padahal `maklon_seed.py:428` melakukannya. |
| T-14 | **VALID (dipersempit)** | `dewi_cmt_partners.rate_per_pcs` tidak punya penulis di luar demo seed; `TEMPLATE_MASTER_DA.xlsx` sheet `11_VENDOR_CMT` = `kode·nama·nama_kontak·telepon·alamat·kapasitas_pcs·keterangan` (tanpa tarif); `variance_flag` (`production_maklon_bridge.py:396`) hanya cek kuantitas; teks gap `product_costing.py:435` menunjuk sumber yang tak bisa diisi. Catatan: layar **Biaya Jahit SPK** (`production_sewing_cost.py`) memang ada dan menulis `cmt_price_snapshot` — jadi mekanisme benar, hanya pintu lahirnya sempit. |
| T-08 | **VALID sebagian** | 3× `APPROVER_ROLES` identik tanpa `accounting` (`employee_expense_claims:53`, `employee_travel_requests:60`, `employee_travel_settlements:52`), `ADMIN_ROLES` di `employee_expense_gl_mapping:40` & `employee_expense_category_master:35`, `rahaza_ar_360:87`, `rahaza_channel_gl_mapping:38`, `employee_expense_summary:31,99`. Peran yang di-seed: `accounting`, `staff_keuangan` (tidak ada `finance`). **Koreksi audit:** `core/pr_approval.py:75 FINANCE_APPROVER_ROLES` SUDAH memuat `accounting`/`staff_keuangan` → bukan 19 gerbang, melainkan **±13 gerbang di 8 berkas**. |
| T-09 | **VALID** | `start_session`/`with_transaction`: 0 (satu hit `start_session` adalah nama endpoint absensi). `utils/saga` dipakai 5 berkas (payroll). `verify_data_integrity.py` hanya INV-JL-1 (arah baris yatim). Jurnal ditulis 2× (`rahaza_posting.py:222` lalu `:249`). |
| T-10 | **VALID** | `server.py:577` indeks `(source_module, source_ref, status)` **tidak unique**. `_find_existing_je` menyaring `status != voided` (read-then-write). `dewi_kasbon._post_kasbon_gl` memanggil `_create_posted_je` tanpa cek existing sendiri. |
| T-11 | **VALID sebagian (turun ke P2)** | Sumber memang terbelah: `dewi_bank_reconciliation._gl_balance_until` = baris tertanam + filter posted; `gl_balances_by_code` & `fin_statements._sum_by_account` (neraca saldo) = cermin tanpa filter status. **TETAPI** `_void_je_by_source` `delete_many` cermin saat void (`rahaza_posting.py:272`), jadi cermin = posted-only by design. Risiko nyata hanya bila ada jalur void/edit lain yang tidak menghapus cermin → perlu detektor (INV-JL-2), bukan refactor besar. |
| T-12 | **VALID** | `create_index` di `server.py` untuk `vendor_shipment_items`, `buyer_shipments`, `vendor_shipments`, `cmt_receipts`, `wh_positions`, `wh_pending_movements`: **0 semua**. |
| T-13 | **VALID** | Tidak ada `.github/`. Repo hanya 1 commit "Auto-generated changes" (bahkan lebih parah dari 7 di audit). `scripts/gate.sh` ada tetapi manual. |
| T-15 | **VALID** | `frontend/src/**/*.test.*` = **0**; README baris 122 mengklaim 204 uji Jest. 217 berkas Python menembak `localhost:8001`. |
| T-16 | **VALID sebagian** | `deploy/Dockerfile.frontend:12` `yarn install` tanpa `--frozen-lockfile`; `react-beautiful-dnd@13` + React 19. **Catatan:** `frontend/yarn.lock` di repo ini ADA (dipakai bring-up sesi ini) — tinggal `--frozen-lockfile`. |
| T-17 | **VALID** | `hris_cycles/hris_reviews/hris_assignments/hris_kpi_assignments`, `rahaza_qc_events`, `rahaza_attendance`, `capacity_config`, `rahaza_material_reservations`: penulis **0**, pembaca ≥1. `dewi_maklon_inventory`, `invoice_change_history`, `workspace_shares`: ditulis, tak dibaca. `td011_cleanup_orphan_collections.py:32-35` mendaftar `dewi_perf_*` (yang berisi data) sebagai yatim — **berbahaya bila dijalankan**. `fg_cost_consumptions` dibaca hanya oleh test. |
| T-18 | **VALID** | `services/stock_service.py` ada; importir **0**; membaca `rahaza_stock_ledger` & `rahaza_material_reservations` (tanpa penulis). |
| T-19 | **VALID** | `core/collection_registry.py` (472 baris) tidak diimpor siapa pun; `data/collection_registry.py` yang dipakai `admin_backup.py`. |
| T-20 | **VALID** | 3 `bulk_approve_*` identik di 3 berkas `employee_*`, masing-masing dengan `APPROVER_ROLES` sendiri (akar T-08). |
| T-21 | **VALID** | `to_list(None)` di `routes/`: **172** dari 1.656 pemanggilan (audit: 248/1.614). |
| T-22 | **VALID** | `verify_token_str` (token di query string) di **9 berkas** routes; `deploy/Caddyfile:36-37` log akses ke `/data/access.log`. |
| T-23 | **VALID** | `server.py:2435` default `'*'` + `allow_credentials=True`; compose menimpanya. |
| T-24 | **VALID** | 416 `create_index`, 293 `include_router` di `server.py`; `def _now` didefinisikan ulang di 239 berkas. |
| T-25 | **Tidak diverifikasi ulang** (butuh pemetaan FE↔BE penuh; bukan bug, hanya permukaan). Diterima sebagai P2 rawat. |
| T-26 | **VALID** | `mobile/` 36 berkas, **0** panggilan `/api`. |
| T-27 | **Tidak diverifikasi ulang** (butuh AST penuh). Diterima sebagai P3. |
| T-28 | **VALID** | `routes/dewi_kpi.py.old`, `routes/dewi_kpi.py.pre-refactor-backup`; 60 `test_*.py` di akar; 197 skrip; 273 berkas .md; `test_result.md` 484 KB. |

**Kesimpulan:** 24 temuan valid (3 dengan koreksi kecil: T-08 jumlah gerbang, T-11 prioritas, T-16 yarn.lock sudah ada), 2 tidak
diuji ulang (T-25, T-27), 0 gugur. Lampiran B audit (koreksi diri) konsisten dengan kode.

---

## B. PLAN PERBAIKAN (dieksekusi sesi berikutnya)

Prinsip: (1) uang & keamanan dulu, (2) perbaikan kecil ber-dampak besar sebelum refactor, (3) setiap langkah punya
pemeriksaan otomatis agar tidak kambuh (gate). Setiap fase = 1 sesi kerja + testing agent.

### FASE 0 — Tindakan segera di VPS (tanpa kode, hari ini)
| # | Langkah | Cara | Verifikasi |
|---|---|---|---|
| 0.1 | Ganti sandi superadmin produksi (**T-02**) | Login → Profil → ganti sandi; atau `PUT /api/auth/change-password`. | Login dengan `Admin@123` → 401. |
| 0.2 | Pastikan `CORS_ORIGINS` & `ALLOW_DEMO_SEED=false` terset di compose (**T-23**) | Sudah ada di `deploy/docker-compose.yml`; cek `.env` VPS. | `curl -H "Origin: https://evil" …` → tanpa `Access-Control-Allow-Origin`. |

### FASE 1 — P0 uang & data (perubahan kecil, risiko rendah) — target 1 sesi
| # | Temuan | Perubahan | Berkas | Verifikasi |
|---|---|---|---|---|
| 1.1 | **T-04** | `computable = bool(bom_id) and all(l["status"]=="ok" for l in lines)`; `apply_model_cost` menolak size `computable=False` (tidak menulis `hpp_bom`/FG `hpp`/katalog) dan mencatat gap. | `core/product_costing.py:660,851` | Uji unit: BOM 2 baris, 1 `unlinked` → `computable=False`, master tidak berubah. Regresi `test_mrp_cost_stock.py`, `tests/test_iter*hpp*`. |
| 1.2 | **T-05** | Hapus `utils/cascade_delete.py`; `master_data.py` impor `from cascade_delete import cascade_delete_po`. Skrip sekali-jalan: cari AR `status=draft` dengan `linked_maklon_po_id` tanpa PO → laporkan (hapus setelah konfirmasi owner). | `backend/utils/cascade_delete.py`, `routes/master_data.py:21`, `scripts/find_orphan_draft_ar.py` (baru) | Hapus PO via `master_data` → `po_accessories`, `production_variances`, mirror `dewi_maklon_pos`, AR draft ikut bersih. |
| 1.3 | **T-06** | Di `PUT /api/dewi/maklon/pos/{id}`: bila `mirror_of=='production_pos'` → 409 `"PO ini cermin PO produksi — ubah di Portal Produksi"`. Frontend: tampilkan pesan & tombol ke layar PO produksi. | `routes/dewi_maklon_pos.py:557`, UI maklon PO edit | PUT pada mirror → 409; PUT pada PO asli → 200. |
| 1.4 | **T-07** | Sebelum hapus job: cek periode terkunci → 409; `_void_je_by_source(db,"production_job",f"wip_fg_job:{jid}")`; `fg_cost_layers.delete_many({gl_job_id})`; `rahaza_hpp_snapshots.delete_many({job_id})`; `rahaza_wip_events.delete_many({job_id})`. Terapkan juga di `vendor_shipment.py:563`. Ekstrak ke `core/production_job_delete.py` agar 2 jalur pakai 1 fungsi. | `routes/production_execution.py:717`, `routes/vendor_shipment.py:563` | Job selesai (ada JE `wip_fg_job:*`) dihapus → JE `voided`, cermin hilang, layer & snapshot 0; job di periode terkunci → 409. |
| 1.5 | **T-08** | Satu konstanta `core/roles.py: FINANCE_ROLES = ("superadmin","admin","owner","accounting","staff_keuangan","manager_keuangan","finance","accountant")`, `APPROVER_ROLES = FINANCE_ROLES + ("hr","hr_manager","manager")`. Ganti ±13 gerbang di 8 berkas. | `employee_expense_claims`, `employee_travel_requests`, `employee_travel_settlements`, `employee_expense_gl_mapping`, `employee_expense_category_master`, `employee_expense_summary`, `rahaza_ar_360`, `rahaza_channel_gl_mapping` | Login `accounting@…` → approve klaim biaya 200 (bukan 403). Tambah uji RBAC ke `backend_test_f6_rbac.py`. |
| 1.6 | **T-14 (a,b)** | (a) `mature_ap_from_cmt_receipt`: baris `rate==0 & qty_actual>0` → `variance_flagged=True`, `variance_reasons+=["tarif CMT 0 untuk N pcs"]`, tampil di dokumen tagihan. (b) Teks gap `cmt_rate_missing` → "Isi Biaya Jahit per SKU di layar **Biaya Jahit SPK**" dengan `target` layar tsb. | `routes/production_maklon_bridge.py:326-430`, `core/product_costing.py:435` | Penerimaan CMT tanpa tarif → tagihan draft berbendera variance + alasan; gap HPP menunjuk layar yang bisa diisi. |
| 1.7 | **T-09 (1)** | Invarian **INV-JL-2**: setiap JE `posted` punya cermin dengan Σdebit = Σkredit = kepala; laporkan JE tanpa cermin. Tambah ke `verify_data_integrity.py` + `gate.sh`. | `scripts/verify_data_integrity.py` | Jalankan pada seed go-live → 0 pelanggaran (atau daftar untuk diperbaiki). |
| 1.8 | **T-10** | Indeks unique parsial `(source_module, source_ref)` dengan `partialFilterExpression: {status: {$ne: "voided"}}`; migrasi: deteksi duplikat aktif dulu (laporkan, jangan hapus otomatis). `_create_posted_je` tangkap `DuplicateKeyError` → kembalikan JE existing. Kasbon: naikkan `gl.ok=false` ke respons (`ok:false`, 500/409) agar layar tidak "berhasil". | `server.py:577`, `routes/rahaza_posting.py`, `routes/dewi_kasbon.py:44,368,432` | Dua POST paralel sumber sama → 1 JE. Kasbon dengan profil posting hilang → respons error terlihat. |

### FASE 2 — Otorisasi (T-01) — target 1–2 sesi, bertahap
| # | Langkah | Detail |
|---|---|---|
| 2.1 | Inventaris otomatis | Skrip `scripts/audit_authz.py` (AST): daftar endpoint tulis tanpa gerbang peran → CSV per router. Jadi baseline & gate ("angka tidak boleh naik"). |
| 2.2 | Dependensi router-level | `core/authz.py: require_roles(*roles)` + `router = APIRouter(dependencies=[Depends(require_roles(...))])` default **tolak** untuk peran eksternal (`vendor`, `cmt_vendor`, `buyer`, `klien_maklon`, `pic_toko`, `marketing_kol`, `cs_staff`) pada semua router internal. Ini menutup 100% lintas-portal dalam 1 perubahan tanpa menyentuh tiap endpoint. |
| 2.3 | 79 DELETE | Tambah gerbang eksplisit per endpoint (admin/owner/spv domain). |
| 2.4 | SDM & Biaya/HPP | `submit_review` hanya self atau atasan/HR; `delete_hpp` RnD/finance; `portal-accounts` admin. |
| 2.5 | Sisanya per domain | Marketing (sudah punya scope guard, tambah gerbang peran), Gudang, R&D, Workspace, LMS. |
| Verifikasi | Matriks RBAC testing agent: tiap peran × endpoint contoh (200/403). `audit_authz.py` di gate: jumlah tanpa gerbang menurun monoton. |

### FASE 3 — SSOT koleksi hantu (T-03, T-17, T-18, T-19) — target 1 sesi
| # | Langkah |
|---|---|
| 3.1 | Gate baru `scripts/check_collection_writers.py`: koleksi yang dibaca kode wajib punya ≥1 penulis (kecuali daftar putih: dibaca hanya untuk migrasi). Pasang di `gate.sh`. |
| 3.2 | **T-03**: pindahkan 23 pembaca `rahaza_work_orders` → `production_jobs` dengan peta field (`quantity→qty`, `qty_completed→completed_qty`, `order_code→job_number`, status). Buat `core/wo_reader.py` (satu adaptor) supaya 23 berkas hanya ganti impor. Dampak: dasbor produksi, laporan eksekutif, kapasitas, notifikasi, next-actions, HPP per WO, scan barcode, AI. |
| 3.3 | **T-17 HRIS**: Portal Saya → Kinerja baca `dewi_perf_*` (bukan `hris_*`); **hapus baris `dewi_perf_*` dari `td011_cleanup_orphan_collections.py`** (mencegah penghapusan data nyata). |
| 3.4 | **T-17 lain**: `rahaza_attendance` → `rahaza_attendance_events` (AI HR); `rahaza_qc_events` → sumber QC nyata (`cmt_receipts`/`production_progress`) atau hapus metrik; `capacity_config` → endpoint GET/PUT + layar kecil, atau hapus pembacanya; koleksi ditulis-tak-dibaca (`fg_cost_consumptions`, `invoice_change_history`, `dewi_maklon_inventory`, `workspace_shares`, `wh_rca_audit`, dst): putuskan tampilkan (read endpoint + tab "Riwayat") atau berhenti menulis. |
| 3.5 | **T-18** hapus `services/stock_service.py`; **T-19** hapus `core/collection_registry.py` atau jadikan ia yang dipakai `admin_backup.py` (pilih satu). |

### FASE 4 — Ketahanan & kinerja (T-12, T-09 (2,3), T-11, T-21, T-24) — target 1 sesi
| # | Langkah |
|---|---|
| 4.1 | **T-12** indeks: `vendor_shipment_items(shipment_id)`, `(po_item_id)`; `buyer_shipments(po_id)`; `vendor_shipments(po_id)`; `cmt_receipts(status)`, `(po_id)`; `wh_positions(rack_id)`, `(status)`, `(barcode)`; `wh_pending_movements(type,status)`, `(source_type,source_id)`; plus `active` pada `rahaza_boms`, `rahaza_employees`, `rahaza_locations`, `rahaza_leave_types`. |
| 4.2 | **T-24** pindahkan 416 `create_index` dari boot ke `migrations/ensure_indexes.py` (dipanggil sekali saat deploy + opsional saat boot via env `ENSURE_INDEXES=1`). |
| 4.3 | **T-09 (2)** balik urutan tulis JE: cermin dulu, kepala terakhir (kegagalan tengah → baris yatim yang sudah dideteksi INV-JL-1). **(3)** Opsional: Mongo replica set 1 node di compose (`--replSet rs0`) → transaksi untuk `_create_posted_je`. |
| 4.4 | **T-11** samakan sumber: `_gl_balance_until` pakai cermin (`rahaza_journal_lines`) — konsisten dengan neraca saldo; tambah uji "saldo rekonsiliasi == saldo kas&bank == neraca saldo" untuk 1 akun. |
| 4.5 | **T-21** `to_list(None)` di jalur terpanas (laporan keuangan, rekap marketing, ekspor): paginasi/`limit` + agregasi di DB; pakai `core/pagination.py`. |
| 4.6 | **T-23** tanpa default `'*'`: bila `ENV=production` dan `CORS_ORIGINS` kosong → gagal boot dengan pesan jelas. **T-02 (kode)** `seed_initial_data`: di production baca `BOOTSTRAP_ADMIN_EMAIL/PASSWORD`, tolak boot bila kosong & belum ada superadmin; hapus baris kredensial dari `README_DEPLOY.md`. |
| 4.7 | **T-22** token unduhan sekali-pakai 5 menit (`POST /api/auth/download-token?resource=`) untuk label/PDF/ekspor/WebSocket; sementara: Caddy `log { format filter { request>uri query delete } }`. |

### FASE 5 — Disiplin rekayasa (T-13, T-15, T-16, T-20, T-28) — target 1 sesi
| # | Langkah |
|---|---|
| 5.1 | **T-13/T-16** `.github/workflows/ci.yml`: `ruff check backend`, `eslint frontend/src`, `pytest tests/unit` (hermetik), `yarn install --frozen-lockfile && yarn build`; `Dockerfile.frontend` → `--frozen-lockfile`. Commit per perubahan bermakna. |
| 5.2 | **T-15** folder `backend/tests/unit/` dengan `mongomock_motor` untuk `core/*` (mulai: `product_costing`, `bom_fill`, `uom`, `production_qty_ledger`, `catalog_stock`, `marketing_returns` — 6 alur yang sudah dibuktikan audit). Jest: kembalikan ≥1 suite (smoke render) atau hapus klaim 204 uji dari README. |
| 5.3 | **T-20** `core/bulk_approve.py: bulk_approve(db, user, ids, collection, number_field, module, allowed_roles)`; 3 pemanggil jadi 3 baris. |
| 5.4 | **T-14 (c)** kolom `tarif_jahit_per_pcs` di sheet `11_VENDOR_CMT` (`TEMPLATE_MASTER_DA.xlsx` + `CONTOH_TERISI`) → `dewi_cmt_partners.rate_per_pcs`; tampilkan sebagai pembanding di layar Biaya Jahit SPK. |
| 5.5 | **T-28** pindahkan `test_*.py`/`backend_test*.py` akar → `tests/legacy/`; hapus `dewi_kpi.py.old`, `.pre-refactor-backup`; arsipkan blok status README ke `docs/CHANGELOG_SESI.md`; `test_result.md` → `docs/archive/`. **T-26** hapus `mobile/` atau beri README "belum dimulai". |

### Backlog rawat (tanpa jadwal): T-25 (307 endpoint tanpa pemanggil — matikan bertahap dengan 410), T-27 (`{items,total}` seragam + `asList()` di FE).

---

## C. URUTAN EKSEKUSI YANG DISARANKAN
1. **Fase 0** hari ini (owner, 10 menit).
2. **Fase 1** sesi berikutnya — 8 perbaikan kecil, semuanya bisa diuji unit + testing agent dalam 1 sesi; ini menutup semua P0 uang/data kecuali T-01/T-03.
3. **Fase 2** (T-01) — mulai dari langkah 2.1–2.2 (tolak peran eksternal di level router: dampak terbesar, perubahan terkecil), lalu DELETE.
4. **Fase 3** (koleksi hantu) — memulihkan dasbor/laporan yang selama ini nol.
5. **Fase 4–5** — ketahanan & disiplin.

## D. DEFINISI SELESAI per fase
- Semua perubahan lolos `bash scripts/gate.sh` + suite `pytest` terkait + testing agent (backend & UI untuk yang menyentuh layar).
- Gate baru (INV-JL-2, `check_collection_writers.py`, `audit_authz.py`) dipasang di `gate.sh` supaya kelas cacat tidak kambuh.
- `memory/PRD.md` diperbarui; DB uji dikembalikan ke `seed/DA_SEED_GOLIVE.archive.gz` setelah uji yang mengubah data.
