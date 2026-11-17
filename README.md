# UTS-PEMOGRAMAN-MOBILE
jawaban dan kode pemograman mobile ada disini
# UTS Pemrograman Mobile - Paket Soal B

Repo: https://github.com/afrinazka/UTS-PEMOGRAMAN-MOBILE.git

---

## 1. Analisis State dan Event pada Penghitung Biaya (20 poin)

**Cakupan:** Pertemuan 6 - mutable state, rememberSaveable, event, recomposition

```kotlin
@Composable
fun KeranjangScreen() {
    var qty by rememberSaveable { mutableIntStateOf(1) }
    val harga = 25000

    Column {
        Text("Jumlah: $qty")
        Text("Total: Rp ${qty * harga}")

        Button(onClick = { qty++ }) {
            Text("Tambah")
        }
        Button(onClick = { if (qty > 1) qty-- }) {
            Text("Kurangi")
        }
    }
}
```

**Jawaban:**

**a. Nilai `qty` setelah seluruh aksi**

Nilai awal `qty = 1`. Tombol Tambah ditekan dua kali (`1 -> 2 -> 3`), lalu tombol Kurangi ditekan satu kali (`3 -> 2`). Jadi nilai akhir `qty` adalah **2**.

**b. Nilai total yang tampil**

`total = qty * harga = 2 x 25000 = 50000`, sehingga yang tampil adalah **"Total: Rp 50000"**.

Setiap kali `qty` berubah, Compose melakukan *recomposition* pada composable yang membaca `qty`, sehingga teks "Jumlah" dan "Total" otomatis ikut diperbarui.

**c. Tombol Kurangi ditekan saat `qty` bernilai 1**

Tidak terjadi perubahan. Event `onClick` tetap dijalankan, tetapi kondisi `if (qty > 1)` bernilai `false`, sehingga `qty--` tidak dieksekusi. `qty` tetap 1 dan tidak ada recomposition karena nilai state tidak berubah. Dengan guard ini jumlah barang tidak pernah kurang dari 1 (tidak menjadi 0 atau negatif).

**d. Alasan menggunakan `rememberSaveable`**

`remember` hanya menyimpan nilai selama recomposition. Jika terjadi *configuration change* (misalnya layar diputar, perubahan bahasa, atau mode gelap/terang) atau proses aplikasi dihentikan oleh sistem, `Activity` dibuat ulang dan nilai `remember` hilang sehingga `qty` kembali ke 1. `rememberSaveable` menyimpan nilai ke *saved instance state* (Bundle), sehingga `qty` tetap terjaga dan jumlah belanja pengguna tidak hilang.

---

## 2. Perbaikan Daftar Menggunakan LazyColumn (20 poin)

**Cakupan:** Pertemuan 5 - LazyColumn dan collection

**Penyebab error:** isi `LazyColumn` berada di dalam `LazyListScope` (bukan scope `@Composable`), sehingga `Text(...)` tidak boleh dipanggil langsung di dalam `forEach`. Daftar item harus dideklarasikan lewat DSL `items()`.

**Import yang dibutuhkan:**

```kotlin
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
```

**Kode perbaikan:**

```kotlin
@Composable
fun DaftarMahasiswa() {
    val nama = listOf("Ayu", "Beni", "Citra", "Doni")

    LazyColumn {
        items(
            items = nama,
            key = { namaMahasiswa -> namaMahasiswa }
        ) { namaMahasiswa ->
            Text(text = namaMahasiswa)
        }
    }
}
```

**Penjelasan:**
- `items(items = nama)` membuat satu item untuk setiap elemen list dan hanya mengomposisi item yang terlihat di layar (lazy), sehingga daftar dapat di-scroll secara efisien.
- `key` harus unik dan stabil agar Compose dapat mengenali item saat daftar berubah (urutan berubah, item ditambah/dihapus) dan menjaga state-nya. Di sini nama bersifat unik, jadi dipakai sebagai key. Jika ada data nyata, sebaiknya gunakan ID unik.

---

## 3. Menerapkan Kode untuk Elemen UI Form Login (20 poin)

**Cakupan:** Pertemuan 4 dan 6 - OutlinedTextField, Button, state, event

**Composable yang tepat untuk setiap elemen:**

| Elemen | Composable |
|---|---|
| Judul "Masuk KampusKu" | `Text` |
| Kolom email | `OutlinedTextField` |
| Kolom password | `OutlinedTextField` (dengan `PasswordVisualTransformation`) |
| Tombol "Masuk" | `Button` (dengan `Text` di dalamnya) |
| Penataan vertikal | `Column` |

**Kode:**

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.input.PasswordVisualTransformation
import androidx.compose.ui.unit.dp

