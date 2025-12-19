# Biro Perjalanan API Documentation

## Authentication Endpoints

### 1. Register Tourist - `POST /auth/register`

**Guards:** Tidak ada (Public)

**Authority:** Siapapun (Public/Anonymous)

**Business Process:**
- Pendaftaran pengguna baru sebagai Tourist
- Menerima username, email, dan password
- Password di-hash untuk keamanan
- Data disimpan ke database dengan role Tourist
- Setelah registrasi, dapat langsung login dengan kredensial yang dibuat

**Request Body:**
```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

---

### 2. Register Employee (Admin Only) - `POST /auth/admin/register-employee`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** Hanya user dengan role `ADMIN`

**Business Process:**
- Pendaftaran karyawan baru
- Menerima data akun (email, username, password) dan profil (nama lengkap, kode karyawan, nomor telepon)
- Data disimpan ke database dengan role Employee
- Karyawan dapat login dan mengakses fitur sesuai role mereka

**Request Body:**
```json
{
  "email": "string",
  "username": "string",
  "password": "string",
  "profile": {
    "fullName": "string",
    "employeeCode": "string",
    "phoneNumber": "string"
  }
}
```

---

### 3. Login - `POST /auth/login`

**Guards:** `LocalAuthGuard`

**Authority:** Siapapun yang memiliki email dan password yang valid

**Business Process:**
- Proses login untuk semua pengguna
- Memvalidasi email dan password ke database
- Jika valid, generate Access Token dan Refresh Token
- Refresh Token di-hash dan disimpan ke database
- Access Token untuk akses endpoint yang dilindungi
- Refresh Token untuk mendapatkan Access Token baru tanpa login ulang

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response:**
```json
{
  "accessToken": "string",
  "refreshToken": "string"
}
```

---

### 4. Refresh Token - `POST /auth/refresh`

**Guards:** `JwtRtAuthGuard` (JWT Refresh Token Guard)

**Authority:** User yang memiliki Refresh Token yang valid

**Business Process:**
- Mendapatkan Access Token baru tanpa login ulang
- User mengirim Refresh Token via header Authorization
- Sistem validasi dan cocokkan dengan hash di database
- Jika valid, return Access Token baru dan Refresh Token baru
- Berguna saat Access Token expired namun user masih dalam sesi aktif

**Request Header:**
```
Authorization: Bearer <refresh_token>
```

**Response:**
```json
{
  "accessToken": "string",
  "refreshToken": "string"
}
```

---

## User Management Endpoints

### Tourist Profile Management

#### 1. Get All Tourists - `GET /user/tourist`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Mengambil daftar semua turis beserta profil mereka
- Menampilkan nama lengkap, nomor identitas, alamat, dan nomor telepon

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

---

#### 2. Create Tourist Profile - `POST /user/tourist`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `TOURIST`

**Business Process:**
- Tourist membuat profil diri sendiri
- Menerima nama lengkap, nomor identitas, alamat, dan nomor telepon
- ID user diambil dari token
- Validasi: user harus role Tourist dan belum punya profil
- Data profil disimpan ke database

**Request Header:**
```
Authorization: Bearer <access_token_tourist>
```

**Request Body:**
```json
{
  "fullName": "string",
  "identityNumber": "string",
  "address": "string",
  "phoneNumber": "string"
}
```

---

#### 3. Update Tourist Profile - `PATCH /user/tourist`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `TOURIST`

**Business Process:**
- Tourist mengupdate profil diri sendiri
- Semua field bersifat optional (kirim yang ingin diubah saja)
- ID user diambil dari token
- Validasi: user harus role Tourist dan sudah punya profil
- Data profil diupdate di database

**Request Header:**
```
Authorization: Bearer <access_token_tourist>
```

**Request Body:**
```json
{
  "fullName": "string",
  "identityNumber": "string",
  "address": "string",
  "phoneNumber": "string"
}
```

---

### Get My Profile

#### 4. Get My Profile - `GET /user/profile`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- User melihat profil dirinya sendiri
- ID user diambil dari token
- Response disesuaikan dengan role:
  - Tourist: nama lengkap, nomor identitas, alamat, nomor telepon
  - Employee: nama lengkap, kode karyawan, nomor telepon
  - Admin: username, email, role

**Request Header:**
```
Authorization: Bearer <access_token>
```

---

### Admin - Employee Management

#### 5. Update Employee Profile (Admin Only) - `PATCH /user/employee/:id/profile`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Update profil karyawan tertentu
- Menerima ID karyawan dari URL dan data update dari body
- Validasi: target harus karyawan yang sudah punya profil
- Jika kode karyawan diubah, cek duplikasi dengan karyawan lain

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

**Request Body:**
```json
{
  "fullName": "string",
  "employeeCode": "string",
  "phoneNumber": "string"
}
```

---

#### 6. Get All Employees - `GET /user/employee`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Mengambil daftar semua karyawan beserta profil mereka
- Menampilkan nama lengkap, kode karyawan, dan nomor telepon

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

#### 7. Get Employee By ID - `GET /user/employee/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Melihat detail karyawan tertentu berdasarkan ID
- Menerima ID dari URL
- Validasi: user dengan ID tersebut harus karyawan
- Menampilkan detail profil karyawan

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

