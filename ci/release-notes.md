Kernel LineageOS 22.2 (Android 15) untuk Redmi Note 9 Pro / miatoll (joyeuse, curtana, excalibur, gram), kernel 4.14.336, ThinLTO, clang 19.0.1 r536225 (sama seperti kernel stock LineageOS), CFI off.

Isi varian ini (sudah termasuk v1.0-lto):
- Perbaikan sentuh FT8756 (layar pengganti) v6: 20 detik setelah boot, kernel memberi pulsa reset ke IC sentuh lalu menjalankan suspend→resume, jadi sentuh hidup tanpa perlu dimatikan-dinyalakan layarnya. Tunggu ~30 detik setelah boot sebelum menilai sentuh.
- File boot.img (boot-lto-root.img) di release ini: boot.img siap flash (stock boot.img + kernel ini), bisa dites dulu dengan `fastboot boot boot-lto-root.img` tanpa flash.

Catatan:
- Varian ini TIDAK pre-patch APatch. Setelah flash, patch boot.img dengan aplikasi APatch kalau mau root.
- Status tes: v5 boot normal tapi sentuh masih mati. v6 masih dalam pengujian.

Cara flash (backup Boot dulu):
1. Backup partisi Boot di recovery.
2. Flash zip ini.
3. Reboot, tunggu ~30 detik, cek sentuh langsung.
4. Kalau ada masalah, restore backup Boot dari recovery.
