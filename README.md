# Reflection

1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

    - Unary RPC: <br>
    Pada unary steraming, client mengirimkan permintaan ke server dan menunggu tanggapan dari server. Metode ini cocok digunakan ketika client hanya ingin mengirimkan data kecil dan satu kali ke server sehingga tidak diperlukannya pengiriman data banyak sekaligus dan hanya memerlukan respons tunggal dari server (tidak ada kebutuhan untuk streaming data atau interaksi berkelanjutan). Unary cocok digunakan untuk kasus seperti sistem autentikasi, pengiriman form, dan payment service.

    - Server Streaming RPC: <br>
    Pada server streaming, client mengirimkan permintaan ke server dan menerima respons berupa stream data yang terus berlangsung dari server. Server mengirimkan informasi banyak sekaligus melalui sebuah grpc stream, sehingga memungkinkan server untuk mengirimkan sejumlah besar data yang mungkin diperlukan client secara bertahap. Server akan mengirimkan data berulang kali melalui 1 koneksi yang sama sehingga data yang terkirimkan adalah potongan data-data yang membentuk data yang lebih besar dan client tidak perlu menunggu hingga semua data tersedia sekaligus. Contohnya adalah pada transaction service.

    - Bi-directional Streaming RPC: <br>
    Pada bi-directional streaming, client dan server saling mengirimkan stream data secara bersamaan. Baik client maupun server dapat mengirim dan menerima data tanpa harus menunggu respons satu sama lain. Cocok digunakan ketika ada kebutuhan untuk komunikasi dua arah antara client dan server, dimana keduanya perlu mengirim dan menerima data secara independen berulang kali. Contohnya adalah pada chat service.

2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

    Dalam autorisasi/autentikasi, untuk tiap potongan data yang dikirimkan oleh grpc diperlukan validasi autorisasi/autentikasi untuk menjamin keamanannya, misalnya melalui token JWT atau integrasi dengan OAuth2. Autorisasi yang ketat diperlukan untuk mengontrol akses pengguna terhadap resources dan bisa menggunakan middleware untuk implementasinya. Untuk enkripsi data, tiap potongan data yang dikirimkan baik oleh server maupun client perlu di enkripsi secara terpisah. Enkripsi ini dapat dilakukan melalui TLS untuk mengamankan komunikasi antara client dan server dan menjamin privasi dari data yang dikirim.

3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

    Ketika kita ingin membuat sistem dengan bidirectional streaming dan juga grpc, terutama dalam skenario aplikasi chat, tantangannya adalah memastikan tidak terjadinya race condition dan perlu dipastikan sinkronisasi data.
    Sinkronisasi pesan untuk memastikan urutan dan konsistensi dalam chat real-time juga menjadi hal yang perlu diperhatikan. Selain itu, mengelola koneksi secara efisien dan menangani aliran pesan dengan concurrency juga bisa menjadi tantangan. Koneksi yang hidup lama akan mempersulit load balancing karena bisa saja terbentuk banyak koneksi yang tidak bisa diputuskan akibat grpc sehingga akan membebankan server tersebut.

4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

    Keunggulan:
    - Terintegrasi dengan ekosistem Tokio, seperti modul-modulnya.
    - Data dikelola dengan asynchronous, yang mempercepat pengolahan data.

    Kekurangan:
    - Lebih kompleks karena penggunaan asynchronous.
    - Berpotensi overhead saat traffic tinggi.
    - Ketergantungan dengan Tokio, karena ReceiverStream berasal dari Tokio.
    - Untuk autentikasi dan autorisasi perlu dilakukan untuk tiap data.

5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

    Dalam pengembangan gRPC di Rust, modularitas dan penggunaan ulang kode dapat digunakan dengan memisahkan fungsi dan layanan gRPC ke dalam modul yang berbeda. Dengan demikian, perubahan dan penambahan terhadap fitur dapat dilakukan dengan lebih mudah. Hal tersebut mungkin dilakukan dengan adanya proto (pada kasus ini services.proto). Implementasi server (grpc_server.rs) dan client (grpc_client.rs) dipisahkan ke modul yang berbeda sehingga meningkatan reusability dan kemudahan maintenance. Selain itu, menggunakan trait untuk mendefinisikan fungsionalitas yang dapat dibagi antar komponen dapat meningkatkan fleksibilitas dan modularitas.

