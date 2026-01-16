# 📘 Penjelasan Teknis Smartphone Query Engine

## Dokumentasi Lengkap untuk Tugas Automata dan Teknik Kompilasi

| Keterangan | Detail |
|------------|--------|
| **Nama** | Muhammad Rizal Nurfirdaus |
| **NIM** | 20230810088 |
| **Kelas** | TINFC-2023-04 |
| **Mata Kuliah** | Automata dan Teknik Kompilasi |
| **Dosen Pengampu** | Sherly Gina Supratman, M.Kom. |
| **Tema** | Text Mining - Mini Query Engine |

---

# BAGIAN 1: DESKRIPSI TEMA DAN KASUS

## 1.1 Tema: Text Mining dengan Mini Query Engine

Proyek ini mengimplementasikan **bahasa query text-mining** untuk melakukan operasi:
- **Filter** - Menyaring data berdasarkan kondisi tertentu
- **Transformasi** - Mengubah format teks (uppercase, lowercase, dll)
- **Analisis Statistik** - Menghitung statistik data (count, average, min, max)

### Kasus yang Dikerjakan

Membangun sistem **Smartphone Query Engine** yang memungkinkan pengguna untuk:
1. Mencari smartphone berdasarkan kriteria tertentu (merek, harga, RAM, dll)
2. Mengubah format tampilan data (huruf besar/kecil)
3. Mendapatkan statistik dari data smartphone

### Domain Specific Language (DSL)

Bahasa query yang dibangun mirip dengan SQL namun lebih sederhana:

```
FILTER <kondisi>           -- Untuk menyaring data
TRANSFORM <fungsi>         -- Untuk transformasi teks
STATS <fungsi_statistik>   -- Untuk analisis statistik
```

---

# BAGIAN 2: DAFTAR TOKEN DAN REGULAR EXPRESSION

## 2.1 Daftar Token

| No | Nama Token | Deskripsi | Pattern |
|----|------------|-----------|---------|
| 1 | KEYWORD | Kata kunci query | FILTER, TRANSFORM, STATS, SELECT, WHERE, AND, OR, NOT |
| 2 | OPERATOR | Operator perbandingan | =, <, >, <=, >=, !=, CONTAINS, LIKE |
| 3 | FUNCTION | Nama fungsi | uppercase, lowercase, capitalize, trim, length, count, sum, average, min, max, distinct |
| 4 | STRING | Nilai string | Teks dalam kutip tunggal atau ganda |
| 5 | NUMBER | Nilai numerik | Integer atau floating point |
| 6 | IDENTIFIER | Nama kolom/variabel | Nama yang dimulai dengan huruf |
| 7 | LPAREN | Kurung buka | ( |
| 8 | RPAREN | Kurung tutup | ) |
| 9 | COMMA | Pemisah | , |
| 10 | EOF | End of file | - |

## 2.2 Regular Expression untuk Setiap Token

### 2.2.1 KEYWORD
```regex
(FILTER|TRANSFORM|STATS|SELECT|WHERE|AND|OR|NOT)
```

### 2.2.2 OPERATOR
```regex
(=|<|>|<=|>=|!=|CONTAINS|LIKE)
```

Atau lebih detail:
```regex
(<=|>=|!=|=|<|>|CONTAINS|LIKE)
```
*Catatan: Operator 2 karakter (<=, >=, !=) harus dikenali terlebih dahulu sebelum operator 1 karakter*

### 2.2.3 FUNCTION
```regex
(uppercase|lowercase|capitalize|trim|length|count|sum|average|min|max|distinct)
```

### 2.2.4 STRING
```regex
'[^']*'|"[^"]*"
```
- `'[^']*'` : String dengan kutip tunggal
- `"[^"]*"` : String dengan kutip ganda

### 2.2.5 NUMBER
```regex
[0-9]+(\.[0-9]+)?
```
- `[0-9]+` : Satu atau lebih digit (integer)
- `(\.[0-9]+)?` : Opsional bagian desimal

### 2.2.6 IDENTIFIER
```regex
[a-zA-Z_][a-zA-Z0-9_]*
```
- `[a-zA-Z_]` : Dimulai dengan huruf atau underscore
- `[a-zA-Z0-9_]*` : Diikuti oleh huruf, angka, atau underscore

### 2.2.7 WHITESPACE (diabaikan)
```regex
[ \t\n\r]+
```

