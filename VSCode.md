# 🖥️ Install Visual Studio Code di Lubuntu

Panduan lengkap install VS Code di Lubuntu menggunakan **Repository Resmi Microsoft** (mendukung auto-update).

---

## 📋 Prasyarat

- Lubuntu (berbasis Ubuntu/Debian)
- Akses terminal
- Koneksi internet

---

## 🚀 Langkah-langkah Install

### Step 1 — Install Dependensi

```bash
sudo apt install wget gpg apt-transport-https
```

Perintah ini menginstall tiga package yang dibutuhkan:

| Package | Fungsi |
|---|---|
| `wget` | Download file dari internet via terminal |
| `gpg` | Verifikasi tanda tangan digital (keamanan) |
| `apt-transport-https` | Mengizinkan `apt` download dari URL HTTPS |

---

### Step 2 — Import GPG Key Microsoft

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | \
sudo gpg --dearmor -o /usr/share/keyrings/microsoft-archive-keyring.gpg
```

**Penjelasan:**

| Bagian | Keterangan |
|---|---|
| `wget -qO-` | Download file, tampilkan ke stdout (tidak disimpan) |
| `\|` (pipe) | Kirim output `wget` ke perintah berikutnya |
| `gpg --dearmor` | Konversi GPG key dari format teks ke format biner |
| `-o /usr/share/keyrings/...` | Simpan hasil ke lokasi trusted keys sistem |

> **Kenapa perlu GPG Key?**
> Sistem menggunakannya untuk **memverifikasi** bahwa package yang didownload benar-benar dari Microsoft, bukan dari sumber yang tidak dikenal atau berbahaya.

---

### Step 3 — Tambahkan Repository Microsoft

```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" | \
sudo tee /etc/apt/sources.list.d/vscode.list
```

**Penjelasan:**

| Bagian | Keterangan |
|---|---|
| `deb` | Tipe repository (binary package) |
| `arch=amd64` | Arsitektur CPU yang digunakan (64-bit) |
| `signed-by=...` | Verifikasi package menggunakan GPG key yang sudah diimport |
| `stable main` | Channel distribusi: versi stabil (bukan beta/insider) |
| `tee /etc/apt/sources.list.d/vscode.list` | Tulis teks ke file daftar repository |

> **Catatan:** Pastikan menggunakan path `/usr/share/keyrings/` (bukan `/etc/apt/keyrings/`) agar tidak terjadi error `Invalid value set for option Signed-By`.

---

### Step 4 — Update Daftar Package

```bash
sudo apt update
```

Perintah ini membaca ulang semua daftar repository, termasuk repository Microsoft yang baru saja ditambahkan. Ini wajib dilakukan sebelum install agar sistem mengenali package `code`.

---

### Step 5 — Install VS Code

```bash
sudo apt install code
```

Proses ini akan mendownload dan menginstall VS Code beserta semua dependensinya secara otomatis.

---

## ✅ Verifikasi Instalasi

Setelah install selesai, jalankan:

```bash
code --version
```

Contoh output:
```
1.89.0
abc12345...
x64
```

Untuk membuka VS Code langsung dari terminal:

```bash
code .         # Buka VS Code di folder saat ini
code file.py   # Buka file tertentu
```

---

## 🔄 Update VS Code

Karena menggunakan repository resmi, VS Code bisa diupdate bersamaan dengan package sistem lainnya:

```bash
sudo apt update && sudo apt upgrade
```

---

## ❌ Troubleshooting

### Error: `Invalid value set for option Signed-By`

Terjadi jika sebelumnya menggunakan path `/etc/apt/keyrings/`. Solusinya:

**1. Hapus file lama:**
```bash
sudo rm -f /etc/apt/sources.list.d/vscode.list
sudo rm -f /etc/apt/keyrings/packages.microsoft.gpg
```

**2. Ulangi dari Step 2** menggunakan path `/usr/share/keyrings/`.

---

### Alternatif: Install via File `.deb`

Jika metode repository masih bermasalah, bisa install manual via file `.deb`:

```bash
# 1. Download file .deb
wget -O vscode.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64"

# 2. Install
sudo dpkg -i vscode.deb

# 3. Perbaiki dependensi jika ada error
sudo apt install -f
```

---

## 📌 Catatan

- Metode repository **direkomendasikan** karena mendukung auto-update
- VS Code membutuhkan arsitektur **64-bit** (`amd64`)
- Jika menggunakan Lubuntu versi lama (32-bit), pertimbangkan alternatif seperti [VSCodium](https://vscodium.com/) atau [Code - OSS](https://github.com/microsoft/vscode)