### Admin - Full User Management

#### 8. Get All Users - `GET /user`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Melihat seluruh pengguna di sistem
- Menampilkan semua role (Admin, Employee, Tourist)
- Data lengkap user beserta profil tanpa filter

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

#### 9. Get User By ID - `GET /user/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Melihat detail user tertentu berdasarkan ID (role apapun)
- Menerima ID dari URL
- Menampilkan data user beserta profil sesuai role mereka

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

#### 10. Toggle User Status - `PATCH /user/:id/toggle-status`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Mengaktifkan atau menonaktifkan akun user
- Menerima ID user dari URL
- Membalik status aktif (aktif → nonaktif, atau sebaliknya)
- User yang dinonaktifkan tidak dapat login
- Berguna untuk menonaktifkan akun tanpa menghapus dari database

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

#### 11. Delete User - `DELETE /user/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN`

**Business Process:**
- Menghapus user secara permanen dari database
- Menerima ID user dari URL
- Menghapus user beserta profil terkait (CASCADE delete)
- Return pesan konfirmasi penghapusan
- **Perhatian:** Operasi permanen dan tidak dapat dibatalkan

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

---

## Travel Spot Endpoints

### 1. Create Travel Spot - `POST /travel-spot`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Membuat tempat wisata baru
- Menerima nama, deskripsi, kota, latitude, dan longitude
- Validasi: tidak boleh ada duplikasi nama di kota yang sama
- Data disimpan ke database
- Return data tempat wisata yang baru dibuat

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "city": "string",
  "latitude": number,
  "longitude": number
}
```

**Contoh:**
```json
{
  "name": "Pantai Kuta",
  "description": "Pantai Kuta adalah pantai yang terletak di Kuta, Badung, Bali...",
  "city": "Badung",
  "latitude": -8.7184,
  "longitude": 115.1686
}
```

---

### 2. Get All Travel Spots - `GET /travel-spot`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua daftar tempat wisata
- Data diurutkan berdasarkan tanggal pembuatan descending (terbaru dulu)
- Return array semua tempat wisata

**Request Header:**
```
Authorization: Bearer <access_token>
```

---

### 3. Search Travel Spots - `GET /travel-spot/search?q=<searchTerm>`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mencari tempat wisata berdasarkan kata kunci
- Menerima kata kunci dari query parameter `q`
- Validasi: kata kunci tidak boleh kosong
- Pencarian di field nama dan kota (case-insensitive)
- Return array tempat wisata yang cocok dengan kata kunci

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `q` (required): Kata kunci pencarian

**Contoh Request:**
```
GET /travel-spot/search?q=pantai
GET /travel-spot/search?q=bali
```

---

### 4. Get Travel Spots by City - `GET /travel-spot/city/:city`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua tempat wisata di kota tertentu
- Menerima nama kota dari URL
- Validasi: nama kota tidak boleh kosong
- Filter berdasarkan kota dan urutkan descending
- Return array tempat wisata di kota tersebut

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Contoh Request:**
```
GET /travel-spot/city/Badung
GET /travel-spot/city/Yogyakarta
```

---

### 5. Update Travel Spot - `PATCH /travel-spot/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Update data tempat wisata berdasarkan ID
- Menerima ID dari URL dan data update dari body (semua field optional)
- Validasi: tempat wisata dengan ID tersebut harus ada
- Jika nama/kota diubah, cek duplikasi kombinasi nama + kota
- Return data tempat wisata yang sudah diupdate

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "city": "string",
  "latitude": number,
  "longitude": number
}
```

**Contoh:**
```json
{
  "description": "Candi Buddha terbesar di dunia yang terletak di Magelang, Jawa Tengah...",
  "latitude": -7.607874,
  "longitude": 110.203751
}
```

---

### 6. Delete Travel Spot - `DELETE /travel-spot/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
Endpoint ini digunakan oleh Admin atau Employee untuk menghapus tempat wisata secara permanen dari database. Sistem menerima ID tempat wisata dari URL. Validasi dilakukan untuk memastikan tempat wisata dengan ID tersebut ada. Setelah berhasil dihapus, sistem mengembalikan pesan konfirmasi penghapusan.

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Contoh Request:**
```
DELETE /travel-spot/123e4567-e89b-12d3-a456-426614174000
```