## 2.3 Sketsa NFA untuk Token STRING

```
NFA untuk STRING dengan kutip tunggal: '[^']*'

        ε           [^']          ε
   ──► (q0) ──'──► (q1) ──────► (q1) ──'──► ((q2))
                     │             ▲
                     └─────────────┘
                        [^'] (loop)

State:
- q0: State awal
- q1: Setelah membaca kutip pembuka, membaca karakter
- q2: State akhir (accepting) setelah membaca kutip penutup
```

## 2.4 Sketsa DFA untuk NUMBER

```
DFA untuk NUMBER: [0-9]+(\.[0-9]+)?

                [0-9]           .            [0-9]
           ┌──────────┐    ┌────────┐    ┌──────────┐
           │          ▼    │        ▼    │          ▼
   ──► (q0) ──[0-9]──► ((q1)) ──.──► (q2) ──[0-9]──► ((q3))
                         ▲                            │
                         │                            │
                         └────────────────────────────┘
                                   [0-9]

State:
- q0: State awal
- q1: Accepting state - integer valid
- q2: Setelah membaca titik desimal
- q3: Accepting state - float valid
```

## 2.5 DFA untuk OPERATOR

```
DFA untuk OPERATOR: =, <, >, <=, >=, !=

                    =
   ──► (q0) ────────────► ((q_eq))     [untuk =]
        │
        ├────<────► ((q_lt)) ────=────► ((q_le))  [untuk < dan <=]
        │
        ├────>────► ((q_gt)) ────=────► ((q_ge))  [untuk > dan >=]
        │
        └────!────► (q_not) ────=────► ((q_ne))  [untuk !=]

State:
- q0     : State awal
- q_eq   : Accepting - operator =
- q_lt   : Accepting - operator <
- q_le   : Accepting - operator <=
- q_gt   : Accepting - operator >
- q_ge   : Accepting - operator >=
- q_not  : Setelah membaca !
- q_ne   : Accepting - operator !=
```

---

# BAGIAN 3: CONTEXT-FREE GRAMMAR (CFG)

## 3.1 CFG untuk Query Language

```
<query>         ::= <filter_query> 
                  | <transform_query> 
                  | <stats_query>
                  | <select_query>

<filter_query>  ::= 'FILTER' <condition_list>

<condition_list>::= <condition> 
                  | <condition> <logical_op> <condition_list>

<condition>     ::= <identifier> <operator> <value>

<logical_op>    ::= 'AND' | 'OR'

<operator>      ::= '=' | '!=' | '<' | '>' | '<=' | '>=' | 'CONTAINS' | 'LIKE'

<value>         ::= <string> | <number> | <identifier>

<transform_query> ::= 'TRANSFORM' <function_list>

<stats_query>   ::= 'STATS' <function_list>

<function_list> ::= <function_call> 
                  | <function_call> ',' <function_list>

<function_call> ::= <function_name> '(' <identifier> ')'

<function_name> ::= 'uppercase' | 'lowercase' | 'capitalize' | 'trim' 
                  | 'length' | 'count' | 'sum' | 'average' | 'min' 
                  | 'max' | 'distinct'

<select_query>  ::= 'SELECT' <column_list> 
                  | 'SELECT' <column_list> 'WHERE' <condition>

<column_list>   ::= <identifier> 
                  | <identifier> ',' <column_list>

<identifier>    ::= IDENTIFIER_TOKEN

<string>        ::= STRING_TOKEN

<number>        ::= NUMBER_TOKEN
```

## 3.2 Contoh Derivasi

### Contoh 1: `FILTER merek = 'Samsung'`

```
<query>
  └── <filter_query>
        ├── 'FILTER'
        └── <condition_list>
              └── <condition>
                    ├── <identifier> → 'merek'
                    ├── <operator> → '='
                    └── <value>
                          └── <string> → 'Samsung'
```

### Contoh 2: `FILTER merek = 'Xiaomi' AND ram >= 8`

```
<query>
  └── <filter_query>
        ├── 'FILTER'
        └── <condition_list>
              ├── <condition>
              │     ├── <identifier> → 'merek'
              │     ├── <operator> → '='
              │     └── <value> → 'Xiaomi'
              ├── <logical_op> → 'AND'
              └── <condition_list>
                    └── <condition>
                          ├── <identifier> → 'ram'
                          ├── <operator> → '>='
                          └── <value> → 8
```

### Contoh 3: `STATS count(nama), average(harga)`

