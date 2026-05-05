1. Unary RPC itu satu request, satu response (paling simpel, cocok buat operasi CRUD atau validasi), server streaming itu satu request tapi server kirim banyak response bertahap (cocok buat data besar atau update berkala seperti log/monitoring), sedangkan bi-directional streaming dua arah secara real-time (client dan server bisa kirim kapan aja, cocok buat chat, game, atau sinkronisasi data interaktif).

2. Yang perlu kita perhatikan ada tiga hal utama: autentikasi, otorisasi, dan enkripsi (wajib pakai TLS agar data aman in-transit, karena gRPC defaultnya HTTP/2 tapi tidak otomatis aman tanpa TLS). Selain itu, kita juga tidak boleh lupa validasi input, rate limiting, dan proteksi dari abuse karena gRPC tetap bisa diserang kalau endpoint kita kebuka tanpa kontrol.

3. Tantangan utama di bi-directional streaming gRPC di Rust terletak di concurrency dan state management: kita harus handle banyak stream async sekaligus tanpa race condition atau deadlock, sambil menjaga konsistensi state (misalnya daftar user online di chat). Selain itu, ada masalah backpressure dan flow control (kalau client lambat atau spam, server bisa kena overload), serta error handling yang tricky karena koneksi bisa putus kapan aja dan harus recover dengan clean tanpa bocorin resource. Terakhir, debugging juga lebih susah karena interaksi dua arah real-time membuat bug menjadi nondeterministic dan sulit direproduksi.

4. Keunggulannya simpel dan idiomatik, yaitu mudah mengubah channel (mpsc) jadi stream gRPC sehingga produksi data bisa dipisah dari pengiriman dan tetap async-friendly. Kekurangannya ada overhead dan potensi bottleneck/backpressure dari channel, serta kontrol error dan alur jadi kurang eksplisit.

5. Membuat struktur kode dengan pemisahan jelas antara layer (proto/definition, service handler, business logic, dan data access) serta pakai trait/interface berguna agar implementasi bisa ditukar tanpa ubah core logic. Bisa menambahkan module per fitur + shared utilities (error handling, auth middleware), sehingga komponen bisa reusable, mudah di-test, dan scalable seiring kompleksitas naik.

6. Implementasi basic biasanya belum cukup. Kita perlu tambah validasi transaksi (amount, currency, idempotency key), integrasi ke payment gateway eksternal (dengan retry & timeout), serta logging/auditing agar semua transaksi bisa dilacak. Selain itu, handle edge case seperti partial failure, concurrency (double charge), dan pastikan ada mekanisme keamanan (signature verification, fraud check) supaya sistem tetap konsisten dan aman.

7. Adopsi gRPC membuat arsitektur jadi lebih contract-driven (pakai Protobuf) dan efisien lewat HTTP/2, tetapi trade-off nya adalah interoperabilitas bisa lebih ribet karena tidak semua platform/tool langsung “native” support seperti REST/JSON, jadi sering butuh adapter atau gateway. Di sisi lain, kalau ekosistemnya konsisten, gRPC justru membuat komunikasi antar service lebih terstruktur, cepat, dan scalable dibanding pendekatan tradisional.

8. HTTP/2 unggul di performa—multiplexing, header compression, dan persistent connection yang membuat latency lebih rendah dan efisien dibanding HTTP/1.1 atau WebSocket, apalagi untuk komunikasi antar service yang intens. Akan tetapi, kelemahannya ada di kompleksitas dan kompatibilitas: debugging lebih susah, tooling tidak seluas REST/JSON, dan kadang perlu fallback/gateway kalau harus interop dengan sistem lama atau client yang belum support penuh HTTP/2.

9. Model request-response di REST itu membuat tiap interaksi harus menunggu siklus kirim dan balik selesai, sehingga kurang optimal untuk real time karena latency bertambah di tiap request. Sementara itu, gRPC dengan bidirectional streaming memungkinkan komunikasi dua arah dengan terus menerus tanpa membuka koneksi baru, sehingga jauh lebih responsif dan efisien untuk use case real time seperti chat atau live update.

10. Pendekatan schema-based di gRPC (Protobuf) membuat kontrak data jelas, type-safe, dan efisien (binary, lebih kecil/cepat), tapi menjadi lebih kaku karena setiap perubahan harus update schema dan regenerate code. Sebaliknya,JSON di REST lebih fleksibel dan mudah diubah tanpa strict contract, tapi rawan inconsistency, kurang efisien, dan error baru kelihatan saat runtime.
    




