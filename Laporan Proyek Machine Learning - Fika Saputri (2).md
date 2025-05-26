# Laporan Proyek Machine Learning - Fika Saputri
### Domain Proyek
Dalam era digital saat ini, jumlah film yang tersedia secara online sangat banyak dan terus bertambah. Hal ini menyulitkan pengguna dalam menemukan film yang sesuai dengan minat mereka. Untuk mengatasi masalah tersebut, sistem rekomendasi hadir sebagai solusi penting, khususnya dalam membantu pengguna menemukan konten yang relevan.

Pada proyek ini, dibangun sistem rekomendasi film berbasis Content-Based Filtering, yang memanfaatkan teknik TF-IDF (Term Frequency-Inverse Document Frequency) dan Cosine Similarity untuk menghitung kemiripan antar film berdasarkan deskripsi konten (overview, genre, aktor, sutradara, dan kata kunci lainnya).

Studi dari Ricci et al. (2011) menunjukkan bahwa content-based filtering sangat efektif dalam memberikan rekomendasi personal dengan mempertimbangkan preferensi pengguna sebelumnya [1]. Selain itu, meningkatnya jumlah pengguna platform streaming seperti Netflix dan Disney+ menunjukkan bahwa personalisasi konten adalah kebutuhan utama.

### Business Understanding
#### Problem Statements
Pengguna kesulitan menemukan film relevan di tengah banyak pilihan, kurang personalisasi mengurangi pengalaman.
#### Goals
- Membangun sistem rekomendasi film berbasis konten yang dapat menyarankan film serupa kepada pengguna berdasarkan film yang mereka sukai.
- Meningkatkan kepuasan pengguna dengan menyediakan rekomendasi film yang relevan dan sesuai dengan preferensi mereka.
- Mengevaluasi kualitas sistem rekomendasi untuk mengukur seberapa baik saran film yang diberikan dalam memenuhi preferensi pengguna.
#### Solution Statements
Menggunakan pendekatan Content-Based Filtering yang berfokus pada fitur film itu sendiri, bukan interaksi pengguna.

###### Solution Approaches:
Proyek ini akan menggunakan pendekatan:
- Content-Based Filtering, yang berfokus pada fitur film seperti genre, overview, aktor, dan sutradara.
- Tidak menggunakan Collaborative Filtering, karena dataset tidak menyediakan data interaksi pengguna seperti rating atau histori tontonan.

### Data Understanding
Proyek ini menggunakan dataset film dari [Kaggle: TMDB 5000 Movie Dataset](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata). Dataset ini berisi informasi metadata film termasuk genre, aktor, sutradara, deskripsi film, dan popularitas. Data ini akan digunakan untuk membangun sistem rekomendasi berbasis konten (Content-Based Filtering).

Dataset terdiri dari dua file utama, yaitu `tmdb_5000_movies.csv` dan `tmdb_5000_credits.csv`. Berikut penjelasan rinci dari masing-masing file:

##### 1. Dataset: tmdb_5000_movies.csv

###### Jumlah Data
- Jumlah baris: **4.803**
- Jumlah kolom: **20**

#####  Kondisi Data
- Terdapat beberapa **missing value** pada kolom: `overview`, `tagline`, `release_date`, dan `runtime`.
- Beberapa nilai `overview` sangat pendek atau kosong.
- Tidak ditemukan **data duplikat** berdasarkan kolom `title`.
- **Outlier** tidak dieksplorasi secara khusus karena fokus fitur berupa teks (tidak numerikal).

#### Uraian Fitur


| Kolom         | Deskripsi                                               |
|---------------|----------------------------------------------------------|
| `id`          | ID unik untuk masing-masing film                        |
| `title`       | Judul film                                              |
| `genres`      | Daftar genre film dalam format string JSON              |
| `overview`    | Ringkasan/deskripsi singkat film                        |
| `keywords`    | Kata kunci terkait isi film                             |
| `tagline`     | Kalimat promosi film                                    |
| `runtime`     | Durasi film dalam satuan menit                          |
| `release_date`| Tanggal rilis film                                      |
| `vote_average`| Rata-rata skor rating dari pengguna                     |
| `popularity`  | Skor popularitas film berdasarkan algoritma TMDB        |
| `homepage`    | Website resmi film (banyak yang kosong)                 |