```
<query>
  └── <stats_query>
        ├── 'STATS'
        └── <function_list>
              ├── <function_call>
              │     ├── <function_name> → 'count'
              │     ├── '('
              │     ├── <identifier> → 'nama'
              │     └── ')'
              ├── ','
              └── <function_list>
                    └── <function_call>
                          ├── <function_name> → 'average'
                          ├── '('
                          ├── <identifier> → 'harga'
                          └── ')'
```

---

# BAGIAN 4: DESAIN PARSER DAN SKETSA AST

## 4.1 Metode Parsing: Recursive Descent Parser

Parser yang diimplementasikan menggunakan teknik **Recursive Descent** yang merupakan jenis **Top-Down Parser**. Setiap non-terminal dalam CFG diimplementasikan sebagai sebuah fungsi.

### 4.1.1 Struktur Parser

```python
class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0
        self.current_token = tokens[0]
        self.automata = QueryAutomata()  # Untuk validasi
    
    def parse(self):           # Entry point
        if keyword == 'FILTER':
            return self.parse_filter()
        elif keyword == 'TRANSFORM':
            return self.parse_transform()
        elif keyword == 'STATS':
            return self.parse_stats()
    
    def parse_filter(self):    # Non-terminal <filter_query>
        ...
    
    def parse_condition(self):  # Non-terminal <condition>
        ...
    
    def parse_transform(self):  # Non-terminal <transform_query>
        ...
    
    def parse_stats(self):      # Non-terminal <stats_query>
        ...
    
    def parse_function_call(self):  # Non-terminal <function_call>
        ...
```

### 4.1.2 Pseudocode Parser

```
FUNCTION parse():
    IF current_token.type != KEYWORD:
        ERROR "Query harus dimulai dengan keyword"
    
    keyword = current_token.value
    
    SWITCH keyword:
        CASE 'FILTER':
            RETURN parse_filter()
        CASE 'TRANSFORM':
            RETURN parse_transform()
        CASE 'STATS':
            RETURN parse_stats()
        DEFAULT:
            ERROR "Keyword tidak dikenali"

FUNCTION parse_filter():
    advance()  // Skip 'FILTER'
    root = ASTNode('filter')
    
    // Parse kondisi pertama
    condition = parse_condition()
    root.children.append(condition)
    
    // Parse kondisi tambahan (AND/OR)
    WHILE current_token.value IN ('AND', 'OR'):
        logical_op = current_token.value
        advance()
        condition = parse_condition()
        condition.value.operator = logical_op
        root.children.append(condition)
    
    RETURN root

FUNCTION parse_condition():
    // Expect identifier
    column = current_token.value
    advance()
    
    // Expect operator
    operator = current_token.value
    advance()
    
    // Expect value
    value = current_token.value
    advance()
    
    RETURN ASTNode('condition', {column, operator, value})
```

## 4.2 Sketsa Abstract Syntax Tree (AST)

### 4.2.1 Struktur Node AST

```python
@dataclass
class ASTNode:
    type: str           # Tipe node: 'filter', 'transform', 'stats', dll
    value: Any          # Nilai node
    children: List      # Child nodes
```

### 4.2.2 AST untuk Query FILTER

Query: `FILTER merek = 'Samsung'`

```
                    ┌──────────────┐
                    │  ASTNode     │
                    │  type: filter│
                    │  value: None │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  ASTNode     │
                    │  type:       │
                    │  condition   │
                    │  value: {    │
                    │   column:    │
                    │    'merek'   │
                    │   operator:  │
                    │    '='       │
                    │   value:     │
                    │   'Samsung'  │
                    │  }           │
                    └──────────────┘
```

### 4.2.3 AST untuk Query dengan AND

Query: `FILTER merek = 'Xiaomi' AND ram >= 8`

```
                         ┌──────────────┐
                         │  ASTNode     │
                         │  type: filter│
                         └──────┬───────┘
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
       ┌──────────────┐                   ┌──────────────┐
       │  ASTNode     │                   │  ASTNode     │
       │  type:       │                   │  type:       │
       │  condition   │                   │  condition   │
       │  value: {    │                   │  value: {    │
       │   column:    │                   │   operator:  │
       │    'merek'   │                   │    'AND'     │
       │   operator:  │                   │   condition: │
       │    '='       │                   │    {column:  │
       │   value:     │                   │     'ram'    │
       │   'Xiaomi'   │                   │    operator: │
       │  }           │                   │     '>='     │
       └──────────────┘                   │    value: 8} │
                                          └──────────────┘
```

