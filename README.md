# Nginx-Lua-Anti-DDoS

Script anti-DDoS untuk server web Nginx menggunakan Lua dengan authentication puzzle berbasis Javascript yang terinspirasi dari Cloudflare "I'm Under Attack Mode". Pengunjung yang mencurigakan mendapat halaman interstisial sekitar lima detik sementara trafiknya dianalisis sebelum diteruskan.

[![Cloudflare I am Under Attack Mode!](https://blog.cloudflare.com/content/images/im_under_attack_page.png.scaled500.png)](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/wiki/funding)

[Master Branch untuk Nginx Lua modern](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/tree/master) - [Build Nginx Lua lama, pakai branch ini](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/tree/old-outdated-nginx-lua-builds)

Bekerja untuk trafik HTTP maupun HTTPS/SSL, tanpa batas ukuran serangan, dengan I/O asynchronous nonblocking penuh.

## Fitur keamanan

- Pembatasan request per IP / anti-flooding
- Storage data/kunci anti-DDoS di server remote, Redis, atau memcached
- Under Attack mode menyala otomatis saat DDoS terdeteksi
- Whitelist dan blacklist alamat IP serta rentang subnet
- Whitelist dan blacklist User-Agent
- Halaman autentikasi "I am Under Attack" untuk menyaring bot
- Username/password untuk membatasi akses ke situs atau path tertentu
- Logging user yang lulus atau gagal puzzle (bisa dipakai Fail2Ban untuk ban IP/bot/alat AI)
- Filtering Range header untuk menahan serangan slowhttp/slowloris, berguna untuk situs download dan streaming video

## WAF (Web Application Firewall)

- Blokir dan whitelist IPv4/IPv6 termasuk rentang subnet
- Blokir dan whitelist User-Agent untuk menahan bot jahat, scanner, dan eksploit
- Inspeksi POST data/field dan blokir request berbahaya
- Inspeksi URL terhadap SQL injection dan XSS
- Inspeksi query string, argumen, semua request header, dan cookies

## Caching dan performa

- Caching halaman web, dengan bypass berdasarkan cookies atau URL
- Cache hanya jika cookies/URL tertentu ada, atau hanya jika ukuran halaman di atas/bawah ambang tertentu
- Sorting, whitelist, dan penghapusan query string
- Minifikasi/kompresi JS, CSS, HTML
- Cache storage memakai Redis, memcached, lrucache, atau shared.DICT

## Dukungan layanan tambahan

Tor (.onion), Lokinet (.loki), Ethereum Name Service (.ENS), I2P, Freenet, IPFS/IPNS, SWARM, Radicale, zeronet, dan hidden service atau port lain yang dilindungi Nginx.

## Error page dan output kustom

- Intersepsi error page untuk diganti halaman milikmu sendiri
- Menyembunyikan error PHP/MySQL dari pengunjung
- Memodifikasi isi halaman atau file yang keluar

## Instalasi

Edit pengaturan di dalam `anti_ddos_challenge.lua` sesuai kebutuhanmu ([lokasi settings](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/blob/master/lua/anti_ddos_challenge.lua#L234)).

Salin script ke folder konfigurasi Nginx:

```
nginx/conf/lua/
```

Lalu tambahkan ini ke HTTP block, atau ke server/location block jika hanya ingin aktif di bagian tertentu:

```lua
lua_shared_dict antiddos 70m; # zona memori bersama untuk melacak request per user unik

access_by_lua_file anti_ddos_challenge.lua;
```

### Contoh nginx.conf

Aktif untuk semua situs di server:

```lua
http {
    lua_shared_dict antiddos 70m;
    access_by_lua_file anti_ddos_challenge.lua;
}
```

Aktif hanya untuk satu situs:

```lua
http {
    lua_shared_dict antiddos 70m;
}

server {
    access_by_lua_file anti_ddos_challenge.lua;
}
```

Aktif hanya di satu location:

```lua
http {
    lua_shared_dict antiddos 70m;
}

location / {
    access_by_lua_file anti_ddos_challenge.lua;
}
```

Opsi setup lainnya (Tor .onion, proxy Cloudflare, pengaturan script) ada di [wiki](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/wiki).

## Kebutuhan

Cukup Nginx + Lua, tidak ada dependensi lain.

- [Openresty](https://openresty.org/en/download.html) menyediakan build Nginx + Lua untuk Windows, Linux, dll.
- [Nginx4windows](http://nginx-win.ecsds.eu/) punya build Windows dengan Lua.
- Atau compile Nginx sendiri dari [source](https://nginx.org/en/download.html) dengan modul Lua.

## Protokol dan server yang didukung

Bekerja dengan Nginx, Nginx + Lua, Openresty, build Nginx kustom dengan Lua, dan Litespeed/OpenLiteSpeed (karena mereka memakai build Lua Openresty, lihat [dokumentasinya](https://openlitespeed.org/kb/openlitespeed-lua-module/)).

Trafik yang dilindungi mencakup HTTP(S), HTTP/2, HTTP/3/QUIC, TCP maupun UDP, dan cocok untuk CMS apa pun (WordPress, Drupal, Joomla, Magento, forum vBulletin/phpBB/mybb/xenforo, dan lainnya) serta situs crypto yang butuh verifikasi JavaScript sebelum akses wallet.

## Serangan yang bisa dimitigasi

Semua serangan Layer 7: DoS/DDoS, HTTP flood, brute force, injection, XSS, XXE, broken authentication dan access control, sensitive data exposure, insecure deserialization, komponen dengan kerentanan yang diketahui, dan banyak lagi.

## Kontribusi

Kalau menemukan bug atau masalah, buka [Issue](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/issues). Fork dan perbaikan dipersilakan lewat [Pull Request](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS/pulls) untuk komunitas yang memakai script ini.

Terima kasih ke @Cloudflare untuk inspirasinya dan komunitas @openresty yang luar biasa. Mari bangun internet yang lebih baik, tempat kecepatan, privasi, keamanan, dan kompresi sama-sama diperhatikan.
