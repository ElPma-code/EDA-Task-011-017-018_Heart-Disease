# Filtering noise and outliers

**Filter sinyal tidak cocok di sini.** Moving average, median filter, Hampel filter, dan
Savitzky-Golay semuanya mengasumsikan urutan data punya makna, seperti pada deret waktu
atau sinyal sensor. Pada dataset ini sumbu x line chart hanya nomor baris, dan urutannya
arbitrer. Menghaluskan nilai baris ke-100 memakai baris ke-99 dan ke-101 berarti mencampur
data tiga pasien yang tidak berhubungan. Klaim ini dibuktikan secara empiris di Step 2 lewat
uji autokorelasi.

**Catatan tentang dua angka "pergeseran korelasi" yang berbeda**

Angka ini muncul dua kali di notebook dengan nilai berbeda karena diukur pada tahap yang berbeda,
dan keduanya kebetulan terjadi pada fitur yang sama (`ca`):

| Perbandingan                          | Tahap                         | Pergeseran terbesar            |
| ------------------------------------- | ----------------------------- | ------------------------------ |
| sebelum vs sesudah dedup (STEP 1)     | dedup saja                    | 0.027                          |
| mentah vs bersih (perbandingan akhir) | dedup + imputasi kode invalid | 0.082 (`ca`: -0.382 -> -0.464) |

Jadi sebagian besar kenaikan sinyal `ca` datang dari pembersihan kode invalid, bukan dari deduplikasi.

**PERINGATAN: arah label `target` patut dicurigai terbalik**

Semua kesimpulan di bawah ditulis dengan asumsi `target` = 1 berarti sakit jantung, mengikuti
nama kolomnya. Tabel korelasi akhir justru menguatkan kecurigaan di bagian _notes: baris mana yang
benar-benar perlu difilter_: `ca` (-0.464), `thal` (-0.362), `exang` (-0.436), dan `oldpeak` (-0.429)
semuanya **negatif** terhadap target, sedangkan `thalach` (+0.420) dan `cp` (+0.432) **positif**.
Secara klinis seharusnya kebalikannya, karena pembuluh tersumbat, defek reversibel, angina saat
olahraga, dan depresi ST adalah penanda penyakit.

Kalau dugaan ini benar, **besaran** di poin 1-7 tetap sah seluruhnya, yang berubah hanya
**arah maknanya**: poin 2, 3, dan 6 harus dibaca terbalik. Konfirmasikan dulu ke berkas UCI asli
(`processed.cleveland.data`) sebelum menulis interpretasi di makalah.

**Konsekuensinya untuk tahap berikutnya**

Karena tidak ada satu pun fitur yang dominan dan tidak ada yang redundan, ini kasus di mana
banyak fitur lemah harus dipakai bersama-sama. Membuang fitur berdasarkan korelasi rendah
justru merugikan, kecuali `fbs` yang memang nol.

Satu catatan praktis: dengan hanya 302 baris unik dari 1025, ukuran sampel efektif jauh lebih kecil
daripada yang terlihat. Pemisahan train/test wajib dilakukan **setelah** deduplikasi, kalau tidak
baris yang sama bisa muncul di kedua sisi dan membuat skor evaluasi tampak jauh lebih baik
daripada kenyataannya.
