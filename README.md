# 📊 End-to-End E-Commerce Analytics Dashboard: Commercial, Behavior & Web Funnel

## 📌 Project Overview
Proyek ini merupakan portofolio analisis data *end-to-end* yang mengeksplorasi data fiktif operasional platform e-commerce retail bernama **"The Look"**. Laporan ini dirancang secara arsitektural untuk memenuhi kebutuhan dua tipe audiens pemangku kepentingan:
1. **Macro View (C-Level Executives):** Menyajikan performa kesehatan finansial, kontrol margin profitabilitas, dan tren revenue dari bulan ke bulan.
2. **Micro View (Growth & Marketing Managers):** Berfungsi sebagai alat investigasi mendalam (*deep-dive analysis*) interaktif untuk membedah perilaku segmen pelanggan, melacak corong konversi situs web, serta mengidentifikasi titik kebocoran pendapatan (*revenue leaks*).

---

## 🗄️ 1. Data Sourcing, Ingestion, & Scale
* **Source Dataset:** Dataset mentah diperoleh dari platform Kaggle (**TheLook E-Commerce Dataset**), yang merepresentasikan skema basis data transaksional riil yang kompleks.
* **Database Engine & IDE:** Menggunakan **PostgreSQL** sebagai mesin relasional dan **DBeaver** sebagai *Integrated Development Environment* (IDE). Data mentah berbentuk CSV di-impor ke dalam skema lokal secara manual sebelum pemrosesan kueri dimulai.
* **Data Scale (Skala Data):** Total volume data yang dikelola mencakup **+2.8 Juta baris data transaksional**, membuktikan performa dashboard ini diuji dalam skenario pemrosesan data berskala besar (*large-scale data processing*):
  * `events` (Log Aktivitas Situs Web): **2.431.963 baris**
  * `order_items` (Detail Item Pesanan): **181.759 baris**
  * `orders` (Data Transaksi Utama): **125.226 baris**
  * `users` (Profil Pelanggan): **100.000 baris**
  * `products` (Katalog Master Produk): **29.120 baris**

---

## ⚙️ 2. Data Pipeline & Transformation (ETL)

### Level Database (SQL Feature Engineering & Pushdown Logic)
Untuk mengoptimalkan performa visualisasi di Power BI dan menghindari *latency* (lag) akibat kalkulasi data mentah, beban komputasi berat digeser ke level database menggunakan strategi **Pushdown Logic**. 

* **Model Segmentasi RFM:** Menggunakan *Common Table Expressions* (CTE) dan *Window Function* `NTILE(5)` untuk memberikan skor absolut (1-5) pada metrik *Recency, Frequency,* dan *Monetary*. Skor ini kemudian dipetakan menggunakan logika `CASE WHEN` ke dalam 6 segmen perilaku bisnis (e.g., *Champions, Loyal, Regular, At-Risk, Hibernating*).
* **Demographic Feature Engineering:** Mentransformasikan kolom usia (`age`) menjadi pengelompokan generasi sosiologis (*Gen Z, Millennials, Gen X, Boomers*) menggunakan klausa `BETWEEN`.

> **💡 customer_segmentation query** 
> <img width="1317" height="414" alt="image" src="https://github.com/user-attachments/assets/c37399a9-1f6f-4ce2-8749-9e799adf8ccb" />
> <img width="1361" height="504" alt="image" src="https://github.com/user-attachments/assets/f01dfd03-50a0-4d74-9a1f-0ce8207e9bf3" />

### Level BI (Power Query & Data Cleansing)
* **Penanganan Data Hilang (Null Handling):** Mengidentifikasi bahwa kolom `user_id` pada tabel `events` memiliki ribuan nilai *null*. Kolom ini tidak dihapus karena merepresentasikan pengunjung **"Guest" (Anonim)**. Nilai *null* digantikan dengan nilai -1 untuk memetakan kolom baru bernama customer_type dan memberi label "Guest" untuk data dengan user_id kosong di Power Query untuk menjaga validitas *Traffic Volume* dan *Conversion Rate*.
* **Pencegatan Waktu (Timestamp Optimization):** Mengekstrak komponen jam (`Hour`) dari kolom tanggal-waktu `created_at` ke dalam kolom terpisah sebelum mengubah tipe data kolom utama menjadi *Date* murni. Langkah ini menjamin keutuhan relasi kalender ke tabel *Dimension Date* tanpa kehilangan detail waktu penting untuk tren dimana jam belanja tinggi.

