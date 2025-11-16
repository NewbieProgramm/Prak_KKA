# Laporan Praktikum Github

## BAGIAN 1: PENGENALAN LOGICAL AGENTS

### Cell 1: Import Libraries
```python
%pip install ipythonblocks
%pip install qpsolvers
from utils import *
from logic import *
from notebook import psource
```
**Penjelasan:** Menginstall dan mengimport library untuk implementasi logical agents. Library `logic` berisi implementasi knowledge base dan algoritma inferensi yang akan digunakan untuk membuat agen berbasis pengetahuan.

**Konsep Teori:** Knowledge-based agents adalah pendekatan AI yang menggunakan representasi eksplisit dari pengetahuan tentang dunia untuk membuat keputusan. Agen ini dapat menerima tugas baru, belajar pengetahuan baru, dan beradaptasi dengan perubahan lingkungan.


## BAGIAN 2: REPRESENTASI LOGIKA PROPOSISIONAL

### Cell 2: Membuat Symbol
```python
Symbol('x')
```
**Penjelasan:** Membuat simbol logika tunggal 'x'. Simbol adalah elemen dasar dalam kalimat logika, bisa merepresentasikan proposisi yang bernilai True atau False.

**Konsep Teori:** Dalam logika proposisional, simbol proposisi adalah pernyataan yang dapat bernilai benar atau salah. Contoh: P = "ada lubang", Q = "ada angin".


### Cell 3: Membuat Multiple Symbols
```python
(x, y, P, Q, f) = symbols('x, y, P, Q, f')
```
**Penjelasan:** Mendefinisikan beberapa simbol sekaligus untuk efisiensi. Simbol-simbol ini nantinya akan digunakan untuk membangun kalimat logika yang lebih kompleks.

**Konsep Teori:** Syntax dalam logika proposisional mendefinisikan kalimat yang diperbolehkan. Simbol proposisi adalah komponen dasar yang dapat dikombinasikan menggunakan operator logika.


### Cell 4: Operasi Logika AND dan NOT
```python
P & ~Q
```
**Penjelasan:** Demonstrasi operator logika: `&` (AND) dan `~` (NOT). Ekspresi ini dibaca "P dan bukan Q", akan true hanya jika P true dan Q false.

**Konsep Teori:** Logical connectives (AND, OR, NOT, IMPLIES, IFF) digunakan untuk membangun kalimat kompleks dari kalimat sederhana. AND (konjungsi) menghasilkan true hanya jika kedua operand true.


### Cell 5-6: Struktur Internal Expr
```python
sentence = P & ~Q
sentence.op      # '&'
sentence.args    # (P, ~Q)
```
**Penjelasan:** Setiap ekspresi logika disimpan sebagai pohon sintaks abstrak dengan `op` (operator) dan `args` (argumen). Struktur ini memudahkan manipulasi dan evaluasi ekspresi.

**Konsep Teori:** Representasi internal menggunakan struktur tree memungkinkan algoritma untuk menavigasi dan memanipulasi kalimat logika secara sistematis.


### Cell 7-10: Eksplorasi Komponen Expr
```python
P.op          # 'P' (simbol atomic)
P.args        # () (tidak ada argumen)
Pxy = P(x, y) # fungsi/predikat
Pxy.op        # 'P'
Pxy.args      # (x, y)
```
**Penjelasan:** Simbol atomic tidak punya argumen, sedangkan fungsi/predikat memiliki argumen. Ini penting untuk first-order logic dimana kita bisa punya relasi dengan parameter.

**Konsep Teori:** Dalam FOL, predikat seperti P(x,y) dapat merepresentasikan relasi antar objek, berbeda dengan proposisional logic yang hanya punya simbol atomic tanpa parameter.


### Cell 11: Nested Expression
```python
3 * f(x, y) + P(y) / 2 + 1
```
**Penjelasan:** Class Expr sangat fleksibel, bisa merepresentasikan ekspresi matematika kompleks dengan berbagai level nesting, tidak terbatas pada logika proposisional.


## BAGIAN 3: OPERATOR IMPLIKASI DAN EKUIVALENSI

### Cell 12: Implikasi dengan Syntax Khusus
```python
~(P & Q) |'==>'| (~P | ~Q)
```
**Penjelasan:** Syntax `|'==>'|` digunakan untuk implikasi (→). Kalimat ini adalah De Morgan's Law: negasi dari konjungsi sama dengan disjungsi dari negasi.