### 4.2.4 AST untuk Query STATS

Query: `STATS count(nama), average(harga)`

```
                         ┌──────────────┐
                         │  ASTNode     │
                         │  type: stats │
                         └──────┬───────┘
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
       ┌──────────────┐                   ┌──────────────┐
       │  ASTNode     │                   │  ASTNode     │
       │  type:       │                   │  type:       │
       │  function    │                   │  function    │
       │  value: {    │                   │  value: {    │
       │   name:      │                   │   name:      │
       │    'count'   │                   │   'average'  │
       │   column:    │                   │   column:    │
       │    'nama'    │                   │    'harga'   │
       │  }           │                   │  }           │
       └──────────────┘                   └──────────────┘
```

### 4.2.5 AST untuk Query TRANSFORM

Query: `TRANSFORM uppercase(nama)`

```
                    ┌──────────────┐
                    │  ASTNode     │
                    │  type:       │
                    │  transform   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  ASTNode     │
                    │  type:       │
                    │  function    │
                    │  value: {    │
                    │   name:      │
                    │  'uppercase' │
                    │   column:    │
                    │    'nama'    │
                    │  }           │
                    └──────────────┘
```

---

# BAGIAN 5: DESAIN IR/DSL DAN ALUR EKSEKUSI

## 5.1 Intermediate Representation (IR)

Dalam implementasi ini, **AST** berfungsi sebagai IR. AST merepresentasikan struktur query dalam bentuk tree yang dapat di-traverse untuk eksekusi.

### 5.1.1 Representasi IR untuk Setiap Tipe Query

**FILTER Query IR:**
```python
{
    'type': 'filter',
    'conditions': [
        {'column': 'merek', 'operator': '=', 'value': 'Samsung'},
        {'logical_op': 'AND', 'column': 'ram', 'operator': '>=', 'value': 8}
    ]
}
```

**TRANSFORM Query IR:**
```python
{
    'type': 'transform',
    'functions': [
        {'name': 'uppercase', 'column': 'nama'}
    ]
}
```

**STATS Query IR:**
```python
{
    'type': 'stats',
    'functions': [
        {'name': 'count', 'column': 'nama'},
        {'name': 'average', 'column': 'harga'}
    ]
}
```

## 5.2 Alur Eksekusi

### 5.2.1 Diagram Alur Eksekusi Lengkap

