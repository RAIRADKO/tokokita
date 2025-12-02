# 📱 TokoKita - Aplikasi Manajemen Produk Flutter

**TokoKita** adalah aplikasi mobile berbasis Flutter yang dirancang untuk mengelola data produk (CRUD) dengan sistem autentikasi pengguna. Aplikasi ini menggunakan arsitektur **BLoC Pattern** untuk manajemen state dan pemisahan logika bisnis dari UI.

## 🛠️ Fitur Utama
* **Autentikasi:** Registrasi dan Login pengguna.
* **Manajemen Produk:**
    * Melihat daftar produk (Read).
    * Menambah produk baru (Create).
    * Mengubah data produk (Update).
    * Menghapus produk (Delete).
* **Validasi Form:** Memastikan input data sesuai format.
* **State Management:** Menggunakan BLoC manual.

---

## 🚀 Alur Kerja Aplikasi & Dokumentasi Code

Berikut adalah penjelasan detail mengenai alur kerja aplikasi mulai dari Login hingga proses CRUD beserta potongan kode pentingnya.

### 1. Proses Registrasi & Login

Sebelum masuk ke dashboard, pengguna harus memiliki akun dan melakukan login.

#### A. Halaman Login

Pengguna memasukkan Email dan Password. Sistem akan memvalidasi apakah input tidak boleh kosong.

<img width="602" height="867" alt="Screenshot 2025-12-02 223217" src="https://github.com/user-attachments/assets/fccf728b-4e5c-4252-aa61-d865148032f1" />

**Penjelasan Kode:**
Ketika tombol Login ditekan, fungsi `_submit()` pada `login_page.dart` akan dipanggil. Fungsi ini memanggil `LoginBloc.login` untuk verifikasi ke API/Mock Data.

```dart
// File: lib/ui/login_page.dart

void _submit() {
  _formKey.currentState!.save();
  setState(() {
    _isLoading = true;
  });
  // Memanggil LoginBloc untuk request ke server/mock
  LoginBloc.login(
    email: _emailTextboxController.text,
    password: _passwordTextboxController.text,
  ).then((value) async {
    if (value.code == 200) {
      // Jika berhasil, simpan token dan ID user
      await UserInfo().setToken(value.token.toString());
      await UserInfo().setUserID(int.parse(value.userId.toString()));
      // Pindah ke halaman ProdukPage
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => const ProdukPage()),
      );
    } else {
      // Tampilkan dialog gagal
      showDialog(...);
    }
  }, onError: (error) {
    // Handle error koneksi dll
    showDialog(...);
  });
  // ...
}
```

#### B. Notifikasi Login (Berhasil/Gagal)

Jika login gagal (misal: password salah), aplikasi menampilkan WarningDialog.

<img width="600" height="861" alt="Screenshot 2025-12-02 221355" src="https://github.com/user-attachments/assets/6ced3ce5-8f4c-4b30-a534-7c2f90391795" />

**Penjelasan Kode:**
Dialog ini muncul pada blok else atau onError dalam fungsi submit di atas.

```dart
// File: lib/ui/login_page.dart (Bagian Warning Dialog)

showDialog(
  context: context,
  barrierDismissible: false,
  builder: (BuildContext context) => const WarningDialog(
    description: "Login gagal, silahkan coba lagi",
  ),
);
```

### 2. Dashboard & List Produk (READ)

Setelah login berhasil, pengguna diarahkan ke halaman utama yang menampilkan daftar produk.

#### A. Tampilan List Produk

Halaman ini menggunakan FutureBuilder untuk mengambil data secara asinkron dari ProdukBloc.

<img width="611" height="874" alt="Screenshot 2025-12-02 221552" src="https://github.com/user-attachments/assets/6e08d00b-413a-424d-bc0e-fb54202994c6" />

**Penjelasan Kode:**
FutureBuilder memanggil ProdukBloc.getProduks() saat widget dibangun. Jika data tersedia (snapshot.hasData), widget ListProduk akan merender daftar item.

```dart
// File: lib/ui/produk_page.dart

body: FutureBuilder<List>(
  future: ProdukBloc.getProduks(), // Request data produk
  builder: (context, snapshot) {
    if (snapshot.hasError) print(snapshot.error);
    
    // Tampilkan ListProduk jika ada data, atau Loading jika belum
    return snapshot.hasData
        ? ListProduk(list: snapshot.data)
        : const Center(child: CircularProgressIndicator());
  },
),
```

#### 3. Tambah Data Produk (CREATE)

