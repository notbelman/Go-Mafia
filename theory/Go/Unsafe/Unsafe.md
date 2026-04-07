## Unsafe в Go

#### [[unsafe.Pointer и uintptr]]
unsafe.Pointer — типобезопасный указатель без типа, uintptr — просто число

#### [[Правила unsafe.Pointer]]
6 правил конвертации, когда GC может сломать uintptr

#### [[Sizeof,Alignof,Offsetof]]
unsafe.Sizeof, unsafe.Alignof, unsafe.Offsetof — размеры и выравнивание

#### [[unsafe — практические use cases]]
Конвертация string↔[]byte без аллокации, доступ к приватным полям
