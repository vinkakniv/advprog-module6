# Advanced Programming - Tutorial 


------------
## Overview

This repository contains the code and reflections for Tutorial 6 in Advanced Programming course by Vinka Alrezky As (NPM: 2206820200)❄️
- [Tutorial 6](#tutorial-6)
------------
# Tutorial 6

### Reflection 

1. Fungsi `handle_connection`  bertujuan untuk membaca setiap baris dari _HTTP request_ melalui koneksi TCP dan mengumpulkannya menjadi vektor string.

    ```rust
        let buf_reader = BufReader::new(&stream); 
    ```
    Baris ini membuat objek `buf_reader` yang merupakan pembaca berbuffer dari _stream_ yang membantu membaca data dari `stream` secara efisien.

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
