# `reload!` в `rails c`

Создаём новый проект `rails` и делаем модель `MyModel`.
Заходим в `rails c`:

```ruby
  m = MyModel.new
  m.is_a? MyModel # true
  reload!
  m.is_a? MyModel # false
```