**Konsep Teori:** Implikasi (P ⇒ Q) bernilai false hanya jika P true dan Q false. Ini adalah operator fundamental dalam reasoning logika.


### Cell 13: Fungsi expr() untuk Parsing String
```python
expr('~(P & Q) ==> (~P | ~Q)')
```
**Penjelasan:** Fungsi `expr()` mem-parse string menjadi objek Expr. Lebih intuitif dan mudah dibaca dibanding syntax Python, terutama untuk kalimat kompleks.

**Konsep Teori:** Parser otomatis mengkonversi notasi infix standar logika menjadi representasi internal yang dapat diproses oleh algoritma inferensi.


### Cell 14: Parse Ekspresi Kompleks
```python
expr('sqrt(b ** 2 - 4 * a * c)')
```
**Penjelasan:** Parser cukup canggih untuk menangani fungsi matematika dan operator kompleks, secara otomatis mendefinisikan simbol yang belum dideklarasi.


## BAGIAN 4: KNOWLEDGE BASE PROPOSISIONAL

### Cell 15: Inisialisasi PropKB
```python
wumpus_kb = PropKB()
```
**Penjelasan:** Membuat knowledge base kosong untuk menyimpan pengetahuan dalam bentuk kalimat logika proposisional. KB adalah komponen inti dari knowledge-based agent.

**Konsep Teori:** Knowledge Base adalah kumpulan sentences yang diekspresikan dalam bahasa representasi pengetahuan. KB menyimpan fakta dan aturan tentang dunia.


### Cell 16: Definisi Simbol Wumpus World
```python
P11, P12, P21, P22, P31, B11, B21 = expr('P11, P12, P21, P22, P31, B11, B21')
```
**Penjelasan:** Mendefinisikan simbol untuk Wumpus World: P[i,j] = "ada pit di posisi [i,j]", B[i,j] = "ada breeze di posisi [i,j]". Subscript angka menunjukkan koordinat grid.

**Konsep Teori:** Dalam Wumpus World, agent menavigasi gua 4×4 mencari emas sambil menghindari pit dan monster. Percept memberikan informasi parsial tentang lingkungan.


### Cell 17: Operasi TELL - Menambah Fakta
```python
wumpus_kb.tell(~P11)
```
**Penjelasan:** Operasi TELL menambahkan pengetahuan baru ke KB. Fakta ~P11 berarti "tidak ada pit di [1,1]", ini adalah starting position yang aman.

**Konsep Teori:** TELL dan ASK adalah dua operasi fundamental pada KB. TELL menambah pengetahuan, ASK mengquery pengetahuan yang sudah ada.


### Cell 18: TELL Aturan Breeze-Pit
```python
wumpus_kb.tell(B11 | '<=>' | ((P12 | P21)))
wumpus_kb.tell(B21 | '<=>' | ((P11 | P22 | P31)))
```
**Penjelasan:** Menambahkan aturan dunia: square memiliki breeze jika dan hanya jika ada pit di square tetangga (atas/bawah/kiri/kanan, bukan diagonal).

**Konsep Teori:** Biconditional (⇔) menyatakan ekuivalensi logika. B ⇔ (P1 ∨ P2) berarti B true tepat ketika salah satu dari P1 atau P2 true. Ini adalah pengetahuan tentang physics dari Wumpus World.


### Cell 19: TELL Percept
```python
wumpus_kb.tell(~B11)
wumpus_kb.tell(B21)
```
**Penjelasan:** Menambahkan observasi agent: tidak ada breeze di [1,1], ada breeze di [2,1]. Percept adalah informasi sensorik yang diterima agent dari environment.

**Konsep Teori:** Agent mengintegrasikan percept dengan pengetahuan tentang aturan dunia untuk melakukan inferensi tentang aspek lingkungan yang tidak terlihat langsung.


### Cell 20: Inspeksi Clauses dalam KB
```python
wumpus_kb.clauses
```
**Penjelasan:** Menampilkan semua klausa dalam KB yang sudah dikonversi ke CNF (Conjunctive Normal Form). Biconditional otomatis diubah menjadi dua implikasi lalu ke bentuk disjungsi.

**Konsep Teori:** CNF adalah konjungsi dari disjungsi literal. Setiap kalimat logika dapat dikonversi ke CNF. Bentuk ini standar untuk banyak algoritma inferensi seperti resolution.


## BAGIAN 5: KNOWLEDGE-BASED AGENT PROGRAM

