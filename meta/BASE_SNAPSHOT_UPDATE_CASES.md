й# BASE SNAPSHOT: что обновлять при изменениях базы

Файл-цель для обновления: `/Users/caoguojun0/Documents/Obsidian/main/interviews/BASE_SNAPSHOT.md`

Если в сообщениях/заметках используется опечатка `BASE_SNAPHOT`, считать, что речь про `BASE_SNAPSHOT.md`.

## Быстрый алгоритм обновления
1. Зафиксировать, что именно изменилось (Theory / Questions / структура / нейминг).
2. Пересчитать затронутые метрики командами из этого файла.
3. Обновить только нужные секции в `BASE_SNAPSHOT.md` по матрице ниже.
4. Проверить согласованность чисел (`.md`, доменные count, coverage).

## Матрица кейсов

| Кейс изменения | Что править в `BASE_SNAPSHOT.md` | Что пересчитать/обновить |
|---|---|---|
| Добавили/удалили любой `.md` в `main/interviews` | `## Общая карта базы` | Общий `.md` count, при необходимости `prompts/questions/theory` count |
| Добавили/удалили `.png` или `.canvas` | `## Общая карта базы` | `.png`/`.canvas` count |
| Добавили новый домен в `theory` (новая папка 1-го уровня) | `## Общая карта базы`, `## Theory: домены и объём`, `## Остальные домены Theory` | Дерево 2 уровня, строка домена в ранжировании, описание кластера/сэмпла |
| Добавили/удалили карточки в `theory/Go/<подтема>` | `## Deep Dive: Go` | Таблица Go: `md_count`, `flashcards_count`, `flashcards_coverage`, `что разобрано` |
| Переименовали файлы в Go-подтеме | `## Deep Dive: Go` | Поле `что разобрано` (basename) и, если нужно, summary по обязательным блокам |
| Добавили/удалили `flashcards` в Go-подтеме | `## Deep Dive: Go` | `flashcards_count`, `flashcards_coverage` конкретной подтемы |
| Изменили материалы в `theory/Go/runtime/scheduler` | `### GMP / Scheduler` | `md_count`, `flashcards_count`, список покрытия, GMP checklist |
| Добавили/удалили карточки в `theory/PostgreSQL/JOIN` | `## Deep Dive: PostgreSQL`, `### JOIN`, `Coverage checklist: JOIN` | `md_count`, список `что разобрано`, checklist (виды JOIN/ON-WHERE/NULL/LATERAL/perf) |
| Добавили/удалили карточки в `theory/PostgreSQL/Индексы` | `## Deep Dive: PostgreSQL`, `### Индексы`, `Coverage checklist: Индексы` | `md_count`, список `что разобрано`, checklist (типы/composite/partial/covering/bloat/reindex) |
| Изменения в `Транзакции`, `Оптимизация запросов`, `replication-cards`, `Шардирование`, `Партиционирование` | `## Deep Dive: PostgreSQL` + соответствующий `###` блок | `md_count`, список `что разобрано` для каждой затронутой подпапки |
| Изменения в `theory/sd/...` | `## Deep Dive: System Design (sd)` | Таблица разделов (`md_count`, `flashcards_count`, кластеры), критичные кластеры coverage |
| Добавили/удалили карточки в других theory-доменах | `## Theory: домены и объём`, `## Остальные домены Theory` | Ранжирование доменов и сэмплы `что разобрано` |
| Добавили/удалили вопрос в `questions/theory` | `## Questions layer: ...` | Таблица распределения `topic`, блок аномалий нейминга, связка theory ↔ questions |
| Добавили/удалили задачу в `questions/tasks` | `## Questions layer: ...`, `## Качество базы...` | Таблица `topic`, возможные дубли slug в задачах |
| Добавили/удалили файл компании в `questions/by-companies` | `## Questions layer: ...` | Список компаний (факт наличия файла) |
| Исправили topic alias (например `Networks` -> `Networking`) | `## Questions layer: ...`, `## Качество базы...` | Обновить таблицы topic + блок аномалий + рекомендации |
| Почистили `.DS_Store` | `## Качество базы...` | Количество `.DS_Store` и список примеров |
| Нашли/устранили дубли файлов | `## Качество базы...` | Блоки дублей (`questions/tasks`, `questions/theory`) |

