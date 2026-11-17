Kernel LineageOS 22.2 (Android 15) untuk Redmi Note 9 Pro / miatoll (joyeuse, curtana, excalibur, gram), kernel 4.14.336, ThinLTO, clang 19.0.1 r536225 (sama seperti kernel stock LineageOS), CFI off. Driver sentuh stock (ft8756_spi, tanpa patch) — perbaikan sentuh v1–v6 dan driver focaltech_touch sudah dicoba dan tidak memperbaiki sentuh, jadi dihapus dari repo ini.

Isi varian ini (sudah termasuk v1.0-lto):
- APatch sudah dipatch ke kernel (kpimg-android 0.13.9) — root tetap ada setelah flash, aplikasi APatch harus tetap terpasang. Tidak perlu patch ulang boot.img.
- DroidSpaces: opsi kernel untuk container (19/21 opsi utama aktif; CFS_BANDWIDTH tidak ada karena SCHED_WALT, CGROUP_NS tidak ada di kernel 4.14 — keduanya opsional, limit --cpus tidak tersedia).
- File boot.img (boot-lto-droidspaces.img) di release ini: boot.img siap flash (stock boot.img + kernel ini), bisa dites dulu dengan `fastboot boot boot-lto-droidspaces.img` tanpa flash.

Catatan:
- DroidSpaces + APatch sudah dikonfirmasi berfungsi di HP.

Cara flash (backup Boot dulu):
1. Backup partisi Boot di recovery.
2. Flash zip ini.
3. Reboot.
4. Kalau ada masalah, restore backup Boot dari recovery.

Cek DroidSpaces setelah flash: aplikasi DroidSpaces > Settings > Requirements > Check Requirements.