---

## Travel Package Endpoints

### Travel Package Management

#### 1. Create Travel Package with Itineraries - `POST /travel-package`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Membuat paket wisata baru beserta jadwal itinerary sekaligus dalam satu transaksi
- Menerima data paket: nama, deskripsi, negara, provinsi, kota, harga dasar, durasi hari, dan daftar itinerary
- Validasi:
  - Tidak ada paket dengan nama yang sama
  - Semua tempat wisata di itinerary ada di database
  - Urutan hari tidak melebihi durasi paket
  - Tidak ada duplikasi kombinasi tempat wisata + urutan hari
- Menggunakan transaksi database (all or nothing)

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "country": "string",
  "province": "string",
  "city": "string",
  "basePrice": number,
  "durationDays": number,
  "itineraries": [
    {
      "travelSpotId": "uuid",
      "daySequence": number,
      "startTime": "HH:mm:ss",
      "endTime": "HH:mm:ss",
      "activityDetail": "string"
    }
  ]
}
```

**Contoh:**
```json
{
  "name": "Paket Wisata Bali Lengkap 3 Hari 2 Malam",
  "description": "Paket tour lengkap mengunjungi 5 destinasi wisata terbaik di Bali...",
  "country": "Indonesia",
  "province": "Bali",
  "city": "Denpasar",
  "basePrice": 2850000,
  "durationDays": 3,
  "itineraries": [
    {
      "travelSpotId": "123e4567-e89b-12d3-a456-426614174001",
      "daySequence": 1,
      "startTime": "08:00:00",
      "endTime": "12:00:00",
      "activityDetail": "Hari pertama dimulai dengan mengunjungi Pantai Kuta..."
    }
  ]
}
```

---

#### 2. Get All Travel Packages - `GET /travel-package?includeItineraries=<boolean>`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua paket wisata yang tersedia
- Parameter opsional `includeItineraries` untuk menyertakan itinerary atau tidak
- Default: hanya data paket tanpa itinerary
- Jika `includeItineraries=true`: tampilkan paket + jadwal itinerary + detail tempat wisata
- Paket diurutkan berdasarkan tanggal pembuatan descending (terbaru dulu)
- Itinerary diurutkan berdasarkan urutan hari ascending

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `includeItineraries` (optional, default: `false`): Boolean untuk menyertakan detail itinerary

**Contoh Request:**
```
GET /travel-package?includeItineraries=true
GET /travel-package?includeItineraries=false
GET /travel-package
```

---

#### 3. Get Travel Package by ID - `GET /travel-package/:id?includeItineraries=<boolean>`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil detail paket wisata tertentu berdasarkan ID
- Parameter opsional `includeItineraries` untuk menyertakan itinerary atau tidak
- Default: paket ditampilkan beserta semua jadwal itinerary
- Jika `includeItineraries=false`: hanya data paket
- Validasi: paket dengan ID tersebut harus ada
- Itinerary diurutkan berdasarkan urutan hari ascending

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Query Parameters:**
- `includeItineraries` (optional, default: `true`): Boolean untuk menyertakan detail itinerary

**Contoh Request:**
```
GET /travel-package/123e4567-e89b-12d3-a456-426614174000?includeItineraries=true
GET /travel-package/123e4567-e89b-12d3-a456-426614174000?includeItineraries=false
```

---

#### 4. Update Travel Package - `PATCH /travel-package/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Update data paket wisata (tidak termasuk itinerary)
- Menerima ID paket dari URL dan data update dari body (semua field optional)
- Validasi:
  - Paket dengan ID tersebut harus ada
  - Jika nama diubah, cek duplikasi nama
  - Jika durasi diubah, cek apakah ada itinerary melebihi durasi baru
