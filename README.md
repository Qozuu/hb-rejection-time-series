# 📊 Time Series Forecasting: Penolakan Donor Darah Kriteria HB

Repository ini berisi proyek analisis deret berkala (*time series*) untuk memodelkan dan memprediksi jumlah penolakan calon pendonor darah akibat kriteria Hemoglobin (HB) di **UDD PMI Kota Surabaya**.

---

## 📌 Deskripsi Proyek
Data operasional harian penolakan donor darah akibat kadar HB yang tidak sesuai standar memiliki karakteristik **non-stasioner** dengan pola fluktuasi harian dan musiman. Proyek ini bertujuan untuk menganalisis sifat deret berkala data tersebut serta membandingkan performa peramalan antara metode statistik klasik parametrik dan pendekatan *machine learning*.

---

## 📊 Metodologi & Studi Komparasi
Proyek ini membandingkan dua pendekatan utama dalam menangani data *time series* non-stasioner:

1. **SARIMA (Seasonal Autoregressive Integrated Moving Average)**
   * **Kategori:** Model Statistik Klasik Parametrik.
   * **Pendekatan:** Memerlukan penanganan kestasioneran eksplisit melalui proses *differencing* (`d = 1`) serta pemodelan komponen musiman mingguan (`s = 7`).
2. **Meta Prophet (Facebook Prophet)**
   * **Kategori:** Model *Additive Time Series* berbasis *Machine Learning*.
   * **Pendekatan:** Mengurai data secara otomatis menjadi komponen tren non-linear, musiman mingguan/tahunan, dan *changepoints* tanpa mengharuskan diferensiasi manual.

---

## 📁 Dataset & Variabel
* **Sumber Data:** Log operasional harian UDD PMI Kota Surabaya (Januari – Agustus 2026).
* **Ukuran Data:** 243 observasi harian (`data_tolak_hb.csv`).

| Variabel | Tipe Data | Deskripsi |
| :--- | :--- | :--- |
| `TglPeriksa` | Datetime (Index) | Tanggal pemeriksaan donor (YYYY-MM-DD) |
| `Jumlah_Pendonor` | Integer | Total calon pendonor yang ditolak akibat kriteria HB |

---

## 📈 Progres Minggu 1: Analisis Kestasioneran Data
Pada tahap awal, dilakukan pengujian eksploratif untuk membuktikan asumsi ketidakstasioneran data:

### 1. Observasi Visual
* **Rolling Statistics (30 Hari):** Garis *Rolling Mean* dan *Rolling Standard Deviation* menunjukkan tren bergerak yang tidak stabil seiring waktu.
* **Autocorrelation Function (ACF):** Terlihat gelombang autokorelasi berulang pada kelipatan lag ke-7, 14, 21, dan 28 yang menandakan adanya **pola musiman mingguan** (*weekly seasonality*).

### 2. Uji Formal: Augmented Dickey-Fuller (ADF Test)

| Metrik Statistik | Nilai Uji | Ambang Kritis (Critical Value) | Evaluasi |
| :--- | :--- | :--- | :--- |
| **ADF Statistic** | `-1.8998` | `-2.8742` (Signifikansi 5%) | Nilai Uji > Critical Value |
| **p-value** | `0.3322` | `0.0500` (Alpha 5%) | **p-value > 0.05** |

> **Kesimpulan:** Berdasarkan hasil uji ADF (*p-value* = 0.3322 > 0.05), Hipotesis Nol (H0) **gagal ditolak**. Data secara sah terbukti bersifat **NON-STASIONER**.

---

## 📂 Struktur Repositori
```text
.
├── README.md
├── data_tolak_hb.csv
└── 01_data_prep_and_stationarity_test.ipynb
```

---

## 🛠️ Tech Stack & Library
* **Bahasa Pemrograman:** Python 3.x
* **Data Processing:** `pandas`, `numpy`
* **Visualisasi Data:** `matplotlib`, `statsmodels`
* **Uji Time Series:** `statsmodels.tsa.stattools.adfuller`, `plot_acf`
* **Environment:** Google Colab / Jupyter Notebook

---

<div align="center">
  <p><b>Copyright © 2026 Khairunisa Olive Ektha</b></p>
  <p>Sains Data — Universitas Pembangunan Nasional "Veteran" Jawa Timur</p>
  <p><sub><i>Studi Kasus Analisis Time Series Operasional UDD PMI Kota Surabaya</i></sub></p>
</div>
