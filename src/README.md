# Rencana isi folder `src`

Implementasi script dilakukan secara bertahap sesuai urutan berikut agar setiap tahap dapat diverifikasi.

1. `make_manifest.py` — membaca BDD100K JSON, filter class, lalu memilih sample train/val secara reproducible.
2. `build_training_data.py` — membuat positive crop dan negative crop dari Selective Search region proposal.
3. `train_svm.py` — HOG feature extraction, Linear SVM training, dan hard-negative mining.
4. `predict.py` — menjalankan Selective Search, memberi score pada region proposal, lalu NMS.
5. `evaluate.py` — menghitung mAP@0.5, precision, recall, dan inference time pada val split.

Setiap script membaca `configs/project_config.yaml` agar parameter tidak tersebar pada banyak file.
