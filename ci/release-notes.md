Kernel LineageOS 22.2 (Android 15) untuk Redmi Note 9 Pro / miatoll (joyeuse, curtana, excalibur, gram), kernel 4.14.336, ThinLTO, clang 19.0.1 r536225 (sama seperti kernel stock LineageOS), CFI off (kompatibel APatch). Driver sentuh stock (ft8756_spi, tanpa patch) — perbaikan sentuh v1–v6 dan driver focaltech_touch sudah dicoba dan tidak memperbaiki sentuh, jadi dihapus dari repo ini.

Isi varian ini (sudah termasuk v1.0-lto):
- File boot.img (boot-lto-root.img) di release ini: boot.img siap flash (stock boot.img + kernel ini), bisa dites dulu dengan `fastboot boot boot-lto-root.img` tanpa flash.

Catatan:
- Varian ini TIDAK pre-patch APatch. Setelah flash, patch boot.img dengan aplikasi APatch kalau mau root.

Cara flash (backup Boot dulu):
1. Backup partisi Boot di recovery.
2. Flash zip ini.
3. Reboot.
4. Kalau ada masalah, restore backup Boot dari recovery.
