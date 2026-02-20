# 🥗 NutriVision — Platform Gizi & Resep Sehat

Aplikasi web modern berbasis **Next.js 14 App Router** untuk menghitung BMI, kebutuhan kalori, dan menjelajahi resep makanan bergizi.

---

## 📂 Struktur Folder

```
nutrivision/
├── app/
│   ├── globals.css          # Global styles (Tailwind + custom glassmorphism)
│   ├── layout.js            # Root layout: metadata, Provider, Navbar, Footer
│   ├── page.js              # Home page (Static — Server Component)
│   ├── not-found.js         # 404 page
│   ├── bmi/
│   │   └── page.js          # Kalkulator BMI → CSR ('use client')
│   └── recipes/
│       ├── loading.js       # Skeleton loader (Suspense fallback)
│       ├── page.js          # Daftar resep → SSR (fetch 'no-store')
│       └── [id]/
│           └── page.js      # Detail resep → SSG (generateStaticParams)
├── components/
│   ├── Navbar.js            # Navbar glassmorphism + scroll effect
│   └── Footer.js            # Footer dengan teknik rendering info
├── context/
│   └── NutriContext.js      # Context API untuk state global
├── utils/
│   └── bmiCalculator.js     # Fungsi kalkulasi BMI, BMR, kalori
├── next.config.js
├── tailwind.config.js
├── postcss.config.js
└── package.json
```

---

## 🚀 Cara Menjalankan Project

### 1. Prasyarat
- Node.js 18+ terinstall
- npm atau yarn

### 2. Install dependencies
```bash
cd nutrivision
npm install
```

### 3. Jalankan development server
```bash
npm run dev
```
Buka http://localhost:3000 di browser.

### 4. Build untuk production (untuk melihat SSG bekerja)
```bash
npm run build
npm start
```

---

## 🧠 Penjelasan Teknik Rendering

### ✅ Client-Side Rendering (CSR) — `app/bmi/page.js`

**Apa itu CSR?**
Rendering terjadi sepenuhnya di browser pengguna. Server hanya mengirim JavaScript bundle, lalu browser menjalankannya dan mengisi konten.

**Kenapa dipakai di halaman BMI?**
- Halaman bersifat sangat interaktif (input real-time)
- Menggunakan `useState` untuk menyimpan input dan hasil kalkulasi
- Tidak memerlukan data dari server — semua kalkulasi lokal
- Perubahan UI harus terjadi secara instan tanpa reload

**Implementasi:**
```js
'use client' // ← directive ini mengaktifkan CSR

const [weight, setWeight] = useState('')
const [result, setResult] = useState(null)
```

**Tandanya:** Directive `'use client'` di baris pertama file.

---

### ✅ Server-Side Rendering (SSR) — `app/recipes/page.js`

**Apa itu SSR?**
HTML di-generate di server setiap kali ada request masuk. Data selalu fresh karena di-fetch ulang setiap request.

**Kenapa dipakai di halaman daftar resep?**
- Data resep dapat berubah kapan saja
- SEO-friendly: search engine mendapat HTML lengkap berisi data
- Tidak ingin konten stale/kadaluarsa

**Implementasi:**
```js
// Async Server Component = SSR by default
export default async function RecipesPage() {
  const res = await fetch('https://dummyjson.com/recipes', {
    cache: 'no-store' // ← INI kunci SSR: nonaktifkan cache
  })
}
```

**Tandanya:** `cache: 'no-store'` dalam fetch options.

---

### ✅ Static Site Generation (SSG) — `app/recipes/[id]/page.js`

**Apa itu SSG?**
Halaman di-generate SEKALI saat build time. Hasilnya disimpan sebagai file HTML statis dan langsung diserve ke pengguna tanpa perlu processing di server.

**Kenapa dipakai di halaman detail resep?**
- Data resep jarang berubah
- Performa maksimal: TTFB (Time To First Byte) sangat rendah
- Tidak ada beban komputasi di server saat request masuk

**Implementasi:**
```js
// Beri tahu Next.js halaman mana yang perlu di-generate
export async function generateStaticParams() {
  const res = await fetch('https://dummyjson.com/recipes?limit=20')
  const data = await res.json()
  return data.recipes.map(r => ({ id: String(r.id) }))
}

// Fetch tanpa 'no-store' = di-cache = SSG
const res = await fetch(`https://dummyjson.com/recipes/${id}`)
```

**Tandanya:** Fungsi `generateStaticParams` yang di-export.

---

## 📦 State Management

### 1. Local State (`useState`) — Wajib
Digunakan di halaman BMI untuk:
- Menyimpan input weight dan height
- Menyimpan hasil kalkulasi
- Mengelola error dan loading state

### 2. Context API — Nilai Tambah
`NutriContext` menyimpan data hasil kalkulasi BMI secara global:
- Data tersedia di semua komponen (Navbar, Footer, dll)
- Tidak perlu prop drilling
- Data persist selama sesi browser

```js
const { saveCalculation, nutriData } = useNutri()
// Simpan hasil kalkulasi ke context
saveCalculation({ bmi, dailyCalories, ... })
// Baca di komponen lain
console.log(nutriData.bmi) // tersedia di Navbar!
```

---

## ⚡ Fitur Tambahan (Nilai Plus)

| Fitur | Implementasi | Benefit |
|-------|-------------|---------|
| **Loading Skeleton** | `app/recipes/loading.js` + inline skeleton | UX tidak blank saat loading |
| **Error Handling** | try/catch di semua fetch + error UI | Tidak crash saat API down |
| **Lazy Loading Images** | `next/image` dengan `loading="lazy"` | Performa halaman lebih cepat |
| **SEO Metadata** | `export const metadata` per halaman | Ranking search engine lebih baik |
| **Responsive Design** | Tailwind responsive prefixes (sm:, lg:) | Tampil baik di semua device |
| **Glassmorphism UI** | backdrop-blur + rgba background | Visual modern dan menarik |
| **Smooth Animations** | CSS keyframes + transition | Transisi halaman terasa mulus |
| **Mobile Navbar** | Hamburger menu dengan toggle | Navigasi mudah di mobile |
| **BMI Scale Bar** | Progress bar dengan pointer | Visualisasi BMI yang intuitif |
| **Macro Breakdown** | Kalkulasi protein/karbo/lemak | Info gizi lebih lengkap |

---

## 🌐 API yang Digunakan

**dummyjson.com/recipes**
- `GET /recipes?limit=20` — Daftar resep (SSR)
- `GET /recipes/{id}` — Detail resep (SSG)

---

## 🎨 Design System

- **Framework:** Tailwind CSS
- **Tema:** Dark glassmorphism
- **Font:** Playfair Display (display) + DM Sans (body)
- **Warna:** Emerald (#10b981) sebagai primary
- **Efek:** backdrop-blur + rgba transparency
- **Animasi:** CSS keyframes (float, fadeIn, slideUp, shimmer)