## Точные секции и поля, которые чаще всего забывают

### 1) Go
- Обязательно синхронизировать сразу два места:
  - общая таблица Go-подтем (`md_count`, `flashcards_count`, `flashcards_coverage`, `что разобрано`)
  - обязательный блок подтемы (`### Каналы`, `### errors`, `### map` и т.д.)
- Для `runtime/scheduler` всегда перепроверять `GMP checklist`.

### 2) PostgreSQL
- Для `JOIN` и `Индексы` недостаточно обновить только `md_count`.
- Нужно синхронно обновлять:
  - блок `### JOIN`/`### Индексы`
  - соответствующий `Coverage checklist`.

### 3) Questions
- После правок `questions/theory` нужно смотреть не только count, но и аномалии нейминга topic.
- После правок `questions/tasks` проверять дубли slug.

## Команды пересчёта по кейсам

### База и верхний уровень
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews
find . -type f -name '*.md' | wc -l
find . -type f -name '*.png' | wc -l
find . -type f -name '*.canvas' | wc -l
for d in prompts questions theory; do
  echo "$d=$(find "$d" -type f -name '*.md' | wc -l | tr -d ' ')"
done
```

### Theory домены
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews/theory
for d in */; do
  echo "${d%/} $(find "$d" -type f -name '*.md' | wc -l | tr -d ' ')"
done | sort -k2 -nr
```

### Go: подпапки + flashcards coverage
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews/theory/Go
for d in */; do
  md=$(find "$d" -type f -name '*.md' | rg -v '/flashcards/' | wc -l | tr -d ' ')
  fc=$(find "$d" -type f -name '*.md' | rg '/flashcards/' | wc -l | tr -d ' ')
  cov=no; [ "$fc" -gt 0 ] && cov=yes
  echo "${d%/}|$md|$fc|$cov"
done | sort
```

### PostgreSQL: обязательные блоки
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews/theory/PostgreSQL
for d in JOIN Индексы "Транзакции" "Оптимизация запросов" replication-cards Шардирование Партиционирование; do
  echo "$d=$(find "$d" -type f -name '*.md' | wc -l | tr -d ' ')"
done
```

### Questions: topic распределения
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews
python3 - <<'PY'
from pathlib import Path
from collections import Counter
import re


def topic(p):
    t = p.read_text(encoding='utf-8', errors='ignore')
    m = re.search(r'^---\n(.*?)\n---\n', t, flags=re.S)
    if not m:
        return '(no topic)'
    m2 = re.search(r'(?m)^topic\s*:\s*(.+?)\s*$', m.group(1))
    return m2.group(1).strip().strip('"\'') if m2 else '(no topic)'

for sec in ['questions/theory', 'questions/tasks']:
    c = Counter(topic(p) for p in Path(sec).rglob('*.md'))
    print('\n'+sec)
    for k,v in sorted(c.items(), key=lambda x:(-x[1], x[0])):
        print(f'{k}: {v}')
PY
```

### Дубли в задачах
```bash
cd /Users/caoguojun0/Documents/Obsidian/main/interviews
python3 - <<'PY'
from pathlib import Path
from collections import defaultdict
import re


def norm(s):
    s=s.lower()
    s=re.sub(r'[_\-\s]+','',s)
    s=re.sub(r'[^0-9a-zа-яё]+','',s)
    return s

g=defaultdict(list)
for p in Path('questions/tasks').rglob('*.md'):
    g[norm(p.stem)].append(str(p))
for k,v in sorted(g.items()):
    if len(v)>1:
        print('DUP:', ' ; '.join(v))
PY
```

## Мини-регламент частоты обновления
- После любого массового импорта карточек: полный апдейт `BASE_SNAPSHOT.md`.
- После точечных правок в 1-2 подтемах: частичный апдейт только затронутых секций.
- Перед подготовкой к собеседованию по компании: обязательно обновить `Questions layer` и `Качество базы`.
