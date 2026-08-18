# Konstruksi & Dokumentasi Data — Studi Kasus Ayam Serayu

Data construction pipeline untuk 626 ribu baris transaksi POS 3 tahun (2023–2025), dari tahapan analisis distribusi titik pola hingga siap dipakai model, disertai dengan bukti empiris di setiap keputusan cleaning.

## Karakteristik Dataset

Dataset ini **tidak punya satu pun missing value**. Kalau divalidasi dengan `df.isna().sum()`
atau `df.describe()` saja, kelihatannya sudah rapi. Tapi dua cacat struktural serius baru
ketahuan setelah divalidasi terhadap aturan bisnisnya:

- **Tabrakan ID transaksi.** Format ID (`TRX-YYYYMMDD-4digit`) cuma menyediakan 10.000
  kombinasi per hari, sementara volume transaksi harian melebihi itu , sehingga dua transaksi berbeda
  bisa kebagian ID yang sama.
- **Granularitas salah.** Data tercatat per baris item, tapi kolom nilai transaksi
  di-_replikasi_ di setiap baris. Menjumlahkannya langsung membuat omzet **tergelembung 3,57×**.

Setiap keputusan pembersihan di notebook ini **diuji secara empiris** berdasarkan karakteristik dataset.

Contoh: 10.416 baris yang identik 100% _tidak_ dihapus, karena uji rekonsiliasi membuktikan
menghapusnya justru merusak kecocokan nilai pada 10.131 transaksi.

## Ringkasan alur (9 langkah)

| Langkah                   | Isi                                  | Temuan kunci                                     |
| ------------------------- | ------------------------------------ | ------------------------------------------------ |
| 1. Telaah & Validasi      | Audit struktur + 4 aturan bisnis     | ID tabrakan, granularitas salah, 0 missing value |
| 2. Strategi Cleaning      | Peta temuan → keputusan → alasan     | Tabel keputusan yang bisa diaudit                |
| 3. Koreksi Data           | Eksekusi + verifikasi integritas     | 0 baris/nilai hilang setelah cleaning            |
| 4. Transformasi & Fitur   | Cyclical encoding, agregasi, scaling | Omzet benar: Rp12,5M (bukan Rp44,6M)             |
| 5. Dokumentasi Konstruksi | Kamus data 28 fitur + rasional       | Reversibel & bisa diaudit ulang                  |
| 6. Pelabelan              | Audit label bawaan + SOP label baru  | Anti label-leakage, batas kelas tersimpan        |
| 7. Laporan Pelabelan      | Distribusi & evaluasi proses         | Tantangan & solusi didokumentasikan              |
| 8. Visualisasi            | 7 grafik siap presentasi             | Histogram, boxplot, time series, dll             |
| 9. Evaluasi Akhir         | Skor kualitas data sebelum/sesudah   | Consistency naik dari 24% → 100%                 |

## Struktur repo

```
ayam-serayu-data-construction/
├── README.md
├── requirements.txt
├── .gitignore
├── notebook/
│   └── Konstruksi_dan_Dokumentasi_Data_AyamSerayu.ipynb
├── data/
│   ├── raw/            # taruh CSV mentah di sini (tidak di-commit, lihat .gitignore)
│   └── processed/      # output kecil siap-commit: kamus data, laporan, artefak SOP
├── assets/
│   └── figures/        # 7 PNG hasil visualisasi
└── docs/
    └── tugas_praktikum.md   # instruksi tugas asli (opsional)
```

## Cara menjalankan

```bash
git clone https://github.com/mikailachmad/ayam-serayu-data-construction.git
cd ayam-serayu-data-construction
pip install -r requirements.txt

# letakkan dataset di data/raw/AyamSerayu_3Years_Transaction_Data.csv
jupyter notebook notebook/Konstruksi_dan_Dokumentasi_Data_AyamSerayu.ipynb
```

Notebook dijalankan `Run All`, kemudian seluruh output (CSV & PNG) otomatis tergenerate ke folder
`output/` lokal.

> Dataset mentah (~80MB) tidak disertakan di repo karena melebihi batas nyaman GitHub.
> File kecil hasil olahan (kamus data, laporan ringkas, artefak SOP) sudah tersedia
> di `data/processed/` sebagai contoh output.

## Temuan yang paling layak disorot

1. **Double counting 3,57×** — omzet naif Rp44,59M vs omzet benar Rp12,50M setelah agregasi
   ke level transaksi yang tepat.
2. **158.569 dari 208.730 transaksi (76%)** punya metode pembayaran berbeda antar-baris item
   dalam satu transaksi yang sama — anomali yang mengindikasikan data sintetis, ditangani
   dengan modus per transaksi + disclaimer eksplisit.
3. **SOP pelabelan anti-leakage** — label `segmen_transaksi` diturunkan dari nilai transaksi,
   sehingga daftar kolom terlarang (`kolom_bocor`) disusun eksplisit agar tidak ada yang
   diam-diam memakainya sebagai fitur model.

## Stack

`pandas` · `numpy` · `matplotlib` 

Tanpa dependency ML, agar notebook bisa dijalankan siapa pun tanpa environment rumit.
