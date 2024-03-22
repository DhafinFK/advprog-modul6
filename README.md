# Tutorial 6 Notes

## Commit 1 Reflection
Kode yang telah ditulis dan di commit adalah sebagian kecil dari web server app. Penjelasan dari bagian-bagian kode yang telah ditulis adalah sebagai berikut:

<hr>

```rust
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};
```
Merupakan bagian kode yang melakukan import untuk I/O, buffered reader, TCPListener untuk mendengarkan TCP connections dan TCP stream yang mewakili koneksi antara local dan remote socket

<hr>

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream)
    }
}
```
Merupakan main function yang menghubungkan TCPT listener dengan localhost di port 7878 dan unwrap digunakan untuk handle case error atau panic. Kemudian masuk infinite loop yang mendengarkan TCP connections terus. Untuk tiap connectino yang valid, program memanggil handle_connection.

<hr>

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader 
        .lines() 
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty()) 
        .collect();

    println!("Request: {:#?}", http_request);
}
```
Fungsi ini mengambil referensi dari TCPStream dan kemudian membungkus stream dengan BufReader untuk membuat pembacaan baris lebih efisien. Kemudian stream tadi akan diubah menjadi vektor berisi string yang berisi http request dan kemudian di print dengan rapih.

## Commit 2 Reflection
![Commit 2 screen capture](assets/images/commit2.png)

Pada commit ke-2 terdapat import baru yaitu
```rust
use std::fs;
```
yang merupakan standard library dari rust file system module untuk read write file.

<hr>

Kemudian terdapat beberapa tambahan pada handle connection function:
```rust
// code

let status_line = "HTTP/1.1 200 OK"; 
let contents = fs::read_to_string("hello.html").unwrap(); 
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

stream.write_all(response.as_bytes()).unwrap();
```
- status)line mendefinisikan status dari http response 
- contents mengisi content dari http response dengan hello.html dalam bentuk string. 
- length merepresentasikan ukuran dari body response
- response menggabungkan bagian bagian tadi menjadi satu http response lengkap sebagai string.
- write_all mencoba menulis message lengkap ke dalam strream memastikan HTTP response telah dikirim dan dalam bentuk bytes karena ditulis response.as_bytes()


## Commit 3 Reflection
Kali ini kode diubah sedemikian rupa sehingga bila kita mengakses url yang belum di define oleh main function, maka program akan mengeluarkan html page seperti berikut:

![Commit 3 screen capture](assets/images/commit3.png)

dan pada commit kali ini function handle_connection diubah lagi sebagai berikut:

```rust
let request_line = buf_reader.lines().next().unwrap().unwrap();
```

untuk mendapatkan request method dan target url

<hr>

```rust
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
    ("HTTP/1.1 200 OK", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND", "404.html")
};
```
untuk menetukan status dan filename apa yang akan digunakan sebagai response html

<hr>

```rust
let contents = fs::read_to_string(filename).unwrap();
let length = contents.len();

let response =
    format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
```
untuk membuat response lengkap berdasarkan isi dari variable yang telah ditentukan sesuai dengan request yang dibuat

## Commit 4 Reflection
Pada commit kali ini, lagi-lagi kode pada handle_connection berubah menjadi:

```rust
// code

let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    "GET /sleep HTTP/1.1" => {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    }
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};

// code
```
- menggunakan match statement bukan if else untuk mencari string yang sesuai dengan string yang di dalam curly brackets. 
- sekarang ditambahkan juga sleep untuk thread selama 5 detik setelah berhasil memberikan respone 200 untuk simulasi slow response.

## Commit 5 Reflection
Pada commit kali ini, setiap kali ada stream baru maka program akan membuat threadpool dengan instance 4 worker, yang berarti bisa meng handle 4 TCP connections sekaligus. Kode nya sebagai berikut:

```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
let pool = ThreadPool::new(4); // Creates a new thread pool with 4 workers.

for stream in listener.incoming() {
    let stream = stream.unwrap();

    pool.execute(|| {
        handle_connection(stream);
    });
}
```

sedangkan implementasi threadpool di lib.rs adalah:
- Struktur ThreadPool: Mewakili pool dari thread pekerja. Ia memiliki vektor dari instance Worker dan bagian pengirim dari sebuah saluran untuk mengirimkan pekerjaan ke pekerja.
- Struktur Worker: Mewakili satu thread pekerja. Setiap pekerja memiliki ID dan handle thread.
- Tipe Job: Alias tipe untuk sebuah closure yang dibungkus yang dapat dijalankan oleh pekerja. Ini memungkinkan server untuk mengantri fungsi arbitrer (dalam hal ini, penanganan koneksi) untuk dieksekusi oleh thread pool.