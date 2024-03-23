# Advanced Programming - Tutorial 


------------
## Overview

This repository contains the code and reflections for Tutorial 6 in Advanced Programming course by Vinka Alrezky As (NPM: 2206820200)❄️
- [Tutorial 6](#tutorial-6)
------------
# Tutorial 6

### _Reflection_ 

- [Commit 1](#commit-1-reflection-notes)
- [Commit 2](#commit-2-reflection-notes)
- [Commit 3](#commit-3-reflection-notes)
- [Commit 4](#commit-4-reflection-notes)
- [Commit 5](#commit-5-reflection-notes)

#### Commit 1 Reflection notes

- Fungsi `handle_connection` bertujuan untuk membaca setiap baris dari _HTTP request_ melalui koneksi TCP dan mengumpulkannya menjadi vektor string.

    ```rust
        let buf_reader = BufReader::new(&stream); 
    ```
    Baris ini membuat objek `buf_reader` yang merupakan pembaca berbuffer dari _stream_ yang membantu membaca data dari _stream_ secara efisien.

    ```rust
        let http_request: Vec<_> = buf_reader
            .lines()
            .map(|result| result.unwrap())
            .take_while(|lines| !lines.is_empty())
            .collect();
    ```
    Baris ini memulai pembacaan setiap baris dari `buf_reader` dan menghasilkan iterator atas setiap baris dalam data.

    Bagian `.map(|result| result.unwrap())` mengubah setiap `Result` menjadi `String` dengan `result.unwrap()`. Jika ada kesalahan saat membaca baris, program akan berhenti karena `unwrap()`.

    Bagian `.take_while(|lines| !lines.is_empty())` mengambil baris dari iterator selama baris tersebut tidak kosong. Ini berdasarkan asumsi bahwa baris kosong menandakan akhir dari _header HTTP Request_.

    Bagian `.collect();` mengumpulkan baris-baris tersebut menjadi vektor. Vektor ini disimpan dalam variabel `http_request`.

    ```rust
        println!("Request: {:#?}", http_request);
    ```
    Baris diatas mencetak `http_request` ke konsol. `{:#?}` adalah _placeholder_ untuk mencetak _output_ dalam format yang mudah dibaca.

#### Commit 2 Reflection notes

- Tambahan _line code_ pada `handle_connection` berfungsi untuk membuat dan mengirim _HTTP response_ kembali ke klien.
    ```rust
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();
        let response = 
            format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    ```
    Bagian di atas mendefinisikan status line dari _HTTP response_, membaca isi dari file `hello.html` menjadi string, menghitung panjang dari isi tersebut, dan memformat _HTTP response_. Respons ini mencakup status line, header "Content-Length", dan isi dari file `hello.html`.

    ```rust
        stream.write_all(response.as_bytes()).unwrap();
    ```
    Baris ini menulis _HTTP response_ kembali ke TCP stream. Jika operasi penulisan gagal karena alasan apa pun, program akan berhenti dan keluar karena `unwrap()`.

- Berikut adalah hasil tangkapan layar saat menjalankan program dengan tambahan kode baru:
    <img src="https://i.imgur.com/WHen92w.png" alt="Commit 2 screen capture 1" width="400"/>
    <img src="https://i.imgur.com/JFjvLDI.png" alt="Commit 2 screen capture 2" width="400"/>

 #### Commit 3 Reflection notes
- Dalam _commit_ ini, sebuah fitur baru telah ditambahkan untuk meningkatkan kemampuan server dalam menangani _HTTP responses_. Ketika sebuah rute yang tidak dikenali diminta (misalnya `http://127.0.0.1:7878/bad`), server akan merespons dengan kode kesalahan 404 dan mengirimkan isi dari halaman `404.html`.    
  ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="utf-8">
            <title>Hello!</title>
        </head>
        <body>
            <h1>Oops!</h1>
            <p>Sorry, I don't know what you're asking for.</p>
        </body>
        </html>
    ```

    ```rust
        fn handle_connection(mut stream: TcpStream) {
          let buf_reader = BufReader::new(&mut stream);
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
  Fungsi membaca baris permintaan dari klien menggunakan `BufReader`, kemudian memeriksa apakah baris permintaan tersebut adalah `GET / HTTP/1.1`. Jika iya, server akan merespons dengan status 200 OK dan mengirimkan isi dari file `hello.html`. Jika tidak, server akan merespons dengan status 404 Not Found dan mengirimkan isi dari file `404.html`.

  Penanganan respons dilakukan dengan membentuk pesan HTTP yang sesuai, termasuk status line, header Content-Length, dan isi konten dari file yang dibaca. Selanjutnya, pesan HTTP tersebut dikirimkan ke klien melalui TCP _stream_.


- Selanjutnya, _refactoring_ dilakukan dengan mengubah bagaimana status baris dan nama file ditentukan. Sebelumnya, mungkin ada beberapa baris kode yang melakukan hal yang sama secara terpisah untuk setiap kasus (`GET / HTTP/1.1` dan kasus lainnya). Dengan _refactoring_, penentuan status baris dan nama file dipindahkan ke dalam satu blok `if-else`.

    ```rust
        fn handle_connection(mut stream: TcpStream) {
            let buf_reader = BufReader::new(&mut stream);
            let request_line = buf_reader.lines().next().unwrap().unwrap();

            let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
                ("HTTP/1.1 200 OK", "hello.html")
            } else {
                ("HTTP/1.1 404 NOT FOUND", "404.html")
            };
            let contents = fs::read_to_string(filename).unwrap();
            let length = contents.len();

            let response = format!(
                "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
            );
            stream.write_all(response.as_bytes()).unwrap();
        }
    ```
- Berikut adalah hasil tangkapan layar saat menjalankan program:

    <img src="https://i.imgur.com/sFslsfd.png" alt="Commit 3 screen capture 1" width="400"/>
    <img src="https://i.imgur.com/vhkwpUx.png" alt="Commit 3 screen capture 2" width="400"/>

#### Commit 4 Reflection notes

- Dalam _commit_ ini, simulasi dilakukan untuk memperlihatkan dampak respons lambat pada server yang berjalan pada satu _thread_ saja.

  Fungsi handle_connection dimodifikasi dengan menambahkan _endpoint_ baru yaitu `/sleep`. Ketika _endpoint_ ini diakses, server akan _delay_ selama 10 detik sebelum memberikan respons. Ini bertujuan untuk mensimulasikan permintaan yang lambat dan membutuhkan banyak _resource_.
  ```rust
      fn handle_connection(mut stream: TcpStream) {
        // --snip--
        let (status_line, filename) = match &request_line[..] {
          "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
          "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10));
            ("HTTP/1.1 200 OK", "hello.html")
          }
          _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
        };
        // --snip--
      }
  ```
  Ketika membuka dua web browser dan mengakses _endpoint_ `http://127.0.0.1:7878/sleep` pada salah satunya dan _endpoint_ `http://127.0.0.1:7878/` pada yang lainnya, akan terlihat bahwa server memerlukan waktu untuk memberikan respons. Hal ini dikarenakan server beroperasi pada satu _thread_ saja, sehingga hanya dapat memproses satu permintaan pada satu waktu. Ketika sedang menangani permintaan `/sleep`, server tidak dapat menangani permintaan lainnya.

#### Commit 5 Reflection notes

- Dalam _commit_ ini, server telah ditingkatkan dengan membuatnya menjadi _multithreaded_. Ini memungkinkan server untuk menangani beberapa permintaan secara bersamaan, yang menghasilkan peningkatan kinerja yang signifikan saat ada banyak _task_.
  Langkah yang diambil adalah mengikuti instruksi dalam Rust book untuk menerapkan ThreadPool. ThreadPool adalah kumpulan _thread_ yang telah diinisiasi yang siap menangani _tasks_ masuk. Ketika _task_ baru masuk, _task_ tersebut diberikan ke salah satu _thread_ dalam pool. Jika semua _thread_ sedang sibuk, _task_ tersebut menunggu hingga salah satu _thread_ tersedia.
  Hal ini memungkinkan server untuk menangani beberapa permintaan secara bersamaan. Ketika ada permintaan yang lambat, tidak lagi menghambat permintaan lain. Sebaliknya, permintaan lambat ditangani oleh satu _thread_, sementara _thread_ lain tetap menangani permintaan lainnya.


- Struktur ThreadPool menyimpan vektor dari instance `Worker` dan sebuah pengirim untuk _multi-producer_, _single-consumer (mpsc) channel_. Setiap Worker mewakili sebuah _thread_ dalam pool dan dibuat dengan ID unik serta penerima untuk _mpsc channel_. 
  Fungsi `ThreadPool::new` menginisialisasi ThreadPool dengan jumlah worker yang ditentukan, sementara fungsi `ThreadPool::execute` memungkinkan _tasks_ dikirimkan ke worker untuk dieksekusi.
  Fungsi `Worker::new` menetapkan setiap Worker untuk secara terus-menerus menunggu dan menerima _task_ dari _mpsc channel_, kemudian menjalankannya.


- Setelah menerapkan ThreadPool, terjadi peningkatan kinerja yang signifikan pada server. Ketika _endpoint_ `http://127.0.0.1:7878/sleep` diakses pada satu web browser dan _endpoint_ `http://127.0.0.1:7878/` diakses pada web browser yang berbeda, _endpoint_ `http://127.0.0.1:7878/` tidak lagi menunggu hingga_ endpoint_ `http://127.0.0.1:7878/sleep` selesai. Respons diberikan secara langsung, karena ditangani oleh _thread_ yang berbeda.
  Perilaku ini menunjukkan efek _multithreading_. Dengan menangani setiap permintaan dalam _thread_ yang berbeda, kinerja dan responsifitas server dapat meningkat secara signifikan.