---

## 🔗 3. Data Modeling (Arsitektur Star Schema)
Dashboard ini menerapkan pemodelan data **Star Schema** yang solid untuk memastikan efisiensi kalkulasi DAX dan filter interaktif yang bersih:
* **Fact Tables:** `fact_order_items` (data komersial) dan `events` (data aktivitas web).
* **Dimension Tables:** `Dim_RFM` (dimensi perilaku hasil kueri SQL), `Dim_Date` (tabel kalender dinamis), `Dim_Users`, dan `Dim_Products`.
* **Relasi:** Seluruh relasi dibangun menggunakan hubungan *One-to-Many* (1 -> *) dengan arah filter tunggal (*Single Direction Layout*).

> **💡 Data Modelling Star Schema** 
> <img width="1372" height="827" alt="image" src="https://github.com/user-attachments/assets/f7d72585-ada5-4b54-942a-ecce5d27e4ae" />


---

## 📊 4. Deep-Dive Analysis per Halaman & Grafik

### 📈 HALAMAN 1: Executive & Commercial Summary
**Tujuan:** Mengaudit kesehatan keuangan eksekutif dan efisiensi manajemen portofolio produk.

> **💡 Dashboard 1 Executive & Commercial Summary**.
> <img width="1463" height="812" alt="image" src="https://github.com/user-attachments/assets/d59857f8-f6a8-451b-a5df-ab5396c6eab9" />


* **Metrik Utama (KPI Cards):**
  * **Total Revenue ($2.72M) & Total Profit ($1.41M):** Merepresentasikan volume dan profitabilitas bisnis yang kuat.
  * **Average Order Value / AOV ($86.62):** Nilai rata-rata pengeluaran pelanggan dalam sekali checkout, menjadi patokan batas toleransi biaya iklan (*Customer Acquisition Cost* / CAC).
  * **Profit Margin (51.92%):** Indikator efisiensi harga modal (COGS) yang sangat sehat, jauh di atas rata-rata industri ritel biasa.
* **Total Revenue MoM Over Year (Line Chart):**
  * *Tujuan:* Menggunakan grafik garis kumulatif tahunan untuk mendeteksi pola tren musiman (*seasonality*).
  * *Insight & Dampak:* Terjadi lonjakan pendapatan masif konsisten di kuartal ke-4 (Oktober - Desember), dengan puncak tertinggi menyentuh $0.33M di akhir tahun. *Impact:* Tim rantai pasok (*Supply Chain*) wajib melakukan penumpukan stok (*restock*) komoditas utama sejak bulan September untuk menghadapi lonjakan permintaan di akhir tahun.
* **Revenue vs Quantity Sold by Product Category (Bar Charts):**
  * *Tujuan:* Menyandingkan dua grafik batang horizontal untuk membandingkan kategori produk dari sudut pandang Nilai Uang (Revenue) vs Volume Barang (Quantity).
  * *Tujuan:* Kategori **"Outerwear & Coats"** adalah penyumbang pendapatan tertinggi ($0.33M) karena harga per unit yang mahal, namun secara volume, **"Intimates"** dan **"Jeans"** memimpin pasar dengan penjualan tertinggi (>3.4K unit). *Impact:* Tim komersial harus memanfaatkan "Intimates" dan "Jeans" sebagai produk pancingan (*hook products*) lewat diskon volume, lalu menerapkan strategi penawaran silang (*cross-selling*) dengan produk bermargin tinggi seperti "Outerwear".

---

