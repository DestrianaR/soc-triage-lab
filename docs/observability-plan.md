# Rencana observabilitas data dan proses

Usulan pekerjaan untuk alur triase SOC dan penambahan LLM lokal. Mulai dengan pemeriksaan lokal, log JSON, dan ringkasan eksekusi yang disimpan secara persisten; layanan berikut digunakan pada tahap layanan pencarian dalam [rencana eksplorasi teknologi](stack-exploration-plan.md).

## Pilihan teknologi dan tanggung jawab

| Bidang | Pilihan awal | Tujuan |
|---|---|---|
| Validasi data | Great Expectations Core dan pemeriksaan Python khusus | Memvalidasi tabel, simpul pada ujung hubungan graf, serta rujukan bukti; menyimpan hasil pemeriksaan. |
| Metrik operasional | Prometheus | Mencatat durasi tahap, kegagalan, rekaman yang diproses, dan waktu penyelesaian berhasil terakhir. |
| Dasbor | Grafana | Menampilkan metrik bersama log dan catatan validasi. |
| Investigasi eksekusi | Log dan catatan JSON terstruktur di OpenSearch | Mencari kegagalan serta memeriksa eksekusi, peringatan, dan bukti yang terdampak. |
| Penelusuran pada tahap lanjutan | OpenTelemetry dan Tempo | Mengikuti pencarian, penilaian, dan pembangkitan respons lintas layanan. |

- Jalankan pemeriksaan GX di dalam tugas pemrosesan; gunakan pemeriksaan khusus untuk ketentuan graf dan bukti yang tidak tercakup dalam pemeriksaan tabel.
- Gunakan OpenSearch untuk log aplikasi dan catatan validasi pada indeks yang terpisah dari peristiwa keamanan sumber.
- Gunakan sumber data OpenSearch pada Grafana untuk mengakses catatan tersebut bersama metrik Prometheus.
- Tambahkan penelusuran ketika panggilan lintas layanan sulit diikuti melalui log yang saling terhubung.

## Pemeriksaan data

| Tahap | Pemeriksaan | Kebijakan awal saat gagal |
|---|---|---|
| Pengumpulan | Masukan yang diharapkan tersedia; jumlah rekaman masukan, diterima, ditolak, dan sengaja dihapus sebagai duplikat dapat dicocokkan. | Tandai eksekusi tidak lengkap ketika masukan wajib tidak tersedia atau jumlah tidak cocok; catat rekaman yang ditolak. |
| Normalisasi | Atribut wajib tersedia; waktu dapat diuraikan; pengenal peristiwa memenuhi ketentuan keunikan. | Karantina rekaman tidak valid dan laporkan penurunan cakupan. |
| Pembentukan graf | Pengenal entitas unik; jenis entitas diizinkan; kedua simpul hubungan tersedia; rujukan bukti dapat ditemukan. | Kecualikan hubungan tidak valid dari pencarian dan tandai validasi graf gagal. |
| Pencarian konteks | Bukti memenuhi batas waktu evaluasi; sumber yang diharapkan tersedia; pemotongan konteks dicatat. | Cegah penggunaan bukti dari masa depan; tandai konteks yang hilang dan penurunan cakupan. |
| Keluaran LLM | Struktur respons valid; pengenal yang dirujuk termasuk dalam masukan; atribut yang dirujuk sesuai dengan nilai sumber. | Tandai klaim faktual tanpa dukungan; wajibkan hasil valid sebelum digunakan pada tahap berikutnya. |

- Untuk telemetri historis, ukur kemajuan pengumpulan atau pemrosesan ulang terhadap rentang masukan yang diharapkan. Waktu peristiwa yang lama saja tidak menunjukkan keterlambatan pengumpulan.
- Rujukan valid menunjukkan keterlacakan; dukungan faktual bagi penafsiran kompleks tetap perlu dibandingkan dengan lembar jawaban uji pembanding.
- Nyatakan hipotesis dan informasi yang belum tersedia secara eksplisit dalam ringkasan.
- Laporkan rekaman yang diterima dan ditolak secara bersamaan agar pemrosesan sebagian tidak terlihat sebagai keberhasilan penuh.

## Pengukuran proses

| Pengukuran | Hal yang dapat dijelaskan |
|---|---|
| Waktu penyelesaian berhasil terakhir, status eksekusi, dan sinyal keaktifan jika relevan | Pemrosesan yang gagal, tidak lengkap, atau terhenti. |
| Durasi, kegagalan, dan percobaan ulang setiap tahap | Tahap yang lambat dan kesalahan berulang. |
| Rekaman yang diproses, ditolak, dan menunggu | Laju pemrosesan, kehilangan data, dan penumpukan antrean. |
| Kegagalan validasi berdasarkan tahap dan jenis pemeriksaan | Ketentuan data yang dilanggar. |
| Durasi permintaan LLM, pelampauan batas waktu, dan respons tidak valid | Keandalan pembangkitan respons dan biaya pemrosesan. |
| Penggunaan CPU, memori, penyimpanan, serta memori GPU jika tersedia | Beban sumber daya pada komputer lokal. |