```
┌─────────────────────────────────────────────────────────────────────┐
│                      INPUT: Query String                            │
│                 "FILTER merek = 'Samsung'"                          │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     STEP 1: LEXICAL ANALYSIS                        │
│                           (Lexer)                                   │
├─────────────────────────────────────────────────────────────────────┤
│  Input:  "FILTER merek = 'Samsung'"                                 │
│  Process: Scan karakter, identifikasi pattern, buat token           │
│  Output: [Token(KEYWORD,'FILTER'), Token(IDENTIFIER,'merek'),       │
│           Token(OPERATOR,'='), Token(STRING,'Samsung'),             │
│           Token(EOF,None)]                                          │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     STEP 2: SYNTAX ANALYSIS                         │
│                      (Parser + Automata)                            │
├─────────────────────────────────────────────────────────────────────┤
│  Input:  List of Tokens                                             │
│  Process:                                                           │
│    1. Automata: START → FILTER_STATE → CONDITION → ACCEPT           │
│    2. Parser: Recursive descent, build AST                          │
│  Output: AST dengan root node type='filter'                         │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     STEP 3: SEMANTIC ANALYSIS                       │
│                        (Interpreter)                                │
├─────────────────────────────────────────────────────────────────────┤
│  Input:  AST + DataFrame                                            │
│  Process:                                                           │
│    1. Cek apakah kolom 'merek' ada dalam DataFrame                  │
│    2. Validasi operator '=' dapat digunakan                         │
│    3. Validasi value 'Samsung' sesuai tipe kolom                    │
│  Output: Validated AST                                              │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     STEP 4: CODE EXECUTION                          │
│                        (Interpreter)                                │
├─────────────────────────────────────────────────────────────────────┤
│  Input:  Validated AST + DataFrame                                  │
│  Process:                                                           │
│    1. Traverse AST                                                  │
│    2. Untuk node 'filter': buat mask df['merek'] == 'Samsung'       │
│    3. Apply mask ke DataFrame                                       │
│  Output: Filtered DataFrame                                         │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      OUTPUT: Result                                 │
│              DataFrame berisi HP Samsung saja                       │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2.2 Alur untuk Setiap Tipe Query

**Alur FILTER:**
```
AST(filter) → evaluate_condition() → create_mask() → apply_mask() → DataFrame
```

**Alur TRANSFORM:**
```
AST(transform) → get_function() → apply_to_column() → create_new_column() → DataFrame
```

**Alur STATS:**
```
AST(stats) → get_stat_function() → calculate_stat() → Dictionary{stat: value}
```

---

# BAGIAN 6: PENJELASAN SIMULASI AUTOMATA (DFA/PDA)

## 6.1 Finite State Automata (FSA) untuk Validasi Query

### 6.1.1 Definisi Formal DFA

**DFA M = (Q, Σ, δ, q₀, F)** dimana:

- **Q** (States) = {START, FILTER_STATE, TRANSFORM_STATE, STATS_STATE, SELECT_STATE, CONDITION, FUNCTION_CALL, ACCEPT, ERROR}

- **Σ** (Alphabet) = {FILTER, TRANSFORM, STATS, SELECT, IDENTIFIER, FUNCTION, AND, OR, COMMA, EOF}

- **q₀** (Initial State) = START

- **F** (Final States) = {ACCEPT}

- **δ** (Transition Function):

### 6.1.2 Tabel Transisi

| Current State | Input | Next State |
|---------------|-------|------------|
| START | FILTER | FILTER_STATE |
| START | TRANSFORM | TRANSFORM_STATE |
| START | STATS | STATS_STATE |
| START | SELECT | SELECT_STATE |
| FILTER_STATE | IDENTIFIER | CONDITION |
| CONDITION | AND | FILTER_STATE |
| CONDITION | OR | FILTER_STATE |
| CONDITION | EOF | ACCEPT |
| TRANSFORM_STATE | FUNCTION | FUNCTION_CALL |
| STATS_STATE | FUNCTION | FUNCTION_CALL |
| FUNCTION_CALL | EOF | ACCEPT |
| FUNCTION_CALL | COMMA | STATS_STATE |
| * | * | ERROR |

### 6.1.3 Diagram State DFA

```
                                  IDENTIFIER
                         ┌──────────────────────┐
                         │                      │
                         ▼                      │
                   ┌───────────┐                │
         FILTER    │  FILTER   │    AND/OR      │
    ┌─────────────►│  _STATE   ├───────────────►│
    │              └─────┬─────┘                │
    │                    │                      │
    │                    │ IDENTIFIER           │
    │                    ▼                      │
    │              ┌───────────┐                │
    │              │ CONDITION ├────────────────┘
    │              └─────┬─────┘
    │                    │ EOF
┌───┴───┐                ▼
│ START │          ┌───────────┐
└───┬───┘          │  ACCEPT   │
    │              └───────────┘
    │                    ▲
    │                    │ EOF
    │              ┌─────┴─────┐
    │   TRANSFORM  │ FUNCTION  │    COMMA
    ├─────────────►│  _CALL    ├───────────┐
    │              └─────┬─────┘           │
    │                    ▲                 │
    │                    │ FUNCTION        │
    │              ┌─────┴─────┐           │
    │              │TRANSFORM  │           │
    │              │  _STATE   │           │
    │              └───────────┘           │
    │                                      │
    │                    ┌─────────────────┘
    │                    ▼
    │   STATS      ┌───────────┐
    └─────────────►│  STATS    │
                   │  _STATE   │
                   └─────┬─────┘
                         │ FUNCTION
                         ▼
                   ┌───────────┐
                   │ FUNCTION  │
                   │  _CALL    │
                   └───────────┘
