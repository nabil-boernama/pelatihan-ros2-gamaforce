# Pelatihan ROS 2 Jazzy: Tim UAV

Repositori ini berisi semua perintah dan kode yang dipakai saat pelatihan ROS 2, supaya kamu tinggal menyalinnya dan tidak perlu mengetik dari nol.

**Lingkungan:** Ubuntu 24.04, ROS 2 Jazzy Jalisco, Python 3.

## Daftar Isi

1. [Cek Environment](#1-cek-environment)
2. [Praktik CLI dengan Turtlesim](#2-praktik-cli-dengan-turtlesim)
3. [Menulis Node Pertamamu](#3-menulis-node-pertamamu)
4. [Challenge](#4-challenge)
5. [Troubleshooting](#5-troubleshooting)
6. [Cheat Sheet](#6-cheat-sheet)

## Cara Memakai README Ini

- Klik ikon **copy** di pojok kanan atas setiap kotak kode, lalu tempel di terminal dengan `Ctrl+Shift+V`.
- Salin **satu kotak kode sekali**, lalu **baca hasilnya** sebelum lanjut. Menyalin tanpa membaca hasilnya membuatmu tidak belajar apa pun.
- Sebelum menjalankan perintah, **tebak dulu** apa hasilnya. Selisih antara tebakanmu dan hasil sebenarnya adalah tempat kamu belajar paling banyak.
- Tab terminal baru: `Ctrl+Shift+T`. Jendela terminal baru: `Ctrl+Alt+T`.
- Hentikan program yang sedang berjalan dengan `Ctrl+C`.
- **Setiap terminal baru** harus memakai `ROS_DOMAIN_ID` yang sama dan sudah di-source (lihat bagian 1).

---

## 1. Cek Environment

### 1.1 Atur Domain ID

Setiap peserta memakai **nomor unik** dari pemateri agar topic antar peserta tidak bercampur di Wi-Fi yang sama.

> Ganti `17` dengan nomor milikmu.

```bash
export ROS_DOMAIN_ID=17
```

Agar otomatis di setiap terminal baru (jalankan **sekali saja**, dengan nomormu):

```bash
echo 'export ROS_DOMAIN_ID=17' >> ~/.bashrc
```

### 1.2 Pastikan ROS 2 Aktif

```bash
source /opt/ros/jazzy/setup.bash
```

Agar otomatis di setiap terminal baru (jalankan **sekali saja**):

```bash
echo 'source /opt/ros/jazzy/setup.bash' >> ~/.bashrc
```

### 1.3 Verifikasi

```bash
printenv ROS_DISTRO
```

Hasil yang diharapkan: `jazzy`.

```bash
ros2 doctor
```

Peringatan boleh ada. Yang penting pemeriksaan inti lolos.

### 1.4 Uji Talker dan Listener

**Terminal 1**

```bash
ros2 run demo_nodes_cpp talker
```

**Terminal 2**

```bash
ros2 run demo_nodes_cpp listener
```

Jika Terminal 2 menerima pesan dari Terminal 1, komunikasi berjalan. Hentikan keduanya dengan `Ctrl+C`.

> **Coba sendiri:** ubah `ROS_DOMAIN_ID` di satu terminal (misalnya `export ROS_DOMAIN_ID=99`) lalu jalankan ulang listener. Apa yang terjadi, dan kenapa?

---

## 2. Praktik CLI dengan Turtlesim

Kita memakai **turtlesim**, simulator 2D berisi seekor kura-kura, sebagai "drone latihan".

### 2.1 Jalankan Turtlesim dan Teleop

**Terminal 1**

```bash
ros2 run turtlesim turtlesim_node
```

**Terminal 2**

```bash
ros2 run turtlesim turtle_teleop_key
```

Gerakkan kura-kura dengan tombol panah. Fokus keyboard harus berada di **Terminal 2**.

### 2.2 Node

Buka **Terminal 3**.

```bash
ros2 node list
```

```bash
ros2 node info /turtlesim
```

### 2.3 Topic

```bash
ros2 topic list -t
```

```bash
ros2 topic info /turtle1/cmd_vel
```

Perintah di atas hanya menampilkan jumlah publisher dan subscriber. Tambahkan `-v` untuk melihat nama node-nya:

```bash
ros2 topic info -v /turtle1/cmd_vel
```

Lihat data yang mengalir (gerakkan kura-kura dari teleop sambil melihat angkanya berubah):

```bash
ros2 topic echo /turtle1/pose
```

Hentikan dengan `Ctrl+C`, lalu lihat laju publikasinya:

```bash
ros2 topic hz /turtle1/pose
```

### 2.4 Message dan Perintah Manual

Lihat isi tipe message yang dipakai untuk menggerakkan kura-kura:

```bash
ros2 interface show geometry_msgs/msg/Twist
```

Lihat isi tipe message posisi:

```bash
ros2 interface show turtlesim/msg/Pose
```

Kirim satu perintah gerak:

```bash
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist '{linear: {x: 2.0}}'
```

Kirim perintah terus-menerus satu kali per detik (kura-kura berputar membentuk lingkaran). Hentikan dengan `Ctrl+C`:

```bash
ros2 topic pub -r 1 /turtle1/cmd_vel geometry_msgs/msg/Twist '{linear: {x: 2.0}, angular: {z: 1.8}}'
```

> **Pertanyaan:** kenapa kura-kura berhenti kalau perintah tidak dikirim terus?

### 2.5 Service

```bash
ros2 service list -t
```

Hapus jejak garis di layar:

```bash
ros2 service call /clear std_srvs/srv/Empty
```

Munculkan kura-kura kedua:

```bash
ros2 service call /spawn turtlesim/srv/Spawn '{x: 2.0, y: 2.0, theta: 0.2, name: turtle2}'
```

Ubah warna pena `turtle1` jadi merah, lalu gerakkan `turtle1` dari teleop:

```bash
ros2 service call /turtle1/set_pen turtlesim/srv/SetPen '{r: 255, g: 0, b: 0, width: 4, off: 0}'
```

Kembalikan semuanya ke kondisi awal:

```bash
ros2 service call /reset std_srvs/srv/Empty
```

### 2.6 Action

```bash
ros2 action list -t
```

```bash
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute '{theta: 1.57}' --feedback
```

Perhatikan: selama kura-kura berputar, terminal menampilkan **feedback**, lalu **result** di akhir.

### 2.7 Lihat ROS Graph

```bash
rqt_graph
```

Tekan tombol refresh, lalu pilih tampilan **Nodes/Topics (all)**.

### 2.8 Misi Mini

1. Buat kura-kura membentuk lingkaran dengan `ros2 topic pub`.
2. Spawn `turtle2`, lalu kendalikan dengan teleop yang di-remap:

```bash
ros2 run turtlesim turtle_teleop_key --ros-args -r turtle1/cmd_vel:=turtle2/cmd_vel
```

3. Setelah `turtle2` muncul, jalankan `ros2 topic list` lagi. Topic apa saja yang baru?
4. Ubah warna pena `turtle1` jadi merah, lalu gambar huruf L.

---

## 3. Menulis Node Pertamamu

Sampai sini kamu mengendalikan kura-kura dari terminal. Sekarang kita menulis program yang melakukannya untuk kita.

### 3.1 Buat Workspace dan Package

```bash
mkdir -p ~/turtle_ws/src
```

```bash
cd ~/turtle_ws/src
```

```bash
ros2 pkg create --build-type ament_python --license Apache-2.0 turtle_challenge --dependencies rclpy geometry_msgs turtlesim
```

Package Python bernama `turtle_challenge` sudah dibuat. Struktur pentingnya:

```
turtle_ws/src/turtle_challenge/
    package.xml
    setup.py
    turtle_challenge/
        __init__.py        <- file Python kita diletakkan di folder ini
```

### 3.2 Daftarkan Semua Node di setup.py

Buka `setup.py`:

```bash
nano ~/turtle_ws/src/turtle_challenge/setup.py
```

Cari bagian `entry_points`. Awalnya kira-kira seperti ini:

```python
    entry_points={
        'console_scripts': [
        ],
    },
```

Ganti dengan ini (kita daftarkan semua node sekaligus supaya tidak perlu mengedit berulang kali):

```python
    entry_points={
        'console_scripts': [
            'circle = turtle_challenge.circle:main',
            'pose_printer = turtle_challenge.pose_printer:main',
            'wall_stopper = turtle_challenge.wall_stopper:main',
            'my_challenge = turtle_challenge.my_challenge:main',
        ],
    },
```

Simpan di `nano` dengan `Ctrl+O`, `Enter`, lalu keluar dengan `Ctrl+X`.

> Format pendaftaran: `nama_perintah = nama_package.nama_file:main`. Setiap node baru yang kamu buat sendiri harus didaftarkan dengan cara yang sama, lalu `colcon build` ulang.

### 3.3 Node 1: Publisher (Menggerakkan Kura-kura)

Perintah di bawah membuat file `circle.py` sekaligus. Salin semuanya dan tempel di terminal.

```bash
cat > ~/turtle_ws/src/turtle_challenge/turtle_challenge/circle.py <<'EOF'
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist


class Circle(Node):
    def __init__(self):
        super().__init__('circle')
        self.pub = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.timer = self.create_timer(0.1, self.on_timer)  # 10 kali per detik

    def on_timer(self):
        msg = Twist()
        msg.linear.x = 2.0     # maju
        msg.angular.z = 1.0    # sambil berputar
        self.pub.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    node = Circle()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
EOF
```

Build dan source:

```bash
cd ~/turtle_ws && colcon build --symlink-install && source install/setup.bash
```

Jalankan (pastikan `turtlesim_node` masih berjalan di terminal lain):

```bash
ros2 run turtle_challenge circle
```

Kura-kura berputar membentuk lingkaran.

> **Coba sendiri:** ubah angka kecepatan di `circle.py` sampai lingkarannya lebih kecil, lalu lebih besar. Karena memakai `--symlink-install`, tidak perlu build ulang. Cukup simpan file lalu jalankan ulang node.

> **Setiap terminal baru** perlu `source ~/turtle_ws/install/setup.bash` agar `ros2 run turtle_challenge ...` ditemukan. Agar otomatis (jalankan sekali):
>
> ```bash
> echo 'source ~/turtle_ws/install/setup.bash' >> ~/.bashrc
> ```

### 3.4 Node 2: Subscriber (Membaca Posisi)

```bash
cat > ~/turtle_ws/src/turtle_challenge/turtle_challenge/pose_printer.py <<'EOF'
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose


class PosePrinter(Node):
    def __init__(self):
        super().__init__('pose_printer')
        self.sub = self.create_subscription(Pose, '/turtle1/pose', self.on_pose, 10)

    def on_pose(self, msg):
        self.get_logger().info(
            f'x={msg.x:.2f} y={msg.y:.2f} theta={msg.theta:.2f}',
            throttle_duration_sec=0.5)


def main(args=None):
    rclpy.init(args=args)
    node = PosePrinter()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
EOF
```

Jalankan (gerakkan kura-kura dari teleop di terminal lain untuk melihat angkanya berubah):

```bash
ros2 run turtle_challenge pose_printer
```

Fakta yang akan sering kamu pakai di challenge:

- Area turtlesim: `x` dan `y` sekitar **0 sampai 11**.
- Titik awal: sekitar **(5, 5)**, menghadap kanan (`theta = 0`).
- `theta` dalam **radian**, rentangnya kira-kira dari minus 3,14 sampai 3,14.

### 3.5 Node 3: Baca, Hitung, Kirim

Node ini membaca posisi, memutuskan, lalu mengirim perintah. Ini **pola dasar semua challenge**.

```bash
cat > ~/turtle_ws/src/turtle_challenge/turtle_challenge/wall_stopper.py <<'EOF'
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from turtlesim.msg import Pose


class WallStopper(Node):
    def __init__(self):
        super().__init__('wall_stopper')
        self.pub = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.sub = self.create_subscription(Pose, '/turtle1/pose', self.on_pose, 10)

    def on_pose(self, pose):
        cmd = Twist()
        if pose.x < 9.0:
            cmd.linear.x = 1.5
        else:
            cmd.linear.x = 0.0      # berhenti sebelum menabrak dinding
        self.pub.publish(cmd)


def main(args=None):
    rclpy.init(args=args)
    node = WallStopper()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
EOF
```

Kembalikan kura-kura ke titik awal, lalu jalankan:

```bash
ros2 service call /reset std_srvs/srv/Empty
```

```bash
ros2 run turtle_challenge wall_stopper
```

Kura-kura maju dari tengah lalu berhenti di sekitar `x = 9`.

Polanya:

1. **Baca** keadaan saat ini (`Pose`).
2. **Hitung** apa yang harus dilakukan (`if`, selisih jarak, sudut).
3. **Kirim** perintah (`Twist`).

---

## 4. Challenge

### 4.1 Aturan Main

- Bekerja **berpasangan**: satu **driver** (mengetik) dan satu **navigator** (membaca dan memberi ide). Bergantian setiap 20 menit.
- **Wajib sampai Level 2.** Level 3 dan 4 untuk yang cepat. Bonus menambah poin.
- Boleh membuka dokumentasi dan internet. Mentor membantu lewat petunjuk bertahap, bukan jawaban langsung.
- Verifikasi: panggil mentor, lalu jalankan node-mu di depannya dari kondisi awal (`/reset`).

### 4.2 Daftar Challenge

| Level | Tugas | Kriteria lulus |
|---|---|---|
| 1 | **Persegi.** Gambar persegi lalu berhenti | Bentuk mendekati persegi, kembali dekat titik awal |
| 2 | **Ke titik.** Pergi ke titik (8, 3) lalu berhenti | Berhenti dalam jarak sekitar 0,2 dari titik, tanpa menabrak dinding |
| 3 | **Patroli.** Kunjungi 4 titik berurutan lalu berhenti | Semua titik dilewati sesuai urutan |
| 4 | **Lawnmower.** Sapu area dengan jalur bolak-balik | Jejak menutupi sebagian besar area tanpa celah besar |
| Bonus | **Pengejar.** `turtle2` mengejar `turtle1` yang digerakkan teleop | `turtle2` terus mendekati `turtle1` |

### 4.3 Kerangka Awal

Buat file kerangka berikut. Kamu hanya perlu mengisi bagian `TODO` di `on_timer`.

```bash
cat > ~/turtle_ws/src/turtle_challenge/turtle_challenge/my_challenge.py <<'EOF'
import math
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from turtlesim.msg import Pose


class MyChallenge(Node):
    def __init__(self):
        super().__init__('my_challenge')
        self.pub = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.sub = self.create_subscription(Pose, '/turtle1/pose', self.on_pose, 10)
        self.pose = None                      # posisi terakhir, diisi oleh on_pose
        self.timer = self.create_timer(0.1, self.on_timer)

    def on_pose(self, msg):
        self.pose = msg

    def on_timer(self):
        if self.pose is None:
            return                            # belum ada data posisi
        cmd = Twist()
        # TODO: hitung cmd.linear.x dan cmd.angular.z di sini
        self.pub.publish(cmd)


def main(args=None):
    rclpy.init(args=args)
    node = MyChallenge()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.try_shutdown()


if __name__ == '__main__':
    main()
EOF
```

Jalankan:

```bash
ros2 run turtle_challenge my_challenge
```

### 4.4 Kotak Alat

- Area turtlesim: `x` dan `y` sekitar 0 sampai 11. Titik awal sekitar (5, 5), `theta = 0` (menghadap kanan).
- Putar 90 derajat sama dengan sekitar **1,57 radian**.
- Waktu putar sama dengan sudut dibagi kecepatan angular. Jarak maju sama dengan kecepatan linear dikali waktu.
- Jarak ke tujuan: `math.hypot(dx, dy)`, dengan `dx = goal_x - x` dan `dy = goal_y - y`.
- Arah ke tujuan: `math.atan2(dy, dx)`.
- Selisih sudut harus dinormalisasi ke rentang minus pi sampai pi, kalau tidak kura-kura bisa memilih putaran yang terjauh:

```python
e = math.atan2(math.sin(e), math.cos(e))
```

Reset kura-kura ke titik awal (lakukan sebelum **setiap** percobaan):

```bash
ros2 service call /reset std_srvs/srv/Empty
```

Hapus jejak garis:

```bash
ros2 service call /clear std_srvs/srv/Empty
```

Munculkan `turtle2` (untuk Bonus):

```bash
ros2 service call /spawn turtlesim/srv/Spawn '{x: 2.0, y: 2.0, theta: 0.0, name: turtle2}'
```

### 4.5 Strategi

- **Mulai dari yang paling kecil.** Pastikan node bisa membuat kura-kura bergerak sedikit, baru tambah logika.
- **Pecah masalah.** Level 2 sebenarnya dua masalah kecil: menghadap tujuan, lalu maju ke tujuan.
- **Cetak nilai.** Kalau perilaku aneh, tampilkan jarak dan selisih sudut dengan `self.get_logger().info(..., throttle_duration_sec=0.5)`.
- **Uji dari kondisi yang sama.** Panggil `/reset` sebelum tiap percobaan.
- **Gunakan kembali.** Level 3 adalah Level 2 ditambah daftar tujuan. Level 4 adalah Level 3 dengan daftar tujuan yang dibuat lewat loop. Simpan versi yang sudah jalan sebelum mengembangkannya.
- **Jangan pakai `while True` atau `time.sleep()` di dalam callback.** Itu membuat node macet. Pakai timer atau callback.

---

## 5. Troubleshooting

| Gejala | Penyebab umum | Solusi |
|---|---|---|
| `ros2: command not found` | Belum source ROS 2 | `source /opt/ros/jazzy/setup.bash` |
| `Package 'turtle_challenge' not found` | Belum source workspace di terminal ini | `source ~/turtle_ws/install/setup.bash` |
| `No executable found` | Entry point belum didaftarkan atau belum build | Cek `setup.py`, lalu `cd ~/turtle_ws && colcon build --symlink-install` |
| Kode diubah tapi perilaku tidak berubah | File belum disimpan, atau node lama masih berjalan | Simpan file, `Ctrl+C`, jalankan ulang node |
| `ros2 topic list` kosong atau topic milik orang lain muncul | `ROS_DOMAIN_ID` berbeda atau bertabrakan | Pastikan nomor unik dan sama di semua terminalmu (`echo $ROS_DOMAIN_ID`) |
| Teleop tidak menggerakkan kura-kura | Fokus keyboard tidak di terminal teleop | Klik terminal teleop, lalu tekan tombol panah |
| Kura-kura tidak bergerak dari node-mu | Nama topic salah, atau `turtlesim_node` belum jalan | `ros2 topic list`, lalu `ros2 topic info -v /turtle1/cmd_vel` |
| Kura-kura terus bergerak setelah node dihentikan | Turtlesim mengingat perintah terakhir | `ros2 service call /reset std_srvs/srv/Empty` |
| Perilaku aneh dan kacau | Dua node mengirim ke `cmd_vel` bersamaan | `ros2 node list`, hentikan node yang tidak perlu |
| Error Python saat node start | Salah indentasi atau salah ketik | Baca error dari **baris pertama**, lihat nomor barisnya |
| `colcon build` dijalankan dari folder yang salah | Harus dari root workspace | `cd ~/turtle_ws` lalu build ulang |

Urutan debugging yang disarankan:

```bash
ros2 node list
```

```bash
ros2 topic list -t
```

```bash
ros2 topic echo /turtle1/pose
```

Lalu cetak nilai penting di dalam kodemu dengan `self.get_logger().info(...)`.

---

## 6. Cheat Sheet

| Perintah | Fungsi |
|---|---|
| `ros2 run <paket> <executable>` | Jalankan satu node |
| `ros2 node list` | Daftar node yang berjalan |
| `ros2 node info <node>` | Topic, service, dan action milik sebuah node |
| `ros2 topic list -t` | Daftar topic beserta tipenya |
| `ros2 topic echo <topic>` | Lihat isi pesan yang mengalir |
| `ros2 topic info -v <topic>` | Publisher dan subscriber beserta nama node |
| `ros2 topic hz <topic>` | Laju publikasi |
| `ros2 topic pub -r 1 <topic> <tipe> '<yaml>'` | Kirim pesan terus-menerus (`--once` untuk sekali) |
| `ros2 interface show <tipe>` | Lihat isi message, service, atau action |
| `ros2 service list -t` | Daftar service |
| `ros2 service call <service> <tipe> '<yaml>'` | Panggil service |
| `ros2 action list -t` | Daftar action |
| `ros2 action send_goal <action> <tipe> '<yaml>' --feedback` | Kirim goal action |
| `rqt_graph` | Visualisasi ROS graph |
| `colcon build --symlink-install` | Build workspace (jalankan dari root workspace) |
| `source install/setup.bash` | Muat hasil build ke terminal ini |

## Referensi

- Dokumentasi resmi ROS 2: [docs.ros.org](https://docs.ros.org), pilih distro Jazzy, bagian **Tutorials**
- Dokumentasi paket turtlesim dan tutorial resminya di docs.ros.org
