Kernel LineageOS 22.2 (Android 15) untuk Redmi Note 9 Pro / miatoll (joyeuse, curtana, excalibur, gram), kernel 4.14.336, ThinLTO, clang 19.0.0 r530567.

Isi varian ini:
- Perbaikan sentuh FT8756 (layar pengganti): sentuh tidak perlu lagi dimatikan-dinyalakan tiap boot.
- File boot.img (boot-lto.img) di release ini: boot.img siap flash (stock boot.img + kernel ini), bisa dites dulu dengan `fastboot boot boot-lto.img` tanpa flash.

Catatan:
- Varian ini hanya kernel bersih (tanpa root). Setelah flash, patch ulang boot.img dengan aplikasi APatch kalau mau root kembali.
- BELUM diuji boot di HP — pertama kali flash, cek sentuh langsung tanpa toggle layar.

Cara flash (backup Boot dulu):
1. Backup partisi Boot di recovery.
2. Flash zip ini.
3. Reboot, cek sentuh langsung.
4. Kalau ada masalah, restore backup Boot dari recovery.