##### 2. Dataset: tmdb_5000_credits.csv
###### Jumlah Data
- Jumlah baris: **4.803**
- Jumlah kolom: **4**

#### Kondisi Data
- Tidak ditemukan missing value.
- Data dalam kolom `cast` dan `crew` disimpan dalam format string JSON.
- Tidak ditemukan duplikat pada kolom `movie_id` atau `title`.

#### Uraian Fitur
| Kolom       | Deskripsi                                                    |
|-------------|---------------------------------------------------------------|
| `movie_id`  | ID film yang dapat digunakan untuk join ke dataset utama     |
| `title`     | Judul film                                                   |
| `cast`      | Daftar aktor/aktris dalam film (string JSON)                 |
| `crew`      | Daftar kru film termasuk sutradara (string JSON)            |


### Data Preparation
Sebelum dilakukan pembersihan data, dua dataset utama yaitu tmdb_5000_movies.csv dan tmdb_5000_credits.csv terlebih dahulu digabung (merge) berdasarkan kolom kunci id dan movie_id.
Langkah ini bertujuan untuk menggabungkan informasi film, seperti judul, sinopsis, genre, dengan informasi tambahan seperti daftar pemeran (cast) dan kru (crew), sehingga semua informasi penting tersedia dalam satu dataframe movies_df.
###### Data Cleaning:
Tahap ini fokus pada persiapan data untuk pemodelan:
- Menghapus kolom yang tidak dibutuhkan dalam analisis, seperti homepage.
- Mengisi missing value pada kolom teks (overview, tagline) dengan string kosong ("") agar tidak error saat proses penggabungan teks.
- Menghapus baris yang memiliki nilai kosong pada release_date karena jumlahnya sedikit dan tidak memungkinkan dilakukan imputasi yang valid.
- Menghapus duplikat berdasarkan judul film untuk memastikan data unik.
###### Data Transformation:
- Data dalam beberapa kolom memiliki format mirip JSON string. Oleh karena itu, dilakukan parsing dan ekstraksi informasi penting menggunakan fungsi bantu khusus:
- Kolom genres dan keywords: diubah dari format JSON string menjadi list berisi nama-nama genre/kata kunci.
- Kolom cast: diambil maksimal 3 pemeran utama.
- Kolom crew: diambil nama sutradara (yang memiliki atribut job = Director).
Setelah itu, dilakukan pembersihan dengan:
- Menghapus spasi dan mengubah semua huruf menjadi huruf kecil untuk menjaga konsistensi teks.
- Membersihkan daftar kata agar dapat digunakan sebagai input fitur teks secara efisien.
Semua fitur tekstual yang relevan kemudian digabungkan ke dalam satu kolom baru bernama tags. Kolom ini menggabungkan:
- Genre
- Keywords
- Nama pemeran utama
- Nama sutradara
- Sinopsis (overview)
Dataset hasil preprocessing kemudian disusun ulang agar hanya menyisakan kolom yang diperlukan untuk proses pemodelan, yaitu:
- id: identitas unik film
- title: judul film
- tags: gabungan fitur relevan yang telah dibersihkan
###### Feature Encoding;
Untuk mengubah fitur teks tags menjadi representasi numerik yang bisa diproses oleh algoritma machine learning, digunakan teknik TF-IDF (Term Frequency–Inverse Document Frequency). TF-IDF memberikan bobot pada kata-kata yang penting di setiap dokumen (film), tetapi tidak terlalu umum di seluruh dataset.


 
### Modeling
###### Pendekatan Content-Based Filtering
Content-Based Filtering adalah skema rekomendasi yang merekomendasikan item (dalam kasus ini, film) kepada user berdasarkan kesamaan fitur atau atribut dari item yang disukai user di masa lalu. Sistem ini membangun profil user berdasarkan preferensi item yang pernah mereka konsumsi dan kemudian merekomendasikan item baru yang memiliki fitur serupa dengan item di profil user.
- Content-Based Filtering bekerja dengan cara:
    - Representasi Item: Setiap item direpresentasikan sebagai vektor fitur berdasarkan kontennya (misalnya, genre, kata kunci, sinopsis, pemeran, sutradara).Dengan menggabungkan semua fitur relevan ke dalam kolom 'tags' dan menggunakan teknik TF-IDF (Term Frequency-Inverse Document Frequency) untuk mengubah teks di kolom 'tags' menjadi vektor numerik. TF-IDF memberikan bobot pada kata-kata berdasarkan seberapa sering muncul dalam satu dokumen (film) dibandingkan seberapa sering muncul di semua dokumen, sehingga kata-kata yang lebih unik dan deskriptif mendapatkan bobot lebih tinggi.
    - Mengukur Kesamaan Item: Setelah item direpresentasikan sebagai vektor, kesamaan antar item diukur.Dengan menggunakan Cosine Similarity. Cosine Similarity mengukur sudut antara dua vektor; semakin kecil sudutnya (semakin mendekati 1), semakin mirip kedua item tersebut. Matriks kesamaan (similarity matrix) dihitung untuk semua pasangan film berdasarkan vektor TF-IDF mereka.
    - Generasi Rekomendasi: Ketika seorang user memilih sebuah film, sistem mencari film lain yang memiliki skor kesamaan tertinggi dengan film yang dipilih user berdasarkan matriks kesamaan. Film-film dengan skor kesamaan tertinggi (selain film yang dipilih itu sendiri) kemudian direkomendasikan.
