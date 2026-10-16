# Praktik Terpadu YOLO - RET503 (PJ1)

**NIM / Nama :** 4222411061 / ZAHRA FITRIAH
**Dataset    :** `dataset_v1` (sidik jari label: C6C077994836)
**Model      :** YOLO26n, dilatih di CPU (AMD Ryzen 3 3250U)

## 1. Data (Tahap 1-2)

- Sumber: subset COCO val2017, 3 kelas: `bottle`, `cup`, `handbag` (dipilih karena dekat dengan objek pada projek robot pengambil sampah).
- Isi: 200 gambar berobjek + 20 gambar latar (tanpa objek) = 220 gambar.
- Jumlah kotak: bottle 199, cup 181, handbag 131 (total 511).
- Pembagian: train 154, val 44, test 22.
- Program 01 melaporkan 45 peringatan. Yang tampil di terminal semuanya berjenis "kotak sangat kecil" (objek jauh dari kamera). Kotak tersebut tidak dihapus karena objek kecil adalah kondisi nyata.
- Uji konversi YOLO -> COCO -> YOLO (program 02 dan 03) utuh: 220 gambar, 511 kotak, 3 kategori, 220 file label.

## 2. Eksperimen (Tahap 3-5)

Ketiga eksperimen memakai data, epoch (30), seed, dan batch (4) yang sama. Hanya E3 yang mengubah satu hal: ukuran gambar.

| Eksperimen | Perubahan | mAP50 | mAP50-95 | P | R | Inferensi (ms) | Waktu latih (menit) |
|---|---|---|---|---|---|---|---|
| E1_pretrained | - (baseline) | 0,160 | 0,095 | 0,266 | 0,168 | 73,5 | 32,4 |
| E2_scratch | bobot acak | 0,000 | 0,000 | 0,000 | 0,037 | 107,6 | 39,5 |
| E3_tuning | imgsz=640 | 0,246 | 0,154 | 0,302 | 0,274 | 142,4 | 78,6 |

Hasil per kelas pada data val (mAP50-95):

| Kelas | Instance val | E1 | E3 |
|---|---|---|---|
| bottle | 38 | 0,114 | 0,147 |
| cup | 35 | 0,170 | 0,310 |
| handbag | 24 | 0,001 | 0,006 |

Catatan pengaturan: batch diturunkan menjadi 4 untuk ketiga eksperimen. Percobaan awal dengan batch 16 membuat E3 (640 piksel) berhenti mendadak di laptop ini; diduga karena keterbatasan memori.

## 3. Hasil test (Tahap 6)

Model terpilih (mAP50-95 val tertinggi): **E3_tuning**. Diuji sekali di data test (22 gambar, 69 instance):
mAP50 **0,343**, mAP50-95 **0,228**, P 0,46, R 0,318.

## 4. Analisis

**Transfer learning (E1 vs E2).** Model dari bobot acak hampir tidak belajar apa pun dalam 30 epoch (mAP 0,000, recall hanya 0,037), sedangkan model dengan bobot COCO mencapai mAP50-95 0,095. Dengan hanya 154 gambar latih, pengetahuan awal dari COCO yang menentukan: data sekecil ini tidak cukup untuk melatih detektor dari nol.

**Tuning (E1 vs E3).** Memperbesar gambar dari 416 ke 640 piksel menaikkan mAP50-95 dari 0,095 ke 0,154 (sekitar 62% lebih tinggi). Harganya: waktu latih 2,4 kali lebih lama (32,4 -> 78,6 menit) dan inferensi sekitar 1,9 kali lebih lambat (73,5 -> 142,4 ms, atau sekitar 7 FPS di CPU ini, dibanding sekitar 14 FPS). Perubahan ini saya pertahankan karena akurasi lebih penting untuk tahap ini, tetapi untuk robot yang membutuhkan respons cepat, kecepatan 7 FPS perlu dipertimbangkan. Perlu dicatat bahwa ini satu kali percobaan pada 44 gambar val, sehingga selisih kecil bisa saja bukan perbedaan sungguhan.

**Perbedaan antar kelas.** `cup` terdeteksi paling baik (mAP50-95 0,310 pada E3), `bottle` menengah (0,147), dan `handbag` hampir tidak terdeteksi (0,006; presisi dan recall 0 pada kedua model). Kenaikan skor E3 berasal dari bottle dan cup. Handbag memiliki data paling sedikit (131 kotak, dibanding 199 untuk bottle), yang sejalan dengan dugaan efek ketidakseimbangan kelas. Namun selisih jumlahnya tidak besar, sehingga ada kemungkinan penyebab lain, seperti bentuk handbag yang sangat beragam dan sering sebagian tertutup. Data ini tidak cukup untuk memisahkan kedua penyebab tersebut.

**Data test.** Skor test (mAP50-95 0,228) lebih tinggi daripada val (0,154), tetapi data test hanya 22 gambar sehingga angkanya mudah berfluktuasi. Hasil ini tidak ditafsirkan sebagai model yang lebih baik pada data baru.

## 5. Kesimpulan dan keterbatasan

- Transfer learning sangat menentukan untuk data sekecil ini, dan memperbesar ukuran gambar membantu, dengan konsekuensi waktu.
- Angka mutlak masih rendah. Penyebab yang masuk akal: data kecil, banyak objek kecil, dan batch kecil akibat keterbatasan CPU. Dugaan ini belum diuji.
- Kelas handbag belum layak dipakai. Langkah perbaikan: tambah gambar handbag, periksa kualitas anotasi, tambah epoch, dan evaluasi di data yang lebih besar.
- Untuk projek robot, sebaiknya alur yang sama diulang dengan foto dari kamera robot sendiri.

## 6. Berkas

- `dataset_v1/` : dataset dan `data.yaml`. Catatan: `path` di dalam `data.yaml` berisi jalur absolut komputer saya (`C:/Users/Lenovo/...`); jika dijalankan di komputer lain, jalur itu perlu diubah.
- `hasil_praktik/E1_pretrained/`, `E3_tuning/` : bobot, `results.png`, `results.csv`, `confusion_matrix.png`
- Hasil E2_scratch hanya tercantum pada tabel di `hasil_praktik_4222411061.txt` dan `.csv`; folder E2 tidak disertakan karena tidak diminta.
- `hasil_praktik_4222411061.txt` dan `.csv` : tabel hasil
- Perintah yang dijalankan: `python 06_latih_bandingkan.py --nim 4222411061 --nama "ZAHRA FITRIAH" --epochs 30 --batch 4 --device cpu`