- Update field yang diberikan dan simpan perubahan

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "country": "string",
  "province": "string",
  "city": "string",
  "basePrice": number,
  "durationDays": number
}
```

**Contoh:**
```json
{
  "basePrice": 2950000,
  "description": "Paket tour lengkap dengan FREE airport transfer!"
}
```

---

#### 5. Delete Travel Package - `DELETE /travel-package/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` saja (bukan Employee)

**Business Process:**
- Menghapus paket wisata secara permanen dari database
- Menerima ID paket dari URL
- Validasi: paket dengan ID tersebut harus ada
- CASCADE delete: semua itinerary terkait akan terhapus otomatis
- Return pesan konfirmasi penghapusan

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

**Contoh Request:**
```
DELETE /travel-package/123e4567-e89b-12d3-a456-426614174000
```

---

### Itinerary Management (Sub-resource dari Travel Package)

#### 6. Add Itinerary to Travel Package - `POST /travel-package/:packageId/itinerary`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Menambahkan jadwal itinerary baru ke paket wisata yang sudah ada
- Menerima ID paket dari URL dan data itinerary dari body
- Data itinerary: ID tempat wisata, urutan hari, waktu mulai, waktu selesai, detail aktivitas
- Validasi:
  - Paket wisata ada
  - Tempat wisata ada
  - Urutan hari tidak melebihi durasi paket
  - Tidak ada duplikasi kombinasi tempat wisata + urutan hari dalam paket
- Return itinerary baru dengan detail tempat wisata

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "travelSpotId": "uuid",
  "daySequence": number,
  "startTime": "HH:mm:ss",
  "endTime": "HH:mm:ss",
  "activityDetail": "string"
}
```

**Contoh:**
```json
{
  "travelSpotId": "b174ecfe-1256-482d-ab40-d1d9ce37d552",
  "daySequence": 2,
  "startTime": "14:00:00",
  "endTime": "17:30:00",
  "activityDetail": "Sore hari relax di Nusa Dua Beach. Aktivitas: snorkeling, banana boat..."
}
```

---

#### 7. Get All Itineraries of Package - `GET /travel-package/:packageId/itinerary`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua jadwal itinerary dari paket wisata tertentu
- Menerima ID paket dari URL
- Validasi: paket dengan ID tersebut harus ada
- Data diurutkan berdasarkan urutan hari ascending, lalu waktu mulai ascending
- Return array itinerary lengkap dengan detail tempat wisata

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Contoh Request:**
```
GET /travel-package/123e4567-e89b-12d3-a456-426614174000/itinerary
```

---

#### 8. Get Specific Itinerary - `GET /travel-package/:packageId/itinerary/:itineraryId`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil detail itinerary tertentu dari paket wisata tertentu
- Menerima ID paket dan ID itinerary dari URL
- Validasi:
  - Itinerary dengan ID tersebut harus ada
  - Itinerary benar-benar milik paket tersebut
- Return detail itinerary lengkap dengan info tempat wisata

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Contoh Request:**
```
GET /travel-package/123e4567-e89b-12d3-a456-426614174000/itinerary/987e6543-e21b-12d3-a456-426614174999
```

---

#### 9. Update Itinerary - `PATCH /travel-package/:packageId/itinerary/:itineraryId`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Update itinerary tertentu dalam paket wisata
- Menerima ID paket dan ID itinerary dari URL, serta data update dari body (semua field optional)
- Validasi:
  - Itinerary ada dan benar-benar milik paket tersebut
  - Jika tempat wisata diubah, pastikan tempat wisata baru ada
  - Jika urutan hari diubah, tidak melebihi durasi paket dan tidak duplikasi
