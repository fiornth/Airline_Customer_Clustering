# Airline Customer Clustering

## Data Description
Dataset : **Airline Customer Value Analysis Case**  
> Dataset ini berisi data customer sebuah perusahaan penerbangan dan beberapa fitur yang dapat menggambarkan value dari customer tersebut

Data ini terdiri dari `62988` baris dan `23` kolom

Feature : 
- MEMBER_NO : ID member
- FFP_DATE : Frequent flyer program join date
- FIRST_FLIGHT_DATE : Tanggal penerbangan pertama
- GENDER : Jenis kelamin
- FFP_TIER : Tier dari frequent flyer program
- WORK_CITY : Kota asal
- WORK_PROVINCE : Provinsi asal
- WORK_COUNTRY : Negara asal
- AGE : Umur customer
- LOAD_TIME : Tanggal data diambil
- FLIGHT_COUNT : Jumlah penerbangan customer
- BP_SUM : Rencana perjalanan
- SUM_YR_1 : Fare revenue
- SUM_YR_2 : Votes prices
- SEG_KM_SUM : Total jarak(km) penerbangan yang sudah dilakukan
- LAST_FLIGHT_DATE : Tanggal penerbangan terakhir
- LAST_TO_END : Jarak waktu penerbangan terakhir ke peesanan penerbangan paling akhir
- AVG_INTERVAL : Rata-rata jarak waktu
- MAX_INTERVAL : Maksimal jarak waktu
- EXCHANGE_COUNT : Jumlah penukaran
- avg_discount : Rata-rata discount yang didapat customer
- Points_Sum : Jumlah poin yang didapat customer
- Point_NotFlight : Point yang tidak digunakan oleh members

## EDA
### Missing dan Invalid Values
* **Handling Missing Values**  
Melakukan handling missing value pada kolom kategorikal seperti dengan nilai yang paling sering muncul (mode) dalam kolom. Dan mengisi missing value pada kolom numerik dengan nilai median karena memiliki positively skewed distribution.
* **Handling Invalid Values**  
    * Terdapat data yang tidak berada dalam range yang tidak tepat, yaitu avg_discount karena discount max hanya 1.0 atau 100%
    * Terdapat data yang tidak valid pada kolom `last_flight_date`, yaitu tanggal 29-02-2014. 2014 bukan tahun kabisat sehingga tidak ada tanggal 29-02. Oleh karena itu, selanjutnya akan diubah menjadi tanggal 28-02.
* **Ubah Tipe data**   
Mengubah tipe data pada kolom yang menginformasikan tanggal dan mengubah tipe data kolom AGE dari float menjadi int.

### Descriptive Analysis
#### Numerik
* Berdasarkan nilai statistik kolom numerik, mayoritas distribusi data bersifat positive skew dimana nilai mean > median. Kolom yang memiliki indikasi skew positif, yaitu:
    * flight_count
    * bp_sum
    * sum_yr_1
    * sum_yr_2
    * seg_km_sum
    * last_to_end
    * avg_interval
    * max_interval
    * points_sum
    * pont_notflight
    
* Kebanyakan distribusi pada kolom numerik merupakan skew positif, kecuali Member_no, ffp_tier, avg_discount Pada kolom avg_discount terdapat nilai yang tidak valid yaitu > 1.0. Karena maksimal discount = 1.0 atau 100%. Sehingga, avg_discount > 1.0 akan di drop

* Mayoritas kolom memiliki outlier, kecuali MEMBER_NO dan FFP_TIER

#### Kategorik
* Mayoritas kolom kategori, seperti work_city, work_province, work_country memiliki nilai unique yang sangat banyak sehingga feature kategori tersebut sepertinya akan di drop dan tidak digunakan dalam pemodelan. Sedangkan pada kolom gender yang memiliki 2 unique value yaitu male dan female, terdapat imbalance value yang didominasi oleh gender male. Sehingga kolom tersebut juga sepertinya tidak digunakan dalam pemodelan.

* Pada visualisasi menunjukkan bahwa :  
    * work_country didominasi oleh CN (China)
    * gender didominasi oleh male
    
Berdasarkan analisis tersebut, diputuskan bahwa kolom **kategori tidak ada yang digunakan untuk pemodelan**.


### Korelasi
Berdasarkan heatmap pertama, fitur yang memiliki nilai korelasi rendah dalam proses clustering ini akan didrop dari dataset, yaitu :
* MEMBER_NO
* AGE
* AVG_INTERVAL
* MAX_INTERVAL
* EXCHANGE_COUNT
* Point_NotFlight

Berdasarkan heatmap kedua, terdapat fitur yang berkorelasi sangat tinggi, yaitu BP_SUM, SUM_YR_1, SUM_YR_2, SEG_KM_SUM, POINT_SUMS. Untuk menghindari multikolinearitas, dalam modeling ini akan dipilih salah satu saja yaitu SEG_KM_SUM, sehingga sisanya yang akan di drop :
* BP_SUM
* SUM_YR_1
* SUM_YR_2
* POINTS_SUM

