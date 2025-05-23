# Laporan Proyek Machine Learning - Fika Saputri
### Domain Proyek
Dalam era digital saat ini, jumlah film yang tersedia secara online sangat banyak dan terus bertambah. Hal ini menyulitkan pengguna dalam menemukan film yang sesuai dengan minat mereka. Untuk mengatasi masalah tersebut, sistem rekomendasi hadir sebagai solusi penting, khususnya dalam membantu pengguna menemukan konten yang relevan.

Pada proyek ini, dibangun sistem rekomendasi film berbasis Content-Based Filtering, yang memanfaatkan teknik TF-IDF (Term Frequency-Inverse Document Frequency) dan Cosine Similarity untuk menghitung kemiripan antar film berdasarkan deskripsi konten (overview, genre, aktor, sutradara, dan kata kunci lainnya).

Studi dari Ricci et al. (2011) menunjukkan bahwa content-based filtering sangat efektif dalam memberikan rekomendasi personal dengan mempertimbangkan preferensi pengguna sebelumnya [1]. Selain itu, meningkatnya jumlah pengguna platform streaming seperti Netflix dan Disney+ menunjukkan bahwa personalisasi konten adalah kebutuhan utama.

### Business Understanding
#### Problem Statements
- Bagaimana memberikan rekomendasi film yang mirip berdasarkan konten film seperti genre, sinopsis, dan tokoh?
- Bagaimana mengukur relevansi antar film untuk meningkatkan pengalaman pengguna?
#### Goals
- Mengembangkan sistem rekomendasi berbasis konten yang dapat memberikan rekomendasi film serupa terhadap film yang ditentukan pengguna.
- Menggunakan metode TF-IDF dan Cosine Similarity untuk mengukur kemiripan antar film berdasarkan teks fitur (overview, genre, keyword, cast, crew).
#### Solution Statements
Menggunakan pendekatan Content-Based Filtering yang berfokus pada fitur film itu sendiri, bukan interaksi pengguna.

###### Solution Approaches:
- Content-Based Filtering: Metode ini memanfaatkan atribut dari film seperti genre, sinopsis, aktor, dan sutradara untuk memberikan rekomendasi berdasarkan kemiripan konten dengan film yang disukai pengguna.

- Collaborative Filtering: Alternatif pendekatan ini memanfaatkan data interaksi pengguna seperti rating atau histori penayangan untuk merekomendasikan film yang disukai pengguna lain dengan preferensi serupa. Namun, pendekatan ini memerlukan data interaksi pengguna yang tidak tersedia dalam dataset TMDB, sehingga tidak digunakan pada proyek ini.

### Data Understanding
Dataset yang digunakan adalah TMDB 5000 Movies Dataset, terdiri dari dua file:
- tmdb_5000_movies.csv: berisi informasi film (judul, genre, sinopsis, popularitas, vote average, dll.)
- tmdb_5000_credits.csv: berisi informasi pemain dan kru film (cast, crew)
Sumber: Kaggle - https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata
Jumlah data: 4803 film

###### Fitur utama:
- genres: daftar genre film
- overview: deskripsi film
- keywords: kata kunci terkait
- cast: daftar pemeran utama
- crew: daftar tim produksi, termasuk sutradara
- runtime: durasi film dalam menit
- release_date: tanggal rilis film

Setelah digabungkan dan dibersihkan, informasi difokuskan pada fitur teks yang dirangkum dalam satu kolom tags.

## pemahaman data:
- Melihat struktur data (head, info)
- Mengecek missing values
- Melakukan join antara dataset movies_df dan credits_df berdasarkan kolom id
- Visualisasi awal: genre paling umum, distribusi durasi film, tren film per tahun
#### EDA dan Insight:
- Genre paling umum: Drama, Action, Comedy
- Aktor yang sering muncul: Robert Downey Jr., Tom Hanks
- Banyak film memiliki overview kosong atau ringkasSebaran durasi film bervariasi antara 30 hingga 200 menit