6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

    Terdapat beberapa langkah yang dapat dilakukan untuk menghandle payment processing logic yg lebih kompleks. Pertama, bisa dibuat menjadi server streaing daripada unary untuk mendukung proses transaksi dalam jumlah besar secara bersamaan, agar pengiriman data-data yang kompleks dapat dilakukan dengan lebih cepat dan mengurangi overhead pembuatan koneksi antara client dan server.
    Selanjutnya, diperlukan validasi data pada PaymentRequest untuk memastikan keamanan dan kelengkapan data sebelum diproses.

7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

    Penggunaan gRPC dapat membuat suatu distributed system berkomunikasi dengan lebih efektif, efisien, meningkatkan kinerja, dan memudahkan integrasi antar komponen sistem. Dengan menggunakan gRPC, terjadi penyederhanaan interaksi antarmodul melalui definisi yang jelas dalam file proto, dibandingkan dengan pendekatan REST yang memerlukan konfigurasi manual untuk metode HTTP. Melalui file proto, client seolah-olah memanggil function dari server sehingga mempermudah konektivitas dan operasi antar teknologi, platform, dan sistem terdistribusinya. Dengan berbasis HTTP/2, gRPC memungkinkan komunikasi yang efisien dan cepat antar layanan dengan overhead rendah dan kemampuan streaming yang baik.

8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

    Keunggulan:
    - Multiplexing dalam HTTP/2 memungkinkan pengelolaan multiple requests dan responses melalui satu koneksi saja, sehingga dapat mengurangi latency dan meningkatkan performa aplikasi.
    - Fitur prioritas request dan server push mempercepat waktu load halaman dengan mengizinkan pengiriman resource sebelum diminta oleh client.

    Kekurangan:
    - Memerlukan biaya overhead yang lebih tinggi baik untuk performance maupun memory karena HTTP/2 bisa mempertahankan 1 koneksi untuk banyak request dan response
    - Implementasi HTTP/2 lebih kompleks dan membutuhkan enkripsi SSL/TLS
    - Penggunaan HTTP/1.1 dengan WebSocketnya memungkinkan komunikasi dua arah dan real-time yang efektif dan bisa lebih sederhana dibandingkan HTTP/2 dalam situasi tertentu.

9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

    Model request-response yang digunakan oleh REST API cenderung membatasi interaksi di mana client mengirim sebuah request dan menunggu sebuah response dari server. Hal ini menyebabkan REST API akan memerlukan waktu lebih lama dibanding dengan gRPC untuk real-time communication karena pada REST API tiap pengiriman request memerlukan pembuatan koneksi baru dari client ke server. Sementara itu, gRPC menggunakan HTTP/2 yang mendukung bidirectional streaming sehingga hanya memerlukan 1 koneksi sampai semua request selesai. Hal ini membuat gRPC lebih optimal untuk real-time communication dan lebih responsif dibandingkan REST API karena kecepatannya yang lebih baik dan latencynya lebih rendah. 

10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

    Dengan pendekatan protocol buffer, gRPC dan protocol buffernya akan membuat suatu interface yang akan menyebabkan client dan server code dan akan saat runtime men-serialize dan men-deserialize message yang masuk dan keluar secara otomatis. Protocol buffer di gRPC secara otomatis memvalidasi dan mengoptimisasi data. Dengan demikian, tiap data yang masuk dan keluar bisa dibilang terjamin bisa digunakan dan tidak salah tipe, dimana jika dibandingkan dengan JSON di REST APIs, sangat mungkin untuk mengirimkan data yang berisikan informasi yang tidak bisa dipakai oleh salah satu pihak karena tidak ada contraint mengenai apa yang perlu dikirim. Selain itu, protocol buffer membantu mengurangi penggunaan bandwidth dan meningkatkan kecepatan pemrosesan data yang sangat berguna untuk aplikasi dengan kebutuhan efisiensi tinggi dan format data yang tetap atau konsisten. 

