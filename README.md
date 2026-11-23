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
- **Rolling Statistics (30 Hari):** Garis *Rolling Mean* dan *Rolling Standard Deviation* menunjukkan tren bergerak yang tidak stabil seiring waktu.
- **Autocorrelation Function (ACF):** Terlihat gelombang autokorelasi berulang pada kelipatan lag ke-7, 14, 21, dan 28 yang menandakan adanya **pola musiman mingguan** (*weekly seasonality*).

### 2. Uji Formal: Augmented Dickey-Fuller (ADF Test)

| Metrik Statistik | Nilai Uji | Ambang Kritis (Critical Value) | Evaluasi |
| :--- | :--- | :--- | :--- |
| **ADF Statistic** | `-1.8998` | `-2.8742` (Signifikansi 5%) | Nilai Uji > Critical Value |
| **p-value** | `0.3322` | `0.0500` (Alpha 5%) | **p-value > 0.05** |

> **Kesimpulan:** Berdasarkan hasil uji ADF ($p\text{-value} = 0{,}3322 > 0{,}05$), Hipotesis Nol ($H_0$) **gagal ditolak**. Data secara sah terbukti bersifat **NON-STASIONER**.

---

## 📈 Progres Minggu 2: Preprocessing, Differencing & Identifikasi Parameter
Tahap kedua memfokuskan pada penyiapan eksperimen yang adil (*fair comparison*) bebas kebocoran data (*data leakage*) sebelum masuk ke pemodelan:

### 1. Audit Frekuensi & Pembagian Data Kronologis (80:20)
- **Kontinuitas Harian:** Seluruh 243 hari observasi terverifikasi lengkap (`0` hari kosong) pada indeks frekuensi harian (`D`).
- **Train-Test Split Kronologis:** Data dibagi berurutan berdasarkan waktu menjadi **194 hari data latih** (`2026-01-01` s.d. `2026-07-13`) dan **49 hari data uji** (`2026-07-14` s.d. `2026-08-31`).

### 2. Pembuktian Stasioneritas Pasca-Differencing (Data Latih)

| Skenario Transformasi | ADF Statistic | p-value | Evaluasi |
| :--- | :--- | :--- | :--- |
| **First-Order Differencing (`d=1`)** | `-7.0847` | `0.0000` | **STASIONER** (`p < 0.05`) |
| **Seasonal + First Differencing (`d=1, D=1, s=7`)** | `-6.1085` | `0.0000` | **STASIONER** (`p < 0.05`) |

### 3. Identifikasi Kandidat Orde SARIMA & Penyiapan Data Prophet
- **Pembacaan ACF & PACF:** Plot ACF pada skenario `d=1, D=1, s=7` terpotong tajam (*cut-off*) setelah **lag 1** dan **lag musiman ke-7**, sedangkan plot PACF meluruh bertahap (*tails off*).
- **Kandidat Model Utama:** $\text{SARIMA}(0,1,1)(0,1,1)_7$ beserta model pembanding $\text{SARIMA}(1,1,1)(0,1,1)_7$, $\text{SARIMA}(1,1,1)(1,1,1)_7$, $\text{SARIMA}(2,1,1)(0,1,1)_7$, dan $\text{SARIMA}(0,1,2)(0,1,1)_7$.
- **Struktur Meta Prophet:** Penyesuaian nama kolom target menjadi `ds` (tanggal) dan `y` (jumlah penolakan) untuk pengujian pada rentang 49 hari yang identik.

---

## 📈 Progres Minggu 3: Pemodelan, Diagnostik Residual & Evaluasi Kinerja
Tahap ketiga mencakup estimasi seluruh kandidat model SARIMA, pengujian asumsi residual, pelatihan model Meta Prophet, serta komparasi akurasi terhadap tolok ukur *Seasonal Naive* pada 49 hari data uji:

### 1. Seleksi Model & Diagnostik Residual SARIMA
- **Model Terpilih:** Berdasarkan evaluasi kriteria informasi **AIC** dan **BIC** pada lima kandidat model, terpilih **`SARIMA(0,1,2)(0,1,1)₇`** dengan nilai **`AIC = 1571,40`** dan **`BIC = 1584,08`**.
- **Uji White Noise (Ljung-Box Test):** Pengujian autokorelasi residual menghasilkan *p-value* sebesar **`0,6827` (lag 7)** dan **`0,8442` (lag 14)** ($p > 0{,}05$), membuktikan residual bersifat acak (*white noise*) dan bebas autokorelasi tersisa.