Pengguna dapat menekan ikon (+) di pojok kanan atas ProdukPage untuk menambah produk baru.

#### A. Form Tambah Produk

Form ini meminta input Kode Produk, Nama Produk, dan Harga.

<img width="604" height="869" alt="Screenshot 2025-12-02 221713" src="https://github.com/user-attachments/assets/b537da8f-9aa5-4574-9255-77d6cea52d96" />

**Penjelasan Kode:**
Saat tombol "SIMPAN" ditekan, fungsi simpan() dijalankan. Fungsi ini membungkus inputan ke dalam objek Produk dan mengirimnya via ProdukBloc.

```dart
// File: lib/ui/produk_form.dart

simpan() async {
  // ... set loading state ...
  Produk createProduk = Produk(id: null);
  createProduk.kodeProduk = _kodeProdukTextboxController.text;
  createProduk.namaProduk = _namaProdukTextboxController.text;
  createProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);

  try {
    // Memanggil API tambah produk
    await ProdukBloc.addProduk(produk: createProduk);
    
    // Kembali ke list produk jika berhasil
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (BuildContext context) => const ProdukPage()),
    );
  } catch (error) {
    // Tampilkan error jika gagal
    showDialog(...);
  }
}
```

#### 4. Detail & Ubah Produk (UPDATE)

Ketika salah satu item produk di list diklik, pengguna akan masuk ke halaman Detail.

#### A. Halaman Detail Produk

Menampilkan rincian produk dan tombol opsi "EDIT" dan "DELETE".

<img width="599" height="872" alt="Screenshot 2025-12-02 221954" src="https://github.com/user-attachments/assets/c244192c-c737-42a0-be19-ae28a799b1bb" />

#### B. Form Ubah Produk

Jika tombol "EDIT" ditekan, pengguna diarahkan ke ProdukForm, namun kolom input sudah terisi data produk yang dipilih (isUpdate() function).

<img width="607" height="873" alt="Screenshot 2025-12-02 221729" src="https://github.com/user-attachments/assets/4da3d6bc-6ea7-4bfb-a524-aa4b2159be4d" />

**Penjelasan Kode:**
Logika update ditangani oleh fungsi ubah() di produk_form.dart. Perbedaannya dengan simpan adalah fungsi ini menyertakan id produk dan memanggil endpoint update.

```dart
// File: lib/ui/produk_form.dart

ubah() async {
  // ... set loading state ...
  Produk updateProduk = Produk(id: widget.produk!.id!); // Menggunakan ID yang ada
  updateProduk.kodeProduk = _kodeProdukTextboxController.text;
  updateProduk.namaProduk = _namaProdukTextboxController.text;
  updateProduk.hargaProduk = int.parse(_hargaProdukTextboxController.text);

  try {
    // Memanggil API update produk
    await ProdukBloc.updateProduk(produk: updateProduk);
    
    // Kembali ke halaman list
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (BuildContext context) => const ProdukPage()),
    );
  } catch (error) {
    showDialog(...);
  }
}
```

#### 5. Hapus Produk (DELETE)

Pada halaman Detail Produk, terdapat tombol "DELETE".

#### A. Konfirmasi Hapus

Sistem akan memunculkan dialog konfirmasi untuk mencegah ketidaksengajaan.

<img width="603" height="872" alt="Screenshot 2025-12-02 222019" src="https://github.com/user-attachments/assets/6b6e5b95-be1a-42af-82ac-c14659342d5d" />

**Penjelasan Kode:** 
Fungsi confirmHapus() menampilkan AlertDialog. Jika user menekan "Ya", ProdukBloc.deleteProduk dipanggil dengan parameter ID produk.

```dart
// File: lib/ui/produk_detail.dart

void confirmHapus() {
  AlertDialog alertDialog = AlertDialog(
    content: const Text("Yakin ingin menghapus data ini?"),
    actions: [
      OutlinedButton(
        child: const Text("Ya"),
        onPressed: () async {
          try {
            // Eksekusi hapus data by ID
            await ProdukBloc.deleteProduk(id: int.parse(widget.produk!.id!));
            
            // Redirect ke halaman list
            Navigator.of(context).pushReplacement(
              MaterialPageRoute(builder: (context) => const ProdukPage()),
            );
          } catch (error) {
            // Handle error
            Navigator.pop(context);
            showDialog(...);
          }
        },
      ),
      OutlinedButton(
        child: const Text("Batal"),
        onPressed: () => Navigator.pop(context), // Tutup dialog
      )
    ],
  );
  showDialog(builder: (context) => alertDialog, context: context);
}
```
