# Panduan Kerja Final Project VISKOM — Kelompok 2-2

Kelompok 2-2 menggunakan **Selective Search + HOG + Linear SVM** pada dataset BDD100K. Luaran sistem harus berupa **bounding box**, bukan hanya hasil image classification.

## Istilah yang dipakai

| Term | Arti |
|---|---|
| **Region proposal** | Candidate bounding box. Selective Search membuat banyak kotak kandidat yang mungkin berisi object. |
| **Ground truth** | Bounding box dan class asli dari annotation BDD100K. |
| **Positive sample** | Crop yang memang berisi target object, misalnya car. |
| **Negative sample** | Crop background yang tidak berisi target object. |
| **Hard negative** | Background yang keliru dianggap object oleh model. Sample ini dipakai lagi saat training agar model membaik. |
| **Inference** | Saat model dipakai untuk menghasilkan detection dari sebuah image. |
| **NMS** | Non-Maximum Suppression. Dipakai untuk membuang duplicate bounding box pada object yang sama. |
| **IoU** | Intersection over Union. Nilai untuk melihat seberapa besar overlap dua bounding box. |

## Gambaran alur

```text
image → Selective Search → region proposal → HOG → Linear SVM → score threshold → NMS → final bounding box
```

Selective Search umumnya menghasilkan sekitar 500–1.500 region proposal per image. Pendekatan ini lebih ringan daripada sliding window karena model tidak memeriksa seluruh bagian image satu per satu. Keterbatasannya, candidate box yang baik dapat tidak terbentuk pada image malam hari, kondisi silau, atau object kecil.

## Baseline eksperimen

Untuk metode klasik, eksperimen awal menggunakan class `car`. Penambahan class dilakukan setelah baseline selesai dijalankan dan dievaluasi.

| Bagian | Baseline awal |
|---|---|
| Class | `car` |
| Training data | 600 annotated image dari `train` |
| Evaluation data | 150 annotated image dari `val` |
| Region size | 96×96 |
| Selective Search | mode `fast`, maksimum 600 region proposal/image |
| Metric | mAP@0.5, precision, recall, dan seconds/image atau FPS |

Nilai baseline tersedia pada `configs/project_config.yaml`. Parameter dapat disesuaikan apabila diperlukan, tetapi perubahan dan alasannya harus dicatat pada experiment log. Folder `test` tidak digunakan untuk menghitung mAP karena tidak memiliki ground truth.

## Siapkan dataset

1. Unduh dataset dari Kaggle yang tercantum pada panduan dosen.
2. Ekstrak gambar ke `data/raw/bdd100k/images/100k/train` dan `data/raw/bdd100k/images/100k/val`.
3. Ekstrak label JSON ke `data/raw/bdd100k/labels/`.
4. Pastikan file berikut tersedia:
   - `labels/bdd100k_labels_images_train.json`
   - `labels/bdd100k_labels_images_val.json`
   - folder gambar `train` dan `val`
5. Pada JSON, gunakan objek dengan `category: "car"` dan `box2d`. Lane serta drivable area tidak digunakan.
6. Buat manifest berisi nama file, split, dan seed agar pemilihan sample dapat direproduksi.

## Pipeline yang harus diimplementasikan

### 1. Annotation validation

- Ambil `name`, setiap `labels[].category`, dan `labels[].box2d`.
- Ubah box menjadi `(x1, y1, x2, y2)` dan buang invalid box atau box terlalu kecil. Initial threshold: width/height minimum 32 pixel.
- Simpan CSV/JSON manifest berisi image yang benar-benar dipakai. Jangan mencampur `train` dan `val`.

### 2. Training sample preparation

- **Positive sample:** crop ground-truth car dari image `train`, resize ke 96×96, lalu extract HOG.
- **Initial negative sample:** crop Selective Search region proposal yang IoU-nya maksimal 0,20 terhadap seluruh ground-truth car pada image tersebut.
- Gunakan negative-to-positive ratio sekitar 3–8 negative sample per positive sample. Background perlu bervariasi dan tidak hanya terdiri dari area aspal kosong.
- Simpan jumlah positive sample, negative sample, image, dan seed pada experiment log.

