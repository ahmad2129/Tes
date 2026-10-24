# Analisis dan Prediksi Time Series: Total Penjualan Harian

Mini project analisis time series dengan Python untuk memahami pola penjualan harian dan memprediksi 30 hari ke depan menggunakan model **SARIMA**.

## Ringkasan Project

| Item | Keterangan |
|---|---|
| Tujuan | Memahami pola penjualan harian dan memprediksi 30 hari ke depan |
| Dataset | `Time Series - data train.csv` (3.000.888 baris, 6 kolom) |
| Periode data | 1 Januari 2013 sampai 15 Agustus 2017 |
| Tools | Python, Pandas, NumPy, Matplotlib, Seaborn, Statsmodels |
| Model final | **SARIMA(0,1,1)x(0,1,1,7)** |
| Akurasi pada data uji | RMSE 0,084 juta, MAPE sekitar 6,6% |

## Dataset

Setiap baris mencatat penjualan satu kategori produk di satu toko pada satu tanggal.

| Kolom | Deskripsi |
|---|---|
| `id` | Nomor identifikasi unik setiap baris |
| `date` | Tanggal pencatatan penjualan |
| `store_nbr` | Nomor identifikasi toko |
| `family` | Kategori produk yang dijual |
| `sales` | Jumlah penjualan kategori produk di toko pada tanggal tersebut |
| `onpromotion` | Jumlah produk dalam kategori terkait yang sedang dipromosikan |

Untuk analisis time series, `sales` dijumlahkan per tanggal sehingga menjadi satu deret total penjualan harian.

## Alur Analisis

1. **Data Understanding dan Exploration.** Memeriksa tipe data, inkonsistensi, missing value, duplikat, outlier (IQR), dan distribusi data.
2. **Data Preparation.** Mengubah `date` menjadi datetime, serta `id` dan `store_nbr` menjadi category.
3. **Persiapan deret waktu.** Dataset hanya memiliki 1.684 tanggal, padahal rentang 1 Januari 2013 sampai 15 Agustus 2017 seharusnya 1.688 hari. Empat hari yang hilang adalah 25 Desember tiap tahun (2013 sampai 2016). Deret diubah menjadi frekuensi harian dengan `asfreq("D")`, lalu tanggal kosong diisi dengan `interpolate()`.
4. **Smoothing.** Simple Moving Average (SMA) 30 hari untuk melihat tren.
5. **Uji stasioneritas.** ADF pada data asli menghasilkan p-value 0,09 (belum stasioner), sehingga dilakukan differencing (d = 1). Setelah itu p-value menjadi sangat kecil (stasioner).
6. **ACF dan PACF.** Grafik awal menunjukkan lonjakan di lag 7 dan 14, yang menandakan pola musiman mingguan. Dilakukan seasonal differencing 7 hari (D = 1), lalu ACF/PACF dibaca ulang dengan 42 lag.
7. **Pemilihan model.** Enam kandidat SARIMA dibandingkan dengan data uji 60 hari terakhir.
8. **Prediksi.** Model terbaik dilatih ulang dengan seluruh data untuk memprediksi 30 hari ke depan.

## Pemilihan Model

Dari ACF setelah d = 1 dan D = 1, lag 1 dan lag 7 menonjol sementara lag 14 dan 21 masuk daerah batas kepercayaan, sehingga q = 1 dan Q = 1. PACF menurun perlahan tanpa terputus tegas, sehingga p dan P dimulai dari 0. Kandidat lain ditambahkan sebagai pembanding.

| Model | AIC | BIC | RMSE (juta) | MAPE (%) |
|---|---|---|---|---|
| **SARIMA(0,1,1)x(0,1,1,7)** | -2731,191 | -2715,045 | **0,084** | **6,641** |
| SARIMA(0,1,1)x(1,1,1,7) | -2737,809 | -2716,281 | 0,085 | 6,775 |
| SARIMA(0,1,2)x(0,1,1,7) | -2823,035 | -2801,509 | 0,088 | 7,460 |
| SARIMA(1,1,1)x(0,1,1,7) | -2824,722 | -2803,193 | 0,095 | 8,651 |
| SARIMA(1,1,1)x(1,1,1,7) | -2824,429 | -2797,519 | 0,095 | 8,632 |
| SARIMA(2,1,2)x(1,1,1,7) | -2829,282 | -2791,611 | 0,100 | 9,329 |