- Update field yang diberikan dan simpan

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "travelSpotId": "uuid",
  "daySequence": number,
  "startTime": "HH:mm:ss",
  "endTime": "HH:mm:ss",
  "activityDetail": "string"
}
```

**Contoh:**
```json
{
  "startTime": "15:00:00",
  "endTime": "18:30:00",
  "activityDetail": "Sore hari relax di Nusa Dua. BONUS: Free mineral water dan buah segar!"
}
```

---

#### 10. Delete Itinerary - `DELETE /travel-package/:packageId/itinerary/:itineraryId`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Menghapus itinerary tertentu dari paket wisata
- Menerima ID paket dan ID itinerary dari URL
- Validasi:
  - Itinerary ada
  - Itinerary benar-benar milik paket tersebut
- Hapus dari database dan return pesan konfirmasi

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Contoh Request:**
```
DELETE /travel-package/123e4567-e89b-12d3-a456-426614174000/itinerary/987e6543-e21b-12d3-a456-426614174999
```

---

## Travel Trip Endpoints

### 1. Create Travel Trip - `POST /travel-trip`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Membuat trip baru atau penjadwalan perjalanan untuk tourist
- Menerima data: ID turis, ID karyawan, ID paket wisata, tanggal mulai, tanggal selesai, catatan (optional)
- Validasi dalam transaksi database:
  - Turis yang dipilih ada dan berstatus turis
  - Karyawan yang dipilih ada dan berstatus karyawan
  - Paket wisata ada
  - Tanggal mulai lebih awal dari tanggal selesai
  - Durasi trip sesuai dengan durasi paket
  - Tidak ada bentrokan jadwal dengan trip lain dari turis yang sama
- Return trip baru dengan detail lengkap (HTTP 201 CREATED)

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "userTouristId": "uuid",
  "userEmployeeId": "uuid",
  "travelPackageId": "uuid",
  "startDate": "ISO8601 datetime",
  "endDate": "ISO8601 datetime",
  "notes": "string (optional)"
}
```

**Contoh:**
```json
{
  "userTouristId": "ed74149c-8962-4f3a-bec8-9ccfebd701e9",
  "userEmployeeId": "aefabbe6-4177-4e12-93cc-de26785b1f8f",
  "travelPackageId": "056ed003-d02c-4dbc-ad42-8519b982d8a0",
  "startDate": "2026-06-15T08:00:00.000Z",
  "endDate": "2026-06-17T17:00:00.000Z",
  "notes": "This is Gacorrr"
}
```

---

### 2. Get All Travel Trips - `GET /travel-trip`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Mengambil semua trip dari semua turis dan karyawan
- Data diurutkan berdasarkan tanggal pembuatan descending (terbaru dulu)
- Return array trip dengan informasi turis, karyawan, dan ringkasan paket

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

---

### 3. Get Upcoming Trips - `GET /travel-trip/upcoming`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua trip yang akan datang (belum dimulai)
- Filter: tanggal mulai > waktu sekarang
- Diurutkan berdasarkan tanggal mulai ascending (paling dekat dulu)
- Berguna untuk melihat jadwal trip mendatang

**Request Header:**
```
Authorization: Bearer <access_token>
```

---

### 4. Get Ongoing Trips - `GET /travel-trip/ongoing`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua trip yang sedang berlangsung
- Filter: tanggal mulai ≤ waktu sekarang AND tanggal selesai ≥ waktu sekarang
- Diurutkan berdasarkan tanggal mulai ascending
- Berguna untuk melihat trip yang sedang aktif

**Request Header:**
```
Authorization: Bearer <access_token>
```

---

### 5. Get Completed Trips - `GET /travel-trip/completed`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Mengambil semua trip yang sudah selesai (riwayat perjalanan)
- Filter: tanggal selesai < waktu sekarang
- Diurutkan berdasarkan tanggal selesai descending (paling baru selesai dulu)
- Berguna untuk melihat riwayat perjalanan

**Request Header:**
```
Authorization: Bearer <access_token>
```

---

### 6. Get My Trips (Tourist) - `GET /travel-trip/my-trips`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `TOURIST`

**Business Process:**
- Melihat semua trip milik turis yang login
- Mengambil ID pengguna dari token
- Data diurutkan berdasarkan tanggal mulai descending (terbaru dulu)
- Tourist hanya bisa melihat trip miliknya sendiri

**Request Header:**
```
Authorization: Bearer <access_token_tourist>
```

---

### 7. Get My Assignments (Employee) - `GET /travel-trip/my-assignments`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `EMPLOYEE` dan `ADMIN`

**Business Process:**
- Melihat semua trip yang di-assign ke karyawan yang login
- Mengambil ID pengguna dari token
- Data diurutkan berdasarkan tanggal mulai descending
- Employee hanya melihat trip yang mereka tangani sendiri

