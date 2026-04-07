#flashcards/errors/multierror

Когда оправдано использовать multierror вместо возврата первой ошибки?
?
![[Multierror#^me-scenario]]

Что возвращает multierror.Append(nil, err1, err2)? Какой тип?
?
![[Multierror#^me-append]]

Что вернёт multierror.Append(nil, nil, nil)?
?
![[Multierror#^me-append-nil]]

Почему возвращать (error, error, error) хуже чем multierror?
?
![[Multierror#^me-why-not-multi-return]]

Работают ли errors.Is/errors.As если multierror обёрнут через fmt.Errorf("%w", multi)?
?
![[Multierror#^me-is-walks-list]]

Как errors.Is ищет ошибку внутри multierror — по первой или по всему списку?
?
![[Multierror#^me-is-walks-list]]

Опиши паттерн накопления ошибок при обработке среза с помощью multierror.Append.
?
![[Multierror#^me-append-example]]