### 👥 HALAMAN 2: Customer Segmentation & Behavior Analysis
**Tujuan:** Mengidentifikasi loyalitas basis pelanggan, karakteristik demografi, dan mengoptimalkan waktu kampanye pemasaran.

> **💡 Dashboard 2 Customer Segmentation & Behavior** 
> <img width="1458" height="818" alt="image" src="https://github.com/user-attachments/assets/d740597e-43d0-4627-828a-ddbb8de52750" />


* **Metrik Utama (KPI Cards):**
  * **Total Active Customers (27.70K):** Ukuran total kolam pembeli aktif yang masuk kualifikasi RFM.
  * **Avg. Order Frequency (1.13x):** Angka ini mengindikasikan bahwa mayoritas dari 27 ribu pelanggan tersebut adalah pembeli satu kali (*one-time buyers*) yang tidak pernah kembali lagi. *Impact:* Biaya akuisisi pelanggan (CAC) perusahaan terancam boncos jika tidak diimbangi dengan strategi peningkatan *Customer Lifetime Value* (LTV).
* **Segment Proportion & Revenue Contribution (Donut Chart & Column Chart):**
  * *Tujuan:* Donut chart mendeteksi ukuran populasi, sementara Column chart melacak sumbangsih uangnya.
  * *Insight & Dampak:* Segmen **'Regular'** mendominasi secara masif dengan proporsi **47.09% (13.05K pelanggan)** dan menyumbangkan pendapatan terbesar bagi perusahaan mencapai **$1.0M**. Hal ini mematahkan asumsi bahwa segmen VIP ('Champions') selalu memegang peranan terbesar. *Impact:* Fokus strategi promosi pemasaran harus bergeser ke arah pemeliharaan segmen mayoritas 'Regular' ini, misalnya dengan insentif kupon belanja kedua untuk mendorong mereka naik kelas menjadi *Loyal Customers*.
* **Demographic Profile by Segment (Stacked Bar Chart):**
  * *Tujuan:* Membedah sebaran demografi generasi di dalam setiap segmen RFM.
  * *Insight & Dampak:* Seluruh lapisan segmen pelanggan, baik VIP maupun pelanggan berisiko tinggi kabur, didominasi mutlak oleh generasi **Millennials** dan **Gen Z**. *Impact:* Tim materi kreatif iklan wajib menghentikan gaya promosi media tradisional konvensional dan mengalihkan 100% fokus *branding* ke platform visual modern seperti TikTok dan Instagram Ads dengan pendekatan estetika khas anak muda.
* **Hourly Purchasing Trends (Line Chart):**
  * *Tujuan:* Melacak fluktuasi volume order harian berdasarkan jam transaksi (0-23).
  * *Insight & Dampak:* Aktivitas pembelian sangat stabil sepanjang siang hari, namun mengalami fenomena penurunan drastis menuju angka nol tepat setelah pukul 19.00 - 20.00 malam. *Impact:* Tim operasional pemasaran digital (*Digital Marketer*) dapat menghentikan penayangan iklan berbayar di atas jam 8 malam untuk menghemat anggaran, dan menjadwalkan pengiriman notifikasi promo (*Push Notification*) pada jendela jam aktif belanja siang hari.

---

### 🛒 HALAMAN 3: Website Analytics & Conversion
**Tujuan:** Memetakan efisiensi perjalanan digital pengguna (*User Digital Journey*) dan melacak kualitas konversi platform.

> **💡 Dashboard 3 Website Analytics & Conversion**
> <img width="1452" height="811" alt="image" src="https://github.com/user-attachments/assets/3eb63057-6a04-4c18-9250-b44baf53240d" />