### 2. Tabel Perbandingan Kinerja Peramalan (Periode Uji: 49 Hari)

| Model | MAE (orang) ↓ | RMSE (orang) ↓ | MASE ↓ |
| :--- | :---: | :---: | :---: |
| Seasonal Naive ($s=7$) | 18,65 | 24,09 | 0,97 |
| $\text{SARIMA}(0,1,2)(0,1,1)_7$ | 16,69 | 22,83 | 0,86 |
| **Meta Prophet ($s=7$)** | **16,06** | **21,35** | **0,83** |

> **Kesimpulan Evaluasi:** Pada periode uji 49 hari, **Meta Prophet ($s=7$)** menghasilkan tingkat kesalahan terkecil pada seluruh metrik (`MAE = 16,06 orang`, `RMSE = 21,35 orang`, `MASE = 0,83`) karena lebih adaptif menangkap tren kenaikan sekaligus siklus puncak akhir pekan dibandingkan `SARIMA(0,1,2)(0,1,1)₇`. Kedua model melampaui akurasi tolok ukur *Seasonal Naive* ($\text{MASE} < 1{,}00$).

---

## 📂 Struktur Repositori
```text
.
├── README.md
├── data_tolak_hb.csv
├── 01_data_prep_and_stationarity_test.ipynb
├── 02_preprocessing_and_order_identification.ipynb
├── 03_modeling_diagnostics_and_evaluation.ipynb
├── train_sarima.csv
├── test_sarima.csv
├── train_prophet.csv
├── test_prophet.csv
├── grafik_aktual_subblok02.png
├── diagnostik_residual_sarima.jpg
└── grafik_perbandingan_model_subblok04.jpg

```

---

## 🛠️ Tech Stack & Library
- **Bahasa Pemrograman:** Python 3.x
- **Data Processing:** `pandas`, `numpy`
- **Visualisasi Data:** `matplotlib`, `statsmodels`
- **Uji Time Series:** `statsmodels.tsa.stattools.adfuller`, `plot_acf`, `plot_pacf`, `acorr_ljungbox`
- **Forecasting & Evaluasi:** `statsmodels.tsa.statespace.sarimax.SARIMAX`, `prophet`, `scikit-learn` (`MAE`, `RMSE`, `MASE`)
- **Environment:** Google Colab / Jupyter Notebook

---

## 📚 Daftar Pustaka & Sumber Data Lengkap
1. **UDD PMI Kota Surabaya.** (2026). *Rekapitulasi Harian Penolakan Calon Pendonor Darah Kriteria Hemoglobin (Januari–Agustus 2026)*. Dataset Operasional Internal Unit Donor Darah PMI Kota Surabaya.
2. **Hyndman, R. J., & Athanasopoulos, G.** (2021). *Forecasting: Principles and practice* (3rd ed.). OTexts. https://otexts.com/fpp3/
3. **Taylor, S. J., & Letham, B.** (2018). Forecasting at scale. *The American Statistician*, 72(1), 37–45. https://doi.org/10.1080/00031305.2017.1380080
4. **Hyndman, R. J., & Koehler, A. B.** (2006). Another look at measures of forecast accuracy. *International Journal of Forecasting*, 22(4), 679–688. https://doi.org/10.1016/j.ijforecast.2006.03.001
5. **Kementerian Kesehatan Republik Indonesia.** (2015). *Peraturan Menteri Kesehatan Republik Indonesia Nomor 91 Tahun 2015 tentang Standar Pelayanan Transfusi Darah*. Kementerian Kesehatan RI.

---

<div align="center">
  <p><b>Copyright © 2026 Khairunisa Olive Ektha</b></p>
  <p>Sains Data — Universitas Pembangunan Nasional "Veteran" Jawa Timur</p>
  <p><sub><i>Studi Kasus Analisis Time Series Operasional UDD PMI Kota Surabaya</i></sub></p>
</div>