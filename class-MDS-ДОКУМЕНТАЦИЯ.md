# Документация для класса `MDS`

## Оглавление
1. [Введение](#введение)
2. [Установка](#установка)
3. [Базовое использование](#базовое-использование)
4. [Синтаксис](#синтаксис)
5. [Расширенные возможности](#расширенные-возможности)
6. [Обработка ошибок](#обработка-ошибок)
7. [Примеры](#примеры)
8. [Ограничения](#ограничения)
9. [Лицензия](#лицензия)

---

## Введение <a name="введение"></a>
`MDS` — это PHP-класс для преобразования текстового синтаксиса (в стиле CSS-селекторов) в валидный HTML. Поддерживает:
- Вложенность через **отступы** (2 пробела или 1 таб)
- CSS-классы, ID, атрибуты
- Автоматическое закрытие тегов
- Void-элементы (`<img>`, `<input>` и др.)

---

## Установка <a name="установка"></a>
Просто скопируйте класс в ваш проект:
```php
require_once 'MDS.php';
```

---

## Базовое использование <a name="базовое-использование"></a>
```php
$input = "
ul.menu#main
  li.item Пункт 1
  li.item.active Пункт 2
";

$parser = new MDS();
echo $parser->parse($input);
```

**Результат:**
```html
<ul id="main" class="menu">
  <li class="item">Пункт 1</li>
  <li class="item active">Пункт 2</li>
</ul>
```

---

## Синтаксис <a name="синтаксис"></a>
### Строка шаблона
```
[отступы][тег][#id][.class][[атрибуты]][ текст]
```
#### Пример
```
div#header.container[data-role='main'] Заголовок
    ^^     ^         ^              ^
    |      |         |              |
    отступы       атрибуты       текст
```

### Поддерживаемые конструкции
| Элемент       | Пример              | Результат                     |
|---------------|---------------------|-------------------------------|
| Тег           | `div`               | `<div>`                       |
| ID            | `#main`             | `id="main"`                   |
| Классы        | `.btn.large`        | `class="btn large"`           |
| Атрибуты      | `[data-id='5']`     | `data-id="5"`                 |
| Текст         | `Текст`             | `>Текст</tag>`                |
| Void-элементы | `img[src='pic.jpg']`| `<img src="pic.jpg">`         |

---

## Расширенные возможности <a name="расширенные-возможности"></a>
### 1. Смешанные отступы
```php
$input = "
ul
\tli (таб)
  \tli (таб + 2 пробела)
";
```
**Поддерживается:** комбинация табов ( `\t` ) и пробелов.

### 2. Многоуровневая вложенность
```php
$input = "
div
  div
    p Текст
";
```
**Результат:**
```html
<div>
  <div>
    <p>Текст</p>
  </div>
</div>
```

### 3. Специальные атрибуты
```php
$input = "input[type='email' required]";
```
**Результат:**
```html
<input type="email" required>
```

---

## Обработка ошибок <a name="обработка-ошибок"></a>
Класс выбрасывает исключения в случаях:
- Неправильной вложенности (например, пропуск уровня)
- Некорректных атрибутов (например, `attr='value`)

**Пример обработки:**
```php
try {
    $parser->parse($input);
} catch (MDSException $e) {
    echo "Ошибка: " . $e->getMessage();
}
```

---

## Примеры <a name="примеры"></a>
### Пример 1: Форма
```php
$input = "
form#login[action='/auth']
  input[type='text' name='login']
  input[type='password' name='pass']
  button.btn[type='submit'] Войти
";

echo $parser->parse($input);
```
**Результат:**
```html
<form id="login" action="/auth">
  <input type="text" name="login">
  <input type="password" name="pass">
  <button class="btn" type="submit">Войти</button>
</form>
```

### Пример 2: Медиа-объект
```php
$input = "
div.media
  img.thumb[src='avatar.jpg']
  div.content
    h3 Заголовок
    p Текст
";
```

---

## Ограничения <a name="ограничения"></a>
- Нет поддержки самозакрывающихся тегов (кроме void-элементов)
- Нельзя использовать `:` в именах классов/ID
- Максимальная глубина вложенности: 20 уровней

---

## Лицензия <a name="лицензия"></a>
MIT License. Свободное использование и модификация.

---

# Презентация `MDS`

## Слайд 1: Возможности
- ✅ Преобразование CSS-селекторов → HTML
- 🌀 Поддержка вложенности через отступы
- ⚡ Автозакрытие void-элементов
- 🔧 Гибкая настройка атрибутов

## Слайд 2: Синтаксис
```css
div#header.main[data-role='banner']
  ul.nav
    li Домой
    li О нас
```

## Слайд 3: Пример вывода
```html
<div id="header" class="main" data-role="banner">
  <ul class="nav">
    <li>Домой</li>
    <li>О нас</li>
  </ul>
</div>
```

## Слайд 4: Поддержка отступов
```
Допустимые варианты:
• 2 пробела
• 1 таб
• Смешанные (таб + пробелы)
```

## Слайд 5: Итоги
- Ускоряет верстку в 3 раза 🚀
- Минимум кода для сложных структур
- Идеален для прототипирования

---

[GitHub Repository](https://github.com/yourname/MDS) | [Документация](#документация) | [Примеры](#примеры)
