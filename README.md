# Final Project VISKOM — Kelompok 2-2

Object detection klasik pada BDD100K dengan **Selective Search + HOG + Linear SVM**.

## Status workspace

Workspace ini menyiapkan struktur kerja tim. Dataset, model, dan metrik hasil belum tersedia karena harus diperoleh melalui eksperimen. Setiap hasil eksperimen dicatat di `reports/experiment_log.md`.

## Isi folder

```text
configs/       parameter eksperimen yang disepakati
data/raw/      dataset BDD100K hasil unduhan (tidak di-commit)
data/interim/  manifest sampel dan crop kandidat
data/processed/ HOG feature siap dipakai untuk training
src/           script pipeline
models/        Linear SVM tersimpan
outputs/       prediction, evaluation, dan visualization
reports/       log eksperimen dan bahan angka untuk PPT
docs/          panduan teknis dan checklist
```

## Baseline awal

- Gunakan satu class terlebih dahulu: `car`.
- Ambil 600 image dari `train` dan 150 image dari `val` dengan `random_seed: 42`.
- `train` dipakai untuk training, sedangkan `val` dipakai untuk evaluation. Folder `test` jangan dipakai untuk menghitung mAP karena tidak punya ground truth.
- Setiap region proposal di-resize ke 96×96 sebelum HOG feature extraction.
- Gunakan Selective Search mode `fast`, lalu Linear SVM dan NMS.

Parameter tersebut merupakan baseline awal. Setiap perubahan parameter harus dicatat beserta alasannya pada experiment log agar hasil dapat ditelusuri.

## Urutan kerja singkat

1. Baca [panduan tim](docs/GUIDE_TEAM_2_2.md) dan tetapkan pembagian tugas.
2. Unduh BDD100K, kemudian ekstrak ke `data/raw/bdd100k/`.
3. Buat Python environment dan instal dependency dari `requirements.txt`.
4. Jalankan pipeline secara berurutan: parsing annotation, sampling, pembuatan training crop, HOG feature extraction, SVM training, prediction, dan evaluation.
5. Catat setiap eksperimen di `reports/experiment_log.md`. Simpan contoh prediction yang bagus maupun yang salah di `reports/figures/`.
6. Setelah selesai, masukkan source code dan PPT ke Google Drive kelompok. Link Drive kemudian ditempel pada sheet pembagian kelompok.

Dokumen utama: [panduan kerja Team 2-2](docs/GUIDE_TEAM_2_2.md).
