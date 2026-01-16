# 🔍 Smartphone Query Engine - Text Mining dengan Automata dan Parser

## 📋 Informasi Mahasiswa

| Keterangan | Detail |
|------------|--------|
| **Nama** | Muhammad Rizal Nurfirdaus |
| **NIM** | 20230810088 |
| **Kelas** | TINFC-2023-04 |
| **Mata Kuliah** | Automata dan Teknik Kompilasi |
| **Dosen Pengampu** | Sherly Gina Supratman, M.Kom. |

---

## 📖 Deskripsi Proyek

**Smartphone Query Engine** adalah sebuah mini-query engine yang dibangun untuk memproses bahasa query text-mining. Proyek ini mengimplementasikan konsep-konsep fundamental dalam **Automata dan Teknik Kompilasi**, termasuk:

- **Lexical Analysis (Lexer/Tokenizer)**
- **Syntax Analysis (Parser dengan Recursive Descent)**
- **Finite State Automata (FSM/DFA)**
- **Abstract Syntax Tree (AST)**
- **Interpreter untuk Eksekusi Query**

Sistem ini memungkinkan pengguna untuk melakukan operasi **filter**, **transformasi teks**, dan **analisis statistik** pada dataset smartphone menggunakan bahasa query yang sederhana dan intuitif.

---

## 🎯 Tujuan Proyek

1. **Memahami Lexical Analysis**: Mempelajari cara memecah string input menjadi token-token yang bermakna
2. **Memahami Syntax Analysis**: Mempelajari cara menganalisis struktur gramatikal dari sekumpulan token
3. **Mengimplementasikan Automata**: Menggunakan Finite State Machine untuk validasi sintaks query
4. **Membangun Interpreter**: Mengeksekusi query dan menghasilkan output yang sesuai
5. **Integrasi Komponen**: Menggabungkan semua komponen menjadi sistem query engine yang fungsional

---

## 🏗️ Arsitektur Sistem

### Alur Pemrosesan Query

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Query String  │ ──► │     LEXER       │ ──► │     PARSER      │ ──► │   INTERPRETER   │
│                 │     │   (Tokenizer)   │     │  (+ Automata)   │     │   (Executor)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │                       │
        │                       ▼                       ▼                       ▼
        │               ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
        │               │    Tokens     │       │      AST      │       │    Result     │
        │               │ [TOK1, TOK2]  │       │   (Tree)      │       │  (DataFrame)  │
        │               └───────────────┘       └───────────────┘       └───────────────┘
        │
        ▼
"FILTER merek = 'Samsung'"
```

### Diagram Komponen

```
                    ┌────────────────────────────────────────┐
                    │        SmartphoneQueryEngine           │
                    │         (Main Controller)              │
                    └───────────────┬────────────────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
            ▼                       ▼                       ▼
    ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
    │    Lexer      │       │    Parser     │       │  Interpreter  │
    │  (Tokenizer)  │       │ (+ Automata)  │       │  (Executor)   │
    └───────────────┘       └───────────────┘       └───────────────┘
            │                       │                       │
            ▼                       ▼                       ▼
    ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
    │    Token      │       │   ASTNode     │       │   DataFrame   │
    │  (DataClass)  │       │  (DataClass)  │       │   (Pandas)    │
    └───────────────┘       └───────────────┘       └───────────────┘
```

---

## 📦 Komponen Utama

### 1. Lexer (Tokenizer)

**Lexer** bertanggung jawab untuk memecah string query menjadi token-token individual. Setiap token memiliki tipe dan nilai.

#### Tipe Token yang Didukung:

| Tipe Token | Deskripsi | Contoh |
|------------|-----------|--------|
| `KEYWORD` | Kata kunci utama | FILTER, TRANSFORM, STATS, AND, OR |
| `OPERATOR` | Operator perbandingan | =, <, >, <=, >=, !=, CONTAINS, LIKE |
| `FUNCTION` | Nama fungsi | uppercase, lowercase, count, average |
| `STRING` | Nilai string | 'Samsung', "Xiaomi" |
| `NUMBER` | Nilai numerik | 5000000, 8.5 |
| `IDENTIFIER` | Nama kolom | nama, merek, harga |
| `LPAREN` | Kurung buka | ( |
| `RPAREN` | Kurung tutup | ) |
| `COMMA` | Koma | , |
| `EOF` | End of file | - |

#### Contoh Tokenisasi:

```
Input:  "FILTER merek = 'Samsung'"