* **Metrik Utama (KPI Cards):**
  * **Total Sessions (682K):** Tingkat kepadatan volume lalu lintas pengunjung *website*.
  * **Conversion Rate (26.7%):** Efektivitas kunjungan situs web menjadi transaksi pembelian sukses. *(Catatan Portofolio: Secara e-commerce nyata, angka 26.7% tergolong anomali karena benchmark asli berkisar di angka 2%-3%. Angka tinggi ini mencerminkan dataset sintetis).*
  * **Cart Abandonment Rate (57.9%):** Metrik paling krusial bagi tim UI/UX. Sebanyak 57.9% sesi di mana pengunjung sudah memasukkan barang ke keranjang belanjaan berakhir terbengkalai tanpa dibayar. *Impact:* Kebocoran pendapatan yang sangat masif. Perusahaan wajib mengaktifkan fitur *Automated Abandoned Cart Email Recovery* dalam kurun waktu 1x24 jam untuk menyelamatkan potensi uang yang hilang.
  * **Cancellation Rate (18.4%):** Sebanyak 18.4% sesi mengalami pembatalan. Indikasi adanya masalah pada gerbang pembayaran (*payment gateway*) atau ketidakpuasan biaya pengiriman di akhir.
* **Shopping Journey & Conversion Funnel (Funnel Chart):**
  * *Tujuan:* Grafik corong secara visual paling efektif menggambarkan akumulasi penyusutan jumlah audiens di setiap tahap penjelajahan.
  * *Insight & Dampak:* Dari 682K kunjungan, sebanyak 432K melakukan aksi *Add to Cart* (indikasi minat produk sangat tinggi), namun menyusut tajam menjadi 182K di tahap *Purchase*. Kebocoran bukan terjadi pada ketertarikan produk, melainkan pada kemudahan transaksi di halaman checkout belanja. *Impact:* Rekomendasi perbaikan total struktur UI/UX halaman pembayaran menggunakan metode *One-Click Checkout*.
* **Acquisition Channel: Traffic Volume vs Conversion Rate (Combo Chart):**
  * *Tujuan:* Menggabungkan grafik batang (Volume Sesi) dan grafik garis (Tingkat Konversi dengan Secondary Y-axis) untuk membandingkan secara langsung aspek Kuantitas vs Kualitas platform eksternal dalam satu ruang visual.
  * *Insight & Dampak:* Saluran **Facebook** mendatangkan volume kunjungan (*batang*) yang relatif rendah, tetapi memiliki tingkat konversi penjualan (*garis*) tertinggi mencapai ~27%. Sebaliknya, Email/Organic mendatangkan banyak orang tetapi kurang menghasilkan transaksi. *Impact:* Rekomendasi alokasi ulang anggaran belanja iklan (*Ad Spend*) komersial dialihkan untuk memperbesar skala iklan di Facebook Ads karena menghasilkan *Return on Ad Spend* (ROAS) yang jauh lebih superior.
* **Abandonment Rate by Browser (Clustered Bar Chart):**
  * *Tujuan:* Menggunakan bar chart horizontal untuk membandingkan performa rasio kegagalan bayar di berbagai alat penjelajah (*browser*).
  * *Insight & Dampak:* Grafik ini berfungsi sebagai instrumen teknis *Quality Assurance* (QA). Di dalam dataset nyata, jika rasio kegagalan melonjak tinggi pada salah satu browser (misalnya Safari menyentuh 90% sedangkan Chrome 40%), ini merupakan indikasi kuat adanya *bug* pada halaman web di ekosistem iOS/Mac yang harus segera ditangani oleh tim Web Developer.
* **Total Sessions by Customer Type (Donut Chart):**
  * *Tujuan:* Menggunakan Donut Chart untuk melihat perbandingan total sesi web antara *User* dengan akun terdaftar dan *Guest* yaitu tamu yang tidak menggunakan akun terdaftar
  * *Insight & Dampak:* Menemukan fakta bahwa 73.34% traffic (500K sesi) berasal dari pengunjung anonim (Guest). Terdapat korelasi kuat antara jumlah pengguna terautentikasi (182K) dengan jumlah transaksi berhasil (182K). Hal ini mengindikasikan adanya Login Wall (kewajiban membuat akun) yang menjadi akar masalah tingginya Cart Abandonment Rate. Impact: Rekomendasi untuk merilis fitur Guest Checkout demi melancarkan proses konversi pengunjung anonim.
---
