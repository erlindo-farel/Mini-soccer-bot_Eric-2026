# Robot Mini Soccer - ERIC

Dokumentasi dua robot mini soccer yang kami buat untuk mengikuti kompetisi robotik **ERIC**. Robot dikendalikan secara manual menggunakan stik PS4 melalui Bluetooth dan digerakkan oleh mikrokontroler ESP32.

## 📘 Pendahuluan

**Robot mini soccer** adalah robot berukuran kecil yang dirancang untuk bermain sepak bola di lapangan mini. Robot bertugas menggiring, mengontrol, dan menendang bola ke arah gawang lawan. Dalam kompetisi, robot dikendalikan manual oleh operator, sehingga kelincahan gerak, kecepatan respons, dan kemampuan mengontrol bola sangat menentukan.

Pada proyek ini kami membuat **2 robot** dengan peran berbeda:

| Robot | Keterangan |
|---|---|
| **Robot 1** | Dilengkapi **solenoid sebagai penendang (kicker)** untuk menendang bola |
| **Robot 2** | **Tanpa kicker**, fokus pada menggiring dan menahan bola |

## ⚙️ Fitur Utama

- 🎮 **Dikendalikan stik PS4:** terhubung nirkabel lewat Bluetooth.
- 🛑 **Pengereman saat berjalan:** robot dapat direm secara aktif ketika sedang melaju sehingga berhenti lebih cepat dan presisi.
- 🔒 **Mode tahan posisi (seperti rem tangan):** robot tetap diam bertahan walaupun didorong lawan.
- ⚽ **Penendang solenoid:** menendang bola dengan cepat dan bertenaga (khusus Robot 1).
- 🔄 **Roda tengah bebas (free wheel):** robot dapat berbelok dan berputar dengan lancar.
- 🖨️ **Body hasil 3D print:** ringan, kuat, dan mudah dimodifikasi.

## 🔧 Komponen dan Kegunaannya

| Komponen | Kegunaan |
|---|---|
| **ESP32** | Mikrokontroler utama. Membaca data dari stik PS4 lewat Bluetooth dan mengatur motor serta solenoid |
| **Stik PS4 (DualShock 4)** | Kontroler untuk mengemudikan robot dan menembak bola |
| **Driver motor L298N** | Mengatur arah putaran dan kecepatan motor DC, serta berperan dalam pengereman |
| **Modul penurun tegangan (step-down)** | Menurunkan tegangan baterai menjadi tegangan stabil (5 V) untuk ESP32 |
| **Solenoid** | Penendang bola. Saat diberi arus, batang solenoid terdorong keluar dengan cepat (Robot 1) |
| **Motor DC gearbox (TT motor, bodi plastik kuning)** | Penggerak roda kiri dan kanan dengan torsi yang cukup |
| **Free wheel (caster wheel)** | Roda tengah penyeimbang yang berputar bebas ke segala arah |
| **Baterai 18650 (holder 3 slot)** | Sumber daya utama, tegangan sekitar 11,1 V (3S) |
| **Filamen PLA** | Bahan cetak 3D untuk body/rangka robot |
| **[Transistor/MOSFET + dioda]** | *(isi sesuai yang dipakai)* Menyalakan solenoid dan melindungi rangkaian dari lonjakan tegangan balik |

## 📡 Mekanisme Komunikasi Bluetooth ESP32 dan Stik PS4

1. **ESP32 bertindak sebagai Bluetooth host.** Library Bluetooth pada ESP32 membuat ESP32 mampu "menjadi" perangkat yang menerima koneksi dari stik PS4.
2. **Pairing.** Stik PS4 dipasangkan dengan ESP32 menggunakan alamat MAC Bluetooth ESP32 (dengan library `[PS4Controller / Bluepad32]`). Setelah terpasang, stik akan otomatis tersambung saat tombol **PS** ditekan.
3. **Pengiriman data.** Stik mengirim data secara terus-menerus: posisi joystick analog (kiri/kanan), status tombol, dan trigger.
4. **Pengolahan oleh ESP32.** Program membaca data tersebut lalu menerjemahkannya menjadi perintah:
   - Joystick → arah dan kecepatan motor (lewat sinyal PWM ke L298N)
   - Tombol rem → motor ditahan/di-brake
   - Tombol tembak → solenoid diaktifkan sesaat
5. **Pengaman koneksi.** Jika stik terputus, robot berhenti otomatis agar tidak melaju tak terkendali.

```
Stik PS4 ──(Bluetooth)──► ESP32 ──► L298N ──► Motor kiri & kanan
                            │
                            └────► Driver solenoid ──► Solenoid (kicker)
```

## 🔌 Wiring Diagram

### Jalur Daya

```
Baterai 18650 (3S, ±11,1 V)
   ├──► L298N (terminal +12V dan GND)
   ├──► Modul step-down ──► 5 V ──► ESP32 (pin VIN/5V)
   └──► Solenoid (lewat driver MOSFET/transistor)   [Robot 1]

Semua GND wajib disatukan (baterai, L298N, step-down, ESP32).
```

### Koneksi ESP32 ke L298N

> Nomor pin berikut hanya contoh. **Sesuaikan dengan kode Anda.**

| ESP32 | L298N | Fungsi |
|---|---|---|
| GPIO `[..]` | ENA | PWM kecepatan motor kiri |
| GPIO `[..]` | IN1 | Arah motor kiri |
| GPIO `[..]` | IN2 | Arah motor kiri |
| GPIO `[..]` | ENB | PWM kecepatan motor kanan |
| GPIO `[..]` | IN3 | Arah motor kanan |
| GPIO `[..]` | IN4 | Arah motor kanan |
| GND | GND | Ground bersama |

| L298N | Terhubung ke |
|---|---|
| OUT1, OUT2 | Motor kiri |
| OUT3, OUT4 | Motor kanan |

### Koneksi Solenoid (khusus Robot 1)

| ESP32 | Driver solenoid | Keterangan |
|---|---|---|
| GPIO `[..]` | Gate/Base MOSFET/transistor | Sinyal pemicu tendangan |
| GND | Source/Emitter | Ground bersama |

- Kaki solenoid terhubung ke baterai (+) dan drain/kolektor driver.
- Pasang **dioda flyback** paralel dengan solenoid (katoda ke sisi +) untuk meredam lonjakan tegangan.

### Prinsip Pengereman

- **Rem saat berjalan:** pin IN1/IN2 (dan IN3/IN4) diberi logika yang sama dengan pin Enable aktif, sehingga motor dihubung singkat secara internal dan berhenti cepat.
- **Mode tahan (rem tangan):** saat tidak ada input joystick atau tombol rem ditekan, motor dibiarkan dalam kondisi brake sehingga sulit didorong.

## 📂 Isi Repository

```
├── README.md
├── src/     ← kode Arduino ESP32
├── stl/     ← file model 3D body robot
├── docs/    ← foto robot, wiring diagram, dokumentasi lain
```

## 🏆 Tentang Kompetisi

Robot ini dibuat untuk kompetisi robotik **ERIC** kategori mini soccer. `[Tambahkan tahun, penyelenggara, hasil/peringkat, dan nama anggota tim]`

## 👥 Tim

- `[Nama anggota 1]`
- `[Nama anggota 2]`
