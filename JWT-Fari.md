Mengapa JWT Menggunakan Pendekatan Stateless?
Berbeda dengan sistem autentikasi tradisional yang menggunakan Session-based Authentication (di mana server harus menyimpan daftar sesi aktif di database atau RAM/Redis), JWT bersifat Stateless.

Artinya, server tidak perlu menyimpan informasi sesi login pengguna sama sekali. Semua data identitas, role, hingga masa berlaku akun sudah tersimpan di dalam token itu sendiri. Server hanya perlu memverifikasi apakah signature token tersebut sah menggunakan secret key yang dimilikinya.

Bedah Komponen Utama JWT
Header

Berisi metadata teknis tentang bagaimana token diproses.

Formatnya berupa JSON yang berisi dua kunci utama: alg (algoritma hashing seperti HS256 atau RS256) dan typ (tipe token, yaitu JWT).

Payload (Data Utama)

Berisi klaim (claims) atau pernyataan mengenai entitas (pengguna) dan data tambahan.

Terdapat 3 jenis klaim di dalam Payload:

Registered Claims: Klaim standar yang direkomendasikan seperti iss (issuer/penerbit), exp (expiration time/waktu kedaluwarsa), sub (subject/ID pengguna), dan iat (issued at/waktu diterbitkan).

Public Claims: Klaim kustom yang didefinisikan secara umum (misal: email, nama).

Private Claims: Klaim khusus yang dibuat untuk berbagi informasi antara pihak-pihak yang menyetujuinya (misal: role: "admin", tenant_id: 12).

Penting: Payload hanya di-encode dengan Base64Url, bukan dienkripsi. Siapa pun yang mendapatkan token dapat membedah isinya. Oleh karena itu, jangan pernah menyimpan informasi sensitif seperti password, nomor kartu kredit, atau PIN di dalam Payload JWT.

Signature (Tanda Tangan Digital)

Merupakan bagian paling krusial untuk menjaga integritas data.

Dibentuk dengan mengambil Header yang di-encode, Payload yang di-encode, lalu digabungkan dan di-hash menggunakan secret key yang hanya diketahui oleh server.

Fungsi: Jika peretas mencoba mengubah data di Payload (misalnya mengubah role: "user" menjadi role: "admin"), signature otomatis menjadi tidak cocok saat diverifikasi oleh server, sehingga akses ditolak.

Alur Lengkap Operasional JWT
Plaintext
[ Client / App ]                        [ Server / API ]
       |                                       |
       |----- 1. POST /login (User, Pass) ---->|
       |                                       | (Verifikasi ke Database)
       |<---- 2. Kembalikan JWT Token ---------| (Buat Header + Payload + Signature)
       |                                       |
  (Simpan Token)                               |
       |                                       |
       |-- 3. GET /api/data ------------------>|
       |   Header: Authorization: Bearer <token>| (Verifikasi Signature pake Secret Key)
       |                                       | (Jika Valid, proses request)
       |<---- 4. Kirim Data Response ----------|
Kelebihan & Kekurangan JWT
Kelebihan:

Skalabilitas Tinggi: Karena stateless, server tidak terbebani pengecekan session ke database untuk setiap request. Sangat cocok untuk arsitektur Microservices.

Lintas Domain (CORS Friendly): Mudah digunakan jika frontend dan backend berada di domain/server yang berbeda.

Performa Cepat: Memangkas I/O ke database untuk urusan verifikasi identitas.

Kekurangan & Tantangan:

Ukuran Token Lebih Besar: Dibandingkan session ID biasa, JWT membawa data Payload sehingga ukuran header HTTP menjadi sedikit lebih besar.

Sulit Di-revoke (Dibatalkan) Secara Instan: Karena server tidak menyimpan daftar sesi, token yang sudah diterbitkan akan tetap aktif sampai masa kedaluwarsanya (exp) habis, kecuali jika diimplementasikan mekanisme blacklist di server.