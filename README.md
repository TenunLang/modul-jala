# modul-jala

**Jala** — kerangka kerja web **MVC** untuk bahasa [Tenun](https://github.com/TenunLang/Tenun). Alur jelas ala Laravel/CodeIgniter, istilah Indonesia, ramah pemula.

Nama "jala" = jaring penangkap ikan → menangkap & mengarahkan permintaan web. Dibangun di atas modul [`web`](https://github.com/TenunLang/modul-web) (HTTP) dan [`tampilan`](https://github.com/TenunLang/modul-tampilan) (template Batik).

```
Permintaan  →  Rute  →  Kontroler  →  (Model)  →  Tampilan  →  Respons
```

## Mulai cepat

```
tenun baru blog        # scaffold proyek MVC (mirip `laravel new`)
cd blog
tenun add jala         # pasang Jala + web + tampilan
tenun run app.tenun    # buka http://localhost:8080
```

`tenun baru <nama>` membuat struktur siap pakai:

`tenun baru <nama>` membuat struktur siap pakai (folder/berkas bernama teknis/English; kode tetap Indonesia):

```
blog/
  app.tenun                bootstrap (impor jala + routes + controllers, jala_layani)
  routes.tenun             definisi rute  ->  fungsi controller
  app/
    config.tenun           konfigurasi
    controllers/
      home.tenun           logika halaman
    models/
      example.tenun        model (data)
  views/
    layout.batik           tata letak (layout, memuat {{konten}})
    home.batik             view (template Batik)
    about.batik
    partials/
      header.batik         partial ({{> partials/header}})
  public/
    style.css              berkas statis
  tenun.json               (butuh: jala)
```

## Generator (ala `artisan make:`)

Tambah berkas ke proyek yang sudah ada:

```
tenun buat:controller produk   # -> app/controllers/produk.tenun
tenun buat:model produk        # -> app/models/produk.tenun
tenun buat:view produk         # -> views/produk.batik
```

## Inti

```tenun
// app/controllers/home.tenun
fungsi home_index(): teks {
    kembali jala_tampil_tata("layout", "home", peta{
        "judul": "Beranda",
        "pesan": "Halo dari Jala!"
    });
}

// routes.tenun
jala_get("/", home_index);
jala_get("/pos/:id", pos_lihat);

// app.tenun
fungsi tangani(m: teks, j: teks, b: teks): teks { kembali jala_tangani(m, j, b); }
jala_layani(8080);
```

> Folder view default `views/` (berkas `.batik`). Partial via `{{> partials/header}}` (Batik). Ubah folder dengan `jala_atur_tampilan("views")`.

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
- `jala_tampil(nama, data)` — render `views/<nama>.batik`.
- `jala_tampil_tata(tata, nama, data)` — view di dalam layout (layout memuat `{{konten}}`).
- `jala_render(tmpl, data)`, `jala_daftar(itemTmpl, daftar)` — render string / perulangan.
- `jala_atur_tampilan(dir)` — ubah folder view (default `views`).
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

`contoh/` berisi aplikasi todo (struktur `app/controllers`, `app/models`, `views/` + `partials/`, `routes.tenun`): daftar/tambah/hapus tugas dengan rute + controller + model + view + validasi.

## Lisensi

MIT.
