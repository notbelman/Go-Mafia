- **Применимо только на Intel Ice Lake / AMD Zen 4+** (AVX-512 + GFNI). +10% снижение GC CPU overhead поверх базового [[Green Tea GC (new)]]. Включено по умолчанию в Go 1.26
- **Почему стало возможным:** все объекты страницы одного size class → структура регулярная. Весь metadata страницы (seen + scanned биты) умещается в **2 × 512-bit регистра** → обработка страницы без обращений к RAM
- **6 шагов scanning kernel:** `seen XOR scanned` → active objects bitmap → bit expand (1 бит/объект → 1 бит/word) через `VGF2P8AFFINEQB` → AND с pointer/scalar bitmap → active pointer bitmap → итерация **64 байта за раз**
- **`VGF2P8AFFINEQB`** (GFNI): аффинное преобразование байта как вектора 8 бит на матрицу 8×8 в **GF(2)** (умножение=AND, сложение=XOR). Матрицы разные для каждого size class, генерируются автоматически code generator'ом

---

## Почему vector стал возможен

Старый GC (graph flood): объекты разных размеров, обход нерегулярный → **невозможно** использовать SIMD. Каждый шаг зависит от предыдущего, данные непредсказуемы. ^va-why-old-impossible

[[Green Tea GC (new)]]: все объекты на странице **одного size class** → размер объекта фиксирован для всей страницы → регулярность → применимы vector инструкции. ^va-why-possible

Страница 8 KiB с объектами одного размера → metadata (seen + scanned биты) умещается в **два 512-bit регистра**. Весь metadata страницы на CPU, без обращений к памяти. ^va-registers

## Scanning kernel (AVX-512)

```
Страница P:
  seen bits:    [0 1 0 1 1 0 ...]   ← в регистр
  scanned bits: [0 0 0 1 0 0 ...]   ← в регистр
                        ↓ шаг 2
  active_objects = seen AND (NOT scanned)
               = [0 1 0 0 1 0 ...]  ← нужно просканировать
                        ↓ шаг 3 (VGF2P8AFFINEQB)
  active_words (6-word объекты):
    [000000 111111 000000 000000 111111 000000 ...]
                        ↓ шаг 4
  pointer/scalar bitmap: [010100 101001 ...]  ← от аллокатора
                        ↓ шаг 5
  active_pointers = active_words AND ptr_bitmap
                        ↓ шаг 6
  итерация 64 байта за раз → буфер pointers
```

**Шаг 1.** Загрузить `seen` и `scanned` биты страницы в два 512-bit регистра. Один бит на каждый объект-слот. ^va-step1

**Шаг 2.** Вычислить **active objects bitmap**:
```
active_objects = seen AND (NOT scanned)   ← объекты для обработки в этом проходе
new_scanned    = seen OR scanned          ← обновить scanned bits
```
^va-step2

**Шаг 3.** **Bit expansion** через `VGF2P8AFFINEQB`: преобразовать active_objects из формата «1 бит / объект» в формат «1 бит / word (8 байт)».

Пример для объектов размером 6 words (48 байт):
```
active_objects:  0 0 1 1 ...           (1 бит на объект)
↓ expand
active_words:    000000 000000 111111 111111 ...  (6 бит на объект)
```
^va-step3

**Шаг 4.** Загрузить **pointer/scalar bitmap** страницы из метаданных аллокатора. Один бит на word: 1 = pointer, 0 = scalar. ^va-step4

**Шаг 5.** Вычислить **active pointer bitmap**:
```
active_pointers = active_words AND pointer_scalar_bitmap
```
Результат: 1 бит на каждое слово страницы, 1 = здесь pointer в живом необработанном объекте. ^va-step5

**Шаг 6.** Итерация по active pointer bitmap, **64 байта за раз** (512-bit регистр). Загрузить pointer значения → записать в буфер для дальнейшей маркировки `seen` bits. ^va-step6

## VGF2P8AFFINEQB

Инструкция из набора **Galois Field New Instructions (GFNI)**, доступна на Intel Ice Lake и AMD Zen 4+. ^va-gfni

Выполняет аффинное преобразование: рассматривает каждый байт вектора как вектор из 8 бит и умножает на матрицу 8×8 в поле **GF(2)**. ^va-gfni-math

**GF(2) арифметика:**
```
умножение = AND
сложение  = XOR
```

За одну инструкцию: произвольная побитовая перестановка над байтом. ^va-gf2

**Применение в Green Tea:** для каждого size class нужна своя матрица расширения (1 бит объекта → N бит для N words объекта). Матрицы рассчитываются заранее как константы. Код генерируется автоматически. ^va-matrices

## Связь
- [[Green Tea GC (new)]] — базовый алгоритм, который это ускоряет
- [[Обзор GC]] — фаза Mark, где применяется vector scanning
