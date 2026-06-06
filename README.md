# modul-jala

**Jala** — kerangka kerja web **MVC** untuk bahasa [Tenun](https://github.com/TenunLang/Tenun). Alur jelas ala Laravel/CodeIgniter, istilah Indonesia, ramah pemula.

Nama "jala" = jaring penangkap ikan → menangkap & mengarahkan permintaan web. Dibangun di atas modul [`web`](https://github.com/TenunLang/modul-web) (HTTP) dan [`tampilan`](https://github.com/TenunLang/modul-tampilan) (template Batik).

```
Permintaan  →  Rute  →  Kontroler  →  (Model)  →  Tampilan  →  Respons
```

## Mulai cepat

```
tenun add jala                                  # ambil Jala (berisi CLI)
tenun tenun_modul/jala/cli/jala.tenun baru Blog # scaffold proyek MVC
cd Blog
tenun add jala                                  # pasang dependensi proyek
tenun                                           # jalankan index.tenun -> http://localhost:8080
```

> `tenun <file.tenun>` menjalankan berkas langsung (tanpa `run`); `tenun` tanpa argumen menjalankan `index.tenun`. CLI scaffold adalah program Tenun di dalam modul (`cli/jala.tenun`), bukan bagian compiler inti.

Perintah `baru` membuat struktur siap pakai (folder berhuruf besar ala Laravel; kode tetap Indonesia):

```
Blog/
  index.tenun              titik masuk (`tenun` menjalankan ini)
  nakhoda.tenun            CLI proyek (generator) — "artisan"-nya Jala
  Routes.tenun             definisi rute  ->  fungsi controller
  App/
    Config.tenun           konfigurasi
    Controllers/
      Home.tenun           logika halaman
    Models/
      Example.tenun        model (data)
  Views/
    Layout.batik           tata letak (memuat {{konten}})
    Home.batik  About.batik
    Partials/
      Header.batik         partial ({{> Partials/Header}})
  Public/
    style.css              berkas statis
  tenun.json               (butuh: jala)
```

## Generator (ala `artisan make:`)

Lewat `nakhoda.tenun` di proyek:

```
tenun nakhoda.tenun controller Produk   # -> App/Controllers/Produk.tenun
tenun nakhoda.tenun model Produk        # -> App/Models/Produk.tenun
tenun nakhoda.tenun view Produk         # -> Views/Produk.batik
```

## Inti

```tenun
// App/Controllers/Home.tenun
fungsi home_index(): teks {
    kembali jala_tampil_tata("Layout", "Home", peta{
        "judul": "Beranda",
        "pesan": "Halo dari Jala!"
    });
}

// Routes.tenun
jala_get("/", home_index);
jala_get("/pos/:id", pos_lihat);

// index.tenun
fungsi tangani(m: teks, j: teks, b: teks): teks { kembali jala_tangani(m, j, b); }
jala_layani(8080);
```

> Folder view default `Views/` (berkas `.batik`). Partial via `{{> Partials/Header}}` (Batik). Ubah folder dengan `jala_atur_tampilan("Views")`.

## Acuan API

### Rute (`src/rute.tenun`)
- `jala_get / jala_post / jala_put / jala_hapus / jala_patch(pola, handler)`.
- `jala_grup(prefix, daftar)` — grup rute berprefiks (`daftar` = fungsi pendaftar).
- `jala_param(nama)` — parameter rute (`/pos/:id` → `jala_param("id")`).
- `jala_namai(nama, pola)`, `jala_url(nama)` — rute bernama.

### Permintaan (`src/permintaan.tenun`)
- `jala_input(nama)` — form lalu query. `jala_kueri`, `jala_form`, `jala_json_teks`, `jala_json_angka`, `jala_header`, `jala_metode`, `jala_jalur`, `jala_badan`.
- Validasi: `jala_perlu(nama)`, `jala_minimal(nama, n)`, `jala_ada_galat()`, `jala_galat_pesan(nama)`.

### Respons (`src/respons.tenun`)
- `jala_teks`, `jala_html`, `jala_json`, `jala_status(kode)`, `jala_redirect(url)`, `jala_redirect_rute(nama)`.
- `jala_json_data(peta)` — respons JSON dari peta. `jala_json_peta(peta)` — encode peta → JSON.

### Tampilan (`src/tampilan.tenun`, via Batik)
- `jala_tampil(nama, data)` — render `Views/<nama>.batik`.
- `jala_tampil_tata(tata, nama, data)` — view di dalam layout (layout memuat `{{konten}}`).
- `jala_render(tmpl, data)`, `jala_daftar(itemTmpl, daftar)` — render string / perulangan.
- `jala_atur_tampilan(dir)` — ubah folder view (default `Views`).
- Sintaks Batik: `{{kunci}}`, `{{#jika k}}...{{/jika k}}`, `{{#kecuali k}}...{{/kecuali k}}`, `{{> partials/nama}}` (sertakan partial), `{{! komentar }}`.

### Aplikasi (`src/aplikasi.tenun`)
- `jala_tangani(metode, jalur, badan)` — dispatcher (panggil dari `tangani`).
- `jala_layani(port)`, `jala_statik(dir)`, `jala_cors(origin)`, `jala_pakai(mw)`.

### Konfigurasi (`src/konfig.tenun`)
- `jala_atur(kunci, nilai)`, `jala_konfig(kunci)`.

### Keamanan (`src/keamanan.tenun`)
Autentikasi JWT (modul `auth`) + proteksi CSRF.

```tenun
// Penjaga rute (di awal controller):
fungsi dashboard(): teks {
    kalau jala_wajib_masuk("kunci-rahasia") == salah { kembali ""; }   // 401 bila tak login
    kembali jala_tampil_tata("layout", "dashboard", peta{ "user": jala_pengguna() });
}

// CSRF: taruh field di form, cek saat POST.
//   <form ...>{{csrf}}...</form>  (csrf = jala_csrf_field(id))
kalau jala_csrf_cek(id) == salah { jala_status(403); }
```

- `jala_terautentikasi(rahasia): bool`, `jala_wajib_masuk(rahasia): bool` (set 401), `jala_pengguna(): teks` (sub token).
- `jala_csrf_field(id): teks` (input tersembunyi), `jala_csrf_baru(id)`, `jala_csrf_cek(id): bool`.

## Model

Model bebas: pakai modul [`orm`](https://github.com/TenunLang/modul-orm) (MySQL/Postgres) atau penyimpanan kunci-nilai bawaan. Lihat `contoh/` (todo dengan model kv) dan controller CRUD.

## Contoh lengkap

`contoh/` berisi aplikasi todo (struktur `App/Controllers`, `App/Models`, `Views/` + `Partials/`, `Routes.tenun`, `index.tenun`, `nakhoda.tenun`): daftar/tambah/hapus tugas dengan rute + controller + model + view + validasi.

## Lisensi

MIT.