### Data Preparation
##### Langkah-langkah:
###### Data Cleaning:
Tahap ini fokus pada persiapan data untuk pemodelan:
- Menghapus kolom yang tidak relevan (homepage).
- Mengisi missing value pada kolom tekstual (overview, tagline) dengan string kosong.
- Menghapus baris dengan missing value pada release_date.
- Mengisi missing value pada runtime dengan nilai median.
- Mengekstrak informasi relevan (nama genre, kata kunci, pemeran, sutradara) dari kolom dengan format mirip JSON string.
- Membersihkan data yang diekstrak (menghilangkan spasi dan mengubah ke huruf kecil).
- Menggabungkan semua fitur yang relevan (genres, keywords, cast, crew, overview) menjadi satu kolom baru bernama tags.
- tags, serta menghapus duplikat berdasarkan judul film.
- Data tags yang telah disiapkan perlu diubah menjadi format numerik agar bisa diukur kesamaannya:
- Menggunakan TfidfVectorizer untuk mengubah teks pada kolom tags menjadi matriks representasi TF-IDF. Ini dilakukan dengan membatasi jumlah fitur (kata) dan menghapus stop words bahasa Inggris.
- Menghitung matriks kesamaan (similarity matrix) antar semua film berdasarkan matriks TF-IDF menggunakan cosine_similarity. Matriks ini menunjukkan seberapa mirip setiap film dengan film lainnya.
### Modeling
###### Pendekatan Content-Based Filtering
- Content-Based Filtering bekerja dengan cara:
    - Menyusun representasi numerik dari konten film (dalam hal ini, fitur tags).
    - Mengukur kemiripan antar film menggunakan cosine similarity.
    - Ketika pengguna memilih sebuah film, sistem akan mencari film lain dengan nilai similarity tertinggi.

- Kelebihan:
    - Tidak memerlukan data interaksi pengguna lain.
    - Dapat memberikan rekomendasi untuk item-item baru.
- Kekurangan:
    - Terbatas pada konten yang tersedia.
    - Tidak bisa menangkap selera pengguna berdasarkan komunitas.
   




### Evaluation
#### Metrik Evaluasi yang digunakan:
Karena sistem rekomendasi ini menggunakan pendekatan content-based filtering tanpa data interaksi pengguna (seperti rating atau klik), maka metrik evaluasi yang digunakan bersifat kualitatif dan kuantitatif manual. Namun untuk memperkuat analisis, proyek ini juga menggunakan metrik Cosine Similarity Score sebagai dasar evaluasi kuantitatif internal antar film.

Cosine Similarity:
Cosine similarity mengukur kesamaan antar dua vektor dalam ruang vektor berdasarkan sudut di antara mereka. Formula cosine similarity:

            cosine_similarity(A, B) = (A * B) / (||A|| * ||B||)
Nilai cosine similarity berada pada rentang 0 hingga 1, di mana nilai yang lebih tinggi menunjukkan tingkat kemiripan yang lebih besar.

Untuk mengevaluasi relevansi rekomendasi secara konten, dilakukan peninjauan terhadap hasil sistem rekomendasi dari film The Dark Knight. Berikut adalah 5 film yang direkomendasikan beserta nilai cosine similarity-nya:

| Rekomendasi Film            | Cosine Similarity |
| --------------------------- | ----------------- |
| The Dark Knight Rises       | 0.4447            |
| Batman Returns              | 0.3714            |
| Batman Begins               | 0.3497            |
| In the Name of the King III | 0.2955            |
| Batman Forever              | 0.2881            |

Analisis:
- Tiga dari lima film merupakan bagian dari franchise Batman, dengan karakter dan tone yang sangat mirip.
- The Dark Knight Rises adalah sekuel langsung dari The Dark Knight, sehingga tingkat kemiripan tinggi (0.44) sangat relevan.
- In the Name of the King III memiliki similarity yang lebih rendah, kemungkinan karena overlap pada kata kunci aksi atau petualangan, meskipun berbeda secara naratif.

Proyek ini berhasil membangun sistem rekomendasi film berbasis konten dengan pendekatan TF-IDF dan cosine similarity. Sistem ini efektif dalam mengenali kesamaan antar film berdasarkan fitur deskriptif seperti genre, sutradara, aktor, dan overview film. Model dapat digunakan sebagai pondasi awal sebelum integrasi dengan pendekatan collaborative filtering di masa depan.