### Cell 21: Source Code KB_AgentProgram
```python
psource(KB_AgentProgram)
```
**Penjelasan:** KB Agent bekerja dalam loop: terima percept → update KB → query aksi terbaik → eksekusi aksi → beritahu KB tentang aksi yang diambil.

**Konsep Teori:** Agent function mengambil percept dan mengembalikan action. KB Agent melakukan ini dengan maintaining representasi eksplisit pengetahuan dan menggunakan logical inference.


## BAGIAN 6: TRUTH TABLE ENUMERATION

### Cell 22-23: Algoritma tt_entails
```python
psource(tt_check_all)
psource(tt_entails)
```
**Penjelasan:** Truth table enumeration adalah model checking: enumerate semua kemungkinan model (assignment truth value ke simbol) dan cek apakah query true di semua model dimana KB true.

**Konsep Teori:** 
- **Entailment**: KB ⊨ α berarti α logically follows dari KB
- **Model**: assignment truth values yang membuat sentence true
- **Model Checking**: verifikasi M(KB) ⊆ M(α), dimana M(S) adalah set semua model dari S

**Contoh Wumpus World**: Jika KB tahu tidak ada breeze di [1,1], maka KB entails tidak ada pit di [1,2] dan [2,1], karena breeze muncul di square adjacent ke pit.


### Cell 24-26: Testing tt_entails
```python
tt_entails(P & Q, Q)      # True - jika P dan Q true, pasti Q true
tt_entails(P | Q, Q)      # False - P atau Q bisa saja hanya P yang true
tt_entails(P | Q, P)      # False - sama, bisa saja hanya Q yang true
```
**Penjelasan:** Demonstrasi konsep entailment. Konjungsi entails komponennya, tapi disjungsi tidak entail komponen individual karena bisa salah satu saja yang true.

**Konsep Teori:** Soundness berarti algoritma inferensi hanya derive kalimat yang benar-benar entailed. tt_entails adalah sound karena mengecek semua model.


### Cell 27: Entailment Kompleks
```python
tt_entails(A & (B | C) & D & E & ~(F | G), A & D & E & ~F & ~G)
```
**Penjelasan:** KB kompleks entails subset faktanya. Jika kita tahu A, D, E true dan F, G false (plus fakta lain), kita pasti tahu subset fakta tersebut.


### Cell 28-29: Query Wumpus KB
```python
wumpus_kb.ask_if_true(~P11), wumpus_kb.ask_if_true(P11)      # (True, False)
wumpus_kb.ask_if_true(~P22), wumpus_kb.ask_if_true(P22)      # (False, False)
```
**Penjelasan:** Query ~P11 return True karena KB bisa buktikan tidak ada pit di [1,1]. Query P22 dan ~P22 kedua return False karena KB tidak punya informasi cukup untuk memutuskan.

**Konsep Teori:** Ketika KB tidak bisa entail α atau ¬α, ini berarti KB tidak punya informasi cukup. Agent harus mencari informasi lebih atau menggunakan strategi lain (misal probabilistik reasoning).


## BAGIAN 7: CONJUNCTIVE NORMAL FORM (CNF)

### Cell 30-31: Konversi ke CNF
```python
psource(to_cnf)
psource(eliminate_implications)
psource(move_not_inwards)
psource(distribute_and_over_or)
```
**Penjelasan:** Konversi CNF dalam 4 langkah:
1. Eliminasi biconditional: A ⇔ B → (A ⇒ B) ∧ (B ⇒ A)
2. Eliminasi implikasi: A ⇒ B → ¬A ∨ B
3. Pindahkan NOT ke dalam: ¬(A ∨ B) → ¬A ∧ ¬B (De Morgan)
4. Distribusi OR over AND: (A ∨ (B ∧ C)) → (A ∨ B) ∧ (A ∨ C)

**Konsep Teori:** CNF adalah bentuk standar dimana kalimat adalah AND dari OR dari literal. Semua kalimat proposisional dapat dikonversi ke CNF. Bentuk ini diperlukan untuk resolution theorem proving.


### Cell 32-35: Contoh Konversi CNF
```python
to_cnf(A |'<=>'| B)                          # ((A | ~B) & (B | ~A))
to_cnf(A |'<=>'| (B & C))                    # ((A | ~B | ~C) & (B | ~A) & (C | ~A))
to_cnf(A & (B | (C & D)))                    # (A & (C | B) & (D | B))
to_cnf((A |'<=>'| ~B) |'==>'| (C | ~D))     # kompleks
```
**Penjelasan:** Berbagai contoh konversi menunjukkan bagaimana equivalence, implikasi, dan nested struktur ditransformasi menjadi conjunction of disjunctions.


