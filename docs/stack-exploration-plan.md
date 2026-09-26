# Rencana eksplorasi teknologi

Pilih cakupan kerja mulai dari alur lokal paling sederhana hingga eksperimen layanan dan peningkatan skala. Setiap tahap merupakan usulan pekerjaan yang menggunakan ketentuan bukti serta kasus uji pembanding yang sama.

## Ketentuan eksplorasi

- Mulai dengan pemrosesan kelompok data secara lokal, lalu tambahkan satu kemampuan dan ukur pengaruhnya.
- Simpan rekaman mentah, pengenal peristiwa dan bukti yang tetap, waktu peristiwa, lokasi sumber, serta ringkasan eksekusi sejak awal.
- Pertahankan pembanding berdasarkan tingkat keparahan, evaluasi berdasarkan waktu, dan analisis kegagalan saat membandingkan model triase.
- Kecualikan label evaluasi dan jawaban serangan terdokumentasi dari masukan inferensi maupun pencarian konteks.
- Hitung statistik fitur yang dipelajari menggunakan data pelatihan; gunakan hanya konteks yang tersedia pada waktu evaluasi peringatan.
- Catat upaya penyiapan dan penelusuran kesalahan, durasi setiap tahap, penggunaan memori puncak, penggunaan penyimpanan, serta langkah pemulihan manual.
- Tetapkan versi dependensi dan catat identitas kumpulan data, aturan, model, serta konfigurasi agar hasil dapat direproduksi.
- Berhenti pada tahap yang memenuhi pertanyaan utama dan tujuan pembelajaran; seluruh tahapan melampaui cakupan awal enam hari.

## Tahap 1: Pemrosesan kelompok data secara lokal

| Tanggung jawab | Pilihan | Hasil yang diharapkan |
|---|---|---|
| Pengumpulan dan normalisasi | Python dengan pemetaan atribut eksplisit | Peristiwa publik menjadi rekaman yang dinormalisasi dan memiliki rujukan sumber. |
| Penyimpanan dan kueri | Berkas Parquet yang dicari melalui DuckDB | SQL mengambil peristiwa berdasarkan perangkat, identitas proses, dan waktu. |
| Deteksi | Sebagian kecil logika aturan publik yang diadaptasi ke SQL/Python dan didokumentasikan | Peringatan mempertahankan pengenal aturan dan peristiwa sumber. |
| Fitur dan penilaian | pandas, scikit-learn, LightGBM | Tingkat keparahan, regresi logistik, dan model berbasis pohon dievaluasi pada pembagian waktu yang sama. |
| Eksekusi | Perintah CLI atau target Make | Setiap tahap membaca masukan tersimpan dan menulis keluaran yang dapat diperiksa. |
| Penyaluran dan kasus | Fungsi kebijakan Python dan catatan JSON | Simulasi keputusan penutupan, pemeriksaan, atau eskalasi menyimpan alasannya. |
| Observabilitas | Pemeriksaan Python, log JSON, ringkasan eksekusi | Jumlah rekaman dapat dicocokkan; rekaman tidak valid dan rujukan yang hilang terlihat. |

- Adaptasi aturan secara manual harus menjelaskan kondisi yang didukung, pemetaan atribut, dan keterbatasannya. Tahap ini belum menunjukkan dukungan terhadap semua aturan Sigma.
- Periksa perilaku aturan yang diadaptasi menggunakan contoh yang seharusnya cocok dan tidak cocok sebelum memakai peringatannya dalam evaluasi.
- Catat hasil pengumpulan dan penilaian tanpa mewajibkan antrean, server pencarian, API, atau penjadwal.
- Kriteria selesai: masukan yang sama dapat dijalankan ulang dan peringatan, label, fitur, metrik, serta catatan kegagalannya dapat diperiksa.

## Tahap 2: Ringkasan dari LLM lokal

