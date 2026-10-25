# DIAMIS: klasifikasi penjelasan tertulis siswa untuk diagnosis miskonsepsi matematika

Kode dan hasil agregat untuk makalah DIMAS-TI 2026, Divisi Penambangan Data.
Penulis: Mahdi Habibi, Gema Satya Danera, Nabilah Aulia Zahra (Teknik Informatika, Universitas Negeri Semarang).

*English: code and aggregate results for a study of text classification on student explanations of math answers, comparing random splits with per-question splits on the MAP / MIME data.*

Repositori ini berisi satu notebook dan hasil agregatnya. **Data kompetisi tidak disertakan**, jadi kamu perlu mengunduhnya sendiri (lihat bagian Data).

## Data

Data berasal dari kompetisi Kaggle [MAP - Charting Student Math Misunderstandings](https://www.kaggle.com/competitions/map-charting-student-math-misunderstandings), yang bersumber dari dataset MIME (Rittle-Johnson dkk., 2025) dengan data dari Eedi. Aturan kompetisi menetapkan lisensi CC BY-NC 4.0 dan membatasi pemakaian pada tujuan nonkomersial, termasuk riset akademik dan pendidikan. Aturan yang sama melarang menyebarkan data ke pihak di luar kompetisi, sehingga berkas data dan turunannya (misalnya terjemahan teks siswa) tidak ada di repositori ini.

1. Masuk ke halaman kompetisi, setujui aturannya, lalu unduh `train.csv` (dan `test.csv` bila ingin membuat berkas submission contoh).
2. Taruh keduanya di folder yang sama dengan `dimas-ti.ipynb`. Notebook membaca `train.csv` dari direktori kerja saat ini.

## Menjalankan

Eksperimen dijalankan di CPU, tanpa GPU: Intel Core i5-5300U 2,30 GHz, 4 inti logis, RAM 8 GB.

```
pip install numpy pandas scipy scikit-learn matplotlib jupyter torch transformers sentencepiece sacremoses
```

Jalankan sel dari atas ke bawah. Sel 1 mencoba meng-clone repositori MIME untuk audit berkas dan dilewati otomatis bila tidak ada internet. Hasil ditulis ke folder `output/`.

| Sel | Isi |
| --- | --- |
| 0 | Membaca `train.csv`, menentukan path dan folder `output/` |
| 1 | Audit berkas repositori MIME (opsional, perlu internet) |
| 2–3 | Fungsi baca tabel dan normalisasi teks |
| 4 | Pembentukan target dan fitur (TF-IDF kata dan karakter, one-hot identitas soal), regresi logistik 5 lipatan |
| 5 | Variasi konfigurasi regresi logistik E1–E3, dan pembuatan berkas submission contoh |
| 6 | LinearSVC, 5 lipatan |
| 7 | Terjemahan teks Inggris ke Indonesia dengan `Helsinki-NLP/opus-mt-en-id` (hasilnya disimpan sebagai cache lokal) |
| 8 | Fine-tuning `cahya/distilbert-base-indonesian` pada teks terjemahan, satu lipatan |
| 9 | Pembuatan gambar untuk makalah |

Pembagian data memakai `StratifiedGroupKFold` (5 lipatan, seed 42) dengan teks penjelasan ternormalisasi sebagai kelompok.

### Dua konfigurasi transformer

Variabel `MODE_CEPAT` di sel 8 memilih konfigurasi. Mode cepat dipakai sebagai uji awal, mode penuh untuk hasil yang dilaporkan.

| | Mode cepat | Mode penuh |
| --- | --- | --- |
| Sampel baris | 2.000 | 10.000 |
| Panjang maksimum token | 128 | 160 |
| Epoch | 1 | 3 |
| Ukuran batch latih | 32 | 16 |

Parameter lain sama pada keduanya: AdamW, laju belajar 5e-5, pemanasan 6%, peluruhan bobot 0,01, batch evaluasi 64, sampel acak dengan seed 42. Seed PyTorch tidak ditetapkan dan tidak ada replikasi seed.

## Berkas di `output/`

| Berkas | Isi |
| --- | --- |
| `DIMAS_TI_MIME_Audit_CV_results.csv` | MAP@3, akurasi top-1, dan Hit@3 per lipatan, regresi logistik konfigurasi lanjutan (5 lipatan) |
| `DIMAS_TI_LR_CV_fold_results.csv`, `DIMAS_TI_LR_CV_summary.csv` | Hasil per lipatan dan ringkasan eksperimen E1–E3 (3 lipatan) |
| `DIMAS_TI_Transformer_Results.csv` | Hasil fine-tuning transformer pada satu lipatan |
| `DIMAS_TI_submission.csv` | Contoh berkas submission berformat Kaggle (tiga baris dari `test.csv` resmi) |

<!-- TODO sebelum dikumpulkan: tambahkan skrip untuk protokol per soal (Tabel 4.1–4.2, uji Wilcoxon, matriks konfusi) dan simpan hasil LinearSVC ke CSV, lalu tambahkan barisnya ke tabel di atas. -->

## Keterbatasan yang perlu diketahui pembaca

- Model transformer dilatih pada teks hasil terjemahan mesin, bukan data berbahasa Indonesia asli, dan hanya dievaluasi pada satu lipatan.
- Transformer dan model klasik tidak dievaluasi pada himpunan uji yang sama.
- Eksperimen berjalan di satu mesin CPU tanpa replikasi seed.
- Skor pembagian acak tidak mewakili soal baru; uraiannya ada di makalah.

## Sitasi dan atribusi

- Rittle-Johnson, B., Adler, R., Durkin, K., Burleigh, L., King, J., dan Crossley, S. A. (2025). *Detecting math misconceptions: An AI benchmark dataset.* AIME-Con.
- Kompetisi MAP - Charting Student Math Misunderstandings (Vanderbilt University, The Learning Agency, Kaggle; data dari Eedi), lisensi data CC BY-NC 4.0.

<!-- TODO: pilih lisensi kode (mis. MIT), tambahkan berkas LICENSE, lalu tulis satu baris lisensinya di sini. -->
