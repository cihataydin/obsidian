Obsidian çalıştırma komutu,

```sh
./Obsidian-1.13.7.appimage --no-sandbox
```

Stoat Tunnel çalıştır,

```sh
cloudflared tunnel run stoat-tunnel
```

Stoat App çalıştır ve durdur,

```sh
cd Development/stoat

docker compose up -d

docker compose down
```

Rastgele şifre üretmek,

```sh
# 22 karakter, A-Z, a-z, 0-9, `+`, `/`, '='
openssl rand -base64 16

# 44 karakter, A-Z, a-z, 0-9, `+`, `/`, '='
openssl rand -base64 32

# 88 karakter, A-Z, a-z, 0-9, `+`, `/`, '='
openssl rand -base64 64

# 32 karakter, 0-9, a-f
openssl rand -hex 16
```