- Sediakan metrik layanan yang terus berjalan melalui titik akses pengambilan metrik.
- Untuk tugas singkat yang memproses kelompok data, simpan ringkasan penyelesaian dan sediakan melalui titik akses persisten atau Pushgateway yang dikonfigurasi. Tentukan pembersihan metrik kedaluwarsa jika menggunakan Pushgateway.
- Bedakan eksekusi yang selesai tetapi gagal dalam pemeriksaan dari eksekusi yang sepenuhnya berhasil.
- Tetapkan ambang waktu dan sumber daya berdasarkan hasil pengukuran awal serta jadwal yang direncanakan.

## Pengaitan catatan dan retensi bukti

| Atribut | Penggunaan |
|---|---|
| `run_id` | Setiap log tahap, hasil validasi, dan ringkasan eksekusi. |
| `alert_id` | Catatan pencarian, penilaian, dan pembangkitan respons untuk peringatan tertentu. |
| `evidence_id` | Rujukan sumber, hubungan graf, dan pemeriksaan klaim. |
| `trace_id`, ketika penelusuran aktif | Rentang penelusuran dan log untuk operasi lintas layanan yang sama. |

- Gunakan label Prometheus dengan himpunan nilai terbatas, seperti tahap, hasil, dan nama pemeriksaan. Simpan pengenal individual eksekusi, peringatan, dan bukti di dalam log serta catatan.
- Simpan salinan sumber, instruksi masukan, paket bukti, pengaturan model, dan respons asli untuk eksekusi uji pembanding; tautkan semuanya dari catatan eksekusi yang dapat dicari.
- Tentukan masa retensi terpisah untuk log operasional dan bukti uji pembanding. Penghapusan data pemantauan yang kedaluwarsa tidak boleh memutus rujukan bukti yang masih disimpan tanpa pemberitahuan.
- Pertahankan ringkasan eksekusi lokal ketika layanan penyimpanan pemantauan tidak tersedia agar dapat diperiksa atau diindeks kemudian.

## Urutan implementasi dan kriteria penerimaan

1. Tambahkan pengenal eksekusi bersama, catatan awal dan akhir setiap tahap, serta ringkasan eksekusi persisten. Selesai ketika satu eksekusi dapat diikuti dari pengumpulan hingga keluaran.
2. Tambahkan pemeriksaan data dan kebijakan kegagalan eksplisit. Selesai ketika rekaman tidak valid, pengenal duplikat, dan simpul hubungan yang hilang dilaporkan bersama pengenal terdampak.
3. Sediakan metrik dan buat dasbor Grafana. Selesai ketika kegagalan pemeriksaan dan proses terhubung ke catatan eksekusi yang dapat dicari.
4. Uji rujukan bukti yang sengaja dibuat tidak tersedia. Selesai ketika dasbor menunjukkan pemeriksaan yang gagal dan catatan eksekusi menunjukkan hubungan, pengenal bukti yang hilang, serta tahap terkait; hubungan tidak valid dikecualikan dari pencarian.
5. Uji tahap yang gagal atau melampaui batas waktu. Selesai ketika eksekusi ditandai gagal atau tidak lengkap, waktu keberhasilan terakhir tidak berubah, dan log menjelaskan kegagalannya.
6. Tambahkan OpenTelemetry dan Tempo ketika dibutuhkan. Selesai ketika peringatan terpilih dapat diikuti lintas layanan dengan log yang saling terhubung.

## Sumber rujukan

- [Ikhtisar GX Core](https://docs.greatexpectations.io/docs/core/introduction/gx_overview/): aturan validasi, hasil validasi, dan Data Docs.
- [Instrumentasi Prometheus](https://prometheus.io/docs/practices/instrumentation/): pengukuran layanan dan tugas pemrosesan kelompok data.
- [Integrasi OpenSearch pada Grafana](https://grafana.com/docs/plugins/grafana-opensearch-datasource/latest/configure/): pencarian catatan tersimpan melalui Grafana.
- [Dokumentasi OpenTelemetry](https://opentelemetry.io/docs/): instrumentasi dan pengiriman telemetri.
- [Dokumentasi Tempo](https://grafana.com/docs/tempo/latest/): penyimpanan dan pencarian jejak eksekusi.