Output: [
    Token(KEYWORD, 'FILTER', pos=0),
    Token(IDENTIFIER, 'merek', pos=7),
    Token(OPERATOR, '=', pos=13),
    Token(STRING, 'Samsung', pos=15),
    Token(EOF, None, pos=24)
]
```

### 2. Parser dengan Automata

**Parser** menganalisis struktur gramatikal dari token-token dan membangun **Abstract Syntax Tree (AST)**. Parser ini menggunakan metode **Recursive Descent** dengan bantuan **Finite State Automata** untuk validasi.

#### State Automata:

```
┌─────────┐
│  START  │
└────┬────┘
     │
     ├──── FILTER ────► FILTER_STATE ────► CONDITION ────┐
     │                                                    │
     ├──── TRANSFORM ─► TRANSFORM_STATE ─► FUNCTION_CALL─┤
     │                                                    │
     └──── STATS ─────► STATS_STATE ─────► FUNCTION_CALL─┤
                                                          │
                                                          ▼
                                                    ┌──────────┐
                                                    │  ACCEPT  │
                                                    └──────────┘
```

#### Struktur AST:

```
Query: "FILTER merek = 'Samsung' AND harga < 10000000"

AST:
        ┌─────────────┐
        │   filter    │
        └──────┬──────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───┴───┐           ┌─────┴─────┐