## BAGIAN 8: RESOLUTION THEOREM PROVING

### Cell 36-38: Algoritma pl_resolution
```python
psource(pl_resolution)
pl_resolution(wumpus_kb, ~P11)  # True
pl_resolution(wumpus_kb, P22)   # False
```
**Penjelasan:** Resolution adalah proof by contradiction: untuk buktikan KB ⊨ α, buktikan KB ∧ ¬α unsatisfiable. Jika mendapat empty clause (kontradiksi), maka α entailed.

**Konsep Teori:** 
- **Resolution Rule**: (l₁ ∨ ... ∨ lₖ) ∧ (m₁ ∨ ... ∨ mₙ) ∧ (lᵢ ≡ ¬mⱼ) ⇒ (l₁ ∨ ... lᵢ₋₁ ∨ lᵢ₊₁ ... ∨ lₖ ∨ m₁ ... mⱼ₋₁ ∨ mⱼ₊₁ ... ∨ mₙ)
- Jika resolve menghasilkan empty clause, KB ∧ ¬α unsatisfiable, maka KB ⊨ α
- Resolution adalah complete untuk propositional logic


## BAGIAN 9: FORWARD CHAINING

### Cell 39-40: Forward Chaining Implementation
```python
psource(PropDefiniteKB.clauses_with_premise)
psource(pl_fc_entails)
```
**Penjelasan:** Forward chaining untuk Horn clauses (implikasi dengan konjungsi positif di premise, satu positif di conclusion). Algoritma iteratif: jika semua premise true, add conclusion ke KB.

**Konsep Teori:** 
- **Horn Clause**: disjungsi literal dengan max 1 positif literal
- **Definite Clause**: Horn clause dengan exactly 1 positif (implikasi)
- Forward chaining: data-driven, mulai dari fakta, aplikasikan rules, derive fakta baru
- Kompleksitas linear terhadap ukuran KB


### Cell 41-46: Testing Forward Chaining
```python
clauses = ['(B & F)==>E', '(A & E & F)==>G', ...]
pl_fc_entails(definite_clauses_KB, expr('G'))  # True
pl_fc_entails(definite_clauses_KB, expr('I'))  # False
```
**Penjelasan:** Membangun KB dengan Horn clauses, test queries. G dapat di-infer melalui chain of reasoning, I tidak bisa karena tidak ada rule yang conclude I.

**Konsep Teori:** Forward chaining memodelkan cara manusia reasoning dari fakta ke kesimpulan. Efisien untuk situational reasoning dimana banyak fakta diketahui.


## BAGIAN 10: DPLL ALGORITHM

### Cell 47-48: DPLL untuk SAT Solving
```python
psource(dpll)
psource(dpll_satisfiable)
```
**Penjelasan:** DPLL adalah complete, sound algorithm untuk SAT (Boolean Satisfiability Problem). Menggunakan backtracking search dengan tiga optimisasi penting.

**Konsep Teori:**
- **Early Termination**: deteksi true/false dari partially completed model
- **Pure Symbol Heuristic**: simbol yang muncul dengan sign sama di semua clauses dapat di-assign untuk satisfy clauses tersebut
- **Unit Clause Heuristic**: clause dengan 1 literal hanya bisa satisfied satu cara, propagate assignment
- **Unit Propagation**: series of forced assignments dari unit clauses

SAT adalah NP-complete problem, DPLL adalah algoritma praktis yang bisa handle problem besar dengan heuristics yang baik.


### Cell 49-54: Testing DPLL
```python
dpll_satisfiable(A & B & ~C & D)                    # {A: True, B: True, C: False, D: True}
dpll_satisfiable((A & B) | (C & ~A) | (B & ~D))    # satisfiable
dpll_satisfiable(A |'<=>'| B)                       # {A: True, B: True}
```
**Penjelasan:** DPLL return model jika satisfiable, False jika unsatisfiable. Model adalah assignment variables yang satisfy sentence.


## BAGIAN 11: WALKSAT ALGORITHM

### Cell 55-60: WalkSAT - Stochastic SAT Solver
```python
psource(WalkSAT)
WalkSAT([A, B, ~C, D], 0.5, 100)                # random search
WalkSAT([A & B, C | D, ~(D | B)], 0.5, 1000)   # None (unsatisfiable)
```
**Penjelasan:** WalkSAT adalah incomplete stochastic algorithm. Mulai random assignment, iteratif flip variable untuk increase satisfied clauses. Parameter p mengontrol random vs greedy.