### 3. HOG feature extraction

Gunakan grayscale dengan initial parameter berikut: orientations=9, pixels_per_cell=(8,8), cells_per_block=(2,2). Semua crop harus memiliki ukuran identik sebelum HOG. Standardize feature bila diperlukan oleh implementasi SVM, lalu simpan preprocessing pipeline bersama model.

### 4. Linear SVM training and hard-negative mining

- Train `LinearSVC` menggunakan positive dan negative HOG feature.
- Jalankan initial model pada sebagian image `train` yang tidak dipakai sebagai positive crop tunggal.
- Ambil background region proposal yang mendapat positive score sebagai **hard negative**, lalu retrain SVM dengan gabungan negative sample lama + hard negative.
- Simpan parameter `C`, iteration count, data size, dan training time. Minimal bandingkan baseline tanpa hard-negative mining dengan sesudah satu mining round jika waktu memungkinkan.

### 5. Detection inference

- Jalankan Selective Search pada full image.
- Buang region proposal yang terlalu kecil/besar tidak wajar atau melebihi maximum proposal count.
- Resize tiap region proposal, extract HOG, dan hitung Linear SVM score.
- Ambil region proposal dengan score di atas threshold. Lakukan NMS dengan initial IoU 0,40 agar overlapping box tidak berulang.
- Simpan prediction per image dalam JSON/CSV: image, class, score, x1, y1, x2, y2.

### 6. Evaluation on `val`

- Match prediction dan ground truth per image dengan IoU ≥ 0,5.
- Satu ground truth hanya boleh dipasangkan dengan satu prediction. Urutkan prediction berdasarkan score tertinggi.
- Laporkan AP/mAP@0.5, precision, recall, jumlah true positive/false positive/false negative, dan inference speed.
- Cantumkan laptop/CPU/GPU specification, library version, image count, dan class count. Kesimpulan harus mengacu pada metrik yang benar-benar diperoleh dari eksperimen.

## Pembagian kerja lima anggota

| Peran | Tanggung jawab | Hasil yang diserahkan ke tim |
|---|---|---|
| 1. Dataset | Download, folder structure, JSON parser, reproducible sampling | train/val manifest dan annotation statistic |
| 2. Region proposal | Selective Search, size filter, IoU, positive-negative crop | region-proposal visualization dan crop data |
| 3. Feature/model | HOG, Linear SVM, hard-negative mining | saved model dan training log |
| 4. Evaluation | NMS, mAP@0.5, precision/recall, timing | metric table dan prediction image |
| 5. Integrasi/PPT | menyatukan pipeline, README, visual, slide | repo/Drive rapi dan PPT final |

## Isi minimum PPT

1. Judul, anggota, dan tujuan deteksi.
2. BDD100K dataset: split yang dipakai, class, sample count, resolution/preprocessing.
3. Pipeline diagram: Selective Search → HOG → Linear SVM → NMS.
4. Cara membuat positive sample, negative sample, dan hard negative.
5. Main parameter: HOG, SVM, region proposal, threshold, NMS.
6. Quantitative result: mAP@0.5, precision, recall, inference time, hardware specification.
7. Prediction visualization: correct detection, false positive, dan false negative.
8. Limitation: missed region proposal pada malam/silau, small object, dan CPU cost.
9. Kesimpulan berdasarkan angka yang benar-benar diperoleh.

## Checklist sebelum pengumpulan

- [ ] Source code dapat dijalankan berdasarkan README.
- [ ] Dataset tidak diunggah ke Git/Drive kecuali memang diwajibkan.
- [ ] Data `train` dan `val` tidak tercampur.
- [ ] Output berupa bounding box dengan NMS, bukan hanya image classification.
- [ ] Metric dihitung dari annotated `val` data.
- [ ] PPT mencantumkan data count, class, parameter, metric, dan hardware specification.
- [ ] Source code dan PPT tersedia di Drive kelompok, akses link dibuka, kemudian link ditempel pada sheet pembagian kelompok.
