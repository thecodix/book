# Bahasa Pemrograman Rust

## Permulaan

- [Pengenalan](ch01-00-introduction.md)
    - [Pemasangan](ch01-01-installation.md)
    - [Halo, Dunia!](ch01-02-hello-world.md)
    - [Bagaimana Rust dan “Nightly Rust” dibuat](ch01-03-how-rust-is-made-and-nightly-rust.md)

- [Tutorial Permainan Tebakan](ch02-00-guessing-game-tutorial.md)

- [Konsep Pemrograman Biasa](ch03-00-common-programming-concepts.md)
    - [Variabel dan Mutability](ch03-01-variables-and-mutability.md)
    - [Tipe Data](ch03-02-data-types.md)
    - [Bagaimana Fungsi Bekerja](ch03-03-how-functions-work.md)
    - [Komentar](ch03-04-comments.md)
    - [Alur Kontrol](ch03-05-control-flow.md)

- [Memahami Kepemilikan](ch04-00-understanding-ownership.md)
    - [Apa itu Kepemilikan?](ch04-01-what-is-ownership.md)
    - [Referensi & Meminjam](ch04-02-references-and-borrowing.md)
    - [Irisan](ch04-03-slices.md)

- [Menggunakan Structs to Structure Related Data](ch05-00-structs.md)
    - [Mendefinisikan dan Instantiating Structs](ch05-01-defining-structs.md)
    - [Contoh Program Menggunakan Structs](ch05-02-example-structs.md)
    - [Metode Sintaks](ch05-03-method-syntax.md)

- [Enums dan Pattern Matching](ch06-00-enums.md)
    - [Mendefinisikan sebuah Enum](ch06-01-defining-an-enum.md)
    - [`match` Operator Aliran Kontrol](ch06-02-match.md)
    - [Aliran Kontrol Ringkas dengan `if let`](ch06-03-if-let.md)

## Dasar Rust Literacy

- [Modul](ch07-00-modules.md)
    - [`mod` dan Sistem File](ch07-01-mod-and-the-filesystem.md)
    - [Mengontrol Visibilitas dengan `pub`](ch07-02-controlling-visibility-with-pub.md)
    - [Pengacuan ke Nama dalam Modul yang Berbeda](ch07-03-importing-names-with-use.md)

- [Koleksi Umum](ch08-00-common-collections.md)
    - [Vector](ch08-01-vectors.md)
    - [String](ch08-02-strings.md)
    - [Hash Map](ch08-03-hash-maps.md)

