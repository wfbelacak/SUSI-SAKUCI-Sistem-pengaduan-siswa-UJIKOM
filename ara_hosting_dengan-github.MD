

<!-- Cara Hosting Web Laravel dan React melalui GitHub -->
A. Persiapan & Deployment React

1. Build Project: Jalankan npm run build atau yarn build di folder React untuk menghasilkan folder build atau dist.
2. Repo GitHub: Buat repostori baru di GitHub.
3. Deploy ke GitHub Pages:
    - Install paket gh-pages: npm install gh-pages --save-dev.
    - Tambahkan "Homepage": https://username.github.io/nama-repo di package.json
    - Tambahkan script deploy di package.json: "deploy": gh-pages -d build".
    - jalankan ke branch gh-pages.
    - Aktifkan Github Pages di setting repostori.

B. Persiapan & Deployment Laravel

1. Konfigurasi API: pastikan .env.production di laravel telah dikonfigurasi, terutama VITE_URL_BARE_URL mengarah ke URL backend yang baru.
2. Push ke Github: Upload kode Laravel ke repostori GitHub terpisah.
3. Host API: Gunakan layanan seperti Render atau Rainway (layanan cloud yang memiliki opsi trial) dan hubungkan dengan repostori GitHub untuk auto-deploy.