| Penambahan | Pilihan awal | Alternatif untuk dipelajari | Biaya tambahan |
|---|---|---|---|
| Pembangkitan respons lokal | Ollama dengan model yang diunduh secara lokal | llama.cpp jika pengendalian lingkungan eksekusi dan kuantisasi secara langsung menjadi tujuan pembelajaran | Penyimpanan model, memori, dan latensi inferensi. |
| Pemilihan konteks | Pencarian DuckDB berdasarkan kecocokan tepat | Aturan pencarian tambahan yang didokumentasikan | Pemeliharaan kueri dan pemilihan bukti. |
| Validasi ringkasan | Skema respons terstruktur dan pemeriksaan bukti | Pemeriksaan manual terhadap lembar jawaban untuk klaim kompleks | Logika validasi dan pemeliharaan uji pembanding. |

- Pilih model setelah mengukur memori yang tersedia dan waktu respons yang dapat diterima; catat identitas serta pengaturannya secara tepat.
- Jalankan model secara lokal tanpa pengalihan diam-diam ke layanan komputasi awan. Tempatkan lingkungan eksekusi model di luar kontainer jika lebih sesuai dengan akselerasi perangkat keras lokal.
- Simpan pengamatan, hipotesis, informasi yang belum tersedia, pengenal bukti, instruksi masukan, dan respons lengkap.
- Kriteria selesai: peringatan terpilih menghasilkan ringkasan yang dapat diperiksa terhadap bukti asli dan fakta yang sudah disiapkan sebagai jawaban acuan.
- Bandingkan ringkasan yang dihasilkan dengan templat laporan tetap untuk mengukur manfaat pembangkitan respons.

## Tahap 3: Konteks berbasis graf

| Penambahan | Pilihan awal | Alternatif untuk dipelajari | Biaya tambahan |
|---|---|---|---|
| Pencarian hubungan | Semantica dengan hubungan peristiwa yang eksplisit | Penggabungan tabel melalui SQL sebagai pembanding | Perancangan skema, validasi graf, dan integrasi. |
| Pencatatan asal-usul data | Catatan lokal yang disimpan secara persisten dan terhubung ke bukti mentah | Catatan JSON yang dapat diperiksa untuk uji pembanding kecil | Penyimpanan dan pemeliharaan tautan sumber. |
| Pencarian teks | Pencarian awal berdasarkan sumber dan entitas yang tepat | Representasi vektor lokal dan pencarian vektor jika panduan sulit ditemukan | Pemrosesan representasi vektor, penyimpanan indeks, dan penyetelan pencarian. |

- Tangani temuan dalam spesifikasi utama sebelum menggunakan kembali pola dari arsip buku catatan komputasi.
- Bedakan hubungan yang diamati dari hubungan yang disimpulkan; gunakan identitas yang tepat untuk menghubungkan peristiwa terstruktur.
- Mulai dengan representasi graf lokal dan hasil ekspor tersimpan. Tambahkan server graf khusus hanya untuk kebutuhan penyimpanan persisten, akses serentak, atau kueri yang nyata.
- Kriteria selesai: kasus dan konfigurasi LLM yang sama menunjukkan apakah pencarian berbasis graf meningkatkan penemuan bukti yang diharapkan serta klaim yang didukung dibandingkan pencarian SQL.
- Akses melalui layanan dapat dipelajari secara terpisah jika graf tidak memberikan manfaat terukur.

## Tahap 4: Layanan pencarian dan dasbor

| Urutan penambahan | Pilihan teknologi | Hasil yang diharapkan |
|---|---|---|
| Pencarian | OpenSearch + Dashboards | Peristiwa mentah dan hasil normalisasi dapat dicari; bukti asli tetap dapat diakses. |
| Layanan pengumpulan | Vector langsung ke OpenSearch | Pemetaan atribut dan jumlah rekaman yang diterima atau ditolak sesuai dengan pembanding lokal. |
| Konversi aturan | Sigma CLI/pySigma dengan modul konversi dan alur pemrosesan yang sudah diverifikasi kompatibilitasnya | Aturan terpilih berjalan sesuai dengan pemetaan indeks yang digunakan. |
| API | FastAPI untuk permintaan penilaian dan ringkasan | Setiap permintaan memiliki pengenal peringatan, bukti, dan eksekusi yang saling terhubung. |
| Validasi data | GX Core beserta pemeriksaan khusus yang sudah tersedia | Laporan validasi menyimpan pengenal terdampak dan kebijakan kegagalan yang eksplisit. |
| Pemantauan | Prometheus + Grafana; catatan JSON di OpenSearch | Kegagalan dan durasi terhubung ke catatan eksekusi terkait. |
| Pengemasan | Docker Compose untuk layanan terpilih | Konfigurasi layanan dapat dijalankan secara konsisten. |

