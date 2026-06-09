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
