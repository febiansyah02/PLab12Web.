# Laporan Praktikum 12: Pemrograman Web 2 - VueJS Komponen & Routing (Single Page Application)

Laporan ini dibuat untuk memenuhi tugas mata kuliah **Pemrograman Web 2** pada program studi Teknik Informatika, Universitas Pelita Bangsa. Praktikum ini berfokus pada transformasi arsitektur antarmuka menjadi **Single Page Application (SPA)** menggunakan **Vue Components** dan **Vue Router** berbasis CDN.

---

## Identitas Mahasiswa
**Nama:** M. Febiansyah Mulyadi

**Kampus:** Universitas Pelita Bangsa

**Mata Kuliah:** Pemrograman Web 2

**Modul Praktikum:** Praktikum 12 (Komponen & Routing SPA)

---

## Struktur Direktori Proyek Baru
Untuk menjaga modularitas kode UI, struktur file pada direktori web server (`C:\xampp\htdocs\lab8_vuejs`) dipecah ke dalam beberapa berkas komponen terisolasi di dalam folder `assets/js/components/`:

```text
lab8_vuejs/
├── assets/
│   ├── css/
│   │   └── style.css
│   └── js/
│       ├── app.js
│       └── components/
│           ├── About.js
│           ├── Artikel.js
│           └── Home.js
└── index.html
```
💻 Implementasi Berkas Komponen Frontend1. Komponen Halaman Utama (assets/js/components/Home.js)Menampilkan halaman beranda selamat datang menggunakan properti objek template internal VueJS .  JavaScriptconst Home = {
    template: `
        <div class="home-container">
            <h2>Selamat Datang di Portal Admin Artikel</h2>
            <p>Gunakan menu navigasi di atas untuk mengelola data artikel secara real-time 
            memanfaatkan RESTful API CodeIgniter 4 dan VueJS.</p>
        </div>
    `
};
2. Komponen Profil Pengembang (assets/js/components/About.js)Tugas Tambahan Modul: Menampilkan data diri mahasiswa beserta avatar/inisial nama pembuat aplikasi sebagai rute baru .  JavaScriptconst About = {
    template: `
        <div class="home-container" style="text-align: center; padding: 30px;">
            <h2>Profil Pengembang</h2>
            <div style="margin: 20px auto; width: 120px; height: 120px; background: #3152d6; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 40px; font-weight: bold;">
                MF
            </div>
            <h3>M. Febiansyah Mulyadi</h3>
            <p><strong>NIM:</strong> 312310XXX</p>
            <p><strong>Kelas:</strong> TI.23.X.X</p>
            <p style="color: #666; font-style: italic;">Teknik Informatika - Universitas Pelita Bangsa</p>
        </div>
    `
};
3. Komponen Manajemen Artikel (assets/js/components/Artikel.js)Menampung seluruh logika operasional CRUD data artikel (GET, POST, PUT, DELETE) yang sebelumnya berada pada app.js lama.  JavaScriptconst Artikel = {
    template: `
        <div>
            <h2>Manajemen Data Artikel</h2>
            <button id="btn-tambah" @click="tambah">Tambah Data</button>
            
            <div class="modal" v-if="showForm">
                <div class="modal-content">
                    <span class="close" @click="showForm = false">&times;</span>
                    <form id="form-data" @submit.prevent="saveData">
                        <h3>{{ formTitle }}</h3>
                        <div>
                            <input type="text" v-model="formData.judul" placeholder="Judul Artikel" required>
                        </div>
                        <div>
                            <textarea v-model="formData.isi" rows="6" placeholder="Isi Artikel" required></textarea>
                        </div>
                        <div>
                            <select v-model="formData.status">
                                <option v-for="option in statusOptions" :key="option.value" :value="option.value">
                                    {{ option.text }}
                                </option>
                            </select>
                        </div>
                        <input type="hidden" v-model="formData.id">
                        <button type="submit" id="btnSimpan">Simpan</button>
                        <button type="button" @click="showForm = false">Batal</button>
                    </form>
                </div>
            </div>

            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Judul</th>
                        <th>Status</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(row, index) in artikel" :key="row.id">
                        <td class="center-text">{{ row.id }}</td>
                        <td>{{ row.judul }}</td>
                        <td>{{ statusText(row.status) }}</td>
                        <td class="center-text">
                            <a href="#" @click.prevent="edit(row)" style="margin-right: 10px;">Edit</a>
                            <a href="#" @click.prevent="hapus(index, row.id)">Hapus</a>
                        </td>
                    </tr>
                    <tr v-if="artikel.length === 0">
                        <td colspan="4" class="center-text">Tidak ada data artikel.</td>
                    </tr>
                </tbody>
            </table>
        </div>
    `,
    data() {
        return {
            artikel: [],
            formData: { id: null, presidential: '', isi: '', status: 0 },
            showForm: false,
            formTitle: 'Tambah Data',
            statusOptions: [
                { text: 'Draft', value: 0 },
                { text: 'Publish', value: 1 }
            ]
        }
    },
    mounted() {
        this.loadData();
    },
    methods: {
        loadData() {
            axios.get(apiUrl + '/post')
                .then(response => { this.artikel = response.data.artikel; })
                .catch(error => console.error("Gagal memuat data:", error));
        },
        tambah() {
            this.showForm = true;
            this.formTitle = 'Tambah Data';
            this.formData = { id: null, judul: '', isi: '', status: 0 };
        },
        edit(data) {
            this.showForm = true;
            this.formTitle = 'Ubah Data';
            this.formData = { id: data.id, judul: data.judul, isi: data.isi, status: parseInt(data.status) };
        },
        saveData() {
            const payload = { judul: this.formData.judul, isi: this.formData.isi };
            if (this.formData.id) {
                axios.put(apiUrl + '/post/' + this.formData.id, payload)
                    .then(response => { alert('Data berhasil diperbarui!'); this.loadData(); this.showForm = false; })
                    .catch(error => console.error(error));
            } else {
                axios.post(apiUrl + '/post', payload)
                    .then(response => { alert('Data berhasil ditambahkan!'); this.loadData(); this.showForm = false; })
                    .catch(error => console.error(error));
            }
            this.formData = { id: null, judul: '', isi: '', status: 0 };
        },
        hapus(index, id) {
            if (confirm('Yakin menghapus data?')) {
                axios.delete(apiUrl + '/post/' + id)
                    .then(response => { alert('Data berhasil dihapus!'); this.artikel.splice(index, 1); })
                    .catch(error => console.error(error));
            }
        },
        statusText(status) {
            return status == 1 ? 'Publish' : 'Draft';
        }
    }
};
🔀 Konfigurasi Router & Master Layout4. File Core App (assets/js/app.js)Menginisialisasi objek mapping rute URL, mendaftarkan komponen-komponen halaman ke dalam instance VueRouter, serta melakukan mounting aplikasi utama .  JavaScriptconst { createApp } = Vue;
const { createRouter, createWebHashHistory } = VueRouter;