- Tambahkan setiap komponen secara terpisah dan pertahankan alur lokal sebagai pembanding.
- Verifikasi keluaran modul konversi Sigma dan kompatibilitas pemetaan atribut; keberhasilan konversi saja belum membuktikan kueri dapat berjalan di OpenSearch.
- Simpan berkas asli dan ringkasan eksekusi lokal agar gangguan layanan tidak menghilangkan bukti uji pembanding.
- Ikuti [rencana observabilitas](observability-plan.md) untuk pemeriksaan, penanganan metrik tugas yang memproses kelompok data, dan skenario kegagalan.

## Tahap 5: Eksperimen lanjutan yang dapat dipilih secara terpisah

Pilih setiap opsi setelah pembanding sederhana yang relevan sudah berfungsi.

| Kemampuan yang ingin ditunjukkan | Kandidat | Pembanding sederhana | Bukti untuk mempertahankan kandidat |
|---|---|---|---|
| Tugas terjadwal yang saling bergantung, percobaan ulang, dan pemrosesan data periode lampau | Airflow | CLI/Make dengan status eksekusi eksplisit | Tahap yang gagal dan interval lampau dapat diproses ulang tanpa menggandakan hasil. |
| Penampungan persisten, pemrosesan ulang, atau konsumen terpisah | Redpanda atau Apache Kafka | Pengumpulan langsung dan berkas sumber tersimpan | Pemulihan mempertahankan pencatatan jumlah rekaman; posisi pembacaan dan perilaku penghapusan duplikat didokumentasikan. |
| Transformasi kelompok data yang lebih besar | PySpark secara lokal, lalu terdistribusi jika beralasan | pandas/DuckDB | Keluaran sebanding dan perbandingan waktu serta memori terukur, termasuk biaya memulai proses. |
| Permintaan yang melintasi layanan | OpenTelemetry + Tempo | Log terstruktur yang saling terhubung | Lokasi permintaan yang lambat atau gagal dapat ditemukan dalam jejak layanan lengkap. |
| Integrasi alur penanganan kasus | TheHive | Catatan kasus JSON | Bukti eskalasi dan siklus hidup kasus dapat ditelusuri melalui API. |

- Redpanda menyediakan kompatibilitas dengan API Kafka; dokumentasikan perantara pesan yang dipilih secara tepat. Pilih Apache Kafka jika pengoperasian Kafka sendiri menjadi tujuan pembelajaran.
- Spark lokal menunjukkan penggunaan API dan model eksekusinya; laporkan peningkatan skala terdistribusi hanya jika diukur pada konfigurasi terdistribusi yang sesungguhnya.
- Pisahkan tanggung jawab penjadwalan dari respons peringatan; penjadwal tugas tidak menentukan kebijakan triase.
- Setiap eksperimen yang dipertahankan harus memiliki manfaat yang diamati atau hasil pembelajaran yang dinyatakan beserta biaya operasinya.

## Dokumentasi rujukan

- [API Python DuckDB](https://duckdb.org/docs/stable/clients/python/overview): kueri SQL lokal terhadap berkas data dan DataFrame.
- [Panduan memulai Ollama](https://docs.ollama.com/quickstart): eksekusi model lokal dan akses API.
- [llama.cpp](https://github.com/ggml-org/llama.cpp): alternatif lingkungan eksekusi model lokal.
- [Modul konversi Sigma](https://sigmahq.io/docs/digging-deeper/backends): konversi kueri dan pemilihan modul.
- [Pencatatan asal-usul data Semantica](https://github.com/semantica-agi/semantica/blob/main/docs/guides/provenance.md): catatan sumber dan proses penurunan data.
- [Sumber rujukan observabilitas](observability-plan.md#sumber-rujukan): dokumentasi validasi, metrik, dasbor, dan penelusuran.