Model yang lebih rumit memiliki AIC/BIC lebih baik pada data latih, tetapi justru lebih meleset pada data uji. Model **SARIMA(0,1,1)x(0,1,1,7)** dipilih karena paling akurat pada data uji, paling sederhana, dan sesuai dengan bacaan ACF.

## Hasil Prediksi 30 Hari (16 Agustus sampai 14 September 2017)

| Ringkasan | Nilai |
|---|---|
| Total prediksi 30 hari | 23,13 juta |
| Rata-rata hari kerja | 0,698 juta |
| Rata-rata akhir pekan | 0,972 juta |
| Hari prediksi tertinggi | 20 Agustus 2017 (1,007 juta) |
| Hari prediksi terendah | 14 September 2017 (0,599 juta) |

Satuan pada tabel di atas adalah juta dari nilai `sales`. Satuan asli kolom `sales` tidak dijelaskan di dokumentasi dataset.

## Insight

Penjualan harian bergerak mengikuti ritme mingguan yang sangat konsisten: **akhir pekan selalu menjadi puncak, sedangkan Kamis menjadi hari paling sepi**. Setelah data distasionerkan dengan differencing dan seasonal differencing 7 hari, model **SARIMA(0,1,1)x(0,1,1,7)** terbukti paling akurat dibanding lima kandidat lain dengan **MAPE sekitar 6,6%**, bahkan model yang lebih rumit justru meleset lebih jauh pada data uji. Prediksi 30 hari ke depan menunjukkan pola yang sama: rata-rata penjualan akhir pekan sekitar **0,97 juta**, atau **sekitar 39% lebih tinggi** dibanding hari kerja yang sekitar **0,70 juta**, dengan total sekitar **23,13 juta**. Model belum mampu menangkap lonjakan sesekali di luar pola mingguan, dan rentang prediksi makin lebar untuk tanggal yang lebih jauh, sehingga hasilnya paling andal untuk **satu sampai dua minggu pertama**.

## Recommendation

Berdasarkan pola tersebut, perencanaan **stok dan jadwal staf** dapat dipertimbangkan mengikuti ritme mingguan, yaitu **menambah persediaan dan tenaga kerja menjelang Sabtu dan Minggu**, serta **menyesuaikan lebih ringan di hari Kamis**. Prediksi disarankan **diperbarui secara berkala dengan data terbaru**, terutama untuk perencanaan lebih dari dua minggu ke depan. Untuk pengembangan selanjutnya, dapat dipertimbangkan menambahkan **informasi promosi (`onpromotion`)** dan efek tanggal (misalnya awal atau akhir bulan) ke dalam model agar lonjakan sesekali ikut tertangkap.

## Keterbatasan

- Uji Ljung-Box pada residual masih menunjukkan sisa pola, artinya model belum menangkap seluruh struktur data.
- Pengujian akurasi memakai satu jendela uji (60 hari terakhir), sehingga belum menggambarkan performa di periode lain.
- Model hanya memakai riwayat penjualan itu sendiri, tanpa variabel promosi, hari libur, atau efek tanggal.
- Tanggal 25 Desember yang hilang diisi dengan interpolasi, bukan nilai asli.

## Cara Menjalankan

1. Siapkan file `Time Series - data train.csv` di folder yang sama dengan notebook.
2. Install library: `pip install pandas numpy matplotlib seaborn statsmodels scikit-learn scipy`
3. Jalankan `Mini_Project_Time_Series_Analysis_Ahmad_Malik_Ibrahim.ipynb` dari atas ke bawah.

## Penulis

**Ahmad Malik Ibrahim**
[GitHub](https://github.com/ahmad2129) | [LinkedIn](https://www.linkedin.com/in/ahmad-malik-ibrahim/)
