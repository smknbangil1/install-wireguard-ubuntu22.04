Berikut langkah-langkah untuk menginstal dan mengonfigurasi WireGuard sebagai server di Ubuntu 22.04:

---

### 1. **Update Sistem**
Perbarui paket dan repository sistem.
```bash
sudo apt update && sudo apt upgrade -y
```

---

### 2. **Install WireGuard**
Instal paket WireGuard dari repository resmi.
```bash
sudo apt install wireguard -y
```

---

### 3. **Konfigurasi Network**
Pastikan sistem memiliki paket `resolvconf` untuk menangani konfigurasi DNS.
```bash
sudo apt install resolvconf -y
```

---

### 4. **Generate Key Pairs**
WireGuard menggunakan kunci publik dan privat untuk otentikasi.

- Buat folder untuk menyimpan kunci:
  ```bash
  sudo mkdir -p /etc/wireguard
  sudo chmod 700 /etc/wireguard
  ```

- Generate kunci privat dan publik server:
  ```bash
  sudo wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key
  ```

- Tampilkan kunci untuk referensi nanti:
  ```bash
  sudo cat /etc/wireguard/server_private.key
  sudo cat /etc/wireguard/server_public.key
  ```

---

### 5. **Konfigurasi WireGuard**
Buat file konfigurasi server:
```bash
sudo nano /etc/wireguard/wg0.conf
```

Isi file dengan konfigurasi berikut:
```ini
[Interface]
PrivateKey = <server_private_key>  # Ganti dengan kunci privat server
Address = 10.0.0.1/24             # IP jaringan WireGuard
ListenPort = 51820                # Port WireGuard
SaveConfig = true

# Aktifkan forwarding
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o <interface_utama> -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o <interface_utama> -j MASQUERADE
```

- **Ganti:**
  - `<server_private_key>` dengan kunci privat server.
  - `<interface_utama>` dengan interface jaringan server (contoh: `eth0`).

---

### 6. **Aktifkan IP Forwarding**
Aktifkan IP forwarding di sistem:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

---

### 7. **Start dan Enable WireGuard**
Mulai layanan WireGuard dan atur agar berjalan otomatis saat boot:
```bash
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

---

### 8. **Konfigurasi Klien**
Untuk setiap klien, buat kunci privat dan publik:
```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

Tambahkan konfigurasi klien di file server:
```bash
sudo nano /etc/wireguard/wg0.conf
```

Tambahkan blok berikut untuk setiap klien:
```ini
[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.0.0.2/32  # IP klien dalam jaringan WireGuard
```

Restart WireGuard setelah memperbarui konfigurasi:
```bash
sudo systemctl restart wg-quick@wg0
```

---

### 9. **Konfigurasi Klien WireGuard**
Di klien, buat file konfigurasi:
```bash
nano client-wg0.conf
```

Isi file dengan:
```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.0.0.2/24
DNS = 8.8.8.8  # Atur DNS sesuai kebutuhan

[Peer]
PublicKey = <server_public_key>
Endpoint = <server_ip>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

- **Ganti:**
  - `<client_private_key>` dengan kunci privat klien.
  - `<server_public_key>` dengan kunci publik server.
  - `<server_ip>` dengan IP server.

---

### 10. **Tes Koneksi**
Di klien, aktifkan koneksi WireGuard:
```bash
wg-quick up client-wg0
```

Cek status WireGuard di server:
```bash
sudo wg
```

Jika berhasil, koneksi klien akan muncul di status server.

---

WireGuard siap digunakan! 🎉