**Request Header:**
```
Authorization: Bearer <access_token_employee_atau_admin>
```

---

### 8. Get Trips by Tourist ID - `GET /travel-trip/tourist/:touristId`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Melihat semua trip dari turis tertentu berdasarkan ID
- Menerima ID turis dari URL
- Validasi: pengguna dengan ID tersebut harus ada
- Data diurutkan berdasarkan tanggal mulai descending
- Berguna untuk tracking riwayat perjalanan customer tertentu

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Contoh Request:**
```
GET /travel-trip/tourist/ed74149c-8962-4f3a-bec8-9ccfebd701e9
```

---

### 9. Get Trips by Employee ID (Admin Only) - `GET /travel-trip/employee/:employeeId`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` saja

**Business Process:**
- Melihat semua trip yang ditangani oleh karyawan tertentu
- Menerima ID karyawan dari URL
- Validasi: pengguna dengan ID tersebut harus ada
- Data diurutkan berdasarkan tanggal mulai descending
- Berguna untuk monitoring beban kerja atau performa karyawan

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

**Contoh Request:**
```
GET /travel-trip/employee/aefabbe6-4177-4e12-93cc-de26785b1f8f
```

---

### 10. Get Trips by Package ID - `GET /travel-trip/package/:packageId`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Melihat semua trip yang menggunakan paket wisata tertentu
- Menerima ID paket wisata dari URL
- Validasi: paket wisata dengan ID tersebut harus ada
- Data diurutkan berdasarkan tanggal mulai descending
- Berguna untuk analisis popularitas paket atau perencanaan ketersediaan

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Contoh Request:**
```
GET /travel-trip/package/056ed003-d02c-4dbc-ad42-8519b982d8a0
```

---

### 11. Get Trip by ID - `GET /travel-trip/:id`

**Guards:** `JwtAuthGuard` (tanpa RolesGuard)

**Authority:** Semua user yang sudah login (`ADMIN`, `EMPLOYEE`, `TOURIST`)

**Business Process:**
- Melihat detail trip tertentu berdasarkan ID
- Menerima ID trip dari URL
- Validasi: trip dengan ID tersebut harus ada
- Return informasi lengkap: turis, karyawan, paket wisata, tanggal, catatan

**Request Header:**
```
Authorization: Bearer <access_token>
```

**Contoh Request:**
```
GET /travel-trip/123e4567-e89b-12d3-a456-426614174000
```

---

### 12. Update Travel Trip - `PATCH /travel-trip/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` dan `EMPLOYEE`

**Business Process:**
- Update trip: tanggal mulai, tanggal selesai, dan catatan
- **Tidak bisa** mengubah turis, karyawan, atau paket
- Menerima ID trip dari URL dan data update dari body (semua field optional)
- Validasi dalam transaksi database:
  - Trip ada
  - Jika tanggal diubah: tanggal mulai < tanggal selesai
  - Durasi baru sesuai durasi paket
  - Tidak ada bentrokan dengan trip lain dari turis yang sama
- Update field yang diberikan dan simpan

**Request Header:**
```
Authorization: Bearer <access_token_admin_atau_employee>
```

**Request Body:**
```json
{
  "startDate": "ISO8601 datetime",
  "endDate": "ISO8601 datetime",
  "notes": "string"
}
```

**Contoh:**
```json
{
  "startDate": "2025-01-20T08:00:00.000Z",
  "endDate": "2025-01-22T17:00:00.000Z",
  "notes": "Updated: Include special dietary requirements"
}
```

---

### 13. Delete Travel Trip - `DELETE /travel-trip/:id`

**Guards:**
- `JwtAuthGuard` (Cek Token)
- `RolesGuard` (Cek Role)

**Authority:** User dengan role `ADMIN` saja

**Business Process:**
- Menghapus trip dari database
- Menerima ID trip dari URL
- Validasi:
  - Trip ada
  - Proteksi historis: tidak bisa menghapus trip yang sudah selesai
  - Hanya trip yang akan datang atau sedang berlangsung yang bisa dihapus
- Hapus dari database dan return pesan konfirmasi
- **Perhatian:** Melindungi integritas data historis

**Request Header:**
```
Authorization: Bearer <access_token_admin>
```

**Contoh Request:**
```
DELETE /travel-trip/123e4567-e89b-12d3-a456-426614174000
```

---