```

## 6.2 Implementasi Automata dalam Kode

```python
class QueryAutomata:
    # Tabel transisi
    TRANSITIONS = {
        (AutomataState.START, 'FILTER'): AutomataState.FILTER_STATE,
        (AutomataState.START, 'TRANSFORM'): AutomataState.TRANSFORM_STATE,
        (AutomataState.START, 'STATS'): AutomataState.STATS_STATE,
        (AutomataState.FILTER_STATE, 'IDENTIFIER'): AutomataState.CONDITION,
        (AutomataState.CONDITION, 'AND'): AutomataState.FILTER_STATE,
        (AutomataState.CONDITION, 'OR'): AutomataState.FILTER_STATE,
        (AutomataState.CONDITION, 'EOF'): AutomataState.ACCEPT,
        (AutomataState.TRANSFORM_STATE, 'FUNCTION'): AutomataState.FUNCTION_CALL,
        (AutomataState.STATS_STATE, 'FUNCTION'): AutomataState.FUNCTION_CALL,
        (AutomataState.FUNCTION_CALL, 'EOF'): AutomataState.ACCEPT,
        (AutomataState.FUNCTION_CALL, 'COMMA'): AutomataState.STATS_STATE,
    }
    
    def transition(self, input_symbol):
        key = (self.current_state, input_symbol)
        if key in self.TRANSITIONS:
            self.current_state = self.TRANSITIONS[key]
            return True
        self.current_state = AutomataState.ERROR
        return False
```

## 6.3 Contoh Input-Output Simulasi Automata

### 6.3.1 Contoh 1: Query FILTER Valid

**Input:** `FILTER merek = 'Samsung'`

**Tokens:** [FILTER, IDENTIFIER, OPERATOR, STRING, EOF]

**Simulasi Transisi:**
```
State: START
  Input: FILTER
  Transition: START --FILTER--> FILTER_STATE ✓

State: FILTER_STATE
  Input: IDENTIFIER (merek)
  Transition: FILTER_STATE --IDENTIFIER--> CONDITION ✓

State: CONDITION
  (operator dan value diproses oleh parser, bukan automata)
  Input: EOF
  Transition: CONDITION --EOF--> ACCEPT ✓

Final State: ACCEPT ✅
Query Valid!
```

### 6.3.2 Contoh 2: Query FILTER dengan AND

**Input:** `FILTER merek = 'Xiaomi' AND ram >= 8`

**Simulasi Transisi:**
```
START --FILTER--> FILTER_STATE ✓
FILTER_STATE --IDENTIFIER--> CONDITION ✓
CONDITION --AND--> FILTER_STATE ✓
FILTER_STATE --IDENTIFIER--> CONDITION ✓
CONDITION --EOF--> ACCEPT ✓

Riwayat: START → FILTER_STATE → CONDITION → FILTER_STATE → CONDITION → ACCEPT
Query Valid! ✅
```

### 6.3.3 Contoh 3: Query STATS

**Input:** `STATS count(nama), average(harga)`

**Simulasi Transisi:**
```
START --STATS--> STATS_STATE ✓
STATS_STATE --FUNCTION--> FUNCTION_CALL ✓
FUNCTION_CALL --COMMA--> STATS_STATE ✓
STATS_STATE --FUNCTION--> FUNCTION_CALL ✓
FUNCTION_CALL --EOF--> ACCEPT ✓

Riwayat: START → STATS_STATE → FUNCTION_CALL → STATS_STATE → FUNCTION_CALL → ACCEPT
Query Valid! ✅
```

### 6.3.4 Contoh 4: Query Invalid

**Input:** `merek = 'Samsung'` (tanpa FILTER)

**Simulasi Transisi:**
```
State: START
  Input: IDENTIFIER (merek)
  Transition: tidak ada transisi dari START dengan input IDENTIFIER
  
