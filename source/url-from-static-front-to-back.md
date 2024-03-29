[На главную](index.md)

# Ссылка из статичного фронта на бэк

Предположим есть приложение состоящее из двух частей: *front* и *back*.

*Front* - чистый js, статика, должен знать url *back* части.

Как передать туда url *back* части без перекомпиляции?

```seqdiag
diagram {
  browser => proxy => front;
}
```

Если написать в js относительный путь, например `/back` он будет ссылаться на тот же адрес, с которого вы его получили.
Таким образом, если front и back живут на одном домене, то роутинг можно возложить на сервер, а во `front` части писать только относительные пути.

```seqdiag
diagram {
  browser -> proxy -> front;
  browser <- proxy <- front [label = "js with `/back` url"];
  browser -> proxy -> back [label = "`/back`"]
}
```

По факту, действительный адрес бэка будет знать ваш прокси-сервер, а фронт будет знать, что всё что начинается с `/back` должно отправляться только туда.

[На главную](index.md)