## Feature Engineering
### Feature Selection I
Feature dipilih menggunakan konsep LRMFC (Loyalty, Recency, Monetary, Frequency, Cabin) :
* Recency (R) : column `LAST_TO_END` dapat mewakili R
* Frequency (F) : column `FLIGHT_COUNT` dapat mewakili F
* Monetary (M) : column `SEG_KM_SUM` dapat mewakili M   
Jika biasanya monetary mewakili total atau average transactions value, dalam hal penerbangan total jarak (km) penerbangan yang sudah dilakukan juga dapat mewakili monetary.


Untuk proses analisis lebih lanjut, terutama mengenai konsep selanjutnya yaitu Loyalty dan Cabin dapat dilakukan feature extraction yang memanfaatkan kolom
* `LOAD_TIME`
* `FFP_DATE`
* `avg_discount`

### Feature Extraction
Terdapat penambahan feature yang memanfaatkan 5 kolom yang sudah ada, yaitu:
* Membuat kolom `Member_Duration` -> dari kolom `LOAD_TIME` dan `FFP_DATE`   
Pengurangan kolom tersebut akan memberikan informasi mengenai durasi atau lamanya penumpang menjadi member.

* Membuat kolom `Flight_Count_perYear` -> dari kolom `LAST_FLIGHT_DATE` dan `FIRST_FLIGHT_DATE`   
Pengurangan kolom tersebut untuk menghitung jumlah tahun penumpang melakukan penerbangan. Kemudian dari kolom `FLIGHT_COUNT` yang sudah ada dibagi dengan jumlah tahun tersebut yang hasilnya akan memberikan informasi mengenai rata-rata frekuensi penerbangan yang dilakukan per tahunnya.


### Feature Selection II
Berdasarkan analisis yang telah dilakukan, diputuskan bahwa feature yang digunakan adalah:
* LAST_TO_END
* Flight_Count_perYear
* SEG_KM_SUM
* Member_Duration
* avg_discount

### Handling Outlier 
Outlier dihandle atau diatasi menggunakan metode IQR.

### Feature Transformation
Melakukan **standardisasi** kepada setiap feature agar memiliki skala yang sama.

## Modeling
Melakukan pemodelan menggunakan model **K-Means dengan clustering = 4**. Hasil clustering tersebut didapatkan berdasarkan elbow method.

## Interpretasi Hasil Clustering
**Deskripsi tiap cluster:**  
1. **Cluster 0** merupakan pelanggan dengan member duration yang lama (rata-rata 81 bulan), sering melakukan penerbangan jarak jauh mengingat rata-rata total jarak penerbangan yang cukup tinggi (rata-rata 13698 km) dan jumlah penerbangan yang rendah (rata-rata 1 kali), cenderung sering melakukan penerbangan internasional. (**Pelanggan setia yang melakukan perjalanan internasional**)
2. **Cluster 1** merupakan pelanggan dengan member duration yang sedang (rata-rata 41 bulan), memiliki total jarak penerbangan dan jumlah penerbangan yang rendah (rata-rata 5436 km dan rata-rata 1 kali), serta memiliki jarak waktu penerbangan terakhir ke pesanan penerbangan paling akhir yang paling tinggi (rata-rata 442 hari). Pelanggan pada cluster ini perlu diberikan suatu treatment agar lebih banyak melakukan perjalanan dan menurunkan resiko untuk meninggalkan layanan. (**Pelanggan potensial yang memiliki kecenderungan tinggi meninggalkan layanan atau jarang melakukan kegiatan penerbangan)**)
3. **Cluster 2** merupakan pelanggan dengan member duration yang cukup rendah (rata-rata 34 bulan), cenderung sering melakukan penerbangan jarak yang lebih dekat mengingat jumlah penerbangan per tahunnya yang tinggi (rata-rata 7 kali) dengan total jarak penerbangan yang cukup tinggi (rata-rata 20150 km) atau cenderung sering melakukan penerbangan domestik, serta memiliki jarak waktu penerbangan terakhir ke pesanan penerbangan paling akhir yang rendah. (**Pelanggan aktif yang sering melakukan perjalanan domestik**)
4. **Cluster 3** merupakan pelanggan yang cukup baru atau member duration yang rendah (rata-rata 31 bulan), memiliki total jarak penerbangan yang sedang dan jumlah penerbangan yang  rendah. (**Pelanggan baru dengan kegiatan penerbangan yang rendah**)

**Rekomendasi Strategi Bisnis:**
* Memberikan promo atau penawaran khusus baik kepada member lama atau baru sebagai bentuk apresiasi kepada mereka (program loyalitas). Banyaknya promo atau jenis penawaran khusus yang diberikan disesuaikan dengan karakteristik mereka, seperti jika member tersebut suka melakukan penerbangan dengan jarak yang jauh maka berikan promo atau penawran khusus untuk jarak penerbangan jauh, begitu pula dengan jarak penerbangan yang sedang atau dekat untuk mendorong mereka melakukan lebih banyak penerbangan dan meningkatkan loyalitas.
* Meningkatkan kualitas pelayanan dan berikan inovasi pemasaran seperti penawaran paket perjalanan atau yang lain sesuai segmentasinya untuk mempertahankan pelanggan dan menarik pelanggan baru.
* Meningkatkan interaksi atau komunikasi dengan pelanggan yang jarang melakukan aktivitas melalui suatu kampanye pemasaran berbasis email atau direct message untuk menjaga mereka terlibat dan terhubung dengan layanan.
