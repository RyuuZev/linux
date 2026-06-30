# Perintah krusial

## Untuk menginstall VMware Tools
Untuk distro Ubuntu/Lubuntu modern, open-vm-tools-desktop biasanya yang bikin fitur clipboard, drag & drop, dan shared folder jalan.
```bash
sudo apt install open-vm-tools open-vm-tools-desktop -y
```
---

## Benerin Resolusi yang hancur karena install open vm tools
Perintah terinisebut digunakan untuk menginstal driver grafis (video) virtual VMware pada sistem operasi Linux berbasis Debian/Ubuntu. Fungsi utamanya adalah agar tampilan antarmuka (GUI) Linux berjalan optimal dan lancar saat dijalankan sebagai Virtual Machine (VM) di dalam software simulasi seperti VMware Workstation atau VMware ESXi.
```bash
sudo apt install xserver-xorg-video-vmware -y
```

## Buat xorg.conf dengan PreferredMode (Fix Permanen)
Cara ini mengubah preferred resolution langsung di level Xorg, sehingga LightDM tidak bisa mereset ke 800x600 lagi.  

### Buat file konfigurasi Xorg:

```bash
sudo nano /etc/X11/xorg.conf.d/10-monitor.conf
```

## Xorg
```bash
sudo apt install xserver-xorg-video-vmware

```
Bikin file
```bash
sudo mkdir -p /etc/X11/xorg.conf.d/
sudo nano /etc/X11/xorg.conf.d/10-vmware.conf
```

Isi dengan
```bash
Section "Device"
    Identifier "VMware SVGA"
    Driver     "modesetting"
EndSection
```


Cek log konfirmasi root cause
```bash
grep -i "randr\|kms\|modesetting" /var/log/Xorg.0.log
```


## Untuk menampilkan share folder
```bash
sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

Biar otomatis

```bash
sudo nano /etc/fstab
```

lalu tambahin ini di bagian paling bawah
```bash
.host:/ /mnt/hgfs fuse.vmhgfs-fuse defaults,allow_other 0 0
```



Cut folder

```bash
sudo chown -R user:user /home/eggs/mnt
```
# Install Fresh Eggs
Fresh Eggs ini tool buat membuat OS linux yang sedang dipakai sekarang untuk menjadi file ISO kembali, tujuannya biar hasil modifikasi OS bisa dibikin menjadi file ISO utuh agar bisa diinstall ulang kapan aja.  
Memakai repository https://github.com/pieroproietti/fresh-eggs

## STEP 1 — Update & Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
```
`sudo apt update` → menyegarkan daftar paket dari internet (bukan install apa-apa, hanya refresh info).
`sudo apt upgrade -y` → mengupgrade semua paket yang sudah usang. Flag `-y` artinya otomatis jawab "yes" tanpa konfirmasi manual.


```bash
sudo apt install -y git curl initramfs-tools squashfs-tools xorriso
```

Penjelasan tiap paket:

- `git` → untuk clone repository fresh-eggs dari GitHub  
- `curl` → untuk download file dari internet (dipakai oleh script fresh-eggs)  
- `initramfs-tools` → tools untuk membuat/mengelola initramfs (file sistem awal saat boot). Ini yang menyebabkan error sebelumnya kalau tidak ada.  
- `squashfs-tools` → untuk mengkompresi seluruh isi sistem menjadi satu file `.squashfs` yang nanti dimasukkan ke ISO
- `xorriso` → untuk membuat file ISO dari folder yang sudah disiapkan


## STEP 2 — Antisipasi Error initramfs

```bash
sudo mkdir -p /etc/initramfs-tools/conf.d
```

`mkdir` = make directory (buat folder).
Flag `-p` = buat semua folder parent jika belum ada, dan tidak error kalau foldernya sudah ada.
Folder `/etc/initramfs-tools/conf.d/` ini kadang tidak otomatis ada di instalasi Lubuntu minimal, tapi dibutuhkan oleh eggs.

```bash
sudo touch /etc/initramfs-tools/conf.d/eggs-safe-initramfs.conf
```
`touch` = membuat file kosong. Perintah ini membuat file konfigurasi yang dicari eggs tapi tidak ditemukan — penyebab error `ENOENT`.
```bash
sudo update-initramfs -u
```
Meregenerasi initramfs agar perubahan konfigurasi di atas langsung berlaku.
Flag `-u` = update (perbarui yang sudah ada, bukan buat dari nol).

## STEP 3 — Clone & Jalankan Fresh Eggs

```bash
git clone https://github.com/pieroproietti/fresh-eggs
```

`git clone` = mengunduh seluruh isi repository dari GitHub ke folder lokal bernama `fresh-eggs` di direktori aktif saat ini.

```bash
cd fresh-eggs
```
`cd` = change directory, masuk ke dalam folder `fresh-eggs` yang baru di-clone.

```bash
sudo ./fresh-eggs.sh
```
Menjalankan script instalasi.

`sudo` = jalankan sebagai root (administrator), karena script ini perlu akses penuh ke sistem  
`./` = jalankan file di direktori saat ini  
Script ini akan otomatis: menambahkan repository NodeSource (untuk Node.js), install Node.js, install penguins-eggs dan semua dependensinya


## STEP 4 — Verifikasi Instalasi

```bash
eggs version
```
Mengecek apakah eggs berhasil terinstall dan menampilkan versinya.
Output normal   
Contohnya: `penguins-eggs/10.1.x linux-x64 node-v18.x.x`
Kalau command tidak ditemukan, berarti instalasi gagal → ulangi STEP 3.


## STEP 5 — Konfigurasi Awal Eggs

```bash
sudo eggs dad -d
```
`dad` = perintah konfigurasi otomatis di eggs (singkatan dari "default automatic deployment").  
Flag `-d` = default, pakai semua nilai standar tanpa banyak tanya.  
Perintah ini membuat file konfigurasi eggs di `/etc/penguins-eggs.d/` berisi pengaturan seperti nama distro, kompresi, lokasi output ISO, dll.  
Cukup tekan Enter saja untuk setiap pertanyaan yang muncul.

## STEP 6 — Install Calamares (GUI Installer)

```bash
sudo eggs calamares --install
```
Calamares adalah installer grafis yang akan ditampilkan saat pengguna booting dari ISO hasil remaster dan ingin menginstall ke harddisk.
Tanpa ini, ISO tetap bisa dipakai sebagai Live system, tapi tidak ada GUI untuk install ke harddisk (hanya bisa install via terminal).  
Flag `--install` = lakukan instalasi calamares dan buat konfigurasinya otomatis.

## STEP 7 — Bersihkan Cache

```bash
sudo eggs tools clean
```
Membersihkan:

- Cache apt yang tidak perlu  
- File temporary dari proses eggs sebelumnya  
- Log lama  

Tujuannya agar ISO yang dihasilkan lebih kecil dan tidak ada sampah dari sesi sebelumnya. Selalu jalankan ini sebelum `produce`.

## STEP 8 — Buat ISO

```bash
sudo eggs produce --verbose
```
Membuat ISO dari sistem yang sedang berjalan, tanpa menyertakan data pribadi pengguna (folder `/home`, dokumen, foto, dll).
Hasilnya adalah ISO "fresh" seperti distro baru — cocok untuk dibagikan ke orang lain.  
Flag `--verbose` = tampilkan semua proses secara detail di terminal, berguna untuk memantau progress dan mendeteksi error.














