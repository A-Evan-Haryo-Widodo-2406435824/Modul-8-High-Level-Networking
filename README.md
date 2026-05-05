# REFLECTION
Evan Haryo Widodo, 2406435824, Kelas A

## 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
- Unary: transmisi data gRPC yang menggunakan siklus satu request dan satu response. Metode ini cocok untuk operasi, seperti autentikasi user atau simple calculation.
- Server Streaming: transmisi data gRPC yang mana Client mengirim satu request dan Server akan merespons dengan aliran data yang berkelanjutan / data besar akan dipartisi menjadi beberapa chunk sebelum dikirim. Metode ini cocok untuk implementasi pada sistem pengunduhan file besar atau sinkronisasi riwayat Whatsapp chat pada Whatsapp Web.
- Bi-directional Streaming: transmisi data gRPC yang menggunakan siklus komunikasi dua arah secara real-time yang mana client dan server dapat saling mengirim pesan secara independen pada waktu kapan pun. Metode ini cocok diterapkan pada sistem, seperti real-time messaging, multiplayer game, real-time analytics, dan lainnya.

## 2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?
- Authentication
    - Dalam gRPC di Rust, proses autentikasi dapat dilakukan dengan validasi token menggunakan JWT atau API key yang dikirim melalui metadata request. Mekanisme ini memastikan bahwa hanya client verified yang dapat mengakses service.
- Authorization
    - Penerapan Role-Based Access Control (RBAC) merupakan langkah best practice untuk memastikan bahwa hanya user berwenang yang dapat memanggil RPC tertentu. Selain itu, authorization ini didahului oleh Authentication terlebih dahulu
- Data Encryption
    - Proses pengiriman data menggunakan metode gRPC ini tetap harus dienkripsi untuk mencegah sniffing atau pun Man-In-The-Middle Attack. Maka dari itu, penerapan TLS/SSL secara wajib dikonfigurasi pada lapisan transport (HTTP/2) untuk mencegah format biner protobuf dimanipulasi.

## 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?
- Concurrency
    - Bidirectional streaming memungkinkan client dan server mengirim dan menerima pesan secara bersamaan. Hal ini membutuhkan pengelolaan concurrency yang baik (misal dalam hal manajemen threads) karena jika tidak, hal ini dapat menyebabkan race condition atau blocking.
- Stream Lifecycle Management
    - Pengelolaan kapan stream dimulai, aktif, dan ditutup dapat dikatakan sangat kompleks, terutama saat terjadi salah satu pihak mengalami disconnect tiba-tiba. Hal tersebut dapat menyebabkan memory leak akibat data yang tidak berhenti mengalir.
- Error Handling
    - Error tidak selalu menghentikan seluruh komunikasi pada bidirectional streaming sehingga diperlukan mekanisme seperti retry atau graceful shutdown.

## 4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?
- Kelebihan
    - Integrasi mudah dengan async Rust karena dapat mengubah Receiver langsung menjadi Stream sehingga bisa langsung digunakan di async
    - Compatible dengan komuniasi berbasis channel karena memudahkan pengiriman data antatr thread tanpa harus mengelola stream manual
- Kekurangan
    - Bergabtung pada channel karena membutuhkan alokasi memori tambahan untuk buffer antrean, misalnya mpsc::channel(4). 
    - Jika ukuran buffer terlalu kecil, producer bisa terblokir menunggu stream dibaca 
    - Jika ukuran buffer terlalu besar, alhasil akan memakan memori berlebih saat traffic load tinggi.

## 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?
- Memisahkan definisi protobuf dan kode hasil generate ke modul independen, seperti src/proto
- Memisahkan handler API, service layer, dan repository layer
- Menggunakan trait Rust untuk implementasi Dependency Injection sehingga setiap layer dapat dilakukan unit-testing tanpa harus menyalakan server gRPC

## 6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?
- Logika pembayaran harus dibungkus dalam transaksi database yang memenuhi prinsip ACID untuk memastikan saldo terpotong dan status pesanan diperbarui secara atomik.
- Implementasi idempotency key pada setiap request pembayaran untuk mencegah user mendapat tagihan / harus membayar lebih dari satu kali (double-charge) jika terjadi gangguan jaringan.
- Layanan perlu berinteraksi dengan API pihak ketiga secara asinkron, seperti Xendit dan lainnya serta menangani webhook untuk konfirmasi status pembayaran.
- Sistem harus melakukan audit logging terhadap proses transaksi untuk keperluan investigasi keamanan jika terjadi anomaly ataupun security attack lainnya.

## 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?
gRPC sangat cocok diimplementasikan pada komunikasi arsitektur microservices yang menuntut latensi rendah dan throughput tinggi karena pengiriman datanya berbasis biner. Namun, gRPC memiliki support visibility pada browser yang rendah karena aplikasi web/frontend standar belum bisa membaca framing HTTP/2 gRPC secara native atau perlu menyertakan proxy tambahan.

## 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?
- Kelebihan:
    - Protokol dapat menanggulangi banyak request/response secara bersamaan dalam satu koneksi TCP tunggal
    - Protokol dapat berlangsung secara terus-menerus (sistem dapat terhubung secara lifetime)
- KEKURANGAN:
    - Proses debugging lebih sulit dibandingkan dengan REST API biasa karena data pada gRPC ini berbasis biner.

## 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?
- RestAPI berlangsung di atas model interaksi stateless unary sehingag untuk mengadaptasi proses real-time, client harus melakukan request ke server secara berulang. Hal ini berkontradiksi dengan gRPC yang dapat mempertahankan satu koneksi TCP secara terus-menerus sehingga hal ini memangkas latency yang tinggi dan tidak perlu melakukan request ulang terus-menerus.

## 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?
- Protobuf: schema-based atau tipe data terstruktur (strictly typed). Meskipun begitu, hal tersebut mengurangi terjadinya runtime error, menghasilkan ukuran payload biner yang sangat kecil dan ringan, dan client library dapat di-generate secara otomatis.
- Rest API: tipe data sangat fleksibel karena rest menerapkan schema-less. Hal ini menyatakan bahwa format json mudah dibaca oleh manusia sehingga ideal untuk diterapkan sebagai format API publik. Namun, fleksibilitas ini menuntut validitas tambahan di level aplikasi untuk mencegah anomali tipe data dan ukuran payload yang jauh lebih besar akibat format berbasis teks.