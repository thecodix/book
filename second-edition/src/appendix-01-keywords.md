## Lampiran A: Kata kunci

Kata kunci berikut dicadangkan oleh bahasa Rust dan mungkin tidak digunakan sebagai pengidentifikasi seperti nama fungsi, variabel, parameter, bidang struct, modul, crate, konstanta, makro, nilai statis, atribut, jenis, sifat, atau lifetime.

### Kata Kunci yang digunakan Saat ini

* `as` - casting primitif, disambiguasi sifat spesifik yang mengandung item, atau mengganti nama item pada pernyataan `use` dan `extern crate`
* `break` - keluar dari pengulangan segera
* `const` - item konstan dan pointer mentah konstan
* `continue` - lanjutkan ke iterasi pengulangan berikutnya
* `crate` - keterkaitan crate eksternal atau variabel makro mewakili crate
  dimana makro didefinisikan
* `else` - posisi mundur untuk aliran kontrol konstruksi `if` dan `if let`
* `enum` - mendefinisikan sebuah enumerasi
* `extern` - eksternal crate, fungsi, dan keterkaitan variabel
* `false` - literal false Boolean
* `fn` - definisi fungsi dan tipe fungsi pointer
* `for` - iterasi pengulangan, bagian dari sifat sintaks impl, dan rangking tertinggi 
  sintaks lifetime
* `if` - percabangan bersyarat
* `impl` - implementasi blok inheren dan sifat
* `in` - bagian dari sintaks pengulangan `for`
* `let` - variabel terikat
* `loop` - Pengulangan tak bersyarat, tanpa batas
* `match` - pencocokan pola
* `mod` - deklarasi modul
* `move` - membuat penutupan untuk mengambil alih kepemilikan semua tangkapannya
* `mut` - menunjukkan mutabilitas dalam referensi, pointer mentah, dan pola terikat
* `pub` - menunjukkan visibilitas publik di bidang struct, blok `impl`, dan modul
* `ref` - dengan referensi mengikat
* `return` - kembali ke fungsi
* `Self` - tipe alias untuk tipe yang menerapkan suatu sifat
* `self` - metode subjek atau modul saat ini
* `static` - variable umum or lifetime yang berlangsung sepanjang eksekusi program
* `struct` - definisi stuktur
* `super` - modul induk dari modul saat ini
* `trait` - definisi sifat
* `true` - literal true Boolean
* `type` - tipe alias dan tipe definisi yang terkait
* `unsafe` - menunjukkan kode, fungsi, sifat, dan implementasi yang tidak aman
* `use` - mengimpor simbol ke dalam bidang
* `where` - jenis kendala klausa
* `while` - pengulangan bersyarat

### Kata kunci yang Dicadangkan untuk Penggunaan Masa Depan

Kata kunci ini tidak mempunyai fungsi apapun, namun dicadangkan oleh Rust untuk
penggunaan masa depan yang potensial.

* `abstract`
* `alignof`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `offsetof`
* `override`
* `priv`
* `proc`
* `pure`
* `sizeof`
* `typeof`
* `unsized`
* `virtual`
* `yield`
