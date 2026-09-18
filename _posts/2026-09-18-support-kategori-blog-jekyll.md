---
layout: post
title: "Membuat Kategori Artikel di Blog Jekyll"
date: 2026-09-18 08:00:00 +0700
tags: [support, blog, jekyll]
---

Blog ini sekarang mendukung **kategori** — jadi artikel bisa dikelompokkan per topik, dan pembaca bisa menjelajah hanya pada topik yang mereka minati.

## Bagaimana Kategori Bekerja di Jekyll

Jekyll tidak punya fitur "kategori" bawaan yang otomatis membuat halaman. Kategori di sini dibangun dari tiga bagian:

| Bagian | File | Peran |
|--------|------|-------|
| Data | `_posts/*.md` | Tag di frontmatter yang menandai kategori |
| Tampilan | `_layouts/category.html` | Layout daftar artikel per kategori |
| Halaman | `blog/categories/<slug>/index.html` | Halaman kategori (isinya 6 baris) |
| Navigasi | `_includes/category-nav.html` | Daftar tautan kategori |

## Menandai Artikel dengan Kategori

Cukup tambahkan tag di frontmatter artikel:

```yaml
---
layout: post
title: "Judul Artikel"
date: 2026-09-18 08:00:00 +0700
tags: [support, blog, jekyll]
---
```

Artikel ini sendiri memakai tag `support`, sehingga muncul di halaman [Kategori Support](/blog/categories/support/).

## Cara Kerja Filter Artikel

Layout kategori memfilter artikel berdasarkan tag:

```liquid
{% raw %}{% assign cat_posts = site.posts | where: "tags", page.category_tag %}{% endraw %}
```

Filter `where` milik Jekyll mencocokkan **setiap elemen** pada properti bertipe array, sehingga `tags: [support, blog]` akan cocok dengan `category_tag: support`.

## Menambah Kategori Baru

Dua langkah saja:

**1. Tambahkan tautan** di `_includes/category-nav.html`:

```html
<li><a href="/blog/categories/kuliner/">Kuliner</a></li>
```

**2. Buat halaman** `blog/categories/kuliner/index.html`:

```yaml
---
layout: category
title: Kuliner
permalink: /blog/categories/kuliner/
category_tag: kuliner
icon: fa-utensils
blurb: "Catatan seputar masakan dan resep."
---
```

Selesai. Halaman kategori otomatis menampilkan semua artikel dengan tag `kuliner`.

## Catatan Penting

- **Daftar kategori ditulis manual**, bukan iterasi seluruh `site.tags`. Iterasi tersebut akan membuat tautan ke *semua* tag (termasuk `dart`, `tutorial`, `investasi`) padahal halaman kategorinya tidak ada — hasilnya banyak tautan rusak.
- Satu artikel bisa punya **beberapa tag**, jadi bisa tampil di lebih dari satu kategori.
- Artikel baru yang diberi tag kategori akan **otomatis muncul** di halaman kategori terkait setelah di-push.

## Kategori yang Tersedia

- [Support](/blog/categories/support/)
- [Flutter](/blog/categories/flutter/)
- [Semua Artikel](/blog/archive/)
