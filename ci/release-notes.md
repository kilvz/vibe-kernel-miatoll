Kernel LineageOS 22.2 (Android 15) untuk Redmi Note 9 Pro / miatoll (joyeuse, curtana, excalibur, gram), kernel 4.14.336, ThinLTO, clang 19.0.0 r530567.

Isi varian ini (sudah termasuk v1.0-lto-root):
- Perbaikan sentuh FT8756 (layar pengganti): sentuh tidak perlu lagi dimatikan-dinyalakan tiap boot.
- APatch sudah dipatch ke kernel (kpimg 0.13.9) — root tetap ada setelah flash, aplikasi APatch harus tetap terpasang. Tidak perlu patch ulang boot.img.
- DroidSpaces: opsi kernel untuk container (19/21 opsi utama aktif; CFS_BANDWIDTH tidak ada karena SCHED_WALT, CGROUP_NS tidak ada di kernel 4.14 — keduanya opsional, limit --cpus tidak tersedia).

Catatan:
- BELUM diuji boot di HP — pertama kali flash, cek sentuh langsung tanpa toggle layar.

Cara flash (backup Boot dulu):
1. Backup partisi Boot di recovery.
2. Flash zip ini.
3. Reboot, cek sentuh langsung.
4. Kalau ada masalah, restore backup Boot dari recovery.

Cek DroidSpaces setelah flash: aplikasi DroidSpaces > Settings > Requirements > Check Requirements.

- Dibangun dengan clang r536225 (sama seperti kernel stock LineageOS) agar APatch bisa boot.
