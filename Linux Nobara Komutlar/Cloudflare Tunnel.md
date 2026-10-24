Cloudflare güncel rpm paketini indir.

```sh
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm
```

DNF ile yerel kurulum yap.

```sh
sudo dnf install ./cloudflared-linux-x86_64.rpm -y
```

Kurulum başarılı mı test et.

```sh
cloudflared -v
```

Cloudflare hesabına giriş yap.

```sh
cloudflared tunnel login
```

Tunnel oluştur.

```sh
cloudflared tunnel create sample-tunnel
```

DNS kaydını yap.

```sh
cloudflared tunnel route dns sample-tunnel sample-domain.com
```

Yapılandırma dosyasını hazırla.

```sh
nano ~/.cloudflared/config.yml
```

Yayınlayacağınız uygulama port 80 de var olsun.

```yml
tunnel: BURAYA_TUNNEL_UUID
credentials-file: /home/cihat/.cloudflared/BURAYA_TUNNEL_UUID.json

ingress:
  - hostname: sample-domain.com
    service: http://localhost:80
  - service: http_status:404
```

Tunnel çalıştır

```sh
cloudflared tunnel run sample-tunnel
```