**Konsep Teori:**
- **Local Search**: tidak sistematis, bisa stuck di local optimum
- **Random Walk**: probability p flip random variable di unsatisfied clause
- **Greedy**: probability (1-p) flip variable yang maximize satisfied clauses
- Lebih cepat dari DPLL untuk problem besar, tapi tidak guaranteed menemukan solusi
- Cocok untuk optimization problem dimana "almost satisfied" sudah cukup baik


### Cell 61-65: WalkSAT Wrapper dan Comparison
```python
def WalkSAT_CNF(sentence, p=0.5, max_flips=10000):
    return WalkSAT(conjuncts(to_cnf(sentence)), 0, max_flips)

%%timeit dpll_satisfiable(...)
%%timeit WalkSAT_CNF(...)
```
**Penjelasan:** Wrapper untuk accept sentence penuh. Perbandingan runtime menunjukkan trade-off: WalkSAT lebih cepat tapi incomplete, DPLL lebih lambat tapi complete.


## BAGIAN 12: SATPLAN - PLANNING AS SAT

### Cell 66-68: SATPlan Algorithm
```python
psource(SAT_plan)
transition = {'A': {'Left': 'A', 'Right': 'B'}, ...}
SAT_plan('A', transition, 'C', 2)  # plan: ['Right', 'Right']
```
**Penjelasan:** Planning problem direformulasi sebagai SAT problem. Encode initial state, transitions, goal sebagai CNF formula dengan time steps. SAT solver menemukan sequence actions.

**Konsep Teori:**
- **Planning**: menemukan sequence actions dari initial ke goal state
- **Encoding**: 
  - State variables: State(s,t) = "di state s pada time t"
  - Action variables: Action(a,t) = "lakukan action a pada time t"
  - Constraints: initial state, transition rules, goal, exclusion (max 1 state/action per time)
- SAT solver menemukan assignment yang satisfy semua constraints
- Jika satisfiable, extract plan dari true action variables


## BAGIAN 13: FIRST-ORDER LOGIC

### Cell 69-75: Criminal KB dalam FOL
```python
clauses.append(expr("(American(x) & Weapon(y) & Sells(x, y, z) & Hostile(z)) ==> Criminal(x)"))
clauses.append(expr("Enemy(Nono, America)"))
clauses.append(expr("Owns(Nono, M1)"))
clauses.append(expr("Missile(M1)"))
crime_kb = FolKB(clauses)
```
**Penjelasan:** Membangun FOL KB tentang law dan geopolitics. Menggunakan:
- **Variables** (x, y, z): quantified over objects
- **Constants** (Nono, America, M1, West): specific objects
- **Predicates** (American, Weapon, Sells, Criminal, etc.): relations
- **Functions**: implicitly dalam terms

**Konsep Teori (First-Order Logic):**
- Lebih ekspresif dari propositional logic: bisa express properties dan relations
- **Quantifiers**: ∀x (universal), ∃x (existential)
- **Terms**: variables, constants, functions
- **Atomic sentences**: Predicate(term₁, ..., termₙ)
- **Complex sentences**: kombinasi atomic dengan connectives dan quantifiers

**Analisis Criminal KB:**
"American yang menjual weapon ke hostile nation adalah criminal" + fakta tentang Nono, missiles, dan West → kesimpulan: West is criminal


### Cell 76-78: Substitution dalam FOL
```python
psource(subst)
subst({x: expr('Nono'), y: expr('M1')}, expr('Owns(x, y)'))
# Result: Owns(Nono, M1)
```
**Penjelasan:** Substitution θ = {x/t} mengganti variable x dengan term t dalam sentence. Fundamental operation untuk unification dan inference di FOL.

**Konsep Teori:** Substitution composition: (θ₁ ∘ θ₂)(x) = θ₁(θ₂(x)). Ground substitution: mengganti semua variables dengan constants.


### Cell 79-83: Unification Algorithm
```python
unify(expr('x'), 3)                                      # {x: 3}
unify(expr('Cat(x) & Dog(Dobby)'), expr('Cat(Bella) & Dog(y)'))  # {x: Bella, y: Dobby}
unify(expr('Cat(x)'), expr('Dog(Dobby)'))               # None (predicate berbeda)
unify(expr('Cat(x) & Dog(Dobby)'), expr('Cat(Bella) & Dog(x)'))  # None (x tidak bisa jadi 2 nilai)
```
**Penjelasan:** Unification menemukan most general unifier (MGU): substitusi paling umum yang membuat dua expression identik.