- Algoritma yang Digunakan:
    - TF-IDF (Term Frequency-Inverse Document Frequency): Digunakan untuk mengubah data tekstual ('tags') menjadi representasi vektor numerik.
- Kelebihan Content-Based Filtering:
    - Tidak memerlukan data interaksi pengguna lain, sehingga bisa digunakan pada data film baru tanpa riwayat rating.
    - Dapat memberikan rekomendasi yang sangat spesifik berdasarkan fitur konten film.
- Kekurangan Content-Based Filtering:
    -    Terbatas hanya pada fitur konten yang tersedia.
    -    Tidak dapat menangkap preferensi atau selera berdasarkan pola komunitas pengguna (cold-start user problem tetap ada).

###### Hasil Rekomendasi (Top N)
Berikut adalah contoh hasil top 5 rekomendasi dari sistem Content-Based Filtering untuk film "The Dark Knight":
Rekomendasi untuk film 'The Dark Knight':
1. The Dark Knight Rises (Similarity: 0.4447)
2. Batman Returns (Similarity: 0.3714)
3. Batman Begins (Similarity: 0.3497)
4. Batman: The Dark Knight Returns, Part 2 (Similarity: 0.2955)
5. Batman Forever (Similarity: 0.2881)

Rata-rata skor similarity dari rekomendasi: 0.3499
Evaluasi: Rekomendasi kurang relevan (similarity <= 0.5).

### Evaluation
#### Metrik Evaluasi yang digunakan:
Dalam proyek ini, sistem rekomendasi film berbasis content-based filtering dikembangkan untuk membantu pengguna yang kesulitan menemukan film relevan di tengah banyak pilihan dan untuk mengatasi kurangnya personalisasi yang dapat mengurangi pengalaman pengguna. Oleh karena itu, metrik evaluasi yang dipilih adalah Precision@N, yang mengukur proporsi rekomendasi yang relevan di antara N rekomendasi teratas. Precision@N sangat relevan karena dapat menunjukkan seberapa efektif sistem dalam menyajikan film yang sesuai dengan preferensi pengguna berdasarkan film yang sudah mereka sukai.