Final State: ERROR ❌
Query Invalid! Expected keyword (FILTER/TRANSFORM/STATS)
```

---

# BAGIAN 7: CONTOH INPUT-OUTPUT LENGKAP

## 7.1 Test Case 1: Filter Berdasarkan Merek

**Input:**
```sql
FILTER merek = 'Samsung'
```

**Proses:**
1. **Lexer Output:**
   ```
   [Token(KEYWORD, 'FILTER'), Token(IDENTIFIER, 'merek'), 
    Token(OPERATOR, '='), Token(STRING, 'Samsung'), Token(EOF)]
   ```

2. **Automata Trace:**
   ```
   START → FILTER_STATE → CONDITION → ACCEPT ✓
   ```

3. **AST:**
   ```
   ASTNode(type='filter', children=[
       ASTNode(type='condition', value={
           'column': 'merek', 
           'operator': '=', 
           'value': 'Samsung'
       })
   ])
   ```

**Output:**
| nama | merek | harga | ram | storage |
|------|-------|-------|-----|---------|
| Samsung Galaxy S24 Ultra | Samsung | 21999000 | 12 | 512 |
| Samsung Galaxy A54 | Samsung | 5499000 | 8 | 256 |
| Samsung Galaxy M34 | Samsung | 3999000 | 6 | 128 |

---

## 7.2 Test Case 2: Filter Berdasarkan Harga

**Input:**
```sql
FILTER harga < 5000000
```

**Output:**
| nama | merek | harga | ram | storage |
|------|-------|-------|-----|---------|
| Samsung Galaxy M34 | Samsung | 3999000 | 6 | 128 |
| Xiaomi Redmi Note 12 | Xiaomi | 2799000 | 6 | 128 |
| Infinix Note 30 Pro | Infinix | 2999000 | 8 | 256 |
| Tecno Spark 10 Pro | Tecno | 1999000 | 8 | 128 |
| Realme C55 | Realme | 2499000 | 6 | 128 |

---

## 7.3 Test Case 3: Transformasi Teks

**Input:**
```sql
TRANSFORM uppercase(nama)
```

**Output (5 data pertama):**
| nama | uppercase_nama | merek |
|------|----------------|-------|
| Samsung Galaxy S24 Ultra | SAMSUNG GALAXY S24 ULTRA | Samsung |
| iPhone 15 Pro Max | IPHONE 15 PRO MAX | Apple |
| Xiaomi 14 Ultra | XIAOMI 14 ULTRA | Xiaomi |
| ASUS ROG Phone 8 Pro | ASUS ROG PHONE 8 PRO | ASUS |
| Google Pixel 8 Pro | GOOGLE PIXEL 8 PRO | Google |

---

## 7.4 Test Case 4: Statistik Harga

**Input:**
```sql
STATS count(nama), average(harga), min(harga), max(harga)
```

**Output:**
```
📊 HASIL STATISTIK:
   • count(nama): 20
   • average(harga): Rp 8.749.450
   • min(harga): Rp 1.999.000
   • max(harga): Rp 21.999.000
```

---

## 7.5 Test Case 5: Filter dengan Kondisi AND

**Input:**
```sql
FILTER merek = 'Xiaomi' AND ram >= 8
```

**Proses Automata:**
```
START → FILTER_STATE → CONDITION → FILTER_STATE → CONDITION → ACCEPT ✓
```

**Output:**
| nama | merek | harga | ram | storage |
|------|-------|-------|-----|---------|
| Xiaomi 14 Ultra | Xiaomi | 15999000 | 16 | 512 |
| Xiaomi 13T Pro | Xiaomi | 7999000 | 12 | 256 |

---

# BAGIAN 8: KESIMPULAN

## 8.1 Ringkasan Implementasi

| Komponen | Deskripsi | Status |
|----------|-----------|--------|
| **Lexer** | Tokenisasi query string menjadi token | ✅ Implementasi lengkap |
| **Parser** | Recursive descent parser dengan validasi automata | ✅ Implementasi lengkap |
| **Automata** | DFA untuk validasi struktur query | ✅ Implementasi lengkap |
| **AST** | Representasi tree dari query | ✅ Implementasi lengkap |
| **Interpreter** | Eksekusi query pada DataFrame | ✅ Implementasi lengkap |
| **Query Engine** | Integrasi semua komponen | ✅ Implementasi lengkap |

## 8.2 Fitur yang Didukung

- ✅ FILTER dengan kondisi tunggal
- ✅ FILTER dengan kondisi majemuk (AND/OR)
- ✅ TRANSFORM untuk manipulasi teks
- ✅ STATS untuk analisis statistik
- ✅ Validasi sintaks dengan automata
- ✅ Error handling yang informatif
- ✅ Mode interaktif untuk pengujian

## 8.3 Pembelajaran

Melalui proyek ini, mahasiswa mempelajari:

1. **Teori Automata**: Implementasi DFA untuk validasi sintaks
2. **Teknik Kompilasi**: Lexer, Parser, AST, dan Interpreter
3. **Regular Expression**: Pattern matching untuk tokenisasi
4. **Context-Free Grammar**: Definisi bahasa formal
5. **Recursive Descent Parsing**: Teknik parsing top-down
6. **Domain Specific Language**: Desain bahasa untuk domain tertentu

---

**Dokumen ini dibuat sebagai penjelasan teknis lengkap untuk tugas mata kuliah Automata dan Teknik Kompilasi.**

**© 2026 Muhammad Rizal Nurfirdaus - TINFC-2023-04**