- [Penanganan Kesalahan](ch09-00-error-handling.md)
    - [Kesalahan yang tidak terpulihkan dengan `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
    - [Kesalahan yang Dapat Dipulihkan dengan `Result`](ch09-02-recoverable-errors-with-result.md)
    - [`panic!` atau tidak `panic!`](ch09-03-to-panic-or-not-to-panic.md)

- [Tipe, Sifat, dan Lifetimes Generik](ch10-00-generics.md)
    - [Tipe Data Generik](ch10-01-syntax.md)
    - [Sifat: Mendefinisikan Prilaku Bersama](ch10-02-traits.md)
    - [Memvalidasi Referensi dengan Lifetimes](ch10-03-lifetime-syntax.md)

- [Pengujian](ch11-00-testing.md)
    - [Menulis tes](ch11-01-writing-tests.md)
    - [Menjalankan tes](ch11-02-running-tests.md)
    - [Organisasi Pengujian](ch11-03-test-organization.md)

- [Sebuah Projek I/O : Membangun Program Baris Perintah](ch12-00-an-io-project.md)
    - [Menerima Argumen Baris Perintah](ch12-01-accepting-command-line-arguments.md)
    - [Membaca sebuah File](ch12-02-reading-a-file.md)
    - [Refactoring untuk Meningkatkan Modularitas dan Penanganan Kesalahan](ch12-03-improving-error-handling-and-modularity.md)
    - [Mengembangkan Fungsi Perpustakaan dengan Test Driven Development](ch12-04-testing-the-librarys-functionality.md)
    - [Bekerja dengan Environment Variable](ch12-05-working-with-environment-variables.md)
    - [Menulis Pesan Kesalahan ke Kesalahan Standar Sebagai Ganti Keluaran Standar](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Berpikir di Rust

- [Fitur Bahasa Fungsional: Iterator dan Penutupan](ch13-00-functional-features.md)
    - [Penutupan: Fungsi Anonim yang Dapat Menangkap Lingkungannya](ch13-01-closures.md)
    - [Memproses Seri Item dengan Iterator](ch13-02-iterators.md)
    - [Meningkatkan Proyek I/O kami](ch13-03-improving-our-io-project.md)
    - [Membandingkan Kinerja: Loops vs. Iterator](ch13-04-performance.md)

- [Lebih banyak tentang Cargo and Crates.io](ch14-00-more-about-cargo.md)
    - [Menyesuaikan Build dengan Release Profiles](ch14-01-release-profiles.md)
    - [Menerbitkan Crate ke Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Ruang kerja Cargo](ch14-03-cargo-workspaces.md)
    - [Memasang Binari dari Crates.io dengan `cargo install`](ch14-04-installing-binaries.md)
    - [Memperluas Cargo dengan Perintah Custom](ch14-05-extending-cargo.md)

- [Pointer Pintar](ch15-00-smart-pointers.md)
    - [`Box<T>` Menunjukkan ke Data pada Heap dan Memiliki Ukuran yang Dikenal](ch15-01-box.md)
    - [Sifat `Deref` Membolehkan Akses ke Data Melalui Referensi](ch15-02-deref.md)
    - [Sifat `Drop` Menjalankan Kode di Pembersihan](ch15-03-drop.md)
    - [`Rc<T>`, Referensi Menghitung Pointer Pintar](ch15-04-rc.md)
    - [`RefCell<T>` dan Pola Mutasi Interior](ch15-05-interior-mutability.md)
    - [Membuat Siklus Referensi dan Memori Aman Bocor](ch15-06-reference-cycles.md)

- [Fearless Concurrency](ch16-00-concurrency.md)
    - [Thread](ch16-01-threads.md)
    - [Pesan yang dilewatkan](ch16-02-message-passing.md)
    - [Negara Bersama](ch16-03-shared-state.md)
    - [Concurrency yang dapat diperluas: `Sync` dan `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [Apakah Rust sebuah Bahasa Pemrograman Berorientasi Objek?](ch17-00-oop.md)
    - [Apa yang dimaksud dengan Pemrograman Berorientasi Objek?](ch17-01-what-is-oo.md)
    - [Sifat Objek untuk Menggunakan Nilai dari Berbagai Jenis](ch17-02-trait-objects.md)
    - [Implementasi Pola Desain Berorientasi Objek](ch17-03-oo-design-patterns.md)

## Topic Lanjutan

- [Pola yang Cocok dengan Struktur Nilai](ch18-00-patterns.md)
    - [Semua Tempat Pola Bisa Digunakan](ch18-01-all-the-places-for-patterns.md)
    - [Refutability: Apakah Pola Mungkin Gagal untuk Sama?](ch18-02-refutability.md)
    - [Semua Sintaks Pola](ch18-03-pattern-syntax.md)

- [Fitur Lanjutan](ch19-00-advanced-features.md)
    - [Rust yang tidak Aman](ch19-01-unsafe-rust.md)
    - [Lifetime Lanjutan](ch19-02-advanced-lifetimes.md)
    - [Sifat Lanjutan](ch19-03-advanced-traits.md)
    - [Tipe Lanjutan](ch19-04-advanced-types.md)
    - [Fungsi & Penutupan Lanjutan](ch19-05-advanced-functions-and-closures.md)

- [Proyek Akhir: Membangun Server Web Multithreaded](ch20-00-final-project-a-web-server.md)
    - [Sebuah Server Web Single Threaded ](ch20-01-single-threaded.md)
    - [Bagaimana Permintaan Lambat Mempengaruhi Throughput](ch20-02-slow-requests.md)
    - [Merancang Tampilan Thread Pool](ch20-03-designing-the-interface.md)
    - [Membuat Thread Pool dan menyimpan Thread](ch20-04-storing-threads.md)
    - [Mengirim Permintaan ke Threads Melalui Saluran](ch20-05-sending-requests-via-channels.md)
    - [Mematikan dan Pembersihan yang Bagus](ch20-06-graceful-shutdown-and-cleanup.md)

- [Appendix](appendix-00.md)
    - [A - Kata Kunci](appendix-01-keywords.md)
    - [B - Operator and Simbol](appendix-02-operators.md)
    - [C - sifat yang diturunkan](appendix-03-derivable-traits.md)
    - [D - Macros](appendix-04-macros.md)
    - [E - Terjemahan](appendix-05-translation.md)
    - [F - Fitur Terbaru](appendix-06-newest-features.md)