│condition│         │ condition │
│merek='Samsung'│   │AND harga<10000000│
└───────┘           └───────────┘
```

### 3. Interpreter

**Interpreter** mengeksekusi AST pada DataFrame dan menghasilkan output sesuai dengan tipe query.

#### Operasi yang Didukung:

| Operasi | Fungsi | Contoh |
|---------|--------|--------|
| **FILTER** | Menyaring data berdasarkan kondisi | `FILTER harga < 5000000` |
| **TRANSFORM** | Mengubah format teks | `TRANSFORM uppercase(nama)` |
| **STATS** | Menghitung statistik | `STATS average(harga)` |
| **SELECT** | Memilih kolom tertentu | `SELECT nama, harga WHERE merek = 'Samsung'` |

---

## 📝 Bahasa Query (Domain Specific Language)

### Sintaks FILTER

```
FILTER <kolom> <operator> <nilai> [AND|OR <kolom> <operator> <nilai>]
```

**Operator yang didukung:**
- `=` : Sama dengan
- `!=` : Tidak sama dengan
- `<` : Kurang dari
- `>` : Lebih dari
- `<=` : Kurang dari atau sama dengan
- `>=` : Lebih dari atau sama dengan
- `CONTAINS` : Mengandung substring
- `LIKE` : Pattern matching

**Contoh:**
```sql
FILTER merek = 'Samsung'
FILTER harga < 5000000
FILTER merek = 'Xiaomi' AND ram >= 8
FILTER harga < 5000000 OR ram >= 12
FILTER deskripsi CONTAINS 'gaming'
```

### Sintaks TRANSFORM

```
TRANSFORM <fungsi>(<kolom>) [, <fungsi>(<kolom>)]
```

**Fungsi yang didukung:**
- `uppercase` : Mengubah ke HURUF BESAR
- `lowercase` : Mengubah ke huruf kecil
- `capitalize` : Mengubah ke Huruf Kapital Tiap Kata
- `trim` : Menghapus spasi di awal dan akhir
- `length` : Menghitung panjang karakter

**Contoh:**
```sql
TRANSFORM uppercase(nama)
TRANSFORM lowercase(merek)
TRANSFORM length(deskripsi)
```

### Sintaks STATS

```
STATS <fungsi>(<kolom>) [, <fungsi>(<kolom>)]
```

**Fungsi yang didukung:**
- `count` : Menghitung jumlah data
- `sum` : Menjumlahkan nilai
- `average` : Menghitung rata-rata
- `min` : Mencari nilai minimum
- `max` : Mencari nilai maksimum
- `distinct` : Menghitung jumlah nilai unik

**Contoh:**
```sql
STATS count(nama)
STATS average(harga)
STATS min(harga), max(harga)
STATS count(nama), average(harga), min(harga), max(harga)
```

---

## 📊 Dataset

Dataset yang digunakan adalah **data_smartphone.csv** yang berisi informasi 20 smartphone dengan kolom-kolom berikut:

| Kolom | Tipe | Deskripsi |
|-------|------|-----------|
| `id` | Integer | ID unik smartphone |
| `nama` | String | Nama lengkap smartphone |
| `merek` | String | Merek/brand smartphone |
| `harga` | Integer | Harga dalam Rupiah |
| `ram` | Integer | Kapasitas RAM (GB) |
| `storage` | Integer | Kapasitas penyimpanan (GB) |
| `baterai` | Integer | Kapasitas baterai (mAh) |
| `kamera` | Integer | Resolusi kamera utama (MP) |
| `layar` | Float | Ukuran layar (inci) |
| `sistem_operasi` | String | Sistem operasi |
| `deskripsi` | String | Deskripsi singkat |

### Merek yang Tersedia:
Samsung, Apple, Xiaomi, ASUS, Google, OnePlus, Vivo, Oppo, Realme, Infinix, Tecno, Honor, Motorola, Nothing, Sony, Nubia

---

## 🧪 Contoh Kasus Uji

### Test Case 1: Filter Berdasarkan Merek
```sql
FILTER merek = 'Samsung'
```
**Hasil:** Menampilkan semua HP dengan merek Samsung

### Test Case 2: Filter Berdasarkan Harga
```sql
FILTER harga < 5000000
```
**Hasil:** Menampilkan HP dengan harga dibawah 5 juta Rupiah

### Test Case 3: Transformasi Teks
```sql
TRANSFORM uppercase(nama)
```
**Hasil:** Mengubah nama HP menjadi HURUF BESAR

### Test Case 4: Statistik Harga
```sql
STATS count(nama), average(harga), min(harga), max(harga)
```
**Hasil:** Menampilkan jumlah HP, rata-rata harga, harga termurah, dan harga termahal

### Test Case 5: Filter dengan Kondisi Ganda
```sql
FILTER merek = 'Xiaomi' AND ram >= 8
```
**Hasil:** Menampilkan HP Xiaomi dengan RAM 8GB atau lebih

---

## 🚀 Cara Menjalankan

### Prasyarat
- Python 3.x
- Jupyter Notebook
- Library: pandas

### Langkah-langkah

1. **Clone atau download repository ini**

2. **Buka terminal dan navigasi ke folder proyek**
   ```bash
   cd "/home/rizal/MyProject/Smartphone Query Engine"
   ```

3. **Buat virtual environment (opsional tapi disarankan)**
   ```bash
   python -m venv venv
   source venv/bin/activate
   ```

4. **Install dependencies**
   ```bash
   pip install pandas jupyter
   ```

5. **Jalankan Jupyter Notebook**
   ```bash
   jupyter notebook text_mining_query_engine.ipynb
   ```

6. **Jalankan semua cell secara berurutan** dengan menekan `Shift+Enter` pada setiap cell, atau gunakan menu `Run > Run All Cells`

---

## 📁 Struktur File

```
Smartphone Query Engine/
│
├── text_mining_query_engine.ipynb  # Notebook utama dengan kode lengkap
├── data_smartphone.csv              # Dataset smartphone
├── README.md                        # Dokumentasi proyek (file ini)
├── Penjelasan.md                    # Penjelasan teknis detail
└── venv/                            # Virtual environment (opsional)
```

---

## 📚 Referensi

1. **Automata Theory**: Hopcroft, Motwani, Ullman - "Introduction to Automata Theory, Languages, and Computation"
2. **Compiler Design**: Aho, Lam, Sethi, Ullman - "Compilers: Principles, Techniques, and Tools" (Dragon Book)
3. **Python Documentation**: https://docs.python.org/3/
4. **Pandas Documentation**: https://pandas.pydata.org/docs/

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan akademik mata kuliah **Automata dan Teknik Kompilasi** di program studi Teknik Informatika.

---

**© 2026 Muhammad Rizal Nurfirdaus - TINFC-2023-04**