Cosine similarity tetap digunakan sebagai metode untuk mengukur kemiripan antar film dan menghasilkan daftar rekomendasi, namun cosine similarity bukan metrik evaluasi melainkan bagian dari proses rekomendasi.
Precision@N mengukur proporsi rekomendasi yang relevan dalam N rekomendasi teratas. Metrik ini relevan karena:
- Sistem ini menggunakan konten film (fitur tags) tanpa data interaksi pengguna seperti - rating atau klik.
- Precision dapat mengukur efektivitas model dalam menghasilkan rekomendasi yang relevan secara kuantitatif.
- Precision@N membantu memahami seberapa baik sistem merekomendasikan item yang sesuai dalam daftar rekomendasi teratas, yang penting untuk pengalaman pengguna.

Sebagai dasar perhitungan kemiripan antar film tetap digunakan Cosine Similarity untuk menghasilkan skor similarity antar film, tetapi cosine similarity bukanlah metrik evaluasi model, melainkan bagian dari mekanisme rekomendasi.

Metrik Evaluasi:
- Menggunakan Precision@N (Precision@5) untuk mengukur proporsi rekomendasi relevan dalam 5 rekomendasi teratas.
- Threshold similarity 0.3 digunakan untuk menentukan relevansi film

#### Hasil Evaluasi Precision@N
Pengujian Precision@5 dilakukan pada beberapa film contoh, dengan threshold skor similarity 0.3 untuk menentukan relevansi film rekomendasi. Misalnya untuk film The Dark Knight, hasilnya adalah:
| Film Input      | Precision\@5 | Film Relevan di Top 5 Rekomendasi                    |
| --------------- | ------------ | ---------------------------------------------------- |
| The Dark Knight | 0.60         | The Dark Knight Rises, Batman Returns, Batman Begins |
- Precision@5 = 0.60, artinya 3 dari 5 rekomendasi yang diberikan relevan dengan preferensi pengguna.
- Film-film yang direkomendasikan seperti The Dark Knight Rises, Batman Returns, dan Batman Begins merupakan film serupa dengan genre, karakter, dan cerita yang sejalan, sehingga meningkatkan kemungkinan kepuasan pengguna.

##### Analisis dan Perbandingan Skema Model
Saat ini, sistem masih menggunakan pendekatan content-based filtering saja. Meskipun belum menerapkan collaborative filtering, hasil evaluasi menunjukkan bahwa sistem mampu memberikan rekomendasi film yang relevan dan membantu pengguna menemukan film serupa dengan yang mereka sukai.
Ke depannya, penggabungan metode collaborative filtering dapat dipertimbangkan untuk meningkatkan personalisasi rekomendasi berdasarkan interaksi dan preferensi komunitas pengguna.

##### Hubungan dengan Business Understanding dan Goals
Sistem rekomendasi ini secara langsung menjawab problem statement dengan:
- Membantu pengguna menemukan film yang relevan di tengah banyak pilihan dengan memberikan rekomendasi berdasarkan kesamaan konten.
- Meningkatkan kepuasan pengguna dengan menyediakan daftar film yang relevan dan sesuai preferensi mereka, yang diukur dengan Precision@N.
- Metrik evaluasi yang digunakan membantu memvalidasi kualitas rekomendasi sehingga tujuan evaluasi model tercapai.

Dengan hasil evaluasi Precision@5 yang cukup tinggi untuk beberapa film, sistem ini menunjukkan keberhasilan dalam memenuhi goals yang diharapkan, yaitu membangun sistem rekomendasi berbasis konten yang efektif dan relevan. Walaupun demikian, sistem masih memiliki keterbatasan pada personalisasi yang lebih dalam, yang dapat diatasi dengan pengembangan selanjutnya.

### Kesimpulan
- Sistem rekomendasi berhasil dibangun menggunakan pendekatan Content-Based Filtering dengan teknik TF-IDF dan Cosine Similarity.
- Sistem mampu merekomendasikan film serupa berdasarkan konten, namun tingkat relevansinya masih terbatas pada fitur yang tersedia.

### Saran Pengembangan
- Menggabungkan pendekatan Hybrid dengan Collaborative Filtering untuk meningkatkan akurasi.
- Menambahkan data interaksi pengguna (jika tersedia) agar model lebih personal.
- Menggunakan metode NLP yang lebih canggih seperti Word2Vec atau BERT untuk memahami konteks teks.



