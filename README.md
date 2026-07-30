# Financial Health Overview Dashboard 

![alt_text](https://res.cloudinary.com/dk2tex4to/image/upload/v1785080097/Screenshot_2026-07-26_223425_vcxc9d.png)


Dashboard ini dikembangkan untuk memantau kondisi bisnis bank menggunakan *Financial Health Framework* sebagai kerangka konseptual, diimplementasikan pada data operasional bank (nasabah, rekening, transaksi, pinjaman, kartu) menggunakan Power BI.

## 1. Latar Belakang

*Financial Health Framework* pada industri perbankan umumnya terdiri atas 5 dimensi:

| Dimensi | KPI Resmi | Fungsi |
|---|---|---|
| Profitability | Net Interest Margin (NIM), Return on Assets (ROA) | Mengukur kemampuan bank menghasilkan laba |
| Liquidity | Loan-to-Deposit Ratio (LDR), Liquidity Coverage Ratio (LCR) | Mengukur kesiapan kas menghadapi kewajiban jangka pendek |
| Capital Adequacy | Capital Adequacy Ratio (CAR) | Mengukur kecukupan modal menyerap risiko |
| Asset Quality | Non Performing Loan (NPL), Provision Coverage Ratio (PCR) | Mengukur kualitas portofolio kredit |
| Operational Efficiency | Cost to Income Ratio (CIR), Profit per Employee | Mengukur efisiensi biaya dan produktivitas |

Dataset yang digunakan pada proyek ini bersifat **operasional** (bukan laporan keuangan resmi bank), terdiri atas 5 tabel: `customers`, `accounts`, `transactions`, `loans`, `cards`. Oleh karena itu, tidak seluruh KPI resmi di atas dapat dihitung - lihat bagian Limitasi untuk detail lengkapnya.

---

## 2. Data Availability Assessment

| Financial Health Indicator | Status | Keterangan |
|---|---|---|
| Net Interest Margin (NIM) | Tidak dapat dihitung | Tidak ada data beban bunga deposito (*interest expense*) maupun rata-rata aset produktif |
| Return on Assets (ROA) | Tidak dapat dihitung | Tidak ada data laba bersih maupun total aset bank |
| Capital Adequacy Ratio (CAR) | Tidak dapat dihitung | Tidak ada data modal bank maupun Aset Tertimbang Menurut Risiko (ATMR) |
| Liquidity Coverage Ratio (LCR) | Tidak dapat dihitung | Tidak ada data High Quality Liquid Assets maupun proyeksi arus kas keluar |
| Cost to Income Ratio (CIR) | Tidak dapat dihitung | Tidak ada data biaya operasional maupun data karyawan |
| Non Performing Loan (NPL) | Dapat dihitung | Tersedia dari kolom `loans.status` (nilai `overdue` dan `written_off`) |
| Loan-to-Deposit Ratio (Proxy) | Dapat dihitung | Tersedia dari `loans.outstanding_idr` dan `accounts.balance_idr` |
| Estimated Loan Interest Income | Dapat dihitung | Tersedia dari `loans.outstanding_idr` × `loans.interest_rate_annual` |

**Kesimpulan:** Dari 5 dimensi Financial Health, hanya **3 dimensi yang bisa direpresentasikan** pada dashboard ini  **Profitability** (sebagian, lewat Derived KPI), **Liquidity** (lewat Proxy KPI), dan **Asset Quality** (KPI resmi, NPL). **Capital Adequacy** dan **Operational Efficiency** sama sekali tidak dapat diimplementasikan karena keterbatasan data.

## 3. KPI yang Digunakan dan Alasan Pemilihannya

### 3.1 Klasifikasi KPI

Karena tidak semua indikator resmi tersedia, KPI pada dashboard ini diklasifikasikan menjadi 3 jenis:

- **Primary KPI** - dihitung langsung dari data tanpa asumsi tambahan.
- **Derived KPI** - metrik hasil perhitungan yang **tidak menggantikan** KPI resmi manapun (konsep berbeda).
- **Proxy KPI** - metrik yang mengukur **konstruk yang sama** dengan KPI resmi, hanya sumber datanya disesuaikan.

### 3.2 Tabel KPI, Formula, dan Alasan

| KPI | Jenis | Dimensi | Formula | Alasan Pemilihan |
|---|---|---|---|---|
| **Total Deposit Balance** | Primary | Liquidity | `SUM(accounts.balance_idr)` | Representasi langsung dana nasabah yang dikelola bank; dasar penghitungan LDR. |
| **Outstanding Loan** | Primary | Profitability / Asset Quality | `SUM(loans.outstanding_idr)` | Mengukur eksposur kredit yang masih berjalan; snapshot kondisi kredit terkini. |
| **Estimated Loan Interest Income** | **Derived** | Profitability (aspek pendapatan bunga) | `SUMX(loans, outstanding_idr × interest_rate_annual)` | Bukan pengganti NIM - NIM adalah *rasio* margin bunga bersih terhadap aset produktif, sedangkan ini adalah estimasi *nilai absolut* pendapatan bunga kotor, tanpa memperhitungkan beban bunga (cost of fund) atau rata-rata aset produktif. Kedua nilai bahkan berpotensi bergerak berlawanan arah, sehingga tidak layak disebut proxy NIM. |
| **Loan-to-Deposit Ratio (Proxy)** | **Proxy** | Liquidity | `Outstanding Loan ÷ Total Deposit Balance × 100%` | Secara konsep tetap mengukur konstruk yang sama dengan LDR resmi (kredit dibagi dana pihak ketiga); hanya definisi "dana pihak ketiga" memakai saldo rekening operasional (tabungan, deposito, bisnis), bukan definisi DPK versi regulasi OJK. |
| **Non Performing Loan (NPL)** | Primary | Asset Quality | `(Jumlah loan status overdue + written_off) ÷ Total Loan × 100%` | KPI resmi industri perbankan; satu-satunya rasio resmi yang bisa dihitung utuh dari dataset ini. |
| **Total Credit Limit** | Primary | Card Portfolio *(supplementary)* | `SUM(cards.credit_limit_idr)` khusus `card_type = 'credit'` | Mengukur kapasitas kredit yang diberikan; kartu debit/prepaid tidak memiliki atribut limit sehingga dikecualikan. |
| **Credit Utilization Rate** | Primary | Card Portfolio *(supplementary)* | `Total Outstanding Credit ÷ Total Credit Limit × 100%` khusus `card_type = 'credit'` | Indikator risiko penggunaan fasilitas kredit kartu. |

### 3.3 KPI yang Sengaja Tidak Dipertahankan

- **Product per Customer** - dihapus dari dashboard karena murni mengukur *Customer Engagement*, tidak memiliki keterkaitan konsep sama sekali dengan 5 dimensi Financial Health (bukan Profitability, Liquidity, Asset Quality, Capital Adequacy, maupun Efficiency).
- **Total Transaction, Total Transaction Value, dsb.** - diklasifikasikan sebagai **Transaction Activity (supplementary)**, bukan Operational Efficiency, karena murni mengukur volume/nilai aktivitas, bukan rasio biaya atau produktivitas seperti CIR/Profit per Employee.

## 4. Karakteristik Data: Event vs Snapshot

Pemilihan jenis visualisasi pada dashboard ini mempertimbangkan karakteristik temporal data:

- **Event Data** - data yang mencatat kejadian/aktivitas pada waktu tertentu, sehingga setiap record merepresentasikan sebuah **peristiwa** (contoh: `transactions`, `loans.disbursement_date`). Data jenis ini valid untuk membangun **tren historis** (line/column chart per bulan).
- **Snapshot Data** - data yang menggambarkan kondisi objek pada satu titik waktu, sehingga setiap record merepresentasikan sebuah **keadaan**, bukan riwayat perubahan (contoh: `accounts.balance_idr`, `loans.outstanding_idr`). Data jenis ini **hanya boleh** ditampilkan sebagai Card (angka tunggal) atau breakdown kategori (bar/donut chart per status/jenis) - **tidak boleh** dipaksakan menjadi tren garis per bulan, karena akan menyesatkan pembaca seolah-olah itu representasi perubahan bisnis dari waktu ke waktu, padahal sebenarnya perbedaan itu berasal dari karakteristik cohort (misalnya bulan pencairan atau bulan pembukaan rekening), bukan pergerakan bisnis riil.

**Penerapan pada dashboard:**
- `Total Loan Disbursed` per bulan (`disbursement_date`) → **valid** sebagai tren, karena pencairan pinjaman adalah event.
- `Outstanding Loan`, `Total Deposit Balance` → ditampilkan sebagai **Card** dan **breakdown kategori** (by status, by account type), bukan tren garis.

## 5. Struktur Dashboard: Financial Health Overview

| Elemen | Jenis Visual | Sumber Data | Keterangan |
|---|---|---|---|
| 5 Card ringkasan | Card | Estimated Loan Interest Income, Outstanding Loan, Total Deposit Balance, Loan-to-Deposit Ratio, NPL Rate | Snapshot kondisi terkini per filter tahun |
| Total Loan Disbursed per Month | Line/Column chart | `loans.disbursement_date`, `loans.principal_idr` | Tren pencairan pinjaman bulanan (event data - valid) |
| Deposit Composition by Account Type | Donut chart | `accounts.account_type`, `accounts.balance_idr` | Breakdown tabungan/deposito/bisnis |
| Outstanding Balance vs Remaining Deposit | Donut chart (2 measure) | `Outstanding Balance`, `Remaining Deposit` | Visualisasi komposisi LDR |
| Loan Count by Status | Donut chart | `loans.status` | Jumlah pinjaman per kategori (active/paid_off/overdue/written_off) |
| Outstanding Balance by Loan Status | Bar chart | `loans.status`, `loans.outstanding_idr` | Nilai uang outstanding per kategori status |
| Slicer: Year | Slicer | Kolom Year (loans/accounts) | Filter periode analisis |

*Catatan: Dimensi Capital Adequacy dan Operational Efficiency tidak ditampilkan pada dashboard ini karena data modal, ATMR, biaya operasional, dan data karyawan tidak tersedia pada dataset.*

## 6. Hasil Dashboard: Perbandingan 2022–2024

| Metrik | 2022 | 2023 | 2024 |
|---|---|---|---|
| Estimated Loan Interest Income | Rp44.94bn | Rp51.53bn | Rp16.85bn |
| Outstanding Loan | Rp2.51bn | Rp2.92bn | Rp946.72M |
| Total Deposit Balance | Rp13.68bn | Rp12.57bn | Rp5.64bn |
| Loan-to-Deposit Ratio | 18.38% | 23.19% | 16.79% |
| Non-Performing Loan (NPL) | 16.21% | 21.06% | -  |
| Jumlah Pinjaman Aktif | 80 (62.02%) | 65 (57.52%) | 35 (60.34%) |
| Jumlah Pinjaman Overdue | 8 (6.20%) | 17 (15.04%) | 5 (8.62%) |
| Jumlah Pinjaman Written-off | 7 (5.43%) | 2 (1.77%) | ~1 (1.72%) |
| Jumlah Pinjaman Paid-off | 34 (26.36%) | 29 (25.66%) | 17 (29.31%) |

* Data 2024 hanya mencakup Januari–Mei karena rentang data pencairan pinjaman pada dataset berakhir pada Mei 2024. Hal ini bukan menunjukkan bahwa aktivitas bisnis berhenti, melainkan merupakan batas cakupan dataset.

* Card NPL tidak tertangkap pada tangkapan layar tahun 2024.

### Observasi

- **NPL meningkat tajam dari 2022 (16.21%) ke 2023 (21.06%)** - konsisten dengan naiknya proporsi pinjaman berstatus *overdue* (dari 6.20% ke 15.04% jumlah pinjaman). Ini sinyal memburuknya kualitas portofolio kredit yang perlu ditelusuri lebih lanjut (jenis pinjaman mana yang paling banyak menyumbang overdue).
- **Loan-to-Deposit Ratio naik dari 18.38% (2022) ke 23.19% (2023)**, menandakan porsi dana nasabah yang disalurkan sebagai kredit meningkat, meski secara absolut masih jauh dari ambang "agresif" (>95%).
- **Estimated Loan Interest Income naik dari 2022 ke 2023** (Rp44.94bn → Rp51.53bn) meski Outstanding Loan hanya naik sedikit (Rp2.51bn → Rp2.92bn) - mengindikasikan bauran pinjaman baru di 2023 kemungkinan memiliki suku bunga rata-rata lebih tinggi, atau volume pencairan baru lebih besar dibanding yang sudah dicicil/lunas.

### Catatan Penting

Angka **Total Deposit Balance** dan **Outstanding Loan** pada tabel di atas ter-filter oleh slicer **Year**, yang tertaut pada tahun **pencairan pinjaman** (`disbursement_date`) dan/atau **tahun pembukaan rekening** (`created_date`) - **bukan** representasi "total deposit/outstanding bank pada tahun tersebut secara keseluruhan". Artinya, angka Rp13.68bn di tahun 2022 adalah **saldo saat ini dari rekening-rekening yang dibuka pada 2022** (cohort), bukan "total simpanan bank pada tahun 2022". Interpretasi cohort ini penting untuk dipahami pembaca agar tidak disalahartikan sebagai laporan neraca tahunan resmi bank.

## 7. Limitasi Proyek

1. Dataset tidak menyediakan laporan keuangan bank secara lengkap (laporan laba rugi, posisi keuangan, data modal, laporan likuiditas), sehingga NIM, ROA, CAR, LCR, dan CIR tidak dapat dihitung secara langsung.
2. Beberapa indikator menggunakan pendekatan **Derived KPI** (Estimated Loan Interest Income) dan **Proxy KPI** (Loan-to-Deposit Ratio) yang tidak dimaksudkan sebagai pengganti rasio keuangan resmi.
3. Dataset terdiri atas *event data* (transaksi, registrasi nasabah, pencairan pinjaman) dan *snapshot data* (saldo rekening, outstanding pinjaman/kartu). Snapshot data tidak dapat digunakan untuk membangun tren historis resmi.
4. **Total Credit Limit** dan **Credit Utilization Rate** hanya dihitung untuk kartu bertipe *credit*, karena kartu debit dan prepaid tidak memiliki atribut limit kredit pada dataset.
5. Indikator **Customer Portfolio**, **Card Portfolio**, dan **Transaction Activity** bersifat *supplementary* - berada di luar cakupan lima dimensi Financial Health Framework, disertakan semata sebagai konteks operasional tambahan.
6. Dimensi **Capital Adequacy** dan **Operational Efficiency** sama sekali tidak dapat direpresentasikan pada dashboard ini.
7. Data 2024 hanya mencakup Januari–Mei; perbandingan lintas tahun untuk 2024 tidak merepresentasikan tahun penuh.
8. Angka Total Deposit Balance dan Outstanding Loan per tahun bersifat cohort-based (berdasarkan tahun pencairan/pembukaan), bukan snapshot neraca tahunan penuh bank - lihat Catatan Metodologis di Bagian 6.
9. Hasil analisis pada dashboard merepresentasikan kondisi berdasarkan dataset yang digunakan dan tidak dapat dianggap sebagai representasi penuh terhadap kondisi keuangan bank komersial sesungguhnya.

## 8. Tools

- **Microsoft Power BI** pengolahan data, DAX measures, dan visualisasi dashboard.
- **DAX Measures** yang digunakan tercantum lengkap pada Bagian 3.2.
- **SQL** yang digunakan untuk menjoin dataset

---

## 9. Struktur Dataset

| Tabel | Deskripsi | Kolom Kunci |
|---|---|---|
| `customers` | Data nasabah | `customer_id`, `registration_date` |
| `accounts` | Data rekening | `account_id`, `customer_id`, `account_type`, `balance_idr`, `created_date` |
| `transactions` | Data transaksi | `trx_id`, `account_id`, `trx_date`, `amount_idr`, `channel` |
| `loans` | Data pinjaman | `loan_id`, `customer_id`, `outstanding_idr`, `interest_rate_annual`, `status`, `disbursement_date` |
| `cards` | Data kartu | `card_id`, `customer_id`, `card_type`, `credit_limit_idr`, `outstanding_idr`, `status` |