@Composable
fun LoginScreen(onLogin: (String, String) -> Unit = { _, _ -> }) {
    var email by rememberSaveable { mutableStateOf("") }
    var password by rememberSaveable { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        Text(
            text = "Masuk KampusKu",
            style = MaterialTheme.typography.headlineSmall
        )

        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            singleLine = true,
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Password") },
            singleLine = true,
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            modifier = Modifier.fillMaxWidth()
        )

        Button(
            onClick = { onLogin(email, password) },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Masuk")
        }
    }
}
```

**Penjelasan state dan event:**
- `email` dan `password` adalah state (`mutableStateOf`) yang disimpan dengan `rememberSaveable`.
- `value` menampilkan isi state, sedangkan `onValueChange` adalah event yang memperbarui state setiap pengguna mengetik, sehingga terjadi recomposition dan teks terbaru tampil.
- `onClick` pada `Button` adalah event yang memanggil `onLogin(email, password)`.

---

## 4. Mengubah Rancangan UI Jadwal Menjadi Source Code (25 poin)

**Cakupan:** Pertemuan 4-5 - Column, Row, Card, LazyColumn, Material 3

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

data class Jadwal(
    val waktu: String,
    val mataKuliah: String,
    val ruang: String
)

@Composable
fun JadwalKuliahScreen() {
    val daftarJadwal = listOf(
        Jadwal("08.00 - 09.40", "Pemrograman Mobile", "Ruang Lab 2"),
        Jadwal("10.00 - 11.40", "Basis Data", "Ruang 304")
    )

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // Header
        Text(
            text = "Jadwal Kuliah",
            style = MaterialTheme.typography.headlineSmall
        )

        Spacer(modifier = Modifier.height(12.dp))

        // Pilihan hari
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            listOf("Senin", "Selasa", "Rabu", "Kamis").forEach { hari ->
                TextButton(onClick = { /* pilih hari */ }) {
                    Text(hari)
                }
            }
        }

        Spacer(modifier = Modifier.height(8.dp))

        // Daftar jadwal
        LazyColumn(
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            items(
                items = daftarJadwal,
                key = { jadwal -> jadwal.mataKuliah }
            ) { jadwal ->
                Card(modifier = Modifier.fillMaxWidth()) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Text(
                            text = jadwal.waktu,
                            style = MaterialTheme.typography.labelLarge
                        )
                        Text(
                            text = jadwal.mataKuliah,
                            style = MaterialTheme.typography.titleMedium
                        )
                        Text(text = jadwal.ruang)
                    }
                }
            }
        }
    }
}
```

**Penjelasan:**
- `Column` menyusun header, pilihan hari, dan daftar secara vertikal.
- `Row` menyusun empat tombol hari secara horizontal.
- `LazyColumn` + `items()` menampilkan daftar jadwal; `key` memakai nama mata kuliah yang unik.
- `Card` membungkus setiap jadwal berisi waktu, mata kuliah, dan ruang.

---

## 5. Menerjemahkan Source Code Menjadi Pseudocode (15 poin)

**Cakupan:** Pertemuan 3 - nullable, collection, perulangan, kondisi

**Source code (dilengkapi):**

```kotlin
fun hitungStatusTagihan(data: List<Int?>): Pair<Int, Int> {
    var lunas = 0
    var belum = 0

    for (nominal in data) {
        if ((nominal ?: 0) > 0) {
            lunas++
        } else {
            belum++
        }
    }
    return Pair(lunas, belum)
}
```

**Pseudocode:**

```
MULAI
    lunas <- 0
    belum <- 0

    UNTUK SETIAP nominal DALAM data:
        JIKA nominal bernilai null MAKA
            nominal <- 0
        AKHIR JIKA

        JIKA nominal > 0 MAKA
            lunas <- lunas + 1
        SELAIN ITU
            belum <- belum + 1
        AKHIR JIKA
    AKHIR UNTUK

    KEMBALIKAN (lunas, belum)
SELESAI
```

**Penjelasan cara program menentukan tagihan lunas dan belum:**
1. Dua penghitung, `lunas` dan `belum`, dimulai dari 0.
2. Program memeriksa setiap nominal dalam daftar `data` satu per satu (perulangan).
3. Jika nominal `null`, operator elvis `?: 0` menggantinya dengan 0, sehingga `null` diperlakukan sebagai belum dibayar.
4. Jika nominal lebih dari 0, tagihan dianggap sudah dibayar dan `lunas` bertambah 1.
5. Jika nominal 0 (atau berasal dari `null`), tagihan belum dibayar dan `belum` bertambah 1.
6. Setelah semua data diperiksa, program mengembalikan pasangan `(lunas, belum)`.

**Contoh:** `data = [50000, null, 0, 25000]` menghasilkan `lunas = 2` (50000 dan 25000) dan `belum = 2` (null dan 0), sehingga hasilnya `Pair(2, 2)`.