const apiUrl = 'http://localhost:8080';

// 1. Definisikan mapping rute URL ke Komponen
const routes = [
    { path: '/', component: Home },
    { path: '/artikel', component: Artikel },
    { path: '/about', component: About }
];

// 2. Buat instance router
const router = createRouter({
    history: createWebHashHistory(),
    routes
});

// 3. Inisialisasi Aplikasi Vue dan gunakan Router
const app = createApp({});
app.use(router);
app.mount('#app');
5. Berkas Tata Letak Induk (index.html)Bertindak sebagai Master Layout dengan menyertakan pustaka vue-router.global.js CDN, menyediakan menu link dinamis <router-link>, dan area penampung komponen visual <router-view> .  HTML<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SPA Frontend VueJS & Vue Router</title>
    <script src="[https://unpkg.com/vue@3/dist/vue.global.js](https://unpkg.com/vue@3/dist/vue.global.js)"></script>
    <script src="[https://unpkg.com/vue-router@4/dist/vue-router.global.js](https://unpkg.com/vue-router@4/dist/vue-router.global.js)"></script>
    <script src="[https://unpkg.com/axios/dist/axios.min.js](https://unpkg.com/axios/dist/axios.min.js)"></script>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
    <div id="app">
        <header>
            <h1>Aplikasi Panel Single Page (SPA)</h1>
            <nav class="nav-menu">
                <router-link to="/">Beranda</router-link> | 
                <router-link to="/artikel">Kelola Artikel</router-link> |
                <router-link to="/about">About Pengembang</router-link>
            </nav>
        </header>

        <main style="margin-top: 20px;">
            <router-view></router-view>
        </main>
    </div>

    <script src="assets/js/components/Home.js"></script>
    <script src="assets/js/components/Artikel.js"></script>
    <script src="assets/js/components/About.js"></script>
    <script src="assets/js/app.js"></script>
</body>
</html>
6. Desain Visual Menu Aktif (assets/css/style.css)Penambahan CSS spesifik kelas bawaan .router-link-exact-active untuk mendeteksi penanda visual halaman menu yang sedang aktif .  CSS.nav-menu {
    padding: 10px;
    background: #eff1ff;
    border-radius: 5px;
    margin-bottom: 15px;
}
.nav-menu a {
    text-decoration: none;
    color: #3152d6;
    font-weight: bold;
    padding: 5px 10px;
}
.router-link-exact-active {
    background-color: #3152d6;
    color: #ffffff !important;
    border-radius: 3px;
}
.home-container {
    padding: 20px;
    border: 1px solid #eff1ff;
    background: #fafafa;
}
📸 Dokumentasi Hasil Pengujian SPA (Single Page Application)Berikut adalah bukti dokumentasi pengujian navigasi antar-komponen di browser tanpa mengalami pemuatan ulang (hard-reload) halaman:  A. Tampilan Komponen Beranda (Home)Deskripsi: Tampilan visual awal saat rute default root (#/) diakses client pertama kali.Bukti Dokumentasi: ![Menu Beranda](GANTI_DENGAN_SCREENSHOT_MENU_HOME.png)B. Tampilan Komponen Kelola Artikel (Artikel CRUD)Deskripsi: Tampilan dinamis list artikel saat rute #/artikel ditekan, seluruh data berhasil di-render ulang dari backend tanpa reload.Bukti Dokumentasi: ![Kelola Artikel](GANTI_DENGAN_SCREENSHOT_MENU_ARTIKEL.png)C. Tampilan Komponen About Pengembang (About)Deskripsi: Hasil pengerjaan tugas mandiri penambahan rute profil mahasiswa /about .  Bukti Dokumentasi: ![About Pengembang](GANTI_DENGAN_SCREENSHOT_MENU_ABOUT.png)💡 KesimpulanMelalui Praktikum 12 ini, konsep Single Page Application (SPA) berhasil diterapkan sepenuhnya. Penggunaan library Vue Router memindahkan kendali manipulasi rute URL dari sisi server (Server-Side Routing) menjadi murni ditangani di browser pengguna (Client-Side Routing). Hasilnya, pertukaran layout antar-halaman visual dapat dieksekusi secara instan, menghemat bandwidth data, serta menciptakan user experience yang responsif dan cepat
