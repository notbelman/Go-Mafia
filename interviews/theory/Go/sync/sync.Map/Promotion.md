**Что это:** dirty становится новым read. ^promotion-def

**Когда:** misses >= len(dirty) ^promotion-trigger

**До:**
```
read:  { a: *val, b: *val }           amended=true
dirty: { a: *val, b: *val, c: *val }
misses: 3
```

**После:**
```
read:  { a: *val, b: *val, c: *val }  amended=false
dirty: nil
misses: 0
```
^promotion-before-after

dirty просто стал read. Старый read ушёл в GC. ^promotion-gc

**Что происходит технически:** atomic.Store нового readOnly в read, dirty = nil, misses = 0. Операция под mu. ^promotion-mechanics
