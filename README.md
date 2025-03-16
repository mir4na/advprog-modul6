## Commit 1 Reflection

Berikut code pada fungsi handle_connection.

```
    fn handle_connection(mut stream: TcpStream) {
        let buf_reader = BufReader::new(&mut stream);
        let http_request: Vec<_> = buf_reader .lines()

        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty()) .collect();

        println!("Request: {:#?}", http_request);
    }
```

Fungsi ini menerima parameter berupa TcpStream yang mewakili koneksi TCP aktif dengan client. Keyword mut pada parameter menandakan bahwa stream dapat diubah (mutable) karena dibutuhkan ketika kita perlu membaca data dari stream tersebut.

Sederhananya, inti dari fungsi ini ialah penggunaan BufReader yang dibuat dari stream TCP. BufReader adalah komponen standar dalam Rust yang menyediakan buffering untuk operasi I/O. Kelas ini memungkinkan kita membaca data baris per baris dari stream dengan metode ```.lines()``` yang cocok untuk protokol berbasis teks seperti HTTP.

Proses pengumpulan HTTP request dilakukan melalui rangkaian method chaining. Pertama, ```.lines()``` membaca stream baris per baris sehingga menghasilkan sequence of Result<String>. Kemudian, ```.map()``` mengubah setiap Result menjadi String dengan memanggil ```unwrap()```. Lalu, method ```.take_while()``` digunakan untuk mengambil baris-baris sampai menemui baris kosong yang dalam protokol HTTP menandakan akhir dari header request. Terakhir, ```.collect()``` mengumpulkan semua baris tersebut ke dalam vector.

Lalu, ketika saya lakukan ```cargo run``` dan mengakses url ```http://127.0.0.1:7878``` terdapat output yang ditampilkan.

```
    C:\Hafizh\kuliah_csui\adpro\hello>cargo run
        Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.01s
        Running `target\debug\hello.exe`
    Request: [
        "GET / HTTP/1.1",
        "Host: 127.0.0.1:7878",
        "Connection: keep-alive",
        "Cache-Control: max-age=0",
        "sec-ch-ua: \"Chromium\";v=\"134\", \"Not:A-Brand\";v=\"24\", \"Google Chrome\";v=\"134\"",
        "sec-ch-ua-mobile: ?0",
        "sec-ch-ua-platform: \"Windows\"",
        "Upgrade-Insecure-Requests: 1",
        "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36",
        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "Sec-Fetch-Site: none",
        "Sec-Fetch-Mode: navigate",
        "Sec-Fetch-User: ?1",
        "Sec-Fetch-Dest: document",
        "Accept-Encoding: gzip, deflate, br, zstd",
        "Accept-Language: en-US,en;q=0.9,en-GB;q=0.8",
    ]
```

Pada output tersebut, request dimulai dengan baris ```GET / HTTP/1.1``` yang mengindikasikan bahwa client menggunakan metode GET untuk mengakses path root (/) dengan protokol HTTP versi 1.1. Lalu, terdapat berbagai HTTP headers yang memberikan informasi tambahan seperti asal host (127.0.0.1:7878), preferensi koneksi (keep-alive), pengaturan cache, informasi browser client (Chrome 134 pada Windows), jenis konten yang dapat diterima, serta berbagai header keamanan modern seperti Sec-Fetch-Site dan Sec-Fetch-Mode yang merupakan bagian dari standar keamanan web sekarang. Semua informasi ini dikumpulkan dengan memanfaatkan BufReader yang membaca TcpStream baris per baris sampai menemukan baris kosong (yang menandakan akhir dari HTTP headers), kemudian dikumpulkan dalam sebuah Vector untuk ditampilkan pada console.