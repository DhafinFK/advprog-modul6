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
