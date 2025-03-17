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

## Commit 2 Reflection

![image](https://github.com/user-attachments/assets/ee9b22e2-e1b7-45fa-9b9e-a184808e9a95)

Berikut perubahan code pada milestone 2.

```
use std::{
    fs,
    ...
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines() 
        .map(|result| result.unwrap()) 
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK"; 
    let contents = fs::read_to_string("hello.html").unwrap(); 
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    stream.write_all(response.as_bytes()).unwrap();
}
```

Pada perubahan code yang baru, fungsi ini tetap membaca dan mengurai HTTP request seperti sebelumnya, tetapi bedanya dengan mempersiapkan respons HTTP. Status line ```HTTP/1.1 200 OK``` menunjukkan bahwa server merespons dengan code status 200 yang berarti permintaan berhasil diproses. Kemudian, server membaca file HTML bernama hello.html dari sistem file menggunakan fungsi ```fs::read_to_string```. Fungsi ini membaca seluruh isi file dan me-return sebagai string yang kemudian disimpan dalam variabel contents.

Respons HTTP dibuat menggunakan fungsi ```format!``` yang menggabungkan status line, header Content-Length (yang menunjukkan panjang konten HTML), dua baris baru (yang menandakan akhir dari header dan awal dari body), dan konten HTML itu sendiri. Respons ini kemudian dikonversi menjadi bytes menggunakan ```.as_bytes()``` dan dikirim ke stream koneksi menggunakan ```stream.write_all()```.

Lalu, ketika saya lakukan ```cargo run``` terdapat output yang ditampilkan pada console.

```
C:\Hafizh\kuliah_csui\adpro\hello>cargo run
warning: unused variable: `http_request`
  --> src\main.rs:16:9
   |
16 |     let http_request: Vec<_> = buf_reader
   |         ^^^^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_http_request`
   |
   = note: `#[warn(unused_variables)]` on by default

warning: `hello` (bin "hello") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target\debug\hello.exe`
```

Pada output yang ditampilkan, muncul suatu peringatan. Peringatan ini muncul saat kompilasi ("unused variable: http_request") yang menunjukkan bahwa variabel http_request tidak digunakan dalam code. Ini adalah peringatan umum dari Rust yang mengingatkan bahwa ada variabel yang tidak digunakan sehingga bisa menjadi indikasi potensi masalah atau code yang tidak efisien. Compiler menyarankan untuk menambahkan underscore di depan nama variabel (_http_request) jika memang variabel tersebut sengaja tidak digunakan yang setidaknya akan menekan peringatan yang tampil pada console.

## Commit 3 Reflection

Terdapat perubahan code pada fungsi handle_connection.

![image](https://github.com/user-attachments/assets/7e508c80-7496-4b1a-91a3-d73db35e8497)

```
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

Pada perubahan code tersebut, server kini membedakan antara request yang valid (GET pada path root "/") dan request yang tidak valid. Fungsi ini sekarang hanya membaca baris pertama dari HTTP request menggunakan ```buf_reader.lines().next().unwrap().unwrap()``` yang mengambil hanya satu baris pertama dari request - yaitu baris yang berisi metode HTTP, path, dan versi protokol.

Server kemudian mengevaluasi baris request tersebut. Jika baris request adalah "GET / HTTP/1.1" yang menandakan request GET ke halaman utama, server merespons dengan status 200 OK dan mengirimkan konten dari file hello.html. Namun, jika baris request berbeda dari pola tersebut (misalnya, mencoba mengakses halaman yang tidak ada), server merespons dengan status 404 NOT FOUND dan mengirimkan konten dari file 404.html.

Berikut perubahan code setelah refactoring.

```
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Setelah adanya refactoring, terjadi pengurangan duplikasi dan peningkatan keterbacaan pada code. Sebelumnya, code memiliki dua blok yang hampir identik untuk menangani kasus 200 OK dan 404 NOT FOUND. Refactoring menggunakan tuple assignment untuk menentukan nilai status_line dan filename berdasarkan kondisi request. Kemudian, logika untuk membaca file, menghitung panjang konten, memformat respons, dan menuliskannya ke stream hanya perlu muncul sekali pada code.

Kenapa refactoring diperlukan? Pertama, refactoring mengurangi duplikasi code yang merupakan prinsip dasar dalam pengembangan perangkat lunak yang baik. Code yang terduplikasi sulit untuk dipelihara karena perubahan harus dilakukan di beberapa tempat sehingga meningkatkan risiko kesalahan. Kedua, code yang di-refactor lebih mudah dibaca dan dipahami karena memisahkan dengan jelas bagian penentuan kondisi (if-else) dari bagian pelaksanaan respons. Ketiga, hal ini membuat code lebih modular sehingga jika ingin menambahkan lebih banyak rute atau jenis respons, struktur code yang di-refactor akan lebih mudah disesuaikan.

## Commit 4 Reflection

Terdapat perubahan pada code tersebut.

```
use std::{
    ...
    thread,
    time::Duration,
};

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"), 
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10)); 
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Perubahan utama dalam code ini adalah penggunaan match statement sebagai pengganti if-else sebelumnya. Pattern matching di Rust memungkinkan ekspresi ```match &request_line[..]``` untuk membandingkan string request dengan pola-pola yang berbeda sehingga server kini memiliki tiga kemungkinan respons: halaman utama untuk ```GET / HTTP/1.1```, halaman yang sama tetapi dengan delay untuk ```GET /sleep HTTP/1.1```, dan halaman 404 untuk semua request lainnya.

Ketika dua browser dibuka secara bersamaan, dengan satu browser mengakses ```127.0.0.1:7878/sleep``` dan browser lainnya mengakses ```127.0.0.1:7878```, maka server menunjukkan keterbatasan. Ketika browser pertama mengakses path ```/sleep```, server memulai delay selama 10 detik. Selama waktu ini, jika browser kedua mencoba mengakses path root ("/"), browser kedua harus menunggu hingga request pertama selesai diproses sebelum mendapatkan respons.

Hal ini terjadi karena server diimplementasikan sebagai aplikasi single-threaded. Di dalam loop ```for stream in listener.incoming()```, setiap koneksi masuk diproses satu per satu secara berurutan. Ketika sebuah koneksi sedang diproses (termasuk delay 10 detik), server tidak dapat melayani koneksi lain hingga pemrosesan koneksi saat ini selesai. Ini mengilustrasikan masalah concurrency yang umum dalam pengembangan server yaitu bagaimana menangani banyak koneksi secara bersamaan tanpa membuat pengguna menunggu satu sama lain.

## Commit 5 Reflection

src/lib.rs

```
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        ThreadPool
    }
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
```

Pada src/lib.rs, dideklarasikan struktur ThreadPool sebagai empty struct. Di sini, ThreadPool memiliki dua fungsi utama yaitu new dan execute. Fungsi new menerima parameter size bertipe usize yang menentukan jumlah thread dalam pool. Saat ini, fungsi new hanya memastikan bahwa size lebih dari 0 melalui ```assert!(size > 0)``` dan mengembalikan instance ThreadPool kosong.

Lalu untuk fungsi execute, fungsi ini menerima parameter generik F dengan batasan ```where F: FnOnce() + Send + 'static``` yang artinya F harus berupa closure yang dapat dipanggil sekali (FnOnce), aman ketika dikirim antar thread (Send), dan memiliki lifetime static ('static). Saat ini, metode ini hanya memanggil std::thread::spawn(f) untuk membuat thread baru setiap kali dipanggil.

Perubahan code pada src/main.rs

```
...
use hello::ThreadPool;


fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
...
```

Pada code di main.rs, web server membuat instance ThreadPool dengan parameter 4 yang mengindikasikan untuk membuat pool dengan 4 thread. Untuk setiap koneksi masuk, server memanggil pool.execute() dengan closure yang memanggil handle_connection(stream). Ini menunjukkan bahwa server sekarang menggunakan thread terpisah untuk menangani setiap koneksi sehingga memungkinkan penanganan beberapa koneksi secara bersamaan.