**Konsep Teori:**
- **Unifier**: substitution θ dimana θ(p) = θ(q)
- **MGU**: unifier paling general (tidak ada substitusi ekstra yang tidak perlu)
- **Occur Check**: variable x tidak boleh unify dengan term yang contain x
- Unification adalah basis untuk resolution dan inference di FOL


## BAGIAN 14: FORWARD CHAINING FOL

### Cell 84-86: Forward Chaining untuk FOL
```python
psource(fol_fc_ask)
answer = fol_fc_ask(crime_kb, expr('Hostile(x)'))
list(answer)  # [{x: Nono}, {x: JaJa}]
```
**Penjelasan:** Forward chaining di FOL: enumerate substitutions untuk variables, unify premise dengan facts di KB, jika match add instantiated conclusion.

**Konsep Teori:**
- Pattern matching dengan unification
- Enumerate semua possible substitutions (ground terms dari KB)
- Check apakah substituted premises ada di KB
- Add new conclusions sampai tidak ada yang baru atau menemukan jawaban

**Perhatian**: fol_fc_ask modifies KB (adds inferred facts), tidak seperti query pure.


## BAGIAN 15: BACKWARD CHAINING FOL

### Cell 87-90: Backward Chaining Implementation
```python
psource(fol_bc_or)   # OR: coba semua rules yang bisa prove goal
psource(fol_bc_and)  # AND: prove semua conjuncts di premise
crime_kb.ask(expr('Hostile(x)'))  # {v_5: x, x: Nono}
```
**Penjelasan:** Backward chaining adalah goal-directed search. Untuk prove goal:
1. **OR**: Cari rules dengan conclusion yang unify dengan goal
2. **AND**: Untuk tiap rule, prove semua premises (rekursif)
3. Standardize variables untuk avoid name clashes

**Konsep Teori:**
- **Goal-driven**: mulai dari query, work backward ke facts
- **AND-OR Search**: OR choice rules, AND prove all premises
- **Depth-first search**: bisa infinite loop, butuh cycle checking
- Lebih efisien dari forward chaining jika goal spesifik
- Digunakan dalam Prolog dan logic programming

**Standardization**: rename variables di rules untuk avoid conflict. Variable baru (v_5, dll) muncul dari proses ini.


## BAGIAN 16: VISUALISASI

### Cell 99: Interactive Visualization
```python
canvas_bc_ask = Canvas_fol_bc_ask('canvas_bc_ask', crime_kb, expr('Criminal(x)'))
```
**Penjelasan:** Visualisasi interaktif backward chaining search tree. Menunjukkan:
- Goal node di top
- Sub-goals yang perlu diprove
- Unification steps
- Facts dari KB yang match
- Path dari goal ke facts (proof)

**Konsep Teori:** Proof tree menunjukkan struktur logical reasoning. Setiap node adalah goal, edges adalah aplikasi rules atau match dengan facts. Leaf nodes adalah facts di KB.


## CONTOH KONSEP UTAMA

### 1. Logical Agents Architecture
- **Knowledge Base**: repository of sentences
- **TELL**: add knowledge
- **ASK**: query knowledge
- **Inference**: derive new sentences from existing

### 2. Propositional Logic
- **Syntax**: symbols, connectives, sentences
- **Semantics**: truth values, models
- **Entailment**: KB ⊨ α
- **Inference algorithms**: truth tables, resolution, forward/backward chaining

### 3. SAT Solving
- **DPLL**: complete systematic search dengan heuristics
- **WalkSAT**: incomplete stochastic local search
- **Applications**: planning, verification, optimization

### 4. First-Order Logic
- **Expressiveness**: quantifiers, functions, predicates
- **Unification**: pattern matching algorithm
- **Inference**: forward chaining (data-driven), backward chaining (goal-driven)

### 5. Trade-offs dalam AI Reasoning
- **Completeness vs Efficiency**: complete algorithms lambat, incomplete cepat
- **Soundness**: hanya derive entailed sentences
- **Expressiveness vs Tractability**: FOL lebih ekspresif tapi lebih kompleks


Sumber (dari github) : [https://github.com/NewbieProgramm/Prak_KKA/blob/main/README.md](https://github.com/NewbieProgramm/Prak_KKA/blob/main/README.md)
