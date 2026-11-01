
# todo-frontend

Repository frontend Svelte + TypeScript dengan Vite, dijalankan memakai Bun.
Backend terpisah berada di repository `todo-api`.

## Jalankan

Pasang Bun 1.2+ dari https://bun.sh. Jalankan backend terlebih dahulu pada
http://localhost:3000. Di terminal lain, dari folder repository frontend:

```sh
bun install
bun run dev
```

Buka http://localhost:5173. Tidak perlu meletakkan kedua repository dalam
folder induk yang sama. Masing-masing memiliki package.json dan dependensi sendiri.

## Integrasi REST API

`src/App.svelte` memanggil `fetch('/api/todos')`.
Proxy dalam `vite.config.ts` meneruskan `/api` ke `http://127.0.0.1:3000`.
Frontend tidak mengimpor kode atau file backend. Integrasinya hanya melalui HTTP.
Jika alamat backend berubah, ubah target proxy di konfigurasi server dan preview.
Dengan proxy ini, browser mengakses origin frontend sehingga CORS tidak diperlukan
untuk demo lokal. Hosting frontend statis membutuhkan reverse proxy /api tersendiri,
atau penyesuaian URL API dan konfigurasi CORS backend.

## Periksa dan build

```sh
bun run check
bun run build
bun --bun vite preview
```

Backend harus tetap aktif saat mencoba hasil build melalui preview.
