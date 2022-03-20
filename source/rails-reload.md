[На главную](index.md)

# `reload!` в `rails c`

Создаём новый проект, `rails` и делаем модель `MyModel`.
Заходим в `rails c`:

```ruby
  m = MyModel.new
  m.is_a? MyModel # true
  reload!
  m.is_a? MyModel # false
```

То есть после перезагрузки `m` перестал быть экземпляром класса `MyModel`.
Держите это в уме, если часто делаете `reload!`, и используете проверки через `is_a?`.

[На главную](index.md)